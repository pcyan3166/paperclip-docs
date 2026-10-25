---
seo_title: Telegram 连接器
seo_description: 允许用户通过向 Telegram 机器人发送消息来启动 Paperclip 工作。涵盖 BotFather 设置、公共 webhook 要求、智能体路由以及修复无响应机器人的方法。
---

# Telegram

用户向您的 Telegram 机器人发送消息，Paperclip 随即开始工作。回复将返回到同一个 Telegram 对话中。

一个连接对应一个机器人和一个智能体。

> **警告：** Telegram 通过调用 Paperclip 来传递消息，因此您的实例需要一个可公开访问的 HTTPS 地址。Telegram 仅接受端口为 **443、80、88 或 8443** 的 webhook。如果您的实例无法通过这种方式访问，此连接器将无法工作——与 [AgentMail](agentmail.md) 不同，它没有仅出站模式。

## 连接前准备

- 必须为实例开启 **Chat connectors**（聊天连接器）。这是一个实验性设置，默认关闭，需由实例管理员在实验性设置中启用。
- 实例需要一个可公开访问的 HTTPS URL，由管理员配置为 `PAPERCLIP_CHAT_WEBHOOK_PUBLIC_URL`，且端口必须是 Telegram 允许的端口之一。
- 一个 Telegram 账户，用于与 BotFather 交互。
- 将要进行回复的智能体。

## 连接 Telegram

### 1. 创建机器人

1. 在 Telegram 中向 [@BotFather](https://t.me/BotFather) 发送消息，并发送 `/newbot`。
2. 为机器人设置显示名称，以及以 `bot` 结尾的用户名。
3. 复制 BotFather 返回的 **bot token**（机器人令牌）。其格式类似于 `123456789:AA...`。

请为此连接器创建一个专用机器人，而不是复用已配置其他 webhook 的机器人——Telegram 机器人一次仅支持一个 webhook，因此在此处连接将替换其之前指向的任何内容。

> **危险：** 机器人令牌是该机器人的完整凭据。切勿将其粘贴到聊天、问题单或截图中。如果令牌泄露，请使用 BotFather 的 `/revoke` 进行轮换并重新连接。

### 2. 在 Paperclip 中连接

1. 打开 **Connectors**（连接器）并选择 **Telegram**。
2. 在 **Access**（访问）步骤中，选择身份以及哪些智能体可以使用该连接。
3. 粘贴 **Bot token**（机器人令牌）。
4. 选择将要进行回复的智能体，并完成设置。

Paperclip 会为您向 Telegram 注册 webhook，并使用生成的密钥以拒绝伪造的投递。您无需将任何 URL 粘贴到 BotFather 中。

## 对话如何转化为工作

| 在 Telegram 中 | 在 Paperclip 中 |
| --- | --- |
| 某人向机器人发送私信 | 为已连接的智能体创建一个任务 |
| 他们继续回复 | 对话在同一任务上继续 |
| 智能体响应 | 回复到达 Telegram 对话中 |

## 选择访问权限

这里涉及两个不同的控制项，分别位于两侧，混淆它们是应当避免的错误。

**在 Telegram 一侧没有发送者白名单。** 任何能找到该机器人的人都可以向其发送消息。不要将机器人的用户名视为机密——应将其视为公开信息，仅与您打算服务的人员共享。

**在 Paperclip 一侧存在控制项。** 连接的 **Access**（访问）选项卡包含 **External identity access**（外部身份访问），并带有 **Allow unlinked people**（允许未关联人员）开关：

| 设置 | 当未识别的人员向机器人发送消息时发生的情况 |
| --- | --- |
| **On**（开启，默认值） | 他们成为受限访客。其任务仅在隔离的工作区和沙箱环境中运行，如果这些环境不可用，Paperclip 将拒绝请求。访客无法批准、雇佣、支出、管理访问权限或重新分配智能体 |
| **Off**（关闭） | 他们完全无法启动工作。只有与 Paperclip 人员关联的发送者才能通过 |

因此，“任何人都可以向机器人发送消息”是真的，但“任何人都能让智能体执行任意工作”并非如此。在发布用户名**之前**，决定您想要哪种设置；对访客的限制是真实的，但它们并不等同于拒绝他们。

在群组中，Telegram 的默认隐私模式意味着机器人仅接收直接提及它的消息。在 BotFather 中关闭此功能将使机器人接收所有群组消息；除非您已决定需要此功能，否则请保持默认设置。

连接的身份和智能体设置与其他连接器相同；请参阅 [How connector access works](access-model.md)。回复智能体在连接中设置。

## 尝试使用

1. 在 Telegram 中打开您的机器人并发送 `hello, can you confirm you are connected?`
2. 预计对话中会在几秒内收到回复。
3. 确认在 Paperclip 中出现了一个匹配的任务，并分配给已连接的智能体。

首先从您自己的账户向机器人发送消息。这可以测试 webhook 投递、路由和任务创建，而不涉及其他人。

> **注意：** 这是操作程序，而非记录的测试结果。

## 故障排除与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| Telegram 未出现在 **Connectors**（连接器）中 | 实例的 **Chat connectors**（聊天连接器）已关闭 | 请管理员启用它 |
| 设置失败，提及 HTTPS 和端口 | 实例的公共 webhook URL 缺失或端口不受支持 | 管理员必须将 `PAPERCLIP_CHAT_WEBHOOK_PUBLIC_URL` 设置为 HTTPS，端口为 443、80、88 或 8443 |
| 机器人存在且接受消息但从不回复 | Telegram 无法访问实例，或另一个系统替换了 webhook | 重新连接该连接，这将重新注册 webhook |
| 机器人曾正常工作但随后无响应 | 令牌已被撤销，或另一个工具对同一机器人调用了 `setWebhook` | 如有需要请轮换令牌并重新连接；使用专用机器人 |
| 机器人在群组中忽略消息 | Telegram 隐私模式仅传递提及机器人的消息 | 直接提及机器人，或重新考虑群组是否适合作为入口 |
| 错误的智能体进行回复 | 回复智能体在连接中设置 | 在连接中更改它 |

限制：每个连接对应一个机器人和一个智能体。没有发送者白名单。Telegram 机器人只有一个 webhook，因此该机器人不能与其他集成共享。

## 相关指南

- [Discord](discord.md)、[Slack](slack.md)、[Microsoft Teams](microsoft-teams.md) — 其他对话渠道。
- [How connector access works](access-model.md)
- [Telegram bot tutorial](https://core.telegram.org/bots/tutorial)
