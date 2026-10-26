---
paperclip_version: v2026.1023.0
seo_title: 工具网关 API
seo_description: 代理决定要做什么，工具网关决定是否允许执行。它位于每个代理与已连接的外部工具之间。
---

# 工具网关

代理擅长决定“要做什么”，**工具网关**则负责判断它们是否获准真正执行。网关位于代理和每个外部工具之间（例如 Model Context Protocol，MCP 服务器、电子表格或内部 HTTP 服务），负责协调每次调用：识别调用者、检查工具访问策略，然后允许、阻止、要求人工批准，或将请求延后到尚未就绪的运行时。所有决策都会写入可供查询的审计记录。

网关周围是你配置的各个组件：**应用**和**连接**是接入的对象；**目录**列出某连接实际提供的工具；**配置档**是命名的工具范围集合，可绑定到公司、代理、项目、例行任务、issue 或网关；**策略**是在其上叠加的规则，例如阻止、限速或先请求批准；**具名 MCP 网关**则是提供给外部 MCP 客户端的入口，让客户端使用标准 MCP，而 Paperclip 在后台应用上述控制。

以下路由均限定在公司范围内，每次请求都会检查公司访问权限。若你刚接触此 API，请先阅读 [API 概览](./overview.md)，了解基础 URL、认证和错误码约定；本页不再重复这些内容。

> **实验性功能，默认关闭。** 工具、配置档和应用界面受实例实验设置控制，操作员必须先启用 `enableApps`。本页末尾的 Smoke Lab 另受 `enableSmokeLab` 控制；关闭时其所有路由都返回 `404 Not Found` 和 `{ "error": "Smoke lab is disabled" }`。整套功能仍属实验阶段，请注意请求和响应结构可能随版本变化。

## 权限

看板调用方在访问敏感路由时需要对应权限键。本地隐式看板操作者和实例管理员无需检查；其他用户必须获得相应权限，否则返回 `403 Forbidden` 和 `{ "error": "Missing permission: <key>" }`。

| 权限键 | 管理范围 |
|---|---|
| `tools:admin` | 创建、列出和更新具名 MCP 网关；签发或撤销网关令牌；管理获准的 stdio 命令模板。 |
| `tools:manage_runtime` | 列出运行时槽位，以及停止或重启槽位。 |
| `tools:view_audit` | 查看网关审计动态。 |
| `tools:manage_connections` | 同步连接的安装对象。 |
| `tools:use` | 对连接执行测试调用；拥有此权限或 `tools:manage_connections` 即可。 |

部分创建或编辑配置的路由不检查权限键，而要求用户是公司中的**有效且非只读成员**。只读查看者会收到 `403 Forbidden` 和 `{ "error": "Viewer access is read-only" }`；没有有效成员资格的用户会收到 `{ "error": "User does not have active company access" }`。

## 组成部分

### 应用和连接

**应用**是要接入的服务，**连接**是访问它的一种配置方式。应用有 `type` 和 `status`；连接有传输方式 `transport`（`remote_http` 或 `local_stdio`）、自身状态（`draft`、`active`、`disabled`、`archived`）以及 `enabled` 标记。连接被禁用、归档或未启用时，所有经由它的调用都会直接被拒绝，原因码为 `deny_disabled_connection`；应用处于 `disabled` 或 `archived` 状态时也相同。

### 目录条目

刷新连接会导入其**目录**，每个条目对应一个工具，包含工具的 `name`、`description`、`inputSchema` 和 `riskLevel`。风险级别固定为 `low`、`medium`、`high`、`critical`、`read`、`write`、`destructive`。每个条目还带有 `versionHash` 和 `schemaHash`，Paperclip 据此发现先前信任的工具是否悄然改变了结构。

### 配置档、条目和绑定

**配置档**是一组命名工具，默认操作 `defaultAction` 为 `deny` 或 `allow`。其**条目**进一步限定范围：每项有 `selectorType`（`application`、`connection`、`catalog_entry`、`tool_name` 或 `risk_level`）和 `effect`（`include` 或 `exclude`）。配置档只有经过**绑定**才会生效。绑定的 `targetType` 可以是 `company`、`agent`、`project`、`routine`、`issue` 或 `gateway`；同一次调用匹配多个绑定时，范围最窄的优先。

### 策略

**策略**按 `priority` 从小到大的顺序在每次调用时评估。策略包含 `policyType`、决定适用调用的 `selectors`、必须同时成立的可选 `conditions`，以及限速策略使用的 `config`。

