# 截图计划——v2026.817.0

发行版截图全面更新的工作笔记。基线 `903bd157c`（v2026.720.0，2026-07-20）→ 发布提交 `213dabab4`（2026-08-17）。共 375 个父提交，`ui/src` 中 439 个文件发生变化，增加 41,580 行、删除 5,295 行。

> **请针对 `213dabab4` 截图，不要使用 `master`。**此发布提交位于 `upstream/candidate/release-2026.817.0`，不是父仓库 `master` 的祖先；本地 `master`（2026-08-13）比它早四天，而且包含发布版本没有的 93 个提交。针对 `master` 截图既会遗漏发布版 UI，也会混入标签之后的 UI。该提交没有对应的 git tag；`git describe` 会返回 `beta/v2026.811.0-beta.0-4-g213dabab4`。

---

## A. 现有截图——重新采集（注册表共 310 条）

310 条中有 228 条在 `SCREENSHOTS_PENDING.md` 中被标记。鉴于截图已过时，应使用 `--all` 重新采集整个注册表。以下三组还需要额外处理，因为盲目运行 `--all` 会生成看似可信、实际错误的图片：

### A1. 悬空的 `depends_on`——上游页面已删除

| 目标 | 当前 `depends_on` | `213dabab4` 时的实际情况 |
|---|---|---|
| `activity/activity-log-full`、`activity/activity-filters`、`activity/activity-filtered-by-agent` | `ui/src/pages/Activity.tsx` | **已删除。**`/activity` 现在由 `pages/audit/CompanyActivity.tsx`（以及 `pages/audit/AuditFeed.tsx`）渲染。`/audit` 会重定向到 `/activity?mode=agents`。 |
| `experimental/cloud-upstream` | `ui/src/pages/CloudUpstream.tsx` | **已删除。**`/company/settings/cloud-upstream` 现在会重定向到 `/company/export`。 |

这是 `routes.mjs` 中仅有的两个在该发布提交里已不存在的 `depends_on` 路径（通过 `git ls-tree -r 213dabab4` 核对）。

- **Activity：**更新 `depends_on` 指向，保留路由并重新采集。该页面确实仍然存在，而且 `docs/guides/day-to-day/activity-log.md` 嵌入了其中三张截图。
- **Cloud upstream：**对应截图已经失效。`docs/experimental/cloud-sync.md:11` 仍在嵌入这张图。应改为拍摄替代页面 `/company/export`，不要继续使用旧名称拍摄重定向页面。

### A2. 现在会重定向的路由

`/instance/settings/*`（8 个目标：profile、general、access、heartbeats、experimental、plugins ×5、adapters ×4、environments）仍然可以访问——`LegacySettingsRedirect` 会将它们转到规范路径 `/{prefix}/company/settings/instance/*`。截图不受地址栏影响（Playwright 截图不包含 URL 栏），但重定向会先渲染 `<PaperclipLoading />`，这正是 `capture.mjs` 需要重试的空白页面情况。应改用规范路径：速度更快，也不会产生不稳定截图。

另外：`company/access` 现在会转到 `/{prefix}/company/settings/access` 并命中 `CompanyAccessLegacyRoute`；成员页面已移至 `/company/settings/members`。

### A3. 已过时的 seed 标志

`seed.mjs:686` 设置了 `enableCloudSync: true`。`instanceExperimentalSettingsSchema` 中已不存在该标志。补丁验证器使用 `.strip()` 而非 `.strict()`，所以调用仍会成功，但这个键会被静默丢弃——虽然无害，仍应移除。必须添加 `enableDecisions` 和 `enableStatusCards`。

---

## B. 新增界面——目前没有截图

按文档需求排序。以下三个页面目前都没有任何图片。

### B1. 决策——`docs/guides/day-to-day/decisions.md`（115 行，0 张图片）

路由：`/{prefix}/decisions`（`WhatNeedsMe`）、`/{prefix}/decisions/queues/:key`（`DecisionQueuePage`）、`/{prefix}/decisions/training`。

| 新目标 | 界面 | 文档章节 |
|---|---|---|
| `decisions/queue-overview` | 排序后的信息流 | 查看队列 |
| `decisions/decision-card` | 智能体提出的决策，包含选项和输入字段 | 回答智能体提出的决策 |
| `decisions/decision-card-stale-target` | “自提出此决策后，1 个目标已更改” | 同上（安全提示） |
| `decisions/toolbar-filters` | 类型/严重程度/项目/工作区 + 分组 + 排序 | 聚焦下一个决策 |
| `decisions/named-queue` | 一个队列栏（PR / 计划 / 问题） | 将工作分组到命名队列 |
| `decisions/triage-chips` | 今天/本周/不限时间的截止时间 + 延后 | 指定何时需要你处理 |
| `decisions/decided-expired-sections` | 折叠的历史记录 | 回答决策（最后一段） |
| `decisions/empty-state` | “你已处理完所有事项” | 队列为空时 |

