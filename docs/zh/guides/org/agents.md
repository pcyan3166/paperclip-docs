---
paperclip_version: v2026.1023.0
seo_title: 智能体
seo_description: 查看、招聘、配置和管理智能体，检查运行、预算、密钥、权限及心跳行为。
---

# 智能体

智能体是构成 Paperclip 公司的 AI 员工，实际工作都由它们完成：CEO 制定战略，工程师交付代码，营销人员起草帖子。Paperclip 中的其他一切——任务、审批、技能和预算——都是为了协调并管理智能体的工作。

Paperclip 中的智能体会被唤醒、执行工作，然后重新休眠。它们不会持续运行，而是以称为“心跳”的短时任务分批执行。两次心跳之间，智能体处于休眠状态：不消耗预算、不在内存中保留上下文，也不会采取行动。具体事件会触发心跳，例如定时计划、任务指派、审查请求或手动调用；适配器会短暂启动智能体运行时，让它取得进展，随后运行时退出，适配器记录执行结果。

本指南涵盖 Paperclip 中完整的智能体功能：点击 **Agents** 后进入的列表页、新智能体招聘流程，以及智能体详情页中的各个部分。初次使用 Paperclip 时，建议从头读到尾；如果只想修改模型、指令文件或预算上限等某一项内容，可直接跳到对应章节。

## 智能体列表

智能体列表是入口。从侧栏打开 **Agents** 后会进入 `/agents/all`。

![智能体列表](../../../user-guides/screenshots/light/agents/list.png)

### 筛选标签

顶部标签用于限定列表范围：

- **All**：公司的全部智能体。
- **Active**：当前可运行的智能体，包括 `active`、`running` 和 `idle`。
- **Paused**：由人员手动暂停或被预算系统暂停的智能体。
- **Error**：最近一次心跳失败的智能体。
- **Built-in**：Paperclip 管理的内置智能体，仅在实例启用内置智能体时显示。

当前标签会反映在网址中（如 `/agents/active`、`/agents/paused`），可收藏筛选结果或分享直达链接。

### 已终止和待处理审批的智能体

已终止的智能体不会出现在列表中，仍在等待招聘审批的智能体也不会出现。待审批招聘会留在审批线程中，直到你作出决定；详见[审批](../day-to-day/approvals.md)。

### 列表视图和组织架构图视图

**New Agent** 旁边的两个图标可切换布局：

- **List view（列表视图）**：按排序显示的扁平列表，每个智能体占一行。
- **Org chart view（组织架构图视图）**：反映汇报关系的层级树（CEO 在顶端，直属下属依次展开）。

组织架构图现在位于此处，没有独立的 Org 页面；详见[组织结构](./org-structure.md)。

### 列表中的每一行

两种视图都呈现相同信息，只是布局不同：

