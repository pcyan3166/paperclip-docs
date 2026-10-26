---
paperclip_version: v2026.1023.0
seo_title: 技能参考
seo_description: 公司技能的文件结构、安装流程、代理绑定、作用域、规范键、版本管理和故障排查参考。
---

# 技能参考

本文介绍公司技能的磁盘文件结构、安装流程、代理绑定方式、作用域规则、规范键、版本管理和故障排查。概念介绍及界面操作请参阅[技能指南](../guides/org/skills.md)；REST API 详情请参阅[代理 API → 技能](./api/agents.md#skills)。浏览 Paperclip 随附技能目录，请从[内置技能](./skills/bundled.md)和[可选技能](./skills/optional.md)索引开始；索引按类别列出所有技能，并链接到各技能完整参考页。

> **适配器注意事项。** 某些适配器（尤其是 `openclaw_gateway`）无法将技能文件推送到运行时。系统仍会记录分配，但实际同步模式会报告为 `unsupported`。详见[作用域规则](#5-作用域规则)。

## 1. 技能文件结构

技能是一个根目录含 `SKILL.md` 的文件夹，可附带 `references/`、`scripts/` 和 `assets/` 子目录，例如：

```
my-skill/
├── SKILL.md
├── references/
│   └── examples.md
├── scripts/
│   └── run.sh
└── assets/
    └── logo.png
```

```markdown
---
name: code-review
description: >
  Use when asked to review a pull request or code diff.
  Don't use when writing new code from scratch.
---

# Code Review

When reviewing code, check the following...
```

`SKILL.md` 由 YAML frontmatter 和 Markdown 正文组成。Paperclip 使用自己的精简 YAML 读取器（`server/src/services/company-skills.ts` 中的 `parseFrontmatterMarkdown`），支持扁平标量、嵌套对象和列表字面量，不支持完整 YAML 语法。

<a id="frontmatter-fields"></a>
### Frontmatter 字段

| 字段 | 要求 | 类型 | 说明 |
|---|---|---|---|
| `name` | 建议提供 | 字符串 | 面向用户的名称；缺失时使用 slug。 |
| `description` | 建议提供 | 字符串 | 代理首先读取的路由说明；支持 `>`、`|` 块标量。 |
| `slug` | 可选 | 字符串 | 稳定的 kebab-case 标识符；缺失时从名称或文件夹名称派生，并通过 `normalizeAgentUrlKey` 规范化。 |
| `key` / `skillKey` | 可选 | 字符串 | 覆盖规范键，见[命名冲突与解析](#7-命名冲突与解析)。 |
| `metadata` | 可选 | 对象 | 与技能一同持久化的任意记录；已识别的子字段见下表。 |

已识别的 `metadata` 子字段（均可选）：

| 路径 | 用途 |
|---|---|
| `metadata.skillKey` / `metadata.canonicalKey` / `metadata.paperclipSkillKey` | 在导入期间保留规范键。 |
| `metadata.paperclip.skillKey` / `metadata.paperclip.key` | 同上，位于 `paperclip` 嵌套对象中。 |
| `metadata.sourceKind` | 来源类型：`paperclip_bundled`、`github`、`skills_sh`、`url`、`local_path`、`project_scan`、`managed_local` 或 `catalog`。 |
| `metadata.sources[]` | skills.sh 等打包工具写入的来源描述符列表。每项可含 `kind`（`github-dir`、`github-file`、`url`）、`repo`、`path`、`commit`、`trackingRef`、`hostname`、`url`、`rawUrl`。 |
| `metadata.owner` / `metadata.repo` / `metadata.ref` / `metadata.trackingRef` | 更新检查所用的 GitHub 坐标。 |

`metadata` 中其他字段会原样往返保存。

<a id="body"></a>
### 正文

结束 frontmatter 的 `---` 后书写 Markdown。正文没有长度上限，但代理确定技能相关后会将完整正文载入上下文，因此请保持简洁，将长内容放在支持文件中。

<a id="supporting-files"></a>
### 支持文件

| 目录 | 清单类型 | 对信任级别的贡献 |
|---|---|---|
| `references/` | `reference` | 仅 Markdown。 |
| `scripts/` | `script` | 脚本/可执行文件。 |
| `assets/` | `asset` | 资源文件。 |

Paperclip 会通过 `classifyInventoryKind` 自动识别这些子目录中的文件。

技能整体**信任级别**取决于清单中最高的分类：`markdown_only` < `assets` < `scripts_executables`。若从项目工作区导入且 `SKILL.md` 位于仓库根目录，Paperclip 会切换到 `project_root` 清单模式，仅扫描 `references/`、`scripts/` 和 `assets/`，避免导入整个仓库。

<a id="example-skills-you-can-read"></a>
### 可供参考的技能示例

以下内置技能随 Paperclip 服务器一同发布（位于服务器旁的 `skills/` 目录中）：

- `paperclip`：基础心跳流程。
- `paperclip-board`：通过聊天以看板成员身份管理公司，包括创建公司、设置 CEO、制定招聘计划、管理代理、处理审批、监控任务、监督费用和审查工作成果。使用 `paperclipai board setup` 进行配置。
- `paperclip-create-agent`：遵循治理要求的招聘流程。
- `paperclip-converting-plans-to-tasks`：将计划转换为已指派的 issue，并正确设置专业分工、依赖关系和并行化。
- `para-memory-files`：基于 Tiago Forte 的 PARA 方法使用文件记忆，涵盖知识图谱、每日笔记、隐性知识，以及每周综合和回忆层。

社区示例见 [`paperclipai/companies`](https://github.com/paperclipai/companies)：

- `companies/skills/company-creator/SKILL.md`
- `companies/skills/readme-updater/SKILL.md`

这些示例是技能文件结构、frontmatter 风格和配套文件布局的规范参考。

### 内置 `paperclip` 技能中的工件上传

随 Paperclip 发布的 `paperclip` 技能带有工件上传助手，确保工作成果附加到 issue，而不是留在代理工作区。其 `SKILL.md` 的 **Generated Artifacts and Work Products** 部分要求：每当运行生成用户可检查的文件，都应在最终处理前上传到当前 issue。看板用户、审核者和云端操作者通常无法访问代理工作区，本地路径对他们没有用。

助手脚本相对技能安装目录位于 `scripts/paperclip-upload-artifact.sh`，参考文档为 `references/artifacts.md`。脚本接收文件路径及 `--title`、`--summary` 参数：

```bash
scripts/paperclip-upload-artifact.sh path/to/output.webm \
  --title "Walkthrough render" \
  --summary "Rendered walkthrough for review"
```

脚本读取运行环境变量 `PAPERCLIP_API_URL`、`PAPERCLIP_API_KEY`、`PAPERCLIP_COMPANY_ID`、`PAPERCLIP_TASK_ID` 和 `PAPERCLIP_RUN_ID`，将文件上传为 issue 附件，创建以附件为基础的工作成果（默认行为），并输出可用于最终评论的 issue 安全 Markdown 链接。上传端点详见[问题 API → 附件](./api/issues.md#attachments)。

### 内置技能中的监控器和观察者

代理若在评论中写“我已设置观察器，CI 完成后会回来检查”，这句话必须属实，否则 issue 会静默停滞。内置 `paperclip` 技能的 **Monitors and Watchers** 部分要求代理评论与 issue 上实际保存的监控状态一致。

运行（心跳）只是短暂执行窗口，结束后不会继续监视。能自动恢复 issue 的唯一机制是持久化的 **issue monitor**：issue 自身保存状态（`monitorNextCheckAt`、`monitorScheduledBy`），执行策略中的 `monitor` 对象则记录 `kind`、`serviceName`、`externalRef`、`timeoutAt` 和 `maxAttempts`。服务器调度器 `tickDueIssueMonitors` 会轮询 `monitorNextCheckAt` 已到期的 issue，并以 `PAPERCLIP_WAKE_REASON=issue_monitor_due` 再次唤醒受派代理。这是基于定时轮询，而非事件订阅：Paperclip 不会在 CI 或其他外部检查完成的瞬间得到通知，只会按计划唤醒代理，让它重新检查。

仅保存时间戳并不足以触发监控：issue 必须指派给代理（设置 `assigneeAgentId`），且不能有用户受派人（`assigneeUserId` 为 null），状态还必须是 `in_progress` 或 `in_review`。即时检查也执行相同条件，因此指派给用户、处于 `backlog`、`blocked` 或已关闭状态的 issue 上的监控不会触发。

技能要求代理遵循三项规则：

- **只有实际安排后才能宣称已设置观察器。** 在评论中描述监控不会创建监控。代理必须通过 `PATCH /api/issues/{id}` 设置 `executionPolicy.monitor.nextCheckAt`（以及 `kind`、`serviceName`、`externalRef`、`timeoutAt`、`maxAttempts`），再确认 issue 符合上述条件且确实返回非空 `monitorNextCheckAt`。也可通过 `POST /api/issues/{id}/monitor/check-now` 即时检查。
- **用可核实的信息描述监控**：监控类型、下次检查时间、尝试次数和超时边界，不能只说“观察器会唤醒我”。代理若说不出这些信息，就不能暗示已经安排监控。
- **不得在将任务标记为 `done` 时暗示仍有活动监控。** `done` 表示不再跟进该 issue，与持续监控相矛盾。确实还需复查时，应保持 `in_progress` 或 `in_review` 并安排监控，不要关闭 issue。

这些要求不依赖代理自觉。处置保护会在没有有效审核路径时，以 `invalid_issue_disposition` 拒绝代理将 issue 移至 `in_review`；有效路径包括交互、审批、人工审核者、类型化执行参与者或实际安排的监控（含有效 `monitorNextCheckAt`）。恢复分类器会将没有活动唤醒路径的 `in_review` 标记为 `in_review_without_action_path`。

Issue monitor 与[任务看门狗](../guides/projects-workflow/task-watchdogs.md)是独立机制；后者由你附加到 issue，用于再次检查已静止的子树。

### 内置技能近期指导

- **唤醒上下文随提示提供。** 适配器会将唤醒负载（精简 issue 摘要及触发运行的一批新评论）作为运行提示中的章节交给代理，技能要求先读此章节；代理不再从单独环境变量查找它。
- **开放问题使用文本框。** 需要用户自由输入时，技能要求提供完整 `ask_user_questions` 负载，用 `answerMode: "text"` 的 `questionSet` 问题渲染文本框，而不是为非选择题虚构选项。只有至少有两个有意义替代方案的真实决策才使用选项卡片。
- **从 Paperclip Runner 招聘。** 原生 Runner 代理若有 `hire_agent` 工具，API 参考要求使用它。新队友继承调用方已验证的 Runner、模型、权限设置、默认环境和托管 AI 连接，但有自己的指令；调用方的密钥、工作区路径、会话和指令不会复制。REST API 对应招聘请求中的 `"inheritRuntimeFrom": "caller"`，此时不能同时提供非空 `adapterConfig`、`runtimeConfig` 或显式 `defaultEnvironmentId`。招聘权限和审批策略仍适用。
- **Paperclip Runner 上的 `paperclip-create-agent`。** Runner 代理没有 `PAPERCLIP_API_URL` 或 `PAPERCLIP_API_KEY`，因此技能要求先用 `get_task_context` 和 `list_agents` 获取上下文，再通过 `search_api` 查找招聘端点、用 `call_api` 调用。如果 API 工具不可用，代理应说明需为公司启用 Runner API 工具、在对话中保留拟议招聘内容，并且不得声称已经创建代理。
- **运行框架已检出时不要重复检出。** 若运行的 **Paperclip Wake Payload** 或 **Paperclip Resume Delta** 表明框架已为当前运行检出当前 issue，技能要求直接继续，不要再次调用 checkout。这只适用于该次运行中的当前 issue；代理切换到其他任务仍须先检出，且不能根据 issue 状态或此前运行推断当前已检出。
- **完成助手随技能发布。** `scripts/paperclip-issue-update.sh` 会写入状态和评论并核实写入成功，现已包含在已安装的 `paperclip` 技能内。代理应从技能自身安装目录运行它，而不是任务工作区；路径不可用时应使用经核实的 `PATCH` 作为后备方案，而不是搜索磁盘。
- **工件助手状态保存在工作区。** `scripts/paperclip-upload-artifact.sh` 将临时响应和重试回执保存在工作区私有目录 `.paperclip/artifact-helper`，因此能在无法访问系统临时目录的 CLI 沙箱运行。设置 `PAPERCLIP_HELPER_STATE_DIR` 可改用其他目录。
- **附带反馈。** `paperclip` 技能提及并列的 `complain` 和 `suggestion-box` 技能，并鼓励代理在工作确有需要时安静地使用一次，不得打断任务。详见[代理投诉与建议](../administration/agent-commentary.md)。
- **审核交接必须有真实卡片。** 代理创建需由人员审核或合并的 PR 时，技能要求把 PR 链接放入工作成果的 `url`，创建明确写出所需操作、且仅面向人工的交互，并将任务置于 `in_review`。单靠 `needs_board_review` 标志或定时监控不会向任何人显示审核卡片。

## 2. 安装流程

技能安装在**公司**层级。安装后，该公司中的任何代理都可以被指派此技能。

### 端点

| 操作 | 端点 |
|---|---|
| 列出技能 | `GET /api/companies/{companyId}/skills` |
| 技能详情（含用量） | `GET /api/companies/{companyId}/skills/{skillId}` |
| 读取技能文件 | `GET /api/companies/{companyId}/skills/{skillId}/files?path=SKILL.md` |
| 更新文件（可编辑技能） | `PATCH /api/companies/{companyId}/skills/{skillId}/files`，见[安全文件更新](#安全文件更新) |
| 查询状态（GitHub 管理） | `GET /api/companies/{companyId}/skills/{skillId}/update-status` |
| 拉取最新提交 | `POST /api/companies/{companyId}/skills/{skillId}/install-update` |
| 删除 | `DELETE /api/companies/{companyId}/skills/{skillId}` |
| 创建空技能（Paperclip 管理） | `POST /api/companies/{companyId}/skills` |
| **从来源导入** | `POST /api/companies/{companyId}/skills/import` |
| **扫描项目工作区** | `POST /api/companies/{companyId}/skills/scan-projects` |

变更路由要求调用者具有 `agents:create` 权限，或调用代理的 `permissions.canCreateAgents=true`。

### 安全文件更新

`PATCH /api/companies/{companyId}/skills/{skillId}/files` 用于写入可编辑技能中的一个文件。请求体含 `path`、`content`，以及可选的 `encoding`（`utf8` 或 `base64`）和 `executable`。另有两个可选字段可避免覆盖他人编辑：

| 字段 | 作用 |
|---|---|
| `expectedVersionId` | 最近读取时技能的 `currentVersionId`。若技能此后已有更新，写入会以 `409` 和 *"Skill version changed. Read the current version before retrying."* 拒绝。重新读取当前版本，并在最新内容上应用修改。 |
| `idempotencyKey` | 1–240 个字符。相同键重试完全相同请求会返回原结果，不会产生第二个版本或活动记录。不同输入重用该键会返回 `409` 和 *"Skill file idempotency key was used with different inputs"*。 |

未提供 `idempotencyKey` 时，路由照常返回更新后的文件；提供后则返回精简回执：

```json
{
  "skillId": "…",
  "path": "SKILL.md",
  "versionId": "…",
  "studioPath": "/skills/studio/…"
}
```

无论哪种响应，写入都要经过 `skills.edit` 策略检查（包括完全相同请求的重试）；仅当字节或可执行标志发生变化时创建新版本，并记录 `company.skill_file_updated`。原生 Runner 的 **Update skill** 工具编辑技能时也调用此路由，详见[让代理改进现有技能](../guides/org/skills.md#letting-an-agent-improve-an-existing-skill)。

<a id="native-runner-skill-tools"></a>
### 原生 Runner 技能工具

使用原生 Runner 的代理可以用 `create_skill` 保存可重用流程，或用 `update_skill` 改进现有可编辑技能。这些工具适用于 Auto（`standard`）和技能测试工作，不适用于 Ask 或尚未接受的 Plan 工作。它们使用公司技能 API 和现有策略；创建技能只会将其加入技能库，不会分配给代理。

`update_skill` 接受以下字段：

| 字段 | 用途 |
|---|---|
| `skillId` | 现有技能的 UUID。 |
| `markdown` | 完整主 `SKILL.md`，必须包含有效的名称和描述 frontmatter，以及非空正文。 |
| `expectedVersionId` | 编辑前读取的 `currentVersionId`。 |
| `idempotencyKey` | 长度为 1–240 个字符的重试键。 |

例如：

```json
{
  "skillId": "<skill UUID>",
  "expectedVersionId": "<currentVersionId>",
  "markdown": "---\nname: release-review\ndescription: Review release notes.\n---\n\n# Review\nCheck each release note against its change.\n",
  "idempotencyKey": "release-review-update-1"
}
```

响应丢失时，使用完全相同的输入和键重试以取得原回执。版本冲突时，读取当前技能、重新应用修改并使用新键。相同键搭配不同输入会冲突；即使是完全相同请求的重试，策略也会重新检查。

`create_skill` 接受 `name`、`description`、`markdown`、`idempotencyKey`，以及可选 `slug`。名称须为小写连字符格式；完整文件中的名称和描述必须与输入一致；若提供 slug，必须与名称相同。公司、代理、任务和运行身份来自已认证运行。创建成功后，任务线程会显示 **Skill created** 卡片，详见[代理在任务期间创建的技能](../guides/org/skills.md#skills-your-agents-create-during-a-task)。实现参考：[update 工具契约](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/packages/paperclip-runner/src/protocol-actions/update-skill.ts)及[工具验证与 REST 绑定](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/server/src/services/skill-tools.ts)。

<a id="import-accepted-sources"></a>
### 导入：支持的来源

`POST /api/companies/{companyId}/skills/import` 接受字段 `source`，由 `parseSkillImportSourceInput` 解析。以下格式均有效：GitHub 仓库网址、`tree/main/skills/paperclip` 目录网址、指向 `SKILL.md` 的 `blob` 网址、`owner/repo` 或 `owner/repo/skill` 简写、skills.sh 网址、`npx skills add owner/repo --skill name` 命令、原始 `SKILL.md` URL，以及本地技能目录路径。

```json
{ "source": "https://github.com/paperclipai/paperclip" }
{ "source": "https://github.com/paperclipai/paperclip/tree/main/skills/paperclip" }
{ "source": "https://github.com/paperclipai/paperclip/blob/main/skills/paperclip/SKILL.md" }
{ "source": "paperclipai/paperclip" }
{ "source": "paperclipai/paperclip/paperclip-create-agent" }
{ "source": "https://skills.sh/paperclipai/paperclip/paperclip-create-agent" }
{ "source": "npx skills add paperclipai/paperclip --skill paperclip-create-agent" }
{ "source": "https://example.com/raw/SKILL.md" }
{ "source": "/Users/me/code/my-project/skills/code-review" }
```

解析规则如下：

- `owner/repo` 和 `owner/repo/skill` 会视为 GitHub 引用。
- `https://skills.sh/...` 与 `npx skills add ...` 会解包为底层 GitHub URL，但技能记录仍使用 `sourceType: skills_sh`，且会保留原始定位符。
- `tree/<ref>` 和 `blob/<ref>` 使用指定 ref；裸仓库 URL 使用默认分支。
- 本地路径可以指向单个 `SKILL.md`、含有一个技能文件的目录或含多个技能的目录；目录下所有 `SKILL.md` 都会导入。
- 本地路径（跟随符号链接后）必须位于获准根目录：公司的托管技能目录、注册的项目工作区，或服务器管理的项目检出目录。其他位置会以 `403`、代码 `skill_workspace_boundary_denied` 和 *"Local skill source is outside approved company workspace roots"* 拒绝。
- `github.com` 上的仓库、`tree/...`、`blob/...` URL 及 `owner/repo` 简写会归入[GitHub 技能来源](#github-技能来源)：创建来源（若同仓库和分支已有来源则复用），并选中匹配的技能。响应仅含 `imported`、`warnings`，活动记录为 `company.skill_source_refreshed`。skills.sh 格式（包括 `owner/repo/skill`）、gist 和其他 URL 使用下述行为。

### 导入过程

1. 解析来源（解析输入、读取元数据、搜索 `SKILL.md`）。
2. 对每项技能解析 frontmatter、派生 `slug` 和规范 `key`、遍历文件清单并分类，进而确定 `trustLevel`。
3. 将技能写入 `companySkills` 表；每个 `(companyId, key)` 一行，见[命名冲突与解析](#7-命名冲突与解析)。
4. 对目录类型来源，将文件物化到 `<paperclipInstanceRoot>/skills/{companyId}/__catalog__/<runtimeName>/`，供适配器读取。
5. 记录 `company.skills_imported` 活动，并发送 `skill_imported` 遥测事件。

响应含 `imported`、`warnings` 和解析后的来源元数据。

### 项目扫描

`POST /api/companies/{companyId}/skills/scan-projects` 会遍历公司内各项目的本地工作区，也可通过 `projectIds` / `workspaceIds` 过滤，并在以下已知位置查找 `SKILL.md`：

- `skills/`、`skills/.curated/`、`skills/.experimental/` 和 `skills/.system/`。
- `.agents/skills/`、`.agent/skills/`、`.augment/skills/`、`.claude/skills/`、`.codebuddy/skills/`、`.commandcode/skills/`、`.continue/skills/`、`.cortex/skills/`、`.crush/skills/`、`.factory/skills/`、`.goose/skills/`、`.junie/skills/`、`.iflow/skills/`、`.kilocode/skills/`、`.kiro/skills/`、`.kode/skills/`、`.mcpjam/skills/`、`.vibe/skills/`、`.mux/skills/`、`.openhands/skills/`、`.pi/skills/`、`.qoder/skills/`、`.qwen/skills/`、`.roo/skills/`、`.trae/skills/`、`.windsurf/skills/`、`.zencoder/skills/`、`.neovate/skills/`、`.pochi/skills/` 和 `.adal/skills/`。
- 工作区根目录（仅当其中包含 `SKILL.md` 时）。

扫描不会破坏现有内容，返回 `imported`、`updated`、`skipped`、`conflicts` 和 `warnings`。若 slug 或 key 已指向不同来源，会列入 `conflicts` 供人工检查，而不是静默覆盖。

### 存储

可编辑的 Paperclip 托管技能位于 `<paperclipInstanceRoot>/skills/{companyId}/<slug>/`。只读来源（GitHub、skills.sh、URL）的 Markdown 正文存于数据库行，仅在适配器需要磁盘文件时才物化到临时位置。

内置技能（服务器 `skills/` 目录中的技能）会在每次列表请求时重新导入（`ensureBundledSkills`），且不能编辑或删除；安装同一 Paperclip 版本时会重新生成。GitHub 同步技能是例外：每次刷新都会存储扫描提交时包内所有文件的完整快照，代理运行时无需从 GitHub 获取文件。

<a id="github-skill-sources"></a>
### GitHub 技能来源

技能来源将公司关联到一个 GitHub 仓库和分支，记录所选技能包，并可一起刷新。界面位置为 **Skills → Sources**，详见[从 GitHub 仓库同步技能](../guides/org/skills.md#sync-skills-from-a-github-repository)。

| 操作 | 端点 |
|---|---|
| 列出可选仓库 | `GET /api/companies/{companyId}/skill-sources/repositories` |
| 列出来源 | `GET /api/companies/{companyId}/skill-sources` |
| 扫描仓库中的技能 | `POST /api/companies/{companyId}/skill-sources/discover` |
| 预览已扫描包中的文件 | `POST /api/companies/{companyId}/skill-sources/preview` |
| 创建来源并导入所选技能 | `POST /api/companies/{companyId}/skill-sources` |
| 来源详情 | `GET /api/companies/{companyId}/skill-sources/{sourceId}` |
| 修改选择并刷新 | `PATCH /api/companies/{companyId}/skill-sources/{sourceId}` |
| 从跟踪分支刷新 | `POST /api/companies/{companyId}/skill-sources/{sourceId}/refresh` |
| 断开连接 | `DELETE /api/companies/{companyId}/skill-sources/{sourceId}` |

仓库列表合并调用者可用的所有 GitHub 连接并去重。每次读取 GitHub 时都会重新检查调用者权限；公开仓库无需连接也可读取。`repositoryUrl` 必须是 HTTPS 的 `github.com` 仓库或分支 URL（`https://github.com/owner/repo` 或 `https://github.com/owner/repo/tree/<branch>`）；裸仓库 URL 使用默认分支。

**发现。**`discover` 接受 `repositoryUrl` 以及可选的 `trackingRef` 和 `connectionId`。它返回解析后的 `commitSha`、`trackingRef`、`candidates`（每个候选项包含 `path`、`name`、`description`、`fileCount`，以及可能的 `error` 或 `warnings`）和总体 `warnings`。若要改用流式响应，请发送 `Accept: application/x-ndjson`：扫描期间会收到 `progress` 和 `candidate` 事件，最后收到携带发现结果的 `complete` 事件，或 `error` 事件。

创建来源时，除发现字段外还需传入本次扫描得到的 40 字符 `commitSha`、要导入的 `selectedPaths`（即 `SKILL.md` 路径）和可选 `excludedFolders`。成功返回 `201` 以及 `source`、`imported`、`updated`、`unchanged` 和 `warnings`。相同仓库和分支已存在来源时返回 `409`，应管理现有来源。

`PATCH` 接受来源当前 `revision`（乐观锁）、`selectedPaths`、`excludedFolders` 和可选 `connectionId`，随后刷新。revision 已过期会返回 `409` 和 *"This source changed. Reload before saving your selection."* 保存选择也会重新连接已断开的来源。

**刷新行为。**

- 选择且通过验证的包会被导入或更新。仅当包内文件发生变化时才会创建新版本；技能的 `id`、key、文件夹和代理分配保持不变。
- 上游发现的新包会记录为 `new`，需选中后才会导入。
- 取消选择的已安装技能会停止同步，但仍保留在公司中。上游已删除的包对应的技能也会保留，并标记为已移除。
- 每个来源同时只能运行一次刷新；并发刷新会返回 `409`。刷新失败时，已安装技能会保留最后一个正常版本，错误会保存到来源的 `lastError`。
- 每次刷新都会记录为 `company.skill_source_refreshed`；断开连接则记录为 `company.skill_source_disconnected`。

断开来源会停止同步该来源的所有技能，但保留已安装项。若需彻底删除，应逐个删除技能。扫描、预览和创建检查 `skills.import` 策略；刷新检查 `skills.update`；改选和断开检查 `skills.edit`。策略资源使用来源类型 `git` 和仓库 URL 作为定位符，详见[公司技能策略](./api/company-skill-policy.md)。同步技能为只读；修改时请编辑仓库并刷新，或将其分叉为可编辑副本。

<a id="3-app-shipped-catalog"></a>
## 3. 应用内置目录

除 GitHub、skills.sh、URL 和本地文件夹导入外，Paperclip 还在应用内置有可直接安装的**技能目录**。安装后，Paperclip 会将公司副本与随应用发布的版本对比，检查更新、审计文件，并在需要时重置回固定来源版本。

目录技能有两种 `CatalogSkillKind`：

| 类型 | 含义 | 示例 |
|---|---|---|
| `bundled` | 应用基线工具集的一部分。 | `doc-maintenance`、`issue-triage`、`task-planning`、`qa-acceptance`、`github-pr-workflow`、`wireframe`、`summarize-status`、`status-card-query` |
| `optional` | 按需加入的扩展技能。 | `agent-browser`、`release-announcement`、`simplified-english`、`design-critique`、`last30days`、`prepare-mcp-integration` |

目录技能还带有可筛选的 `category`。例如 `product` 类 `wireframe` 技能教代理生成低保真黑白 UI SVG 线框图，推荐给 `designer`、`product`、`engineer` 角色；`research` 类 `last30days` 技能研究最近 30 天 Reddit、X、YouTube 及其他网站上的相关讨论，推荐给 `researcher`、`marketer`、`product-manager`、`analyst`。`wireframe` 虽属内置类别，但 `defaultInstall` 为 `false`，仅在你选择时安装，不会强制加到每个代理。

基线工具集中还包含面向管理者的 `paperclip-operations` 类 `summarize-status`，推荐给 `general` 和 `manager`。它将项目、工作区概览或单个项目工作区的当前状态写成简短摘要，并将其作为新版本写回摘要字段。摘要不是任务清单，而是行动优先：开头列出现在需要采取的 1–3 项具体行动以解除阻塞，每项说明要做什么、为何阻碍进度，并内嵌链接；之后用一两段简明对话式文字说明现状。若确实没有待办事项，就如实说明并指出值得关注的下一项，不用填充内容。它的 `defaultInstall` 同样为 `false`。

同属 `paperclip-operations` 类的 `status-card-query`（也推荐给 `general`、`manager`）支持实验性[状态卡](../experimental/status-cards.md)。用户用自然语言描述状态卡（如“本周更新的受阻或审核中的发布工作”），技能负责通过公开 API 创建和维护卡片（每个代理最多 20 张，每条提示最多 4,000 字符），并在编译模式下由 Summarizer 将文字转换为范围受限的 `CompanySearchQuery` 对象（按状态、优先级、受派人、项目、标签和近期程度筛选，而不是关键词堆砌），再由同一次已指派运行写入首个摘要。只有启用 `enableStatusCards` 后才可用，且 `defaultInstall` 为 `false`。详情见 [Status Card Query](./skills/bundled/paperclip-operations/status-card-query.md)。

大部分目录技能随应用发布；少数是**引用型技能**，指向固定在某个提交的外部 GitHub 仓库，由应用获取文件。例如 `last30days` 来自固定提交的 `github.com/mvanhorn/last30days-skill`。其安装和管理方式与其他目录技能相同；只有 `source` 字段会标出原始 owner、repo、固定的 `ref` / `commit` 和可浏览 URL。固定提交意味着它仍有逐字节精确、受版本管理的保证。

安装后，目录技能成为普通公司技能行，但会标记为**目录管理**：`sourceType` 和 `metadata.sourceKind` 均为 `catalog`；目录类型（`bundled` / `optional`）保存在 `metadata.catalogKind`，精确固定的来源字节摘要存于 `metadata.originHash`。

> 浏览目录的路由只读，仅需认证。安装、审计、更新和重置会更改公司技能库，因此需要 `agents:create`（或 `permissions.canCreateAgents=true`），与其他技能变更路由相同。

### 浏览目录

目录对所有公司相同，因此浏览路由不限定公司：

| 操作 | 端点 |
|---|---|
| 列出目录技能 | `GET /api/skills/catalog` |
| 查看某项目录技能 | `GET /api/skills/catalog/{catalogId}` |
| 列出目录技能文件 | `GET /api/skills/catalog/{catalogId}/files` |

列表路由接受三个可选查询过滤器：

- `kind`：`bundled` 或 `optional`。
- `category`：精确匹配技能类别。
- `q`：自由文本查询，会匹配技能的 id、key、slug、name、description、category、kind、推荐角色和标签。

每个目录条目（`CatalogSkill`）包含 `id`、`key`、`kind`、`category`、`slug`、`name`、`description`、`trustLevel`、`compatibility`、`defaultInstall`、`recommendedForRoles`、`requires`、`tags`、文件清单 `files[]`（每个文件包含 `path`、`kind`、`sizeBytes`、`sha256`）以及 `contentHash`，后者是随应用发布技能的逐字节精确身份标识。来自外部仓库的引用型技能还会包含可选的 `source` 对象，描述字节来源：`type`（`github`）、`hostname`、`owner`、`repo`、固定的 `ref` 和 `commit`、仓库内的 `path` 以及可浏览的 `url`。

文件路由每次返回一个文件，默认读取 `SKILL.md`；传入 `?path=references/example.md` 可读取清单中的其他文件。资源文件无法预览。

### 安装目录技能

```http
POST /api/companies/{companyId}/skills/install-catalog
```

```json
{
  "catalogSkillId": "<catalog id, key, or unique slug>",
  "slug": "my-doc-maintenance",
  "force": false
}
```

请求字段：

- `catalogSkillId`（必填）：按目录 id、key 或目录中唯一的 slug 解析。
- `slug`（可选）：覆盖技能在公司库中使用的 slug。省略或设为 `null` 时，使用目录 slug。
- `force`（可选）：覆盖已经占用目标 slug/key 的现有行。

**安装不会将技能附加给任何代理。** 它只会将技能加入公司库，与导入相同。之后再按其他技能的方式分配，见[将技能分配给代理](#4-将技能分配给代理)。响应会给出 `action`（`created`、`updated` 或 `unchanged`）、生成的公司 `skill`、来源 `catalogSkill` 和 `warnings`。全新安装返回 `201`，就地刷新返回 `200`。

### 更新目录技能

目录管理技能使用与 GitHub 技能相同的 `GET /skills/{skillId}/update-status` → `POST /skills/{skillId}/install-update` 流程，但会与应用当前发布版本比较。发现较新版本时：

```http
POST /api/companies/{companyId}/skills/{skillId}/install-update
```

```json
{ "force": false }
```

两种情况会阻止更新：

- **软阻止——本地修改。**若安装字节与固定的 `originHash` 不匹配，更新会暂停，以免悄悄丢失修改。重新请求时传入 `{ "force": true }`，即可丢弃本地修改并安装新版本。
- **硬性阻止——审计发现。**若技能审计结果为 `fail`（见下文），更新会被直接阻止。`force` **不能**覆盖硬性阻止。

### 信任前先审计

```http
POST /api/companies/{companyId}/skills/{skillId}/audit
```

审计只检查已安装技能的字节，**不会执行代码**：

| `verdict` | 含义 |
|---|---|
| `pass` | 没有发现问题。 |
| `warning` | 只有软性发现，例如技能含脚本、引用网络命令，或字节与 `originHash` 不符。 |
| `fail` | 至少有一项硬阻止发现，例如远程获取并执行、窃取密钥模式、文件过大或非文本、缺失或无效的 `SKILL.md`。 |

结果还包含去重后的发现代码 `codes`、`installedHash`、`originHash` 和 `scanVersion`。`fail` 会硬性阻止 `install-update`。仅支持本地路径和目录管理技能的审计。

### 重置到固定来源

```http
POST /api/companies/{companyId}/skills/{skillId}/reset
```

```json
{ "force": false }
```

重置会将目录管理技能恢复为安装时对应的逐字节精确来源版本，适用于放弃本地编辑并恢复随附版本。仅目录管理技能支持重置。对应的 `paperclipai skills` 浏览、安装、审计、更新和重置命令见 [Skills CLI 命令](./cli/skills.md)。

<a id="4-assigning-skills-to-agents"></a>
## 4. 将技能分配给代理

技能必须先安装到公司层级，之后才能附加到代理。附加技能有两种方式。

### 招聘时：`desiredSkills`

`POST /api/companies/{companyId}/agents` 和 `POST /api/companies/{companyId}/agent-hires` 的请求体都可选传入 `desiredSkills` 数组：

```json
{
  "name": "Backend Engineer",
  "adapterType": "claude_local",
  "desiredSkills": ["paperclip", "code-review", "deploy-runbook"]
}
```

每个元素可以是：

- 公司技能 **UUID**（`id`）；
- 规范 **key**（`paperclipai/paperclip/paperclip`）；
- 仅当公司内 slug 唯一时才可使用的 **slug**（`code-review`）。

服务器会将每项引用解析为规范 key，并将收到的列表原样写入 `adapterConfig.paperclipSkillSync.desiredSkills`；期望集合恰好等于提交的内容，不会额外并入其他键。未知或有歧义的引用会以 `Invalid company skill selection (...)` 拒绝请求。

### 招聘后同步：`POST /api/agents/{agentId}/skills/sync`

```json
{
  "desiredSkills": ["paperclip", "code-review"]
}
```

该路由会让代理的技能附加项与请求完全一致：列表内技能会被附加，不在列表内的会被移除。空列表会移除全部技能。

每项也可以传对象而不是字符串（`{ "key": "...", "versionId": "..." }`），将代理固定到该技能的特定版本。服务器只检查该版本是否属于指定技能；任何非空 `versionId` 还要求启用 `enableBetaSkills`。界面比 API 限制更严格，只为内置 `paperclip` 技能版本提供选择器，见[内置 `paperclip` 技能的 Beta 版本](#内置-paperclip-技能的-beta-版本)。

响应为 `AgentSkillSnapshot`：

```json
{
  "adapterType": "claude_local",
  "supported": true,
  "mode": "persistent",
  "desiredSkills": ["paperclipai/paperclip/paperclip", "company/<id>/code-review"],
  "entries": [
    {
      "key": "paperclipai/paperclip/paperclip",
      "runtimeName": "paperclip",
      "desired": true,
      "managed": true,
      "state": "installed",
      "origin": "company_managed"
    }
  ],
  "warnings": []
}
```

条目会注明 `key`、`runtimeName`、`desired`、`managed`、`state`、`origin` 等信息。完整结构见 `packages/shared/src/validators/adapter-skills.ts`。

### 检查当前状态

读取当前状态可用：

```http
GET /api/agents/{agentId}/skills
```

此路由返回同一快照，不修改任何内容。未实现 `listSkills` 的适配器会根据持久化偏好构造快照，并标记 `mode: "unsupported"`。
<a id="5-scoping-rules"></a>
## 5. 作用域规则

**技能作用于公司，而非组织。** 技能存放在公司层级，同一公司内的任何代理都可以被分配已安装技能。系统没有内置团队或角色级作用域，粒度由每个代理 `desiredSkills` 中的 key 决定。

**公司之间不共享。** 公司 A 安装的技能对公司 B 不可见。要在两个公司使用同一技能，必须分别安装来源；各自获得独立的 `companySkills` 行和独立版本。

**适配器同步模式决定运行时可见性。** `GET /api/agents/{agentId}/skills` 返回的快照包含 `mode`：

| 模式 | 行为 |
|---|---|
| `persistent` | 适配器将技能文件写入代理工作目录，并在运行之间保留；多数本地适配器使用此模式。 |
| `ephemeral` | 适配器为每次运行生成技能文件，之后清理；沙箱适配器默认使用此模式。 |
| `unsupported` | Paperclip 记录分配，但无法向运行时推送文件。`openclaw_gateway` 等适配器属此类；需在远程运行时管理技能。 |

适配器通过 `requiresMaterializedRuntimeSkills` 声明物化策略，否则使用旧版硬编码集合：`cursor`、`gemini_local`、`opencode_local`、`pi_local`。

**内置技能位于技能库中。** `paperclip` 及其他 `paperclip_bundled` 技能会作为只读条目导入公司库，但不会再被强制添加进代理解析后的技能集合。`resolveDesiredSkillAssignment` 只保存同步请求中明确给出的目标技能。

**界面中的技能库技能与非托管技能。** Agent → Skills 标签页以两个分区和一个可折叠组显示技能：

- **已安装技能**：已选择给此代理的公司库技能；取消勾选会将其从期望集合移除。
- **其他技能**：可添加到此代理的公司库技能；勾选会将其加入期望集合。
- **“({N}) 个用户安装、非 Paperclip 管理的技能”**：默认折叠；显示适配器自行发现的只读技能（如主机上的全局技能包）。其来源为 `user_installed` 或 `external_unknown`，不能在此编辑。

## 6. 技能、插件和适配器

|  | **技能** | **插件** | **适配器** |
|---|---|---|---|
| 用途 | 代理按需读取的指令文档 | 添加 API 路由、界面或运行时服务的代码包 | Paperclip 与代理运行时之间的桥梁 |
| 格式 | 含 `SKILL.md` 及可选 `references/`、`scripts/`、`assets/` 目录的文件夹 | 按插件 SDK 构建的 Node 包 | 含 `executeRun`、`listSkills`、`syncSkills` 等的模块 |
| 作用层级 | 公司技能库 | 实例级 `~/.paperclip/adapter-plugins/` | 内置于服务器或注册为外部适配器 |
| 加载时机 | 代理运行开始时，且路由描述匹配时 | 服务器启动时，挂载到 API 和 UI | 每次运行按代理的 `adapterType` 加载 |
| 编写者 | 能编辑公司技能库的人员 | 插件作者（可使用 [`paperclip-create-plugin` 技能](https://github.com/paperclipai/paperclip/blob/master/.agents/skills/paperclip-create-plugin/SKILL.md)） | 适配器作者，见[创建适配器](./adapters/creating-an-adapter.md) |
| 版本管理 | 固定到 Git 提交（GitHub/skills.sh）或实时更新（本地） | 由插件包版本固定 | 由 Paperclip 版本固定 |
| 错误影响 | 代理读取错误指令并产出不佳结果 | API 界面无法加载 | 运行无法启动 |

简单来说：**技能是交给聪明的人也能照着执行的文档；插件是服务器代码；适配器是运行时所用的通信协议。**

## 7. 命名冲突与解析

### 规范键

每个已持久化技能都有 `key`，即公司内唯一标识符，由 `deriveCanonicalSkillKey` 派生：

| 来源 | 键格式 | 示例 |
|---|---|---|
| Paperclip 内置 | `paperclipai/paperclip/{slug}` | `paperclipai/paperclip/paperclip` |
| GitHub / skills.sh | `{owner}/{repo}/{slug}` | `paperclipai/paperclip/paperclip-create-agent` |
| URL | `url/{host}/{hash}/{slug}` | `url/example.com/9f2a3b1c4d/code-review` |
| 本地路径（项目扫描、原始文件夹） | `local/{hash}/{slug}` | `local/4e8a91c2d3/code-review` |
| Paperclip 管理（界面创建） | `company/{companyId}/{slug}` | `company/5cbe79ee.../delegation-checklist` |

若来源提供显式 `key` / `metadata.skillKey`，以其为准。这能让 skills.sh 发布的技能在镜像之间保持稳定身份。

### 引用解析

传入技能引用（如 `desiredSkills` 或同步路由）时，服务器按 `resolveSkillReference` 的顺序查找：

1. 按 `id` 精确匹配 UUID。
2. 匹配规范化的 `key`。
3. 匹配规范化的 `slug`，但只有公司内恰好一项技能使用该 slug 时才成功。

slug 命中多项时返回 `Invalid company skill selection (ambiguous references: <slug>; ...)`；引用不存在时返回 `Invalid company skill selection (unknown references: <reference>; ...)`。需要脚本跨公司保持幂等时，优先使用规范 `key`，不要使用 slug。

```
Invalid company skill selection (ambiguous references: <slug>; ...)
```

```
Invalid company skill selection (unknown references: <reference>; ...)
```

### 项目扫描冲突

项目扫描发现的 `SKILL.md` 若规范 key 已被不同来源路径占用，不会覆盖现有行；响应会通过 `conflicts[]` 返回 `existingSkillId`、`existingSourceLocator` 和可读原因。若 slug 已被另一技能占用也相同处理。可删除其中一项，或修改新技能的 `name` / `slug` 后重新扫描以解决冲突。

### Slug 规范化

Slug 由 `normalizeAgentUrlKey` 规范化为小写、连字符分隔形式，不含空白或标点。`Code Review`、`code_review` 和 `Code--Review` 均会解析为 `code-review`。

## 8. 版本语义

行为取决于来源：

| 来源 | 是否固定 | 更新方式 | 说明 |
|---|---|---|---|
| `github` | 是，`sourceRef` 保存解析后的 commit SHA。 | `GET /skills/{id}/update-status` → `POST /skills/{id}/install-update` | `metadata.trackingRef` 保存分支或标签；将该引用的最新提交与固定 SHA 比较。 |
| `github`（同步来源） | 是，固定到最近扫描提交的快照。 | `POST /skill-sources/{sourceId}/refresh`，或对其任一技能调用 `install-update`。 | 属于[GitHub 技能来源](#github-技能来源)。`update-status` 比较分支最新提交与来源最后扫描提交；`install-update` 刷新整个来源。 |
| `skills_sh` | 是（底层解析到 GitHub）。 | 同 `github`。 | `sourceLocator` 保留原始 `skills.sh/...` URL，以维持徽标和标签。 |
| `url` | 否。 | 无。 | 视为某一时点的快照；重新导入 URL 可刷新。 |
| `local_path`（托管） | 实时。 | 无，读取时刷新文件。 | 位于 `<paperclipInstanceRoot>/skills/{companyId}/`，通过 `PATCH /skills/{id}/files` 编辑。 |
| `local_path`（项目扫描） | 实时。 | 重新执行扫描端点。 | `SKILL.md` 消失时会自动清理（`pruneMissingLocalPathSkills`）。 |
| `paperclip_bundled` | 固定到 Paperclip 版本。 | 升级 Paperclip。 | 每次列表请求都会重新导入，不能编辑。内置 `paperclip` 另含代理可固定的 Beta 版本，见[下文](#内置-paperclip-技能的-beta-版本)。 |
| `catalog` | 固定到随应用发布的目录版本（`metadata.originHash`）。 | `GET /skills/{id}/update-status` → `POST /skills/{id}/install-update`；通过 `POST /skills/{id}/reset` 恢复来源。 | 随应用发布的目录技能，更新时与应用版本比较，见[应用内置目录](#3-应用内置目录)。 |

### `update-status` 响应

`update-status` 对 GitHub 技能返回 `supported`、`reason`、`trackingRef`、`currentRef`、`latestRef` 和 `hasUpdate`。非 GitHub 来源返回 `supported: false` 及原因；对不支持的技能调用 `install-update` 会返回 `422 Unprocessable Entity`。

```json
{
  "supported": true,
  "reason": null,
  "trackingRef": "main",
  "currentRef": "9f2a3b1c4d8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a",
  "latestRef": "0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b",
  "hasUpdate": true
}
```

### 安装更新语义（`install-update`）

属于 GitHub 技能来源的技能若来源已断开或不再被选择，`update-status` 返回 `supported: false` 和 *"Source is disconnected. Reconnect it in Skills → Sources."*。若已不再选择，`install-update` 返回 `409` 和 *"This skill is no longer selected. Manage its source to resume syncing."*；刷新失败或上游已移除时也会报错。

`install-update` 会用相同来源 URL 重新导入，按规范 key 或 slug 找到匹配技能，在原行更新并返回刷新后的记录。技能 `id` 不会改变，因此按 ID 附加的关系无需重新同步。

<a id="beta-releases-of-the-bundled-paperclip-skill"></a>
<a id="内置-paperclip-技能的-beta-版本"></a>
### 内置 `paperclip` 技能的 Beta 版本

内置 `paperclip` 技能（规范 key `paperclipai/paperclip/paperclip`）包含冻结快照**发行版**，代理可以固定到某一版本，而不是使用实时默认版。固定原因和选择器行为见指南[固定 Paperclip 技能的 Beta 版本](../guides/org/skills.md#pinning-a-beta-release-of-the-paperclip-skill)；这里说明数据和传输格式。

实验设置 `enableBetaSkills` 名称为 **Beta skills**，说明为“允许代理固定 Paperclip 核心技能的 Beta 版本”。其层级是 `preference`（租户控制，Paperclip Cloud 框架不会管理），不同于可能锁定的 `managed` 设置。`cloudDefault` 与 `selfHostedDefault` 均为 `false`，因此需操作员手动启用。

服务器读取 `skills-releases/paperclip/releases.json`，为每家公司内置 `paperclip` 技能清单中的每个项目植入一个版本。每项含 `id`、`releaseName`、`releasedAt`、`notes` 和 `dir`（相对于注册表根目录解析的快照文件夹）。植入不会更改技能当前版本；实时默认版保持不变，只有固定发行版的代理会使用该快照。目前有两个发行版：

| `id` | `releaseName` | `releasedAt` |
|---|---|---|
| `v0` | `V0 — Original` | `2026-07-15T07:52:54-05:00` |
| `v7-roster` | `V7 — Roster champion` | `2026-07-21` |

两者都是冻结的历史快照；`v7-roster` 早于 `SKILL.md`、`references/api-reference.md` 和 `references/company-skills.md` 的后续三次修改，实时默认版包含较新的指导。

重新植入时会逐字节校验。如果公司已有某发行版 ID，但所存清单哈希与注册表快照不一致，会失败并报 `Bundled skill release <id> does not match its seeded snapshot.`，而不会改写代理可能正在使用的版本。

`company_skill_versions` 行包含 `release_id`、`release_name`、`released_at` 三列，在 `CompanySkillVersion` 中映射为 `releaseId`、`releaseName` 和 `releasedAt`；普通版本三者均为空。局部唯一索引 `company_skill_versions_skill_release_idx`（`company_skill_id`、`release_id`，条件为 `release_id is not null`）保证一个技能每个发行版 ID 至多一行。`GET /api/companies/{companyId}/skills/{skillId}/versions` 返回技能版本，其中 `releaseId` 非空的行就是发行版。

通过 `POST /api/agents/{agentId}/skills/sync` 固定版本时，每项可用字符串或对象形式：

```json
{
  "desiredSkills": [
    "code-review",
    { "key": "paperclipai/paperclip/paperclip", "versionId": "<company skill version uuid>" }
  ]
}
```

字符串形式和 `"versionId": null` 都表示不固定，使用实时默认版。返回的 `AgentSkillSnapshot` 会在扁平 `desiredSkills` 之外，于 `desiredSkillEntries`（`{ key, versionId }` 列表）报告固定版本。若 `enableBetaSkills` 关闭时仍传入非空 `versionId`，请求返回 `400` 和 `Beta skill version pins require the Beta skills experimental setting to be enabled.`

关闭该设置时，现有固定值会保留但不应用。心跳和代理配置解析都会以 `versionPinsEnabled: false` 构建版本选择映射，将每个 key 映射到 `null`，因此所有代理都使用实时技能。重新启用后会恢复已保存的固定设置。

<a id="9-troubleshooting-why-a-skill-isnt-loading"></a>
## 9. 故障排查：技能为何没有加载

按顺序检查，通常第一个匹配项就是问题所在。

### “我导入了技能，但列表里没有”

- 刷新列表。`GET /companies/{companyId}/skills` 会触发 `ensureSkillInventoryCurrent`，清理已消失的本地技能并重新导入内置技能。界面会自动刷新，API 调用方可能需显式重新查询。
- 检查技能行的 `compatibility`。不是 `compatible` 的技能不会显示在代理选择器中。
- 项目扫描可能发生 slug/key 冲突；检查扫描响应的 `conflicts[]`，不要只看 `imported[]`。

### “我已分配技能，但代理看不到”

- 查看代理快照：`GET /api/agents/{agentId}/skills`。
- 若 `mode: "unsupported"`，适配器无法将技能推送到运行时。分配已记录但没有写入文件；请在远程运行时（如 OpenClaw）直接管理技能。
- 若 `mode: "ephemeral"`，文件只在运行期间存在；代理未运行时工作区为空是正常现象。
- 查看快照中的 `warnings`，同步错误、文件缺失和适配器拒绝项都会显示在那里。

### “Invalid company skill selection (ambiguous/unknown references)”

- 传入的 slug 匹配多项技能，或 key/UUID/slug 无匹配。请改用规范 `key` 进行无歧义查找；需要获取 key 时先列出技能。

### “技能行存在，但文件查看器为空”

- 本地路径技能：来源目录可能已移动；下次列表请求时该行会被清理。请从新路径重新导入。
- GitHub 技能：当前 `sourceRef` 之前的提交可能因强制推送而不存在。重新运行 `update-status` 和 `install-update`。

### “GitHub 技能一直停留在旧提交”

- `update-status` 显示 `hasUpdate: false`？说明已是跟踪引用的最新提交。若想跟踪其他引用（例如从 `main` 改为标签），使用明确 `tree/<ref>` 路径重新导入 URL。对 `github.com` URL，这会为该分支创建独立[技能来源](#github-技能来源)，其中技能各自安装；请将代理改派给新技能，并在不再需要旧来源时断开它。
- `install-update` 返回 422？来源可能已无法解析（例如上游删除了 `SKILL.md`）。可使用 `tree/<sha>/...` URL 手动固定，或删除技能。

### “Local skill source is outside approved company workspace roots”

- 导入路径不在公司的托管技能目录、注册项目工作区或服务器管理的项目检出目录内（错误码 `skill_workspace_boundary_denied`）。将目录注册为项目工作区，或将技能移入其中，再重新导入。检查前会跟随符号链接，因此链接到范围外的位置也会被拒绝。

### “Skill version changed. Read the current version before retrying.”

中文释义：技能版本已更改。请读取当前版本后重试。（以上英文为接口返回的固定错误消息。）

- 更新文件时提交了 `expectedVersionId`，但你读取后其他人已保存技能。重新获取技能，在当前内容上应用修改，并使用新的 `currentVersionId` 重试；若使用幂等键，也应换新键。

### “GitHub 同步技能没有获取上游更新”

- 来源不会自动刷新；请在 **Skills → Sources** 点 **Refresh**，或调用 `POST /skill-sources/{sourceId}/refresh`。
- 上游新技能只会被列出，不会安装；选中并保存选择。
- 检查来源的 `lastError`。刷新失败会保留最后一个正常版本。

### “Frontmatter 修改没有生效”

- Paperclip 的 YAML 读取器刻意采用严格但精简的语法。若某个值意外为空，请检查：
  - 块是否以独占一行的 `---` 开始和结束。
  - 缩进是否使用空格，而不是制表符。
  - 列表项是否使用后接空格的 `- `。
  - 块标量（`>`、`|`）是否比对应字段多缩进两个空格。
- 请使用常规解析器验证 YAML。如果常规解析器可以解析，但 Paperclip 无法解析，请提交缺陷报告；本文前文介绍了内置解析器的规则。

### “代理得到了一项它没有请求的技能”

- 这可能是必需内置技能。服务器会将带有 `metadata.sourceKind: paperclip_bundled` 的 `paperclipai/paperclip/*` 技能并入每个已解析的 `desiredSkills` 集合，因此从请求中移除也没有作用。

### “我固定了 Beta 版本，代理却仍运行默认版”

- 在 **Settings → Experimental** 检查 **Beta skills**。关闭 `enableBetaSkills` 时，固定值会被忽略但不会删除，代理仍使用实时技能。
- 固定只在技能仍属于代理期望集合时有效；移除技能时固定值也随之移除。
- 确认固定值已保存：设置**启用**时，`GET /api/agents/{agentId}/skills` 会在 `desiredSkillEntries` 中报告；设置关闭时该响应会显示 `versionId: null`，虽然代理上仍保存固定值。因此关闭状态下不要据此判断固定值丢失。

### “我删除了技能，它又回来了”

- 这是内置技能。服务器会在每次公司技能列表请求时从磁盘 `skills/` 重新导入。要隐藏内置技能，需从 Paperclip 发行版中移除，或运行自行分叉构建的版本。

## 另请参阅

- [技能指南](../guides/org/skills.md)：界面操作、文件清单、信任级别和编写技能清单。
- [代理 API → 技能](./api/agents.md#skills)：`GET /agents/{id}/skills` 与 `POST /agents/{id}/skills/sync` 的请求和响应结构。
- [适配器参考](./adapters/overview.md)：各适配器对 `persistent`、`ephemeral` 和 `unsupported` 同步模式的支持情况。
- [创建适配器](./adapters/creating-an-adapter.md)：为新运行时实现 `listSkills` / `syncSkills`。

---

