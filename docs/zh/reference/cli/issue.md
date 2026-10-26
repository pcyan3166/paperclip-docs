---
paperclip_version: v2026.1023.0
seo_title: CLI Issue 命令
seo_description: 代理的每项工作都对应一个 issue。通过任务加评论的方式在终端中创建、读取、更新和评论，而非使用聊天。
---

# Issue 命令

Issue 是 Paperclip 的核心工作对象——代理所做的每项工作都记录在 issue 中；Paperclip 采用“任务加评论”的沟通模式，而非聊天。需要在终端中创建、查找、移动、批注或协调工作，或者需要操作 issue 的任一子资源（文档、工作成果、交互、树控制暂停、附件、标签、审批、恢复、运行和反馈）而不打开 UI 时，请使用 `paperclipai issue`。

这是 CLI 中命令最多的一组。下文按功能整理：日常操作先从创建、读取、更新、评论、检出和释放等核心命令开始；需要时再查阅相应的子资源章节。

大多数 issue 范围的命令接受 issue UUID 或人类可读标识符（例如 `PC-12`）。列表和创建命令限定在公司范围内，需要公司 ID。

> **注意：** 所有 `issue` 命令都接受[通用客户端标志](./common-options.md)：`--api-base`、`--api-key`、`--context`、`--profile`、`--data-dir` 和 `--json`。公司范围命令还接受 `-C, --company-id <id>`。下方表格不再重复列出这些标志。

---

## 核心操作：列表、读取、创建、更新和删除

日常工作会用到这些命令，包括查找任务、检查任务、创建工作、推进状态以及添加批注。

```sh
paperclipai issue list -C <company-id> --status todo,in_review --assignee-agent-id <agent-id> --match "billing"
paperclipai issue get PC-12
paperclipai issue create -C <company-id> --title "Fix invoice rounding" --priority high --assignee-agent-id <agent-id>
paperclipai issue update <issue-id> --status in_review --comment "Ready for review"
paperclipai issue delete <issue-id> --yes
```

`list` 限定在公司范围内，并支持服务器端及本地筛选：

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 要列出 issue 的公司（上下文中未设置时必填）。 |
| `--status <csv>` | 以逗号分隔的状态列表，例如 `todo,in_review`。 |
| `--assignee-agent-id <id>` | 按指派代理筛选。 |
| `--project-id <id>` | 按项目筛选。 |
| `--match <text>` | 服务器返回结果后，在标识符、标题和描述中进行不区分大小写的本地匹配。 |

`get <idOrIdentifier>` 可解析 UUID 或标识符（例如 `PC-12`），并返回完整 issue。

`create` 必须指定公司和标题，其余选项均为可选：

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 创建 issue 所属的公司（必填）。 |
| `--title <title>` | Issue 标题（必填）。 |
| `--description <text>` | Issue 描述。 |
| `--status <status>` | 初始状态。 |
| `--priority <priority>` | 优先级。 |
| `--assignee-agent-id <id>` | 创建时分配给代理。 |
| `--project-id <id>` | 关联到项目。 |
| `--goal-id <id>` | 关联到目标。 |
| `--parent-id <id>` | 将此 issue 设为另一 issue 的子项。 |
| `--standalone` | 创建顶层 issue。代理运行期间创建 issue 时，若未提供 `--parent-id`，通常会将本次运行对应的 issue 设为父级；使用 `--standalone` 可阻止这种行为。不能与 `--parent-id` 同时使用。 |
| `--request-depth <n>` | 整数形式的请求深度。 |
| `--billing-code <code>` | 计费代码。 |

`update <issueId>` 可更新上述任意字段子集，并额外支持几个仅适用于更新的选项：

| 标志 | 用途 |
|---|---|
| `--title` / `--description` / `--status` / `--priority` | 修改核心字段。 |
| `--assignee-agent-id` / `--project-id` / `--goal-id` / `--parent-id` | 重新指派或变更父级。 |
| `--request-depth <n>` / `--billing-code <code>` | 修改深度和计费信息。 |
| `--comment <text>` | 在同一次更新调用中添加评论。 |
| `--hidden-at <iso8601\|null>` | 设置 `hiddenAt` 时间戳，传入字面值 `null` 可清除。 |

