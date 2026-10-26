---
paperclip_version: v2026.1005.0
seo_title: 设置 Gmail 连接器
seo_description: 获取 Google 开发者预览版访问权限，使用“读取”或“草稿”能力组连接 Gmail，并限制哪些智能体可以搜索您的邮件。
---

# 设置 Gmail 连接器

> **警告：** **Google 验证待定。** Paperclip 尚未完成 Google 应用验证。在授权过程中，您可能会看到“未验证应用”的警告。如果 Google 提供 **Advanced**（高级）选项以继续前往 Paperclip，在审查所请求的访问权限后，您可以选择继续。并非所有账户都提供此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果您无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：** 在验证待定期间，Google Workspace 连接器以及任何已保存的 Google 账户会暂时从 **Connectors**（连接器）页面隐藏。这仅将其从列表中隐藏。现有的 Google 连接将继续运行，其访问权限和权限保持不变，需要 Google 服务的智能体仍可在其任务中通过连接卡片向您请求该服务。

分为两个阶段：首先让 Google 将您注册为开发者预览版用户，然后进行连接。第一阶段耗时较长，且完全在 Google 端进行。

关于连接器连接后能做什么和不能做什么，请参阅 [Gmail](gmail.md)。

## 1. 获取开发者预览版访问权限

Google 的 Workspace MCP 服务器处于开发者预览版阶段。Google 会注册用于登录的 Workspace 电子邮件以及拥有 OAuth 客户端的 Google Cloud 项目。注册仅涵盖您指定的电子邮件和项目。

