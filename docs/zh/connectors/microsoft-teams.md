---
seo_title: Microsoft Teams 连接器
seo_description: 允许用户从 Microsoft Teams 启动 Paperclip 工作。Entra 应用和 Azure Bot 设置、租户管理员审批、RSC 授权暴露的内容以及故障排查。
---

# Microsoft Teams

用户在 Microsoft Teams 中向您的智能体发送消息——无论是在个人聊天、团队频道还是群聊中——Paperclip 便会开始工作。

这是设置最复杂的渠道。它需要一个 Entra 应用程序、一个单租户 Azure Bot、一个 Teams 应用包以及租户管理员的审批。请为此预留充足的时间，并在安装前阅读下方的访问权限说明。

> **警告：** 位于 `teams.live.com` 的个人版或免费版 Teams 账户无法完成此设置。您需要一个 Microsoft 365 工作或学校组织，以便在其中注册 Entra 应用程序。

## 连接前准备

- 必须为该实例开启 **Chat connectors（聊天连接器）**。这是一个实验性设置，默认关闭，需由实例管理员在实验性设置中启用。
- 一个 Microsoft 365 工作或学校租户，以及在其中注册 Entra 应用程序的权限。
- 一位能够授予应用程序权限并允许安装 Teams 应用的租户管理员。如果这不是您本人，请在开始之前联系他们——没有他们，设置无法完成。
- **您的 Paperclip 实例必须能通过 HTTPS 从互联网访问。** Azure 通过调用 Paperclip URL 来传递每条消息，因此未配置公共基础 URL 的实例无法接收任何内容。如果 Paperclip 在步骤 2 中未显示消息端点，原因就在于此——请要求管理员配置实例的公共地址。
- 将要响应的智能体。

> **警告：** 此版本仅支持 Microsoft 365 商业云租户。不支持 GCC、GCC High、DoD 以及由 21Vianet 运营的 Microsoft 365。

## 安装可读取的内容

这一点在此处比在其他渠道中更为重要，因此请在安装前做出决定。

Teams 应用使用两个资源特定的同意权限：**ChannelMessage.Read.Group** 和 **ChatMessage.Read.Chat**。在应用安装的位置，这些权限允许其接收 **该团队或群聊中的每一条消息，无论是否有人提及该智能体。**

这是 Teams 向已安装机器人传递消息的方式，而非 Paperclip 的选择，但后果由您管理：在将应用安装到某个团队之前，告知该团队中的人员应用可以看到什么，并仅在可接受的情况下安装。与机器人的个人聊天仅暴露该聊天内容。

## 连接 Microsoft Teams

涉及四个门户，且顺序至关重要：Paperclip 生成 Azure 所需的消息端点，因此 **从 Paperclip 开始**，而不是在那里结束。相同的 Application ID 在三个地方被重用。

### 1. 注册 Entra 应用程序

在 **Microsoft Entra 管理中心** → **App registrations（应用注册）** 中，选择 **New registration（新注册）** 并选择 **Accounts in this organizational directory only (Single tenant)（仅限此组织目录中的账户（单租户））**。注册后，复制：

- **Application (client) ID（应用程序（客户端）ID）**
- **Directory (tenant) ID（目录（租户）ID）**

然后在 **Certificates & secrets（证书和密钥）** → **Client secrets（客户端密钥）** 下，选择 **New client secret（新客户端密钥）** 并复制其 **Value（值）**——而不是其 Secret ID。Entra 仅显示一次该值。

### 2. 在 Paperclip 中打开设置并复制消息端点

1. 打开 **Connectors（连接器）** 并选择 **Microsoft Teams**。
2. 在 **Access（访问）** 步骤中，选择身份以及哪些智能体可以使用该连接。
3. 设置屏幕显示 **Paperclip messaging endpoint（Paperclip 消息端点）**。复制它——Azure 在下一步需要它。保持此屏幕打开；您将返回此处以完成设置。

### 3. 创建 Azure Bot，将其指向 Paperclip，并添加 Teams 渠道

在 **Azure** → **Create Azure Bot（创建 Azure Bot）** 中：

1. 将 **Microsoft App ID** 类型设置为 **Single Tenant（单租户）**，将 **Creation type（创建类型）** 设置为 **Use existing app registration（使用现有应用注册）**，然后输入步骤 1 中的 Application ID 和 Tenant ID。
2. 创建后，打开 **Settings（设置）** → **Configuration（配置）** 并粘贴步骤 2 中的 Paperclip **Messaging endpoint（消息端点）**。
3. 打开 **Settings（设置）** → **Channels（渠道）** 并启用 **Microsoft Teams**。

> **警告：** 上述最后两项均为必需。仅创建机器人是不够的——没有消息端点，Azure 无处传递消息；没有 Teams 渠道，Teams 根本不会路由到机器人。跳过其中任何一项的设置看起来是完整的，但实际上静默地接收不到任何内容。

### 4. 构建 Teams 应用包