组件：`DecisionCard`、`DecisionsToolbar`、`DecisionQueueRail`、`DecisionShelf`、`DecisionTriageStrip`、`DecisionDateChips`、`DecisionResolver`。

### B2. 状态卡片——`docs/experimental/status-cards.md`（136 行，0 张图片）

路由：`/{prefix}/status`、`/{prefix}/status/:cardId`。受 `enableStatusCards` 控制；还需要预置内置 **Summarizer** 智能体。

| 新目标 | 界面 | 文档章节 |
|---|---|---|
| `status-cards/board` | 磁贴网格和页眉汇总 | 阅读看板 |
| `status-cards/new-card-dialog` | “你想关注什么？” | 创建第一张卡片 |
| `status-cards/detail-summary` | 抽屉、Summary 标签和修订版本选择器 | 阅读看板 |
| `status-cards/detail-settings` | 更新策略和 Advanced 触发器 | 更新方式 |
| `status-cards/detail-watched-issues` | 实时查询匹配结果 | 阅读看板 |
| `status-cards/detail-history` | 每次更新的令牌数/费用/模型 | 费用说明 |
| `status-cards/archived-row` | 显示已归档 → 查看/恢复 | 归档与恢复 |

组件：`pages/StatusCards/{index,StatusCardTile,StatusCardDetailDrawer,CreateStatusCardDialog,StatusCardSettingsForm,ArchivedStatusCardRow}.tsx`。

### B3. 机密提案——`docs/administration/secret-scopes.md`（已有 4 张图片，但没有提案相关图片）

路由：`/{prefix}/company/settings/secrets`，**Proposals** 标签页（由本地 `activeTab` 状态控制，因此需要点击 `steps`，不能只改 URL）。

| 新目标 | 界面 |
|---|---|
| `secrets/proposals-tab` | 待处理提案列表，带琥珀色数量徽标 |
| `secrets/proposal-review` | 展开的单项提案，包含 Approve / Reject |
| `secrets/proposal-resolved` | 一项已批准提案和一项已拒绝提案 |

组件：`pages/secrets/ProposalsTab.tsx`、`proposal-review.tsx`、`components/AgentSecretAccessEditor.tsx`。

### B4. 聊天式任务——`docs/experimental/task-chat.md`（135 行，0 张图片）

受 `enableTaskChatRedesign` 控制。显示在普通的问题详情路由 `/{prefix}/issues/{issueId}` 上，不需要新增路由。

| 新目标 | 界面 | 文档章节 |
|---|---|---|
| `task-chat/thread` | 对话气泡、作者标题、模式标签 | 查看讨论串 |
| `task-chat/folded-turn` | `✓ Worked · 38s · 3 tools · +34 −3` | 轮次 |
| `task-chat/tool-call-diff` | 展开的工具行和差异面板 | 工具调用和差异 |
| `task-chat/composer-modes` | Agent · Plan · Ask 选择器 | 撰写区 |
| `task-chat/side-pane-plan` | Properties · Plan · Artifacts 侧边栏 | 侧边栏 |

> **尚未解决的风险。**丰富的轮次内容（思考块、工具调用、差异、实时状态行）需要真实的适配器转录记录。截图实例没有 LLM 提供商，而 `process` 演示适配器只会输出普通 stdout，无法生成这些内容。父仓库提供基于 fixture 的开发测试工具 `/dev/task-chat-lab`（`pages/TaskChatLab.tsx` + `task-chat-fixtures.ts`），仅在 DEV 环境且功能标志启用时可用；流水线确实会通过 Vite dev 中间件提供 UI。Phase 2 已记录决策：使用真实 seed 数据拍摄基础讨论串；仅当某些状态确实无法通过其他方式到达时，才回退到测试工具。文档截图即使来自 fixture 测试工具，也必须如实呈现最终发布的 UI。

### B5. 目录技能——新增 3 个参考页面

`reference/skills/bundled/paperclip-operations/status-card-query.md`、`reference/skills/optional/content/simplified-english.md`、`reference/skills/optional/software-development/prepare-mcp-integration.md`。

按照约定，技能参考页面不带截图——已对照现有 `reference/skills/**` 页面核实。唯一值得共用的一张截图是显示新增条目的技能目录；`skills/list` 和 `skills/skills-list` 已经覆盖该目录，且已列入重新采集范围。**无需新增目标。**但 seed 必须安装这些新技能，才能让它们显示在目录中。

---

## C. Seed 数据——阻塞因素