1. 使用将授权 Paperclip 的 Workspace 电子邮件以及拥有 OAuth 客户端的 Google Cloud 项目，在 [Google Workspace Developer Preview](https://developers.google.com/workspace/preview) 处申请。
2. 等待 Google 群组通知，然后是 Google 的最终项目注册邮件。Google 的指导时间通常为几天。
3. 在连接之前，通过 Google 的成员申请表单添加所有额外的测试人员电子邮件和所有额外的 Cloud 项目。未注册的账户会在授权时失败，而不是在连接时失败。

如果您计划使用 **Connect with Paperclip**（通过 Paperclip 连接）而不是您自己的 OAuth 客户端，Workspace 电子邮件仍然需要注册——该路径上的 Cloud 项目属于 Paperclip。

## 2. 如果您自带 OAuth 客户端

如果您的实例已注册 Paperclip Cloud 且提供 **Connect with Paperclip** 选项，请跳过本节。

在 [Google Cloud console](https://console.cloud.google.com/auth/clients) 中：

1. 在项目上启用 **Gmail API** 和 **Gmail MCP API**。
2. 创建类型为 **Web application**（Web 应用）的 OAuth 客户端，并将 Paperclip 的回调添加为授权重定向 URI。Paperclip 会在设置过程中显示确切的 URI——请从那里复制；它是您实例源上的 `/api/tools/oauth/callback` 路由，且必须完全匹配。
3. 保留客户端 ID 和客户端密钥以备使用。

[Set up your own Google OAuth app](google-setup.md) 对此进行了全面介绍，包括重定向 URI 故障模式，并且这是适用于其他所有 Google 连接器的相同流程。Google 自己的操作指南是 [Configure MCP servers](https://developers.google.com/workspace/guides/configure-mcp-servers)。

## 3. 在开始之前决定能力组

这是至关重要的决定，且在整个连接生命周期内固定不变。

| 组 | 请求的作用域 | 使用场景 |
| --- | --- | --- |
| **Read only**（仅读取） | `gmail.readonly` | 智能体需要查找和读取邮件。不会向邮箱写入任何内容。 |
| **Read & create drafts**（读取并创建草稿） | `gmail.readonly`, `gmail.compose` | 智能体应留下草稿供人员审查和发送。 |

这两个组都不允许执行发送操作。设置从 **Read & create drafts** 开始。如果智能体只需要读取，请在连接时（步骤 4）切换为 **Read only**——除非您有具体理由需要草稿，否则这是更安全的选择。

若要稍后更改组，请创建具有所需组的第二个连接，并移除第一个连接。组是连接的一部分，因此 **Reconnect**（重新连接）会为相同的组重新运行登录，而不是更改组。

## 4. 连接

设置只有一个屏幕。Paperclip 会填写连接所属人员以及哪些智能体可以使用它的默认值，您可以在登录前更改任何不符合要求的内容。

1. 在侧边栏中选择 **Connectors**（连接器）。
2. 找到 **Gmail** 并选择 **Connect**（连接）。
3. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。对于邮箱，默认值通常不是您想要的，因此请选择 **Change**（更改）：
   - 在 **Which humans can use this credential?**（哪些人员可以使用此凭据？）下，为邮箱选择 **Just me**（仅我）通常最合适——智能体只会在您担任责任人的运行中使用该凭据。选择 **Any human in the organization**（组织中的任何人员）后，符合条件的智能体即可在运行责任人属于该凭据适用人员范围时使用邮箱。您可以在连接的身份卡片上将适用人员范围设为 **Any human in the company**（公司中的任何人员），或选择 **Humans I pick**（我选择的人员）并指定成员。对于邮箱，请缩小适用人员范围：若保持开放，任何同事发起的运行都可能读取收件箱。此设置需要连接管理器权限，适用于真正共享的收件箱，而非个人邮箱。
   - 在 **Which agents can use this connection**（哪些智能体可以使用此连接）下，选择 **Just agents I pick**（仅我选择的智能体）并命名它们。邮箱不应默认交给每个智能体。
   - 在 **What should Paperclip be able to do?**（Paperclip 应该能做什么？）下，从步骤 3 中选择能力组。
4. 选择设置路径：
   - **Connect with Paperclip**（通过 Paperclip 连接）——Paperclip 管理的 Google 客户端。仅在实例注册了 Paperclip Cloud 且 Cloud 提供 Gmail 配置文件时提供。当提供此选项时，设置默认使用它。
   - **Use your own Google OAuth app**（使用您自己的 Google OAuth 应用）——选择此项以提供步骤 2 中的客户端 ID 和密钥。**Use Paperclip instead**（改用 Paperclip）可切换回。
5. 使用已注册的 Workspace 账户完成 Google 的同意屏幕。

## 5. 设置操作

打开 **Permissions**（权限）选项卡。

- 保持读取操作为 **Allowed**（允许）。
- 如果您连接了草稿组，请考虑将 `create-draft` 设置为 **Ask first**（先询问）。草稿会进入真实邮箱，且连接器自身的指南建议对其进行审批。这是建议，而非 Paperclip 强制执行的措施——与发送不同，无论您如何设置，发送操作都会被直接阻止。

## 6. 通过读取进行验证

以您打算使用的智能体身份，运行一次您已知答案的读取操作——针对您在 Gmail 中可见的发送者进行线程搜索——并确认结果匹配。这共同确认了凭据、账户和智能体的权限。

然后检查连接的活动是否显示 Gmail 调用。仅匹配的输出并不能证明邮箱实际上已被读取。

不要通过创建草稿来验证。

## 如果授权失败

| 症状 | 原因 |
| --- | --- |
| Google 在同意屏幕前拒绝 | 登录的 Workspace 账户或 Cloud 项目未注册开发者预览版。 |
| 未提供 **Connect with Paperclip** | 实例未注册 Paperclip Cloud，或 Cloud 未提供 Gmail 配置文件。 |
| 同意完成但连接显示 **Setup incomplete**（设置未完成） | 最后一步未完成。选择 **Finish setup**（完成设置）。 |
| 重定向 URI 不匹配 | 在您的 Google OAuth 客户端上注册的回调 URI 与 Paperclip 显示的 URI 不匹配。 |

更多详情见 [验证连接器并修复故障连接器](verify-and-troubleshoot.md)。

## 相关

- [Gmail](gmail.md)
- [连接器访问机制说明](access-model.md)
- [为人员和智能体使用独立账户](separate-accounts.md)
- [Google Workspace 搜索](google-workspace-search.md)
