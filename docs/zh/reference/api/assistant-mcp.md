---
paperclip_version: v2026.1023.0
seo_title: 助手 MCP 协议与工具
seo_description: 让助手以获得授权的 Paperclip 用户身份接入。发现工作和配置工具、传输任务文件、订阅事件并撤销访问权限。
---

# Assistant MCP

将外部助手连接到一个组织，并以获得授权的人类用户身份使用。你可以通过 MCP 工具查看工作、委派任务、添加反馈、获取结果，并在授权范围内更改配置。

此接口目前处于实验阶段，需要启用 `enablePublicMcp`。引导式设置流程见[助手连接](../../experimental/assistant-connections.md)。

<a id="instance-requirements"></a>
## 实例要求

在 Experimental 设置中启用 **Assistant connections (MCP)**。自托管和 Cloud 托管实例默认都关闭此项。实例还需要启用已验证用户账户和组织成员关系；隐式本地运维访问权限和智能体 API 密钥都不能批准人类 OAuth 授权。

使用已配置的身份验证公网 URL，或者明确设置来源地址：

```sh
PAPERCLIP_PUBLIC_URL=https://YOUR-PAPERCLIP-HOST
```

来源地址不能包含路径、内嵌凭据、查询参数或片段。只有 localhost 或回环地址的开发环境可以使用 HTTP。自托管私有部署需要使用能够直接访问该端点的客户端。在部署入口正确配置可信代理。

`enablePublicMcp` 会立即生效。关闭后，服务发现、登录、工具调用和事件交付都会被阻止。连接管理和撤销仍可使用，已委派的工作也会继续。

## 协议与授权

规范资源为：

```http
POST /mcp/paperclip
```

此端点使用无状态 HTTP，并返回 JSON。支持基于 initialize 的 MCP 客户端和 MCP `2026-07-28` 请求；不需要 GET/SSE 会话。每次调用都会重新检查 bearer 授权、当前成员关系和公司可用性。这些 OAuth token 不是普通 REST API 凭据。

| 端点 | 用途 |
|---|---|
| `/.well-known/oauth-protected-resource/mcp/paperclip` | 发现受保护的资源和权限范围。 |
| `/.well-known/oauth-authorization-server` | 发现 OAuth 功能和端点。 |
| `/mcp/oauth/register` | 注册公开 OAuth 客户端。 |
| `/mcp/oauth/authorize` | 启动浏览器授权。 |
| `/mcp/oauth/token` | 兑换授权码或刷新访问权限。 |
| `/mcp/oauth/revoke` | 撤销 token。 |
| `/mcp/oauth/device_authorization` | 为直连实例启动设备授权。 |
| `/mcp/setup` 和 `/mcp/setup.md` | 读取客户端设置说明。 |

浏览器授权使用 S256 PKCE，并将授权绑定到精确的资源地址。访问 token 在十五分钟后过期；`offline_access` 会启用可轮换的 30 天刷新 token。重放已使用的刷新 token 会撤销其授权。

公开客户端支持 Client ID Metadata Documents 和动态注册。人类用户仍需确认客户端身份和一个组织；仅完成注册不会授予任何权限。

### 权限范围

| 权限范围 | 直连实例中的权限 |
|---|---|
| `paperclip:read` | 必需。读取获准访问的工作和非密钥配置。 |
| `paperclip:write` | 修改任务、评论、文档、交付物和附件。 |
| `paperclip:configure` | 修改允许范围内的智能体、项目、指令和技能。 |
| `offline_access` | 无需每次会话都重新进行浏览器同意，即可刷新访问权限。 |

同意页面上的 **Write all of your Paperclip data** 复选框会在用户角色允许的情况下，一次性批准所有请求的变更权限范围。扩展权限范围需要重新进行同意流程。公司权限和领域规则仍可拒绝已授予权限范围内的个别调用。

## 发现工具

先调用 `paperclip_connection`，确认人员和组织身份，然后在公司范围的操作中使用返回的 `companyId`。通过 `paperclip_search_tasks` 搜索用户提供的任务标题或标识符，并在后续调用中使用返回的任务 UUID。

`tools/list` 会返回当前工具 schema。`paperclip_search_api` 会搜索同一封闭目录并返回操作 schema；`paperclip_call_api` 则会使用这些准确参数调用已发现的操作。这两个工具都不接受任意 REST 端点、URL、请求头或额外权限。