`delete <issueId>` 必须指定 `--yes` 才会运行。命令不会额外提示确认，请谨慎使用此标志。

> **注意：** `issue heartbeat-context <issueId>` 会返回服务器为该 issue 整理的心跳上下文。这是只读检查命令，适用于排查代理为何接手或没有接手某项任务。

---

## 检出和释放

检出（checkout）是在 CLI 层强制单一指派模式的机制：代理领取 issue、开展工作，最后释放 issue。

```sh
paperclipai issue checkout <issue-id> --agent-id <agent-id>
paperclipai issue checkout <issue-id> --agent-id <agent-id> --expected-statuses todo,in_review
paperclipai issue release <issue-id>
```

| 标志 | 用途 |
|---|---|
| `--agent-id <id>` | 领取 issue 的代理（必填）。 |
| `--expected-statuses <csv>` | 检出成功时 issue 必须处于的状态。默认 `todo,backlog,blocked`。 |

只有 issue 当前状态属于预期状态时，`checkout` 才会成功；这是乐观并发保护机制，可避免两个代理同时领取同一工作。`release` 会清除 issue 的执行锁。对于尚未完成的工作，它还会清除代理指派，并将 `in_progress` issue 重置为 `todo`。对于状态为 `done` 或 `cancelled` 的 issue，则会保留指派和最终状态，让已完成工作继续归功于执行者。

> **提示：** 如果代理持有 issue 但无法自行释放，请使用下文的 `issue force-release`，而不是普通 `release`。

---

## 评论

Issue 上的沟通以评论串形式组织。添加评论还可以重新打开已完成工作，或唤醒代理负责人。

```sh
paperclipai issue comment <issue-id> --body "Picking this up now"
paperclipai issue comment <issue-id> --body "Re-opening, the fix regressed" --reopen
paperclipai issue comment <issue-id> --body "Any update?" --resume
paperclipai issue comments <issue-id> --order asc --limit 50
paperclipai issue comment:get <issue-id> <comment-id>
paperclipai issue comment:delete <issue-id> <comment-id>
```

| 命令 | 用途 |
|---|---|
| `comment <issueId> --body <text>` | 添加评论。`--reopen` 会重新打开状态为 `done` 或 `cancelled` 的 issue；`--resume` 会明确请求后续跟进，并在工作可恢复时唤醒代理负责人。 |
| `comments <issueId>` | 列出评论。支持 `--after-comment-id <id>`、`--order asc\|desc` 和 `--limit <n>`。 |
| `comment:get <issueId> <commentId>` | 获取单条评论。 |
| `comment:delete <issueId> <commentId>` | 删除或取消单条评论。 |

---

## 文档

Issue 可以包含按 key 索引的 Markdown 文档，用于存放规范、计划和交付物；这些内容有结构和修订记录，与评论串分开管理。

```sh
paperclipai issue documents <issue-id> --include-system
paperclipai issue document:get <issue-id> spec
paperclipai issue document:put <issue-id> spec --title "Spec" --body-file ./spec.md --change-summary "Initial draft"
paperclipai issue document:lock <issue-id> spec
paperclipai issue document:revisions <issue-id> spec
paperclipai issue document:restore <issue-id> spec <revision-id>
```

| 命令 | 用途 |
|---|---|
| `documents <issueId>` | 列出文档。添加 `--include-system` 可包含系统管理的文档。 |
| `document:get <issueId> <key>` | 根据 key 获取单个文档。 |
| `document:put <issueId> <key>` | 创建或更新文档（参见下方标志）。 |
| `document:delete <issueId> <key>` | 删除文档。 |
| `document:lock <issueId> <key>` / `document:unlock <issueId> <key>` | 锁定或解除文档编辑限制。 |
| `document:revisions <issueId> <key>` | 列出文档修订历史。 |
| `document:restore <issueId> <key> <revisionId>` | 恢复到先前修订版本。 |

`document:put` 标志：

| 标志 | 用途 |
|---|---|
| `--title <title>` | 文档标题。 |
| `--format <format>` | 文档格式。默认为 `markdown`。 |
| `--body <markdown>` | 内联正文。 |
| `--body-file <path>` | 从文件读取正文，而不使用 `--body`。 |
| `--change-summary <text>` | 随新修订一同记录的变更摘要。 |
| `--base-revision-id <id>` | 预期当前修订版本，用于乐观并发控制；如果文档已更新到更高版本，本次写入会失败。 |

