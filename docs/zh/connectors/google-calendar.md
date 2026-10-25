---
paperclip_version: v2026.1005.0
seo_title: Google Calendar 连接器
seo_description: 允许智能体读取日历和空闲状态，并可选地管理事件。包含能力组、邀请副作用以及安全的读取测试。
---

# Google Calendar

> **警告：** **Google 验证待定。** Paperclip 尚未完成 Google 应用验证。在授权过程中，您可能会看到“未验证应用”的警告。如果 Google 提供 **Advanced**（高级）选项以继续前往 Paperclip，在审查所请求的访问权限后，您可以选择继续。并非所有账户都提供此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果您无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：** 在验证待定期间，Google Workspace 连接器以及任何已保存的 Google 账户将从 **Connectors**（连接器）页面暂时隐藏。这仅将其从列表中隐藏。现有的 Google 连接将继续运行，其访问权限和权限保持不变，需要 Google 服务的智能体仍可在其任务中通过连接卡片向您请求该服务。

智能体可以读取您 Google 账户上的日历，查找事件并检查空闲状态。在管理型连接中，它们还可以创建、更新、删除事件以及对事件做出响应。

> **警告：** Google Calendar 在授权前需要 Google Workspace 开发者预览版注册。Google 必须注册用于登录的 Workspace 电子邮件，如果您使用自己的 OAuth 客户端，还必须注册拥有该 OAuth 客户端的 Cloud 项目。请先在 [Google Workspace Developer Preview](https://developers.google.com/workspace/preview) 处申请。

## 连接前准备

- 一个能够查看您希望智能体使用的日历的 Google Workspace 账户。
- 该账户的开发者预览版注册，需由 Google 完成并确认。在连接之前注册每个额外账户；未注册的账户将在授权时失败，而非在连接时失败。
- 如果您的实例未注册 Paperclip Cloud，则不会提供 **Connect with Paperclip**（通过 Paperclip 连接）选项，您需要拥有自己的 Google OAuth 客户端，并启用 **Calendar API** 和 **Calendar MCP API**。[Set up your own Google OAuth app](google-setup.md) 是完整的流程——请在开始此处操作之前完成。

## 选择能力组

该组在连接的生命周期内是固定的。若要更改，请创建新连接。

| 组 | 智能体可以执行的操作 | 请求的作用域 |
| --- | --- | --- |
| **Read only**（仅读取） | 读取日历列表，读取和搜索事件，检查空闲/忙碌状态 | `calendar.calendarlist.readonly`, `calendar.events.freebusy`, `calendar.events.readonly` |
| **Read & manage**（读取与管理） | 上述操作，外加创建、更新、删除和对事件做出响应 | `calendar.calendarlist.readonly`, `calendar.events` |

**Read & manage**（读取与管理）不再单独请求空闲/忙碌作用域；`calendar.events` 已涵盖建议时间的功能。使用较早版本建立的连接请求了更多权限。如果在此更改之前建立的 **Connect with Paperclip** 连接停止工作，请选择 **Reconnect**（重新连接）以使用较小的权限集进行登录。使用您自己的 OAuth 应用的连接将保留其先前的授权，直到您重新连接为止。

## 连接 Google Calendar

1. 打开 **Connectors**（连接器）并选择 **Google Calendar**。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他登录方法，请选择 **Change**（更改）。
3. 检查能力组。设置从可以执行更改的组开始；若要仅读取连接，请选择 **Change** 并在 **What should Paperclip be able to do?**（Paperclip 应该能够做什么？）下选择它。当您的实例提供时，Paperclip 使用 **Connect with Paperclip**。若要改用您自己的客户端，请选择 **Use your own Google OAuth app**（使用您自己的 Google OAuth 应用）并提供来自 [Set up your own Google OAuth app](google-setup.md) 的客户端 ID 和密钥。
4. 使用已注册的 Workspace 账户完成 Google 的同意屏幕。

## 选择访问权限

智能体可以访问哪些日历由 Google 决定，而非 Paperclip：它包括授权账户日历列表中显示的任何内容，包括与该账户共享的日历。Paperclip 中没有日历选择器。如果智能体不应看到某个日历，请在 Google Calendar 中移除共享，或使用没有该日历的账户进行授权。

此连接器审查的操作：

| 操作 | 组 |
| --- | --- |
| `list-calendars`, `list-events`, `search-events`, `get-event`, `suggest-time` | 两者 |
| `create-event`, `update-event`, `respond-to-event` | 仅限 **Read & manage** |
| `delete-event` | 仅限 **Read & manage**，归类为破坏性操作 |

> **警告：** 事件写入在 Paperclip 之外具有副作用。创建或更新包含参与者的事件可能会从您的账户发送邀请和通知，且 `respond-to-event` 会以您的身份回复邀请。连接器自身的指导原则是所有事件变更都应经过批准——请将它们保持在 **Ask first**（先询问）状态。

[How connector access works](access-model.md) 涵盖身份和智能体选择；[Set action permissions](action-permissions.md) 是操作指南。

## 尝试使用

读取一个您可以肉眼确认的事件，且不涉及其他人：

```txt
What is on my Google Calendar tomorrow? List event titles and times only. Do not create, change, or respond to anything.
```

将结果与 Google Calendar 进行比较。时间返回时带有事件自身的时间区，因此请对照日历的显示时间区进行检查，而不是假设使用您本地的时间区。

不要使用共享日历上的测试事件进行验证——参与者会收到通知。如果您想确认写入操作，请在没有参与者的私有日历上创建事件，并在之后将其删除。

> **注意：** 这是示例任务，而非记录的测试结果。

## 故障排除和限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| Google 在同意屏幕前拒绝 | 登录账户的开发者预览版注册未完成 | 完成注册并重试 |
| 未提供 **Connect with Paperclip** | 实例未注册 Paperclip Cloud，或 Cloud 未宣传 Calendar 配置文件 | 使用您自己的 Google OAuth 应用，或询问管理员 |
| 缺少某个日历 | 它不在授权账户的日历列表中 | 在 Google Calendar 中订阅或共享该日历；无需重新连接 |
| 缺少事件写入 | 连接是以 **Read only** 建立的 | 建立 **Read & manage** 连接 |
| 时间因固定偏移量看起来不正确 | 事件的时间区与您正在读取的时间区不同 | 对照日历的时间区进行比较 |
| **Needs attention**（需要关注） | Google 令牌已过期或授权已被撤销 | 选择 **Reconnect** |

限制：一个连接覆盖一个 Google 账户。日历共享和访问控制更改，以及日历的创建或删除，均未暴露——仅暴露事件。Google 的 Workspace MCP 服务器处于开发者预览版，因此请将接口视为可能发生变化。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/google-calendar/)

- [Google Workspace Search](google-workspace-search.md) — 跨 Gmail、Drive、Calendar 和 Chat 的单一只读搜索。
- [How connector access works](access-model.md)
- [Verify a connector and fix a broken one](verify-and-troubleshoot.md)
- [Google Calendar API MCP reference](https://developers.google.com/workspace/calendar/api/v3/reference/mcp)