常用类型为 `allow`、`block`、`require_approval` 和 `rate_limit`。`trust_rule` 较特殊，只能通过信任规则提升和撤销路由管理；直接创建会返回 `422` 和 `Trust rules are managed through the trust-rule promotion and revoke endpoints`。`redact` 与 `validate` 会被直接拒绝，并返回 `Tool policy type '<type>' is not supported at runtime`。

`rate_limit` 策略的 `config` 必须包含大于零的数值 `limit` 和 `windowSeconds`，否则返回 `422 Unprocessable Entity` 和 `Rate-limit policy config requires positive numeric limit and windowSeconds`。可选的 `keyBy` 指定限额统计对象；默认按公司、代理、连接和工具分桶。

条件分为 `arguments`、`actor`、`context`、`risk`、`credentialScope`、`trustBoundary` 和 `timeWindow` 等组，策略适用要求所有条件均成立。`arguments` 组支持 `fieldEquals`、`fieldNotEquals`、`fieldIn`、`fieldMatches`、`fieldExists` 和 `fieldAbsent`。若 `fieldMatches` 的模式可能带来危险计算，会在创建时拒绝，并返回 `Tool policy fieldMatches includes an unsafe regular expression`。

<a id="explicit-tool-grants"></a>
### 显式工具授权

除了配置档和策略，代理或人员还可以持有已存储的 `tools:use` 权限授权。如果策略尚未对调用作出决定，Paperclip 会先检查此授权，再检查配置档；匹配时以 `allow_explicit_grant` 为原因放行。

授权的 `scope` 精确决定覆盖范围：

- `null` 或空对象 `{}` 表示宽泛授权，覆盖所有工具。
- `allow`（单个字符串或字符串列表）必须能匹配本次工具。每项格式为 `tool:<name>`（网关名或上游工具名）、`connection:<connectionId>` 或 `application:<applicationId>`。列表无匹配项时授权不覆盖调用；空列表，以及含空白或非字符串项的列表，均不覆盖任何调用。
- 其他键会进一步收窄范围，而且必须全部匹配。支持的单值键为 `actorType`、`agentId`、`projectId`、`routineId`、`issueId`、`gatewayId`、`applicationId`、`connectionId`、`catalogEntryId`、`applicationKey`、`providerType`、`toolName`、`riskLevel`；也支持对应的复数形式（如 `agentIds`、`toolNames`），其值必须是非空字符串列表。
- Paperclip 不认识的键、类型错误的值，或非对象的 scope，都会使授权不匹配。

不匹配的授权不会扩大为更宽泛的放行；调用会继续检查配置档，若配置档也不允许，最终原因为 `deny_default`。工具发现也遵守相同规则，因此授权范围外的工具不会出现在工具列表中。如果代理升级前可用的工具消失，请先检查授权的 `scope`。

### 决策结果

每次经网关协调的调用最终会得到五种决策之一：`allow`、`deny`、`require_approval`、`rate_limited` 或 `defer_runtime`。结果还包含 `reasonCode`（例如 `deny_disabled_connection`、`allow_trust_rule`、`deny_policy_block`）、人类可读的 `explanation`、生效的 `effectiveProfileIds` 和命中的 `matchedPolicyIds`。

## 具名 MCP 网关

具名网关为外部 MCP 客户端（编辑器、桌面代理或其他服务）提供统一的受管端点。客户端使用你签发的 Bearer 令牌认证，获取工具列表并调用工具；Paperclip 根据令牌解析网关，应用网关配置档和策略，然后只转发获准的调用。

### 列出网关

```
GET /api/companies/{companyId}/tools/gateways
```

需要 `tools:admin`。列表返回 `{ "gateways": [ ... ] }`。

### 创建网关

```
POST /api/companies/{companyId}/tools/gateways
```

需要 `tools:admin`，创建成功返回 `201 Created`。创建字段如下：

| 字段 | 必填 | 说明 |
|---|---|---|
| `name` | 是 | 1–160 个字符，公司内唯一。 |
| `profileId` | 是 | 此网关提供的工具配置档。 |
| `slug` / `displaySlug` | 否 | 各为 1–120 个字符。 |
| `description` | 否 | 最多 4,000 个字符，或 `null`。 |
| `defaultProfileMode` | 否 | `gateway_only`（默认）、`inherit_context_then_gateway` 或 `gateway_then_context`。 |
| `contextScopeType` | 否 | 默认 `none`。 |
| `contextScopeId` | 否 | 1–200 个字符，或 `null`。 |
| `agentId`、`projectId`、`issueId`、`approvalIssueId` | 否 | UUID 或 `null`。 |
| `authConfig`、`headerPolicy`、`metadataPolicy`、`onDemandToolsConfig` | 否 | 配置见下文。 |
| `metadata` | 否 | 任意 JSON 对象，或 `null`。 |

