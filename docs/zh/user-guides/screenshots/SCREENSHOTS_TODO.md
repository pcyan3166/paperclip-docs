# 截图待办事项——信息架构更新

> **警告——只能在一次性演示 Paperclip 实例上运行。绝不要针对真实数据操作。**任何采集到的 PNG 都不得包含真实用户头像、公司名称、任务标题或电子邮件。

默认视口：`1440 × 900`。注明的页面需同时采集浅色和深色主题。
所有文件保存到 `docs/user-guides/screenshots/light/<topic>/`（条件允许时也保存到 `/dark/<topic>/`）。

## 演示环境前提

采集前，请在演示实例中创建真实感较强的数据集，使截图内容充实：

1. 创建一家测试公司，例如 slug 为 `ACME`、名称为 `Acme Robotics`，并设置品牌色和徽标。
2. 邀请 2–3 名测试用户（使用临时邮箱）。保留一条待处理邀请，以便显示邀请状态徽标。
3. 打开第二个浏览器/设备，使用其中一条邀请链接，以便在队列中生成待处理的**加入请求**。
4. 在演示实例上安装三个捆绑的示例插件，使插件列表有内容：
   - `@paperclipai/plugin-hello-world-example`
   - `@paperclipai/plugin-file-browser-example`
   - `@paperclipai/plugin-kitchen-sink-example`
5. 创建一次虚拟导出，让 Export 页面显示历史记录行。
6. 除董事会用户外，再创建一名实例管理员，使 Access 列表中至少有多行。

下文使用公司 slug `ACME`，并以 `{slug}` 表示。

---

## 目标路径和路由

所有文件保存到 `docs/user-guides/screenshots/light/<group>/<name>.png`（如可行，同时保存对应的 `/dark/...` 副本）。

### 公司（`screenshots/light/company/`）

| 文件名 | URL | 备注 |
|---|---|---|
| `settings.png` | `/{slug}/company/settings` | 滚动到顶部。确保已上传徽标、设置品牌色并填写描述。 |
| `access.png` | `/{slug}/company/settings/access` | 显示至少 3 名拥有不同角色的成员。 |
| `invites.png` | `/{slug}/company/settings/invites` | 显示一条有效链接以及一条已撤销/过期链接。 |
| `join-requests.png` | `/{slug}/inbox/requests` | 至少有一个待处理请求。 |
| `export.png` | `/{slug}/company/export` | 保持默认选项，尚未提交任何内容。 |
| `import.png` | `/{slug}/company/import` | 尚未选择文件时的空状态。 |

### 实例设置（`screenshots/light/settings/`）

| 文件名 | URL | 备注 |
|---|---|---|
| `profile.png` | `/instance/settings/profile` | 使用虚构头像和姓名，例如 “Demo User”。 |
| `instance-general.png` | `/instance/settings/general` | 滚动到顶部——显示部署徽标，以及 censor-username 和 feedback-sharing 区域。 |
| `instance-access.png` | `/instance/settings/access` | 除董事会用户外，至少有一名实例管理员。 |
| `scheduler-heartbeats.png` | `/instance/settings/heartbeats` | 显示至少有一行内容的心跳列表。 |
| `experimental.png` | `/instance/settings/experimental` | 所有标志都保持默认值（关闭），并显示警告横幅。 |

### 插件（`screenshots/light/plugins/`）

| 文件名 | URL / 操作 | 备注 |
|---|---|---|
| `list.png` | `/instance/settings/plugins` | 安装全部三个示例插件，使 “Available Plugins” 和 “Installed Plugins” 区域都有内容。 |
| `install.png` | `/instance/settings/plugins` → 点击 “Install Plugin” | 截取已打开的安装对话框。 |
| `detail.png` | 点击左侧边栏中 Kitchen Sink 的插件名称，或导航到其仪表板界面 | 截取插件自身提供的页面。Kitchen Sink 的界面内容最丰富。 |
| `settings.png` | 点击 Kitchen Sink 上的 **Configure** | 插件设置表单（自动生成的 schema UI）。 |
| `jobs-log.png` | 在插件设置页面点击 **Status** 或 **Jobs** 标签 | 至少显示一条已记录的作业运行。如果插件尚无作业，等待 1–2 分钟让 worker 运行，或者使用会注册示例作业的 Kitchen Sink。 |