- 左侧是智能体的插画角色；汇报链损坏时会显示琥珀色警告图标。
- **Name、role 和 title**，例如 “Ada” 下方显示 “Engineer - Backend Engineer”；名字旁的小皇冠标记[你的主要智能体](#你的主要智能体)。
- **Live run**：有运行正在执行时显示脉动蓝色 “Live” 标签，点击可直接打开实时记录。
- **Model 和 adapter**：显示运行模型及 AI 运行时，较宽屏幕上可见。
- **Last heartbeat**：例如 “4m ago”。
- **Status**：当前状态，如 `idle`、`running` 或 `paused`。

点击整行可打开详情页；点击实时标签会直接打开该运行。

### 从列表创建智能体

页面右上角的 **New Agent** 会打开下一节介绍的招聘流程。

> **注意：**如果公司尚无智能体，列表会显示空状态并提示你创建第一个。你创建的第一个智能体始终是 CEO；没有办法绕过这一规则，也无需尝试规避。原因见下一节。

## 招聘智能体

Paperclip 有两种添加智能体的方式：

1. 你（看板用户）在智能体列表点击 **New Agent**，直接创建智能体。
2. 现有智能体（通常是 CEO 或经理）提出招聘申请，该申请会进入你的审批队列。审批流程见[审批](../day-to-day/approvals.md)。

本节介绍你亲自点击 **New Agent** 的直接创建流程。

![“Meet your next agent”对话框，第 1 步：输入智能体名称](../../../user-guides/screenshots/light/agents/new.png)

### 第 1 步：名称

点击 **New Agent** 会打开标题为 **Meet your next agent** 的简短对话框，顶部进度条显示两步：**1. Name** 和 **2. Adapter**。在 **Agent name** 输入名称（例如 “Ada”），再点 **Choose adapter**。此步骤只需填写名称。

### 第 2 步：适配器

对话框切换为 **Choose an adapter**，让你选择智能体的工作方式。选择适配器卡片，即决定由哪种 AI 运行时提供能力（Claude Code、Codex、OpenCode、远程网关等）。若选择 Paperclip Runner，还会出现 **Runner** 下拉菜单，用于选择它驱动的运行时。

点击 **Configure agent** 继续，或点 **Back** 修改名称。适配器详情和选择建议见[智能体适配器](./agent-adapters.md)。

### 连接、配置并完成

对话框结束后会进入设置页面，顶部显示角色图标和名称，左侧列出剩余步骤：

1. **Connect**：仅需登录模型提供方的适配器（Claude、Codex、Grok）会显示此步骤。选择 **Environment** 并连接账号。
2. **Configure**：选择 **Runtime** 选项（模型和适配器专属字段，例如 API key）及智能体运行的 **Environment**。页面底部的 **Test your agent** 卡片会检查运行时和模型能否响应；完成设置前点击 **Run test**。
3. **Confirmation**：点击 **Finish setup**。Paperclip 创建智能体并显示 **Your agent is ready**，可选择 **Edit configuration** 或为新智能体分配第一个任务。

### Paperclip 会自动填写的设置

为简化流程，部分设置会自动确定，之后仍可在[Harness / Runtime](#harness-runtime)页面修改：

- **Role**：公司第一个智能体始终是 CEO，之后新建的默认角色为 `general`。
- **Reports to**：新智能体默认向 CEO 汇报；CEO 向你汇报。
- **Title**：留空。
- **Heartbeat on interval**：默认关闭。收到工作或点击 **Run now** 时才唤醒，不会按计时器运行。

设置流程没有预算字段。预算在 **Audit → Budgets** 中管理，见下方[预算](#预算)。技能也不会在创建时挑选：Paperclip 自己的运行时技能会自动附加；智能体创建后，可在其[技能](#技能)页添加公司技能。技能库详见[技能指南](./skills.md)。

### 新智能体开始工作前的设置检查

创建后不会立即领取工作。Paperclip 会先完成必要准备（包括由[插件](../../administration/plugins.md)管理的资源），再测试保存的运行时配置。设置期间分配的任务不会丢失，会留在队列中，智能体就绪后开始处理。

设置运行时，详情页标题下方的状态行会显示 **Preparing agent** 或 **Verifying agent**，并说明任务仍在队列中。暂停、恢复或终止操作进行时也会显示 **Pausing agent**、**Resuming agent**、**Terminating agent** 或 **Cleaning up agent**。步骤失败时徽标转红，错误旁出现 **Retry**；修复原因后点击 **Retry** 重跑该步骤。

> **提示：**若错误信息不足以定位问题，打开智能体的[Harness / Runtime](#harness-runtime)页并点 **Run test**，查看逐项检查结果。

### 智能体招聘智能体时的审批

你通过 **New Agent** 创建时通常会立即就绪；若组织要求审批新智能体，确认步骤会显示 **Agent submitted for approval**。智能体通过招聘 API 创建时，申请会进入审批队列，新智能体处于 `pending_approval`，直至你作出决定。提案摘要显示 **Name**、**Role**、**Title**、**Icon**、**Capabilities**、**Adapter** 和申请的 **Skills**；摘要不显示月预算和汇报对象，需打开 **See full request** 查看原始负载。可按其他审批一样选择 **Approve**、**Reject** 或 **Request revision**。详见[审批：审核招聘申请](../day-to-day/approvals.md#reviewing-a-hire-request)。

使用内置 `paperclip-create-agent` 技能起草招聘申请的智能体，会为新成员写简短的角色说明（负责什么、拥有哪部分工作），而非冗长操作手册；汇报线、能力和技能分别写入对应字段。你要求的公司专属指令会保留。若需要更详细 `AGENTS.md`，可明确要求，或招聘完成后到[指令](#指令)页补充。

### 新智能体的默认能力

标准智能体创建或招聘获批时，会获得以下权限键，以便协助设置队友，无需你逐项手动授权：

- **智能体和技能**：`agents:configure`、`agents:suggest-changes`、`skills:create`、`skills:suggest-changes`。
- **工具**：`tools:manage_connections`、`tools:manage_profiles`、`tools:view_audit`、`tools:use`、`tools:manage_runtime`。
- **审计和收件箱**：`audit:view_agent_actions`、`inbox:manage`（仅限该智能体负责用户的收件箱）。
- **任务**：`tasks:assign`、`tasks:assign_scope`（仅限该智能体汇报子树）、`tasks:manage_active_checkouts`。

招聘其他智能体由 `canCreateAgents` 设置控制，不依赖 `agents:create` 权限。默认**不会**授予 `environments:manage`、`tools:admin`、`users:invite`、`users:manage_permissions`、`pipelines:write` 或 `joins:approve`。

这些授权仍受以下边界保护：

- **责任用户仍是权限上限。** 授权不会允许智能体执行其代表的人本身无权执行的操作；审批门槛和受保护变更检查仍然生效。
- **低信任和内置智能体除外。** 使用[低信任审核预设](../../administration/trust-and-low-trust-review.md)的智能体及 Paperclip 管理的内置智能体保留较窄权限。
- **不更改既有智能体。** 默认授权只适用于新智能体；升级时不会回填，已移除的授权也不会恢复。

权限键说明见[角色与权限](../../administration/roles-and-permissions.md#the-permission-keys)。

<a id="the-agent-detail-page"></a>
## 智能体详情页

详情页显示智能体头像、角色和状态。头像、名称、行为指令等塑造智能体的“角色”；公司主要智能体（通常是 CEO）可能会显示专门标记。顶部横幅会提示待审批、预算暂停、适配器配置问题或其他需要处理的状态。

每个智能体详情页都使用相同的框架：

- **标题栏**：显示智能体角色图像、姓名、适配器和职称，并提供一组操作。
- **侧边菜单**：列出智能体的页面，并分为四组。
- **页面内容**：在右侧显示当前选中的页面。

| 分组 | 页面 |
|---|---|
| **Agent** | [Overview](#概览)、[Instructions](#指令)、[Skills](#技能) |
| **Runtime** | [Harness / Runtime](#harness-runtime)、[Secrets & variables](#机密工具与渠道)、[Tools](#机密工具与渠道)、[Channels](#机密工具与渠道) |
| **Governance** | [Permissions / Trust](#权限与信任)、[API Keys](#api-密钥)、[Revisions](#修订记录) |
| **Audit** | Activity、[Runs](#运行记录)、Costs、[Budgets](#预算) |

Audit 的四个入口不会打开智能体内部子页，而会跳至已按此智能体筛选的公司级 Audit 页面；运行历史、支出和预算都在那里。

顶部操作在各页面都可用：

- **Star**：收藏智能体，方便再次查找。
- **Set as my primary**：姓名下方的小链接，把此智能体设为个人默认；详见[你的主要智能体](#你的主要智能体)。
- **Assign Task**：打开新任务对话框并预选此智能体为负责人。
- **Run now**：立即启动一次心跳并跳转到新运行，适合验证指令修改。
- **Run with provider trace**：同样会启动运行，但额外记录该运行与模型提供方之间的确切流量。仅实例管理员或本地单用户设置可见；追踪记录 24 小时后过期。
- **Pause / Resume**：在 `paused` 和 `active` 之间切换。
- **Clear error**：智能体处于 `error` 时显示，将其恢复为 idle。
- **More actions (`⋯`)**：包含 Duplicate Agent、Copy Agent ID、Reset Sessions 和 Terminate。

> **危险：**Terminate 会永久关闭智能体。如果以后可能需要它，请使用 Pause。暂停、终止和删除的区别见[审批：看板覆盖权限](../day-to-day/approvals.md#board-override-powers)。

### 智能体的角色图像

每个智能体都有一个角色图像，从 17 种配色中选择，用于快速区分成员；图像会出现在智能体列表和组织架构图、任务线程与评论、活动流、收件箱、搜索结果等位置。

- 新建智能体时随机分配配色。设置期间会显示灰暗、困倦状态的预览；运行测试通过或智能体创建后会变亮并庆祝，预览所用角色就是之后保留的角色。
- 在角色图像推出前创建的智能体会按 ID 分配配色，因此外观稳定。
- 导出并重新导入公司包时，会保留每个智能体的角色图像。

详情页标题栏的角色会随状态反应：

| 智能体状态 | 角色姿势 |
|---|---|
| Running | 工作中 |
| Error | 困惑 |
| Paused、terminated 或 awaiting approval | 休息 |
| 其他情况 | 待机 |

角色会动画并转向跟随光标；系统启用减少动态效果时显示静态图。如果图像加载失败，Paperclip 会显示智能体姓名缩写。

> **注意：**目前应用中没有配色选择器。若要更换智能体角色，可通过 API 提交新的 `appearance` 值，见[Agents API：更新智能体](../../reference/api/agents.md#update-agent)。

<a id="your-primary-agent"></a>
### 你的主要智能体

同一公司有多个智能体时，你可以选择自己的 **primary agent**，作为个人常用入口。该选择按个人保存，队友可选择不同智能体，并会跨设备同步。

无需手动初始化：你在公司创建的第一个智能体会自动成为主要智能体。要更改时，打开另一智能体的页面，点姓名下方的 **Set as my primary**。已有主要智能体时，Paperclip 会显示 **Change your primary agent?**，并并排展示两个角色供确认。已终止或仍待审批的智能体不能设为主要智能体。

主要智能体有以下便利：

- 在详情页标题和智能体列表的姓名旁显示**皇冠**（悬停可见 **My primary agent**）。
- 在侧边栏的智能体列表中排在最前。
- 作为默认回退选项：新任务对话框会先选择最近使用的负责人；若没有最近选择，则选择主要智能体。没有最近会话时，打开 **Chat** 会直接进入与主要智能体的对话。

设为主要智能体时，Paperclip 还会让你成为该智能体的成员。若之后智能体被终止或你离开其成员列表，皇冠会消失；Paperclip 不会自动替你选择新成员，可随时重新设置。

### 页面顶部的横幅

智能体状态为 `pending_approval` 时，页面顶部会显示 **This agent is pending board approval and cannot be invoked yet.**，旁边有 **Approve agent**。

另有两种含义不同的琥珀色横幅：

- **Invalid org chain**：汇报关系链损坏（经理已终止、经理不存在或形成循环），会阻止工作；在修复前不能分配任务，横幅会说明修复方式。
- **Escalation path is paused**：智能体本身可以正常运行，但其 `reports_to` 链上某个上级处于 **paused**。这只是警告，不会阻止工作；智能体无法把升级请求交给已暂停的经理，升级会被拒绝。横幅会指出暂停的经理，并建议恢复该经理或更改汇报对象。

升级路径警告只会显示给自身可接收和运行工作的智能体；智能体自己暂停或终止时，暂不检查其升级路径。若汇报链本身无效，会优先显示 **Invalid org chain**。导入实例后尤其要留意：导入会默认暂停所有智能体，恢复工作者却忘记恢复经理，是常见的升级路径问题。

<a id="dashboard-tab"></a>
## 概览

**Overview** 是智能体主页，可用来了解“这个智能体是谁、最近做了什么”。

![包含 Latest Run、Identity、Harness / Runtime、Capabilities、Skills 和 Recent Tasks 卡片的智能体概览](../../../user-guides/screenshots/light/agents/dashboard.png)

### 最近一次运行

页面顶部显示当前运行（标记为 **Live Run**）或最近已结束的运行（标记为 **Latest Run**），包括短运行 ID、状态、处理的任务以及发生时间。点击可打开运行详情。

### 四张摘要卡

- **Identity**：状态、**Role**、**Title**、汇报对象（**Reports to**）和直属下属数量（**Direct reports**）。
- **Harness / Runtime**：**Adapter**、**Model**、当前 **Session** 和 **Last run**；点击 **Configure** 前往[Harness / Runtime](#harness-runtime)页。
- **Capabilities**：能力摘要；没有填写时显示 “No capability summary has been added.”
- **Skills**：为此智能体启用的技能；**Manage** 链接可打开[技能](#技能)页。

### 最近任务

列表最多显示该智能体最近处理的六个任务。**See All** 会打开按此智能体筛选的完整任务列表。运行或重新分配工作前，可用它快速了解上下文。

### 审计

页面底部的 **Activity**、**Runs**、**Costs** 和 **Budgets** 四个快捷入口会跳至按此智能体筛选的公司级 Audit 页面。支出图表和逐次运行成本都在该处；详见[成本与预算](../day-to-day/costs.md)。

<a id="instructions-tab"></a>
## 指令

**Instructions** 页面用于编辑智能体的角色说明和应保留的文件。对大多数智能体来说，这也是它自己的文件夹，可保存跨任务使用的笔记和工作材料。

![智能体 Instructions 页面，显示文件列表和 AGENTS.md 编辑器](../../../user-guides/screenshots/light/agents/instructions.png)

### 受管理和外部指令包

Claude Code、Codex、Cursor、Gemini 和 OpenCode 等本地适配器支持**指令包**，即属于智能体的一组文件。Paperclip 可以管理文件夹布局，也可指向磁盘上的现有目录：

- **Managed**：Paperclip 在自己的存储中为每个智能体维护文件夹。你可在此编辑，智能体也能添加文件；这是让文件跨任务保留的模式。
- **External**：提供磁盘上的 `rootPath`，Paperclip 在该位置读写。适合把指令保存在需要作为权威来源的代码仓库中。外部目录保持原有行为，不会作为智能体跨任务保留的个人文件夹。

页面顶部有模式切换和根路径字段；任何改动都会点亮页面底部的 **Save changes** 与 **Discard** 按钮。

### 入口文件

每个指令包都有一个**入口文件**，通常为 `AGENTS.md`，适配器会在每次运行时将它提供给智能体。文件夹中的其他文件可供智能体按需打开，但不会自动加载；入口文件可以链接到它们。入口文件不能删除。

<a id="agent-files-persist-across-tasks"></a>
### 智能体文件会跨任务保留

使用受管理指令包时，此页面的文件夹是智能体唯一、当前的主目录。运行开始时，Paperclip 会提供该目录的私有副本，并把 `AGENT_HOME` 环境变量指向它。智能体在那里读取指令，也可创建笔记、记忆、参考材料、图片或其他二进制文件和子目录。Paperclip 会在轮次边界保存已验证的变更：仍保持打开状态的原生 Codex 会话会跨轮次继续使用同一可写副本；其他会话则在提供方停止后回收。只有被智能体修改或删除的文件会同步回主目录，因此下个任务从已保存文件夹开始。

使用时请注意：

- **任务工作区彼此分离。** 智能体个人文件夹位于任务工作目录之外；交付物仍应放在任务工作区。个人文件夹用于智能体自己的工作材料，不属于任务的 Git 变更。
- **没有修订历史。** Paperclip 只保留当前文件。如果两个运行，或一次运行与浏览器中的手动编辑，同时修改同一文件，最后完成同步的一方会覆盖另一方，Paperclip 无法恢复被覆盖版本。未改动文件保持原样，不相关的新文件也会保留。临时运行副本会在所属会话停止时删除，不是档案。
- **防止旧副本覆盖你的新修改。** 若文件自你打开后已更改，保存会被拒绝且草稿会保留；重新加载后可重新应用修改。
- **普通文件可持久保存。** 支持文本和二进制文件，不支持符号链接和特殊文件。入口指令必须为有效 UTF-8，且不超过 1 MiB。
- **存储有上限。** 文件夹最多容纳 100,000 个文件和目录，每个文件最多 256 MiB，总量最多 2 GiB。超限不会暂停智能体，也不会让运行失败；运行会显示 **Agent storage warning**，该次运行对文件夹所作的更改都不会保存，下次运行从最后一次已保存的文件夹开始。删除或缩小文件后警告会消失。其他原因导致保存失败时会显示 **Agent file sync failed for this run**。
- **检查保存结果。** 任务成功不代表智能体文件改动已保存。依赖这些改动执行后续任务前，请查看运行详情中的文件同步警告或回执。
- **也要备份文件系统。** 智能体文件保存在实例存储中，而不是数据库里，备份必须同时涵盖两者。详见[在实例备份中保留智能体文件](../../how-to/back-up-and-restore-a-company.md#keep-agent-files-in-your-instance-backup)。

此功能推出前已存在的智能体会保留已有文件；第一次使用该文件夹时，Paperclip 会将其现有指令带入。

实现参考：[运行时文件指令](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/server/src/services/agent-instruction-working-copies.ts)和[文件存储上限与同步](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/server/src/services/agent-file-store.ts)。

**谁可以修改文件。** 智能体可以编辑自己的文件夹，但仅限其责任用户当前获准修改的范围。读取或编辑其他智能体的文件需要拥有配置该智能体的权限；仅仅属于同一家公司并不足够。使用[低信任](../../administration/trust-and-low-trust-review.md)预设的智能体不能在普通工作中修改自己的指令；唯一例外是用户直接在其 Agent Chat 中要求它这么做。详见[在自己的聊天中修改指令](../../administration/trust-and-low-trust-review.md#instruction-edits-from-your-own-chat)。

<a id="recommended-bundle-structure-agents-soul-heartbeat-tools"></a>
### 推荐的文件包结构：AGENTS / SOUL / HEARTBEAT / TOOLS

从简开始。Paperclip 运行时已经告知智能体如何在 Paperclip 中检出任务、评论、委派和提问；已安装技能与仓库指令也会提供详细流程。因此，新智能体的 `AGENTS.md` 只需说明它是谁、负责什么。不要把通用 Paperclip 操作手册复制进去，以免重复运行时已提供的内容，甚至相互矛盾。

当工作涉及复杂战略或运营事务时，可把额外说明拆分到多个文件，再由入口文件引用。常见约定是四个文件：

| 文件 | 作用 | 回答的问题 |
|---|---|---|
| `AGENTS.md` | **做什么**——角色、职责、负责范围及公司专属规则，也是入口文件。 | *“我的工作是什么？”* |
| `SOUL.md` | **你是谁**——角色气质、语气、决策理念及重视的事情；描述持久人格，不列任务。 | *“我该如何思考和表达？”* |
| `HEARTBEAT.md` | **如何执行**——有固定周期工作的角色专属清单。 | *“现在应按什么顺序做？”* |
| `TOOLS.md` | **可以使用什么**——智能体可访问的工具、API 和技能说明；通常从空文件开始，随着使用逐渐补充。 | *“我有哪些工具？”* |

拆分文件时，可在 `AGENTS.md` 末尾引用它们：

```markdown
## 参考资料

- `./HEARTBEAT.md` — 我的定期检查清单。
- `./SOUL.md` — 我的身份和行为方式。
- `./TOOLS.md` — 我的工具说明。
```

这样拆分的好处包括：

- **每个文件只对应一种修改原因。**只调整 `SOUL.md` 中的人格设定时，无需改动 `AGENTS.md` 中的角色职责；更新日常流程时，也无需重写人格设定。
- **入口文件保持简短。**智能体按需读取 `SOUL.md` 或 `HEARTBEAT.md`。
- **符合人们理解角色的方式。**岗位职责、人格、日常流程和工具分别存放，避免混为一谈。

#### 编写 `SOUL.md`

好的 `SOUL.md` 像简短宣言，而不是项目简历。先写战略姿态（智能体优先考虑什么、为什么），再写声音和语气（如何写作与表达）。建议控制在约 40 行简短内容以内；太长时，模型会不再把它当作身份，而会将其当成可争论的指令。

好的示例：

- *“默认采取行动。尽快交付，而不是犹豫，因为停滞通常比一次错误决策代价更高。”*
- *“表达直接。先讲重点，再补充上下文。不要把请求埋在后面。”*
- *“权衡时优先考虑学习速度和可逆性。双向门快速通过，单向门谨慎决策。”*

不好的示例：

- *“乐于助人且专业。”*（太泛，没有可执行约束）
- *“处理 P0 事故时，先看仪表板，然后……”*（这是流程，应放在 `HEARTBEAT.md` 或技能中）

可以这样思考：*如果为这个岗位招聘新人类员工，我们希望对方内化这种岗位的哪些思维方式？* 这些内容就属于 `SOUL.md`。

#### 编写 `HEARTBEAT.md`

`HEARTBEAT.md` 是短清单，适用于每次醒来都执行相同工作的智能体，例如每周指标摘要、内容日历检查、待审 PR 队列检查。只保留角色特有的步骤；无需重复说明如何检出任务、评论或结束运行，因为 Paperclip 每次运行都会提供这些指引。多个智能体都适用的流程通常应放进[技能](./skills.md)。

#### 编写 `TOOLS.md`

`TOOLS.md` 通常从简单占位文件开始，再逐步充实。你或智能体可在其中记录特定工具的使用特点、适配器注意事项，或预期会调用的自定义 API。由于智能体文件夹跨任务保留，智能体写下的说明下次仍然存在。无需提前填满，可把它作为持续维护的工作笔记。

#### 何时只用简单结构即可

大多数智能体只需一份简短角色说明 `AGENTS.md`。符合以下任一情况时再考虑增加文件：

- 智能体有重要的独特个性或语气（例如面向客户的岗位、管理层）。
- 角色确实有每次醒来都会重复的专属例程。
- 角色说明已超过一屏，并混杂职责、个性和流程。

可以先只使用一个文件，之后再把章节从 `AGENTS.md` 移到 `SOUL.md` 或 `HEARTBEAT.md`；这是正常的重构方式。

#### Paperclip 如何初始化这些文件

每个新智能体最初只有一份简短的 `AGENTS.md`：

- **CEO 角色**会获得一行角色说明，说明它负责公司战略、优先级、资源分配和团队协调。
- **其他角色**会获得一行占位说明，称其为 Paperclip 公司中的智能体；请将其替换为一两句角色介绍。
- 从[团队目录](./team-catalog.md)安装的智能体，以及由 `paperclip-create-agent` 技能起草的智能体，也会从简短的角色说明开始，而不是使用冗长的操作手册。

既有智能体保留原有文件，Paperclip 不会重写。可通过文件列表上的 **+** 按钮新增 `SOUL.md`、`HEARTBEAT.md`、`TOOLS.md` 或其他文件；除约定外，这些文件名没有特殊含义。入口文件由你的设置决定。

### 编辑文件

左栏显示智能体的文件和文件夹；点击文件后会在右侧 Markdown 编辑器打开。点文件列表顶部的 **+**（**Add agent file**）添加文件，可输入 `TOOLS.md` 等名称，也可输入含目录的路径。要删除文件，点击编辑器上方的 **Delete**（入口文件不提供删除）。目前重命名需先删除再创建。

编辑器支持不超过 1 MiB 的文本文件。二进制文件或超出限制的文件会显示下载链接。Markdown 编辑器支持拖放或粘贴图片；图片会上传到公司资源库，并插入 Markdown 图片链接。

### 不支持指令包的适配器

不支持文件系统指令包的适配器（例如 OpenClaw gateway 和部分远程提供方）会显示简化的指令编辑器。规则相同：保存即提交修改，智能体会在下次心跳时读取。

### 修改何时生效

点击页面底部的 **Save changes** 保存，或点 **Discard** 放弃。智能体在下一次运行时使用新指令；正在执行的运行不会被中断。运行期间智能体对自身文件夹所作的修改会在运行停止后保存；可在运行详情检查存储或同步警告。若要立即验证修改，保存后点标题栏的 **Run now**。

<a id="skills-tab"></a>
## 技能

此页面控制附加到当前智能体的公司技能，是共享[公司技能库](./skills.md)在单个智能体上的视图：技能在库中编写和导入，此处决定分配给哪些智能体。

![智能体 Skills 页面，开关分组为 Enabled on this agent 和 Available from the library](../../../user-guides/screenshots/light/agents/skills.png)

### 页面标题

页面顶部显示已启用数量（例如 “3 of 12 enabled”）、技能如何到达智能体的标签（**Kept in workspace**、**Applied on next run** 或 **Tracked only**，最后一种表示适配器不支持由 Paperclip 管理技能）以及保存状态。右侧有 **Search skills** 搜索框和跳转到公司技能库的 **Browse skills store** 按钮。

### 页面区块

技能按区块分组并显示数量：

1. **Enabled on this agent**：当前智能体使用的库技能；关闭行开关即可移除。
2. **Available from the library**：公司技能库中的其他技能；打开开关即可分配给此智能体。
3. **Automatic and detected skills (read-only)**：Paperclip 自动应用的技能，以及工作区磁盘上已存在、但并非通过 Paperclip 安装的技能。每项显示锁图标而不是开关，并给出来源和位置提示；仅有内容时出现，也可折叠。

### 每条技能行

每行显示图标、名称、类别标签、一句话摘要和来源。点击库技能可打开公司技能库中的详情页并编辑其来源。

### 自动保存

停止切换开关片刻后会自动保存，状态依次显示 **Saving soon…**、**Saving…** 和 **Saved**；无需单独点击保存。失败时会显示 **Couldn’t save**。

### 适配器无法管理技能时

部分适配器（例如 `openclaw_gateway`）不允许 Paperclip 管理技能。页面会显示原因横幅并禁用开关；请直接在适配器中管理技能。页面底部会注明当前适配器。

### 警告

若已启用的技能不再存在于公司技能库，Paperclip 会显示琥珀色提示，说明该技能“已启用但在组织库中缺失”，并提供 **Remove**。你可以重新导入技能，或点击 **Remove** 清除此关联。技能的作用、编写方法及精简上下文的建议见[技能指南](./skills.md)。

<a id="harness-runtime"></a>
<a id="configuration-tab"></a>
## Harness / Runtime

此页面用于修改智能体如何运行：姓名、汇报关系、适配器和模型、运行位置、环境变量及唤醒时机。这里不用于修改智能体**做什么**（请到[指令](#指令)），也不用于修改它**知道什么**（请到[技能](#技能)）。

![智能体 Harness / Runtime 页面，显示 Agent identity 和 Adapter 卡片](../../../user-guides/screenshots/light/agents/configuration.png)

### 页面卡片

从上到下包括：

- **Agent identity**：**Name**、**Title** 和 **Reports to**；修改汇报对象即可调整组织架构图位置。
- **Adapter**：适用于当前实例的 **Adapter type**、**Model** 以及适配器专属字段（如 **Thinking effort**）。切换适配器属于结构性变更，通常会把模型和选项重设为新适配器的安全默认值，应谨慎选择。部分适配器（如 OpenCode）要求模型采用特定格式；格式无效时表单会显示行内错误。
- **Configuration**：适配器专属设置；较少使用的 **Command** 和 **Extra args** 等选项收在 **Advanced** 中。
- **Environment**：智能体运行的位置；可沿用公司默认环境，也可指定环境。
- **Environment variables**：每次运行都会传给智能体运行时的值。
- **Run Policy**：控制智能体自行唤醒的时间，见下文。
- **Debugging**：**Capture raw provider traces** 开关，仅实例管理员可见。

适配器专属字段仅影响当前智能体，不会影响其他智能体。

### 运行策略

**Run Policy** 控制心跳：

- **Heartbeat on interval**：打开后按计时唤醒，再通过 **Run heartbeat every … sec** 设置间隔。此间隔是最短间隔而不是保证频率；分配、审批、任务评论等事件也可能让繁忙智能体更频繁运行。`@` 提及不会唤醒智能体，只提供上下文。关闭后，只有收到事件或你点 **Run now** 才会运行。
- **Advanced Run Policy**：展开可设置 **Timeout (sec)**、**Interrupt grace period (sec)**、**Wake on demand**、**Cooldown (sec)**、**Max concurrent runs** 和 **Continue after max-turn stop**。

新智能体默认关闭 **Heartbeat on interval**。

### 测试智能体

页面底部的 **Test your agent** 卡片会检查运行时和模型能否响应。点 **Run test**；成功时显示 **Connection successful**，失败时说明需要处理的问题。更改适配器、模型或环境后应重新测试。

### 保存更改

修改会自动跟踪。页面底部提示 **You have unsaved changes.**，并提供 **Discard** 和 **Save changes**。Discard 会恢复所有已改字段；保存后会在[修订记录](#修订记录)页新增条目。

## 机密、工具与渠道

智能体可访问的内容包括：

- **Secrets & variables（密钥与变量）**：通过 **Secret access** 选择该智能体可以使用的组织密钥；环境变量仍在[Harness / Runtime](#harness-runtime)中设置。不要将密钥写入普通指令或任务评论。
- **Tools（工具）**：查看该智能体可以使用的已连接应用和工具，以及其实际访问权限。只启用工作所需的工具，并确认适配器支持这些能力。
- **Channels（渠道）**：查看该智能体参与的聊天渠道，也就是它可接收消息的入口。

## 权限与信任

权限决定智能体可读取、修改、分配或批准的内容。应按角色授予最低必要权限，特别留意访问机密、管理智能体、编辑技能或作出公司级决策的权限。来源不明的技能和指令也会影响运行行为；高信任角色使用前应先审阅完整内容。

**Permissions / Trust** 页面设置智能体的信任级别及允许执行的操作：

- **Trust（信任）**：**Trust preset** 决定智能体可在多大范围内读取和操作 Paperclip 工作。详见[信任与低信任审核](../../administration/trust-and-low-trust-review.md)。
- **Permissions（权限）**：提供 **Can create new agents**、**Can create/import skills** 和 **Can assign tasks** 开关。CEO 以及可以创建智能体的成员会自动启用任务分配。

切换这些设置后会立即保存。

<a id="configuration-revisions"></a>
## 修订记录

每次保存智能体配置，Paperclip 都会记录一次修订。**Revisions** 页按时间倒序显示 **Configuration Revisions**；每条包括短 ID、变更时间、变更来源及被修改的设置。

点击任意条目的 **Restore** 可恢复到该配置；恢复操作本身会保存为新修订，因此也可撤销恢复。

> **提示：**变更导致智能体异常且来不及排查时，可先暂停智能体、恢复到最近一次已知正常修订、再恢复运行，之后再调查问题配置。

<a id="runs-tab"></a>
## 运行记录

运行历史不再位于智能体内部页面。点击侧栏 **Audit** 下的 **Runs**（或[概览](#概览)底部的 **Runs** 快捷入口）会进入公司级 Audit 页的 **Runs** 标签，并自动筛选此智能体。

![Audit 页面 Runs 标签，按一个智能体筛选并列出最近运行状态、触发方式和耗时](../../../user-guides/screenshots/light/agents/runs.png)

### 运行列表

列表上方有两个筛选器：

- **Agent**：选择一个智能体或 **All agents**。
- **Status**：选择一种运行状态或 **All statuses**。

**Clear filters** 会重置这两个筛选器。每条记录显示：

- 智能体姓名、短运行 ID 和状态标签。
- 一行运行摘要。
- 运行原因（Timer、Assignment、On-demand 或 Automation）、运行耗时，以及开始时间距今多久。

列表按开始时间倒序显示最近运行。

### 运行详情

点击运行打开详情。智能体页面左侧会显示其其他运行，方便在不离开页面的情况下逐个查看；移动端提供 **Back to runs** 返回链接。详情由多个区块组成：

- **Status & timing**：状态、开始和结束时间、经过时间、持续时长；仅实时运行显示 **Cancel**。
- **Retry and resume controls**：失败或超时的运行可点 **Retry**，在相同任务上下文中重新唤醒智能体。因进程丢失而失败（`errorCode = process_lost`）的运行可点 **Resume**，从相同位置继续。
- **Metrics**：输入/输出/缓存 token、总成本、提供方和模型。
- **Session continuity**：适配器使用的运行前后 session ID；若运行期间会话更换，会突出显示。
- **Invocation card**：适配器使用的确切命令、工作目录、提示词、上下文和环境。密钥会替换为 `***REDACTED***`，JWT 会做掩码处理。
- **Transcript**：智能体与模型之间的完整对话，工具调用内联显示。
- **Log viewer**：带时间戳的原始 stdout、stderr 和系统日志片段，适合转录不足以解释问题时查看。
- **Touched issues**：运行期间交互过的问题及其链接；**clear session for these tasks** 可在坏运行破坏会话状态后清理。
- **Claude login**（Claude Code 智能体）：若运行因 Claude Code 需要认证而失败，可点 **Login to Claude Code** 自动执行 `claude login`。

### 实时运行

运行执行期间，转录和日志查看器实时更新。标题栏显示动画状态、已运行时间计数器和取消按钮。滚动到底部的辅助功能会持续跟随最新输出；当你向上滚动查看历史时会停止跟随，避免页面被拉回底部。

### 取消、重试和恢复

- **Cancel**：中断正在运行或排队中的运行，状态变为 `cancelled`。
- **Retry**：使用相同任务上下文创建新运行；适用于可恢复故障，如临时网络错误或适配器异常。
- **Resume**：仅在适配器进程丢失导致失败时显示。Paperclip 会重新唤醒智能体并传入 `resumeFromRunId`，让它接续而不是重头开始。

<a id="budgets"></a>
<a id="budget-tab"></a>
## 预算

每个智能体都可以有自己的月度预算。预算位于公司级 Audit 页的 **Budgets** 标签；点击智能体侧栏 Audit 下的 **Budgets** 即可打开。

![Audit 页面 Budgets 标签，显示 Budget control plane 摘要和活动预算事件](../../../user-guides/screenshots/light/agents/budget.png)

### Budgets 标签中的内容

- **Budget control plane**：汇总 **Active incidents**、**Pending approvals**、**Paused agents** 和 **Paused projects** 数量。
- **Active incidents**：已达到警告阈值或硬上限的预算。遇到硬上限时可用 **Raise budget & resume** 一次提高预算并恢复，或点 **Keep paused** 继续暂停。
- **Agent budgets**：每个已配置预算的智能体一张卡片，显示上限、已花费和 **Remaining**。修改金额后点 **Update budget**；尚未设上限时点 **Set budget**。

Budgets 标签显示全公司的预算，不会自动筛选到单个智能体；请在 **Agent budgets** 中找到该智能体卡片。

> **注意：**只有设置了预算，智能体才会显示预算卡片。通过 API 招聘且带月预算的智能体会自动创建预算；应用内设置流程不会设置预算。要为其他智能体设置上限，请使用 CLI 的 `budget agent:update` 命令（见[成本与预算命令](../../reference/cli/cost.md)）或[设置智能体预算 API](../../reference/api/costs.md#set-agent-budget)。

### 各阈值的行为

- **低于警告阈值**：正常运行，按计划执行。
- **警告阈值与硬上限之间**：预算状态为 `warning`，智能体继续运行。应检查它是否在执行预期工作，避免在低价值工作上消耗预算。
- **达到或超过硬上限**：智能体会因 `pauseReason = "budget"` 暂停，直到提高上限并恢复，或月度窗口重置。

达到硬上限时，可能会创建 **Budget Override** 审批，供你批准一次性增加或永久提高预算。

### 智能体级与公司级预算

智能体预算限制单个智能体；Paperclip 还会合计全公司所有智能体支出并应用公司级预算。两者如何联动及如何调优组合预算，详见[成本与预算](../day-to-day/costs.md)。

<a id="keys"></a>
## API 密钥

智能体在进程内运行时，会使用短期 JWT 认证 Paperclip API。若在受管理心跳之外运行（例如本地 CLI 操作者检查任务、CI 作业触发 webhook，或远程适配器需要长期令牌），则使用长期 **API key**。

在智能体侧栏 **Governance** 下打开 **API Keys**。

### 签发密钥

1. 在 **Create API Key** 中输入名称，例如 `production`、`ci` 或 `local-cli`。名称清晰可让之后审计更容易。
2. 点击 **Create**。
3. Paperclip 会在横幅中显示完整令牌：“API key created — copy it now, it will not be shown again.” 请立即复制；关闭横幅后 Paperclip 只保存哈希，无法再次显示令牌。

横幅提供显示/隐藏切换和复制到剪贴板按钮。完成后点 **Dismiss**。

### 活动密钥

所有未撤销密钥会列在 **Active Keys** 下，显示名称和创建日期，可用于轮换检查。

### 撤销密钥

点击活动密钥旁的 **Revoke**。密钥会移至页面底部的 **Revoked Keys** 区域，并立即被 Paperclip API 拒绝。撤销不可逆，撤销过的密钥不会重新使用。轮换时先创建新密钥，在所有使用位置替换，再撤销旧密钥。

### 密钥的传递方式

智能体 API key 通常在适配器环境或本地运行智能体时的 shell 中设置为 `PAPERCLIP_API_KEY`。Paperclip API 请求使用 `Authorization: Bearer $PAPERCLIP_API_KEY`。本地适配器在心跳内调用 API 时，Paperclip 会自动注入短期运行 JWT，无需长期密钥；远程适配器和手动 CLI 调用则需要长期密钥。

> **提示：**积极撤销不用的密钥。重新签发新密钥比担心遗忘的令牌是否仍在外流更简单。

## 心跳模型：智能体如何被调用

本节汇集理解智能体运行时行为所需的概念。心跳机制会解释为什么智能体列表和详情页中的运行状态如此呈现。

### 执行模型

每次心跳都遵循六个步骤：

1. **触发**：计划、任务分配、审查请求或手动调用唤醒智能体。`@` 提及不是触发器；要让其他智能体参与，应分配任务或请求审查。
2. **适配器调用**：Paperclip 调用智能体配置的适配器。
3. **智能体进程**：适配器启动智能体运行时，例如 Claude Code CLI。
4. **Paperclip API 调用**：智能体检查分配、领取任务、执行工作并更新状态。
5. **收集结果**：适配器收集输出、用量、成本和会话状态。
6. **记录运行**：Paperclip 保存运行结果，供审计和调试。

运行详情页会显示各步骤对应内容：触发原因显示为调用来源标签；适配器调用在 Invocation 卡片；智能体进程体现在转录和日志；API 调用可在 Touched issues 查看；结果收集填入指标区块；完整记录即此运行。

### 智能体身份

每次运行时，Paperclip 都会注入以下环境变量：

| 变量 | 说明 |
|---|---|
| `PAPERCLIP_AGENT_ID` | 智能体唯一 ID。 |
| `PAPERCLIP_COMPANY_ID` | 智能体所属公司。 |
| `PAPERCLIP_API_URL` | Paperclip API 基础 URL。 |
| `PAPERCLIP_API_KEY` | 用于 API 身份验证的短期 JWT。 |
| `PAPERCLIP_RUN_ID` | 当前心跳运行 ID。 |

触发来源特定时还会设置：

| 变量 | 说明 |
|---|---|
| `PAPERCLIP_TASK_ID` | 触发本次唤醒的问题。 |
| `PAPERCLIP_WAKE_REASON` | 唤醒原因，例如 `issue_assigned`、`issue_commented`。 |
| `PAPERCLIP_WAKE_COMMENT_ID` | 触发本次唤醒的特定评论。 |
| `PAPERCLIP_APPROVAL_ID` | 已解决的审批 ID。 |
| `PAPERCLIP_APPROVAL_STATUS` | 审批决定（`approved` 或 `rejected`）。 |

这些变量会出现在运行的 Invocation 卡片中。Paperclip 在展示前会删减可能是密钥、Bearer token、密码或 JWT 的敏感内容，但其余结构与智能体实际收到的相同。

### 加密身份

智能体的 **Identity** 面板显示持久的 Ed25519 指纹，并可复制其公钥 PEM。新建智能体时会生成密钥对；旧智能体会在下一次受支持的受管理运行时生成。生成前面板显示 **Not created yet**。

受管理运行时（包括受管理远程主机）会获得私钥，因此只选择你信任其持有此身份的主机。API Bearer 身份验证与加密身份彼此独立；导入公司或复制智能体时会生成新身份。运行时变量和备份要求见[智能体加密身份](agent-identity.md)。

### 会话持久化

智能体通过会话持久化跨心跳保留对话上下文。适配器会在每次运行后序列化会话状态（例如 Claude Code session ID），并在下次唤醒时恢复，因此智能体无需从头重读所有内容就能记住正在处理的工作。

运行详情会显示**运行前 session ID**与**运行后 session ID**。两者不同表示运行期间会话轮换，相同则表示继续同一对话。想让智能体下次从头开始时，可用标题栏 **More actions** 菜单中的 **Reset Sessions** 清除状态。

### 智能体状态

| 状态 | 含义 |
|---|---|
| `active` | 可以接收心跳。 |
| `idle` | 处于活动状态，但当前没有运行心跳。 |
| `running` | 心跳正在执行。 |
| `error` | 上次心跳失败。 |
| `paused` | 人工暂停或超出预算。 |
| `terminated` | 永久停用。 |

设置期间、暂停中或恢复中状态显示为 `paused`；终止和清理尚未完成时显示为 `terminated`。标题下的状态行会指出实际正在进行的步骤，详见[新智能体开始工作前的设置检查](#新智能体开始工作前的设置检查)。列表行状态点和详情标题徽标直接反映上述状态。列表的 **Active** 筛选会包含 `active`、`running` 和 `idle`，因为它们都表示智能体健康、正在工作或可工作；**Paused** 和 **Error** 则分别用于排查。

**关于 `paused` 的一项说明：**你仍可以把工作分配给暂停的智能体，例如先排队，稍后再恢复；界面会清楚显示暂停状态。其他智能体则不能这样做。智能体尝试分配给暂停对象时，Paperclip 会拒绝并提示：`Cannot assign work to a paused agent. Assign an invokable agent, leave the issue unassigned, or escalate to a board operator instead.` 否则智能体可能把升级请求交给暂停的经理，造成任务永远不运行。详见[委派 → 智能体之间委派的限制](./delegation.md#guardrails-on-agent-to-agent-delegation)。

### 对日常工作的影响

- 没有执行工作的智能体不会继续产生费用（最近一次运行的费用除外）；心跳间隔没有空转支出。
- 修改配置会在**下一次心跳**生效，而不是当前运行中途生效。运行期间编辑 Instructions，当前运行仍使用旧指令。
- 测试更改时可保存后点 **Run now**，实时查看运行，比等待定时器更快。
- 不确定是否保留某智能体时可以先暂停；暂停不会继续累积支出，历史也会保留。

## 常见工作流

以下是日常常见操作的端到端步骤。

### 快速试验新指令

1. 打开智能体并进入 **Instructions** 页面。
2. 编辑入口文件（通常是 `AGENTS.md`），点 **Save changes**。
3. 在标题栏点 **Run now**，Paperclip 会直接带你进入新运行。
4. 查看实时转录。行为符合预期即可结束；否则继续修改并重复。

这个循环刻意设计得很短：保存后下一次心跳使用新内容，手动运行可立即触发该心跳。

### 诊断行为异常的智能体

1. 在侧栏 **Audit** 下点 **Runs**，将 **Status** 设为 failed，打开最近一次失败运行。
2. 阅读转录，找出智能体开始困惑的位置。
3. 打开 **Invocation** 卡片，确认收到的环境、工作目录和指令路径正确。
4. 检查 **Touched issues**，确认它确实处理了预期任务。
5. 若会话状态异常，使用运行详情的 **clear session for these tasks**，或溢出菜单里的 **Reset Sessions**。
6. 若问题在配置，在 **Revisions** 页恢复最近一次已知正常配置。

### 暂时让智能体静默

1. 在标题栏点 **Pause**，状态变为 `paused`，之后不再启动心跳。
2. 按需调查；智能体历史会保留。
3. 准备好后点 **Resume**。

可放心使用暂停：它可逆、成本低；发现明显异常但暂时无暇排查时，先暂停通常最合适。

### 轮换 API 密钥

1. 打开侧栏 **Governance** 下的 **API Keys** 页面。
2. 点 **Create**，为新密钥命名（例如 `production-2026-04`），并复制黄色横幅中显示的令牌。
3. 在使用旧令牌的系统中替换为新令牌。
4. 返回 **API Keys** 页面，点旧密钥旁边的 **Revoke**。
5. 确认旧密钥移至页面底部 **Revoked Keys** 区域。

### 提高触及上限的智能体预算

1. 在智能体侧栏 **Audit** 下点 **Budgets**。
2. 若智能体已触及硬上限，在 **Active incidents** 中找到它并点 **Raise budget & resume**，一步提高预算并恢复。
3. 否则在 **Agent budgets** 找到对应卡片，修改金额并点 **Update budget**。
4. 若智能体仍暂停，在其页面标题栏点 **Resume**。若系统创建了 **Budget Override** 审批，按[审批](../day-to-day/approvals.md)流程处理。

## 相关指南

- [技能](./skills.md)：公司技能库如何运作、如何编写技能，以及如何保持智能体上下文精简。
- [智能体加密身份](agent-identity.md)：公钥、运行时信任和身份恢复。
- [智能体意见与建议](../../administration/agent-commentary.md)：默认反馈技能和原生工具，以及如何查看智能体反馈。
- [智能体适配器](./agent-adapters.md)：可用适配器、适用场景及配置方法。
- [审批](../day-to-day/approvals.md)：招聘申请、战略和预算覆盖的治理流程。
- [成本与预算](../day-to-day/costs.md)：API 支出计算、预算执行及全公司预算调优。
