---
paperclip_version: v2026.1012.0
seo_title: HTTP Webhook 适配器
seo_description: 向你控制的服务发送 JSON webhook。如果运行时位于远程、长期运行，或已作为 API 公开而非本地 CLI，请使用此适配器。
---

# HTTP

`http` 适配器会向你控制的服务发送 JSON webhook 请求。如果运行时位于远程、长期运行，或已作为 API 公开而非本地命令，请使用此适配器。

> **信息：** `http` 是 Paperclip 运行时使用的内置内部适配器。目前它在智能体配置中的适配器类型下拉菜单里显示为 **“即将推出”**，无法手动选择。若要立即使用，请通过 API 或导入公司导出包的方式配置智能体。

---

## 适用场景

- 智能体运行在云端或其他服务之后。
- 希望 Paperclip 触发 webhook，并由远程服务处理其余工作。
- 运行时已经接受 JSON 正文，并返回简单的成功或失败响应。

## 不适用场景

- 运行时只是本地脚本或命令。请使用 [Process](./process.md)。
- 需要会话持久化或内置 CLI 行为。请改用一种本地适配器。
- 需要 Paperclip 解析远程运行时的丰富 stdout 执行记录。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `url` | 是 | 绝对 `http://` 或 `https://` 端点。 |
| `method` | 否 | 使用的 HTTP 方法。默认为 `POST`。 |
| `headers` | 否 | 额外请求头。 |
| `payloadTemplate` | 否 | 在添加标准 Paperclip 字段之前，合并到请求正文中的 JSON 对象。 |
| `timeoutMs` | 否 | 请求超时毫秒数。`0` 表示不设超时。 |

> **注意：** HTTP 适配器不会启动子进程。它只发送一次请求、等待响应，并将所有非 2xx 响应视为失败。

---

## 请求正文

Paperclip 发送的 JSON 载荷始终包含：

- `runId`
- `agentId`
- `context`
- `connectionInstructions`（对象或 `null`——请参阅[连接说明](#连接说明)）

Paperclip 会先合并 `payloadTemplate` 中的字段，再添加以上标准字段。如果键名冲突，以标准 Paperclip 字段为准。

请求正文示例：

```json
{
  "runId": "run-123",
  "agentId": "agent-123",
  "context": {
    "taskId": "issue-123",
    "wakeReason": "scheduled",
    "commentId": null
  },
  "customField": "value"
}
```

执行完成后，你的服务可以使用 `PAPERCLIP_API_URL` 和 Paperclip API 密钥回调控制平面。

---

<a id="connection-instructions"></a>
## 连接说明

连接可以携带供智能体使用的说明，例如如何使用 Honcho 或 Mem0 等记忆服务提供商。当智能体获得授权使用的任一连接启用了说明时，Paperclip 会将说明发送给你的服务，使模型能够实际读取它们。

说明会作为顶层 `connectionInstructions` 字段出现在请求正文中：

- 适用时为 `{ "text": "...", "digest": "..." }` 对象；
- 不适用时为 `null`。

```json
{
  "runId": "run-123",
  "agentId": "agent-123",
  "context": { "taskId": "issue-123" },
  "connectionInstructions": {
    "text": "## Connection instructions\n\n...",
    "digest": "3f2a..."
  }
}
```

`text` 是可直接使用的 Markdown。请将它附加到发送给模型的提示词中；不要传递整个对象。`digest` 是 `text` 的 SHA-256 十六进制哈希，便于缓存内容或检测不同轮次间说明是否发生变化。

与其他标准字段一样，`connectionInstructions` 会覆盖 `payloadTemplate` 中同名的键。用户如何编写和编辑说明，请参阅[已保存的连接说明](../../connectors/connection-instructions.md)。

---

## 私有与公开端点

HTTP 适配器在建立连接前会在套接字边界检查每个请求，避免你意外将 webhook 指向本地网络中的资源。默认允许公开源（origin）。解析到私有、保留或回环 IP 地址的源（origin）（包括 `localhost`、`10.0.0.0/8`、`192.168.0.0/16` 等）会被拒绝。链路本地元数据端点（例如云服务商的 `169.254.x.x`）始终会被阻止，不能通过设置重新允许。

检查还会固定 DNS 解析结果：防护逻辑批准哪个地址，套接字就会连接到哪个地址。这样可防止主机名在检查时解析为公开地址，却在稍后解析为私有地址。

如果运行时确实位于私有源（origin）（例如内部服务或你自己子网中的 webhook），可在 Paperclip 主机上使用环境变量 `PAPERCLIP_HTTP_ADAPTER_PRIVATE_ENDPOINT_ALLOWLIST` 显式允许。此变量是以逗号分隔的**精确源（origin）**列表；每个源由协议、主机和端口共同组成（例如 `http://hooks.internal.example:8080`）。匹配时不区分大小写，且每个条目必须满足以下条件：

- 使用 `http` 或 `https`；
- 除 `/` 外不含其他路径；
- 不含用户信息、查询字符串或片段。

无法解析为规范源（origin）的条目会被忽略，而不会宽松匹配。因此，拼写错误只会让条目静默退出允许列表，不会意外开放超出预期的范围。只有与允许列表中的某一条目完全匹配，该源（origin）才可以访问私有网络——不同的端口或协议都属于不同的源。完整条目请参阅[环境变量参考](../deploy/environment-variables.md)。

---

## 环境测试

**Test Environment**（测试环境）按钮会检查：

- URL 已填写，且协议为 `http` 或 `https`。
- 配置的请求方法有效。
- 如果端点可访问，它能响应快速的 `HEAD` 探测。

如果探测在私有网络中失败，只要运行时确实可以收到正式请求，仍可能可以接受。但请注意，私有源地址也必须列入[私有与公开端点](#私有与公开端点)中说明的允许列表；否则，无论探测结果如何，正式请求都会在连接前被拒绝。

---

## 示例

```json
{
  "adapterType": "http",
  "adapterConfig": {
    "url": "https://agent.example.com/paperclip/heartbeat",
    "method": "POST",
    "headers": {
      "Authorization": "Bearer <shared-secret>"
    },
    "payloadTemplate": {
      "source": "paperclip"
    },
    "timeoutMs": 10000
  }
}
```

---

## 实用说明

- 在远程服务上使用共享密钥或基于请求头的身份验证。
- 尽可能让端点具备幂等性，以便支持重试。
- webhook 被接受时就返回 2xx 响应，不要等到所有后续工作都完成后再返回。

---

## 后续阅读

- [创建适配器](./creating-an-adapter.md)
- [外部适配器](./external-adapters.md)
