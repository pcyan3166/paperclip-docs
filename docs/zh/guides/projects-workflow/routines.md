---
paperclip_version: v2026.1012.0
seo_title: 心跳与例程：安排工作
seo_description: 了解定时心跳为何默认关闭，以及如何通过例程安排重复工作，避免智能体暂停、意外 token 费用和难以管理的仪表板。
---

# 心跳与例程

刚招聘几个智能体时，你可能很想给每个智能体设一个定时器：“每隔几分钟醒来看看有没有事情做。”这听起来很主动，实际上却很快会让侧边栏堆满已暂停的智能体、产生意外 token 账单，还得不断调整仪表板来保持安静。

Paperclip 采用不同的默认方式：智能体保持休眠，直到实际工作到来。本指南介绍驱动智能体的两种机制——**按间隔执行心跳**和**例程**——以及分别适用于什么情况。还会介绍 Routines UI，包括列表页、详情页中的触发器和变量设置，以及运行历史。

---

## 智能体开始运行的两种方式

智能体可以通过以下四种方式被唤醒：

| 来源 | 触发条件 |
|---|---|
| **Timer** | 心跳间隔到期（例如每 5 分钟一次，无论是否有工作）。 |
| **Assignment** | 为智能体分配任务，或其未关闭任务收到新评论。 |
| **On demand** | 你在 UI 中点击 “Wake now”，或其他智能体交给它工作。 |
| **Automation** | 例程按计划触发，并将一个问题交给智能体执行。 |

第一种方式——定时器——是唯一一种在没有实际事件发生时仍会唤醒智能体的方式。其他三种都由事件触发：智能体是**因为有工作要做**才运行，而不是**为了看看有没有工作**才运行。

---

## 为什么定时心跳需要主动启用

新智能体的定时心跳默认**关闭**。这是有意为之，而且通常应保持此设置。

启用了定时心跳的智能体会每隔 N 秒醒来一次，无论期间是否有变化。每次唤醒都会消耗 token、在收件箱中生成运行记录，并多出一项需要你阅读的内容。如果五个智能体都每五分钟运行一次，你离开一小时后回来，可能会看到六十条运行记录，其中大部分内容都是“没有工作，继续休眠”。

面对这些噪声，人们往往会开始暂停智能体——这也是为什么有人会在智能体页面寻找批量暂停按钮。但暂停是一种粗暴覆盖：它会停止所有唤醒，包括你真正需要的事件触发唤醒。更好的办法是从源头保持心跳关闭，避免产生噪声。

> **经验法则：**如果你经常暂停并恢复某个智能体，问题通常在于该智能体的心跳设置不合适，而不是你的工作流有问题。

---

## 智能体的心跳设置

打开任意智能体并找到 **Run Policy** 区域。你会看到：

- **Heartbeat on interval**——总开关，默认关闭。只有智能体需要轮询没有自身事件触发机制的来源时才开启（见下文“定时器适用的情况”）。
- **Run heartbeat every _N_ sec**——仅在开启定时间隔后显示。如果确实需要定时器，建议按分钟或小时设置，而不是按秒。

**Advanced Run Policy** 下还有：

- **Wake on demand**——保持开启。分配、评论和例程都依赖此选项来唤醒智能体。
- **Cooldown (sec)**——两次运行之间的最短间隔，适用于突发工作。
- **Max concurrent runs**——智能体可并行处理的任务数。

大多数智能体适合采用以下组合：**Heartbeat on interval = off**、**Wake on demand = on**。这样智能体会在仪表板中全天候（24/7）显示为 “active”，但只有实际收到工作时才会消耗资源。

---

## 例程：安排计划工作的正确方式

计划工作应该交给例程。例程是附带触发器的可复用任务定义。触发器触发时，例程会创建（或重新打开）一个任务；如果已将任务分配给智能体，该分配会唤醒智能体。

例程支持两类触发器：

- **Schedule**——使用你所在时区的 cron 表达式，例如“每个工作日上午 9 点”“每月 1 日”或“每周日午夜”。Paperclip 会计算下次运行时间，并按计划触发。
- **Webhook**——一个可从 Paperclip 外部调用的签名 URL。当其他系统需要启动工作时使用。

侧边栏中的 **Routines** 页面可查看例程。

### 创建例程

1. 打开 **Routines** 页面并点击 **New Routine**。
2. 输入标题和简短任务说明。
3. 添加触发器。计划任务请选择 **Schedule** 并输入 cron 表达式——默认值 `0 10 * * *` 表示按本地时区每天 10:00 触发。
4. 将例程分配给负责执行工作的智能体。
5. 保存并启用。

触发器触发后，Paperclip 会创建任务、分配给智能体，并立即通过分配事件唤醒智能体。运行会出现在收件箱中，并明确链接回触发该任务的例程，因此你可以追查“这个智能体为什么运行？”并找到具体原因。

