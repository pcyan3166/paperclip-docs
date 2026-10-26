---
paperclip_version: v2026.1023.0
seo_title: 实验性功能
seo_description: 通过可选标记发布、并在真实使用中评估的实际功能。此处包含哪些功能，以及启用后意味着什么。
---

# 实验性功能

Paperclip 会通过可选标记发布一些功能，让用户在这些功能成为默认行为之前先行体验。它们都是实际可用的功能——今天就可以启用——但仍在真实使用中接受评估，因此被放在专门的 **Experimental**（实验性功能）设置页面，而不是默认对所有人开放。

本节为每项实验性功能单独提供页面，介绍其用途、启用方法和使用方式。

![Experimental 设置页面](../../user-guides/screenshots/light/settings/experimental.png)

## “实验性”是什么意思

Paperclip 中的实验性功能：

- **已经发布并可正常运行。** 这些不是空壳；每个标记都会控制一套完整的功能界面。
- **由功能开关控制。** 大多数标记默认关闭。Paperclip Runner 也是实验性功能，尽管在自托管实例上默认开启；Cloud 可以管理不同的值。
- **不提供兼容性保证。** 打开页面时，应用会明确说明：*“实验性功能可能随时出现故障。这些功能需要主动选择启用，不提供任何兼容性保证。它们可能在没有通知的情况下更改、失效或被移除。请勿依赖这些功能支持关键或生产工作流程。”*

更改开关前，请先阅读相应功能的页面：隐藏界面和禁用执行可能有不同影响。对于必须保持稳定的工作流程，请勿依赖实验性行为；升级后也请阅读发行说明。不同版本间，功能可能被重命名、重新设计、移入核心设置或退役。

## 启用功能

所有实验性标记都集中在一个页面，且作用于整个实例（即 Paperclip 实例中的每个公司都会受到影响）：

1. 打开 **Settings → Experimental**（设置 → 实验性功能）。
2. 找到功能卡片并打开开关。

**Experimental** 位于设置侧边栏靠近底部的位置。

设置会立即生效，无需重启或迁移。本节每项功能页面都会重复写明应查找的准确开关名称。

页面最多将开关分为三个分组：**Experimental features**（实验性功能）、**Paperclip Developer Mode**（Paperclip 开发者模式）和 **Legacy**（旧版功能），每组中的卡片按字母顺序排列，方便查找。如果开发者或旧版分组中的开关全部被隐藏，对应的分组标题也会消失。

## 找不到某个开关时