> **提示：** 多行内容请优先使用 `--body-file`。这样可避免较长 Markdown 出现在 shell 历史中，也无需处理复杂引号。

---

## 工作成果

工作成果是代理针对 issue 产出的结构化交付物。创建和更新都通过 JSON 请求体传递，因此无需为每个字段单独添加一个标志即可使用完整对象结构。

```sh
paperclipai issue work-products <issue-id>
paperclipai issue work-product:create <issue-id> --payload-json '{"kind":"report","title":"Q2 analysis"}'
paperclipai issue work-product:update <work-product-id> --payload-json '{"title":"Q2 analysis (final)"}'
paperclipai issue work-product:delete <work-product-id>
```

| 命令 | 用途 |
|---|---|
| `work-products <issueId>` | 列出 issue 的工作成果。 |
| `work-product:create <issueId> --payload-json <json>` | 根据 `CreateIssueWorkProduct` JSON 请求体创建工作成果。 |
| `work-product:update <workProductId> --payload-json <json>` | 根据 `UpdateIssueWorkProduct` JSON 请求体更新工作成果。注意，此命令接受工作成果 ID，而非 issue ID。 |
| `work-product:delete <workProductId>` | 删除工作成果。 |

---

## 交互

交互是 issue 评论串上的结构化交流，最常见的例子是代理暂停等待输入时发出的 `ask_user_questions`。可使用以下命令检查待处理交互、接受、拒绝、取消或回复交互。

```sh
paperclipai issue interactions <issue-id>
paperclipai issue interaction:create <issue-id> --payload-json '{ ... }'
paperclipai issue interaction:accept <issue-id> <interaction-id> --selected-client-keys a,b
paperclipai issue interaction:reject <issue-id> <interaction-id> --reason "Out of scope"
paperclipai issue interaction:cancel <issue-id> <interaction-id> --reason "No longer needed"
paperclipai issue interaction:respond <issue-id> <interaction-id> --answers-json '[{"key":"q1","value":"yes"}]' --summary-markdown "Confirmed scope"
```

| 命令 | 用途 |
|---|---|
| `interactions <issueId>` | 列出评论串交互。 |
| `interaction:create <issueId> --payload-json <json>` | 根据 `CreateIssueThreadInteraction` 请求体创建交互。 |
| `interaction:accept <issueId> <interactionId>` | 接受交互。`--selected-client-keys <csv>` 可缩小接受的 client key 范围。 |
| `interaction:reject <issueId> <interactionId>` | 拒绝交互。可选 `--reason <text>`。 |
| `interaction:cancel <issueId> <interactionId>` | 取消 `ask_user_questions` 交互。可选 `--reason <text>`。 |
| `interaction:respond <issueId> <interactionId>` | 回复问题交互。`--answers-json <json>` 是回答数组（必填）；`--summary-markdown <markdown>` 是可选摘要。 |

---

## 树状态、预览和暂停记录

树控制接口作用于一个 issue 及其所有后代。你可以检查树状态，在提交前预览控制变更，也可以对整个子树施加或解除暂停（暂停/恢复/取消/还原）。

```sh
paperclipai issue tree-state <root-issue-id>
paperclipai issue tree-preview <root-issue-id> --payload-json '{"mode":"pause"}'
paperclipai issue tree-holds <root-issue-id> --status active --mode pause --include-members
paperclipai issue tree-hold:create <root-issue-id> --payload-json '{"mode":"pause","reason":"awaiting budget"}'
paperclipai issue tree-hold:get <root-issue-id> <hold-id>
paperclipai issue tree-hold:release <root-issue-id> <hold-id> --payload-json '{}'
```

