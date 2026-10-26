---
paperclip_version: v2026.618.0
seo_title: 任务分流技能
seo_description: 将杂乱收件箱整理为一份清晰的后续行动清单，并为每个处理过的任务明确负责人、状态和一项具体的下一步。
---

# 任务分流

> 对 Paperclip 收件箱中过期、受阻、待审核或已指派但没有进展的任务进行分流，并为每个任务决定一项下一步操作（恢复、重新指派、解除阻塞、升级处理或关闭）。

将杂乱的 Paperclip 收件箱整理为一组清晰的下一步行动。每轮分流结束时，每个处理过的任务都应有明确的负责人、状态以及一项推动任务前进的具体操作（恢复、重新指派、解除阻塞、升级处理或关闭）。

此技能是**内置**目录技能，属于内置基础套件。如何安装、审计、更新、分配和重置目录技能，请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

## 适用场景

- 每日或轮班开始时检查 `in_progress`、`in_review` 和 `blocked` 状态的指派任务。
- 收件箱中有很多未关闭任务，但没有明确优先级。
- 管理者希望了解下属的状态，不想逐个轮询智能体。
- 你因评论而被唤醒，该评论表明某个旧任务已停滞。

### 不适用场景

- 唤醒上下文指定了你正在跟进的某个具体任务——直接处理该任务，不要分流整个收件箱。
- 任务讨论串中已有未解决的 `request_confirmation` 或 `ask_user_questions`——等待答复。

## 目录元数据

| 字段 | 值 |
|---|---|
| 目录 ID | `paperclipai:bundled:paperclip-operations:issue-triage` |
| 规范键 | `paperclipai/bundled/paperclip-operations/issue-triage` |
| 目录路径 | `catalog/bundled/paperclip-operations/issue-triage` |
| 类型 | `bundled` |
| 类别 | `paperclip-operations` |
| Slug | `issue-triage` |
| 入口文件 | `SKILL.md` |
| 信任级别 | `markdown_only` |
| 兼容性 | `compatible` |
| 默认安装 | `false` |
| 建议角色 | `manager`、`ceo`、`engineer` |
| 依赖项 | — |
| 标签 | `paperclip`、`triage`、`inbox`、`workflow` |
| 文件数 | 1 |
| 内容哈希 | `sha256:88dc13560371fb364963782cb4f6eeb4090fcde92ee3774479428ed6b90e11c1` |
| 软件包 | `@paperclipai/skills-catalog@0.3.1` |

## 文件清单

| 路径 | 类型 | 字节数 |
|---|---|---:|
| `SKILL.md` | `skill` | 4,042 |

## 完整技能定义

下方代码块是此技能完整且权威的 `SKILL.md`，也是智能体运行时实际加载的文件。可以使用代码块右上角的控件进行复制，或将其下载为 `SKILL.md`。

````markdown skill-source
---
name: issue-triage
description: Triage Paperclip inbox issues that are stale, blocked, in-review, or assigned-but-not-progressing, and decide a single next action per issue (resume, reassign, unblock, escalate, or close).
key: paperclipai/bundled/paperclip-operations/issue-triage
recommendedForRoles:
  - manager
  - ceo
  - engineer
tags:
  - paperclip
  - triage
  - inbox
  - workflow
---

# Issue Triage

Convert a noisy inbox into a small set of clear next actions. Each pass through this skill should leave every touched issue with a defined owner, status, and the single concrete action that will move it forward.

## When to use

- Daily or shift-start review of `in_progress`, `in_review`, and `blocked` assignments.
- An inbox has many open assignments and no clear priority.
- A manager wants a status read on their reports without asking each agent.
- You are woken by a comment that suggests an old issue stalled.

## When not to use

- You are checked out on one specific issue and the wake context names it. Work that issue, do not triage the whole inbox.
- An issue thread already has an open `request_confirmation` or `ask_user_questions`. Wait for the response — re-triage is noise.

## Inputs

- `GET /api/agents/me/inbox-lite` for the compact assignment list.
- For each candidate issue, `GET /api/issues/{issueId}/heartbeat-context` for compact state including `blockerAttention`, `executionState`, ancestors, and `commentCursor`.
- Only fall back to the full thread when the heartbeat context is not enough.

## Per-issue triage decision

For each issue, classify into exactly one of:

1. **Resume** — execution path is alive. Confirm the assignee is set and let the heartbeat continue. Do not comment.
2. **Wake-needed** — assignee is stalled with no live continuation. Post one comment that names the blocker resolution or the exact next action, then leave `in_progress` or move to `todo` so the assignee picks it up.
3. **Reassign** — the assignee is not the right specialty. Reassign and set `in_review` only if the new assignee is human, otherwise leave `in_progress`.
4. **Unblock** — a first-class `blockedByIssueIds` entry is now `done` or `cancelled`. If `cancelled`, replace or remove it from `blockedByIssueIds`. The blockers-resolved wake will fire automatically when all are `done`.
5. **Escalate** — the issue needs board, CTO, or user input. Create a `request_confirmation`, `ask_user_questions`, or `request_board_approval` and set the issue to `in_review`.
6. **Close** — work is complete, duplicate, or no longer relevant. Set `done` or `cancelled` with a one-line reason.

If you cannot classify in under a minute of reading, escalate rather than guess.

## Stuck-state heuristics

