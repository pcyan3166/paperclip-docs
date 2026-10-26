---
paperclip_version: v2026.1012.0
seo_title: 设置连接器操作权限
seo_description: 将每个连接器操作切换为“允许”、“先询问”或“关闭”，在提供商更改操作列表时刷新该列表，并了解新操作的处理方式。
---

# 设置操作权限

连接器暴露的每个操作在每条连接上都有三种设置之一。这是决定被允许的智能体实际能做什么的关卡。

## 三种设置

| 设置 | 作用 |
| --- | --- |
| **Allowed** | *“无需批准即可运行。”* |
| **Ask first** | *“每次调用必须经人工批准。”* 智能体等待；该调用变为审查请求。 |
| **Off** | *“智能体无法运行此操作。”* |

普通新连接上的活动操作默认起始状态为 **Allowed**，包括写入和破坏性操作。当希望采用更严格的策略时，请选择 **Ask first** 或 **Off**。提供商的授权同意仍是一个独立的限制：被允许的操作不能超过凭据的上游权限。

## 更改单个操作

1. 打开连接器并选择 **Permissions** 选项卡。
2. 使用 **Find an action** 过滤列表。
3. 选择该操作所在行的设置。

更改将在下一次调用时生效。无需重新连接，也无需重启。

## 一次性更改整个组

列表分为 **Read** 和 **Write** 组。每个组标题旁都有一个 **Set all** 菜单。选择 **Set all: Off**、**Set all: Ask first** 或 **Set all: Allowed** 可一次性更改该组中的所有操作。当组内操作具有不同设置时，菜单显示 **Mixed**。

这是锁定新连接的快速方法：将整个 **Write** 组设置为 **Ask first** 或 **Off**，然后根据需要单独开放各行中的操作。每个操作的设置始终保留在下方可用。

## 为什么新连接可以做出更改

连接服务时，Paperclip 会向提供商请求其读取和写入操作所需的权限；如果连接器提供只读版和读写版，设置将从能够做出更改的版本开始。这样可避免智能体日后因凭据无法完成任务而被阻止。连接时，只读选项仍可在 **Change** 下找到。

智能体实际可执行的操作仍在此处逐个决定。以写入权限连接不会将任何操作从 **Off** 或 **Ask first** 切换为 **Allowed**，重新连接或更改登录设置也永远不会这样做。

对于 API 密钥连接，密钥携带提供商赋予它的权限。Paperclip 无法为已存在的密钥添加权限，因此如果操作因缺乏访问权限而失败，请创建具有所需权限的密钥并重新连接。当提供商明确表示登录缺少权限时，操作将失败并提示：*“提供商未授予此操作所需的权限。请重新连接此连接并允许所需的读取和写入访问。”*

列表上方的摘要会回显当前状态——**Allowed for**、**Ask first for** 和 **Off for**——以便您无需滚动整个目录即可确认批量更改。

## 阅读风险分类

每个操作都带有 Paperclip 根据提供商的工具元数据和名称推导出的分类：

- **read** — 无已知变更。
- **write** — 创建或更改某些内容。
- **destructive** — 删除或以其他方式不可逆。

提供商注释具有权威性。如果服务器将工具标记为 `destructiveHint`，则无论其名称如何，它都是破坏性的。相反，当提供商未提供信息时，使用基于名称的推断：诸如 `delete`、`remove`、`destroy` 和 `unpublish` 等动词分类为破坏性；`create`、`update`、`write`、`send`、`publish`、`post`、`archive` 及类似词汇分类为写入。

某些提供商因通用规则会导致错误而获得特定提供商的处理：

- **PostHog** — 未注释的工具被视为 **write**，而非 read。其 `exec` 工具始终为 **destructive**。
- **Notion** — 名称不使用 create/update/delete 动词（move、duplicate、convert）的变更被明确枚举为写入。
- **Shopify** — `cancel-cart`、`cancel-checkout` 和 `complete-checkout` 始终为 **destructive**。
- **记忆服务提供商** — 经过审查的检索工具为读取，存储和更新为写入，遗忘或重置工具为破坏性。Supermemory 的 `add_memory` 也可以遗忘，因此整个操作为 **destructive**。