| 命令 | 用途 |
|---|---|
| `tree-state <issueId>` | 获取根 issue 当前的树控制状态。 |
| `tree-preview <issueId> --payload-json <json>` | 预览 `PreviewIssueTreeControl` 变更的影响，不应用变更。 |
| `tree-holds <issueId>` | 列出暂停记录。可使用 `--status active\|released`、`--mode pause\|resume\|cancel\|restore` 和 `--include-members` 筛选。 |
| `tree-hold:create <issueId> --payload-json <json>` | 根据 `CreateIssueTreeHold` 请求体创建暂停记录。 |
| `tree-hold:get <issueId> <holdId>` | 检查单条暂停记录。 |
| `tree-hold:release <issueId> <holdId>` | 解除暂停记录。`--payload-json <json>` 默认为 `{}`。 |

> **提示：** 对大范围任务执行暂停或取消前，务必先运行 `tree-preview`。根 issue 上的暂停操作会传播到整个子树；预览能准确显示会影响哪些 issue。

---

## 附件

可向 issue 或某条评论添加附件，列出、下载和删除附件。

```sh
paperclipai issue attachments <issue-id>
paperclipai issue attachment:upload <issue-id> -C <company-id> --file ./diagram.png
paperclipai issue attachment:upload <issue-id> -C <company-id> --file ./log.txt --comment-id <comment-id>
paperclipai issue attachment:download <attachment-id> --out ./diagram.png
paperclipai issue attachment:delete <attachment-id>
```

| 命令 | 用途 |
|---|---|
| `attachments <issueId>` | 列出 issue 的附件。 |
| `attachment:upload <issueId>` | 上传文件。必须提供 `-C, --company-id <id>` 和 `--file <path>`；传入 `--comment-id <id>` 可将文件附加到指定评论。 |
| `attachment:download <attachmentId>` | 下载附件。`--out <path>` 会将内容写入文件；省略时字节流会输出到 stdout。 |
| `attachment:delete <attachmentId>` | 删除附件。 |

> **注意：** 未指定 `--out` 时，`attachment:download` 会将原始字节写入 stdout。非纯文本文件请将输出重定向到文件（`> out.bin`）。

---

## 标签

标签限定在公司范围内，并由公司中的所有 issue 共用。

```sh
paperclipai issue label:list -C <company-id>
paperclipai issue label:create -C <company-id> --name "needs-review" --color "#4f46e5"
paperclipai issue label:delete <label-id>
```

| 命令 | 用途 |
|---|---|
| `label:list` | 列出公司中的标签；通过 `-C` 限定公司。 |
| `label:create` | 创建标签。必须提供 `--name <name>` 和 `--color <hex>`（例如 `#4f46e5`）；通过 `-C` 限定公司。 |
| `label:delete <labelId>` | 按 ID 删除标签。 |

---

## 审批

可以将已有审批关联到 issue、列出审批或解除关联。要创建审批本身，请参阅[审批命令](approval.md)。

```sh
paperclipai issue approvals <issue-id>
paperclipai issue approval:link <issue-id> <approval-id>
paperclipai issue approval:unlink <issue-id> <approval-id>
```

| 命令 | 用途 |
|---|---|
| `approvals <issueId>` | 列出与 issue 关联的审批。 |
| `approval:link <issueId> <approvalId>` | 关联已有审批。 |
| `approval:unlink <issueId> <approvalId>` | 移除关联。 |

---

## 已读状态与归档状态

这些命令会为当前凭据标记 issue 的收件箱状态，适用于编写收件箱分流脚本。

```sh
paperclipai issue read <issue-id>
paperclipai issue unread <issue-id>
paperclipai issue archive <issue-id>
paperclipai issue unarchive <issue-id>
```

| 命令 | 用途 |
|---|---|
| `read <issueId>` | 将 issue 标记为已读。 |
| `unread <issueId>` | 将 issue 标记为未读。 |
| `archive <issueId>` | 将 issue 从收件箱归档。 |
| `unarchive <issueId>` | 将 issue 恢复到收件箱。 |

---

## 恢复和强制释放

工作卡住时（例如 issue 卡在检出状态，或服务器已标记某项恢复操作），可使用以下命令解除阻塞。

```sh
paperclipai issue recovery-actions <issue-id>
paperclipai issue recovery:resolve <issue-id> --outcome restored --source-issue-status todo --resolution-note "Re-queued after restart"
paperclipai issue force-release <issue-id>
paperclipai issue child:create <parent-issue-id> --payload-json '{"title":"Subtask"}'
```