### 应了解的例程策略

两项策略用于控制计划重叠或调度器离线时的行为：

- **Concurrency policy**——下一个触发点到达时，例程仍在运行怎么办。选项包括：合并为一个排队的后续运行、跳过新触发点，或始终排队。
- **Catch-up policy**——例程或调度器暂停期间错过触发点时怎么办。选项包括：跳过错过的时段，或按有上限的批次补跑。

多数情况下，默认值（活动期间合并，错过时跳过）最合适：不会堆积任务，重启后也不会突然涌入大量工作。

---

## Routines 列表

![Routines 列表](../../../user-guides/screenshots/light/routines/list.png)

从侧边栏的 **Routines** 进入例程页面。页面没有标签；页眉有 **View all runs** 和 **Create routine** 按钮，下方是例程列表及当前显示数量。

### 列表行和徽标

每行显示：

- **Title**——例程名称。例程未处于活动状态时，名称旁会显示灰色状态标签：`paused`、`draft`（未指定默认智能体）或 `archived`。
- **Project**——项目名称及颜色标记；未指定项目时显示 “No project”。
- **Default agent**——智能体图标和名称；草稿没有默认智能体时显示 “No default agent”。
- **Last run**——最近一次执行的本地化时间，以及其状态。尚未触发的行显示 “Never”。
- **Run now**——立即触发例程的按钮（见下文）。
- **Enabled toggle**——在 `active` 和 `paused` 之间切换例程。旁边的标签会显示 `On`、`Off`、`Draft` 或 `Archived`。已归档例程的开关会禁用。
- **More menu**——`⋯` 按钮提供 **Edit**、**Run now**、**Pause**/**Enable** 和 **Archive**/**Restore**。

点击行会打开详情页。按钮、开关和菜单不会导航离开列表。

### 文件夹、排序和分组

例程可以放在文件夹中。默认按文件夹分组；尚未归档到文件夹的例程位于 **Unfiled** 组。尚未创建文件夹时，页面会显示建议整理例程的横幅和 **Create your first folder** 按钮（可关闭），工具栏也会显示 **New folder** 按钮。

有了文件夹后，左侧文件夹栏可在 **All routines**、各个文件夹和 **Unfiled** 之间切换。打开文件夹菜单可执行 **Rename**、**Edit color** 或 **Delete**。删除文件夹会将例程移到 Unfiled，而不是删除例程。若要批量归档多个例程，点击 **Select**，勾选目标例程并选择 **Move to...**；完成后点击 **Done**。

列表上方有两个控件：

- **Sort**——按 **Updated**、**Created**、**Last run** 或 **Title** 排序。再次点击当前选项可切换升序/降序。
- **Group**——按 **Folder**（默认）、**Project**、**Agent** 或 **None** 分组。按 Project 分组时，没有项目的例程归到 “No project”；按 Agent 分组时，草稿归到 “Unassigned”。

分组标题可折叠；排序、分组和折叠状态会按公司保存，重新加载后仍会保留。

列表没有单独的文本搜索功能；例程数量较多时，主要通过文件夹、分组和可折叠分组导航。

### 从列表创建例程

点击 **Create routine** 会打开模态编辑器，填写创建例程所需的最低信息：

- **Title**——会自动调整大小的文本框。按 Enter 可移至说明；按 Tab 可依次切换负责人、项目和说明。
- **For / in**——用于选择默认**负责人智能体**和默认**项目**的内联选择器。创建时两者均为可选项。最近使用的负责人会排在列表顶部。
- **Instructions**——Markdown 编辑器。写在这里的内容会成为例程生成的每个任务的正文。
- **Advanced delivery settings**——默认折叠的区域，包含并发和补跑策略；每个下拉菜单旁都有通俗说明。

没有默认智能体的草稿例程仍然有效——可以保存，但会保持暂停，直到分配智能体。保存后，Paperclip 会使列表缓存失效并直接导航到新例程详情页，打开 Triggers 区域，以便立即添加计划或 webhook。

### 立即运行、暂停和归档

- **Run now** 会打开小型对话框，可填写运行时变量、仅为本次运行覆盖负责人或项目，并在例程使用执行工作区时选择工作区。提交后会将运行作为真实执行问题加入队列。
- **Pause** 会将例程状态切换为 `paused`。暂停的例程会忽略触发器，但保留原有配置。
- **Archive** 会将例程从默认列表移除，但不会删除。之后可以恢复已归档例程。

### 查看所有运行

查看执行情况时，点击页眉中的 **View all runs**。页面会跳转到 Audit 区域的 **Runs** 部分（`/activity/runs`），其中列出每次心跳运行的状态和生成的工作链接。要确认“今天早上的例程是否都运行了？”，这是最快的办法。

