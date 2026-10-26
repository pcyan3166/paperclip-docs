---
paperclip_version: v2026.1005.0
seo_title: CLI 运行与心跳命令
seo_description: 心跳运行是代理的一次服务器端执行。触发运行、跟踪运行并检查适配器实际执行的操作。
---

# 运行（心跳）命令

*心跳运行*是代理的一次服务器端执行：Paperclip 运行时唤醒代理、提供上下文、运行适配器（LLM、编码或研究工作）并记录结果的过程。`run` 子命令允许你在终端中列出、检查、跟踪和控制这些运行。想查看代理当前正在做什么、阅读日志、跟踪它处理过的 issue、取消异常运行或记录看门狗决策时，都可以使用这些命令，而不必打开 UI。

> **注意：** 不要将这些子命令与不带子命令的 `run` 命令混淆。`paperclipai run`（不带子命令）会在本机引导并启动一个**本地 Paperclip 实例**。本文介绍的 `run` 子命令是 API 客户端，用于观察和控制上下文所指向实例中的心跳运行。它们是完全不同的概念，只是名称相同。

---

## 一句话理解执行模型

代理不是在 CLI 内部运行的。代理被唤醒时（例如通过 `agent wake`、`heartbeat run`、分配任务或在任务上发表评论、例行任务或调度器），**服务器**会运行适配器并生成一次心跳运行。CLI 是查看该运行的窗口：它通过 API 读取运行记录、事件和日志，也可以请求服务器取消运行或记录看门狗决策。你在这里输入的任何命令都不会在本地执行模型——你是在观察和引导服务器端的工作。

理解这一点有助于正确使用这些命令。`run log` 和 `run events` 是对进行中或已完成运行的*读取*，不会“连接”到你拥有的本地进程。`run cancel` 是向服务器发出的请求，不是本地终止进程。应将这些命令视为控制平面调用。

---

## 通用选项

每条 `run` 子命令都接受以下标准客户端选项：

| 标志 | 用途 |
|---|---|
| `--data-dir <path>` | 指定本地 Paperclip 数据目录。 |
| `--api-base <url>` | 覆盖 API 基础 URL；在解析顺序中优先级最高。 |
| `--api-key <token>` | 显式提供令牌，而不使用已保存的配置文件。 |
| `--context <path>` | 使用指定的 `context.json`，而非 `~/.paperclip/context.json`。 |
| `--profile <name>` | 从上下文文件中选择具名角色配置文件。 |
| `--json` | 输出机器可读 JSON，而非行内文本格式。编写脚本时请使用此选项。 |

公司范围子命令 `list` 和 `live` 还接受 `-C, --company-id <id>`。其余子命令直接按运行或操作 ID 定位，不需要公司 ID。

完整标志参考和 API 基础 URL 的确切解析顺序请参阅[通用选项](./common-options.md)；`--json` 用法请参阅[输出与脚本](./output-and-scripting.md)。

---

## 查找公司的运行

使用 `run list` 分页查看公司的心跳运行，也可以按单个代理筛选。只关心当前排队或正在运行的任务时，使用 `run live`。

```sh
paperclipai run list --company-id <company-id>
paperclipai run list --company-id <company-id> --agent-id <agent-id> --limit 50
paperclipai run live --company-id <company-id>
paperclipai run live --company-id <company-id> --limit 50 --min-count 5
```

| 命令 | 标志 | 行为 |
|---|---|---|
| `run list` | `--agent-id <id>`、`--limit <n>` | 列出公司心跳运行，按最近活动时间倒序。可用 `--agent-id` 限定单个代理，用 `--limit` 限制结果数量。 |
| `run live` | `--limit <n>`、`--min-count <n>` | 只列出排队中和运行中的任务。`--min-count <n>` 会在结果不足时补充最近的*已完成*运行，因此即使公司当前很安静，也会显示近期历史记录，而不是空列表。 |