---

## 额外演示环境前提（信息架构更新批次）

扩充演示数据集，使新增主题批次中的页面显示填充后的视图：

1. 在 `ACME` 中创建 2 个智能体：
   - 管理者智能体 `Ada`（适配器不限；为她添加指令文件，使 Instructions 标签页有内容）。
   - 向 Ada 汇报的工作者智能体 `Bob`。为 Bob 分配至少一项技能，使 Skills 标签页不为空。
   创建后，从 URL 中记下 `{agentId}` 占位符对应的值。
2. 创建一个名为 `Website` 的项目，同时设置本地 `cwd` 和 `repoUrl`。为项目预算策略填入数据，使 Budget 标签页有内容。
3. 创建一个关联到 `Website` 项目的目标 `Launch v1`。
4. 创建一个例程（任意智能体，使用 cron 触发器），并手动运行一次，使其有一条执行记录。
5. 在 `Website` 项目中创建 3 个状态各异的问题（`todo`、`in_progress`、`done`）。将其中一个分配给 Bob，使其显示在 Inbox → Mine 下。在其中一个问题上发布若干评论，使聊天/活动标签页有内容。
6. 至少打开上述某个问题的执行工作区一次（例如触发一次心跳），使工作区详情包含运行时日志。
7. 除默认 local 适配器外，至少再安装一个适配器，使 AdapterManager 列表中有多行。
8. 通过 “Add skill” 至少添加一项技能，使技能页面的对话框周围有上下文。
9. 运行几次会产生费用的任务（使用任意真实或模拟的提供商事件），使 Costs 概览中的提供商、计费方和财务数据都非零。
10. 通过 `paperclipai login` 生成 CLI 身份验证代码，使 `/cli-auth/{id}` 显示真实的待处理请求。
11. 生成董事会认领令牌（使用 `paperclipai board claim ...` 或管理员流程），使 `/board-claim/{token}` 页面能够显示。

下文使用这些占位符：

- `{slug}` → `ACME`
- `{agentId}` → Bob 的智能体 ID（工作者智能体）
- `{projectId}` → Website 项目 ID
- `{goalId}` → Launch v1 目标 ID
- `{routineId}` → 演示例程 ID
- `{issueId}` → 含有聊天内容的演示问题 ID
- `{workspaceId}` → 项目的主工作区 ID
- `{executionWorkspaceId}` → 隔离执行工作区 ID（详情页截图通过数据库写入 seed 数据）
- `{cliAuthId}` → 来自 `/cli-auth` 的代码 ID
- `{boardClaimToken}` → 来自董事会认领链接的令牌

---

## 目标路径和路由（信息架构更新）

### 智能体（`screenshots/light/agents/`）

| 文件名 | URL | 备注 |
|---|---|---|
| `list.png` | `/{slug}/agents/all` | 显示至少 2 个智能体（Ada 和 Bob），且状态不同。 |
| `new.png` | `/{slug}/agents/new` | 空白新建智能体表单，显示页面顶部。 |
| `dashboard.png` | `/{slug}/agents/{agentId}/dashboard` | Bob 的仪表板，并填充近期活动。 |
| `instructions.png` | `/{slug}/agents/{agentId}/instructions` | 指令编辑器中已解析文件路径，并显示文件内容。 |
| `skills.png` | `/{slug}/agents/{agentId}/skills` | 至少一项已分配技能，并显示可用技能列表。 |
| `configuration.png` | `/{slug}/agents/{agentId}/configuration` | 滚动到顶部——显示适配器配置表单。 |
| `runs.png` | `/{slug}/agents/{agentId}/runs` | 至少显示一条运行记录。 |
| `budget.png` | `/{slug}/agents/{agentId}/budget` | 显示已配置上限的预算策略卡片。 |

