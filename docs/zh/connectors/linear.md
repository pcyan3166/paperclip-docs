---
paperclip_version: v2026.1023.0
seo_title: Linear 连接器
seo_description: 允许智能体读取、创建和更新 Linear 问题。通过浏览器登录，无需注册 OAuth 应用，了解工作区范围的含义、读取测试及修复方法。
---

# Linear

智能体可以读取 Linear 问题、创建新问题以及更新现有问题——这有助于在智能体于 Paperclip 中执行工作时，将工程工作保留在 Linear 中。

## 连接前准备

- 一个能够访问您希望智能体使用的工作区和团队的 Linear 账户。

对于浏览器登录，仅此而已。当您连接时，Paperclip 会向 Linear 的 MCP 服务器注册其自身的 OAuth 客户端，因此无需在 Linear 的开发者设置中创建应用。

## 连接 Linear

1. 打开 **Connectors** 并选择 **Linear**。
2. 阅读主按钮上方的访问权限说明行。它说明了连接将以谁的身份登录以及哪些智能体可以使用它。若要选择其他身份或缩小智能体范围，请选择 **Change**。
3. 选择 **Continue to sign in**，以连接应具备访问权限的账户身份登录，并批准访问。

Paperclip 向 Linear 请求 `read` 和 `write` 范围，以便智能体可以读取问题，也可以创建和更新问题。智能体实际可以执行哪些操作，仍由您在 **Permissions** 选项卡中决定。

登录需要您的 Paperclip 地址为 HTTPS，或为本地 `localhost` 地址。

### 改用 API 密钥

如果浏览器登录不适用，请选择 **Change** 并选择 **Use an API key**。

1. 在 Linear 的设置中创建个人 API 密钥。选择您计划允许的操作实际所需的权限和团队。
2. 将其粘贴到 **Your Linear key** 字段中，然后选择 **Connect**。

仅限 **Read** 权限的密钥即可使用，这是获取一个确实无法在 Linear 中更改任何内容的连接的最简单方式。工作区和角色访问权限仍会在密钥之上生效。

> **注意：** 无论您通过何种方式连接，Linear 的服务器都会暴露读取和写入工具。只读密钥会使 Linear 拒绝写入操作；请在 **Permissions** 选项卡中也将这些操作设置为 **Off**，以确保操作列表与密钥实际能执行的操作相匹配。

> **注意：** 本指南的早期版本要求您先注册自己的 Linear OAuth 应用。现在不再需要这样做。使用您自己的应用建立的连接仍可继续工作。

## 选择访问权限

访问范围由授权的 Linear 账户决定。这通常意味着该人员可以看到的每个团队和项目，因为 Linear 工作区通常在内部是开放的。

> **注意：** Paperclip 中没有团队或项目选择器。连接器定义中存在描述工作区、团队和项目范围的字段，但它们不会缩小智能体可以触及的范围——限制因素是授权账户在 Linear 中的自身访问权限。如果智能体应仅触及一个团队的问题，请使用仅限该团队的账户进行授权，或依赖操作设置和明确指令，而不是假设存在资源过滤器。

问题创建和更新属于写入操作。在您了解智能体行为时，请将它们保持在 **Ask first** 状态——一个热衷于提交问题的智能体会给整个团队造成干扰。读取操作可以保持为 **Allowed**。参见 [设置操作权限](action-permissions.md)。

身份在此处对于归属很重要：智能体创建或评论的问题将显示在授权账户名下。使用专用账户可以将智能体活动与个人活动区分开来。参见 [为人员和智能体使用独立账户](separate-accounts.md)。

## 尝试使用

```txt
Find Linear issue ENG-142 and tell me its status, assignee, and latest comment. Do not change it.
```

与 Linear 中的问题进行对比。读取一个已知问题可以确认凭据和智能体的权限，而不会向您的团队看板添加任何内容。

> **注意：** 这是示例任务，而非记录的测试结果。请替换为您自己工作区中的问题键。

## 故障排除与限制

| 问题 | 可能原因 | 修复方法 |
| --- | --- | --- |
| 授权未完成 | Linear 窗口被关闭，或同意被拒绝 | 返回连接并再次尝试；您的设置会被保留 |
| 找不到某个问题 | 授权账户无法看到该团队或项目 | 在 Linear 中授予访问权限；无需重新连接 |
| 智能体触及的团队比预期多 | 访问范围取决于授权账户，而非 Paperclip 过滤器 | 使用权限更有限的账户进行授权 |
| 智能体创建的问题看起来像是来自个人 | 连接使用的是个人身份 | 使用专用的智能体账户 |
| 在自托管实例上无法开始登录 | Paperclip 在非本地地址上通过纯 HTTP 提供服务 | 通过 HTTPS 提供 Paperclip 服务，或使用 **Use an API key** |
| 在密钥连接上写入失败并出现权限错误 | API 密钥仅具有 **Read** 权限 | 对于只读密钥这是预期行为；将写入操作设置为 **Off** |
| **Needs attention** | 授权已在 Linear 中撤销，或 API 密钥已被删除 | 选择 **Reconnect** |

限制：每个连接对应一个工作区。Paperclip 内部没有团队或项目限制。Linear 自身的速率限制适用。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/linear/)

- [Jira](jira.md)、[Asana](asana.md)、[Todoist](todoist.md) — 其他工作跟踪连接器。
- [为人员和智能体使用独立账户](separate-accounts.md)
- [设置操作权限](action-permissions.md)
- [Linear API documentation](https://developers.linear.app/)
