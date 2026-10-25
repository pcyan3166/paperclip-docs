---
paperclip_version: v2026.1023.0
seo_title: Calendly 日程安排连接器
seo_description: 允许智能体查看 Calendly 的活动类型和可用时间，并决定其可执行哪些预订操作。包括浏览器登录、权限范围、安全测试及故障排除。
---

# Calendly

智能体可以查看您的 Calendly 账户——包括您的活动类型、可用时间以及您登录的身份——并在您允许的情况下，使用 Calendly 的日程安排操作，例如预订或创建链接。

Calendly 出现在每个实例的目录中；无需启用实验性设置。

日程安排操作会触达真实用户。预订会进入某人的日历，通常还会向他们发送电子邮件，因此请像对待发送邮件一样谨慎对待此处的写入操作。

## 连接前准备

- 一个 Calendly 账户。该连接将支持该账户能够访问的任何活动类型和日历。
- 一个 Calendly 在登录后可以回调的 Paperclip 地址：一个 HTTPS 地址，或者当您在本地机器上运行 Paperclip 时，使用本地 `http://` 回环地址。

没有 API 密钥选项。Calendly 的托管服务器仅接受使用自动注册客户端的浏览器登录，因此个人访问令牌和手动创建的 OAuth 客户端在此处无法使用。

## 连接 Calendly

1. 打开 **连接器** 并选择 **Calendly**。
2. 阅读主按钮上方的访问说明行。它说明了该连接属于谁以及哪些智能体可以使用它。若要选择不同身份或缩小智能体范围，请选择 **更改**。
3. 选择 **继续前往 Calendly** 并在 Calendly 自身的授权界面中批准访问。

Paperclip 会自动向 Calendly 注册其客户端，因此无需在开发者控制台中进行任何设置。

## 选择访问权限

Paperclip 向 Calendly 请求两个权限范围：`mcp:scheduling:read` 和 `mcp:scheduling:write`。Calendly 将它们作为完全访问权限一起提供，这意味着即使您只希望智能体检查可用时间，该连接也能执行超出读取范围的操作。

新连接默认对所有智能体将所有操作设置为 **允许**。在将此权限交给智能体之前，请打开 **权限** 选项卡并查看 Calendly 实际发布的内容：

| 操作类型 | 建议设置 |
| --- | --- |
| 读取您的个人资料、活动类型和可用时间 | **允许** |
| 创建日程安排链接 | **先询问** |
| 预订、重新安排或取消 | **先询问** 或 **关闭** |

当智能体确实安排了某事时，请要求它先检查账户、活动类型、时区、可用时间和参与者详情。错误的时区是预订悄悄出错的最常见原因。参见 [设置操作权限](action-permissions.md)。

## 尝试使用

从一个不触及任何内容的读取操作开始：

```txt
Using Calendly, tell me which account you are connected as and list my event types with their durations. Do not book or create anything.
```

将该列表与您的 Calendly 仪表板进行比较。匹配的列表可以一次性确认登录状态、账户以及智能体的权限。

> **注意：** 这是示例任务，而非记录在案的测试结果。

## 故障排除与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 登录失败或返回错误 | Calendly 无法访问回调地址，或首次注册尝试被拒绝 | 确保通过 HTTPS（或本地回环）访问 Paperclip，然后尝试重新登录 |
| 缺少某个活动类型 | 它属于另一个 Calendly 账户或用户 | 使用拥有该活动类型的账户进行连接 |
| 时间看起来相差数小时 | 智能体使用了错误的时区 | 要求智能体在提出时间建议前说明其使用的时区 |
| 智能体意外预订了会议 | 日程安排写入操作被设置为 **允许** | 将预订操作设置为 **先询问** 或 **关闭**，并在 Calendly 中取消该预订 |
| **需要关注** | 授权已被撤销或过期 | 选择 **重新连接** |

限制：每个连接仅支持一个 Calendly 账户，且浏览器登录是唯一方法。可用的操作取决于 Calendly 托管服务器所暴露的内容。

## 相关指南

- [Google Calendar](google-calendar.md) — 读取和更改 Google 日历中的事件。
- [连接器访问机制](access-model.md)
- [设置操作权限](action-permissions.md)
- [Calendly MCP 服务器文档](https://developer.calendly.com/docs/mcp/calendly-mcp-server)

## 来源

- [Calendly 定义](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/calendly.json) — 方法、权限范围、指南和警告。
- [默认操作策略](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions.ts) — 新连接对所有智能体开放，操作设置为允许。
