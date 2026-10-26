---
paperclip_version: v2026.1023.0
seo_title: 文档变更记录
seo_description: 按次记录文档新增、重写和扩充内容。产品版本更新请参阅 Paperclip 更新日志。
---

# 文档变更记录

本页记录**文档本身**每次新增、重写或扩充的内容，不是 Paperclip 产品更新日志。

文档跟随 Paperclip 的[日历版本](https://github.com/paperclipai/paperclip/releases)（`YYYY.MDD.P`）发布，因此每条记录都会标注文档更新至对应 Paperclip 版本。产品本身的功能和修复历史请参阅 [Paperclip 版本发布页](https://github.com/paperclipai/paperclip/releases)。更新安装请参阅[更新 Paperclip](../how-to/update-paperclip.md)。

<details class="accordion" open>
<summary>v2026.1023.0 文档 <span class="accordion-meta">2026 年 10 月 23 日</span></summary>
<div class="accordion-body">

本次发布聚焦于控制谁能看到什么以及费用如何计算。任务和项目可设为私有并共享给指定人员；费用视图将预估、未定价及订阅支出与 API 支出分开；公司可以选用小型决策模型快速回答是/否问题。连接器目录新增 14 个应用，代理现在也可以报告其工具和工作流的问题及建议。

**新增页面**

- [自定义模型提供方](../connectors/custom-model-providers.md)：通过自己的网关、代理环境中的模型服务器或 Amazon Bedrock 提供模型访问，并用于任何兼容代理。
- [决策模型 API](api/decision-models.md)：选择公司共用的连接，支付小型结构化模型请求的费用，并查看请求记录。
- [Cases](../experimental/cases.md)：实验性持久工作记录，用于保存跨越单个任务的工作（如博客文章或文档页面），包括字段、文档和关联任务。
- [外部代理：OpenAI Dot](../experimental/external-agents.md)：通过配对码邀请 ChatGPT 中的 Dot 成为代理；它保留自己的模型、应用和记忆。
- [代理加密身份](../guides/org/agent-identity.md)：每个代理都获得持久 Ed25519 密钥对，以便跨运行、改名和重启识别其签名。
- [代理投诉与建议](../administration/agent-commentary.md)：代理可报告哪些工具或工作流问题，以及在哪里查看报告。
- [运行时命令](cli/runtime.md)：使用 `paperclipai runtime setup` 准备主机，以便 Paperclip Runner 运行 Cursor 或 Pi。
- 新增连接器指南：[Bitly](../connectors/bitly.md)、[Calendly](../connectors/calendly.md)、[Context7](../connectors/context7.md)、[Enterpret](../connectors/enterpret.md)、[Exa](../connectors/exa.md)、[Firecrawl](../connectors/firecrawl.md)、[Gauge](../connectors/gauge.md)、[GSC Wizard](../connectors/gsc-wizard.md)、[Make](../connectors/make.md)、[并行搜索](../connectors/parallel-search.md)、[Superagent](../connectors/superagent.md)、[Tavily](../connectors/tavily.md)、[Telem.AI](../connectors/telem.md) 和 [Windsor.ai](../connectors/windsor-ai.md)。

**更新页面**

- [问题](../guides/day-to-day/issues.md)、[项目](../guides/projects-workflow/projects.md)、[问题 API](api/issues.md) 和[目标与项目 API](api/goals-and-projects.md)：私有任务和项目的可见范围、共享方法，以及隐私如何从父项或项目继承；还介绍等待 PR 审核，以及后续任务的 **Created from** 来源字段。
- [费用与预算](../guides/day-to-day/costs.md) 和[费用 API](api/costs.md)：预估、未定价和仍在统计中的支出，以及与 API 支出并列显示的订阅费用；还包括关联重复账户、概览页按用户拆分、提供方费用报告和发票，以及新增的决策视图。
- [角色与权限](../administration/roles-and-permissions.md)、[公司设置](../administration/company.md) 和[代理 API](api/agents.md)：Operator 现在默认拥有公司编辑权限；决策模型和快速响应可在公司设置中配置。
- [代理](../guides/org/agents.md)：新代理开始工作前通过的设置检查、个人主要代理和代理加密身份。
- [Slack](../connectors/slack.md)：自动设置和安装机器人、**Require at-mention** 设置，以及成员如何连接自己的 Slack 账户。
- [设置审核机器人](../connectors/github-review-bot-setup.md)、[了解审核机器人](../connectors/understanding-github-review-bots.md)、[GitHub 设置](../connectors/github-setup.md) 和 [GitHub](../connectors/github.md)：更简单的专用 GitHub App 配置。
- [Paperclip Runner](adapters/paperclip-runner.md)、[Cursor](adapters/cursor-local.md)、[Codex](adapters/codex.md)、[Pi](adapters/pi.md) 和[适配器概览](adapters/overview.md)：Cursor 和 Pi 在 Runner 上的资格条件、实验性 OpenAI Dot 提供方、Codex CLI 最低版本，以及 **Harness** 选择器。
- [助手连接](../experimental/assistant-connections.md) 和 [Assistant MCP API](api/assistant-mcp.md)：更多直接执行工作和配置的工具，以及 `paperclip:configure` scope。
- [工件](../guides/day-to-day/artifacts.md)：CSV 文件以表格打开；HTML 报告在受限沙箱中渲染后打开。
- [插件 SDK](plugins/sdk.md) 和[插件](../administration/plugins.md)：插件设置和状态的代理生命周期契约、休眠前排空空闲任务、工作区获取与同步诊断，以及服务器不可用时插件条目的行为。
- [Anthropic](../connectors/anthropic.md)、[OpenAI](../connectors/openai.md)、[OpenRouter](../connectors/openrouter.md) 和 [xAI](../connectors/xai.md)：在服务器主机上通过浏览器登录，以及从聊天恢复缺失的个人 AI 凭据。还新增了 [Linear](../connectors/linear.md) API key 登录、[Airtable](../connectors/airtable.md) token 登录和 [Hugging Face](../connectors/hugging-face.md) 公共 Hub（无需登录）连接方式。
- 其他更新包括：[连接器目录](../connectors.md)、[工作区](../guides/projects-workflow/workspaces.md)（可在界面中修复不可用分支）、[环境变量](deploy/environment-variables.md)、[数据库](deploy/database.md)、[技能参考](skills.md)、[任务聊天](../experimental/task-chat.md)、[设置](../administration/settings.md)、[实验功能概览](../experimental/overview.md)，以及连接器指南中的设置按钮文案。

</div>
</details>

<details class="accordion">
<summary>v2026.1012.0 文档 <span class="accordion-meta">2026 年 10 月 12 日</span></summary>
<div class="accordion-body">

本次发布让外部助手加入团队。Claude Code、Codex、OpenCode 等 MCP 客户端可代表用户登录 Paperclip，读取、创建和委派工作。连接还可以携带说明并自动传递给代理。升级前操作员应注意两项破坏性变更：工作区恢复锁使用了新协议；已存工具授权现在严格按记录执行。

**新增页面**

- [助手连接（MCP）](../experimental/assistant-connections.md)，以及 [Assistant MCP API](api/assistant-mcp.md) 和 [MCP 命令](cli/mcp.md) 参考：通过浏览器或设备码授权连接 Claude Code、Codex、OpenCode 或任意 MCP 客户端，选择是否允许写入，并可稍后撤销。
- [已保存的连接说明](../connectors/connection-instructions.md)：在连接上保存一次使用说明，之后每个可使用它的代理都会在每轮收到说明。
- [检查 AI 账户用量](../connectors/ai-usage.md)：通过 **Check usage** 操作查看已保存 OpenAI（Codex）、Claude、Grok 或 OpenRouter 账户的当前提供方限额。
- [任务固定的 AI 路由](../experimental/task-pinned-ai-routing.md)：实验性连接池可在新任务间轮换已保存 AI 账户，同时让各任务继续使用启动时的账户。
- [Neon](../connectors/neon.md)：通过 Neon 登录或 API key 连接无服务器 Postgres，可将其固定到一个项目并设为只读。

**更新页面**

- [更新 Paperclip](../how-to/update-paperclip.md)：说明恢复锁破坏性变更；升级或回滚前须停止所有旧服务器和工作进程，并清除旧锁目录；还说明本版本中会修改现有数据行的迁移。
- [工具网关 API](api/tool-gateway.md) 和[访问模型](../connectors/access-model.md)：保存的授权按其内容执行，未知选择器会拒绝访问；升级后请检查自定义授权。
- [HTTP 适配器](adapters/http.md)、[进程适配器](adapters/process.md) 和[创建适配器](adapters/creating-an-adapter.md)：自定义适配器如何通过 `connectionInstructions` 或 `PAPERCLIP_CONNECTION_INSTRUCTIONS_FILE` 获取连接说明。
- [代理](../guides/org/agents.md)、[代理 API](api/agents.md)、[委派](../guides/org/delegation.md) 和[团队目录](../guides/org/team-catalog.md)：新公司和新招聘代理的默认说明更短，以及新代理默认获得的配置授权。
- [技能](../guides/org/skills.md)、[技能参考](skills.md) 和[编写公司技能](../how-to/write-a-company-skill.md)：代理可通过 `update_skill` 工具更新公司技能，并使用版本检查和安全重试。
- [MCP 聚合器](../connectors/mcp-aggregators.md)、[Composio](../connectors/composio.md)、[Arcade](../connectors/arcade.md) 和[连接器目录](../connectors.md)：已连接的上游账户现在会与原生账户一起显示在一个 Apps 目录中；AgentMail 现在是默认连接，[Asana](../connectors/asana.md) 恢复了登录方式。
- [Agent Chat](../experimental/agent-chat.md)、[聊天式任务](../experimental/task-chat.md) 和[问题](../guides/day-to-day/issues.md)：Chat 成为主导航，可选合并收件箱和任务列表，重建的 **New task** 对话框，以及“模型容量已满”和“模型不可用”恢复卡片。
- [Claude Code](adapters/claude-code.md)、[Codex](adapters/codex.md)、[OpenCode](adapters/opencode.md)、[Paperclip Runner](adapters/paperclip-runner.md) 和[代理适配器](../guides/org/agent-adapters.md)：Claude Sonnet 5.5、GPT-6.1 Sol、更新的 Harness 固定版本、远程 Codex 支持范围，以及服务器重启后仍能继续的 Codex 运行。
- [Anthropic](../connectors/anthropic.md)、[OpenAI](../connectors/openai.md)、[xAI](../connectors/xai.md) 和 [OpenRouter](../connectors/openrouter.md)：可在代理设置中就地重新连接不可用账户，并检查用量。
- [信任与低信任审核](../administration/trust-and-low-trust-review.md)：低信任代理可在其范围内处理自指派工作、接受直接向其发送消息或指派的人员工作，并可在用户从其聊天提出要求时更新自己的 `AGENTS.md`。
- 其他更新：[设置](../administration/settings.md)、[实验功能概览](../experimental/overview.md)、[任务看门狗](../guides/projects-workflow/task-watchdogs.md)（新增 5 分钟和 15 分钟静默阈值）、[Docker](deploy/docker.md)、[沙箱提供方](adapters/sandbox-providers.md)（exe.dev **Source VM**）、[插件 SDK](plugins/sdk.md)（AI 连接路由）、[工件](../guides/day-to-day/artifacts.md)、[工作模式](../guides/day-to-day/work-modes.md)、记忆连接器指南，以及[验证与故障排查](../connectors/verify-and-troubleshoot.md)。

**截图与界面刷新**

所有截图都已使用 Paperclip v2026.1012.0 重新采集，包含浅色和深色模式。本版本有些界面发生移动，因此相关指南也同步更新：

- [费用与预算](../guides/day-to-day/costs.md)、[活动日志](../guides/day-to-day/activity-log.md)、[工作时间线](../guides/day-to-day/work-timeline.md) 和[设置月度预算](../how-to/set-monthly-budget.md)：费用、预算、运行和时间线现归入 **Audit**。代理预算可从 CLI 或 API 设置，项目预算则位于项目的 **Budget** 标签页。
- [代理](../guides/org/agents.md)、[你的第一个代理](../guides/getting-started/your-first-agent.md)、[查看代理工作](../guides/getting-started/watching-agents-work.md) 和[组织结构](../guides/org/org-structure.md)：代理页面侧边菜单（含 **Overview** 和 **Harness / Runtime**）、两步式 **New agent** 对话框、**Run now**，以及作为 Agents 页面视图的组织图。
- [项目](../guides/projects-workflow/projects.md)、[例行任务](../guides/projects-workflow/routines.md)、[问题](../guides/day-to-day/issues.md)、[审批](../guides/day-to-day/approvals.md)、[技能](../guides/org/skills.md)、[公司设置](../administration/company.md)、[设置](../administration/settings.md) 和[插件](../administration/plugins.md)：同步界面当前的标签名称、按钮和流程，包括即时加入邀请、Experimental 设置列表，以及各文档中的 **Settings → Experimental** 路径。
- 移除展示本版本已不存在界面的截图，包括 Auto-Create Recovery Tasks。 [Cloud Sync](../experimental/cloud-sync.md) 和 [Auto-Create Recovery Tasks](../experimental/auto-create-recovery-tasks.md) 现标记为已移除。

</div>
</details>

<details class="accordion">
<summary>v2026.1005.0 文档 <span class="accordion-meta">2026 年 10 月 5 日</span></summary>
<div class="accordion-body">

本次发布让代理能在任务之间保留更多内容：跨任务持久化的代理文件、从 GitHub 同步的公司技能、可观看并接管的实时浏览器，以及原生 Runner 中的 Grok Build。此外还有三项可能注意到的破坏性变更：@ 提及代理不再启动运行；键盘快捷键始终启用；连接器通过单屏完成配置。

**新增页面**

- [Browser Use Cloud](../connectors/browser-use-cloud.md)：为代理提供受管浏览器，可从任务浏览器标签跟踪操作、亲自接管页面，并查看每次会话费用。
- [记忆连接器](../connectors/memory-connectors.md)，以及 [Cognee](../connectors/cognee.md)、[Honcho](../connectors/honcho.md)、[Supermemory](../connectors/supermemory.md) 和 [Zep](../connectors/zep.md) 指南：介绍默认关闭的 **Memory connectors** 设置背后的五种实验性长期记忆提供方及其选择方法。
- [Agent Chat](../experimental/agent-chat.md)：每个代理有一个持续会话、独立侧边栏和首页；任务完成后可回报聊天，并可在会话中回复审批。
- [Paperclip Runner](adapters/paperclip-runner.md)：原生 Runner 支持的提供方（包括 Grok Build）、权限模式，以及现在默认启用并受治理的 `search_api` 和 `call_api` 工具。

**更新页面**

- [问题](../guides/day-to-day/issues.md)、[问题 API](api/issues.md) 和[代理](../guides/org/agents.md)：提及现在只提供上下文。要让其他代理参与任务，应指派给它或请求审核。还介绍了仅凭提示创建任务以及由代理命名任务。
- [代理](../guides/org/agents.md)、[代理 API](api/agents.md) 和[备份与恢复公司](../how-to/back-up-and-restore-a-company.md)：代理文件现在会跨任务持久保存，并与 Instructions 编辑器共用，同时说明文件位置和备份方法。
- [技能](../guides/org/skills.md)、[技能参考](skills.md) 和[编写公司技能](../how-to/write-a-company-skill.md)：新增 **Sources** 标签，可从 GitHub 仓库选取技能文件夹、按需刷新，并在刷新失败时保留最后一个可用版本。
- [你的第一个连接器](../connectors/first-connector.md) 及连接器指南：设置整合到一个屏幕，显示连接以谁的身份登录、哪些代理能用；点击 **Change** 可修改其中任一项。MCP 聚合器 [Zapier](../connectors/zapier.md)、[Arcade](../connectors/arcade.md)、[Composio](../connectors/composio.md)、[Executor](../connectors/executor.md) 不再需要实验性设置。
- [Slack](../connectors/slack.md) 和[配置 Slack 与 Discord 通知](../how-to/wire-slack-discord-notifications.md)：你在看板发布的回复会进入 Slack 线程；代理在普通任务和例行任务中也可使用 Slack 工具。
- [GitHub](../connectors/github.md)、[Linear](../connectors/linear.md)、[Hugging Face](../connectors/hugging-face.md) 和 Google 连接器指南：GitHub Code Review Bot 有了独立卡片；Linear 支持浏览器登录；修正 Hugging Face 权限范围。Google 指南说明连接器现在请求的权限范围更小，以及这些连接器暂时从 Connections 页面隐藏。
- [设置](../administration/settings.md)、[命令面板](../guides/day-to-day/command-palette.md) 和[实例管理 API](api/instance-admin.md)：快捷键始终启用；实例开关、个人偏好和 `/api/auth/preferences` 路由已移除。`PAPERCLIP_HIDDEN_SETTINGS` 现在也支持通配符和指定例外。
- [工件](../guides/day-to-day/artifacts.md)、[聊天式任务](../experimental/task-chat.md) 和[工作模式](../guides/day-to-day/work-modes.md)：新增带预览的丰富工件卡片、逐消息模型及推理力度选择器，以及可在任务标签页打开的 Markdown 和文本附件。
- [Grok](adapters/grok-local.md)、[Claude Code](adapters/claude-code.md)、[Codex](adapters/codex.md)、[OpenCode](adapters/opencode.md) 和其他适配器页面：Paperclip Runner 上的 Grok Build、更新的模型目录和 Harness 固定版本，以及 Codex ACP 引擎的 `CODEX_API_KEY`。
- [环境变量](deploy/environment-variables.md)：Runner API 工具和捕获限制、工作区快照超时，以及更正后的 `PAPERCLIP_CLOUD_CONNECTOR_*` 名称。
- 其他更新：[信任与低信任审核](../administration/trust-and-low-trust-review.md)、[成员与访问](../guides/org/members-and-access.md)、[例行任务](../guides/projects-workflow/routines.md)、[沙箱提供方](adapters/sandbox-providers.md)、[插件 SDK](plugins/sdk.md)、[连接意图 API](api/connection-intents.md)、[公司技能策略](api/company-skill-policy.md) 和 [`connections` 命令](cli/connections.md)。

</div>
</details>

<details class="accordion">
<summary>v2026.1001.0 文档 <span class="accordion-meta">2026 年 10 月 1 日</span></summary>
<div class="accordion-body">

本次发布带来代理角色头像、重建的 Slack 设置、安全的例行任务 webhook 和实验性 MCP 聚合器。还更改了两个默认值：编码代理现在以全自动模式运行；旧版 Composio 代理已退役。本条还汇总了 v2026.916.1 之后发布的连接器指南。

**新增页面**

- [通过 MCP 聚合器连接应用](../connectors/mcp-aggregators.md)：通过 Arcade、Composio、Executor 或 Zapier 使用多种应用；说明如何启用实验设置、选择提供方，以及 Paperclip 和聚合器各自管理的部分。
- v2026.916.1 后发布的连接器指南：[Arcade](../connectors/arcade.md)、[Composio](../connectors/composio.md)、[Executor](../connectors/executor.md)、[Fireflies](../connectors/fireflies.md)、[Railway](../connectors/railway.md) 和 [You.com](../connectors/youcom.md)。

**更新页面**

- [代理](../guides/org/agents.md) 和[代理 API](api/agents.md)：每个代理现在都有随其在应用中显示的插画角色；说明角色如何随代理状态改变、怎样通过 `appearance` 设置，以及头像图片路由的工作方式。
- [代理适配器](../guides/org/agent-adapters.md) 及 [Claude Code](adapters/claude-code.md)、[Codex](adapters/codex.md)、[Gemini CLI](adapters/gemini-cli.md)、[OpenCode](adapters/opencode.md)、[Kimi](adapters/kimi-local.md)、[Grok](adapters/grok-local.md) 适配器页面：编码代理默认以全自动模式启动，并说明如何调低自动化程度。`engine: auto` 现在会报告设置错误，而不再静默回退到 CLI；Codex 默认不活动超时时间为 30 分钟。
- [Slack](../connectors/slack.md)：七步设置向导、关联 Slack 账户、将 Slack 对话中的沟通说明传递到任务，以及下载代理头像作为机器人头像。
- [例行任务](../guides/projects-workflow/routines.md)、[创建每日例行任务](../how-to/create-a-daily-routine.md) 和[例行任务 API](api/routines.md)：计划和 webhook 触发器向导、一次性凭据、测试投递、webhook URL 可能无法公开访问时的提示，以及例行任务内显示的运行列表；还更正了 `hmac_sha256` 和 `github_hmac` 的签名请求头。
- [问题](../guides/day-to-day/issues.md)、[聊天式任务](../experimental/task-chat.md) 和[技能](../guides/org/skills.md)：代理仍在工作时即可回答问题或审批；代理可从完成的任务创建技能；新增首次任务引导技能。聊天式任务现描述为默认界面，Classic Task Interface 则是旧版切换项。
- [工具网关 API](api/tool-gateway.md) 和[Paperclip 识别但不列出的提供方](../connectors/recognized-providers.md)：旧版 Composio 代理及按工具包划分的服务路由已移除。旧连接会以 `422 composio_broker_retired` 拒绝，需重新创建为 Composio MCP 连接。
- [Zapier](../connectors/zapier.md)、[Arcade](../connectors/arcade.md)、[Composio](../connectors/composio.md) 和 [Executor](../connectors/executor.md)：四者现在在管理员启用实验性 **MCP aggregators** 设置前均隐藏。
- [沙箱提供方](adapters/sandbox-providers.md)：新增 CreateOS 提供方及其配置字段。
- [插件](../administration/plugins.md) 和[插件 SDK](plugins/sdk.md)：支持在自定义镜像的发行版目录中携带预构建插件，并新增包裹整个应用的 `appShellOverlay` 槽位。
- 其他更新：[工件](../guides/day-to-day/artifacts.md)、[命令面板](../guides/day-to-day/command-palette.md)、[实验功能](../experimental/overview.md)、[任务计划分解面板](../experimental/plan-decomposition-panel.md)、[Connections v3](../experimental/connections-apps.md)、[连接自定义 MCP 服务器](../connectors/custom-mcp-servers.md)、[为人员和代理使用不同账户](../connectors/separate-accounts.md)、[公司技能策略](api/company-skill-policy.md) 和[技能参考](skills.md)。

</div>
</details>

<details class="accordion">
<summary>v2026.916.1 文档 <span class="accordion-meta">2026 年 9 月 21 日</span></summary>
<div class="accordion-body">

这是一个补丁版本：Paperclip 检查任务是否暂停期间，任务对话的发送按钮不再一开始就禁用。文档原本已正确描述撰写框行为，因此无需修订页面。本条还汇总了 v2026.916.0 后发布的页面。

**新增页面**

- [设置 HTTPS](deploy/https.md)：通过 Tailscale 或反向代理为实例提供浏览器信任的 HTTPS 地址，并让连接 webhook 可被网络外服务访问。
- [了解 GitHub PR 审核机器人](../connectors/understanding-github-review-bots.md)：说明安装审核机器人、选择其运行时机，以及要求合并前检查通过，是三项独立决定。
- [设置 GitHub 审核机器人](../connectors/github-review-bot-setup.md)：端到端配置实验性的代理审核器，包括注册并安装 GitHub App、配置提及和推送事件，以及观察真实 PR 上的失败检查转为通过。

**更新页面**

- [信任与低信任审核](../administration/trust-and-low-trust-review.md)：新增 GitHub 机器人代理的选择建议。由于 PR 可能包含公司之外的指令，应使用有明确工作边界、隔离工作区和沙箱的专用低信任代理。
- [GitHub](../connectors/github.md) 和[设置 GitHub 连接器](../connectors/github-setup.md)：聊天配置路径现在也涵盖 PR 审核机器人，并转到新的设置指南。
- 小幅更新：[部署概览](deploy/overview.md) 和 [Tailscale 私有访问](deploy/tailscale-private-access.md) 链接到新 HTTPS 指南。

</div>
</details>

<details class="accordion">
<summary>v2026.916.0 文档 <span class="accordion-meta">2026 年 9 月 16 日</span></summary>
<div class="accordion-body">

**新增页面**

- [连接意图 API](api/connection-intents.md)：代理如何请求人员接入其无法访问的服务，包括进入任务线程的连接卡片、`requested` → `connected` 生命周期，以及代理调用的运行时工具与人员响应的看板路由之间的分工。
- [`connections` 命令](cli/connections.md)：`connections search` 和 `connections request`，以及代理如何在活动心跳中发现可连接服务并提出连接请求。
- [`managed-agent` 命令](cli/managed-agent.md)：配置并验证受限 Anthropic 托管代理和环境，然后保存 Paperclip 派发工作时读取的公司配置档。
- [`test-drive` 命令](cli/test-drive.md)：一条命令启动隔离本地实例，创建预设公司和 CEO 代理、配置提供方凭据并打开仪表板，免去自行接线即可试用 Paperclip。

**更新页面**

- [角色与权限](../administration/roles-and-permissions.md) 和[公司管理](../administration/company.md)：Owner 和 Admin 角色现在默认获得四项常用 `tools:*` 权限（`manage_connections`、`manage_runtime`、`use`、`admin`）；`tools:view_audit` 与 `tools:manage_profiles` 仍需显式授权。`tools:admin` 不是其他权限的超集，不会隐含另外三项。
- [将代理连接到 GitHub](../how-to/connect-agent-to-github.md)：新增 **选项 C：托管 GitHub 连接**。Paperclip 通过 Connectors 管理每次运行的凭据，无需自行签发或轮换；自动使用已连接 GitHub 身份创建提交；不会回退到 PAT，而是安全失败；并通过连接 webhook 同步关联 PR 状态。
- [代理 API](api/agents.md)：新增看板专用、公司级、按 `profileKey` upsert 的**托管和远程代理配置档**；所有序列化代理都会将明文 `adapterConfig.env` 替换为 `***REDACTED***`，而 `secret_ref` 绑定仍会原样返回；还可重新发现调用者自己的活动 setup-token 登录会话。
- [工具网关 API](api/tool-gateway.md)：代理可在 `/api/agents/me/connections/...` 下调用 `start-authorization` 启动 OAuth，或调用 `token` 签发短期上游凭据；两者都限定在代理的活动运行内。
- [实例管理 API](api/instance-admin.md)：记录带云端管理下限的实例设置读取/修补路由，以及用于干净关停、暂停新工作的任务排空端点。
- [插件 SDK](plugins/sdk.md)：新增两个受能力控制的上下文客户端：`ctx.access`（公司成员与邀请）和 `ctx.authorization`（授权、策略摘要和授权审计记录）；还为环境驱动工作进程新增登录 PTY 和双向通道流式传输。
- [环境变量](deploy/environment-variables.md)：新增适用于 Gmail/Workspace OAuth 代理的 **Paperclip ID Connector** 环境变量组（`PAPERCLIP_ID_CONNECTOR_BASE_URL`、`_ENVIRONMENT`、`_INSTANCE_ID` 和签名/封装私钥），以及 HTTP 适配器的 `PAPERCLIP_HTTP_ADAPTER_PRIVATE_ENDPOINT_ALLOWLIST`。
- [Codex 适配器](adapters/codex.md)：默认模型现为明确的 `gpt-5.6-sol`（裸 `gpt-5.6` 别名会自动重写）；新增 `fastMode` 快速模式、`modelReasoningEffort` 推理力度级别，以及由公司级凭据缓存支持的交互式沙箱设备登录，可通过 `PAPERCLIP_CODEX_AUTH_CACHE` 禁用。
- [Claude Code 适配器](adapters/claude-code.md)：支持 Claude Fable 5.1，要求 Claude Code CLI 至少为 `2.1.251`，版本不足会以 `claude_cli_version_incompatible` 快速失败；较旧 CLI 会温和降级 `--effort`。
- [Grok 本地适配器](adapters/grok-local.md)：在现有按量计费 `XAI_API_KEY` 模式之外，新增通过公司级 `GROK_HOME` 和沙箱设备登录流程使用订阅（SuperGrok）认证。
- [HTTP 适配器](adapters/http.md)：在 socket 边界使用固定 DNS 检查每个请求的 SSRF 防护，默认阻止私有、环回和链路本地源，仅当私有源与允许列表完全匹配时才可访问。
- [沙箱提供方](adapters/sandbox-providers.md)：新增 Daytona 可选 warm-runner 生命周期（`runnerLifecycleMode`、`runnerIdleTimeoutMs`）、逐次调用的存活超时、基于镜像的资源覆盖，以及在真实沙箱终端中交互式登录代理。
- 其他更新：[OpenCode 适配器](adapters/opencode.md)在模型预检中会先刷新一次过期目录再拒绝运行；[命令面板](../guides/day-to-day/command-palette.md)在侧边栏保留最近五个任务并提供快捷操作；CLI [适配器](cli/adapter.md) 和[设置命令](cli/setup-commands.md)页面记录 `detect-model`，并说明 `onboard` 会生成工具操作签名密钥。

</div>
</details>

<details class="accordion">
<summary>v2026.831.1 文档 <span class="accordion-meta">2026 年 9 月 2 日</span></summary>
<div class="accordion-body">

**新增页面**

- [Kimi Code 适配器](adapters/kimi-local.md)：以本地代理运行 Moonshot Kimi Code CLI（`kimi_local`），介绍共享 ACP 引擎和无头 CLI 回退、模型与思考力度级别、会话恢复、技能注入及三种认证方式。

**更新页面**

- [适配器概览](adapters/overview.md)：在内置适配器列表及 ACP 引擎级别中加入 Kimi Code。
- [环境变量](deploy/environment-variables.md)：新增 `PAPERCLIP_WORKSPACE_REAPER_COOLDOWN_DAYS`（终端工作区归档前等待时间）、通过 `SENTRY_DSN` 可选启用 Sentry 错误监控，以及操作员控制项 `PAPERCLIP_HIDDEN_SETTINGS` 和 `PAPERCLIP_SETTING_DEFAULTS`。
- [实例设置](../administration/settings.md)：新增面向托管 Paperclip 实例操作员的章节，说明如何按键隐藏设置界面和覆盖设置默认值；这些值均不会持久化保存。
- [公司管理](../administration/company.md)、[成员与访问](../guides/org/members-and-access.md) 和[角色与权限](../administration/roles-and-permissions.md)：设置现在使用统一导航；Invites 移入 Members 页面标签；移除公司品牌颜色和公司级附件大小限制。
- [Grok 本地适配器](adapters/grok-local.md)：`permissionMode` 不再默认为 `dontAsk`；未设置时不会传入权限模式标志；无人值守策略为 `--always-approve`。
- [第一个公司](../guides/getting-started/your-first-company.md) 和[五分钟上手](../guides/getting-started/five-minute-path.md)：引导流程改为单卡片向导，直接从创建代理开始；移除单独的使命设置步骤，之后再设置目标。
- [任务看门狗](../guides/projects-workflow/task-watchdogs.md)、[自动创建恢复任务](../experimental/auto-create-recovery-tasks.md) 和[问题](../guides/day-to-day/issues.md)：静默运行检测现在只显示 UI 级别提示，不会创建 issue、评论或唤醒；搁置任务恢复转交给看板操作，而不再接管工作；自动运行摘要评论只包含最终输出，不含代理思考过程。
- [认证 API](api/authentication.md)：无效代理令牌现在返回说明原因的 `401`，不再回退为匿名调用者。
- [公司 API](api/companies.md) 和 [Cases API](api/cases.md)：从公司数据结构和品牌路由中移除 `brandColor`；附件上限改由部署级 `PAPERCLIP_ATTACHMENT_MAX_BYTES` 设置，不再是公司字段。
- CLI [安装](cli/installation.md) 和[设置](cli/setup-commands.md)页面、[本地开发](deploy/local-development.md)、[Modal 适配器](adapters/modal.md)及多份指南现在声明 Node.js 最低版本提高到 **24.11.0**。

</div>
</details>

<details class="accordion">
<summary>v2026.824.1 文档 <span class="accordion-meta">2026 年 8 月 25 日</span></summary>
<div class="accordion-body">

**更新页面**

- [CLI 设置命令](cli/setup-commands.md)：`onboard` 安装后台服务后，会引导至正在运行的实例：等待服务实际绑定端口、显示仪表板 URL 并在浏览器中打开。无头运行会打印 URL；设置 `PAPERCLIP_NO_BROWSER=1` 可跳过浏览器启动。

</div>
</details>

<details class="accordion">
<summary>v2026.824.0 文档 <span class="accordion-meta">2026 年 8 月 24 日</span></summary>
<div class="accordion-body">

**新增页面**

- [Tailscale HTTPS 代理](deploy/tailscale-https-broker.md)：操作员端辅助服务，为托管工作区里的开发服务器生成真实、证书有效的 `https://` 预览地址，替代仅环回可访问的链接。

**更新页面**

- [工作区](../guides/projects-workflow/workspaces.md)：按服务选择是否将工作区开发服务器公开为 tailnet 上的 HTTPS 预览，以及看板中的显示方式。
- [更新 Paperclip](../how-to/update-paperclip.md) 和 [CLI 安装](cli/installation.md)：介绍四个发布渠道（`stable`、`beta`、`nightly`、`canary`），以及用于查看当前安装所跟随渠道的 `paperclipai channels` 命令。
- [导出与导入](../guides/power/export-import.md)：大型包现在分块、可续传上传；导入中断后可从已完成部分继续，而无需重新开始。
- [公司 API](api/companies.md)：记录支持可续传导入的分块传输路由 `/api/companies/import/transfers`。
- [密钥 API](api/secrets.md)：代理可调用密钥目录路由，在不暴露完整元数据的情况下选择要引用的密钥。
- [代理 API](api/agents.md)：新增 Claude 订阅（setup-token）登录流程，公司所有者可用订阅登录 Claude，无需粘贴 API key。
- [适配器 API](api/adapters.md)：新增适配器设备登录路由（使用代码和 URL 浏览器登录），首先支持 Codex。
- [工件](../guides/day-to-day/artifacts.md)：在 Plan 和 Artifact 文档中支持 Google Docs 风格的行内评论，包括锚定高亮、线程回复、解决/重新打开和可共享评论链接。
- [问题](../guides/day-to-day/issues.md) 和[关注事项 API](api/attention.md)：说明谁可以解决交互卡片（`anyone`、`not_creator`、`human_only`）以及公司范围的交互治理。
- [问题 API](api/issues.md)：新增工作区文件资源可用性检查。
- 小幅同步更新：[环境变量](deploy/environment-variables.md)（工作区 Git 扫描限制）和[决策](../guides/day-to-day/decisions.md)。

</div>
</details>

<details class="accordion">
<summary>v2026.817.0 文档 <span class="accordion-meta">2026 年 8 月 17 日</span></summary>
<div class="accordion-body">

**新增页面**

- [决策 API](api/decisions.md)：提议和解决决策、决策包、命名队列、分流（截止时间和延后处理），以及保留/归档路由。
- [状态卡 API](api/status-cards.md)：共享状态卡看板，包括创建卡片、编译后的查询、摘要写入和修订、刷新策略及代理创建卡片的限制。
- [状态卡](../experimental/status-cards.md)：实验性看板本身，包括驱动卡片的一句话消息、图块、五种卡片状态、变更判断方式和费用。
- [聊天式任务](../experimental/task-chat.md)：实验性实时对话式任务页面，包括消息气泡、折叠轮次、行内工具调用和差异、三种撰写模式及可调整大小的侧窗格。
- [`service` CLI](cli/service.md)：将 Paperclip 安装、启动并作为后台服务检查。
- [Status Card Query 技能](skills/bundled/paperclip-operations/status-card-query.md)：教代理管理状态卡的内置技能。
- [简明英语技能](skills/optional/content/simplified-english.md) 和 [Prepare MCP Integration 技能](skills/optional/software-development/prepare-mcp-integration.md)：两个新的可选目录技能。

**更新页面**

- [决策](../guides/day-to-day/decisions.md)：新增命名队列、分流截止时间和回答代理提议的决策，并全面加入截图。
- [密钥 API](api/secrets.md)：代理密钥提案、代理令牌必须绑定有效运行的要求，以及看板批准/拒绝流程。
- [活动 API](api/activity.md)：代理操作审计动态、两层访问模型和 CSV 导出；`/audit` 已并入单一 Activity 页面。
- [插件 SDK](plugins/sdk.md)：如何响应交互和审批，如何处理适配器编写的 `command` 操作，并在执行前重新验证 `cwd`。
- [备份与恢复公司](../how-to/back-up-and-restore-a-company.md)：备份包刻意不包含的内容、改用 ZIP 上传而非内联 JSON，以及将大型导入作为后台作业执行。
- [更新 Paperclip](../how-to/update-paperclip.md)：围绕提交更新前的检查、切换渠道、回滚和升级前备份重写。
- [Cloud CLI](cli/cloud.md)：停用云上游命令并说明替代方式。
- [问题 API](api/issues.md)、[关注事项 API](api/attention.md)、[环境变量](deploy/environment-variables.md)、[CLI 安装](cli/installation.md)、[导出与导入](../guides/power/export-import.md)、[沙箱提供方](adapters/sandbox-providers.md)、[技能参考](skills.md) 和[问题](../guides/day-to-day/issues.md)：同步至本版本。

**截图**

- 所有截图均基于 v2026.817.0 重新采集，包含浅色和深色模式，共 342 张；此前的截图比当前父提交早 375 个提交。
- 新增决策、状态卡、聊天式任务和密钥提案审核标签页截图；前三份指南此前均没有图片。

</div>
</details>

<details class="accordion">
<summary>v2026.722.0 文档 <span class="accordion-meta">2026 年 7 月 22 日</span></summary>
<div class="accordion-body">

**新增页面**

- [密钥文件夹](../administration/secret-folders.md)：整理密钥文件夹。
- [Connections v3](../experimental/connections-apps.md)：实验性 Connections v3（Apps）基础功能。

**更新页面**

- [密钥 API](api/secrets.md) 和[代理 API](api/agents.md)：记录运行绑定的代理密钥访问（`GET /api/agents/me/secrets/:key/value`）。
- [本地代理（ACPX）](adapters/acpx-local.md)：新增无需 Bash 封装器的原生 Windows 执行方式。
- [环境变量](deploy/environment-variables.md)：`PAPERCLIP_*` 绑定透传和退出机制。
- [Codex 适配器](adapters/codex.md)：收窄 `CODEX_HOME` 沙箱同步允许列表。
- [插件 SDK](plugins/sdk.md)：环境同步导出，以及 `onEnvironmentSyncIn` / `onEnvironmentSyncOut` 钩子。
- [`company` CLI](cli/company.md)：新增 `export --force` 标志。

</div>
</details>

<details class="accordion">
<summary>v2026.720.0 文档 <span class="accordion-meta">2026 年 7 月 20 日</span></summary>
<div class="accordion-body">

**新增页面**

- [工具网关 API](api/tool-gateway.md)：MCP 工具网关，包括应用和连接、目录条目与风险级别、配置档/条目/绑定、工具访问策略、具名 MCP 网关和令牌、审计动态及 Smoke Lab；还记录 `tools:*` 权限键和两项实验性开关。
- [摘要槽 API](api/summary-slots.md)：内置 Summarizer 和摘要槽（槽位寻址、生成、修订及 `enableSummaries` 开关）。

**更新页面**

- [技能](../guides/org/skills.md)：Skill Studio（三栏编写工作区、已保存输入、测试运行、运行模板、版本历史）、嵌套技能文件夹、My Skills 视图、从项目导入技能以及公司技能分叉。
- [本地代理（ACPX）](adapters/acpx-local.md)：上游适配器退役后，页面精简为退役说明，并指向 Claude Code / Codex / Gemini CLI，同时说明自动迁移。

</div>
</details>

<details class="accordion">
<summary>v2026.707.0 文档 <span class="accordion-meta">2026 年 7 月 7 日</span></summary>
<div class="accordion-body">

**新增页面**

- [Ramp 技能](../reference/skills/optional/finance/ramp.md)：内置 Ramp 财务技能。
- 自定义沙箱镜像：记载于[沙箱提供方](adapters/sandbox-providers.md)。

**更新页面**

- [工作时间线](../guides/day-to-day/work-timeline.md)：新增工作时间线视图。
- [密钥范围](../administration/secret-scopes.md)：更新密钥范围内容。

</div>
</details>

<details class="accordion">
<summary>v2026.626.0 文档 <span class="accordion-meta">2026 年 6 月 26 日</span></summary>
<div class="accordion-body">

**更新页面**

- [Hermes 适配器](adapters/hermes.md) 和 [Hermes Gateway](adapters/hermes-gateway.md)：介绍两种内置 Hermes 适配器。
- [工作模式](../guides/day-to-day/work-modes.md)：新增 ask 工作模式。
- [例行任务](../guides/projects-workflow/routines.md)：新增例行任务日期变量。
- [插件 SDK](plugins/sdk.md)：插件目标命令，以及任务看门狗和工作区文件下载。

</div>
</details>

<details class="accordion">
<summary>v2026.618.0 文档 <span class="accordion-meta">2026 年 6 月 18 日</span></summary>
<div class="accordion-body">

**新增页面**

- Novita Agent Sandbox 提供方（驱动程序 `novita`）：加入[沙箱提供方](adapters/sandbox-providers.md)。
- 内置 `paperclip-board` 技能：加入[技能](../guides/org/skills.md)。

**更新页面**

- 适配器：[Codex](adapters/codex.md)、[Gemini CLI](adapters/gemini-cli.md)、[OpenCode](adapters/opencode.md)、[Pi](adapters/pi.md)、[OpenClaw Gateway](adapters/openclaw-gateway.md)，以及[沙箱提供方](adapters/sandbox-providers.md)中的 Kubernetes。
- [代理 API](api/agents.md)、[插件 SDK](plugins/sdk.md) 和[环境变量](deploy/environment-variables.md)（`TRUST_PROXY` / OTEL）。
- 日常指南：[工件](../guides/day-to-day/artifacts.md)、[问题](../guides/day-to-day/issues.md) 和[例行任务](../guides/projects-workflow/routines.md)。

</div>
</details>

<details class="accordion">
<summary>v2026.609.0 文档 <span class="accordion-meta">2026 年 6 月 9 日</span></summary>
<div class="accordion-body">

**新增页面**

- [`token` CLI](cli/token.md)：`token agent` / `token board` API key 命令。
- [`connect` CLI](cli/connect.md)：交互式 `connect` 设置向导。
- [团队目录 API](api/teams-catalog.md)：团队目录 REST API。

**更新页面**

- 为 49 个页面添加版本标记 `v2026.609.0`，并将三个新页面加入导航。

</div>
</details>

<details class="accordion">
<summary>v2026.529.0 文档 <span class="accordion-meta">2026 年 5 月 29 日</span></summary>
<div class="accordion-body">

**更新页面**

- [Claude Code 适配器](adapters/claude-code.md)：通过 `ANTHROPIC_API_KEY` 查询 `/v1/models` 实时发现模型（缓存 60 秒，内置回退、Bedrock ID 和刷新控件）。
- [工作区](../guides/projects-workflow/workspaces.md)：复用工作区的环境一致性，以及依赖任务仅在完成操作后唤醒。
- 继承的 nightly 草稿：[资源成员 API](api/resource-memberships.md)、文档批注、插件管理器中的内置插件、技能 CLI 与目录，以及首位管理员认领流程。

</div>
</details>

<details class="accordion">
<summary>v2026.525.0 文档 <span class="accordion-meta">2026 年 5 月 25 日</span></summary>
<div class="accordion-body">

**新增页面**

- Modal 沙箱提供方：加入[沙箱提供方](adapters/sandbox-providers.md)。
- [Workspace Diff Viewer 插件](plugins/workspace-diff.md)：支持分栏/统一差异、工作树/指定 ref 对比切换、基准 ref 输入和固定工具栏。

**更新页面**

- [插件 SDK](plugins/sdk.md)：审查 SDK 接口并介绍托管资源概念。
- [例行任务](../guides/projects-workflow/routines.md)：例行任务环境运行契约和 secret-ref 绑定选择器。
- 新增一条故障排查说明：创建新密钥后遇到 401 的处理方式。

</div>
</details>

<details class="accordion">
<summary>v2026.517.0 文档 <span class="accordion-meta">2026 年 5 月 17 日</span></summary>
<div class="accordion-body">

**新增页面**

- [Grok 本地适配器](adapters/grok-local.md)：新增 `grok_local` 适配器，并加入适配器概览和导航。

**更新页面**

- [问题](../guides/day-to-day/issues.md) 和[问题 API](api/issues.md)：锁定工作流（锁定/解锁、派生文档重定向）以及看板视图缩放控制。
- [沙箱提供方](adapters/sandbox-providers.md)：Cloudflare 可靠性调优说明。

</div>
</details>

<details class="accordion">
<summary>v2026.513.0 文档 <span class="accordion-meta">2026 年 5 月 13 日</span></summary>
<div class="accordion-body">

**新增页面**

- [本地开发插件](../how-to/develop-a-plugin-locally.md)：演示 `paperclipai plugin init`、本地路径安装、开发监视器和重新加载。
- [受阻收件箱](../guides/day-to-day/blocked-inbox.md)：受阻收件箱标签、标记变体、过滤器、排序和分流。

**更新页面**

- [问题](../guides/day-to-day/issues.md) 和[问题 API](api/issues.md)：恢复操作及逐层处理子问题。
- [Claude Code 适配器](adapters/claude-code.md)：恢复会话的工作区。
- [插件 SDK](plugins/sdk.md)：工作进程入口验证。
- [插件管理](../administration/plugins.md)：本地开发插件。

</div>
</details>

---

_本变更记录从本仓库首次跟踪的 v2026.513.0 开始。Paperclip 的完整功能和修复历史请参阅 [Paperclip 版本发布页](https://github.com/paperclipai/paperclip/releases)。_
