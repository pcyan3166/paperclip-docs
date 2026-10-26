---
paperclip_version: v2026.1005.0
seo_title: 设置您自己的 Google OAuth 应用
seo_description: 一种可复用的流程，用于使用您自己的 OAuth 客户端连接任何 Google Workspace 连接器——预览访问权限、API、回调 URI 及故障排除。
---

# 设置您自己的 Google OAuth 应用

> **警告：** **Google 验证待定。** Paperclip 尚未完成 Google 应用验证。在授权过程中，您可能会看到“未验证应用”的警告。如果 Google 提供 **Advanced**（高级）选项以继续前往 Paperclip，在审查所请求的访问权限后，您可以选择继续。并非所有账户都提供此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果您无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：** 在验证待定期间，Google Workspace 连接器以及任何已保存的 Google 账户将从 **Connectors**（连接器）页面暂时隐藏。这仅将其从列表中隐藏。现有的 Google 连接将继续运行，其访问权限和权限保持不变，需要 Google 服务的智能体仍可在其任务中通过连接卡片向您请求该服务。

本目录中有九个连接器与 Google Workspace 通信：Gmail、Calendar、Chat、Docs、Drive、People、Sheets、Slides 和 Workspace Search。使用 **您自己的** OAuth 客户端连接其中任何一个都遵循以下共享步骤，而本页即为该流程。

阅读一次并重复使用。为您的连接器和能力组选择合适的 API 和范围。Google Chat 还需要配置一个 Chat 应用。

