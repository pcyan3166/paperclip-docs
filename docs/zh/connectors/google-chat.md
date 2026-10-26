---
paperclip_version: v2026.1005.0
seo_title: Google Chat 连接器
seo_description: 允许智能体搜索和阅读 Google Chat 对话，并可选择发送消息。这是一个工具连接器，而非供人员与智能体聊天的渠道。
---

# Google Chat

> **警告：** **Google 验证待处理。** Paperclip 尚未完成 Google 应用验证。在授权过程中，您可能会看到未验证应用的警告。如果 Google 提供 **Advanced**（高级）选项以继续前往 Paperclip，在审查所请求的访问权限后，您可以选择继续。并非所有账户都提供此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果您无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：** 在验证待处理期间，Google Workspace 连接器以及任何已保存的 Google 账户会暂时从 **Connectors**（连接器）页面隐藏。这仅将其从列表中隐藏。现有的 Google 连接将继续运行，其访问权限和权限保持不变，需要 Google 服务的智能体仍可在其任务中通过连接卡片向您请求该服务。

智能体可以搜索 Google Chat 对话并阅读消息，在发送连接上，可以向授权账户所属的空间发布消息。

这是一个工具连接器：智能体使用*您的* Google 账户读取和写入 Chat。它不是供人员通过向智能体发消息来开始工作的渠道——Google Chat 不是 Paperclip 的对话渠道之一。如果您需要的是这种功能，请参阅 [Slack](slack.md)、[Discord](discord.md)、[Microsoft Teams](microsoft-teams.md) 或 [Telegram](telegram.md)。

