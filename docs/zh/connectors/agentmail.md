---
paperclip_version: v2026.1012.0
seo_title: AgentMail 连接器
seo_description: 使用 AgentMail 为 Paperclip 智能体提供专属电子邮件收件箱。每次对话都会转化为一个任务。包含设置、路由、发件人限制及故障排查指南。
---

# AgentMail

AgentMail 为智能体提供专属的电子邮件地址。到达的邮件会转化为 Paperclip 任务，智能体处理该任务，其回复将通过同一线程发送回去。

这是一个对话通道，而非智能体调用以操作您自身邮件的工具集。它不读取现有的邮箱——该收件箱归属于智能体。如需读取您自己的 Gmail，请参阅 [Gmail](gmail.md)。

> **警告：** 任何知道该地址的人都可以向不受限制的收件箱发送邮件，且传入的消息可以创建任务并启动智能体工作。有两个独立的控制项管理此行为——AgentMail 自身的允许列表，以及 Paperclip 的 **Allow unlinked people**（允许未关联人员）设置。在公开该地址之前，请同时设置这两项；请参阅 [两个发件人控制项，位于两侧](#two-sender-controls-on-two-different-sides)。

## 连接前准备

- 一个 AgentMail 账户以及来自 [AgentMail API 密钥页面](https://console.agentmail.to/dashboard/api-keys) 的 API 密钥。组织或 pod 密钥可以创建新地址；仅限单个收件箱的密钥只能连接该收件箱。
- 将拥有该收件箱的智能体。一个收件箱归属于一个智能体。
- 若要使用您自己的域名，请先在 AgentMail 中验证该域名。`agentmail.to` 上的收件箱无需验证。Paperclip 不注册域名或管理 DNS；请参阅 AgentMail 的 [自定义域名指南](https://docs.agentmail.to/custom-domains)。

AgentMail 在所有实例上均可用。您无需开启任何实验性设置。

## 连接 AgentMail

设置分为两个简短步骤：选择智能体，然后选择其地址。

1. 打开 **Connectors**（连接器），找到 **AgentMail**，并选择 **Add connection**（添加连接）。屏幕显示 **Give an agent an email address**（为智能体提供电子邮件地址）。
2. 在 **Agent**（智能体）步骤中，在 **Agent** 下选择智能体。
3. 提供 API 密钥。如果您或您的团队已保存了可用的 AgentMail 密钥，Paperclip 会在列表中提供；否则，选择 **Enter a new API key**（输入新的 API 密钥）并粘贴一个。Paperclip 将其作为机密存储，且从不向智能体显示。选择 **Continue**（继续）。
4. 在 **Email address**（电子邮件地址）步骤中，输入名称，并从旁边的下拉菜单中选择域名。域名默认为您第一个已验证的自定义域名，或者如果您没有自定义域名，则为 `agentmail.to`。Paperclip 会在您输入时检查地址：
   - *"This email address is already in use. Choose a different address."*（此电子邮件地址已被使用。请选择其他地址。）表示已有人使用该地址。请选择 **Try:**（尝试：）后的建议之一，或输入其他名称。
   - *"AgentMail confirms availability when you create the address."*（创建地址时 AgentMail 将确认可用性。）表示 Paperclip 尚无法确定。您仍可以继续；当地址创建时，AgentMail 拥有最终决定权。
5. 若要使用 AgentMail 账户中已存在的收件箱，请选择 **Use an existing inbox**（使用现有收件箱）并从列表中选取。已分配给其他智能体的收件箱会被标记为 **already assigned**（已分配）。
6. 选择 **Create email address**（创建电子邮件地址）（对于现有收件箱，则选择 **Connect email address**（连接电子邮件地址））。

当屏幕显示 **Your agent’s email is ready**（您的智能体电子邮件已就绪）时，智能体即可在该地址接收邮件。选择 **Done**（完成）返回 **Connectors**（连接器），或选择 **Email settings**（电子邮件设置）以打开收件箱设置。

新密钥初始状态可供公司内所有人员及所选智能体使用。如果您重用已保存的密钥，其现有访问权限将保持不变。您可以稍后在收件箱的 **Access**（访问）选项卡中调整访问权限。这些已保存账户的更改将应用于使用相同密钥的所有收件箱。

### 高级选项

电子邮件步骤中的 **Advanced options**（高级选项）包含大多数人可以保持默认的设置：

- **Set up a custom domain ↗**（设置自定义域名 ↗）链接到 AgentMail 的域名指南。
- **Receiving**（接收）选择 Paperclip 获取新邮件的方式：
  - **Live connection**（实时连接）（默认）—— Paperclip 保持与 AgentMail 的出站连接，并自动重连。无需公共 URL，因此可在防火墙后工作。
  - **Webhook**（网络钩子）—— AgentMail 向 Paperclip 发送数据。您的 API 密钥必须具有该收件箱的 webhook 创建、读取和删除权限，否则设置将失败，并提示您修复密钥或使用 **Live connection**（实时连接）。
- 一个提醒，指出不受限制的收件箱可以接收任何人的邮件，以及 **Review trust settings**（审查信任设置）以检查该智能体可以访问哪些任务和工具。

> **注意：** 如果拥有该收件箱的智能体运行在低信任级别，其收件箱在连接前需要工作边界和活动的沙箱环境。在 **Agent**（智能体）步骤中，Paperclip 会提示 *"This agent needs a work boundary before it can receive email."*（此智能体在接收电子邮件前需要工作边界。）—— 选择 **Configure work boundary**（配置工作边界）进行设置。

### 如果设置被中断

您可以离开并稍后返回。浏览器刷新会保留您输入的内容，但 API 密钥除外，因为浏览器从不保存 API 密钥。未完成的收件箱会在 **Connectors**（连接器）中显示 **Finish setup**（完成设置），该选项将从您停止的确切位置继续，包括在出错前已创建的地址。

如果地址已创建但连接未完成，电子邮件步骤将显示该地址及 **Finish connecting**（完成连接）。若要改用其他地址，请选择 **Choose a different address**（选择其他地址）。第一个收件箱仍保留在您的 AgentMail 账户中；Paperclip 绝不会静默创建替代收件箱。

要从 Paperclip 中移除未完成或已完成的收件箱，请在 **Connectors**（连接器）中使用 **Remove connection**（移除连接）。收件箱及其邮件仍保留在 AgentMail 中，任务历史仍保留在 Paperclip 中。

## 当智能体请求电子邮件地址时

您无需预先设置收件箱。当处理任务的智能体需要自己的电子邮件地址时，它会在任务上通过卡片向您请求，而不是提供设置链接或请求在聊天中粘贴密钥。

该卡片仅询问 API 密钥。选择已保存的密钥或输入新密钥，然后选择 **Connect AgentMail**（连接 AgentMail）。Paperclip 保存密钥，为该智能体创建地址（或连接仅限收件箱的密钥允许的那个收件箱），并仅在收件箱实际可用时才允许智能体继续。新密钥可供公司内所有人员及该智能体使用。如果密钥错误，**Change API key**（更改 API 密钥）允许您在创建任何地址之前尝试另一个；**Not now**（暂不）表示拒绝。

## 邮件如何转化为工作

| 阶段 | 发生的情况 |
| --- | --- |
| 消息到达 | 新线程为拥有该收件箱的智能体创建一个任务；该线程上的后续消息继续同一任务 |
| 智能体工作 | 常规任务工作。内部评论和智能体的最终响应保留在 Paperclip 中，从不发送电子邮件 |
| 智能体回复 | 只有明确的发送或回复才会离开 Paperclip。回复继续现有线程；新消息启动一个独立的子任务 |
| 投递 | 智能体可以检查其发送内容的投递状态 |

有两个值得注意的后果。发送不会关闭任务，因此智能体可以发送并继续工作。并且由于内部讨论从不离开 Paperclip，您可以审查线程而无需担心草稿发送给发件人。

## 选择访问权限

智能体只能使用分配给它的收件箱。针对智能体不拥有的收件箱上线程的请求将被拒绝，因此一个智能体不能通过此连接器读取另一个智能体的邮件。

连接自身的设置——拥有凭据的身份，以及 **Any agent**（任何智能体）或 **Just agents I pick**（仅我选择的智能体）——与其他任何连接器的工作方式相同。[连接器访问权限如何工作](access-model.md) 包含详细信息。请注意，决定智能体可以读取谁邮件的是收件箱分配，而非智能体列表。

<a id="two-sender-controls-on-two-different-sides"></a>
### 两个发件人控制项，位于两侧

限制谁可以通过电子邮件联系智能体需要同时使用以下两项，它们执行不同的功能：

| 控制项 | 位置 | 功能 |
| --- | --- | --- |
| **Allowlists and blocklists**（允许列表和阻止列表） | AgentMail | 决定哪些发件人的邮件能到达收件箱。AgentMail 为 **新消息和回复维护单独的列表** —— 请检查两者，否则限制可能会通过其中一个列表泄露 |
| **Allow unlinked people**（允许未关联人员） | Paperclip，在连接的 **Access**（访问）选项卡下的 **External identity access**（外部身份访问）中 | 决定来自 Paperclip 不识别的发件人的已到达邮件如何处理。关闭表示仅与 Paperclip 人员关联的发件人可以启动工作；开启表示他们作为受限访客运行，在隔离的工作区和沙箱中，无法批准、雇佣、支出或管理访问权限 |

Paperclip 不验证或读取 AgentMail 的列表，因此第一行确实是提供商需要正确配置的部分。但第二行属于您，这也是“任何知道地址的人都可以给它发邮件”不一定意味着“任何人都可以指挥智能体”的原因。在公开地址之前，请同时设置这两项。

AgentMail 自身的参考文档是 [allowlists and blocklists](https://docs.agentmail.to/knowledge-base/allowlists-blocklists)。

## 试用

在不发送任何出站邮件的情况下进行验证：

1. 打开连接并确认收件箱已列出且连接状态健康。
2. 从您控制且已加入允许列表的地址，向收件箱发送一条简短消息。
3. 预计几秒内会为拥有该收件箱的智能体创建一个新任务，您的消息作为初始上下文。

这确认了完整的接收路径——凭据、收件箱分配和路由——而无需 Paperclip 发送邮件。仅当您打算发送真实邮件且发往您控制的地址时，才扩展到回复。

> **注意：** 这是流程说明，而非记录的测试结果。它从您自己的账户向自己的收件箱发送一条消息；没有任何内容投递给第三方。

## 管理收件箱

从 **Connectors**（连接器）打开收件箱。每个收件箱都有独立的 **Settings**（设置）、**Access**（访问）、**Conversations**（对话）和 **Activity**（活动）选项卡。

- **Settings**（设置）以智能体的地址为首。点击它可复制，或选择 **View inbox**（查看收件箱）在 AgentMail 控制台中打开收件箱。下方，**Receiving email**（接收邮件）显示接收模式及 **Last mail check**（上次邮件检查），并带有 **Pause**（暂停）和 **Resume**（恢复）。**Reconnect inbox**（重新连接收件箱）允许您粘贴 **New API key**（新 API 密钥）或切换接收模式，而 **Disconnect inbox**（断开收件箱连接）会停止在 Paperclip 中接收邮件，同时收件箱仍保留在 AgentMail 中。
- **Access**（访问）保存账户的凭据和智能体控制。此处的更改将应用于使用相同 AgentMail 账户的所有收件箱。
- **Conversations**（对话）将每个邮件线程链接到其任务。
- **Activity**（活动）显示投递情况以及 Paperclip 发布的任何内容。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| *"This email address is already in use."*（此电子邮件地址已被使用。） | 其他人已拥有该地址 | 选择 **Try:**（尝试：）后的建议，或输入其他名称 |
| 密钥仅连接一个收件箱 | 它是仅限收件箱的密钥 | 选择已保存的账户密钥或输入一个以创建新地址，或选择 **Use the existing inbox instead**（改用现有收件箱） |
| 设置失败并询问 webhook 权限 | API 密钥无法管理此收件箱的 webhook | 在 AgentMail 中授予 webhook 创建、读取和删除权限，或选择 **Live connection**（实时连接） |
| 邮件到达 AgentMail 但未出现任务 | 接收未建立，或收件箱未分配给智能体 | 检查连接的健康状态以及收件箱是否有拥有它的智能体 |
| 无法创建自定义域名收件箱 | 域名未在 AgentMail 中验证 | 在 AgentMail 中验证域名，然后重试 |
| 智能体无法读取您可见的线程 | 该线程位于未分配给该智能体的收件箱上 | 将收件箱分配给该智能体，或将工作交给拥有该收件箱的智能体 |
| 收件箱已断开且无法恢复 | 已断开的收件箱不可重用 | 创建新的收件箱连接 |
| 地址已创建但设置停止 | 地址分配后发生提供商或网络错误 | 在 **Connectors**（连接器）中选择 **Finish setup**（完成设置）以恢复；不会重复创建 |

限制：一个收件箱，一个拥有它的智能体。Paperclip 不强制限制谁可以向收件箱写入。附件和线程历史来自 AgentMail，因此智能体能看到的内容即为 AgentMail 保留的内容。

## 相关指南

- [连接器访问权限如何工作](access-model.md)
- [验证连接器并修复损坏的连接器](verify-and-troubleshoot.md)
- [Gmail](gmail.md) —— 读取您自己的邮箱，而不是为智能体提供专属邮箱。
- [AgentMail 收件箱文档](https://docs.agentmail.to/inboxes)

## 来源

- [AgentMail 设置和内联流程](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/features/connections/AgentMailIntentSetup.tsx) —— 请求智能体的默认访问权限和已保存密钥设置。
- [电子邮件连接服务](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/server/src/services/email-channels.ts) —— 收件箱所有权和对话生命周期。
