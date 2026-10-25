---
paperclip_version: v2026.1023.0
seo_title: 验证并修复连接器
seo_description: 使用只读测试调用确认新连接器是否正常工作，阅读连接器列表中的状态词，并修复常见的故障模式。
---

# 验证连接器并修复故障连接器

本页包含两项任务：在不触及 Paperclip 外部任何内容的情况下证明新连接有效，以及诊断已停止工作的连接。

## 首先：如何验证此类型？

下方的读取测试流程适用于**应用工具**连接。其他类型没有操作列表，因此该流程不适用于它们。

| 类型 | 验证方法 |
| --- | --- |
| **应用工具** — Notion、Jira、Stripe 以及目录中的大多数项目 | 使用下方的读取测试 |
| **消息通道** — Slack 聊天、Discord、Telegram、Teams、iMessage Photon、GitHub 代码审查机器人、AgentMail | 从提供商处发送一条真实消息，并等待智能体的回复。没有可供测试的操作列表，在收到回复之前，设置尚未完成。连接器自身的页面包含该序列 |
| **模型提供商** — Anthropic、OpenAI、OpenRouter、Grok、自定义端点、Bedrock | 选择兼容的运行时、连接和模型，然后在智能体的执行环境中运行一个简短任务。已连接徽章不能证明模型访问权限。参见 [自定义模型提供商](custom-model-providers.md) |
| **助手连接 (MCP)** | 从外部助手调用 `paperclip_connection` 并验证同意的人员和组织。在创建工作之前列出智能体或任务。参见 [助手连接](../../experimental/assistant-connections.md) |

[连接器访问权限的工作原理](access-model.md) 解释了为何控制方式不同。

## 验证新的工具连接

始终从读取操作开始。

1. 打开连接器并选择 **Permissions**（权限）选项卡。
2. 确认操作列表已加载。对于应用工具连接，空列表可能意味着发现过程未完成，或者提供商未针对此凭据返回任何工具。选择 **Refresh actions**（刷新操作）并检查任何错误；模型凭据和聊天通道没有操作目录。
3. 在测试期间，确认每个写入和破坏性操作均设置为 **Off**（关闭）或 **Ask first**（先询问）。
4. 使用连接器的内置测试调用，以特定智能体身份运行一个读取操作。Paperclip 会提示 **Choose which agent to test as**（选择以哪个智能体身份进行测试），因为结果取决于智能体的有效策略，而不仅仅取决于连接本身。
5. 阅读结果。Paperclip 会在可能时将返回内容布局为可读字段、表格或卡片，以便您无需阅读 JSON 即可进行检查。一行摘要会告知结果——例如 **Worked. 3 rows came back.**（成功。返回了 3 行数据。）或当调用成功但未返回任何数据时的 **Worked. No data to show.**（成功。无数据可显示。）。**Show raw response**（显示原始响应）提供提供商的实际负载；当结果无法安全布局时，它会自行打开。

API 等效操作：

```http
GET  /api/tool-connections/{connectionId}/test-agents
GET  /api/tool-connections/{connectionId}/test-agents/{agentId}/access
POST /api/tool-connections/{connectionId}/test-calls
GET  /api/tool-connections/{connectionId}/test-calls/{actionRequestId}
POST /api/tool-connections/{connectionId}/health-check
```

不要使用写入操作来验证连接器。针对真实账户半成功的写入操作，比干净失败的读取操作更糟糕，作为诊断手段而言。

## 阅读状态词

| 状态 | 含义 |
| --- | --- |
| **Healthy**（健康） | 凭据有效。这**不**意味着特定智能体可以使用该连接——这取决于智能体访问权限和操作自身的设置，这些在调用时单独检查。 |
| **Connected**（已连接） | 此连接器存在一个连接。 |
| **Not connected**（未连接） | 尚未建立连接。*"连接它以便智能体可以使用。"* |
| **Setup incomplete**（设置未完成） | 存在连接记录，但设置从未完成。*"在智能体可以使用此账户之前完成设置。"* |
| **Needs attention**（需要关注） | 某事需要人工处理：过期的凭据，或待处理的审查请求。 |
| **Paused**（已暂停） | *"已暂停——智能体目前无法使用。"* |

**Connected**（已连接）或 **Healthy**（健康）均不能证明特定智能体的调用会成功。**Connected** 表示存在记录；**Healthy** 表示凭据有效。*此*智能体是否允许运行*该*操作是另一层逻辑，在调用时解决。请以您实际打算使用的智能体身份运行读取测试。

## 常见故障

### 登录已过期或被撤销

*"您的授权已过期或被撤销。重新连接以继续。"* 或 *"需要重新连接——再次登录以恢复访问权限。"*

选择 **Reconnect**（重新连接）并再次登录。连接、其智能体访问权限及其操作设置均保留；仅替换凭据。参见 [重新授权、撤销或断开连接](reauthorize-and-disconnect.md)。

### 密钥停止工作

*"密钥停止工作——重新连接以修复。"* 对于 API 密钥连接，选择 **Replace the stored credential**（替换存储的凭据）并 **Paste your new key**（粘贴您的新密钥）。Paperclip 在保存前会验证它：*"That key didn't check out. Try another."*（该密钥未通过验证。请尝试另一个。）意味着该值被提供商拒绝，而非输入到错误字段中。

