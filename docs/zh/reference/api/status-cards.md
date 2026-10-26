---
seo_title: Status Cards API
seo_description: 关于公司工作的持续性问题及其动态答案。为状态卡提供一段通俗说明，描述要关注的内容和更新格式，并按需查看更新。
---

# 状态卡

**状态卡**是一个关于公司工作的持续性问题，以及对此问题不断更新的答案。你用一段通俗语言说明要关注什么、更新应如何呈现；Paperclip 的摘要器会将其编译为结构化的公司搜索查询，随后随着匹配任务的变化，持续编写 Markdown 摘要。

此 API 对应这一完整流程：创建状态卡后，Paperclip 会排入编译任务；请求刷新时，会排入更新任务；摘要器智能体通过两个专用端点写回编译后的查询和最终摘要，只有它可以调用这两个端点。

状态卡的作用范围限定在公司内，每个请求都会检查公司访问权限。如果你刚开始使用此 API，请先阅读 [API 概览](./overview.md)，了解基础 URL、身份验证和错误代码约定。本页基于这些内容展开，不会重复说明。

> **实验性功能——默认关闭。** 状态卡由实例级实验性设置控制。管理员启用 `enableStatusCards` 之前，本页的**所有**端点都会返回 `404 Not Found`，响应内容为 `{ "error": "Status cards are not enabled" }`。由于该功能仍处于实验阶段，以下请求和响应格式可能会在不同版本间变化。

关于状态卡面板本身、如何在界面中创建卡片、阅读卡片以及更新策略的作用，请参阅实验性章节中的[状态卡](../../experimental/status-cards.md)。

---

## 启用状态卡

此处的每个路由都会先检查实例是否启用了状态卡。服务器读取实例的实验性设置；如果 `enableStatusCards` 不为 `true`，就会立即停止处理，并返回：

```json
{ "error": "Status cards are not enabled" }
```

HTTP 状态为 `404 Not Found`。此开关作用于整个实例，而非单个公司；它同时控制 API 和界面。

除非卡片指定了自己的智能体，否则还必须为公司配置并启用内置的**摘要器**智能体。如果两者都不可用，任何会启动运行的请求都会失败，返回 `422 Unprocessable Entity`、`{ "error": "Summarizer built-in agent is not configured" }`，并在 `details` 对象中包含 `code: "summarizer_not_configured"` 及内置智能体当前的 `status`。请参阅[内置智能体](./built-in-agents.md)。

---

## 谁可以调用哪些接口

这些路由涉及三个访问级别：

| 级别 | 含义 |
|---|---|
| 公司访问权限 | 调用方可以查看该公司；拥有此权限即可调用读取路由。 |
| `tasks:assign` | 调用方可以向智能体分配工作。创建或管理状态卡都需要此权限，因为这两类操作都会排入智能体任务。没有此权限的面板用户会收到 `403 Forbidden` 和相应的授权说明。 |
| 仅摘要器 | 只有卡片指定的智能体（卡片的 `agentId` 覆盖值或内置摘要器）可以写入，而且必须在分配给它的生成任务所对应的运行中写入。 |

智能体还受到一项额外限制：**智能体只能管理由自己创建的状态卡**。尝试管理其他人创建的卡片会返回 `403 Forbidden` 和 `{ "error": "Agents can only manage status cards they authored" }`。在公司范围内，读取不受此限制。

智能体调用方受 `STATUS_CARD_AGENT_MAX_CARDS` 和 `STATUS_CARD_AGENT_MAX_INTEREST_PROMPT_LENGTH` 两项硬限制约束：

| 限制 | 数值 | 超限响应 |
|---|---|---|
| 单个智能体在一家公司内可创建的卡片数 | 20 | `422 Unprocessable Entity`，`{ "error": "Agents can author at most 20 status cards" }`。删除卡片会释放名额。 |
| 智能体调用方的 `interestPrompt` 长度 | 4,000 个字符 | `422 Unprocessable Entity`，`{ "error": "Agent-authored status card prompts cannot exceed 4000 characters" }`。 |

面板调用方仍受请求架构规定的通用 20,000 字符限制约束。

对于无法访问的卡片（属于其他公司或不存在），服务器始终以相同方式响应：`404 Not Found` 和 `{ "error": "Status card not found" }`。