> **警告：** Google Chat 在授权前需要 Google Workspace Developer Preview 注册。Google 必须注册用于登录的 Workspace 电子邮件，如果您自带客户端，还必须注册拥有 OAuth 客户端的 Cloud 项目。请先在 [Google Workspace Developer Preview](https://developers.google.com/workspace/preview) 处申请。

## 连接前准备

- 一个已属于您希望智能体读取的空间的 Google Workspace 账户，且已确认 Developer Preview 注册。
- 个人 Google 账户没有 Workspace 意义上的 Google Chat 空间；此连接器期望使用 Workspace 账户。

### 现在决定您的设置路径

这是值得在一切之前检查的关卡，因为这两条路径所需的工作量差异很大：

| 路径 | 所需条件 | 提供时机 |
| --- | --- | --- |
| **Connect with Paperclip**（通过 Paperclip 连接） | 仅需登录，无需 Cloud 控制台操作 | 仅当实例已注册 Paperclip Cloud *且* Cloud 提供 Chat 配置文件时。如果设置屏幕上没有显示，则对您不可用 |
| **Use your own Google OAuth app**（使用您自己的 Google OAuth 应用） | 一个启用了 **Chat API** 和 **Chat MCP API** 的 Cloud 项目，一个已注册的回调 URI，**以及一个已配置的 Chat 应用** | 始终提供 |

**Chat 是唯一需要配置 Chat 应用的 Google 连接器**，而不仅仅是启用 API。在 Cloud 控制台中，打开 **Chat API** → **Configuration**（配置）并完成应用的标识信息——应用名称、头像 URL 和描述——然后设置其功能和可见性。Google 不会针对没有配置应用的 Chat API 进行授权，且失败会出现在同意屏幕而非更早的阶段。

[Set up your own Google OAuth app](google-setup.md) 是包含此步骤的完整程序。

## 选择能力组

| 组 | 智能体可以执行的操作 | 请求的作用域 |
| --- | --- | --- |
| **Read only**（仅读取） | 搜索对话，列出和搜索消息 | `chat.spaces.readonly`, `chat.memberships.readonly`, `chat.messages.readonly`, `chat.users.readstate.readonly` |
| **Read & send**（读取与发送） | 上述操作，外加发送消息 | 读取作用域，外加 `chat.messages.create` |

该组在连接的生命周期内是固定的。

## 连接 Google Chat

1. 打开 **Connectors**（连接器）并选择 **Google Chat**。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。要选择不同身份、缩小智能体范围或使用其他登录方法，请选择 **Change**（更改）。
3. 检查能力组。设置从可以执行更改的组开始；若要仅读取连接，请选择 **Change** 并在 **What should Paperclip be able to do?**（Paperclip 应该能够做什么？）下选择它。当您的实例提供时，Paperclip 使用 **Connect with Paperclip**（通过 Paperclip 连接）。若要改用您自己的客户端，请选择 **Use your own Google OAuth app**（使用您自己的 Google OAuth 应用）并提供来自 [Set up your own Google OAuth app](google-setup.md) 的客户端 ID 和密钥。
4. 使用已注册的 Workspace 账户完成 Google 的同意屏幕。

## 选择访问权限

智能体可以访问哪些对话由 Google 决定：即授权账户所属的空间和直接消息。Paperclip 没有空间选择器。若要缩小范围，请使用属于较少空间的账户进行授权。

审查的操作：

| 操作 | 组 |
| --- | --- |
| `search-conversations`, `list-messages`, `search-messages` | 两者 |
| `send-message` | 仅限 **Read & send**（读取与发送） |

> **警告：** 发送的消息对空间中的每个人可见，并以授权账户的名义到达，而非智能体的名义。连接器的指导原则是发送应经过批准——请将 `send-message` 保持为 **Ask first**（先询问）。

[连接器访问机制说明](access-model.md) 涵盖了身份和智能体选择。

## 尝试使用

选择一个授权账户所属的空间，以及其中一条您可以亲自看到发送者和措辞的近期消息。然后：

```txt
In the Google Chat space "deploys", find the most recent message and tell me who
sent it and when. Do not send a message.
```

预期结果是您已在 Chat 中看到的发送者和时间戳。匹配特定的已知消息是关键——听起来合理的摘要并不能证明智能体读取了任何内容。

如果您必须确认发送，请发布到一个为此目的创建且仅包含您自己的空间。在 Chat 中，“与自己”的直接消息是模糊的，是一个糟糕的测试目标；而草稿空间是明确的，同样无害。

> **注意：** 这是示例任务，而非记录的测试结果。

## 故障排除和限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| Google 在同意屏幕前拒绝 | Developer Preview 注册未完成 | 完成注册并重试 |
| 自管理客户端授权失败 | Cloud 项目中未配置 Google Chat 应用 | 配置 Chat 应用，然后重试 |
| 未提供 **Connect with Paperclip**（通过 Paperclip 连接） | 实例未注册 Paperclip Cloud，或 Cloud 未提供 Chat 配置文件 | 使用您自己的 Google OAuth 应用 |
| 结果中缺少某个空间 | 授权账户不是该空间的成员 | 在 Google Chat 中加入该空间；无需重新连接 |
| `send-message` 缺失 | 连接是以 **Read only**（仅读取）方式建立的 | 建立 **Read & send**（读取与发送）连接 |
| 人员期望向智能体发消息但得不到回复 | Google Chat 不是 Paperclip 对话渠道 | 改用受支持的渠道 |
| **Needs attention**（需要关注） | Google 令牌已过期或被撤销 | 选择 **Reconnect**（重新连接） |

限制：一个连接覆盖一个 Google 账户。创建、加入或离开空间以及更改成员资格未暴露。表情回应、消息编辑和删除不可用。适用 Developer Preview。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/google-chat/)

- [Slack](slack.md)、[Discord](discord.md)、[Microsoft Teams](microsoft-teams.md)、[Telegram](telegram.md) — 人员可用于联系智能体的渠道。
- [Google Workspace 搜索](google-workspace-search.md) — 跨 Gmail、Drive、Calendar 和 Chat 的单一只读搜索。
- [连接器访问机制说明](access-model.md)
- [Google Chat API MCP reference](https://developers.google.com/workspace/chat/api/reference/mcp)