直连实例的目录包含 37 个工具：

| 工具 | 用途和修改范围 |
|---|---|
| `paperclip_connection` | 已连接的人员、组织、权限范围和管理链接。 |
| `paperclip_list_agents`、`paperclip_list_projects`、`paperclip_search_tasks`、`paperclip_read_task` | 查找智能体、项目和持久化工作。 |
| `paperclip_create_task`、`paperclip_add_comment`、`paperclip_update_task`、`paperclip_finish_task`、`paperclip_block_task` | 创建、引导或更改工作。需要写入权限。 |
| `paperclip_list_deliverables`、`paperclip_read_document`、`paperclip_list_document_revisions` | 获取已保存结果和文档历史。 |
| `paperclip_write_document`、`paperclip_register_deliverable`、`paperclip_update_deliverable` | 保存文档和任务交付物。需要写入权限。 |
| `paperclip_pending_approvals` | 读取待处理审批及其 Paperclip 决策链接。 |
| `paperclip_get_agent`、`paperclip_read_agent_instructions`、`paperclip_list_agent_instruction_revisions` | 读取获准范围内的智能体配置和指令历史。 |
| `paperclip_update_agent`、`paperclip_update_agent_instructions` | 更改获准范围内的智能体设置和指令。需要配置权限。 |
| `paperclip_get_project`、`paperclip_list_project_repositories` | 读取项目设置和可用仓库选项。 |
| `paperclip_create_project`、`paperclip_update_project`、`paperclip_set_project_repositories` | 创建或修改项目及仓库选择。需要配置权限。 |
| `paperclip_list_skills`、`paperclip_get_skill`、`paperclip_read_skill_file` | 查看公司技能及其文件。 |
| `paperclip_create_skill`、`paperclip_update_skill`、`paperclip_write_skill_file` | 按照常规技能策略创建或编辑技能。需要配置权限。 |
| `paperclip_get_upload_url`、`paperclip_get_download_url` | 传输任务附件。上传需要写入权限。 |
| `paperclip_search_api`、`paperclip_call_api` | 发现并调用同一组允许的操作。权限范围取决于所选操作。 |

智能体修改操作可以编辑身份信息、汇报关系、技能、模型、预算和现有 AI 连接绑定，但不能更改凭据、执行命令或权限策略。项目操作不会创建远程仓库。审核和审批决定仍在 Paperclip 中完成。

托管目录代理使用另一套规模更小、以工作为主的工具目录。不要将直连实例的配置、文件传输或 Events 支持视为代理功能。托管部署以及公共商店或目录的可用性需要单独验证；直连客户端配置不依赖列表服务。

## 写入与重试规则

每项有意发起的修改都使用新的 UUID `requestId`。重试时，使用同一 ID 和完全相同的参数。持久化回执会重放已确认结果；即使重新连接时仍是同一个人，只要参数发生变化就会被拒绝。

响应中的 `outcome: unknown` 表示操作仍在进行或结果尚未确认。再次操作前，先读取任务及其评论。新的请求 ID 可能会造成重复更改。`outcome: rejected` 表示已明确拒绝；重新提交前请阅读错误消息并检查当前状态。

更改文档或指令文件前，先读取文件，并提供返回的修订版本或哈希。写入技能文件时需要当前版本 ID。写入内容过时时会失败，不会覆盖更新的工作。

任务分配和反馈可能会安排智能体工作，并消耗公司已配置的预算。创建任务后会立即返回，不会等待执行结束；任务创建成功并不能证明智能体已经运行。有效运行所有权、审核门控、依赖关系和常规审批仍然适用。

## 传输任务文件

上传文件的步骤：

1. 在本地计算准确的文件字节数和 SHA-256 值。
2. 使用 `companyId`、`taskId`、`requestId`、`filename`、`contentType`、`byteSize` 和 `sha256` 调用 `paperclip_get_upload_url`。
3. 将准确的文件字节通过 PUT 上传到返回的临时 URL。
4. 适用时，使用返回的附件 ID 注册交付物。

PUT 成功后会立即保存附件。执行环境需要文件和 HTTP 工具才能传输；如果缺少这些工具，请从任务页面手动上传。