文本模式下，这两条命令都会为每次运行输出一条行内记录，显示运行 `id`、`status`、`agentId`、`invocationSource`、`triggerDetail`、`startedAt`、`finishedAt` 和 `logBytes`。如需完整运行对象，请添加 `--json`。

> **提示：** `run live --json | jq '.[].id'` 是获取正在运行任务 ID 的快捷方法，可将结果直接传给 `run log` 或 `run cancel`。

---

## 检查单次运行

拿到运行 ID 后，可以进一步查看详细信息。

```sh
paperclipai run get <run-id>
paperclipai run events <run-id> --after-seq 0 --limit 200
paperclipai run log <run-id> --offset 0 --limit-bytes 16384
paperclipai run log <run-id> --text
paperclipai run issues <run-id>
```

### `run get`

返回 `<run-id>` 对应的完整心跳运行记录，是查询运行状态、时间信息、触发方式和字节计数的权威来源。

### `run events`

列出运行的结构化事件流，即运行时逐步执行操作的有序记录。

| 标志 | 默认值 | 用途 |
|---|---|---|
| `--after-seq <n>` | `0` | 只返回此序号之后的事件。增量轮询时，可传入上次看到的最高 `seq`。 |
| `--limit <n>` | `200` | 最多返回的事件数量。 |

文本模式下，每个事件会输出一条行内记录，包含 `seq`、`eventType`、`stream`、`level` 和 `message`。跟踪实时运行时，可短间隔轮询，并将 `--after-seq` 更新为上次收到的最后一个 `seq`。

### `run log`

读取运行的原始日志字节。日志按字节寻址，因此要使用偏移量而不是行号分页读取。

| 标志 | 默认值 | 用途 |
|---|---|---|
| `--offset <bytes>` | `0` | 从指定字节偏移量开始读取。 |
| `--limit-bytes <bytes>` | 未设置 | 最多读取的字节数。省略时由服务器决定。 |
| `--text` | 关闭 | 当 API 返回 `text` 字段时，仅将该文本输出到 stdout，不附带外层记录。 |

可将 `run get` 或 `run list` 返回的 `logBytes` 作为游标：读取一段后，将已读取的字节数加到 `--offset` 上，再继续读取，以跟踪不断增长的日志。需要通过管道传递原始输出或直接阅读时使用 `--text`；否则命令会返回结构化封装对象。

### `run issues`

列出与一次运行关联的 issue，即代理在该次执行中创建、检出、评论或完成的任务。每行显示 issue 的 `identifier`、`id`、`status`、`priority`、`title` 以及关联回运行的 `runStatus`。它将“这次运行做了什么”与“哪些工作发生了变化”关联起来，也是进入 [`issue`](./issue.md) 命令的自然入口。

---

## 取消运行

```sh
paperclipai run cancel <run-id>
```

请求取消排队中或正在运行的心跳任务。这是服务器端请求：运行时会停止该任务并返回更新后的记录（如果没有可取消的任务则返回 `null`）。运行卡住、循环执行或在错误任务上持续消耗预算时，可使用此命令。工作是在服务器端运行的，因此取消请求才是正确的控制方式；本地没有可中断的进程。

> **警告：** 取消运行不会回滚代理已执行的操作。它已更新的 issue、发布的评论或完成的工作区操作都会保留。取消后可运行 `run issues <run-id>` 检查最终状态。

---

## 工作区操作

运行在执行工作区中运行命令（例如构建、测试套件或 Git 操作）时，每项操作都会作为*工作区操作*附加到运行记录。你可以检查这些操作，准确查看代理运行了什么并读取输出。

```sh
paperclipai run workspace-operations <run-id>
paperclipai run workspace-log <operation-id> --offset 0 --limit-bytes 16384
paperclipai run workspace-log <operation-id> --text
```

`run workspace-operations <run-id>` 会列出该次运行的操作，每行显示 `id`、`status`、`phase`、`command`、`cwd` 和 `logBytes`。从列表中取出操作 `id`，传给 `run workspace-log` 以读取该操作的输出。