无效负载会返回 `422 Unprocessable Entity`：`{ "error": "Invalid gateway payload", "issues": [ ... ] }`。

- **`headerPolicy`**：控制是否透传调用方请求头（`callerPassthrough`，默认关闭且 `allowedHeaders` 为空）、始终附加的 `staticHeaders`、`generatedMetadata` 请求头，以及允许返回的 `responseHeaders`。默认会转发 MCP 要求的请求头和安全缓存请求头。
- **`metadataPolicy`**：控制允许向上游工具传递哪些标识：`forwardCompanyId`、`forwardGatewayId`、`forwardProjectId`、`forwardIssueId`、`forwardAgentId`、`forwardRunId` 和 `forwardCorrelationId`。默认仅启用 `forwardCorrelationId`，不会泄露公司信息。
- **`authConfig`**：Bearer 令牌行为。默认启用 Bearer 认证，令牌前缀为 `pcgw`，默认有效期 90 天，启用 `requireFiniteExpiry` 和 `longLivedTokenRequiresOverride`。OAuth 已有配置入口，但默认禁用。
- **`onDemandToolsConfig`**：默认关闭。启用后会提供 `search_tools` 和 `run_tool` 两个元工具，而不是返回冗长的静态工具列表，避免大型目录占满客户端上下文。

### 更新网关


需要 `tools:admin`。请求体必须包含 `companyId`，否则返回 `400 Bad Request` 和 `{ "error": "companyId is required" }`。创建字段在此处均为可选项，并可设置 `status`（`draft`、`active`、`disabled`、`archived`）；至少要提供一个字段。

```
PATCH /api/tool-gateway/gateways/{gatewayId}
```

<a id="mint-a-gateway-token"></a>
### 签发网关令牌


签发令牌需要 `tools:admin`，且请求体必须包含 `companyId`。成功返回 `201 Created`。

| 签发字段 | 必填 | 说明 |
|---|---|---|
| `name` | 是 | 1–160 个字符。 |
| `clientLabel` | 是 | 1–160 个字符，说明令牌供哪个客户端使用。 |
| `ownerNote` | 是 | 1–1,000 个字符，说明所有者及用途。 |
| `subjectType` | 否 | 当前仅接受 `gateway_client`。 |
| `subjectId` | 否 | 1–240 个字符，或 `null`。 |
| `allowedActions` | 否 | `tools/list`、`tools/call` 中的任意项，默认两者都允许。 |
| `expiresAt` | 否 | 日期；设为 `null` 表示永不过期。 |
| `expiryOverrideReason` | 否 | `expiresAt` 为 `null` 时必填。 |

`subjectType` 若不是 `gateway_client` 会被拒绝：`heartbeat_run` 由运行时管理，`board_user` 和 `agent` 为后续 OAuth 及用户绑定流程保留。无过期时间的令牌需要显式设置 `expiresAt: null` 和 `expiryOverrideReason`，否则返回 `Non-expiring gateway tokens require an override reason`。

<a id="revoke-a-gateway-token"></a>

```
POST /api/tool-gateway/gateways/{gatewayId}/tokens
```

### 撤销网关令牌


撤销令牌需要 `tools:admin`，且请求体必须包含 `companyId`。

```
POST /api/tool-gateway/gateway-tokens/{tokenId}/revoke
```

### MCP 端点

可以通过公开 ID 或内部 UUID 访问同一网关：


网关通过仅支持 `POST` 的 MCP 可流式 HTTP 传输提供服务。`GET` 返回 `405 Method Not Allowed` 和 `Allow: POST`，提示 MCP 客户端无需建立服务器发送事件流。

`POST` 使用 JSON-RPC。通过 `Authorization: Bearer <gateway-token>` 认证；缺少令牌时返回 `401 Unauthorized` 和 `{ "error": "Bearer token is required" }`。

| 方法 | 行为 |
|---|---|
| `initialize` | 返回 `protocolVersion` `2025-03-26`、声明 `tools` 能力的 `capabilities`，以及名称为 `Paperclip MCP Gateway` 的 `serverInfo`。 |
| `notifications/initialized` | 返回 `202 Accepted`，无响应体。 |
| `tools/list` | 返回网关配置档允许的工具，包含 `name`、`title`、`description` 和 `inputSchema`。 |
| `tools/call` | 执行工具，使用 `params.name` 和 `params.arguments`。缺少名称时返回 JSON-RPC 错误 `-32602` 和 `params.name is required`。 |