> 智能体详情页没有 `keys` 标签——需求说明中提到过，但没有注册对应路由。请跳过。

### 成本（`screenshots/light/costs/`）

Costs 标签由 React 状态控制（不由 URL/hash 控制）。导航后点击对应标签触发器来采集每个标签。

| 文件名 | URL 和交互 | 备注 |
|---|---|---|
| `overview.png` | `/{slug}/costs`（默认标签） | 使用默认 MTD 预设；推理和财务指标均非零。 |
| `budgets.png` | `/{slug}/costs` → 点击 **Budgets** 标签 | 至少显示一张策略卡片（公司级或智能体级）。 |
| `providers.png` | `/{slug}/costs` → 点击 **Providers** 标签 | 至少显示一张提供商额度卡片。 |
| `billers.png` | `/{slug}/costs` → 点击 **Billers** 标签 | 至少显示一张计费方卡片。 |
| `finance.png` | `/{slug}/costs` → 点击 **Finance** 标签 | 填充时间线和按计费方分组的网格。 |

### 问题（`screenshots/light/issues/`）

| 文件名 | URL 和交互 | 备注 |
|---|---|---|
| `list.png` | `/{slug}/issues` | 列表中包含多种状态和负责人。 |
| `inbox.png` | `/{slug}/inbox/mine` | 默认 Mine 标签中至少有一个问题分配给已登录用户。 |
| `inbox-unread.png` | `/{slug}/inbox/unread` | 由第二个用户发布一条评论后截图。 |
| `inbox-requests.png` | `/{slug}/inbox/requests` | 包含管理员演示前提批次创建的待处理加入请求。 |
| `detail-chat.png` | `/{slug}/issues/{issueId}`（Chat 标签） | 默认标签——显示对话。 |
| `detail-activity.png` | `/{slug}/issues/{issueId}` → **Activity** 标签 | 显示状态/字段更改事件。 |

> 需求说明中提到了 `/{slug}/my-issues`，但 `App.tsx` 中没有注册此路由；对应视图是 `/{slug}/inbox/mine`（上文已列出）。

### 项目（`screenshots/light/projects/`）

| 文件名 | URL | 备注 |
|---|---|---|
| `list.png` | `/{slug}/projects` | 至少有一张项目卡片。 |
| `overview.png` | `/{slug}/projects/{projectId}/overview` | 概览已填入目标和工作区摘要。 |
| `issues.png` | `/{slug}/projects/{projectId}/issues` | 显示限定到项目的问题列表。 |
| `workspaces.png` | `/{slug}/projects/{projectId}/workspaces` | 至少关联一个工作区。 |
| `configuration.png` | `/{slug}/projects/{projectId}/configuration` | 项目设置表单。 |
| `budget.png` | `/{slug}/projects/{projectId}/budget` | 显示已配置上限的策略卡片。 |

### 目标（`screenshots/light/goals/`）

| 文件名 | URL | 备注 |
|---|---|---|
| `list.png` | `/{slug}/goals` | 显示 `Launch v1` 和其他已 seed 的目标。 |
| `detail.png` | `/{slug}/goals/{goalId}` | 目标详情中显示子问题。 |

### 例程（`screenshots/light/routines/`）

| 文件名 | URL | 备注 |
|---|---|---|
| `list.png` | `/{slug}/routines` | 至少显示一个带计划触发器的例程。 |
| `detail.png` | `/{slug}/routines/{routineId}` | 显示 Triggers 区域，以及手动运行产生的至少一条执行记录。 |

### 执行工作区（`screenshots/light/workspaces/`）

