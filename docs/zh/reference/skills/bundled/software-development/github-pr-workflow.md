---
paperclip_version: v2026.618.0
seo_title: GitHub 拉取请求工作流技能
seo_description: 准备审核者无需追问即可合并的拉取请求：规范分支、可审核的提交、高信息量的说明和验证证据。
---

# GitHub 拉取请求工作流

> 从功能分支准备 GitHub 拉取请求，处理分支规范、提交组织、标题/正文、验证说明、UI 工作的截图，以及对审核意见的回复。

准备一份审核者无需追问即可合并的 GitHub 拉取请求：规范分支、组织易审核的提交、撰写信息量高的标题和正文、提供验证证据、为 UI 工作添加截图，并清楚地回复审核意见。

此技能是**内置**目录技能，属于内置基础套件。如何安装、审计、更新、分配和重置目录技能，请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

## 适用场景

- 功能上已完成变更，准备创建拉取请求。
- 审核者留下了评论，你需要回复并推送修复。
- 拉取请求已打开一天以上，需要重新整理（例如冲突未解决、缺少描述或验证说明）。

### 不适用场景

- 变更尚未在功能上完成——先完成工作。
- 仓库使用的不是 GitHub，而是其他代码托管平台——请遵循对应平台的约定。

## 目录元数据

| 字段 | 值 |
|---|---|
| 目录 ID | `paperclipai:bundled:software-development:github-pr-workflow` |
| 规范键 | `paperclipai/bundled/software-development/github-pr-workflow` |
| 目录路径 | `catalog/bundled/software-development/github-pr-workflow` |
| 类型 | `bundled` |
| 类别 | `software-development` |
| Slug | `github-pr-workflow` |
| 入口文件 | `SKILL.md` |
| 信任级别 | `markdown_only` |
| 兼容性 | `compatible` |
| 默认安装 | `false` |
| 建议角色 | `engineer` |
| 依赖项 | — |
| 标签 | `github`、`pull-requests`、`code-review`、`release` |
| 文件数 | 1 |
| 内容哈希 | `sha256:90f278c89aa0711be150c1cd2456ca25620d02f36995b113ca9837d756a37f6c` |
| 软件包 | `@paperclipai/skills-catalog@0.3.1` |

## 文件清单

| 路径 | 类型 | 字节数 |
|---|---|---:|
| `SKILL.md` | `skill` | 3,970 |

## 完整技能定义

下方代码块是此技能完整且权威的 `SKILL.md`，也是智能体运行时实际加载的文件。可以使用代码块右上角的控件进行复制，或将其下载为 `SKILL.md`。

````markdown skill-source
---
name: github-pr-workflow
description: Prepare a GitHub pull request from a feature branch — branch hygiene, commit shape, title/body, verification notes, screenshots for UI work, and replies to review comments.
key: paperclipai/bundled/software-development/github-pr-workflow
recommendedForRoles:
  - engineer
tags:
  - github
  - pull-requests
  - code-review
  - release
---

# GitHub Pull Request Workflow

Ship a PR a reviewer can land without follow-up clarifying questions. The aim is high signal in the title and body, evidence the change works, and clean replies when feedback comes in.

## When to use

- You are about to open a PR for a change that is functionally complete.
- A reviewer left comments and you need to respond and push fixes.
- A PR has been open more than a day and needs to be brought back into shape (stale conflicts, missing description, missing verification).

## When not to use

- The change is not yet functionally complete. Finish the work first; draft PRs that bounce on review are noise.
- The repository uses a non-GitHub forge. Adjust to that forge's conventions; do not force GitHub-isms.

## Branch hygiene before opening

- Rebase or merge from the target base so the diff is current.
- Squash WIP commits into reviewable units. Prefer one commit per logical change; do not force one-commit-per-PR if the work is genuinely multi-step.
- Confirm tests, typecheck, and lint pass locally. Note any deliberate skips in the PR body.
- Remove debug prints, commented-out code, and `TODO` markers that are not tracked.

## PR title

- Imperative mood, under 70 characters.
- Lead with the user-visible change, not the file touched. `Allow CSV export from reports table` beats `Update reports.tsx`.
- If the repo uses an issue prefix convention (`PAP-1234:`, `[security]`), follow it.
- No trailing period.

## PR body

Use this structure:

```md
## Summary
- 1–3 bullets describing what changed and why.

## Implementation notes
- Anything non-obvious in the diff: trade-offs, dropped alternatives, gotchas.
- Migration or config implications.

## Verification
- The exact commands or steps you ran.
- Screenshots or short clips for UI changes (required if pixels moved).
- Edge cases you exercised by hand.

## Risk and rollback
- What breaks if this is reverted, and how to revert cleanly.
```

Skip the `Risk and rollback` section only for clearly trivial PRs (typos, docs).

## Verification evidence

- Tests passing in CI is necessary, not sufficient. Reviewers also need to know the change behaves correctly end to end.
- For UI work, include screenshots of the golden path and one edge case. Tag dark and light mode if the project supports both.
- For migrations, include a dry-run plan and reversal steps.
- For performance changes, include a before/after measurement, not adjectives.