其他方法返回 `404` 及 JSON-RPC 错误 `-32601`（`Method not found`）。调用被拒时，JSON-RPC `error` 对象的 `data.reasonCode` 可用于区分策略阻止和上游故障。

Paperclip 会限制并发工具列表请求数。达到上限且等待队列也已满时，`tools/list` 返回 `503 Service Unavailable`、JSON-RPC 错误 `-32603`、消息 `Tool discovery is busy. Retry shortly.`，且 `data.reasonCode` 为 `tool_discovery_busy`。这不表示网关或令牌有问题，稍后重试即可。



```
GET  /mcp/gateways/{gatewayPublicId}
POST /mcp/gateways/{gatewayPublicId}
GET  /api/tool-gateway/gateways/{gatewayId}/mcp
POST /api/tool-gateway/gateways/{gatewayId}/mcp
```

## 代理会话

具名网关面向外部客户端。Paperclip 内部运行的代理使用**会话**：作用域限定于单个代理和单次运行的短期令牌。

<a id="create-a-session"></a>
### 创建会话

```
POST /api/tool-gateway/sessions
```

看板或代理调用方均可创建会话。代理自己的 `companyId`、`agentId` 和 `runId` 从其令牌读取；看板调用方需在请求体提供这三项，可另传 `issueId`、`projectId` 和 `ttlMs`。缺少任一必填 ID 时返回 `400 Bad Request` 和 `{ "error": "companyId, agentId, and runId are required" }`。

`201 Created` 响应会返回代理所需信息：

```json
{
  "sessionId": "...",
  "token": "...",
  "expiresAt": "2026-07-20T12:00:00.000Z",
  "toolsUrl": "/api/tool-gateway/tools",
  "callUrl": "/api/tool-gateway/tools/call"
}
```

<a id="revoke-a-session"></a>
### 撤销会话

```
POST /api/tool-gateway/sessions/{sessionId}/revoke
```

响应为 `{ "sessionId": ..., "revokedAt": ... }`。代理只能撤销自己的会话。

### 列出和调用工具


两者都使用 `x-paperclip-tool-gateway-token` 请求头中的会话令牌认证，**不是** `Authorization`。缺少令牌时返回 `401 Unauthorized` 和 `{ "error": "Tool gateway session token is required" }`。

如果服务器正在处理的工具列表请求已达上限，列出工具可能返回 `503 Service Unavailable` 和 `{ "error": "Tool discovery is busy. Retry shortly.", "reasonCode": "tool_discovery_busy" }`。这是暂时的背压信号，请稍等后重试。

调用请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `tool` | 是 | 工具名。缺失或非字符串时返回 `400` 和 `"tool" is required and must be a string`。 |
| `parameters` | 否 | 工具参数，默认 `{}`。 |
| `timeoutMs` | 否 | 等待上游工具的时长，最多 60 秒，默认 10 秒。 |
| `approvedActionRequestId` | 否 | 先前调用返回 `require_approval` 时，用于兑现相应批准。 |
| `idempotencyKey` | 否 | 保障重试安全；相同键不会重复运行工具。 |

若提供方在调用过程中拒绝已保存的 OAuth 令牌，Paperclip 会刷新令牌，但不会重放调用。响应为 `409 Conflict`，代码 `oauth_refreshed_retry_required`，消息为 *"The provider rejected the saved OAuth token, which has now been refreshed. Retry the action explicitly; the original call was not replayed."* 确认重复执行安全后，再显式发送调用。

```
GET  /api/tool-gateway/tools
POST /api/tool-gateway/tools/call
```

### 代理授权自己的连接

部分连接需要代理自行完成授权，例如完成 OAuth 流程或为上游服务签发短期凭据。代理可使用自己的运行令牌调用以下路由；仅活动运行中的代理可用：


`start-authorization` 会启动 OAuth 授权并返回 `{ "url": ... }`；适用时还会返回 `handoff` 对象。请将用户引导至 `url` 指定的地址完成授权。请求体可选包含 `subjectUserId`、`scopes` 和 `returnTo`。

