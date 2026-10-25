---
paperclip_version: v2026.1023.0
seo_title: Resend 连接器
seo_description: 让智能体使用 Resend 进行邮件发送——管理域名、受众群体及投递状态。为何发送并非配置测试、域名范围限制及故障排查。
---

# Resend

智能体可以操作您的 Resend 账户——检查投递状态、查看域名和受众群体，并在您授权的情况下发送电子邮件。

> **警告：** 此连接器可访问真实的发送管道。从 Resend 发送的邮件会进入真实收件箱，计入您的套餐用量，并影响您域名的发送声誉。请勿通过发送消息来验证此连接器。

Resend 用于从您自己的域名进行程序化邮件发送。如果您希望智能体拥有自己的收件箱并将对话视为任务，请使用 [AgentMail](agentmail.md)。如果您希望智能体读取您的个人邮箱，请使用 [Gmail](gmail.md)。

## 连接前准备

- 一个具有您希望智能体操作的域名访问权限的 Resend 团队账户。当 Resend 提示时，请登录该连接所属的团队。
- 如果涉及发送功能，需具备已验证的发送域名。Resend 要求通过 DNS 验证域名后才能从该地址发送，此连接器不会改变这一要求。

## 连接 Resend

1. 打开 **Connectors** 并选择 **Resend**。
2. 阅读主按钮上方的访问权限说明。该说明指出了连接登录的身份以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他登录方式，请选择 **Change**。
3. 选择 **Continue to Resend** 并完成浏览器登录。

Paperclip 会自动注册其客户端，因此无需在开发者控制台中进行任何配置。

## 选择访问权限

访问范围即为授权 Resend 团队的：其域名、受众群体及投递历史。

Paperclip 向 Resend 请求 `full_access` 权限范围。该单一授权涵盖对团队内所有内容的读取和管理，包括发送电子邮件和删除资源。Resend 为此登录方式不提供更窄的授权范围。

> **重要：** 在 Paperclip 中将操作设置为 **Off** 可阻止智能体调用这些操作，但不会缩小 Resend 签发的令牌权限。令牌本身仍具备 `full_access` 允许的所有权限。Paperclip 的操作设置是边界，因此请谨慎设置。

请明确区分各项操作，因为它们的后果差异巨大：

| 操作 | 后果 |
| --- | --- |
| 读取投递状态、域名、受众群体 | 无数据外泄 |
| 管理受众群体和联系人 | 更改邮件列表中的成员 |
| 发送电子邮件 | 立即且不可逆地向真实人员发送真实邮件 |

> **危险：** 请将发送操作保持为 **Off**，或至少设置为 **Ask first**。能够不受监督发送邮件的智能体可能会向您的客户发送邮件。邮件无法撤回，错误不仅影响收件人，还会损害您的域名声誉。

受众群体的变更也需谨慎——移除联系人会丢失订阅记录，添加联系人可能涉及同意权问题。请参阅 [Set action permissions](action-permissions.md)。

## 试用

通过读取操作进行验证：

```txt
List the verified sending domains on the Resend account and tell me the delivery status of the most recent email. Do not send anything.
```

将其与 Resend 仪表板进行对比。这既能确认凭据的有效性，又能同时显示域名配置，且不会发送任何内容。

> **注意：** 已验证的域名意味着 Resend 已确认其 DNS 记录，并不代表此凭据有权从该域名发送。智能体是否实际能够发送取决于 API 密钥自身的权限以及 Paperclip 中发送操作的设置——列出域名无法告知这两点。在推理此连接的能力时，请将这三者分开考虑。

如果您最终需要确认发送功能，请仅向您个人控制的地址、从您拥有的域名发送，且仅在您决定智能体应具备该能力之后进行。

> **注意：** 此为示例任务，非记录在案的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 未列出任何域名 | 账户中没有域名，或没有已验证的域名 | 在 Resend 中添加并验证域名 |
| 发送被拒绝 | 域名未验证，或发送操作设置为 **Off** | 在 Resend 中验证域名；检查 **Permissions** 选项卡 |
| 邮件已发送但未到达 | 收件人过滤，或域名声誉及 DNS 记录问题 | 检查 Resend 的投递日志以及您的 SPF、DKIM 和 DMARC 记录 |
| 智能体意外向某人发送邮件 | 发送操作被设置为 **Allowed** | 将其设置为 **Off**，并在 Resend 的日志中审查已发送内容 |
| 发送被限流 | Resend 套餐的速率限制 | 检查您的套餐限制 |
| **Needs attention** | 授权已被撤销 | 选择 **Reconnect** |

限制：每个连接对应一个 Resend 账户。域名验证由 Resend 负责，无法绕过。Paperclip 无法撤回已发送的邮件。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/resend/)

- [AgentMail](agentmail.md) — 为智能体提供独立收件箱，并将邮件转化为任务。
- [Gmail](gmail.md) — 读取现有邮箱。
- [Set action permissions](action-permissions.md)
- [Resend remote MCP server](https://resend.com/changelog/remote-mcp-server)
