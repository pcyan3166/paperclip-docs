---
paperclip_version: v2026.618.0
seo_title: 任务规划技能
seo_description: 将请求转化为执行者可以落实的计划：明确子任务依赖图、实际阻塞项、指定负责人，并预先定义验收标准。
---

# 任务规划

> 将 Paperclip 任务或请求转化为结构化实施计划，明确子任务依赖图、阻塞项、负责人和验收标准，然后将计划保存为该任务的 `plan` 文档。

把 Paperclip 任务或请求转化为执行者确实可以落实的实施计划：建立子任务依赖图、明确实际阻塞项和具体负责人，并定义验收标准；将计划保存为任务的 `plan` 文档。避免制定读起来不错、却无法拆分成实际工作的计划。

此技能是**内置**目录技能，属于内置基础套件。如何安装、审计、更新、分配和重置目录技能，请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

## 适用场景

- 任务要求你规划、界定范围、拆分工作、设计发布过程、提出工作方案或类似工作。
- 用户希望先看到书面计划，再批准实施。
- 管理者需要委派一项复杂工作，但尚不确定如何拆分。
- 你接手的任务太大，无法在一次心跳运行中完成，需要拆分。

### 不适用场景

- 任务只是一个可以在当前心跳运行中完成的简单变更——直接完成即可。
- 任务属于取证调查（例如“为什么会坏掉”）——先诊断根本原因。
- 当前已有 `plan` 文档，且变更较小——更新现有文档，不要从头开始。

## 目录元数据

| 字段 | 值 |
|---|---|
| 目录 ID | `paperclipai:bundled:paperclip-operations:task-planning` |
| 规范键 | `paperclipai/bundled/paperclip-operations/task-planning` |
| 目录路径 | `catalog/bundled/paperclip-operations/task-planning` |
| 类型 | `bundled` |
| 类别 | `paperclip-operations` |
| Slug | `task-planning` |
| 入口文件 | `SKILL.md` |
| 信任级别 | `markdown_only` |
| 兼容性 | `compatible` |
| 默认安装 | `false` |
| 建议角色 | `manager`、`engineer`、`product` |
| 依赖项 | — |
| 标签 | `paperclip`、`planning`、`issues`、`delegation` |
| 文件数 | 1 |
| 内容哈希 | `sha256:487c3116eead89fef4f71c984329c41836bd69951b7ef9ed8736c69435e02438` |
| 软件包 | `@paperclipai/skills-catalog@0.3.1` |

## 文件清单

| 路径 | 类型 | 字节数 |
|---|---|---:|
| `SKILL.md` | `skill` | 4,938 |

## 完整技能定义

下方代码块是此技能完整且权威的 `SKILL.md`，也是智能体运行时实际加载的文件。可以使用代码块右上角的控件进行复制，或将其下载为 `SKILL.md`。

````markdown skill-source
---
name: task-planning
description: Turn a Paperclip issue or request into a structured implementation plan with child task graph, blockers, owners, and acceptance criteria, then save it as the issue `plan` document.
key: paperclipai/bundled/paperclip-operations/task-planning
recommendedForRoles:
  - manager
  - engineer
  - product
tags:
  - paperclip
  - planning
  - issues
  - delegation
---

# Task Planning

Produce implementation plans that the Paperclip executor can actually run: explicit child issues, real blockers, named owners, and a defined acceptance bar. Avoid plans that read well but cannot be split into work.

## When to use

- An issue asks you to "plan", "scope", "break down", "design the rollout", "propose the work", or similar.
- A user wants a written plan before approving implementation.
- A manager needs to delegate non-trivial work and the shape of the work is not obvious yet.
- You inherited an issue too large to deliver in one heartbeat and need to split it.

## When not to use

- The issue is a single small change you can ship in the same heartbeat. Just ship it.
- The issue is forensic ("why did this break"). Use a diagnosis skill first; plan only after the root cause is named.
- A current `plan` document already exists and the change is minor. Update that document; do not start fresh.

## Outputs