`token` 会签发代理可向上游服务出示的短期令牌。必须有运行 ID；没有时会以 `401` 和 `{ "error": "Agent run id required", "code": "run_id_required" }` 拒绝。若同时发送 `X-Paperclip-Run-Id` 请求头，其值必须与令牌中的运行 ID 匹配，否则返回 `403` 和 `run_id_mismatch`。成功时返回 `200`，并包含 `status: minted`、`token`、`tokenType`、`expiresAt`、`ttlSeconds`、`scope` 和 `grantId`。如果连接使用静态凭据而不是凭据代理，签发会被 `409 Conflict` 拒绝，`status` 为 `use_env_lease`，提示应通过有审计记录的环境租约访问凭据。



```
POST /api/agents/me/connections/{connectionId}/start-authorization
POST /api/agents/me/connections/{connectionId}/token
```

## 审批与信任规则

策略要求 `require_approval` 时，此调用会成为**操作请求**，而不是立即执行；随后由人员决定。


列表路由接受 `status` 查询参数，默认 `pending`。批准和拒绝操作都需要 `companyId`，可放在请求体或查询参数中，并要求有效的非只读成员资格。

每个操作请求都保存 `canonicalArgumentsHash` 和经删减的 `canonicalArgumentsSummary`，确保人员批准的参数与稍后执行的参数完全一致；批准一次调用不会暗中授权另一组参数。许多请求还包含 `previewMarkdown`，用可读方式概述工具即将执行的操作。

多次批准同一操作后，可将其提升为长期规则：


信任规则会记住创建时对应的目录 `versionHash` 和 `schemaHash`。若上游工具之后被隔离、移除或改变结构，该规则会自动失效，调用重新进入审批流程。信任绑定的是实际批准过的工具版本，而不只是工具名称。

```
GET  /api/companies/{companyId}/tools/action-requests
POST /api/tool-gateway/action-requests/{id}/approve
POST /api/tool-gateway/action-requests/{id}/decline
```

```
POST /api/companies/{companyId}/tools/action-requests/{actionRequestId}/trust-rule
POST /api/companies/{companyId}/tools/trust-rules/{policyId}/revoke
GET  /api/companies/{companyId}/tools/trust-rules
```

## 数据删减

Paperclip 不会存储原始工具参数。写入审计记录前，参数值会被概述和清理：看起来像秘密的键（例如 `api_key`、`authorization`、`password`、`token`、`client_secret`）会被替换为 `[REDACTED]`；符合已知凭据格式的值也会被替换。长字符串会截断，长数组会限制长度。响应中的 `redactionPlan` 包含 `redactedFieldCount`、`redactedFields` 列表，以及可用于跨调用比较、但不会暴露原始值的哈希。

## 运行时槽位

`local_stdio` 连接会启动实际进程，**运行时槽位**是 Paperclip 对该进程的管理对象，记录其状态、健康度、最近错误和空闲截止时间。


以上路由均需 `tools:manage_runtime`。公司路径也提供相同操作及健康状态查询：


由于 stdio 连接会启动命令，因此必须受获准的**命令模板**清单限制：


以上路由均需 `tools:admin`。模板会固定 `command`、`args`、允许读取的 `envKeys` 和提供的 `tools`，确保连接只能启动管理员审核过的命令。

```
GET  /api/tool-gateway/runtime-slots?companyId={companyId}
POST /api/tool-gateway/runtime-slots/{slotId}/stop
POST /api/tool-gateway/runtime-slots/{slotId}/restart
```

```
GET  /api/companies/{companyId}/tools/runtime-slots
POST /api/companies/{companyId}/tools/runtime-slots/{id}/stop
POST /api/companies/{companyId}/tools/runtime-slots/{id}/restart
GET  /api/companies/{companyId}/tools/runtime-health
```

```
GET  /api/companies/{companyId}/tools/stdio-templates
POST /api/companies/{companyId}/tools/stdio-templates
POST /api/companies/{companyId}/tools/stdio-templates/{templateId}/disable
```

## 审计动态


需要 `tools:view_audit`，用于查看网关执行过的所有操作。

| 查询参数 | 说明 |
|---|---|
| `window` | `1h`、`24h`、`7d` 或 `30d`，默认 `24h`。其他值返回 `400` 和 `window must be one of 1h, 24h, 7d, 30d`。 |
| `limit` | 1–100，默认 `100`。 |
| `app` | `applicationId` 或 `connectionId` UUID；非 UUID 返回 `400` 和 `app must be an applicationId or connectionId UUID`。 |
| `agent` | `agentId` UUID，同样会验证。 |
| `outcome` | 结果过滤条件，见下表。 |
| `search` | 自由文本，服务器会在代理、应用和连接名称，以及操作名称、工具名称和原因码中匹配。 |
| `cursor` | 上一页返回的 `nextCursor`。无法读取时返回 `400` 和 `Invalid audit cursor`。 |

