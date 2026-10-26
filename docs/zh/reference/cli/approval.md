---
paperclip_version: v2026.609.0
seo_title: CLI 审批命令
seo_description: 在终端中处理受控制面板审批约束的操作，例如聘用智能体或确定 CEO 策略；这些操作需要人工或获授权的自动化签核。
---

# 审批命令

当智能体遇到受控制面板审批约束的操作（例如聘用其他智能体、确定 CEO 策略），需要由人工（或经控制面板授权的自动化）决定时，请使用这些命令。审批是位于智能体*想做的事*与它*获准做的事*之间的治理层。`approval` 命令组可用于列出队列、检查单个请求、创建请求、作出决定（批准、拒绝、要求修改）、重新提交修改后的请求，以及发表评论；这一切都可在终端中完成，无需打开 Paperclip 界面。

这些命令与界面中的审批队列功能一致，可用于脚本、集成或无界面操作。如果你将 CLI 交给 AI 操作员，它也可以使用这些命令发起和处理审批。

> **注意：** 审批决定由 **board** 身份作出。智能体身份限定在一个公司和一个智能体中，负责*请求*审批；清理审批队列则属于控制面板操作。

---

## 命令摘要

| 命令 | 用途 |
|---|---|
| `approval list` | 列出公司的审批，可按状态筛选。 |
| `approval get` | 按 ID 获取单项审批。 |
| `approval create` | 为受治理的操作创建审批请求。 |
| `approval approve` | 批准待处理请求。 |
| `approval reject` | 拒绝请求。 |
| `approval request-revision` | 将请求退回给请求方修改。 |
| `approval resubmit` | 重新提交请求，可选择附带新的负载。 |
| `approval comment` | 在审批上发表评论。 |

每个子命令都接受[通用客户端选项](./common-options.md)（`--data-dir`、`--api-base`、`--api-key`、`--context`、`--profile`、`--json`）。按公司限定范围的子命令（`list`、`create`）还必须提供 `-C, --company-id`。单项审批子命令直接操作审批 ID，无需公司 ID。

---

## 列出审批

列出关联到某公司的审批。这就是待分流的队列。

```sh
paperclipai approval list --company-id <company-id>
paperclipai approval list --company-id <company-id> --status pending
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | **必填。** 要查看其审批的公司。 |
| `--status <status>` | 将列表筛选为单一状态（例如 `pending`）。 |

不带 `--json` 时，每行以紧凑内联记录形式显示审批 `id`、`type`、`status` 以及提出请求的智能体或用户。带上 `--json` 时会获得原始数组，适用于编写脚本或将结果传给其他工具。

```sh
# Find everything still waiting on a decision, as JSON, for a script
paperclipai approval list --company-id <company-id> --status pending --json
```

> **提示：** 每次分流时先运行 `approval list --status pending`，这是从“有哪些事项需要我处理”到作出决定的最快方式。

---

## 获取单项审批

作出决定前，完整检查单项审批。

```sh
paperclipai approval get <approval-id>
```

此命令不限定公司范围：审批 ID 可全局访问，因此无需传入 `--company-id`。如果你需要完整负载（例如拟议的聘用配置、策略文本和关联任务 ID）而不只是摘要，可搭配 `--json` 使用。

---

## 创建审批

为受治理的操作创建请求。智能体通常会在工作流程中自行创建请求，但控制面板操作员或自动化也可以明确创建。

```sh
paperclipai approval create \
  --company-id <company-id> \
  --type hire_agent \
  --payload '{"name":"CTO","role":"engineering_lead"}' \
  --requested-by-agent-id <agent-id> \
  --issue-ids <issue-id-1>,<issue-id-2>
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | **必填。** 审批所属公司。 |
| `--type <type>` | **必填。** 审批类型：`hire_agent` 或 `approve_ceo_strategy`。 |
| `--payload <json>` | **必填。** JSON **对象**形式的审批负载（不能是数组或标量）。 |
| `--requested-by-agent-id <id>` | 发起请求的智能体。 |
| `--issue-ids <csv>` | 以逗号分隔的任务 ID，用于关联到该审批。 |