| 文件名 | URL | 备注 |
|---|---|---|
| `configuration.png` | `/{slug}/execution-workspaces/{executionWorkspaceId}/configuration` | 工作区设置、上下文和具体位置。 |
| `services.png` | `/{slug}/execution-workspaces/{executionWorkspaceId}/services` | 运行时服务和带控制项的作业。 |
| `runtime-logs.png` | `/{slug}/execution-workspaces/{executionWorkspaceId}/runtime-logs` | 操作日志行（配置、服务启动、作业失败）。 |
| `issues.png` | `/{slug}/execution-workspaces/{executionWorkspaceId}/issues` | Tasks 标签——关联到此工作区的问题。 |

### 适配器——用户侧（`screenshots/light/adapters/`）

| 文件名 | URL 和交互 | 备注 |
|---|---|---|
| `list.png` | `/instance/settings/adapters` | 安装至少 2 个适配器；显示 npm 和 local 来源图标。 |
| `install.png` | `/instance/settings/adapters` → 点击 **Add adapter** | 显示 npm/local 路径表单的打开对话框。 |
| `detail.png` | 点击某个已安装适配器行，展开其配置 | 显示适配器专属配置面板。 |

### 组织（`screenshots/light/org/`）

| 文件名 | URL | 备注 |
|---|---|---|
| `chart.png` | `/{slug}/org` | 显示组织图中 Ada → Bob 的汇报关系。 |

### 技能——扩展项（`screenshots/light/skills/`）

| 文件名 | URL 和交互 | 备注 |
|---|---|---|
| `list.png` | `/{slug}/skills` | 公司技能列表中至少有一项导入的技能。 |
| `add-skill-dialog.png` | `/{slug}/skills` → 点击 **Add skill**（对话框第一步） | 截取显示初始表单的对话框。 |
| `add-skill-dialog-confirm.png` | 同一对话框 → 前进到确认步骤 | 确认/预览视图。 |
| `file-inventory.png` | 打开技能行，显示文件清单面板 | 文件列表中要能看到路径。 |

### CLI 身份验证和董事会认领（`screenshots/light/auth/`）

| 文件名 | URL | 备注 |
|---|---|---|
| `cli-auth.png` | `/cli-auth/{cliAuthId}` | 显示待处理 CLI 身份验证代码页面。批准前截图。 |
| `board-claim.png` | `/board-claim/{boardClaimToken}` | 显示认领前状态下的董事会认领落地页。 |

### 实验功能（`screenshots/light/experimental/`）

供 `docs/experimental/*` 使用。采集前，Seed 在第 14e 步启用控制这些功能的标志（`seed.mjs`）。

| 文件名 | URL | 备注 |
|---|---|---|
| `environments.png` | `/instance/settings/environments` | 需要 `enableEnvironments`；显示环境列表和 Add environment。 |
| `cloud-upstream.png` | `/company/settings/cloud-upstream` | 需要 `enableCloudSync`；连接卡片处于尚未连接状态。 |
| `file-viewer-browse.png` | `/{slug}/issues/{issueId}?browse=1` | 需要 `enableExperimentalFileViewer`；文件查看器面板处于浏览模式。 |
| `external-objects.png` | `/{slug}/issues/{externalObjectIssueId}` | 需要 `enableExternalObjects`；问题描述中引用 GitHub PR/issue URL。 |
| `server-info.png` | `/{slug}/dashboard` + 点击 “Open account menu” | 需要 `enableServerInfoDebugView`；账户抽屉显示 Server 区域。 |
| `recovery-preview.png` | `/instance/settings/experimental` + 点击 “Preview” | Auto-Create Recovery Tasks 的 “Confirm auto-recovery” 对话框。 |

---

## 截图脚本（参考）

在 dev-browser 技能目录中运行，并确保演示实例位于 `http://localhost:3100`：