如果在 Experimental 设置中关闭 **Streamlined UI**，Routines 页面会改为显示 **Routines** 和 **Recent Runs** 标签。Recent Runs 是按例程创建的执行任务筛选的任务列表，并带有每几秒刷新的实时运行指示器。

---

## 例程详情页

![例程详情](../../../user-guides/screenshots/light/routines/detail.png)

打开例程后会进入详情页。页面顶部有一条窄页眉，左侧是**子侧栏（sub-sidebar）**，将例程分为不同部分，让你一次编辑一项，而无需滚动一张很长的表单。

### 页眉

页眉将常用控件放在同一行：

- **可编辑标题**——标题是内联文本框。按 Enter 可跳转到 Overview 区域；如果已经位于该区域，则会聚焦到说明。
- **Run 按钮**——打开与列表行相同的“立即运行”对话框，可用于手动触发任意例程。
- **Automation toggle**——大型开关。开启时状态为 `active`，关闭时为 `paused`。右侧标签会显示 `Active`、`Paused`、`Draft` 或 `Archived`，并按状态着色。没有默认智能体时尝试开启会被拒绝，并提示 “Default agent required”。
- **Plugin badge**——插件管理例程时，徽标会显示插件名称和资源键，提醒你不要手动编辑。

### 子侧栏：区块，而非标签

左侧子侧栏分组列出例程的各个部分；每部分都有自己的子路由（`/routines/:id/<section>`），因此可深度链接并使用浏览器后退按钮。旧版 `?tab=` 链接仍有效，会重定向到对应部分。

第一组是可编辑的模板内容：

- **Overview**——负责人行（“For [assignee] in [project]”）和 Markdown 指令编辑器。还会简要显示例程的 **Triggers**、**Next run**（仅 webhook 的例程会显示 “On webhook delivery”），最近五次运行，以及 **View all runs** 和 **View routine activity** 链接。
- **Triggers**——例程的所有触发器各自显示为卡片，并提供添加新触发器的编辑器。
- **Variables**——标题和指令中检测到的 `{{name}}` 占位符。
- **Delivery**——并发和补跑策略。
- **Secrets**——例程任务可引用的环境变量值。

**Operate** 分组将例程的日常操作集中在例程自身页面中，无需离开页面前往独立审计界面：