下载时，先列出交付物，再使用附件 ID 调用 `paperclip_get_download_url`。返回的链接有效期为十分钟，且只访问对应附件。临时 URL 等同于凭据；不要将其写入持久任务文本或日志。传输权限已撤销或过期时，请求会被拒绝。

## 订阅任务事件

支持 Events 的客户端可使用 MCP `2026-07-28` 的 `server/discover`、`events/list`、`events/subscribe` 和 `events/unsubscribe`。需提供匹配的 `MCP-Protocol-Version` 和 `Mcp-Method` 请求头、每请求协议元数据，以及工具调用所需的 `Mcp-Name`。

| 事件 | 过滤条件 | 通知数据 |
|---|---|---|
| `paperclip.task.status_changed` | `companyId`、`taskId`；可选 `statuses` | 任务状态和链接。 |
| `paperclip.task.comment_created` | `companyId`、`taskId` | 评论 ID 和任务链接。 |
| `paperclip.task.document_updated` | `companyId`、`taskId` | 文档键、修订编号和任务链接。 |

事件通过经过验证的 webhook 交付。客户端提供回调 URL，并可选提供签名密钥，然后负责管理刷新和取消订阅。只有用户要求监控该任务时，才应连接事件流。收到事件后，请读取当前工作状态；通知只携带标识符，不携带文档或评论正文。

仅建立连接并不会自动订阅事件。不支持 Events 的客户端（包括 CLI stdio 代理）可以轮询读取工具。关闭功能或撤销访问后，后续事件将停止交付。

## 管理和撤销连接

已通过身份验证的浏览器用户可通过以下端点管理自己的授权：

```http
GET    /api/mcp/setup
GET    /api/mcp/connections
DELETE /api/mcp/connections/{id}
```

设置响应会提供实时功能开关状态、规范服务器 URL 和非敏感邀请信息，但不会授予访问权限。删除操作要求来源地址符合已配置的浏览器来源，并且调用方是该授权的所有者。

连接列表中的每项包含 `id`、`companyId`、`clientName`、`companyName`、`scopes`、`createdAt` 和 `revokedAt`，另有 `user` 对象，其中包含授权者的 `name` 和 `image`。用户姓名不可用时，`user` 为 `null`；较旧服务器也可能省略此对象，因此应将其视为可选字段。列表仍会包含已撤销授权（`revokedAt` 已设置）；Paperclip 自身页面会过滤已撤销授权，只显示活动连接。

### 浏览器同意端点

Paperclip UI 中的同意页面会使用以下端点。此列表用于帮助你在日志中识别这些请求；助手不会调用它们。

| 端点 | 用途 |
|---|---|
| `GET /api/mcp/requests/{id}` | 在 `/mcp-connect/{id}` 页面查看待处理浏览器授权请求的详情。 |
| `POST /api/mcp/requests/{id}/consent` | 批准或拒绝此请求。需要已登录用户和 Paperclip 浏览器来源地址。 |
| `POST /api/mcp/requests/{id}/dot-pairing/preview` | 显示一次性 Dot 配对码将连接到哪个公司和智能体，但不使用该代码。 |
| `POST /api/mcp/requests/{id}/dot-pairing` | 兑换一次性 Dot 配对码并连接智能体。 |
| `GET /api/mcp/device?user_code=...` | 在 `/mcp-device` 页面查看待处理设备授权的详情。 |
| `POST /api/mcp/device/consent` | 通过用户码批准或拒绝设备授权。需要已登录用户和 Paperclip 浏览器来源地址。 |

Dot 端点也可通过 `/api/dot-mcp/requests/{id}` 下的专用别名调用，Dot 的连接页面会使用此别名。配对流程见[外部智能体](../../experimental/external-agents.md)。

设备授权会将 `/mcp-device` 作为其 `verification_uri` 返回，并在十分钟后过期。`paperclip_get_upload_url` 和 `paperclip_get_download_url` 返回的临时文件链接分别指向 `/mcp/files/upload` 和 `/mcp/files/download`；请使用返回的准确 URL，不要自行拼接。

组织范围的管理请使用 **Connectors → Assistant Connection (MCP)**；账户范围的管理请使用 `/assistant-connections`。撤销权限会阻止后续调用，但不会取消已委派任务或撤销正在进行的修改。

## 相关指南

- [助手连接](../../experimental/assistant-connections.md)
- [设备登录和 stdio 代理](../cli/mcp.md)
- [身份验证](authentication.md)