| 命令 | 用途 |
|---|---|
| `recovery-actions <issueId>` | 列出 issue 的活动恢复操作。 |
| `recovery:resolve <issueId>` | 处理恢复操作（参见下方标志）。 |
| `force-release <issueId>` | 强制从代理检出中释放 issue。当普通 `release` 不够时使用的管理员应急入口。 |
| `child:create <issueId> --payload-json <json>` | 根据 `CreateChildIssue` 请求体在父级下创建子 issue。 |

`recovery:resolve` 标志：

| 标志 | 用途 |
|---|---|
| `--outcome <outcome>` | 可使用 `restored`、`false_positive`、`blocked` 或 `cancelled` 中的一个值（必填）。 |
| `--source-issue-status <status>` | `restored` 结果使用 `todo`、`done` 或 `in_review`；仅 `blocked` 结果允许使用 `blocked`（必填）。 |
| `--action-id <id>` | 同时有多个活动恢复操作时，指定目标操作。 |
| `--resolution-note <text>` | 随处理结果记录的备注。 |

> **警告：** `force-release` 会覆盖代理当前的检出状态。仅当代理确实卡住时使用；正常工作的代理应自行释放任务。

---

## 运行

读取与 issue 关联的心跳运行，包括执行过的运行、进行中的运行和当前活动运行。这些是 issue 产生的服务器端运行，与本地 `run` 引导命令不同。运行命令本身请参阅[运行命令](run.md)。

```sh
paperclipai issue runs <issue-id>
paperclipai issue live-runs <issue-id>
paperclipai issue active-run <issue-id>
```

| 命令 | 用途 |
|---|---|
| `runs <issueId>` | 列出与 issue 关联的所有心跳运行。 |
| `live-runs <issueId>` | 只列出排队中和正在运行的任务。 |
| `active-run <issueId>` | 显示唯一的活动运行；不存在时返回 null。 |

以上三条命令都接受 issue UUID 或标识符。

---

## 反馈投票和轨迹

反馈会记录针对 issue 及其目标提交的投票和轨迹，是评估代理输出的原始材料。更广泛的反馈功能请参阅[反馈命令](feedback.md)。

```sh
paperclipai issue feedback:votes <issue-id>
paperclipai issue feedback:vote <issue-id> --payload-json '{"targetType":"comment","targetId":"<id>","vote":"up"}'
paperclipai issue feedback:list <issue-id> --vote down --status open --include-payload
paperclipai issue feedback:export <issue-id> --from 2026-01-01 --format ndjson --out ./feedback.ndjson
```

| 命令 | 用途 |
|---|---|
| `feedback:votes <issueId>` | 列出 issue 的反馈投票。 |
| `feedback:vote <issueId> --payload-json <json>` | 根据 `UpsertIssueFeedbackVote` 请求体创建或更新投票。 |
| `feedback:list <issueId>` | 列出反馈轨迹（参见下方筛选标志）。 |
| `feedback:export <issueId>` | 将反馈轨迹导出到 stdout 或文件。 |

`feedback:list` 和 `feedback:export` 共用以下筛选与导出标志：

| 标志 | 用途 |
|---|---|
| `--target-type <type>` | 按目标类型筛选。 |
| `--vote <vote>` | 按投票值筛选。 |
| `--status <status>` | 按轨迹状态筛选。 |
| `--from <iso8601>` / `--to <iso8601>` | 限定创建时间范围。 |
| `--shared-only` | 只包含允许分享/导出的轨迹。 |
| `--include-payload` | 包含已存储的请求体快照。`feedback:export` 默认包含请求体。 |

`feedback:export` 另外接受：

| 标志 | 用途 |
|---|---|
| `--out <path>` | 将导出写入文件，而不是 stdout。 |
| `--format <format>` | `json` 或 `ndjson`。默认值为 `ndjson`。 |

---

## 另请参阅

- [通用选项](./common-options.md)——所有客户端命令共用的标志。
- [输出与脚本](./output-and-scripting.md)——在管道中使用 `--json`。
- [审批命令](approval.md)——创建并处理此处关联的审批。
- [运行命令](run.md)——检查和控制心跳运行。
- [反馈命令](feedback.md)——更广泛的反馈和轨迹功能。
- [目标命令](goal.md)和[项目命令](project.md)——issue 所属的容器。