- **Runs**——例程生成的任务，可在此搜索并直接更新（见下文[运行历史](#run-history)）。
- **Activity**——按日期分组的例程、触发器和运行事件时间线，包括编辑、启用、暂停、触发器变更、webhook 投递和密钥轮换。

例程修订历史位于页眉的 **History** 按钮中，可恢复之前的修订；点击 **Back to overview** 可返回概览。

有未保存更改的区块旁会显示小型琥珀色圆点；有实时运行时，**Runs** 旁会显示蓝色脉冲圆点。在窄屏幕上，侧栏会折叠为顶部按组显示的单个区块选择器。

### 保存修改

每个可编辑区块都有自己的固定**保存栏（save bar）**；只有在内容变更后才会显示。区块处于未保存状态时，保存栏会从底部滑入，显示未保存更改数量（点击数量可查看具体字段），并提供 **Discard** 和 **Save changes** 按钮。按 `⌘S` / `Ctrl+S` 保存，按 `Esc` 打开确认放弃修改的对话框。如果其他人正在你编辑时修改例程，保存栏会切换为冲突界面，提供 **Reload latest** 或 **Overwrite anyway**。非所有者看到的是只读栏，而不是保存栏。

### 评论例程说明

现在可以像评论问题文档一样评论例程指令，并进行内联编辑。**Overview** 区域中的 Markdown 指令编辑器采用与问题文档相同的批注界面，因此说明不再是静态文字，而是你和队友可以共同讨论的文档。

指令编辑器上方会显示带对话气泡图标和数字的**评论数量标记（comment count chip）**。尚无评论时，悬停提示为 “Open comments on description”；存在讨论串后，提示为 “Open *N* unresolved comments on description”。点击该标记可在说明旁打开评论面板，再次点击则关闭。在宽屏上，面板会停靠在文档旁边，并在滚动时保持在顶部导航下方，避免讨论串被导航栏遮住。

要发表评论，先选择说明中的一段文字并针对该选区添加评论——面板打开时可按 `⌘⇧M` / `Ctrl+Shift+M`。评论会锚定到高亮的确切文字，因此讨论会始终关联到对应的指令部分。评论和回复会一次性加载；数量标记和面板读取同一数据源，因此新建或解决讨论串后，数量会自动更新，无需手动刷新。

只有说明处于干净、已保存状态时才能发表评论。如果 Overview 区域有未保存修改、正在保存，或例程存在保存冲突，“新建评论”操作会禁用并显示简短原因，例如 “Save the draft to anchor new comments” 或 “Resolve the document conflict before adding new comments.” 这是有意设计：评论必须锚定到已保存的修订，因此需要先保存编辑。

### 保存失败时

编辑说明和其他例程修改一样，都会执行保存；Paperclip 会显示失败原因，而不会吞掉错误。如果保存遇到冲突（例如你编辑期间其他人更改了例程），会出现 “Routine changed” 提示，要求重新加载最新修订，保存栏也会切换到冲突界面。其他保存错误会显示 “Failed to save routine” 提示，并附带底层原因；没有具体消息时使用 “Paperclip could not save the routine.”。无论哪种情况，在保存成功之前，编辑器中的修改都会保留。

### 触发器卡片和易读计划

在 **Triggers** 区域，每个触发器都是一张卡片，显示类型图标、标题和简短状态行。计划触发器的标题会将 cron 翻译成**通俗语言**，例如 “Every weekday at 09:00”、“Every day at 10:00” 或 “Every 15 minutes”。转换器不认识的表达式会显示标题 **Schedule**。状态行显示计划时区。其他类型的标题为 **Webhook**、**GitHub webhook** 或 **API trigger**，状态行则说明当前情况，例如 “Ready · Waiting for an event”、“Receiving events” 或 “Authentication failed · Check the key in your app”。暂停的触发器只显示 **Paused**。

每张卡片右侧会显示 **Edit** 按钮（**Edit schedule**、**Edit webhook** 或 **Edit api**）和删除触发器的垃圾桶图标。删除后会显示简短的 “removed” 提示及 **Undo** 按钮，以便撤销误操作。设置尚未完成的触发器会显示 **Resume setup**，而不是 Edit。

点击 **Edit** 会展开卡片（此时按钮变为 **Close**）：

- **Schedule triggers** 显示计划编辑器和 **Time zone** 字段，带有 **Save schedule** 和 **Cancel** 按钮。
- **Webhook triggers** 显示只读的 **Webhook URL** 字段，可复制；已签名 webhook 会提供 **Replace key** 按钮；另有 **Check connection** 按钮，用于等待应用发送事件。
- **API triggers** 只会说明该触发器通过 API 启动例程。

每张展开的卡片还会显示 **Pause trigger** 按钮；触发器已暂停时则显示 **Enable trigger**，这样无需移除触发器就可以暂时停止它。

<a id="run-history"></a>
### 运行历史

![运行历史](../../../user-guides/screenshots/light/routines/run-history.png)

**Runs** 区域使用 Paperclip 中熟悉的问题列表，显示该例程创建的所有任务。因此列表中包含常见的状态和负责人列、搜索框及视图控件，并且范围限定于当前例程。正在运行的任务会标记为实时状态。

由于它是真正的问题列表，你可以直接在这里管理运行任务：无需打开任务即可内联修改状态或负责人；更新生效后，例程运行数据会立即刷新。点击行可打开任务本身，完成后可通过面包屑返回 **Runs**。视图偏好按例程保存。

通过 **Run** 按钮触发例程后，Paperclip 会将你带到 **Runs**，以便观察新任务出现。

未创建任务的运行（例如并发策略或活动门控跳过的触发点）不会显示在此处。你仍可在 **Overview** 的最近运行中看到它们及其状态徽标；API 的 [List Runs](../../reference/api/routines.md#list-runs) 端点会返回包括 `failureReason` 在内的完整记录。

---

## Cron 选择器

![Cron 选择器](../../../user-guides/screenshots/light/routines/cron-picker.png)

添加或编辑计划触发器时，Paperclip 会使用 **ScheduleEditor** 取代原始 cron 输入框。编辑器是一个小型表单，包含预设下拉菜单，以及根据选项显示的后续字段：

- **Every minute**——没有额外字段，生成 `* * * * *`。
- **Every hour**——选择分钟，生成 `M * * * *`。
- **Every day**——选择小时和分钟，生成 `M H * * *`。
- **Weekdays**——选择小时和分钟，生成 `M H * * 1-5`。
- **Weekly**——选择星期、小时和分钟，生成 `M H * * D`。
- **Monthly**——选择每月日期、小时和分钟，生成 `M H D * *`。
- **Custom (cron)**——用于输入预设不支持的原始 cron 表达式。

界面中的小时使用 12 小时制并带 AM/PM 标记，底层 cron 则采用 24 小时制。预设模式下分钟按 5 分钟递增（0、5、10……），与调度器精度一致。为了便于阅读，星期按周一到周日排列；存储时仍遵循标准 cron 约定（0 = Sunday）。

编辑器支持双向转换：如果在自定义字段中粘贴的 cron 表达式与某个预设匹配，下一次渲染时表单会识别它并切换回相应预设。无法分类的表达式会继续显示为 “Custom”，不会丢失编辑内容。

控件下方会用一行通俗文字概括当前设置，例如 “Every weekday at 9:00 AM”。例程详情页触发器行也会显示同一说明。

### 时区处理

每个计划触发器都会保存明确的 **timezone**，创建时取自浏览器的 `Intl.DateTimeFormat().resolvedOptions().timeZone`。调度器会按该时区评估 cron 表达式，而不是按 UTC。因此夏令时转换期间也会正确处理：设为“欧洲/阿姆斯特丹工作日上午 9 点”的例程在夏令时转换前后都会于当地时间 9 点触发，而不会相差一小时。

之后需要更改时区时（例如你搬家了），点击触发器卡片上的 **Edit schedule**，更新 **Time zone** 字段并点击 **Save schedule**。卡片状态行始终显示该触发器保存的时区。

---

## Webhook 触发器

计划是启动例程的一种方式，另一种是 webhook。Webhook 触发器不会按时间触发，而是提供一个 URL，外部系统可以向其发送 POST 请求——例如 GitHub 事件、部署脚本或其他 SaaS 工具。收到调用后，例程就会启动。当工作应当**因为其他地方发生了某件事**而运行，而非在固定时间运行时，应选择 webhook。

添加 webhook 与添加计划的位置相同：打开例程，进入 **Triggers** 区域并添加触发器。简短向导会引导完成余下步骤。

### 选择计划或 webhook

第一步会询问 **“When should this routine run?”**，并提供两项选择：

- **On a schedule**——每天、工作日或每周运行一次，即上文介绍的 cron 方式。
- **When another app sends a webhook**——当 GitHub、其他应用或脚本中发生事件时运行。

选择 webhook 后，Paperclip 会询问**谁发送 webhook**：**Another app or script** 或 **GitHub**。你的选择只影响后续显示的设置说明；底层触发器相同。Paperclip 还会提醒：**公共服务需要可从公共网络访问的 HTTPS webhook URL**。详见[Webhook URL 是否可访问？](#is-your-webhook-url-reachable)。

### 复制 URL 和密钥

继续后，Paperclip 会立即创建 webhook，并显示需要复制到发送方系统中的两项内容：

- **Webhook URL**（GitHub 路径中标为 **Payload URL**）——外部系统发送 POST 请求的地址。
- **Secret key**（GitHub 路径中标为 **Secret**）——用于验证投递确实来自你的系统的凭据。

请立即复制两者。**密钥只在设置期间可见**——离开向导后会隐藏，字段会被 **Generate new key** 按钮取代。如果没有保存密钥，不要尝试找回旧值，直接生成新密钥即可（见[轮换密钥](#rotating-the-secret)）。

Paperclip 还会提供可直接复制的 **Agent instructions** 区块，其中包含 URL、密钥和逐步连接说明，方便你将设置工作交给智能体处理。

### 通用方式（其他应用或脚本）

自定义应用或脚本的设置步骤如下：

- 创建指向 **Webhook URL** 的 webhook，并使用 **POST** 方法。
- 请求正文必须是 **JSON 对象**，不能是数组或裸字符串；请求头使用 **`Content-Type: application/json`**。
- 使用应用支持的方式进行身份验证：
  - **Signing secret**——应用提供 “signing secret” 字段时，将 **Secret key** 粘贴到该字段。Paperclip 接受针对准确请求正文计算的 HMAC-SHA256 签名，签名可放在 **`X-Hub-Signature`** 或 **`X-Hub-Signature-256`** 请求头中，格式为 `sha256=<hex digest>`。大多数 SaaS 工具都采用此签名方式。
  - **Custom header**——应用允许设置自定义请求头时，添加名为 **`Authorization`** 的请求头，值为 **`Bearer`**、一个空格和密钥。
- 只订阅确实应该启动例程的事件。
- 每个事件都发送唯一的 **`Idempotency-Key`** 请求头，并在重试时重用相同值。这样即使设置测试被重试，也不会在 webhook 正式启用后重复启动例程。应用无法发送该请求头时，Paperclip 会将正文完全相同的签名重试视为同一事件。

应用发送的 JSON 会附加到例程创建的任务中，并放在明确标记的数据区块里，提醒智能体将其视为数据而非指令。过大的载荷会在任务描述中截短，但完整载荷仍会保存在例程运行记录中。

> 在启用此共享设置之前创建的 webhook 使用仅 Bearer 身份验证方式。它们会继续按原样工作，向导也会继续显示可直接粘贴的 **Authorization header value**。

### 示例：Fireflies 会议摘要

通用方式的一种常见用途是：Fireflies 生成会议摘要后，启动例程执行“阅读摘要，列出决策和负责人行动项”。不需要选择专用 Fireflies 触发器；它是使用 **Another app or script** 路径创建的普通 webhook：

1. 为例程的智能体授权 Fireflies 连接（参见 [Fireflies](../../connectors/fireflies.md)）。Webhook 只告诉智能体**哪场**会议已就绪；智能体会通过该连接读取会议内容。
2. 添加 webhook 触发器并选择 **Another app or script**。
3. 在 Fireflies 的 Webhooks V2 设置中粘贴 **Webhook URL**，并将 Paperclip 的 **Secret key** 粘贴到 Fireflies 的 **Signing Secret** 字段。这是例程自己的密钥，不是 Fireflies API 密钥。
4. 只订阅 `meeting.summarized` 事件（在 Fireflies 中标为 **Meeting Summarized**），保存后像往常一样检查连接并点击 **Finish setup**。

Fireflies 必须能够通过公共互联网访问 URL，因此下方的可访问性说明同样适用。

### GitHub 方式

选择 **GitHub** 后，Paperclip 会提供适用于 GitHub 的步骤：

1. 在仓库中打开 **Settings → Webhooks → Add webhook**。
2. 将 **Webhook URL** 粘贴到 **Payload URL**，并将 **Content type** 设为 **application/json**。
3. 将 **Secret** 粘贴到 GitHub 的 **Secret** 字段。GitHub 会在每个请求中加入 **`X-Hub-Signature-256`** 请求头进行签名，因此这一路径无需使用 Bearer 身份验证。
4. 选择要启动例程的事件，启用 webhook 并保存。

<a id="is-your-webhook-url-reachable"></a>
### Webhook URL 是否可访问？

只有发送应用能够访问 Paperclip 提供的 URL，webhook 才能工作。URL 看起来无法从外部访问时，向导和触发器保存设置中都会显示黄色警告，说明原因：

- **Other apps can’t reach this localhost URL**——地址指向 `localhost` 或 loopback 地址；对请求发送方而言，这表示“它自己的机器”。
- **This webhook URL appears to be private**——地址属于私有网络，或使用内部主机名。只有同一网络中的发送方可以访问；GitHub 等公共应用通常无法访问。
- **This Tailscale URL may not be public**——地址以 `.ts.net` 结尾。即使使用 HTTPS，Tailscale Serve 也只对 tailnet 内部开放；tailnet 外部的发送方需要 Tailscale Funnel 或其他公共 HTTPS 地址。如果此 URL 已启用 Funnel，可以继续。
- **Use HTTPS for webhooks from other apps**——URL 使用普通 HTTP。许多应用会拒绝此类地址，而且 HTTP 不会加密 webhook 凭据或载荷。
- **Check the webhook URL**——地址不是有效的 HTTP 或 HTTPS URL。

警告只是建议，不会阻止保存：同一台机器上的脚本等本地或私有网络用途仍可继续。公共发送方应使用可公开访问的 HTTPS 地址；横幅会链接到 [HTTPS 和公共访问](../../reference/deploy/https.md)设置说明。Paperclip 只能判断 URL 本身，无法检查 DNS、防火墙或 Funnel 状态，因此没有警告也不代表保证可访问。

### 完成设置以启用 webhook

新建 webhook 触发器后会处于 **setup-pending** 状态，这是有意设计的。待设置期间收到的任何投递**只用于测试连接**，不会启动例程或创建任务。这样可以先接好系统并确认连接正常，避免意外启动真实工作。

向导最后一步是 **Check connection**，用于验证连接：

- 从发送应用发送测试事件。在 GitHub 中打开 webhook 的 **Recent Deliveries** 并选择 **Redeliver**；其他应用可以点击 “send test” 按钮，或执行本来应该触发 webhook 的操作。
- 保持页面打开。收到签名投递且身份验证成功后，状态会变成 **Test event received · Connection working**——此时不会创建运行记录或任务。如果看到 **Event arrived, but the key was rejected**，请返回发送应用修正密钥后重新发送。
- 点击 **Finish setup** 启用 webhook。之后到达的真实事件才会启动例程。

需要注意：**测试事件不会重放。**完成设置不会重新触发你在测试期间发送的投递——只有 webhook 启用**之后**到达的事件才会启动例程。如果例程本身处于暂停状态，完成设置会保存 webhook，但触发器仍保持休眠，直到你启用例程的自动触发器。

### 签名模式和重放窗口

每个 webhook 触发器都具有**签名模式（signing mode）**，用于决定 Paperclip 如何验证传入请求；带时间戳的模式还会使用**重放窗口（replay window）**：

- **Signing mode**——`app_webhook`、`bearer`、`hmac_sha256`、`github_hmac` 或 `none`。**Another app or script** 路径创建 `app_webhook`：接受 Bearer token 或 HMAC-SHA256 正文签名。`github_hmac` 对应 GitHub 路径，`bearer` 是旧版仅 Bearer 模式。下拉菜单下方会简要说明触发端点如何验证每种请求。
- **Replay window (seconds)**——签名请求的时间戳最多可回溯多长时间；仅 `hmac_sha256` 时间戳模式使用。默认值为 300 秒。其他模式没有时间戳，因此不适用此设置。

<a id="rotating-the-secret"></a>
### 轮换密钥

密钥泄露或需要重新签发凭据时，点击触发器卡片上的 **Edit webhook**，再选择 **Replace key** 并确认 “Replace the webhook key?” 提示（向导中的 **Generate new key** 按钮效果相同）。轮换会生成新密钥并立即使旧密钥失效，但**会保留相同的 webhook URL**，因此你只需更新发送应用中的密钥，无需更改端点。新密钥只会在同一提示中显示一次，请立即复制。

### 启用、禁用和归档触发器

每个触发器都有独立的 **enabled** 状态，因此可以单独关闭某个触发器，而不影响例程中的其他触发器。也可以**归档（archive）**触发器，将其从活动触发器集合中移除，之后再恢复；或者选择 **Delete** 永久删除。一个例程可同时有多个触发器，例如每晚计划和 GitHub webhook，它们会分别独立触发。

---

<a id="variable-templates"></a>
## 变量模板

例程可以接受**变量**：在标题或说明中使用 `{{name}}` 占位符引用的命名输入。例程触发时，Paperclip 会将这些值插入生成的任务中，因此同一个例程可以生成内容不同的多次执行。

### 定义变量

变量会自动检测。标题或说明中的任意 `{{name}}` 占位符都会成为受跟踪变量，并使 **Variables** 面板显示在说明编辑器下方。每个变量都可以设置：

- **Type**——`text`、`textarea`、`number`、`boolean`、`select` 或 `date`。
- **Label**——在立即运行对话框中显示的易读名称。
- **Default value**——没有其他值时使用的默认值。
- **Options**——仅用于 `select` 的逗号分隔选项列表，会在运行对话框中显示为下拉菜单。

`date` 类型会在立即运行对话框中显示日期选择器。如果例程提示中需要引用具体日期（例如 `{{reportDate}}`），可使用此类型，每次运行时选择日期，而无需编辑例程正文。

编辑标题或正文中的占位符时，面板会同步更新：将 `{{name}}` 改为 `{{customer}}` 会重命名变量；删除最后一处引用时，该变量也会被移除。

### 运行时使用变量

触发器触发时，Paperclip 会填入当前默认值并创建任务。你可以通过列表的三点菜单、详情页页眉或 API 打开“立即运行”对话框；该对话框可为单次运行覆盖任意变量，而不会修改例程定义。也可以仅为此次运行覆盖负责人和项目。

标题和描述中的变量都会使用简单的 Mustache 风格插值传入执行问题。例程中的原始占位符语法会保留，因此清除覆盖值后，例程仍会继续使用默认值。

外部 webhook 调用方可以在请求正文中传入变量。每种签名模式都会先验证正文，之后使用验证过的载荷进行插值。这样无需编写胶水代码，就能实现“每当 GitHub 打开 PR 时，创建一个审查任务并在正文中加入 PR 标题”等自动化。

---

## 补跑行为

两项策略用于控制计划重叠或调度器离线时的行为：

- **Concurrency policy**——下一个触发点到达时例程仍在运行，应该怎么办。可以合并为一个排队的后续运行、跳过新触发点或始终排队。
- **Catch-up policy**——例程或调度器暂停期间错过的触发点如何处理。可以跳过错过的时间窗口，或按有上限的批次补跑。

多数情况下，默认设置（活动期间合并，错过时跳过）最合适：避免运行堆积，也避免重启后意外涌入大量工作。

如果 Paperclip 长时间停机后重启，而你设置了 `enqueue_missed_with_cap`，调度器最多会创建配置上限数量的补跑任务，然后丢弃其余任务。这是有意为之：设置上限可防止周末长时间中断后，周一早晨生成数百个重复任务。

---

## 定时器适用的情况

定时心跳并非错误，只是经常被过度使用。以下情况适合使用定时器：

- 智能体需要监视**没有 webhook 的外部系统**——例如 RSS 源、没有推送机制的邮件收件箱，或仅提供轮询接口的第三方 API。没有事件可以触发唤醒，因此定时触发本身就是事件。
- 智能体的工作确实是“**每隔 N 分钟检查并作出响应**”，而你尝试过例程后发现并不适用。

即使符合上述情况，也应尽量延长间隔。智能体每小时轮询一次通常不会比每分钟轮询一次差，却能节省 60 倍成本。

---

## 关闭心跳与暂停并不相同

这两种状态在仪表板上看起来相似，但行为大不相同：

|  | 关闭心跳 | 暂停 |
|---|---|---|
| 定时器触发 | 否 | 否 |
| 任务分配时唤醒 | **是** | 否 |
| 收到评论时唤醒 | **是** | 否 |
| 例程触发时唤醒 | **是** | 否 |
| 点击按需唤醒 | **是** | 否 |
| 仪表板显示为 “active” | 是 | 否 |

**关闭心跳**是安静且高效的默认设置——智能体随时待命，只是不再定时触发。**暂停**则是因具体原因采取的强制停止：例如智能体行为异常、你正在修改配置，或预算保护机制暂停了它。如果你经常通过暂停智能体来控制噪声，应关闭它们的心跳并恢复为 active。

---

## 健康的配置应该是什么样

对于在后台运行的小型 AI 公司，理想的稳定状态是：

- **大多数智能体**关闭心跳，保持 “active”；只有收到你的任务分配、队友委派的子任务或例程触发时才运行。
- 使用**少量例程**安排重复工作，例如晨间站会摘要、周一指标汇总、每周维护任务。每个例程对应一名智能体。
- **一两个专业智能体**可以启用心跳，但前提是它们必须轮询没有 webhook 的外部系统，而且轮询间隔应较长。
- 只在确有问题时使用**暂停**，不要把它当作控制运行量的手段。

这样的配置能让智能体页面从需要不断操作的驾驶舱，变成它本来应该有的样子：只有真实事件发生时才会提示你的安静状态面板。

---

## 准备就绪

心跳和例程决定智能体**何时**运行。一次设置好默认值，大多数“管理智能体”的压力就会消失。下一篇指南介绍 Skills——智能体唤醒后决定它**如何**运行的可复用指令集。

[技能 →](../org/skills.md)

---

<a id="appendix-the-heartbeat-protocol-for-agent-developers"></a>
## 附录——心跳协议（面向智能体开发者）

无论智能体因定时器、任务分配、评论、例程还是直接唤醒而运行，每次心跳都会执行相同协议。这是智能体与 Paperclip 之间的契约。

如果你正在编写智能体适配器或自定义智能体，请按顺序实现以下九个步骤。

### 1. 身份

获取自己的智能体记录：

```
GET /api/agents/me
```

响应包含智能体 ID、公司、角色、指挥链和预算。

### 2. 跟进审批

如果环境中设置了 `PAPERCLIP_APPROVAL_ID`，先处理该审批：

```
GET /api/approvals/{approvalId}
GET /api/approvals/{approvalId}/issues
```

如果审批已解决关联问题，就关闭这些问题；否则发表评论并说明它们仍然打开的原因。

### 3. 获取任务分配

```
GET /api/companies/{companyId}/issues?assigneeAgentId={yourId}&status=todo,in_progress,in_review,blocked
```

结果按优先级排序，这就是你的收件箱。

### 4. 选择工作

- 优先处理 `in_progress`，其次处理 `in_review`（仅在由该问题评论唤醒时），最后处理 `todo`。
- 除非你能够解除阻塞，否则跳过 `blocked`。
- 如果设置了 `PAPERCLIP_TASK_ID` 且任务分配给你，应优先处理该任务。
- 如果由评论唤醒（`PAPERCLIP_WAKE_REASON=issue_commented` 且存在 `PAPERCLIP_WAKE_COMMENT_ID`），先阅读该评论。@ 提及仅提供上下文，不会唤醒智能体。

### 5. 检出任务

开始任何工作前，先检出任务：

```
POST /api/issues/{issueId}/checkout
Headers: X-Paperclip-Run-Id: {runId}
{ "agentId": "{yourId}", "expectedStatuses": ["todo", "backlog", "blocked", "in_review"] }
```

如果任务已经归你所有，检出会以幂等方式成功。如果由其他智能体持有，会返回 `409 Conflict`——停止处理并选择其他任务。**绝不要重试 409。**

### 6. 理解上下文

```
GET /api/issues/{issueId}
GET /api/issues/{issueId}/comments
```

阅读祖先问题以理解任务缘由。如果由特定评论唤醒，找到该评论并将其视为直接触发原因。

### 7. 执行工作

使用你的工具和能力完成任务。

### 8. 更新状态

更改状态时始终在请求头中包含运行 ID：

```
PATCH /api/issues/{issueId}
Headers: X-Paperclip-Run-Id: {runId}
{ "status": "done", "comment": "What was done and why." }
```

如果任务受阻：

```
PATCH /api/issues/{issueId}
Headers: X-Paperclip-Run-Id: {runId}
{ "status": "blocked", "comment": "What is blocked, why, and who needs to unblock it." }
```

### 9. 按需委派

为下属创建子任务：

```
POST /api/companies/{companyId}/issues
{ "title": "...", "assigneeAgentId": "...", "parentId": "...", "goalId": "..." }
```

子任务必须始终设置 `parentId` 和 `goalId`。

### 关键规则

- **开始工作前必须先 checkout**——不要手动 PATCH 为 `in_progress`。
- **绝不要重试 409**——该任务属于其他人。
- **退出心跳前，必须为进行中的工作发表评论。**
- **子任务必须始终设置 parentId。**
- **绝不要取消跨团队任务**——应将其重新分配给你的经理。
- **遇到阻塞时升级处理**——使用你的指挥链。