### 凭据属于他人

*"此连接的凭据不属于所选身份。其所有者必须重新连接。"*

个人连接必须运行在该人员拥有的凭据上。使用早期版本创建的一些个人连接将其密钥或秘密 URL 存储为公司凭据，这些连接现在会停止并显示此消息，而不是静默使用错误的凭据。拥有该连接的人员应选择 **Reconnect**（重新连接）并输入新的密钥或秘密 URL。连接保留其名称、智能体访问权限和操作设置；仅凭据转移至所有者。其他人无法代为执行此操作。

### 提供商提示缺少权限

*"提供商未授予此操作所需的权限。重新连接此连接并允许所需的读取和写入访问权限。"*

登录或密钥未覆盖操作所需的内容。重新连接并批准请求的访问权限，或者对于 API 密钥，创建一个具有读取和写入权限的密钥并粘贴进去。Paperclip 无法扩大已存在的密钥或令牌。

检查授权记录的作用域可能有所帮助，但需谨慎阅读：某些提供商从不说明授予了哪些作用域，此时列表仅显示 Paperclip 请求的内容。[登录授权上记录的作用域](access-model.md#scopes-recorded-on-a-sign-in-grant) 解释了如何区分两者。

### 授权从未完成

*"授权未完成。在登录窗口中完成设置或重试。"* 通常意味着弹出窗口过早关闭，或提供商正在等待管理员。对于需要租户批准的提供商，提供商页面上的警告会说明这一点。

### 操作被拒绝

- *"This action is off and won't run."*（此操作已关闭，不会运行。）——操作设置为 **Off**（关闭）。
- *"This action is new and hasn't been turned on yet."*（此操作是新的，尚未开启。）——在刷新时发现，尚未启用。
- *"Denied — see Review for why"*（已拒绝——查看审查以了解原因）——**Ask first**（先询问）请求被拒绝。
- *"The connected account may not have permission for this action."*（连接的账户可能没有此操作的权限。）——Paperclip 允许了它；提供商不允许。请在提供商处修复，而非在 Paperclip 中。

### 提供商返回错误

*"The app returned an error result."*（应用返回了错误结果。）并带有代码，如 `UNAUTHORIZED`、`FORBIDDEN`、`NOT_FOUND`、`INVALID_ARGUMENT`、`RATE_LIMIT` 或 `RESOURCE_EXHAUSTED`。这些来自服务。`RATE_LIMIT` 和 `RESOURCE_EXHAUSTED` 表示提供商正在限流；其余指向账户自身的权限或参数。

### 工具发现繁忙

*"Tool discovery is busy. Retry shortly."*（工具发现繁忙。请稍后重试。）并带有代码 `tool_discovery_busy` 意味着服务器正在为允许的最大并发调用者列表列出工具。连接或其凭据没有问题。等待片刻后再次列出工具。参见 [工具网关参考](../../reference/api/tool-gateway.md) 了解 API 结构。

### 任务需要 AI 凭据

打开任务的身份验证卡片，选择 **Fix connection**（修复连接）或显示的提供商连接操作。连接您的账户或重新连接现有账户；成功设置将恢复任务。如果另一人拥有该凭据，该人员必须修复它。选择共享账户不会绕过责任用户的访问检查。

对于配额错误，在支持的已保存 AI 账户上使用 **Check usage**（检查使用情况）。缺失的使用字段意味着 **Not reported**（未报告），而非零。参见 [检查 AI 账户使用情况](ai-usage.md)。

对于自定义端点，对照运行时、模型 ID 以及服务是否可从执行环境访问来检查 API 格式。回环 URL 指向智能体的主机或沙箱。参见 [自定义模型提供商](custom-model-providers.md)。

### 缺少托管登录选项

**Connect with Paperclip**（使用 Paperclip 连接）仅当您的实例已注册 Paperclip Cloud 且 Cloud 宣传该连接器配置文件时才会出现。如果缺失，连接器列表提供 **Enable Paperclip-managed sign-in**（启用 Paperclip 托管登录）或 **Continue enrollment**（继续注册），在此期间客户拥有的 OAuth 路径仍可用。

### 列表中没有任何项看起来像您想要的提供商

提供商可能被识别但被保留在目录之外，或者它可能根本没有自助服务路径。这两种情况都列在 [连接器](../../connectors.md) 的底部。

## 接下来查看何处

- 连接器的活动视图列出针对它进行的调用：`GET /api/tool-connections/{connectionId}/activity`。
- 公司 [活动日志](../guides/day-to-day/activity-log.md) 记录连接生命周期事件，包括 `tool_app.connected`、`tool_connection.catalog_refresh` 以及信任规则的创建和撤销。

## 相关

- [重新授权、撤销或断开连接](reauthorize-and-disconnect.md)
- [设置操作权限](action-permissions.md)
- [回答连接器审查请求](review-requests.md)
- [调试卡住的心跳](../how-to/debug-stuck-heartbeat.md)