`--payload` 的值会被解析为 JSON，且必须是 JSON 对象——传入列表或裸字符串会被拒绝，并报错 `Invalid payload JSON`。`--issue-ids` 的值会按逗号拆分、去掉首尾空白，并丢弃空条目。

> **警告：** 请将 `--payload` 值作为单个 shell 参数引用，并确保 JSON 有效。被截断或格式为数组的负载会在请求发送到服务器之前失败。

---

## 决定审批

三个子命令都可对待处理审批作出决定。它们都将审批 ID 作为参数，并接受相同的两个可选标志。

```sh
paperclipai approval approve <approval-id> --decision-note "Cleared budget; proceed with the hire."
paperclipai approval reject <approval-id> --decision-note "Out of scope for this quarter."
paperclipai approval request-revision <approval-id> --decision-note "Tighten the role to backend only, then resubmit."
```

| 标志 | 用途 |
|---|---|
| `--decision-note <text>` | 解释决定的简短说明。 |
| `--decided-by-user-id <id>` | 将此 ID 作为作出决定的用户 ID。 |

| 命令 | 适用情况 |
|---|---|
| `approve` | 请求合理，且你批准执行相应操作。 |
| `reject` | 不应继续执行此请求。 |
| `request-revision` | 请求大体可行但需要修改——将其退回给请求方，而不是直接终止。 |

`request-revision` 是高效的折中办法：它保留请求，并准确说明需要修改的内容，让请求方修正后重新提交，而不必从头开始。

> **提示：** 简短、具体的决定说明比长篇大论更有价值。请求方应该可以根据反馈采取行动，而不需要猜测——这正是 `request-revision` 的目的。

---

## 重新提交修改后的请求

收到 `request-revision` 后，请求方会修正提案并重新提交。你可以原样重新提交，也可以在同一步中提供修正后的负载。

```sh
# Resubmit unchanged
paperclipai approval resubmit <approval-id>

# Resubmit with a corrected payload
paperclipai approval resubmit <approval-id> --payload '{"name":"CTO","role":"backend_lead"}'
```

| 标志 | 用途 |
|---|---|
| `--payload <json>` | 可选替换负载，必须是 JSON 对象。 |

传入 `--payload` 时，会像 `approval create` 一样解析并验证 JSON 对象；省略时则原样重新提交现有负载。这样可以完成“请求 → 修改 → 重新提交”的闭环，而无需从头新建审批。

---

## 在审批上发表评论

可以在审批上发表评论以提出澄清问题或补充背景，而不作出决定。

```sh
paperclipai approval comment <approval-id> --body "What is the projected monthly cost of this hire?"
```

| 标志 | 用途 |
|---|---|
| `--body <text>` | **必填。** 评论内容。 |

决定之前可用评论沟通具体细节。只有确实希望请求方修改并重新提交提案时，才使用 `request-revision`。

---

## 常见分流流程

```sh
# 1. See what is waiting
paperclipai approval list --company-id <company-id> --status pending

# 2. Inspect the one that needs attention
paperclipai approval get <approval-id> --json

# 3a. If it's good, approve it
paperclipai approval approve <approval-id> --decision-note "Approved."

# 3b. If it needs work, send it back
paperclipai approval request-revision <approval-id> --decision-note "Narrow the scope, then resubmit."

# 4. The requester fixes and resubmits
paperclipai approval resubmit <approval-id> --payload '{"...":"..."}'
```

---

## 另请参阅

- [通用选项](./common-options.md)——每个审批子命令接受的共享客户端标志（`--company-id`、`--api-base`、`--api-key`、`--json` 等）。
- [输出和脚本](./output-and-scripting.md)——管道处理时 `--json` 输出的格式。
- [任务命令](./issue.md)——使用任务审批链接命令，将审批关联到触发该请求的工作。
- [智能体命令](./agent.md)——请求聘用或执行受审批约束工作的智能体。
- [活动](./activity.md)——记录审批决定的审计日志。