1. An updated issue document with key `plan` (markdown).
2. A short comment on the issue that links to the plan document and names the next action.
3. Where the plan requires approval, an issue-thread interaction of kind `request_confirmation` bound to the latest plan revision.

Do not create implementation subtasks until the plan is accepted.

## Plan structure

Required sections, in order:

1. **Goal** — one paragraph. What changes for the user, the operator, or the system once this work lands.
2. **Context reviewed** — bullet list of documents, files, and prior issues you read. Lets reviewers spot missing inputs.
3. **Constraints and non-goals** — what must hold (compatibility, security, performance) and what this plan deliberately will not do.
4. **Approach** — the chosen path, with a short rationale. If you considered alternatives, name them and why you rejected them.
5. **Work breakdown** — ordered list of child issues. Each child has:
   - Title in imperative form.
   - Owner specialty (Engineer, QA, Designer, Security, DevRel, Manager, etc.).
   - Scope and deliverables.
   - Acceptance criteria.
   - Blocks/blocked-by relationships expressed by phase letter or child title.
6. **Acceptance** — the bar for the parent issue. How the user knows the whole thing is done.
7. **Risks and mitigations** — short list. Skip if there are none.
8. **Deferrals** — what is intentionally pushed to follow-up issues, with why.

## Rules of thumb for splitting

- One child issue, one specialty. If two specialties have to coordinate inside the same issue, split it.
- One child issue, one acceptance verdict. If a reviewer would say "this is half done", split it.
- A child must be checkout-able by the owner from its title and description alone. Reviewers should not have to re-read the parent plan to understand a child.
- Order children by real blocker chains, not by author preference. Parallel children should explicitly say `blockers: none`.
- Avoid `polish` or `cleanup` child issues without acceptance criteria — they never close.

## Filing the plan

Use the Paperclip API to write the plan document, then comment:

- `PUT /api/issues/{issueId}/documents/plan` with the markdown body. If `plan` already exists, include the latest `baseRevisionId`.
- `POST /api/issues/{issueId}/comments` with a short summary that links the plan: `/<prefix>/issues/<issue-id>#document-plan`.
- If approval is required: `POST /api/issues/{issueId}/interactions` with `kind: request_confirmation`, `targetRevisionId` set to the new plan revision, `continuationPolicy: wake_assignee`, and `idempotencyKey: "confirmation:{issueId}:plan:{revisionId}"`.
- Set the issue to `in_review` after creating the confirmation. Stay assigned so the acceptance wakes the planner.

When the plan is accepted, see the companion skill for converting accepted plans into Paperclip executable tasks. Key requirements covered there: produce a compact task matrix (task, owner, initial status, blockers); encode every hard dependency as `blockedByIssueIds` — parent/child nesting alone does not block execution; and verify the created issue graph before closing the source planning issue.

## Anti-patterns

- Plan disguised as a description edit. Use the `plan` document.
- "Phases A–Z" with no work breakdown inside the phases.
- Children with descriptions that say "see parent" — they fail at delegation time.
- Acceptance written as "code review approval". Reviewers need a behavior bar, not a process bar.
- Plans that bury blocker chains in prose. Use explicit blocked-by lines.
````

## 中文译文

# 任务规划

制定 Paperclip 执行器确实可以执行的实施计划：明确子 issue、真实阻塞关系、具体负责人和验收标准。避免计划读起来很好，却无法拆分为实际工作。

## 适用场景

- issue 要求你“规划”“界定范围”“拆解任务”“设计发布方案”“提出工作方案”等。
- 用户希望先看到书面计划，再批准实施。
- 经理需要委派非简单工作，但工作拆分方式尚不明确。
- 你接手的 issue 太大，无法在一次心跳中交付，需要拆分。

## 不适用场景

- issue 只涉及一项可以在同一次心跳中交付的小改动。直接完成即可。
- issue 要求调查“为什么出了问题”。应先使用诊断技能；只有明确根因后再制定计划。
- 当前已有 `plan` 文档，且变更较小。更新现有文档，不要从头开始。

## 输出