```bash
cd /path/to/dev-browser && npx tsx <<'EOF'
import { connect, waitForPageLoad } from "@/client.js";

const BASE = "http://localhost:3100";
const SLUG = "ACME";
const OUT  = "/path/to/paperclip-docs/docs/user-guides/screenshots/light";

const shots: [string, string][] = [
  [`/${SLUG}/company/settings`,            `${OUT}/company/settings.png`],
  [`/${SLUG}/company/settings/access`,     `${OUT}/company/access.png`],
  [`/${SLUG}/company/settings/invites`,    `${OUT}/company/invites.png`],
  [`/${SLUG}/inbox/requests`,              `${OUT}/company/join-requests.png`],
  [`/${SLUG}/company/export`,              `${OUT}/company/export.png`],
  [`/${SLUG}/company/import`,              `${OUT}/company/import.png`],
  [`/instance/settings/profile`,           `${OUT}/settings/profile.png`],
  [`/instance/settings/general`,           `${OUT}/settings/instance-general.png`],
  [`/instance/settings/access`,            `${OUT}/settings/instance-access.png`],
  [`/instance/settings/heartbeats`,        `${OUT}/settings/scheduler-heartbeats.png`],
  [`/instance/settings/experimental`,      `${OUT}/settings/experimental.png`],
  [`/instance/settings/plugins`,           `${OUT}/plugins/list.png`],
];

const client = await connect();
const page = await client.page("paperclip", { viewport: { width: 1440, height: 900 } });
for (const [path, out] of shots) {
  await page.goto(BASE + path, { waitUntil: "networkidle", timeout: 15000 }).catch(()=>{});
  await page.waitForTimeout(1200);
  await page.screenshot({ path: out });
  console.log("ok", path);
}
await client.disconnect();
EOF
```

安装对话框、插件详情、插件设置和作业日志截图需要手动点击交互——完成上述批量采集后，请逐项处理。

---

## 截图脚本——信息架构更新批次（参考）

运行前，从演示实例填入占位符 ID。批量完成 URL 目标后，仍需逐项手动操作并截图：Costs 标签、Adapters 和 Skills 对话框，以及 Issue 的 chat/activity 标签。