在 [Teams Developer Portal](https://dev.teams.microsoft.com/apps) → **Apps（应用）** 中，选择 **New app（新应用）**：

1. 在 **Configure（配置）** → **App features（应用功能）** → **Bot（机器人）** 下，使用 **相同的 Application ID** 添加现有机器人，并启用 **Personal（个人）**、**Team（团队）** 和 **Group chat（群聊）** 范围以及文件支持。
2. 在 **Configure（配置）** → **Permissions（权限）** 下，添加两个资源特定的同意 **Application（应用程序）** 权限。Paperclip 显示要使用的确切清单块——首先在 Paperclip 设置屏幕上输入您的 Client ID，然后使用 **Copy manifest settings（复制清单设置）**。
3. 完成所需的应用详情和图标，在其中说明该应用可以接收已安装团队或群聊中的每条消息，然后下载应用包。

> **注意：** 这些资源特定的同意权限通过 Teams 清单绑定到 Entra 应用。它们 *不是* Microsoft Graph 权限，您不会在 Entra 的 API 权限列表中找到它们。

### 5. 在需要提及功能生效的位置安装

在 **Teams** → **Apps（应用）** → **Manage your apps（管理您的应用）** 中，选择 **Upload an app（上传应用）** → **Upload a custom app（上传自定义应用）**，选择下载的应用包，并在每个预期的个人聊天、群聊或团队中安装。

一次团队安装覆盖该团队的标准频道。如果无法上传，Teams 管理员必须启用或批准自定义应用。

### 6. 在 Paperclip 中完成

返回设置屏幕并粘贴 **Application / Client ID（应用程序 / 客户端 ID）**、**Directory / Tenant ID（目录 / 租户 ID）** 和 **Client secret value（客户端密钥值）**。选择将要响应的智能体，并完成设置。

Microsoft 自己的操作指南是 [Create a bot for Teams](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/create-a-bot-for-teams)。

## 对话如何转化为工作

| 在 Teams 中 | 在 Paperclip 中 |
| --- | --- |
| 有人在个人聊天中向机器人发送消息 | 为连接的智能体创建一个任务 |
| 受支持的消息到达已安装团队频道或群聊中的机器人 | Paperclip 在启动智能体工作前应用其触发器和发送者授权检查 |
| 对话继续 | 回复继续同一任务 |

## 选择访问权限

可达范围由应用在 Teams 中的安装位置决定，而非在 Paperclip 中。连接中没有团队或频道选择器。

一次团队安装覆盖该团队的 **标准** 频道。**私有和共享频道需要单独安装，且在此版本中不受支持**——智能体不会看到其中的消息，因此不要围绕私有频道规划工作。

外部安装并非完整的权限边界。Paperclip 还会检查发送者的关联身份和成员资格，或未关联人员的配置策略。在邀请用户之前，请审查这些设置和连接上的响应智能体。参见 [How connector access works](access-model.md)。

## 尝试使用

从个人聊天开始，这暴露的内容最少：

1. 在 Teams 中，打开与该应用的个人聊天并发送 `hello, can you confirm you are connected?`
2. 预计在该聊天中几秒内收到回复。
3. 确认在 Paperclip 中出现一个匹配的任务，分配给连接的智能体。

仅在该功能正常工作后，再安装到团队中——并首先告知其成员应用可以读取什么。

> **注意：** 这是程序说明，而非记录测试结果。

## 故障排查和限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| **Connectors（连接器）** 中未出现 Microsoft Teams | 该实例的 **Chat connectors（聊天连接器）** 已关闭 | 要求管理员启用它 |
| 无法注册或安装应用 | 账户是个人版或免费版 Teams 账户 | 使用 Microsoft 365 工作或学校租户 |
| 无法完成同意 | 权限需要租户管理员审批 | 要求租户管理员授予同意 |
| 设置后身份验证失败 | 客户端密钥是从 ID 字段复制的，或已过期 | 创建新密钥，复制其值，并重新连接 |
| 机器人在个人聊天中回复，但在频道中不回复 | 应用未安装在该团队中，或频道是私有或共享的 | 安装到团队中；不支持私有和共享频道 |
| 尽管设置已完成，但任何地方都没有收到消息 | Azure Bot 的消息端点未设置或错误，或其 **Microsoft Teams** 渠道从未启用 | 在 Azure 的 **Settings · Configuration（设置 · 配置）** 和 **Settings · Channels（设置 · 渠道）** 下检查两者 |
| Paperclip 未显示可复制的消息端点 | 实例未配置公共基础 URL | 要求管理员配置；Azure 无法向不可达的实例传递消息 |
| Azure 报告向端点传递失败 | Paperclip 在该 URL 下无法从互联网访问 | 确认实例的公共 HTTPS 入口 |
| 消息到达但未创建任务 | 连接不健康，或未分配智能体 | 检查连接状态和已分配的智能体 |
| 用户对机器人能看到所有内容感到惊讶 | RSC 授权传递已安装团队或聊天中的所有消息 | 解释访问权限，或从该团队卸载 |

限制：每个连接仅限一个租户和一个智能体。不支持私有或共享频道。仅限 Microsoft 365 商业云租户——不支持 GCC、GCC High、DoD 或 21Vianet。Paperclip 无法缩小已安装应用接收的内容；这是由 Teams 权限模型固定的。

## 相关指南

- [Slack](slack.md)、[Discord](discord.md)、[Telegram](telegram.md) — 其他对话渠道。
- [How connector access works](access-model.md)
- [Create a bot for Teams](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/create-a-bot-for-teams)