`run workspace-log` 按字节寻址，接受与 `run log` 相同的日志标志：

| 标志 | 默认值 | 用途 |
|---|---|---|
| `--offset <bytes>` | `0` | 从指定字节偏移量开始读取。 |
| `--limit-bytes <bytes>` | 未设置 | 最多读取的字节数。 |
| `--text` | 关闭 | API 返回 `text` 字段时只输出文本。 |

> **注意：** `run log` 读取整次运行的总日志；`run workspace-log` 读取运行中某一条具体命令的日志。排查构建或测试失败时，真正的错误通常在工作区日志中。

---

## 看门狗决策

看门狗是服务器端监控器，会标记看似卡住、失控或偏离正常流程的运行。`run watchdog-decision` 用于记录你对已标记运行的判断，以便监控器决定后续操作。

```sh
paperclipai run watchdog-decision <run-id> --decision continue --reason "Long compile is expected here"
paperclipai run watchdog-decision <run-id> --decision snooze --snoozed-until 2026-06-01T18:00:00Z
paperclipai run watchdog-decision <run-id> --decision dismissed_false_positive --evaluation-issue-id <issue-id>
```

`--decision` 为必填项，值必须是以下之一：

| 决策 | 含义 |
|---|---|
| `continue` | 允许运行继续；你判断它运行正常。 |
| `snooze` | 暂停看门狗操作，直到未来某个时间。使用 `--snoozed-until <iso8601>` 指定时间。 |
| `dismissed_false_positive` | 看门狗判断有误，将告警标记为误报。 |

| 标志 | 用途 |
|---|---|
| `--decision <decision>` | **必填。** 可选值为 `snooze`、`continue` 或 `dismissed_false_positive`。 |
| `--reason <text>` | 对决策的自由文本说明。请记录理由，方便后续操作员了解情况。 |
| `--snoozed-until <iso8601>` | 暂停看门狗操作的截止时间。`snooze` 决策必填。 |
| `--evaluation-issue-id <id>` | 相关的看门狗评估 issue（如果存在）。 |

> **提示：** 做出看门狗决策前，可先快速查看 `run events` 和 `run workspace-log`。只有在了解运行看似缓慢的原因后，才决定 `continue`；长时间但正常的测试与程序挂起看起来很像，日志能帮助区分两者。

---

## 常见的观察流程

通过终端查看公司运行情况时，通常按以下步骤操作：

```sh
# 1. See what is live right now.
paperclipai run live --company-id <company-id> --min-count 5

# 2. Drill into a run that looks interesting or stuck.
paperclipai run get <run-id>
paperclipai run events <run-id> --limit 50

# 3. Read the raw log, or the failing command's log.
paperclipai run log <run-id> --text
paperclipai run workspace-operations <run-id>
paperclipai run workspace-log <operation-id> --text

# 4. See what work it changed, then act.
paperclipai run issues <run-id>
paperclipai run cancel <run-id>          # if it has gone wrong
```

这是通过 CLI 运维公司的、以读取为主的工作流程。修改状态的部分——唤醒代理、分配工作、发布评论——由 [`agent`](./agent.md) 和 [`issue`](./issue.md) 命令负责。

---

## 另请参阅

- [代理命令](./agent.md)——唤醒代理并设置本地代理 CLI 会话；这里所观察的运行由代理产生。
- [Issue 命令](./issue.md)——管理运行创建或涉及的任务。
- [提示命令](./prompt.md)——将工作作为任务交给代理；任务会唤醒代理并产生一次运行。
- [仪表板命令](./dashboard.md)——查看 UI 中显示的相同公司顶层状态。
- [活动命令](./activity.md)——查看运行引起变更的审计记录。
- [通用选项](./common-options.md)——共享标志和 API 基础 URL 解析方式。
- [输出与脚本](./output-and-scripting.md)——使用 `--json` 并通过管道处理运行数据。
