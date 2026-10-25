---
paperclip_version: v2026.1005.0
seo_title: Gmail 连接器
seo_description: 允许智能体搜索和读取 Gmail，并可选择起草回复。无法发送。设置选项、访问范围、读取测试及修复方法。
---

# Gmail

> **警告：** **Google 验证待定。** Paperclip 尚未完成 Google 应用验证。在授权过程中，您可能会看到“未验证应用”的警告。如果 Google 提供 **Advanced**（高级）选项以继续前往 Paperclip，在审查所请求的访问权限后，您可以选择继续。并非所有账户都提供此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果您无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：** 在验证待定期间，Google Workspace 连接器以及任何已保存的 Google 账户将从 **Connectors**（连接器）页面暂时隐藏。这仅将其从列表中隐藏。现有的 Google 连接将继续运行，其访问权限和权限保持不变，需要 Google 服务的智能体仍可在其任务中通过连接卡片向您请求该服务。

智能体可以搜索和读取您的 Gmail，在草稿连接中，会在您的草稿文件夹中留下草稿供您审查。通过此连接器无法发送。

> **警告：** Gmail 在授权前需要 Google Workspace 开发者预览版注册。Google 必须注册登录的 Workspace 电子邮件，并且——如果您使用自己的 OAuth 客户端——拥有该客户端的 Cloud 项目。请先申请：[Google Workspace Developer Preview](https://developers.google.com/workspace/preview)。

[Set up the Gmail connector](gmail-setup.md) 是逐步操作程序。本页解释您正在做出的选择以及连接可以达到的范围。

## 选择能力组

有一个决定比其他决定更重要，并且它在连接的生命周期内是固定的。

| 组 | 智能体可以做什么 | 请求的作用域 |
| --- | --- | --- |
| **仅读取** | 搜索和读取消息、线程、草稿和标签 | `gmail.readonly` |
| **读取并创建草稿** | 上述功能，外加为某人创建草稿以供审查和发送 | `gmail.readonly`, `gmail.compose` |

设置从 **读取并创建草稿** 开始；除非您有创建草稿的具体理由，否则在连接时通过 **Change**（更改）切换到 **仅读取**。若要稍后更改组，请创建具有所需组的第二个连接并移除第一个连接；组不是现有连接上的开关。

## 为什么无法发送

这一点值得精确陈述，因为凭据和执行是不同的事物。

Google 的 `gmail.compose` 作用域（草稿组请求的作用域）在 Google 端确实允许发送。Paperclip 不依赖作用域来防止发送。Google Workspace 连接仅限于经过审查的操作列表，列表之外的任何操作都是被关闭的，而不仅仅是未被允许。对于 Gmail，经过审查的写入列表恰好只有一个操作，即 `create-draft`。

实际效果是：没有任何操作设置、能力组或已批准的审查请求会产生已发送的消息，并且 Google 稍后添加到服务器上的类似发送的操作默认是关闭的，而不是新可用的。如果邮件必须发出，则由人员从 Gmail 发送草稿。

## 智能体实际可以做什么

| 操作 | 组 |
| --- | --- |
| `search-threads`, `list-threads`, `get-thread` | 两者 |
| `search-messages`, `get-message` | 两者 |
| `list-drafts`, `get-draft`, `list-labels` | 两者 |
| `create-draft` | 仅限 **读取并创建草稿** |

操作名称在匹配前会进行标准化，因此像 `google.gmail/get_message` 这样的提供商变体将解析为同一项。您连接的实时列表位于其 **Permissions**（权限）选项卡中，**Refresh actions**（刷新操作）会重新读取它。Google 添加的任何不在审查列表上的内容都会返回 **disabled**（已禁用）——该阻止措施是审查列表所执行的，无论连接是如何创建的，它都有效。

> **注意：** 这是 Paperclip 的审查列表，而非对 Google 服务器的承诺。Google 提供目录并可以更改它；审查列表是 Paperclip 将从中启用的内容。

## 选择访问权限

Gmail 是一个邮箱，因此身份问题具有非同寻常的重要性。

- **Just me**（仅我）通常是答案。凭据属于您，智能体仅在您是责任人的运行中使用它。
- **Organization identity**（组织身份）使一个邮箱对责任人在凭据的人类受众中的合格智能体可读。这适用于共享支持收件箱，而不适用于个人邮件。它需要连接管理器权限。

在 **Which agents can use this connection**（哪些智能体可以使用此连接）下，优先选择 **Just agents I pick**（仅我选择的智能体）。邮箱不是公司中每个智能体的良好默认值。

在 **Permissions**（权限）选项卡上，保持读取为 **Allowed**（允许）。在草稿连接中，将 `create-draft` 设置为 **Ask first**（先询问）——草稿会进入真实邮箱，且连接器自身的指导原则是草稿创建应经过批准。[How connector access works](access-model.md) 涵盖该模型；[Set action permissions](action-permissions.md) 是操作指南。

Google 端的控制仍然叠加适用：Workspace 管理员可以限制哪些第三方应用可以持有 Gmail 作用域，并且在您的 Google 账户中撤销授权将停止连接，无论 Paperclip 的设置如何。

## 尝试一下

以您打算使用的智能体身份运行一个您已知答案的读取操作：

```txt
Search my Gmail for the most recent message from our domain registrar and tell me its subject and date.
```

将主题和日期与 Gmail 进行比较。这一步确认了凭据、账户和智能体的权限，并且不会留下任何痕迹。

然后打开连接的活动记录，确认 Gmail 调用已记录在那里。看似合理的主题行本身并不能证明邮箱已被读取——活动记录才是区分检索到的消息和自信猜测的依据。

不要用草稿进行验证。草稿是写入操作，它会在真实邮箱中放入内容。

> **注意：** 这是示例任务，而非记录的测试结果。请使用您可以目视确认的查询。

## 故障排除和限制

| 问题 | 可能原因 | 修复 |
| --- | --- | --- |
| Google 在同意屏幕前拒绝 | 登录的 Workspace 账户，或自管理客户端上的 Cloud 项目未注册开发者预览版 | 完成注册，包括每个额外的测试者电子邮件，然后重试 |
| 未提供 **Connect with Paperclip**（使用 Paperclip 连接） | 实例未注册到 Paperclip Cloud，或 Cloud 未宣传 Gmail 配置文件 | 使用 **Use your own Google OAuth app**（使用您自己的 Google OAuth 应用），或询问管理员关于注册事宜 |
| `create-draft` 缺失 | 连接是使用 **仅读取** 创建的 | 创建具有草稿组的连接 |
| 缺少发送或移至回收站操作 | 预期行为——它在审查列表之外 | 无需修复。没有支持的发送路径 |
| 同意成功但连接显示 **Setup incomplete**（设置未完成） | 最后一步未完成 | 选择 **Finish setup**（完成设置） |
| **Needs attention**（需要关注）并带有重新连接提示 | Google 令牌已过期或授权已撤销 | 选择 **Reconnect**（重新连接） |

限制：一个连接只能到达一个邮箱。删除、移至回收站、标记为垃圾邮件和更改标签在两个组中均不可用。Google 的 Workspace MCP 服务器仍处于开发者预览版，因此请将接口视为可能发生变化。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/gmail/)

- [Set up the Gmail connector](gmail-setup.md)
- [Google Workspace Search](google-workspace-search.md) — 跨 Gmail、Drive、Calendar 和 Chat 的单一只读搜索。
- [How connector access works](access-model.md)
- [Use separate accounts for people and agents](separate-accounts.md)
- [Gmail API scopes](https://developers.google.com/workspace/gmail/api/auth/scopes) — Google 对每个作用域允许内容的定义。
