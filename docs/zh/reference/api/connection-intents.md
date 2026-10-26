---
paperclip_version: v2026.1023.0
seo_title: 连接意图 API
seo_description: 智能体无法访问某项服务时，可以创建连接意图，让责任人连接应用并解除任务阻塞。
---

# 连接意图

**连接意图**是智能体请求人员连接某项当前无法访问的服务的方式。例如，智能体正在处理任务，需要使用 GitHub，但没有为它配置可用的 GitHub 连接。此时，智能体不会直接失败，而是创建连接意图：Paperclip 会在任务讨论串中向此运行的责任人发送连接卡片；智能体结束当前运行并等待。人员连接应用（或拒绝请求）后，意图会解决，系统会唤醒智能体的心跳，让它继续处理任务。

因此，连接意图有一个简短的生命周期：智能体**请求**某项服务后，会在任务上创建一个处于待处理状态的 `connection_intent` 交互。指定的人员可以在操作期间将其推进到不同阶段（`requested`、`authorizing`、`needs_retry`），之后通过选择连接来**完成**意图，或者附带可选原因**拒绝**意图。完成时会将所选连接接入请求智能体，委托该人员的身份授权并将连接安装到智能体上，然后将意图标记为 `connected`。

> **这些端点供内部使用。** 它们服务于两类特定调用方：智能体运行时（通过运行时工具 token）和回答连接卡片的看板 UI。这些端点并非面向一般 API 使用者的通用接口；正常情况下，你不会像调用 [Tool Gateway](./tool-gateway.md) 那样编写脚本直接调用它们。本页为了完整性而记录这些接口，且运行时部分通过 MCP 暴露。如果你刚开始使用 API，请先阅读 [API 概览](./overview.md)，了解基础 URL、身份验证和错误约定。

此接口分为两部分：由智能体调用的**运行时工具**，以及由指定人员调用的**看板路由**。

---

## 运行时工具（面向智能体）

这些路由挂载在服务器根路径下，**不在** `/api` 路径下。它们使用运行时工具 bearer token 进行身份验证，该短期 token 绑定到一次心跳运行。每次调用都会通过其运行记录重新验证 token，因此运行结束后，该运行的 token 就会失效。缺少 token、token 无效或已过期时，会返回 `401 Unauthorized` 和 `{ "error": "Runtime tools token is missing, invalid, or expired" }`。

在底层，token 会解析到运行所属的公司、智能体和责任人，以及该运行绑定的任务。未绑定任务的运行不能请求连接（返回 `422` 和 `Connection requests require a task-bound heartbeat run`）；已处于 `done` 或 `cancelled` 状态的任务也会被拒绝。

### MCP 端点

这些工具以精简的模型上下文协议（MCP）服务器形式提供：

```
GET  /mcp/runtime-tools
POST /mcp/runtime-tools
```

此端点只通过 `POST` 使用 JSON-RPC 2.0 协议。`GET` 请求会返回 `405 Method Not Allowed` 和 `Allow: POST` 请求头，因此尝试打开服务器发送事件流的 MCP 客户端会立即发现此端点不提供事件流。

| 方法 | 行为 |
|---|---|
| `initialize` | 返回 `protocolVersion` `2025-03-26`、`capabilities.tools`（其中 `listChanged: false`）以及 `serverInfo`，名称为 `paperclip-runtime-tools`，版本为 `1`。 |
| `notifications/initialized` | 返回 `202 Accepted`，没有正文。 |
| `tools/list` | 返回下表中的两个工具，每个工具包含 `name`、`description` 和 `inputSchema`。 |
| `tools/call` | 执行工具；传入 `params.name` 和 `params.arguments`。 |

未知的 `params.name` 会返回 `404` 和 JSON-RPC 错误 `-32601`、`Unknown tool: <name>`。其他未知方法会返回 `404` 和 `-32601`、`Unknown method: <method>`。

提供以下两个工具：

| 工具 | 参数 | 用途 |
|---|---|---|
| `connections_search` | `query`（字符串，可选） | 按服务名称或描述需求的自然语言查找可连接服务，并显示每项服务对当前智能体的就绪状态。 |
| `connection_request` | `service`（字符串，必填） | 按 slug 请求特定服务；如果当前已经可用，则返回就绪状态，否则创建连接意图。 |

### REST 便捷路由

不想使用 MCP 的调用方也可以通过普通 JSON 调用相同的两个操作：

```
POST /runtime-tools/connections/search
POST /runtime-tools/connections/request
```

`search` 接受 `{ "query": <string> }` 正文。字符串会修剪首尾空白，最多 4,000 个字符，默认值为 `""`（表示列出所有服务）。查询可以是服务名称，也可以是一整句话来描述智能体的需求。服务名称匹配优先于功能匹配；附加词语不会阻止匹配；少量拼写错误和分词名称（如“Agent Mail”）仍可找到服务。结果最多 40 项。响应示例如下：

```json
{
  "version": 1,
  "query": "github",
  "results": [
    {
      "service": "github",
      "name": "GitHub",
      "description": "...",
      "logoUrl": "...",
      "methods": [ { "key": "...", "label": "...", "auth": "...", "purpose": "tool" } ],
      "state": "ready",
      "connectionId": "..."
    }
  ]
}
```