每条事件都带有易读的 `normalizedOutcome`：

| `normalizedOutcome` | 含义 |
|---|---|
| `allowed` | 调用已放行。 |
| `asked_first` | 执行前请求了批准。 |
| `waiting` | 调用被延后，通常是等待尚未就绪的运行时。 |
| `blocked` | 被策略或限速规则阻止。 |
| `failed` | 已执行但发生错误。 |
| `unknown` | 无法归入以上结果。 |

事件还会附带用于显示的 `agentDisplayName`、`appDisplayName`、`applicationDisplayName`、`connectionDisplayName` 和 `toolDisplayName`。响应格式为 `{ "events": [ ... ], "nextCursor": ... }`；将 `nextCursor` 传回以获取下一页，直到其值为 `null`。底层操作包括 `tool_gateway.session_created`、`session_revoked`、`session_rejected`、`discovery`、`call_allowed`、`call_denied`、`call_completed`、`call_failed`、`call_deferred`、`approval_requested` 和 `runtime_mcp_delivery`。

```
GET /api/tool-gateway/audit?companyId={companyId}
```

## 通过界面接入应用

应用界面提供了上述功能的简便入口：从目录库选择服务并连接，Paperclip 会生成合适的配置档和先询问再执行的规则。


`preflight` 返回绘制目录应用连接界面所需的元数据；可选传入 `methodKey` 查询参数，预览指定认证方式。`connect` 一步创建应用、连接和目录，成功返回 `201 Created`。若应用使用 OAuth，响应中的 `auth` 对象会包含 `startUrl`，需要引导用户打开该地址。随后 `finish` 根据 `access` 创建配置档、条目、绑定和先询问规则，并在响应中告知各自创建数量。

仪表板可轮询 `attention`，获取需要人工处理的应用，例如健康检查失败、凭据过期或出现尚未审核的新工具。

OAuth 另提供以下两个路由：


必须配置公开基础 URL，否则返回 `422 Unprocessable Entity` 和 `OAuth connections require PAPERCLIP_PUBLIC_URL or an auth public base URL`。

也可从现有配置文件开始：


此路由将 MCP JSON 配置预览为连接草稿，不会创建任何内容，直到你确认操作。

预置示例是可安装并可就地冒烟测试的完整配置：


```
GET  /api/companies/{companyId}/tools/gallery
GET  /api/companies/{companyId}/tools/apps/{galleryKey}/preflight
POST /api/companies/{companyId}/tools/apps/connect
POST /api/companies/{companyId}/tools/apps/{connectionId}/finish
GET  /api/companies/{companyId}/tools/apps/attention
```

```
POST /api/tools/oauth/{connectionId}/start
GET  /api/tools/oauth/callback
```

```
POST /api/companies/{companyId}/tools/mcp/import-json
```

```
GET  /api/companies/{companyId}/tools/examples
POST /api/companies/{companyId}/tools/examples/{id}/install
POST /api/companies/{companyId}/tools/examples/{id}/smoke
```

## 直接管理应用、连接和配置档

如需自行构建，各对象都提供常规 CRUD 接口。

应用：


连接：


删除连接实际上会将其归档，而不是彻底移除；如果它是应用的最后一个连接，应用也会一并归档。

连接可以**安装**到 `company` 或 `agent`，以指定谁能看到它：


`PUT` 会整体替换安装集合，并要求 `tools:manage_connections`。

#### 连接身份（授权）

同一连接可以有多个**身份**。`kind` 为 `organization` 的授权是连接上所有人共用的身份；`kind` 为 `user` 的授权则属于单个成员。例如，五个人可以共用同一 Gmail 连接，但各自以自己的身份操作。


**列表**路由返回授权、`capabilities` 对象、调用者的 `currentUserId`，以及连接管理员可用来选择受众的 `members`。返回内容受权限过滤：普通成员能看到组织授权和自己的个人授权，看不到其他人的个人身份。查看所有人身份需要连接管理权限 `tools:manage_connections`。

**`installations`** 路由用于添加授权，请求体可带 `providerTenant`、`credentialSecretRefs` 列表和 `isDefault`，成功返回 `201 Created`。**`members`** 的 `PUT` 通过 `memberUserIds` 数组替换授权的受众；创建者可执行，其他人需要配置访问权。**删除**授权会撤销它：你始终可以撤销自己拥有的授权（你是其主体或创建者），否则需要配置访问权。

