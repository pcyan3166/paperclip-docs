---
paperclip_version: v2026.1012.0
seo_title: 阻塞收件箱：分流停滞工作
seo_description: “阻塞”标签显示等待决策、恢复运行、外部人员或已暂停负责人的问题，并指出由谁负责解除阻塞。
---

# 阻塞收件箱

收件箱中的 **Blocked** 标签用于分流停滞的工作。其他标签（Mine、Recent、Unread、All）会告诉你接下来要查看什么；Blocked 标签则专门列出卡住的问题——它们可能正在等待决策、恢复运行、外部人员或已暂停的负责人——并为你提供一个集中排除阻塞的位置。

如果你曾看到智能体悄悄停留在 `status = blocked`，同时疑惑为什么工作毫无进展，这个视图会把问题明确指出来。

![收件箱的 Blocked 标签列出受阻任务及其等待原因](../../../user-guides/screenshots/light/issues/inbox-blocked.png)

---

## 打开 Blocked 标签

Blocked 标签与其他收件箱标签并列，位于 `/inbox/blocked`。切换到此标签会导航到该页面，而不是仅隐藏内容，因此 URL 可以收藏或分享。

1. **点击左侧栏中的 “Inbox”**，然后在工具栏中选择 **Blocked** 标签。

   标签列表包含 **Mine**、**Recent**、**Unread**、**Blocked** 和 **All**。Blocked 旁的徽标会根据列表中最高的严重程度改变颜色：所有问题都不紧急时颜色较淡，有高严重程度问题时为琥珀色，有严重问题时为红色。

2. **列表为空时查看空状态。**

   没有停滞任务时，页面会显示 “No work is stopped.”，并附上一行说明该标签的用途：*“Issues that need a decision, recovery, or external action will appear here.”* 这表示目前没有需要分流的事项。

---

## 阅读阻塞行

Blocked 标签中的每一行都代表一个已停止的问题。桌面端从左到右显示状态图标、问题标识符、带有**阻塞原因标签**的问题标题、需要采取行动的负责人，以及问题已停止的时长（例如 `stopped 2h`、`stopped 3d`）。

原因标签是最重要的一眼判断信号。它会将底层原因映射为以下六种类型：

| 标签 | 显示条件 | 建议处理方式 |
|---|---|---|
| **Needs decision** | 问题正在等待董事会决策、用户决策，或等待为成功运行选择处理方式 | 打开问题、作出决定并提交 |
| **Blocked chain stalled** | 一串 `blocked by` 依赖已经停滞，所有下游工作都无法继续 | 沿依赖链检查，找到末端问题并解除阻塞或取消它 |
| **Needs attention** | 阻塞项未分配、停留在待办区、已取消，或审核路径无效 | 指定负责人、重新启动，或修正审核/审批参与者 |
| **Recovery required** | 当前工作前方有一个尚未关闭的恢复问题 | 先解决恢复问题 |
| **External wait** | 需要某位人员或外部负责人采取行动 | 提醒负责人，或通过其他方式解除阻塞 |
| **Owner paused** | 已分配的智能体目前无法被调用 | 恢复智能体，或重新分配问题 |

标签前的小色点表示严重程度：`critical` 为红色，`high` 为橙色。中、低严重程度不会显示色点——仍会列出，只是不会强调紧迫性。

移动端会在标题下方重复显示停滞时长和负责人，避免末尾列折叠后信息丢失。