每个方法可以包含 `purpose`，其值为 `tool`、`channel` 或 `ai`。`channel` 和 `ai` 方法还会包含公司范围内的 `setupPath`，智能体可将其分享给用户。`tool` 方法和 AgentMail 应使用 `connection_request`。只有在实例开启 **Chat connectors** 设置时，聊天渠道方法才会出现；AgentMail 始终会出现。搜索到匹配结果并不代表账户已获授权，也不代表工具已经安装。

每个结果中的 `state` 是以下值之一：`ready`（已有连接且当前智能体可以使用，同时设置了 `connectionId`）、`needs_user_action`（存在匹配连接，但目前不可用）、`available`（可以连接此服务，但尚无连接）或 `unavailable`（此服务不提供连接方法）。

`request` 接受 `{ "service": <string> }` 正文；字符串会修剪首尾空白，长度为 1–120 个字符。如果服务不可用，会返回 `422` 和 `Connection service <slug> is not available`。否则响应如下：

```json
{
  "version": 1,
  "service": "github",
  "state": "needs_user_action",
  "connectionId": null,
  "interactionId": "...",
  "instruction": "A connection card was sent to the responsible user. End this run and wait for continuation."
}
```

对于 `agentmail`，卡片是直接发送给责任人的 API 密钥卡片。该流程会将密钥分享给公司内所有人员，但只为请求的智能体安装访问权限；同时会创建或连接一个收件箱，并且只有服务器确认收件箱可用后才会完成。仅保存密钥并不代表请求已就绪。如果设置流程中途停止，重新加载后会在同一请求下恢复，不会重复创建账户或收件箱。完成卡片要求具备连接管理权限，否则设置会失败并返回 `AgentMail inbox setup requires connection-management authority`。

如果服务已经连接，`state` 为 `ready`，`connectionId` 已设置，`interactionId` 为 `null`，并且指令会告知该服务在当前运行中可用。每个运行和服务组合的请求都是幂等的，因此重复调用不会产生多张卡片。

---

## 看板路由（面向人员）

这些路由挂载在 `/api` 下，用于响应连接卡片。每个路由都限定看板用户：调用方必须是看板用户、有权访问意图所属公司，并且必须是卡片**指定的用户**。其他任何人都会收到 `403 Forbidden` 和 `{ "error": "Only the addressed user can act on this connection request" }`。路径中的 `{interactionId}` 是任务上的连接意图交互 ID。

### 读取设置选项

```
GET /api/connection-intents/{interactionId}/setup-options
```

返回人员回答卡片所需的信息：交互本身、请求的 `service`（名称、说明、logo 和连接 `methods`）、该人员有资格使用的此服务现有连接 `existingConnections`，以及 `requestedAgentId`。对于已部分完成的 AgentMail 卡片，响应还会包含 `emailSetup`，其中有已保存的 `credentialConnectionId` 和 `readyConnectionId`，因此卡片可以继续操作。如果服务已不再提供，此端点会返回 `404` 和 `Connection service is no longer available`。

### 更新阶段

```
POST /api/connection-intents/{interactionId}/phase
```

将意图推进到处理中阶段。请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `phase` | 是 | `requested`、`authorizing` 或 `needs_retry` 之一。 |

其他值会返回 `422` 和 `{ "error": "phase must be requested, authorizing, or needs_retry" }`。

### 完成意图

```
POST /api/connection-intents/{interactionId}/complete
```

将选定连接接入请求智能体，并将意图标记为 `connected`。请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `connectionId` | 是 | 要使用的连接 UUID，且必须属于此意图请求的服务。 |

所选连接必须与请求的服务匹配，否则会返回 `404` 和 `Connection does not match this intent`；连接还必须处于 `active` 且 `enabled` 状态，否则返回 `409` 和 `Finish and test this connection before using it for the task`。已解决的意图会返回 `409` 和 `Connection intent is already resolved`。完成操作会将该人员的身份授权委托给智能体，并将连接安装到该智能体上；分享公司级连接则要求具备连接管理权限（`tools:manage_connections`）。成功后会记录 `issue.connection_intent_connected` 活动，并唤醒已分配智能体的心跳。

### 拒绝意图

```
POST /api/connection-intents/{interactionId}/decline
```

将意图解决为 `declined`。请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `reason` | 否 | 修剪首尾空白后的文本，最多 4,000 个字符。 |

此操作会记录 `issue.connection_intent_declined` 活动，并与 `complete` 一样唤醒已分配的智能体，让它作出响应。

### 意图自动过期的情况

这种情况无需调用路由。如果人员通过其他方式连接了服务（例如从 **Connectors** 操作），并且请求智能体在当前任务中已经拥有可用访问权限，Paperclip 会自动将待处理意图设为 `expired`。交互的 `result` 会保存原因 *"An already authorized connection is available to the requesting agent."* 系统会在工具刷新后向智能体发送新的后续运行，也不会代替该人员记录人工批准。

在将请求标记为已满足之前，Paperclip 会重新检查该人员的公司成员关系、连接健康状态、授权和智能体安装状态，以及任务的工具策略。AI 账户、聊天渠道、额外权限和上游服务提供商的请求会继续使用各自流程，不会按此方式过期。

---

## 后续阅读

- [Tool Gateway API](./tool-gateway.md)：了解已完成连接所接入的应用、连接、目录和策略引擎。
- [Issues API](./issues.md)：连接意图作为交互保存在任务讨论串中。
- [API 概览](./overview.md)：查看基础 URL、身份验证、公司范围和通用错误代码表。