1. 更新 issue 文档，键为 `plan`（Markdown 格式）。
2. 在 issue 上发表评论，简要链接到计划文档并指出下一步行动。
3. 若计划需要审批，创建绑定到最新计划修订版本、类型为 `request_confirmation` 的 issue 线程交互。

计划获批前，不要创建实施子任务。

## 计划结构

按以下顺序包含必需章节：

1. **目标（Goal）**——用一段话说明工作完成后，用户、操作者或系统会发生什么变化。
2. **已审阅的上下文（Context reviewed）**——用列表列出你阅读过的文档、文件和既有 issue，帮助审核者发现输入是否缺失。
3. **约束和非目标（Constraints and non-goals）**——说明必须满足的条件（兼容性、安全、性能等），以及本计划明确不处理的事项。
4. **方案（Approach）**——说明选定的路径及简短理由。若考虑过其他方案，指出拒绝它们的原因。
5. **工作拆分（Work breakdown）**——按顺序列出子 issue。每个子项都要包含：
   - 祈使句形式的标题。
   - 负责的专业角色（Engineer、QA、Designer、Security、DevRel、Manager 等）。
   - 范围与交付物。
   - 验收标准。
   - 用阶段字母或子项标题表示的阻塞/被阻塞关系。
6. **验收（Acceptance）**——说明父 issue 的完成标准，以及用户如何知道整个工作已经完成。
7. **风险与缓解措施（Risks and mitigations）**——简短列出风险。没有风险时可以省略。
8. **后续处理（Deferrals）**——说明哪些事项有意留到后续 issue 处理，以及原因。

## 拆分工作的经验规则

- 一个子 issue 对应一个专业领域。如果两个领域必须在同一个 issue 中协调，应拆分。
- 一个子 issue 对应一个验收结论。如果审核者可能会说“只完成了一半”，就应拆分。
- 负责人只看子项标题和描述就应能检出并开始处理，不应要求重新阅读父计划。
- 按真实阻塞链排列子项，而不是按作者偏好排序。并行子项应明确写出 `blockers: none`。
- 不要创建没有验收标准的 `polish` 或 `cleanup` 子项；这类任务永远无法关闭。

## 提交计划

使用 Paperclip API 写入计划文档，然后发表评论：

- 使用 `PUT /api/issues/{issueId}/documents/plan` 写入 Markdown 正文。若 `plan` 已存在，必须包含最新的 `baseRevisionId`。
- 使用 `POST /api/issues/{issueId}/comments` 发布简短摘要并链接计划：`/<prefix>/issues/<issue-id>#document-plan`。
- 若需要审批，使用 `POST /api/issues/{issueId}/interactions` 创建交互，设置 `kind: request_confirmation`、`targetRevisionId` 为新计划修订版本、`continuationPolicy: wake_assignee`，以及 `idempotencyKey: "confirmation:{issueId}:plan:{revisionId}"`。
- 创建确认请求后，将 issue 设为 `in_review`。保持自己为受派人，以便计划获批时唤醒规划者。

计划获批后，使用配套技能将获批计划转换为 Paperclip 可执行任务。关键要求：生成紧凑任务矩阵（任务、负责人、初始状态、阻塞项）；将每项硬依赖编码为 `blockedByIssueIds`——父子层级本身不会阻止任务执行；关闭源规划 issue 前验证已创建的 issue 关系图。

## 反面模式

- 把计划伪装成描述编辑。应使用 `plan` 文档。
- 只列出“阶段 A–Z”，却没有各阶段的工作拆分。
- 子项描述只写“见父项”——委派时无法据此开展工作。
- 把验收写成“代码审核通过”。审核者需要的是行为标准，而不是流程标准。
- 在散文中隐藏阻塞链。应明确写出被哪些项阻塞。

## 另请参阅

- [内置技能](../../bundled.md)——按类别整理的全部内置目录技能。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、分配和故障排查。
- [应用内置目录](../../../skills.md#3-app-shipped-catalog)——内置和可选目录技能的版本管理与更新方式。
- [技能指南](../../../../guides/org/skills.md)——界面操作指南和内置目录概览。