**委派**允许个人授权所有者把身份借给代理。`POST` 需要 `agentId`，且必须由该用户本人调用；相应 `DELETE` 仅限所有者或连接管理员调用，用于收回委派。

这些路由大多要求**配置访问权**（你是连接创建者，或持有 `tools:manage_connections`），而不是某个单独权限键。只读查看者和没有有效公司访问权的成员会在其他检查前被拒绝。

#### 用量

连接详情还提供用量摘要：


`range` 默认 `7d`，仅接受 `7d` 或 `30d`；其他值返回 `400 Bad Request` 和 `Usage range must be 7d or 30d`。

#### 旧版 Composio 连接

旧版按工具包划分的 `/api/tool-connections/{connectionId}/services` 服务路由及其驱动的 Composio 代理已移除。Composio 现在是普通的远程 MCP 连接，见[通过 MCP 聚合器连接应用](../../connectors/mcp-aggregators.md)。

通过旧代理保存的连接仍然存在，但 Paperclip 不再运行它。读取时其 `healthStatus` 为 `error`，工具不会出现在发现结果中；针对它调用工具、刷新目录、重新连接或启动 OAuth 都会以 `422 Unprocessable Entity` 和代码 `composio_broker_retired` 拒绝。响应会说明处理方式：

> *"This legacy Composio connection is no longer supported. Add a new Composio MCP connection from Connectors, then remove this connection. Existing credentials and permissions are not migrated."*

中文释义：不再支持此旧版 Composio 连接。请从 Connectors 添加新的 Composio MCP 连接，然后移除此连接。现有凭据和权限不会迁移。（以上英文为接口返回的固定消息。）

配置档、条目和绑定：


最后一个路由回答最常见的问题：给定当前所有配置，这个代理实际能使用哪些工具？

连接刷新目录后，配置档从未见过的新工具会被暂时拦下，而不是悄然放行：


审核响应会返回 `allowedCount` 和 `keptBlockedCount`，以防上游服务器仅通过添加工具就扩大自身访问权限。

策略：


重新排序会按 100 的步长重写 `priority`，请求必须包含公司内**所有**非信任策略；只提交部分列表会返回 `422` 和 `Reorder must include every non-trust policy for the company`。复制策略始终会以**禁用**状态创建，以便你先编辑副本再启用。

```
GET    /api/companies/{companyId}/tools/applications
POST   /api/companies/{companyId}/tools/applications
PATCH  /api/tool-applications/{applicationId}
DELETE /api/tool-applications/{applicationId}
```

```
GET    /api/companies/{companyId}/tools/connections
POST   /api/companies/{companyId}/tools/connections
GET    /api/tool-connections/{connectionId}
PATCH  /api/tool-connections/{connectionId}
DELETE /api/tool-connections/{connectionId}
POST   /api/tool-connections/{connectionId}/health-check
POST   /api/tool-connections/{connectionId}/reconnect
POST   /api/tool-connections/{connectionId}/catalog/refresh
GET    /api/tool-connections/{connectionId}/catalog
GET    /api/tool-connections/{connectionId}/activity
```

```
GET /api/tool-connections/{connectionId}/installs
PUT /api/tool-connections/{connectionId}/installs
```

```
GET    /api/tool-connections/{connectionId}/grants
POST   /api/tool-connections/{connectionId}/grants/installations
DELETE /api/tool-connections/{connectionId}/grants/{grantId}
PUT    /api/tool-connections/{connectionId}/grants/{grantId}/members
POST   /api/tool-connections/{connectionId}/grants/{grantId}/delegations
DELETE /api/tool-connections/{connectionId}/grants/{grantId}/delegations/{delegationId}
```

```
GET /api/tool-connections/{connectionId}/usage?range={7d|30d}
```

```
GET    /api/companies/{companyId}/tools/profiles
POST   /api/companies/{companyId}/tools/profiles
PATCH  /api/tool-profiles/{profileId}
DELETE /api/tool-profiles/{profileId}
POST   /api/tool-profiles/{profileId}/duplicate
POST   /api/tool-profiles/{profileId}/entries
PATCH  /api/tool-profile-entries/{entryId}
DELETE /api/tool-profile-entries/{entryId}
POST   /api/companies/{companyId}/tools/profiles/{profileId}/bind
POST   /api/companies/{companyId}/tools/profiles/{profileId}/unbind
GET    /api/companies/{companyId}/tools/profiles/effective/agents/{agentId}
```

```
GET  /api/tool-profiles/{profileId}/new-tools
POST /api/tool-profiles/{profileId}/new-tools/review
```