如果你使用的是托管版 Paperclip（例如托管云服务或其他人运行的共享服务器），运维人员可能会隐藏部分或全部开关，因为这些功能由他们代为管理。被隐藏的开关不会显示在页面上，但仍然具有设置值：隐藏控件只会保留其当前值，因此即使看不到卡片，功能仍可能处于开启状态。如果需要更改，请联系负责运行该实例的人员。运维人员可参阅[托管部署的运维控制](../administration/settings.md#operator-controls-for-hosted-deployments)，了解具体机制。

<a id="if-a-toggle-is-locked"></a>
## 开关被锁定时

在由 Paperclip Cloud 管理的实例上，部分功能的状态由 Cloud 决定。打开 Experimental 页面后，你会在这些功能名称旁看到一个小锁形标记 **Managed by Paperclip Cloud**（由 Paperclip Cloud 管理），并且开关会变成灰色。开关仍会显示功能是开启还是关闭，只是无法切换。点击也不会有反应。

这不是账户权限问题。Paperclip Cloud 会为实例所在的服务集群设置这些值，应用每次读取设置时都会重新应用，因此托管值不会发生偏移——你或其他人在实例内部进行的任何操作都无法改变托管值。如果需要开启或关闭其中某项功能，必须在 Paperclip Cloud 一侧进行更改，而不是在此页面操作。

只有部分功能由此方式管理。没有锁形标记的功能仍可按上述说明自行开启或关闭；两类功能会同时显示在同一个列表中，因此在任意版本中都可以通过标记辨别。

**自托管实例不受影响。** 如果你自行运行 Paperclip，就不会有托管配置或锁形标记，页面上的每张卡片也都可以编辑。决定是否托管的依据是环境变量 [`PAPERCLIP_MANAGED_CONFIG`](../reference/deploy/environment-variables.md#cloud-managed-instances)，且只有 Paperclip Cloud 环境才会设置此变量。

## 功能列表

下表遵循页面自身的分组和顺序。本节中有对应文档的功能，其名称链接到该文档。

### 实验性功能

| 功能 | 提供的内容 |
| --- | --- |
| [智能体聊天](agent-chat.md) | 每个智能体有一个持续对话。**Chat** 位于 **Work** 分组最前；智能体会澄清目标，将实际工作交给任务处理，并在任务完成后汇报。 |
| [Assistant 连接（MCP）](assistant-connections.md) | 通过 OAuth 将外部 Assistant 以你的用户身份连接，并分别授予读取、写入和配置权限。 |
| Beta skills（测试版技能） | 允许智能体固定使用 Paperclip 核心技能的测试版。关闭后，每个智能体都会恢复为默认的最新技能，但已保存的固定版本不会删除。 |
| [内置智能体](../reference/api/built-in-agents.md) | 由 Paperclip 管理的内置智能体界面：名册徽标、**Built-in agents** 标签页和设置控件。 |
| [案例](cases.md) | 由任务创建并持续迭代的持久工作成果。添加 **Cases** 标签页和智能体案例 API（[API 参考](../reference/api/cases.md)）。 |
| [聊天连接器](../connectors.md#chat-channels-experimental) | 将智能体连接到 Slack、Discord、Microsoft Teams 和 Telegram 对话。关闭后，现有聊天连接仍会继续运行。 |
| [合并收件箱和任务列表](task-chat.md#an-optional-combined-task-list) | 通过一个 **Tasks** 导航项访问收件箱和任务视图。 |
| [决策](../guides/day-to-day/decisions.md) | 在主侧边栏中显示 **Decisions** 项目，集中列出等待你输入的任务。 |
| [启用环境](environments.md) | 在设置中管理环境，并为项目和智能体分配环境，包括自定义沙盒镜像。 |
| [启用外部对象](external-objects.md) | 检测任务中的外部 URL，并显示被引用的拉取请求、工单及其他工作对象的实时状态。 |
| [启用隔离工作区](isolated-workspaces.md) | 执行工作区控件：为每次执行创建项目的独立副本，避免智能体相互干扰。启用后会显示 **Use Isolated Workspaces By Default** 卡片。 |
| Experimental fast responses（实验性快速响应） | 在公司设置中显示 **Connections**，并允许平台生成简短确认信息，让用户知道智能体已开始工作（[详情](../guides/day-to-day/costs.md#fast-responses)）。 |
| [实验性文件查看器](file-viewer.md) | 在任务详情页添加控件，便于浏览和预览任务工作区文件。 |
| First task: propose with a plan document（首次任务：通过计划文档提出方案） | 对于首次提出的单任务请求，幕僚长会撰写简短的计划文档和复选框卡片。只适用于启用此功能后创建的组织。 |
| [GitHub 审查机器人](../connectors/understanding-github-review-bots.md) | 让智能体使用各自的 App 身份连接到 GitHub，以便审查拉取请求并回复问题。此功能独立于聊天连接器；关闭后只会隐藏设置界面，现有机器人仍会运行。 |
| [记忆连接器](../connectors/memory-connectors.md) | 为智能体设置 Mem0、Zep、Supermemory、Cognee 或 Honcho。隐藏设置界面后，现有连接仍会继续运行。 |
| [OpenAI Dot](external-agents.md) | 将 OpenAI Dot 加为独立的智能体选项。需要启用 Assistant connections (MCP)，并使用支持身份验证且有公开 HTTPS URL 的实例。 |
| [Paperclip Runner](../reference/adapters/paperclip-runner.md) | 原生智能体执行功能，在自托管实例上默认开启，在 Cloud 上默认关闭。必须明确将智能体配置为使用此功能。 |
| [简化英语交互](../reference/skills/optional/content/simplified-english.md) | 智能体使用 ASD-STE100 简化技术英语撰写确认信息、问题和建议任务。 |
| [状态卡片](status-cards.md) | 由 Summarizer 持续更新的共享看板摘要。启用状态卡片时也会启用 Summaries。 |
| Streamlined UI（精简界面） | **默认开启。** 提供简化侧边栏、聚焦的任务详情布局和上下文导航。关闭后会恢复旧版外壳和导航。 |
| [Summaries（摘要）](../reference/api/summary-slots.md) | 在项目和工作区页面显示由 Summarizer 生成的状态栏。状态卡片依赖此功能。 |

### Paperclip 开发者模式

用于开发、测试和调试 Paperclip 本身的内部工具。

| 功能 | 提供的内容 |
| --- | --- |
| [空闲时自动重启开发服务器](auto-restart-dev-server.md) | `pnpm dev:once` 的所有本地执行完成后，重启已过时的服务器启动实例。 |
| Managed Environment Only（仅限托管环境） | 隐藏本地环境，并让所有智能体都在平台托管环境中运行。 |
| Paperclip Developer Mode（Paperclip 开发者模式） | 内部维护工具和可观察性链接，例如运行页面上的 Honeycomb 跟踪查询。 |
| Run tasks in this worktree（在此工作树中运行任务） | 仅当服务器在 Git 工作树中运行时显示。 |
| [服务器信息调试视图](server-info-debug-view.md) | 在账户抽屉中添加 “Server” 区域，显示服务器重启时间和当前运行的提交。 |
| Smoke Lab | 在 **Apps → Developer** 下添加 **Smoke Lab** 标签页和 **Integration smoke** 仪表板卡片。仅限私有部署。 |
| [任务计划分解](plan-decomposition-panel.md) | 在任务详情页显示已接受计划的分解历史。 |

### 旧版功能

*“这些功能即将移除。”*

| 功能 | 提供的内容 |
| --- | --- |
| [经典任务界面](task-chat.md) | 恢复先前的任务详情页。聊天式任务页面现在是默认界面。 |
| [目标侧边栏链接](../guides/projects-workflow/goals.md) | 恢复主侧边栏中的 **Goals** 项目。 |

### 不在设置页面上的功能

以下功能在本节有对应页面，但此版本的设置页面中没有它们的开关：

| 功能 | 状态 |
| --- | --- |
| [任务固定的 AI 路由](task-pinned-ai-routing.md) | 属于实验性功能且默认关闭，但 `enableAiConnectionRouters` 标记在 Experimental 页面没有卡片。需要安装路由器插件。 |
| [任务看护器](task-watchdogs.md) | **不再需要标记。** 所有任务都始终提供看护器控件。 |
| [自动创建恢复任务](auto-create-recovery-tasks.md) | **已退役**，对应开关、预览和运行操作也已移除。 |
| [Cloud Sync](cloud-sync.md) | **已退役**，包括对应开关。请使用[公司 Import/Export](../how-to/back-up-and-restore-a-company.md) 在实例之间迁移公司。 |

## 后续阅读

- [实例设置](../administration/settings.md) — 其他实例级设置页面，以及 Experimental 页面在其中的位置。