## Replying to review comments

- Reply on every comment, even with just "fixed in <commit-sha>" — silent fixes leave the reviewer guessing.
- Push fixes as new commits while review is active; do not amend during review unless the reviewer agrees.
- If you disagree with feedback, say so with one sentence of rationale and let the reviewer decide. Don't escalate over comments.
- Re-request review explicitly after pushing changes.

## Merge checklist

- All required checks green.
- All review comments resolved.
- PR title/body still accurate (update if scope changed mid-review).
- Linked issue moves to `in_review` or `done` per project convention.
- Delete the branch after merge unless it is a long-lived integration branch.

## Anti-patterns

- PR description that says "see commits". Reviewers should not need to read the log.
- Mixing refactor and behavior change in the same PR with no separation in the body.
- "Address feedback" commits that bundle unrelated edits. One commit per round of feedback is fine; one commit for everything in flight is not.
- Force-pushing during active review without telling the reviewer.
````

## 中文译文

# GitHub 拉取请求工作流

> 创建可供审核者直接合并的拉取请求，无需后续追问。重点是让标题和正文信息明确、提供变更有效的证据，并在收到反馈时清楚地回复。

## 适用场景

- 准备为功能已完成的变更创建拉取请求。
- 审核者留下了意见，需要回复并推送修复。
- 拉取请求已打开一天以上，需要重新整理（例如冲突未解决、缺少说明或验证信息）。

## 不适用场景

- 功能尚未完成。请先完成工作；创建后会在审核中被退回的草稿拉取请求只会增加噪声。
- 仓库使用的不是 GitHub，而是其他代码托管平台。请遵循该平台的约定，不要强行套用 GitHub 的做法。

## 创建前的分支规范

- 从目标基线分支变基或合并，使差异基于最新代码。
- 将 WIP（进行中）提交压缩成便于审核的单元。每个逻辑变更优先使用一个提交；若工作确实分为多个步骤，不要强求整个拉取请求只能有一个提交。
- 确认本地测试、类型检查和 lint 均通过。在拉取请求正文中说明有意跳过的检查。
- 删除调试输出、注释掉的代码，以及未纳入跟踪的 `TODO` 标记。

## 拉取请求标题

- 使用祈使语气，少于 70 个字符。
- 先写用户可见的变更，而不是被修改的文件。`Allow CSV export from reports table`（允许从报表表格导出 CSV）优于 `Update reports.tsx`（更新 reports.tsx）。
- 若仓库使用 issue 前缀约定（如 `PAP-1234:`、`[security]`），请遵循该约定。
- 末尾不要加句号。

## 拉取请求正文

使用以下结构：

```md
## 摘要
- 用 1–3 条要点说明变更内容及原因。

## 实现说明
- 说明差异中不明显的内容：权衡、放弃的替代方案、注意事项。
- 说明迁移或配置方面的影响。

## 验证
- 写明实际运行的命令或执行的步骤。
- UI 有变化时附上截图或短视频（必需）。
- 列出手动验证过的边界情况。

## 风险与回滚
- 说明回滚会造成什么影响，以及如何干净地回滚。
```

只有在变更显然非常简单（例如文档或拼写修正）时，才可省略“风险与回滚”部分。

## 验证证据

- CI 测试通过是必要条件，但还不够。审核者还需要了解端到端行为是否正确。
- UI 变更应包含主流程和一个边界情况的截图。项目支持深色和浅色模式时，两种模式都应标注或展示。
- 迁移应包含试运行计划和回退步骤。
- 性能变更应提供前后测量数据，不要只用形容词描述。

## 回复审核意见

- 回复每条意见，即使只回复“已在 `<commit-sha>` 中修复”也可以；默默修复会让审核者无法确认情况。
- 审核期间将修复作为新提交推送；除非审核者同意，否则不要 amend（改写）提交。
- 若不同意反馈，用一句话说明理由，然后由审核者决定。不要因评论升级冲突。
- 推送变更后，明确重新请求审核。

## 合并检查清单

- 所有必需的检查均通过。
- 所有审核意见均已解决。
- 拉取请求标题和正文仍准确反映变更（若审核中途范围发生变化，请更新）。
- 按项目约定，将关联 issue 移至 `in_review` 或 `done`。
- 合并后删除分支，除非它是长期使用的集成分支。

## 反面模式

- 拉取请求说明只写“见提交记录”。审核者不应被迫阅读提交日志。
- 在同一个拉取请求中混合重构和行为变更，却不在正文中区分说明。
- 用“处理反馈”之类的提交打包无关改动。每轮反馈一个提交是合理的；把所有进行中的工作塞进一个提交则不合理。
- 审核进行期间强制推送，却不告知审核者。

## 另请参阅

- [内置技能](../../bundled.md)——按类别整理的全部内置目录技能。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、分配和故障排查。
- [应用内置目录](../../../skills.md#3-app-shipped-catalog)——内置和可选目录技能的版本管理与更新方式。
- [技能指南](../../../../guides/org/skills.md)——界面操作指南和内置目录概览。