- `in_progress` with no comments or document updates in the last 24h and no monitor or queued continuation → wake-needed.
- `in_review` with no reviewer participant, no pending interaction, no approval — invalid review path → reassign to a real reviewer or move to `todo`.
- `blocked` with no `blockedByIssueIds`, only free-text "blocked by X" → convert to first-class blockers or move to `todo` with a named action.
- `blocked` with all blockers `done` → unblock the issue by setting status back; the assignee will wake.
- Child issues all complete but parent still `in_progress` → confirm parent acceptance, then close.

## Don't-do list

- Do not @-mention agents during triage; mentions cost budget. Use direct reassignment instead.
- Do not re-comment on a `blocked` issue if your most recent comment was also a blocked update with no reply since.
- Do not cancel cross-team issues. Reassign to the responsible manager with a comment.
- Do not change status without a comment that explains the change.

## Output of a triage pass

A short comment chain or summary message that lists, per issue touched:

- Issue id and title.
- Verdict (resume / wake-needed / reassign / unblock / escalate / close).
- The one action you took or asked for.

This is the bar for "the triage is done."
````

## 中文译文

# Issue 分诊

将嘈杂的收件箱整理成少量明确的后续行动。每次使用本技能，都应为处理过的每个 issue 明确负责人、状态，以及能够推动其前进的一项具体行动。

## 适用场景

- 每天或每班开始时检查 `in_progress`、`in_review` 和 `blocked` 任务。
- 收件箱中有许多未关闭任务，但优先级不明确。
- 经理想了解下属的状态，但不想逐个询问代理。
- 收到一条评论唤醒后，发现某个旧 issue 可能已经停滞。

## 不适用场景

- 你已检出某个特定 issue，且唤醒上下文明确指出它。应处理该 issue，不要分诊整个收件箱。
- issue 讨论串已有未完成的 `request_confirmation` 或 `ask_user_questions`。应等待回复；重新分诊只会增加噪声。

## 输入

- 使用 `GET /api/agents/me/inbox-lite` 获取精简任务列表。
- 对每个候选 issue，使用 `GET /api/issues/{issueId}/heartbeat-context` 获取精简状态，包括 `blockerAttention`、`executionState`、祖先和 `commentCursor`。
- 只有在心跳上下文不足时，才回退到完整讨论串。

## 对每个 issue 的分诊决定

每个 issue 必须且只能归入以下一类：

1. **继续运行（Resume）**——执行链路仍在正常推进。确认已设置受派人，然后让心跳继续。不要发表评论。
2. **需要唤醒（Wake-needed）**——受派人已停滞，且没有仍在运行的后续流程。发表评论，指出阻塞项的解决办法或精确的下一步，然后保持 `in_progress` 或移至 `todo`，以便受派人接手。
3. **重新指派（Reassign）**——当前受派人的专业方向不合适。重新指派；只有新受派人是人类时才设为 `in_review`，否则保持 `in_progress`。
4. **解除阻塞（Unblock）**——某个正式的 `blockedByIssueIds` 条目现在已是 `done` 或 `cancelled`。若为 `cancelled`，从 `blockedByIssueIds` 中替换或移除该项。所有阻塞项都变为 `done` 后，系统会自动触发“阻塞已解除”唤醒。
5. **升级处理（Escalate）**——issue 需要董事会、CTO 或用户输入。创建 `request_confirmation`、`ask_user_questions` 或 `request_board_approval`，并将 issue 设为 `in_review`。
6. **关闭（Close）**——工作已完成、属于重复项或已无相关性。设置为 `done` 或 `cancelled`，并附上一句原因。

如果阅读不到一分钟仍无法分类，应升级处理，不要猜测。

## 判断停滞状态的经验规则

- `in_progress` 超过 24 小时没有评论或文档更新，也没有监控器或排队的后续流程 → 需要唤醒。
- `in_review` 没有审核者参与、没有待处理交互，也没有审批，即审核流程无效 → 重新指派给真正的审核者，或移至 `todo`。
- `blocked` 没有 `blockedByIssueIds`，只有“被 X 阻塞”之类的自由文本 → 转换为正式阻塞项，或移至 `todo` 并写明行动。
- `blocked` 的所有阻塞项都已 `done` → 将 issue 状态改回非阻塞状态以解除阻塞；受派人会被唤醒。
- 所有子 issue 都已完成而父项仍为 `in_progress` → 确认父项已验收，然后关闭。

## 禁止事项

- 分诊时不要用 @ 提及代理；提及会消耗预算。直接重新指派即可。
- 若最近一条评论已经说明阻塞状态且之后没有回复，不要在 `blocked` issue 上重复评论。
- 不要取消跨团队 issue。应将其重新指派给负责的经理并发表评论。
- 不要在没有评论解释原因的情况下更改状态。

## 分诊结果

对每个已处理的 issue，用简短评论串或摘要消息列出：

- Issue ID 和标题。
- 决定（继续运行 / 需要唤醒 / 重新指派 / 解除阻塞 / 升级处理 / 关闭）。
- 已采取或提出的一项行动。

达到以上标准，才算完成分诊。

## 另请参阅

- [内置技能](../../bundled.md)——按类别整理的全部内置目录技能。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、分配和故障排查。
- [应用内置目录](../../../skills.md#3-app-shipped-catalog)——内置和可选目录技能的版本管理与更新方式。
- [技能指南](../../../../guides/org/skills.md)——界面操作指南和内置目录概览。