```
GET    /api/companies/{companyId}/tools/policies
POST   /api/companies/{companyId}/tools/policies
PATCH  /api/companies/{companyId}/tools/policies/{policyId}
DELETE /api/companies/{companyId}/tools/policies/{policyId}
POST   /api/companies/{companyId}/tools/policies/{policyId}/duplicate
POST   /api/companies/{companyId}/tools/policies/reorder
```

## 信任前先测试

以下三个接口可让你演练调用而不必立即执行真实操作。

干运行用于查看“如果执行会怎样”：


它会用真实决策引擎评估假设调用，并返回 `{ "decision": ..., "auditEvent": ... }`，但不会执行工具。将 `writeAuditEvent` 设为 `true` 可记录此次演练；否则 `auditEvent` 为 `null`。

实时测试会以指定代理身份真正调用工具：


这些路由都要求 `tools:use` 或 `tools:manage_connections`，且只能以允许你指派工作的代理进行测试。`test-agents` 会列出代理及其对此连接的 `effectiveAccess`；`test-agents/{agentId}/access` 返回单个代理完整的连接 `access` 摘要。`test-calls` 接受 `agentId`、`toolName` 和可选的 `parameters`。若调用需要批准，响应会返回操作请求；再轮询最后一个路由查询状态。如果工具网关服务完全未配置，这些路由会返回 `501 Not Implemented` 和 `{ "error": "Tool gateway service is not configured" }`。

还可以查询某次代理运行中的决策结果：


```
POST /api/companies/{companyId}/tools/policy/test
```

```
GET  /api/tool-connections/{connectionId}/test-agents
GET  /api/tool-connections/{connectionId}/test-agents/{agentId}/access
POST /api/tool-connections/{connectionId}/test-calls
GET  /api/tool-connections/{connectionId}/test-calls/{actionRequestId}
```

```
GET /api/companies/{companyId}/tools/runs/{runId}/decisions
```

## Smoke Lab

Smoke Lab 是独立的演练环境，包含模拟 OAuth 提供方和一组环回测试服务，可在不接触真实供应商的情况下验证完整的连接、授权、调用和审计流程。

> **双重门控。** Smoke Lab 需要启用实验设置 `enableSmokeLab`，并且不能在公开暴露的实例上运行。关闭时这里的所有路由都会返回 `404 Not Found` 和 `{ "error": "Smoke lab is disabled" }`。


`install-fixtures` 会创建预配置的应用、连接、目录和配置档；首次返回 `201 Created`，已存在时返回 `200 OK`。`reset` 会将所有内容恢复初始状态。

运行记录演练实际执行的步骤：


运行包含 `trigger` 和 `status`；每个记录步骤包含 `path` 和 `status`。看板用户和代理都可以创建运行并记录步骤，因此代理也能自行驱动端到端检查。

模拟 OAuth 提供方位于同一前缀下，行为与真实提供方相同，针对它配置连接时不需要特殊处理：


重定向 URI 会与实例自身源地址比对，因此测试授权码不会被发送到外部主机。

```
GET  /api/companies/{companyId}/smoke-lab/services
POST /api/companies/{companyId}/smoke-lab/services/start
POST /api/companies/{companyId}/smoke-lab/services/stop
POST /api/companies/{companyId}/smoke-lab/install-fixtures
POST /api/companies/{companyId}/smoke-lab/reset
```

```
GET   /api/companies/{companyId}/smoke-lab/runs
POST  /api/companies/{companyId}/smoke-lab/runs
GET   /api/companies/{companyId}/smoke-lab/runs/{runId}
PATCH /api/companies/{companyId}/smoke-lab/runs/{runId}
POST  /api/companies/{companyId}/smoke-lab/runs/{runId}/steps
```

```
GET  /api/companies/{companyId}/smoke-lab/oauth/authorize
POST /api/companies/{companyId}/smoke-lab/oauth/authorize
POST /api/companies/{companyId}/smoke-lab/oauth/token
GET  /api/companies/{companyId}/smoke-lab/oauth/userinfo
POST /api/companies/{companyId}/smoke-lab/oauth/revoke
```

## 后续阅读

- [API 概览](./overview.md)：基础 URL、认证、公司范围和通用错误码表。
- [公司技能策略](./company-skill-policy.md)：将相同的防护机制应用于代理可使用的技能。
- [代理](./agents.md)：配置档和网关会话所关联的代理。
- [密钥](./secrets.md)：连接凭据的存储位置。

---