决策、状态卡片摘要和机密提案都由**智能体创建**。除非 `req.actor.type === "agent"` 且同时包含 `agentId` 和 `runId`，否则 `POST /companies/:id/decisions` 会返回 403 *“Agent run context required”*（需要智能体运行上下文），参见 `server/src/routes/decisions.ts:41`。Seed 当前以本地董事会用户身份访问实例，因此无法创建上述数据。

可使用以下支持的路径，无需直接修改数据库：

1. `POST /api/agents/:id/keys` → 返回明文 `token`（`services/agents.ts:896`，仅限董事会）。
2. 使用 `Authorization: Bearer <token>` 和 `X-Paperclip-Run-Id: <runId>` 调用智能体范围内的端点——`middleware/auth.ts:372` 会将 `actor.type` 设为 `"agent"`，并从该请求头读取 `runId`。
3. Seed 已会生成真实运行 ID（`runnerRunId`，来自 Task Runner 唤醒），因此可以用真实运行作为上下文。

这样可以启用上述三类操作：`POST /companies/:id/decisions`、`PUT /status-cards/:id/{query,summary}` 和 `POST /agents/me/secret-proposals`。

需要在运行时确认的风险：`createApiKey` 会将 `responsibleUserId` 保存为 `req.actor.userId ?? null`，而智能体密钥身份验证会拒绝没有责任用户的密钥（`forbidden("Responsible user is unavailable for this agent key")`）。如果 `local_trusted` 董事会身份没有 `userId`，该密钥就无法使用，Seed 必须先有真实董事会用户。

直接写入数据库（参考 `seed-execution-workspace.mjs` 模式）是后备方案，但决策由 HMAC 签名（`services/decision-signing.ts`、`decision-spec-v1`），所以手动插入的数据行必须有有效签名。应优先使用智能体密钥路径。

---

## C2. 首次完整运行发现的选择器漂移

首次 `--all` 运行采集了 342/342 张截图，没有跳过任何项目，但记录了 12 次步骤失败和 8 次裁剪回退。每一项都属于**预先存在**但选择器已不匹配重新设计 UI 的目标——没有任何新增目标失败。步骤或裁剪失败会在输出中静默处理（采集器会刻意回退到“当时到达的任意状态”），因此这些目标之前一直在悄悄生成错误图片：文档承诺显示模式选择器，实际只是普通收件箱；文档承诺显示单个面板，实际却截取了整个仪表板。

| 目标 | 之前 | 现在 |
|---|---|---|
| `work-modes/work-mode-picker` | 按 `c` 打开撰写区 | 已移除 `c` 快捷键——点击侧栏 **New Task** 按钮 |
| `skills/assign-to-agent` | 按钮“Attach to agents” | 已更名为 **Add to agent** |
| `routines/cron-picker` | 点击第一个 `combobox` | RoutineDetail 现在使用标签页——直接前往 `/routines/:id/triggers` |
| `org/org-chart-add-agent` | `/Add agent/i` | 按钮现在是 **New agent** |
| `onboarding/sidebar-new-company-button` | `/workspace switcher/i` | 标签现在是 **Open <Company> company switcher** |
| `issues/detail-sidebar` | 点击 `title="Show properties"` | 面板默认已打开，因此该切换按钮隐藏——无需步骤 |
| `dashboard/activity-feed` | `//h3[…]/ancestor::div[@class~rounded]` | Card 现在是标题的**同级元素**——使用 `parent::div` |
| `dashboard/task-breakdown-panel` | 同一 ancestor xpath | 同样修正 |
| `dashboard/stale-tasks-panel` | 裁剪 “Stale” 面板 | **已从 `Dashboard.tsx` 移除该面板**——删除目标（没有文档引用） |
| `tasks/comment-input-box` | 裁剪 textarea 的祖先元素 | 仍会回退到整页截图；没有文档引用，暂时保留 |

上述八个被引用的目标均已修复，并在浅色和深色主题下重新拍摄，步骤失败数为零。

> **仍需手动删除：**移除 `dashboard/stale-tasks-panel` 后，`light/dashboard/stale-tasks-panel.png` 和对应的 dark 文件仍被 Git 跟踪，但已不再使用。它们已从注册表移除，也没有任何内容引用。

## D. 工作顺序

1. Phase 1——添加注册表条目、`routes.mjs` 目标、`SCREENSHOTS_TODO.md` 行以及 B1–B4 文档图片嵌入；修复 A1/A2 路由和 `depends_on`。
2. Phase 2——扩展 `seed.mjs`（智能体密钥辅助函数、决策、状态卡片、机密提案、新技能、功能标志）；在 `213dabab4` 上启动实例，并在采集前目视检查每个新界面。
3. Phase 3——运行 `npm run screenshots:refresh:all`，然后审阅 PNG 图片。
4. Phase 4——准备部署；未经批准不得执行。