---

## 卡片对象

| 字段 | 含义 |
|---|---|
| `id`、`companyId` | 身份标识。 |
| `createdByUserId`、`createdByAgentId` | 卡片的创建者；两者恰有一个会被设置。 |
| `title` | 卡片名称。除非固定名称，否则由编译器自动生成。 |
| `titlePinned` | 为 `true` 时，重新编译不会更改 `title`。 |
| `interestPrompt` | 驱动卡片的唯一说明：要关注什么，以及更新应如何呈现。 |
| `queries` | 编译后的公司搜索查询；首次编译完成前为空。 |
| `queryVersion` | 每次写入编译后的查询时递增。 |
| `queryCompiledAt`、`queryCompiledByAgentId` | 最近一次编译查询的时间及执行者。 |
| `agentId` | 此卡片专用的摘要器覆盖值；使用内置摘要器时为 `null`。 |
| `refreshPolicy` | 更新策略，见[刷新策略](#刷新策略)。 |
| `state` | `compiling`、`active`、`error`、`paused_budget` 或 `paused_hours`。 |
| `pendingChangeCount`、`lastChangeAt` | 自上次更新以来发现的变化数，以及这些变化开始累积的时间。 |
| `fingerprint`、`fingerprintAt` | 监控任务的快照；变更检测会以此为基准进行比较。 |
| `mentionedIssueIds` | 最新摘要中提及并加入监控集合的任务。 |
| `documentId` | 保存摘要文本的文档；首次写入前为 `null`。 |
| `lastUpdateRunKind` | 上次内容更新类型为 `full` 或 `incremental`；配置变更后为 `null`。 |
| `lastGeneratedAt`、`lastModel` | 上次写入摘要的时间，以及智能体报告所用的模型。 |
| `generatingIssueId` | 当前正在执行的生成任务；没有任务时为 `null`。 |
| `failureReason` | 当 `state` 为 `error` 时提供通俗的失败说明，否则为 `null`。 |
| `nextEvalAt` | 调度器下次评估此卡片的时间。处于 `manual` 模式或卡片已归档时为 `null`。 |
| `archivedAt`、`archivedByUserId`、`archivedByAgentId` | 归档记录信息。 |
| `createdAt`、`updatedAt` | 时间戳。 |

读取路由返回的卡片会**补齐**数据库中未存储的三个字段：`summaryBody`（当前 Markdown）、`todayTokens` 和 `todayCostCents`（此卡片从 UTC 午夜起的消耗），以及 `watchedIssueCount`（编译查询当前匹配的任务数；无法计算时省略）。

### 卡片状态

| 状态 | 含义 |
|---|---|
| `compiling` | 刚创建或提示内容刚更改。智能体正在编译查询并写入第一份摘要。 |
| `active` | 正常工作。面板是否将其显示为最新、过期或更新中，由 `pendingChangeCount` 和 `generatingIssueId` 推导。 |
| `error` | 上次生成任务在未写入摘要的情况下到达 `done`、`cancelled` 或 `blocked` 状态。`failureReason` 会说明原因。 |
| `paused_budget` | 已达到每日令牌上限。自动更新会暂停，但手动刷新仍会运行。 |
| `paused_hours` | 当前不在卡片的活跃时段内。时段开始后，累积的变化会合并为一次更新。 |

### 刷新策略

创建和更新时都可以传入 `refreshPolicy` 对象：

| 字段 | 必需条件 | 说明 |
|---|---|---|
| `mode` | 否 | `manual`（默认值）、`interval` 或 `reactive`。 |
| `intervalMinutes` | `interval` 模式下必填 | 正整数。在 `interval` 模式下省略会导致校验失败，错误为 *"Required for interval mode"*。 |
| `debounceSeconds` | `reactive` 模式下必填 | 正整数。在 `reactive` 模式下省略会导致校验失败，错误为 *"Required for reactive mode"*。 |
| `maxUpdatesPerHour` | 否 | 正整数，适用于 `reactive` 模式；未设置时默认为 6。 |
| `triggers` | 否 | 哪些变化会被计入；详见下表。 |
| `activeHours` | 否 | 格式为 `{ start, end, timezone }`；`start` 和 `end` 使用 24 小时制的 `HH:MM` 格式，`timezone` 必须是有效的 IANA 标识符。超出此时段时，卡片进入 `paused_hours`。 |
| `dailyTokenCap` | 否 | 正整数。未设置时，系统默认每日最多使用 100,000 个令牌（按 UTC 日计算），达到上限后暂停并进入 `paused_budget`。 |

`triggers` 决定哪些变化值得触发更新：

| 触发器 | 默认值 | 触发条件 |
|---|---|---|
| `statusTransitions` | `true` | 受监控任务的状态发生变化。 |
| `membershipChanges` | `true` | 任务开始或不再匹配查询。 |
| `humanComments` | `true` | 有人评论了受监控的任务。 |
| `assigneeChanges` | `true` | 受监控任务的负责人发生变化。 |
| `anyUpdate` | `false` | 任何变化，包括进行中任务的频繁变动。 |

活跃时段和令牌上限只限制**自动**工作。手动请求 `refresh` 或 `recompile` 时仍会执行。

---

## 列出状态卡

```
GET /api/companies/{companyId}/status-cards
```

返回公司的状态卡，按最近更新时间从新到旧排序；每张卡均按上文所述补齐字段。任何有权访问该公司的调用方都可以使用此路由。

| 查询参数 | 说明 |
|---|---|
| `archived` | `true` 返回已归档卡片；`false`（默认值）返回未归档卡片。 |

### 示例

```bash
curl -sS \
  -H "Authorization: Bearer {token}" \
  "https://paperclip.example.com/api/companies/{companyId}/status-cards?archived=false"
```

---

## 创建状态卡

```
POST /api/companies/{companyId}/status-cards
```

创建卡片并立即将编译任务加入队列。需要 `tasks:assign` 权限。

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `interestPrompt` | 是 | 卡片的持续性说明；去除首尾空白后长度为 1–20,000 个字符。智能体调用方上限为 4,000 个字符。 |
| `title` | 否 | 长度为 1–300 个字符。省略时由编译器为卡片命名。 |
| `titlePinned` | 否 | 默认为 `false`。`true` 可保护标题，使其不被重新编译修改。 |
| `agentId` | 否 | 此卡片专用的摘要器覆盖值；使用内置摘要器时传入 `null`。必须属于该公司，否则返回 `422 Unprocessable Entity` 和 `{ "error": "Summarizer agent must belong to this company" }`。 |
| `refreshPolicy` | 否 | 默认值为 `{ "mode": "manual" }`，并采用默认触发器。 |

**状态码：** `201 Created`。响应返回已加入编译队列的卡片：`state: "compiling"`，且已设置 `generatingIssueId`。

创建卡片和启动编译是一个全有或全无的操作：如果无法创建编译任务或无法唤醒智能体，新卡片会被删除，并返回底层错误。

智能体调用方必须属于正在创建卡片的公司，否则返回 `403 Forbidden` 和 `{ "error": "Agent cannot author status cards for this company" }`。

### 示例

```bash
curl -sS -X POST \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/companies/{companyId}/status-cards" \
  -d '{
    "interestPrompt": "Watch the launch project. Tell me what is blocked and the single next action.",
    "refreshPolicy": {
      "mode": "interval",
      "intervalMinutes": 30,
      "dailyTokenCap": 50000
    }
  }'
```

---

## 获取状态卡

```
GET /api/status-cards/{id}
```

返回一张补齐了附加字段的卡片。有权访问公司即可调用。

---

## 更新状态卡

```
PATCH /api/status-cards/{id}
```

更改卡片配置，或归档、恢复卡片。需要 `tasks:assign` 权限；智能体只能修改自己创建的卡片。

请求体必须至少包含一个字段，否则校验失败并返回 *"At least one field is required"*：

| 字段 | 说明 |
|---|---|
| `interestPrompt` | 长度为 1–20,000 个字符（智能体为 4,000 个字符）。**更改此字段会排入重新编译任务**，并将卡片重置为 `compiling`。 |
| `title` | 长度为 1–300 个字符；传入 `null` 可将命名权交还给编译器。 |
| `titlePinned` | 是否允许重新编译时更改卡片名称。 |
| `agentId` | 摘要器覆盖值，或 `null`；必须属于该公司。 |
| `refreshPolicy` | 完整的策略对象，会替换已存储的策略并重新安排卡片。 |
| `archived` | `true` 表示归档卡片，`false` 表示恢复卡片。 |

以下两种操作会产生需要留意的副作用：

- **编辑 `interestPrompt`** 会排入编译任务并清除 `lastUpdateRunKind`，因此下一次内容更新会完整重建，而不是增量修改。响应为编译任务返回的卡片。
- **归档**（`archived: true`）会清除 `nextEvalAt` 和 `generatingIssueId`，并取消正在进行的生成任务。**恢复**（`archived: false`）时，如果卡片已有编译查询且没有任务正在运行，就会以 `restore` 为触发原因排入一次完整刷新。

更改智能体也会清除 `lastUpdateRunKind`，原因相同：新的写入者不应增量修改并非由自己生成的文本。

---

## 删除状态卡

```
DELETE /api/status-cards/{id}
```

永久删除卡片及其更新历史。需要 `tasks:assign` 权限；智能体只能删除自己创建的卡片。成功时返回空响应体和 `204 No Content`。

---

## 更新历史

```
GET /api/status-cards/{id}/updates
```

返回卡片的更新记录，按从新到旧排序。有权访问公司即可调用。

每项记录包含：

| 字段 | 含义 |
|---|---|
| `kind` | `compile`、`full` 或 `incremental`。 |
| `trigger` | `manual`、`interval`、`reactive` 或 `restore`。 |
| `status` | `running`、`ok` 或 `failed`。 |
| `generationIssueId`、`runId` | 此次更新对应的任务和智能体运行。 |
| `changes` | 此次更新整合的任务变化；每项包含 `issueId`、`identifier`、`from`、`to` 和 `changeKind`。 |
| `inputTokens`、`outputTokens`、`costCents`、`model` | 根据运行的费用事件归集的此次更新用量和费用。 |
| `queryVersion` | 此次更新所使用的编译查询版本。 |
| `changeSummary` | 智能体为此次更新编写的单行说明。 |
| `startedAt`、`finishedAt` | 开始和结束时间。 |
| `error` | 当 `status` 为 `failed` 时设置。 |

`changeKind` 的取值为 `new`、`removed`、`status`、`assignee`、`human_comment` 或 `updated`。

---

## 摘要修订版本

```
GET /api/status-cards/{id}/summary-revisions
```

返回卡片以往的摘要，按从新到旧排序。每项包含 `id`、`revisionNumber`、`title`、`body`、`changeSummary` 和 `createdAt`。从未写入摘要的卡片会返回空数组。有权访问公司即可调用。

---

## 重新编译

```
POST /api/status-cards/{id}/recompile
```

根据卡片当前的 `interestPrompt` 重新生成编译查询，然后写入一份新的完整摘要。需要 `tasks:assign` 权限；智能体只能重新编译自己创建的卡片。

请求体为空。

**响应字段：**

| 字段 | 含义 |
|---|---|
| `card` | 卡片当前处于 `compiling` 状态，且已设置 `generatingIssueId`。 |
| `generatingIssue` | 已创建或已加入的编译任务。 |
| `alreadyGenerating` | 如果完全针对当前提示内容的编译任务确实已在运行，且本请求已加入该任务，则为 `true`。 |

**状态码：** 新排入队列的编译任务返回 `202 Accepted`。加入已在运行的编译任务返回 `200 OK`，因此重复点击不会排入两个运行。状态为 `blocked` 的任务不算正在运行：它正在等待人工处理；重新触发会重新打开该任务，并按一次新运行处理。

已归档的卡片不能编译：返回 `422 Unprocessable Entity` 和 `{ "error": "Archived status cards cannot be compiled" }`。

---

## 刷新

```
POST /api/status-cards/{id}/refresh
```

请求立即更新卡片。需要 `tasks:assign` 权限；智能体只能刷新自己创建的卡片。

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `full` | 否 | 默认为 `false`。`true` 会强制完整重建，不让 Paperclip 自行选择更新方式。 |

手动刷新始终会运行：它会绕过卡片的活跃时段和每日令牌上限，即使变更检测没有发现新内容也会执行。

**响应字段：**

| 字段 | 含义 |
|---|---|
| `card` | 请求处理后的卡片。 |
| `generatingIssue` | 更新任务；未排入任务时为 `null`。 |
| `alreadyGenerating` | 如果已有更新正在运行且本请求已加入，则为 `true`。 |
| `enqueued` | 本请求确实排入工作时为 `true`。 |
| `kind` | 已排入工作时存在，值为 `full` 或 `incremental`。 |
| `changes` | 已排入工作时存在，表示此次更新将整合的变化。 |

**状态码：** 当 `enqueued` 为 `true` 且 `alreadyGenerating` 为 `false` 时返回 `202 Accepted`，即本请求启动了运行。其他情况均返回 `200 OK`：包括加入正在运行的任务，或策略评估决定等待、暂停或不执行操作。

需要预先处理的两种拒绝情况：

- 尚未编译查询的卡片：`409 Conflict`，`{ "error": "Compile the status-card query before refreshing it" }`。
- 已归档的卡片：`422 Unprocessable Entity`，`{ "error": "Archived status cards cannot be refreshed" }`。

### 示例

```bash
curl -sS -X POST \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/status-cards/{id}/refresh" \
  -d '{ "full": true }'
```

---

## 试运行

```
GET /api/status-cards/{id}/dry-run
```

立即执行卡片的编译查询并返回匹配结果，不写入任何内容，也不消耗模型令牌。面板中的**受监控任务**标签页会读取此接口。

与其他读取路由不同，此路由要求明确通过 `company_scope:read` 检查。受限在较窄边界内的调用方会收到 `403 Forbidden` 和 `{ "error": "Status-card dry-run is outside this actor's low-trust authorization boundary" }`。

**响应字段：**

| 字段 | 含义 |
|---|---|
| `cardId` | 状态卡。 |
| `queryVersion` | 本次执行的编译查询版本。 |
| `queries` | 每条编译查询对应一项，包含查询本身（`query`）及其 `result`。这些查询的结果会取并集，因此同一任务可能出现在多项结果中。 |
| `mentionedIssues` | 最新摘要提及的任务摘要；除查询匹配项外，这些任务也会受到监控。 |

---

## 写入编译后的查询

```
PUT /api/status-cards/{id}/query
```

摘要器智能体在编译过程中调用的两个端点中的第一个。本文档介绍此接口，以便你了解卡片会发生什么；通常不需要自行调用。

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `queries` | 是 | 1–10 条公司搜索查询；执行结果会取并集。 |
| `title` | 是 | 长度为 1–300 个字符。如果卡片的 `titlePinned` 为 `true`，则忽略此字段。 |
| `changeSummary` | 是 | 长度为 1–2,000 个字符，不能为空。 |
| `generationIssueId` | 是 | 此次写入所属的编译任务。 |

写入成功后，服务器会保存查询、递增 `queryVersion`、记录 `queryCompiledAt` 和 `queryCompiledByAgentId`，使卡片保持在 `compiling` 状态，并在更新记录中写入一项 `compile` 记录，以及一项预期会在同一次运行中接续发生的完整摘要 `running` 记录。

---

## 写入摘要

```
PUT /api/status-cards/{id}/summary
```

摘要器智能体调用此端点写入完成的摘要，无论此次摘要来自编译任务还是更新任务。

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `markdown` | 是 | 摘要正文，长度为 1–200,000 个字符。以 Markdown 文档形式保存。 |
| `title` | 否 | 长度为 1–300 个字符。 |
| `changeSummary` | 是 | 长度为 1–2,000 个字符，不能为空。会附加到新修订版本。 |
| `generationIssueId` | 是 | 此次写入所属的生成任务。 |
| `model` | 否 | 智能体实际使用的模型，最多 200 个字符，也可以为 `null`。保存到卡片的 `lastModel` 字段。 |

### 两种写入都必须满足的规则

以下任一条件不满足都会返回 `403 Forbidden`：

- 调用方必须是智能体，并且必须是卡片的 `agentId` 覆盖智能体，或在该公司带有 `summarizer` 内置标记的智能体；否则返回 `{ "error": "Only the card's summarizer agent may write status cards" }`。两者都保持有效，因此切换智能体前创建的任务仍可写入其结果。
- 写入必须指定卡片当前正在等待的任务；否则返回 `{ "error": "Status-card write does not match the active generation task" }`。
- 该任务必须分配给当前调用的智能体；否则返回 `{ "error": "Generation task is not assigned to this agent" }`。
- 该任务必须仍处于活动状态；否则返回 `{ "error": "Generation task is no longer active" }`。
- 任务负载必须指向此卡片和公司；否则返回 `{ "error": "Generation task does not target this status card" }`。
- 请求必须来自该任务对应的运行；否则返回 `{ "error": "Status-card write must run from the linked generation task" }`。

发生冲突时返回 `409 Conflict`：查询写入返回 `{ "error": "Status-card compilation was superseded by a newer task" }`；摘要写入返回 `{ "error": "Status-card generation was superseded by a newer task" }`。

已归档的卡片会拒绝两类写入：查询写入返回 `422 Unprocessable Entity` 和 `{ "error": "Archived status cards cannot accept generation writes" }`；摘要写入返回 `422 Unprocessable Entity` 和 `{ "error": "Archived status cards cannot accept summaries" }`。写入摘要前还必须先编译查询，否则返回 `409 Conflict` 和 `{ "error": "Compile the status-card query before writing its summary" }`。

### 写入成功后的处理

摘要会作为新的文档修订版本追加，而非替换旧文本。随后卡片结束生成流程：`state` 恢复为 `active`，清除 `generatingIssueId` 和 `failureReason`，写入 `lastGeneratedAt`、`lastModel` 和 `lastUpdateRunKind`，重置待处理变更计数，并按照刷新策略重新安排 `nextEvalAt`。

受监控集合也会同时发生两项处理。服务器会解析 Markdown 中提及的任务（例如标识符 `ABC-123` 或 `/issues/{uuid}` 链接），在该卡片所属公司内查找，并将最多 200 个任务存入 `mentionedIssueIds`；这些任务后续发生变化时也会触发更新。未知标识符和其他公司的任务会被忽略。指纹则保留生成时的基准快照，因此摘要编写期间发生变化的任务会在下次差异比较中显示，不会被悄悄合并掉。

响应包含更新后的 `card`、`document` 和新建的 `revision`。更新记录中的对应项会以 `status: "ok"` 结束，并记录此次运行的令牌用量、费用总计和模型。

**响应字段：**

| 字段 | 含义 |
|---|---|
| `card` | 已完成生成流程的卡片。 |
| `document` | 摘要文档，包含 `latestBody` 和 `latestRevisionNumber`。 |
| `revision` | 刚追加的修订版本。 |

---

## 活动日志

每个修改类路由都会在公司的活动日志中记录一项 `entityType: "status_card"` 实体记录：

| 操作 | 记录时机 |
|---|---|
| `status_card.created` | 创建卡片时。详情包含初始 `state`。 |
| `status_card.updated` | 修改卡片时。详情包含已更改的 `fields` 以及卡片当前的 `archived` 状态。 |
| `status_card.deleted` | 删除卡片时。 |
| `status_card.recompile_requested` | 请求重新编译时。详情包含 `generatingIssueId` 和 `alreadyGenerating`。 |
| `status_card.refresh_requested` | 请求刷新时。详情包含 `full`、`generatingIssueId`、`alreadyGenerating` 和 `enqueued`。 |
| `status_card.query_written` | 智能体写入编译查询时。详情包含 `queryVersion`、`generationIssueId` 和 `changeSummary`。 |
| `status_card.summary_written` | 智能体写入摘要时。详情包含 `queryVersion`、`generationIssueId`、`documentId` 和 `changeSummary`。 |

如何读取日志，请参阅[活动](./activity.md)页面。

---

## 后续阅读

- [状态卡](../../experimental/status-cards.md)——状态卡面板、更新策略说明，以及如何启用此功能。
- [状态卡查询技能](../skills/bundled/paperclip-operations/status-card-query.md)——教智能体创建状态卡并将提示编译为查询的内置技能。
- [内置智能体](./built-in-agents.md)——如何配置摘要器，以及其生命周期状态的含义。
- [摘要栏位](./summary-slots.md)——另一种由摘要器写入的内容区域，固定显示在项目或工作区标题处。
- [任务](./issues.md)——编译和更新任务都是普通任务。
- [API 概览](./overview.md)——基础 URL、身份验证、公司作用范围，以及通用错误代码表。