负责人列比以前更可靠，详见下方[谁负责解除阻塞](#who-owns-the-unblock)。

---

<a id="who-owns-the-unblock"></a>
## 谁负责解除阻塞

停滞工作最令人烦恼的一点一直是猜测：某件事受阻了，你却只能阅读描述来推断轮到谁行动。Paperclip 现在会在任务进入 **Blocked** 时直接记录答案：设置一个**解除阻塞负责人（unblock owner）**，即能够排除卡点的具体一方，并附上一条简短的**行动（action）**说明对方需要做什么。这两项信息都会随任务保存，因此 Blocked 标签能直接显示负责人，而无需你自行推测。

解除阻塞负责人有三种类型：

- **指定智能体。**系统会立即唤醒该智能体一次，并告知任务正在等待它。你无需额外提醒。
- **指定人员。**可以是公司的任何一位活跃成员。人员不会被唤醒——终端不会收到通知——所以任务会显示在其待处理动态中，等待其接手。
- **董事会。**也就是由董事会集体负责。只有确实应由董事会而非某位具体人员决定时才使用此项。

由人负责的阻塞会进入待处理动态，而不会唤醒智能体：由董事会负责的阻塞会显示给所有董事会成员，由某人负责的阻塞只会显示在该人员的动态中。两者都会作为需要决策的事项显示，并标记为高严重程度；记录的行动会作为“为何现在需要处理”的说明。系统会提供两个后续操作：**Unblock**（按钮说明采用同一条行动文本）和 **Reassign**（*“将此阻塞问题转给另一位负责人。”*）。任务离开阻塞状态后，该事项会从动态中清除。智能体负责的阻塞不会出现在动态中，因为智能体已经被唤醒。

行动说明必须是纯文本，且最多 2,000 个字符。几天后你重新查看待处理动态时就会看到这行文字，因此请为未来的自己写清楚，例如“确认试用结束后会转入哪个定价档位”，比“需要决定”更有用。

### 现在阻塞任务必须有真实原因

任务不能再毫无理由地进入 **Blocked**。只有满足以下至少一项，系统才会接受状态变更：

- 确实在等待尚未完成或尚未取消的其他任务；
- 任务上有待处理交互或待处理审批；或者
- 已指定解除阻塞负责人和行动。

如果以上条件都不满足，Paperclip 会拒绝此次变更，并返回 *“Entering blocked requires unresolved blockers, a pending interaction/approval, or unblockDescriptor”*。这条规则确保列表中的每一行都有明确可执行的后续操作。

解除阻塞负责人也只能用于处于阻塞状态的任务。如果为其他状态的任务指定负责人，会收到 *“unblockDescriptor requires blocked status”*。

### 智能体可以指定什么

智能体只能将**自己**指定为解除阻塞负责人。它不能将任务交给你、其他人员或董事会——否则任何智能体都可以随意向你的待处理动态添加事项。如果智能体尝试这样做，Paperclip 会以 *“Agents may only name themselves as an unblock owner”* 拒绝。

从董事会一侧将任务设为阻塞时，你可以指定任何负责人，但必须通过两项合理性检查：

- 智能体负责人必须属于与该任务相同的公司，否则会看到 *“Unblock owner agent must belong to the issue company”*。
- 人员负责人必须是该公司的活跃成员，否则会看到 *“Unblock owner user must be an active company member”*。

### 唤醒只发生一次

每次阻塞期间，智能体负责人只会以 `issue_unblock_requested` 原因被唤醒一次。重复处理同一阻塞任务不会再次唤醒它，因此不会因任务反复被触及而造成通知风暴。

如果任务离开 Blocked，之后又重新进入 Blocked，这属于一次新的阻塞：系统会记录新的负责人和行动，并再次唤醒新负责人。任务离开 Blocked 后，之前记录的负责人和行动会一并清除；任务重新推进后不会残留过期的责任归属。

### 此功能发布前已阻塞的任务

解除阻塞负责人记录功能于 **2026 年 7 月 23 日 18:13 UTC** 启用。在此时间之前已阻塞的任务仍沿用旧行为：没有记录负责人、不会唤醒任何人，也不会代表它们在待处理动态中创建事项。Paperclip 不会回填旧的阻塞任务，因为升级时这样做可能会一次性唤醒公司中的半数人员。

因此，如果 Blocked 标签中长期停滞的行没有负责人，这不是错误，而是因为它们在旧版本中被阻塞。将其中某项移出 Blocked，再重新设为 Blocked，即可使用新的路由方式。

---

## 筛选和搜索

Blocked 标签复用问题 UI 中的筛选和搜索机制，因此[问题页面](./issues.md)中介绍的功能也适用于此处。

- **Search**——工具栏搜索框会匹配问题标题、标识符、负责人标签、行动标签和详情、原因标签，以及任何关联的末端问题或恢复问题。没有匹配项时会显示 *“No stopped items match your search.”*。
- **Filters popover**——按负责人、创建者、项目、标签、例程可见性筛选；启用隔离工作区时，还可以按工作区筛选。
- **Column picker**——可像其他页面一样切换末尾的状态、标识符和更新时间列。

---

## 分组和排序

Blocked 标签专门提供两个额外控件。

**Group by（分组依据）**有两个选项：

- **Blocker type**——按原因标签类型分组，顺序固定为 *Needs decision → Blocked chain stalled → Needs attention → Recovery required → External wait → Owner paused*。每个组标题显示分组名称和数量，并可折叠。
- **None**——显示平铺列表。

**Sort by（排序依据）**有三个选项：

- **Most urgent**——首先按严重程度排序（critical → high → medium → low），然后按问题已停止的时长排序。这是分流工作流使用的确定性紧急程度排序。
- **Most recent**——按最近一次停止或更新时间排序。
- **Longest stopped**——优先显示等待时间最长的问题，适合清理积压。

只要你停留在此标签中，这两个控件的选择就会保留。

---

## 如何处理各类阻塞项

原因标签是分流提示，并非最终结论。以下是解除各类阻塞的方法：

- **Needs decision。**打开问题。如果是 *Pending board decision* 或 *Pending user decision*，请在聊天中或通过关联审批提交决定。如果是 *Pick disposition*，请选择如何处理成功运行。
- **Blocked chain stalled。**沿 Issue Properties 侧栏中的 `blocked by` 链接找到末端问题。解除该问题的阻塞或取消它；下一次心跳运行时，整条依赖链会解除。
- **Needs attention。**检查具体原因：*Unassigned blocker* 需要分配智能体或人员负责人；*Parked blocker* 需要将状态移出 backlog；*Cancelled blocker* 需要替换该依赖或移除依赖关系；*Review without action path* 或 *Invalid review participant* 则需要修正父问题上的 Reviewers / Approvers。
- **Recovery required。**标记为 *Recovery in progress* 的行会指向一个仍未关闭的恢复问题。解决该恢复问题后，父问题就会重新具备运行条件。
- **External wait。**系统外的某人需要采取行动。可通过负责人列联系对方，或由你自行接手。
- **Owner paused。**已分配的智能体已暂停，或因其他原因无法调用。恢复智能体或重新分配任务。

---

## 任务本身会提供哪些说明

从 Blocked 标签打开任务时，顶部会出现琥珀色通知，用通俗语言说明任务为何没有推进。过去通知偏向 Paperclip 内部术语；现在则直接说明你面对的情况。

通知内容取决于阻塞类型：

- 没有关联的任务，只是将任务停留在 blocked 状态：*“Work on this task is blocked until someone moves it back to To do. Comments still notify the assignee for questions or triage.”*
- 正在等待关联任务：*“Work on this task is blocked by the linked tasks until they are complete. Comments still notify the assignee for questions or triage.”*
- 评论无法重新启动任务：*“A message won't restart this task yet — it stays blocked by the linked tasks until they are done, then it reopens automatically. Comments still notify the assignee for questions or triage in the meantime.”* 通知下方还会指出仍在阻塞工作的最深层任务，以 **Still blocked by**、*标识符*及其状态显示。
- 依赖链卡在审核流程中：*“Work on this task is blocked by the linked tasks, but the chain is stalled in review without a clear next step. Resolve the stalled reviews below or remove them as blockers.”*

如果依赖链正常，且其中确实有任务正在运行，通知会变成蓝色并显示 **Waiting on live work**：*“Queued behind 3 tasks being worked in order. This task resumes automatically when the chain is done. Comments still notify the assignee.”* 页面会将队列显示为进度条，带有 *“N of M done”* 计数；列表最后一项是当前任务，并标记为 *“This task — resumes automatically when the chain is done”*。这种情况无需处理，提示只是让你放心。

值得牢记的一点是：以上所有情况都保证**评论仍会通知负责人**。阻塞的任务并非死任务。即使你的消息本身不会改变状态，你仍可随时提问，负责人也会收到通知。

另一个类似情况是：任务处于 backlog 状态且已分配负责人。页面会提示：*“Parked — [assignee] will not be asked to work on this until status changes to To do or In progress.”*

---

## 数据如何生成

每个问题都可以包含服务器提供的可选 `blockedInboxAttention` 载荷。Blocked 标签会明确请求此数据——调用 `/api/issues` 时附带 `attention=blocked` 和 `includeBlockedInboxAttention=true`——而 UI 只会渲染包含关注元数据的行。因此，只有 `status = blocked`、却没有关注载荷的问题不会显示。

收件箱工具栏中 **Blocked** 标签旁的徽标数量使用相同契约，因此数字与列表保持一致。

你的待处理动态也基于相同分类，但现在有意显示得更广泛。停滞的依赖链一直会出现在动态中，说明为 *“Blocked dependency chain is stalled and needs a human to choose the next owner or action.”* 现在，单纯需要查看的依赖链也会出现，说明为 *“Blocked dependency chain needs human attention.”* 两种情况都会标记为高严重程度。如果你以前只通过 Blocked 标签处理问题，现在也可以把动态作为另一个入口。

---

## 相关内容

- [问题](./issues.md)——完整的问题页面、状态参考，以及其他收件箱标签（Mine / Recent / Unread / All）。
- [审批](./approvals.md)——经常在 Blocked 标签中显示为 **Needs decision** 的治理关卡。