```bash
cd /path/to/dev-browser && npx tsx <<'EOF'
import { connect, waitForPageLoad } from "@/client.js";

const BASE = "http://localhost:3100";
const SLUG = "ACME";
const AGENT_ID       = "REPLACE_AGENT_ID";
const PROJECT_ID     = "REPLACE_PROJECT_ID";
const GOAL_ID        = "REPLACE_GOAL_ID";
const ROUTINE_ID     = "REPLACE_ROUTINE_ID";
const ISSUE_ID       = "REPLACE_ISSUE_ID";
const WORKSPACE_ID   = "REPLACE_WORKSPACE_ID";
const CLI_AUTH_ID    = "REPLACE_CLI_AUTH_ID";
const BOARD_CLAIM    = "REPLACE_BOARD_CLAIM_TOKEN";
const OUT            = "/path/to/paperclip-docs/docs/user-guides/screenshots/light";

const shots: [string, string][] = [
  // Agents
  [`/${SLUG}/agents/all`,                                    `${OUT}/agents/list.png`],
  [`/${SLUG}/agents/new`,                                    `${OUT}/agents/new.png`],
  [`/${SLUG}/agents/${AGENT_ID}/dashboard`,                  `${OUT}/agents/dashboard.png`],
  [`/${SLUG}/agents/${AGENT_ID}/instructions`,               `${OUT}/agents/instructions.png`],
  [`/${SLUG}/agents/${AGENT_ID}/skills`,                     `${OUT}/agents/skills.png`],
  [`/${SLUG}/agents/${AGENT_ID}/configuration`,              `${OUT}/agents/configuration.png`],
  [`/${SLUG}/agents/${AGENT_ID}/runs`,                       `${OUT}/agents/runs.png`],
  [`/${SLUG}/agents/${AGENT_ID}/budget`,                     `${OUT}/agents/budget.png`],

  // Costs (overview only via URL; other tabs need manual clicks)
  [`/${SLUG}/costs`,                                         `${OUT}/costs/overview.png`],

  // Issues & inbox
  [`/${SLUG}/issues`,                                        `${OUT}/issues/list.png`],
  [`/${SLUG}/inbox/mine`,                                    `${OUT}/issues/inbox.png`],
  [`/${SLUG}/inbox/unread`,                                  `${OUT}/issues/inbox-unread.png`],
  [`/${SLUG}/inbox/requests`,                                `${OUT}/issues/inbox-requests.png`],
  [`/${SLUG}/issues/${ISSUE_ID}`,                            `${OUT}/issues/detail-chat.png`],

  // Projects
  [`/${SLUG}/projects`,                                      `${OUT}/projects/list.png`],
  [`/${SLUG}/projects/${PROJECT_ID}/overview`,               `${OUT}/projects/overview.png`],
  [`/${SLUG}/projects/${PROJECT_ID}/issues`,                 `${OUT}/projects/issues.png`],
  [`/${SLUG}/projects/${PROJECT_ID}/workspaces`,             `${OUT}/projects/workspaces.png`],
  [`/${SLUG}/projects/${PROJECT_ID}/configuration`,          `${OUT}/projects/configuration.png`],
  [`/${SLUG}/projects/${PROJECT_ID}/budget`,                 `${OUT}/projects/budget.png`],

  // Goals
  [`/${SLUG}/goals`,                                         `${OUT}/goals/list.png`],
  [`/${SLUG}/goals/${GOAL_ID}`,                              `${OUT}/goals/detail.png`],

  // Routines
  [`/${SLUG}/routines`,                                      `${OUT}/routines/list.png`],
  [`/${SLUG}/routines/${ROUTINE_ID}`,                        `${OUT}/routines/detail.png`],

  // Execution workspaces
  [`/${SLUG}/execution-workspaces/${WORKSPACE_ID}/configuration`, `${OUT}/workspaces/configuration.png`],
  [`/${SLUG}/execution-workspaces/${WORKSPACE_ID}/runtime-logs`,  `${OUT}/workspaces/runtime-logs.png`],
  [`/${SLUG}/execution-workspaces/${WORKSPACE_ID}/issues`,        `${OUT}/workspaces/issues.png`],

  // Adapters (user)
  [`/instance/settings/adapters`,                            `${OUT}/adapters/list.png`],

  // Org
  [`/${SLUG}/org`,                                           `${OUT}/org/chart.png`],

  // Skills
  [`/${SLUG}/skills`,                                        `${OUT}/skills/list.png`],

  // CLI auth & board claim
  [`/cli-auth/${CLI_AUTH_ID}`,                               `${OUT}/auth/cli-auth.png`],
  [`/board-claim/${BOARD_CLAIM}`,                            `${OUT}/auth/board-claim.png`],
];

const client = await connect();
const page = await client.page("paperclip", { viewport: { width: 1440, height: 900 } });
for (const [path, out] of shots) {
  await page.goto(BASE + path, { waitUntil: "networkidle", timeout: 15000 }).catch(()=>{});
  await page.waitForTimeout(1200);
  await page.screenshot({ path: out });
  console.log("ok", path);
}
await client.disconnect();
EOF
```

批量采集后的手动补充步骤：

- **Costs：**依次点击 Budgets / Providers / Billers / Finance 标签，分别保存为 `costs/{budgets,providers,billers,finance}.png`。
- **问题详情：**在 `/{slug}/issues/{issueId}` 上点击 **Activity** 标签，保存为 `issues/detail-activity.png`。
- **Adapters：**点击 **Add adapter** 并保存为 `adapters/install.png`；展开已安装的适配器行并保存为 `adapters/detail.png`。
- **Skills：**点击 **Add skill**，分别截取初始表单（`skills/add-skill-dialog.png`）和确认状态（`skills/add-skill-dialog-confirm.png`）；打开技能行并保存为 `skills/file-inventory.png`。

---

## 目标路径和路由（v2026.817.0 发布批次）