> **注意：** 仅当您自带 OAuth 客户端时才需要此页面。如果连接器上提供了 **Connect with Paperclip**（通过 Paperclip 连接）且您愿意使用它，则该路径无需在 Cloud 控制台中进行任何操作——请跳至 [首先检查您是否可以使用托管路径](#check-you-can-use-the-managed-path-first)。

<a id="check-you-can-use-the-managed-path-first"></a>
## 首先检查您是否可以使用托管路径

存在两条路径，在花费时间在 Google Cloud 控制台上之前，了解您处于哪条路径是值得的。

| 路径 | 需要您提供什么 | 何时提供 |
| --- | --- | --- |
| **Connect with Paperclip** | 无需在 Cloud 控制台中进行任何操作。Paperclip 拥有该 OAuth 客户端 | 仅当实例已注册到 Paperclip Cloud *且* Cloud 提供该连接器的配置文件时。如果缺失，请询问管理员检查注册状态和配置文件可用性 |
| **Use your own Google OAuth app** | 本页上的所有内容 | 在连接器可用的地方；提供商批准和实例限制仍然适用 |

无论哪种方式，**第 1 步中的 Google 开发者预览注册均适用。** 在托管路径中，Google 仍然必须注册登录的 Workspace 账户；只有 Cloud 项目不同。

## 1. 获取开发者预览访问权限

Google 的 Workspace MCP 服务器处于开发者预览阶段。Google 分别注册两件事：将用于登录的 **Workspace 电子邮件**，以及拥有该 OAuth 客户端的 **Google Cloud 项目**。注册仅涵盖您指定的内容。

1. 在 [Google Workspace Developer Preview](https://developers.google.com/workspace/preview) 处申请，指定将授权 Paperclip 的 Workspace 电子邮件以及将拥有该客户端的 Cloud 项目。
2. 等待 Google 群组通知，然后是 Google 的项目注册邮件。Google 的指导通常是一两天。
3. 在连接之前，通过 Google 的成员申请表单添加每个额外的测试人员电子邮件和每个额外的 Cloud 项目。

> **警告：** 未注册的账户或项目将在 Google 的授权屏幕处失败，而不是在您在 Paperclip 中开始设置时失败。如果同意被拒绝，请检查注册状态、OAuth 应用验证或测试状态，以及您的 Workspace 管理员的访问策略。

此注册是按 Workspace 账户和按 Cloud 项目进行的，而不是按连接器进行的。执行一次，所有 Google 连接器都将受益。

## 2. 启用正确的 API

在 [Google Cloud 控制台](https://console.cloud.google.com/apis/library) 中，在您注册的项目上，为您正在设置的连接器启用 API。每个连接器需要其自己的 MCP API 以及底层服务 API，其中三个还需要 Drive API。

| 连接器 | 启用这些 API |
| --- | --- |
| **Gmail** | Gmail API, Gmail MCP API |
| **Google Calendar** | Calendar API, Calendar MCP API |
| **Google Chat** | Chat API, Chat MCP API — 外加一个已配置的 Chat 应用，见 [步骤 4](#4-google-chat-only-configure-a-chat-app) |
| **Google Docs** | Drive API, Docs API, Docs MCP API |
| **Google Drive** | Drive API, Drive MCP API |
| **Google People** | People API, People MCP API |
| **Google Sheets** | Drive API, Sheets API, Sheets MCP API |
| **Google Slides** | Drive API, Slides API, Slides MCP API |
| **Google Workspace Search** | Gmail API, Drive API, Calendar API, Chat API, Workspace MCP API |

在一个项目上设置多个连接器是正常的——启用它们所需内容的并集。

> **注意：** Google Sheets 是本目录中的例外。其自有客户端路径工作方式相同，但它还提供一种机器人账户设置，完全不需要 Cloud 项目。在为此页面操作之前，请参阅 [Google Sheets](google-sheets.md)。

## 3. 创建 OAuth 客户端并注册 Paperclip 的回调

首先配置 **Google Auth Platform → Branding**，设置应用标识和支持联系人。在 **Audience**（受众）下，选择适当的内部或外部受众；对于处于测试阶段的外部应用，添加每个已授权的测试人员。在 **Data Access**（数据访问）下，添加您打算使用的连接器和能力组所列的范围，然后保存。不要为只读连接添加写入范围。这些设置不替代开发者预览注册或 Google 的验证要求。

然后打开 [Google Cloud 控制台凭据屏幕](https://console.cloud.google.com/auth/clients)：

1. 创建一个类型为 **Web application**（Web 应用）的 **OAuth 客户端**。
2. 将 Paperclip 的回调添加为 **Authorized redirect URI**（已授权重定向 URI）。Paperclip 在设置期间会显示确切的 URI——请从那里复制而不是手动输入。它是您实例自身源上的 `/api/tools/oauth/callback` 路由。
3. 保存，然后保留 **client ID**（客户端 ID）和 **client secret**（客户端密钥）备用。

> **警告：** 重定向 URI 必须完全匹配，包括方案、主机、端口和尾部路径。不匹配会在同意屏幕产生 Google 的 `redirect_uri_mismatch` 错误。

Google 自己的操作指南是 [Configure MCP servers](https://developers.google.com/workspace/guides/configure-mcp-servers)。

<a id="4-google-chat-only-configure-a-chat-app"></a>
## 4. 仅限 Google Chat：配置 Chat 应用

Google Chat 需要其他连接器不需要的一项内容——Chat API 必须在项目上有一个已配置的 Chat 应用，而不仅仅是被启用。

在 Cloud 控制台中，打开 **Chat API** → **Configuration**（配置）并完成应用的标识：其应用名称、头像 URL 和描述。在 **Functionality**（功能）下，关闭 **Enable interactive features**（启用交互功能）；在 **Logs**（日志）下，启用错误日志记录并保存，如 Google 的 MCP 设置指南所述。Google 不会授权针对没有配置应用的 Chat API 的连接器。

## 5. 在 Paperclip 中连接

1. 打开 **Connectors**（连接器）并选择该连接器。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。
3. 检查 **capability group**（能力组）。大多数 Google 连接器提供读取组和写入组，设置从写入（或草稿）组开始。若要改为只读连接，请选择 **Change**（更改）并在 **What should Paperclip be able to do?**（Paperclip 应该能做什么？）下选择读取组。**该组在连接的生命周期内是固定的**：若要更改它，您需要建立新连接，因为 **Reconnect**（重新连接）会为同一组重新运行登录。
4. 选择 **Use your own Google OAuth app**（使用您自己的 Google OAuth 应用）并提供第 3 步中的客户端 ID 和密钥。
5. **以已注册的 Workspace 账户身份登录**，完成 Google 的同意屏幕。

然后打开 **Permissions**（权限）选项卡并设置操作。读取设为 **Allowed**（允许）；写入和破坏性操作设为 **Ask first**（先询问）或 **Off**（关闭），直到您观察过智能体的工作。

## 通过读取进行验证

使用您可以自己打开的资源，通过 URL 或 ID 标识，并使用您已知的值——然后检查答案是否匹配。明确标识资源，而不是要求智能体“按标题”查找：经过审查的读取操作接受标识符，因此基于标题的请求测试的是搜索而非连接。

每个连接器页面都有其自身资源类型的示例。

## 如果失败

| 症状 | 可能原因 | 修复方法 |
| --- | --- | --- |
| Google 在同意屏幕前拒绝 | 注册、应用验证/测试状态或 Workspace 策略阻止了访问 | 确认注册、测试用户资格和管理员批准。不要绕过被阻止的授权屏幕 |
| `redirect_uri_mismatch` | OAuth 客户端上的重定向 URI 与 Paperclip 显示的不匹配 | 从 Paperclip 的设置屏幕精确复制 URI |
| 同意完成但连接显示 **Setup incomplete**（设置未完成） | 最后一步未完成 | 选择 **Finish setup**（完成设置） |
| 首次使用时出现 API 未启用错误 | 第 2 步中的某个 API 在项目中缺失——通常是 MCP API 而非服务 API | 启用并重试；无需重新连接 |
| 未提供 **Connect with Paperclip** | 实例未注册到 Paperclip Cloud，或 Cloud 未提供该配置文件 | 使用您自己的 OAuth 客户端，或询问管理员关于注册的问题 |
| 您预期的工具缺失或已禁用 | 它超出了您连接的能力组的已审查操作列表 | 检查连接器页面的操作列表；列表外的预览工具被阻止，且没有开关可以启用它 |
| 尽管 API 已启用，但 Google Chat 授权失败 | 项目上没有配置 Chat 应用 | 完成 [步骤 4](#4-google-chat-only-configure-a-chat-app) |

更多详情见 [验证连接器并修复故障连接器](verify-and-troubleshoot.md)。

## 相关指南

- [Gmail](gmail.md) 和 [设置 Gmail 连接器](gmail-setup.md) — Gmail 有自己的页面，因为其能力组和发送限制需要更多解释。
- [Google Calendar](google-calendar.md), [Google Chat](google-chat.md), [Google Docs](google-docs.md), [Google Drive](google-drive.md), [Google People](google-people.md), [Google Sheets](google-sheets.md), [Google Slides](google-slides.md), [Google Workspace 搜索](google-workspace-search.md)
- [连接器访问机制说明](access-model.md)
- [Configure MCP servers](https://developers.google.com/workspace/guides/configure-mcp-servers) — Google 自己的指南。