## 刷新操作列表

操作列表属于提供商。当他们添加、重命名或移除工具时，您的列表在重新读取之前是过时的。

在 **Permissions** 选项卡上选择 **Refresh actions**。API 等效操作为：

```http
GET  /api/tool-connections/{connectionId}/catalog
POST /api/tool-connections/{connectionId}/catalog/refresh
```

刷新报告会显示发现了多少个操作——*“Found 24 actions”*——以及如果有操作被保留，有多少需要您确认：*“3 new actions need your OK.”*

<a id="what-happens-to-a-brand-new-action"></a>
## 全新操作的处理方式

**这取决于连接的创建方式。** 在刷新生产连接之前，了解您处于哪种情况是值得的。

| 连接的设置方式 | 新发现或更改的操作 |
| --- | --- |
| **Managed — "Connect with Paperclip"** | 新或更改的操作通常在刷新时被隔离。启用安全默认值时，读取分类的操作除外。检查生成的操作列表 |
| **Your own credential or OAuth client** — 大多数连接器的普通目录设置 | 在发现时变为 **active**。随后受该连接已生效的操作策略管理，而非保留在单独的审查队列中 |
| **A custom MCP server you pasted a URL for** | 在发现时变为活动，如上所述 |
| **A Paperclip example connection** | 安全默认值豁免读取分类的操作；新或更改的写入和破坏性操作被保留 |

对于普通情况，向导在您完成设置时将应用程序的操作默认值投影到策略中，而不是使用审查队列作为访问状态。这意味着 **您不应假设刷新只能减少智能体可执行的操作。**

> **Warning:** 托管连接并不保证每个新操作都需要批准。刷新行为取决于连接的配置和操作的分类。审查刷新后的列表，并将不需要的操作设置为 **Off**。

无论保留审查是否适用，有两个细节值得注意：您已设置为 **Off** 的操作保持关闭状态，而不是重新出现以供审查；配置为安全默认值的连接豁免分类为 **read** 的操作，仅保留写入和破坏性操作。

## 说明不更改权限

**Agent instructions** 切换控制智能体提示中的指导，而非其对工具的访问权限。关闭它不会关闭操作，开启它也不会使 **Off** 操作可用。参见 [Saved connection instructions](connection-instructions.md)。

对于聚合器账户，操作列表属于保存的网关。导入的应用卡片不会为该上游应用创建单独的策略。

## 无法解除的限制

权限开关不能授予连接器从未被允许执行的操作。

- **Gmail** 永久禁止发送邮件、移至垃圾箱、标记为垃圾邮件以及修改标签。没有设置可以启用发送邮件。参见 [Gmail](gmail.md)。
- **Google Workspace** 连接器仅允许您连接的 capability group 中经过审查的读取和写入工具。列表之外的预览工具将被拒绝。
- 使用只读 capability group 创建的连接不会通过更改开关获得写入操作；您应改为使用写入组重新连接。

## Shell 例外

每个操作的设置通过 Paperclip 的工具网关管理工具调用。它们不管理智能体在其自身工作区 shell 中运行的命令。

当您更改绑定到智能体身份的 GitHub 连接上的权限时，Paperclip 会在此时说明：*“Shell Git and gh use this account for the run and are not constrained by per-tool Ask-first controls.”* 将此理解为仅限于 GitHub 以及 shell Git 和 `gh`——这并非表示其他连接器被沙箱化。

## 相关

- [连接器访问机制说明](access-model.md)
- [回复连接器审核请求](review-requests.md)
- [工具网关](../reference/api/tool-gateway.md)
- [执行策略](../guides/power/execution-policy.md)

## 来源

- [Connection setup](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/features/connections/ConnectionSetupFlow.tsx) — 单屏访问选择和设置行为。
- [Remote MCP setup](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/features/connections/remote-mcp/RemoteMcpConnectionSetup.tsx) — 特定提供商的连接控制。