此批次包含 v2026.817.0 发布的新界面。所有内容都会自动 seed——`seed-new-surfaces.mjs` 会提出决策、写入状态卡片摘要，并通过真实智能体运行创建机密提案，因此下方路由将显示真实内容，而非空状态。`routes.mjs` 是机器可读的权威版本；请保持两份内容同步。

### 决策（`screenshots/light/decisions/`）

需要启用实验标志 `enableDecisions`（由 `seed.mjs` 设置）。

| 文件 | 路由 | 备注 |
|---|---|---|
| `queue-overview.png` | `/{slug}/decisions` | 排序后的信息流。 |
| `decision-card.png` | `/{slug}/decisions` | 展开 “Ship dark mode…” 行。 |
| `triage-chips.png` | `/{slug}/decisions` | 仅裁剪待分类信息条。 |
| `toolbar-filters.png` | `/{slug}/decisions` | 点击 **Filter** 打开面板。 |
| `named-queue.png` | `/{slug}/decisions/queues/launch-blockers` | 包含两项内容的命名队列。 |
| `history-curtains.png` | `/{slug}/decisions` | 打开 **Decided** 折叠区。 |
| `empty-state.png` | `/{emptySlug}/decisions` | “You're all caught up”——需要第二家空公司。 |

### 状态卡片（`screenshots/light/status-cards/`）

需要启用 `enableStatusCards`，且内置 Summarizer 处于 `ready` 状态。Seed 会将 Summarizer 切换为长期运行的 `process` 适配器并清除其 `pausedAt`，随后以智能体身份写入每张卡片编译后的查询和摘要。

| 文件 | 路由 | 备注 |
|---|---|---|
| `board.png` | `/{slug}/status` | 两张 Fresh 磁贴。 |
| `new-card-dialog.png` | `/{slug}/status` | 点击 **New card**。 |
| `detail-summary.png` | `/{slug}/status/{cardId}` | Drawer，Summary 标签。 |
| `detail-settings.png` | `/{slug}/status/{cardId}` | Settings 标签。 |
| `detail-watched-issues.png` | `/{slug}/status/{cardId}` | Watched issues 标签。 |
| `detail-history.png` | `/{slug}/status/{cardId}` | History 标签。 |

### 机密提案（`screenshots/light/secrets/`）

| 文件 | 路由 | 备注 |
|---|---|---|
| `proposals-tab.png` | `/{slug}/company/settings/secrets` | 点击 **Proposals** 标签；分别有一项待处理、已批准和已拒绝提案。 |

### 聊天式任务（`screenshots/light/task-chat/`）

单独采集。`enableTaskChatRedesign` 是实例级设置，会替换普通任务页面；如果不关闭，所有经典问题/任务/工作模式截图都会变成新版聊天界面。`run.mjs` 会启用该标志，只拍摄 `phase: "task-chat"` 目标，然后再关闭标志。

| 文件 | 路由 | 备注 |
|---|---|---|
| `thread.png` | `/{slug}/issues/{issueId}` | 对话视图。 |
| `composer-modes.png` | `/{slug}/issues/{issueId}` | 打开 Agent · Plan · Ask 选择器。 |
| `side-pane.png` | `/{slug}/issues/{issueId}` | Properties · Plan · Artifacts 侧边栏。 |

---

## 采集完成后

- 检查所有 PNG 中均未显示真实用户数据（头像、公司名称、任务标题、电子邮件）。
- 将新增 PNG 与 `docs/user-guides/guides/administration/` 中的 Markdown 文件一并提交。

### 实验性 GitHub 审查机器人

使用真实设置组件和仅供浏览器的演示响应；需同时采集两种主题。使用 `--only github-review` 运行截图流水线（参阅其 README）。

| 目标 | 设置阶段 |
|---|---|
| `github-review/connect` | 连接 GitHub App |
| `github-review/repositories` | 选择仓库 |
| `github-review/identity` | 连接你的账户 |
| `github-review/prompts` | 配置行为和审查指令 |
