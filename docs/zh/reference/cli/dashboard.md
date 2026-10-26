---
paperclip_version: v2026.609.0
seo_title: CLI 控制面板命令
seo_description: 在 shell 中快速查看公司概况，一次调用即可获取与控制面板显示相同的顶层数据。
---

# 控制面板命令

`dashboard` 命令可在终端中快速查看公司概况，一次调用即可获取真人在 Paperclip 控制面板上看到的相同顶层数据。需要快速检查健康状态时可使用此命令，例如了解有多少智能体正在运行、还有多少未完成工作、本月支出如何，以及是否有事项等待人工处理。此命令只读，可以随时运行、循环执行，或放入监控脚本中。

---

## 概览

此命令只有一个子命令：

```sh
paperclipai dashboard get --company-id <company-id>
```

`dashboard get` 会发出一次 `GET /api/companies/<company-id>/dashboard` 请求并打印摘要。它不会更改状态、触发运行或唤醒智能体，只会报告信息。因此，不必分别翻阅 `issue list`、`agent list`、`approval list` 和 `cost summary`，就能回答“这家公司现在是什么状态？”，是最省力的方式。

此命令按公司限定范围，因此必须指定公司。它不接受位置参数形式的公司 ID；请通过 `-C` / `--company-id` 指定公司，或让命令从环境变量或配置文件中解析（见[解析公司](#解析公司)。

---

## `dashboard get`

获取单个公司的控制面板摘要。

```sh
paperclipai dashboard get --company-id <company-id>
```

### 选项

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 要生成摘要的公司。除非可从 `PAPERCLIP_COMPANY_ID` 或上下文配置文件中解析，否则必填。 |
| `--json` | 以 JSON 格式输出原始摘要对象，而非格式化的人类可读视图。编写脚本时使用。 |
| `--api-base <url>` | 覆盖此次调用使用的 API 基础 URL。 |
| `--api-key <token>` | 通过智能体身份验证进行调用时使用的 Bearer 令牌。 |
| `--context <path>` | CLI 上下文文件路径。 |
| `--profile <name>` | 要使用的 CLI 上下文配置文件。 |
| `-d, --data-dir <path>` | Paperclip 数据目录根路径（将状态与 `~/.paperclip` 隔离）。 |
| `-c, --config <path>` | Paperclip 配置文件路径（用于推断本地 API 基础地址）。 |

这些是所有按公司限定范围的 CLI 命令共享的标准客户端标志。身份验证、上下文和 API 地址的完整解析方式请参阅[通用选项](./common-options.md)。

> **注意：** `dashboard get` 是少数在命令定义中明确要求 `--company-id` 的命令之一。即使配置文件中已有默认公司，编写脚本时仍建议明确传入 `-C`，以消除目标歧义。

---

## 解析公司

系统按以下顺序解析公司，并使用第一个匹配项：

1. 命令行中的 `-C` / `--company-id`。
2. `PAPERCLIP_COMPANY_ID` 环境变量。
3. 所选上下文配置文件中记录的 `companyId`。

如果以上来源均无法确定公司，命令会明确报错，提示传入 `--company-id`、设置 `PAPERCLIP_COMPANY_ID`，或使用 `paperclipai context set` 设置配置文件默认值。

如果智能体身份配置文件已固定到某家公司，则无需任何标志即可运行命令：

```sh
paperclipai dashboard get
```

如果控制面板操作员需要在多家公司间工作，则每次都应明确指定目标：

```sh
paperclipai dashboard get --company-id <company-id>
```

---

## 返回内容

摘要是一个对象，涵盖了原本需要手动汇总的各个方面。其结构保持稳定，因此适合用于脚本：

| 字段 | 含义 |
|---|---|
| `companyId` | 摘要所描述的公司。 |
| `agents` | 处于 `active`、`running`、`paused` 和 `error` 状态的智能体数量。 |
| `tasks` | 按类别统计的任务数量：`open`、`inProgress`、`blocked` 和 `done`。 |
| `costs` | 当前月份的 `monthSpendCents`、`monthBudgetCents` 和 `monthUtilizationPercent`。 |
| `pendingApprovals` | 等待人工决策的审批数量。 |
| `budgets` | 治理状态信号：`activeIncidents`、`pendingApprovals`、`pausedAgents` 和 `pausedProjects`。 |
| `runActivity` | 按天统计的心跳运行结果序列，每项包含 `date`、`succeeded`、`failed`、`other` 和 `total`。 |

金额以美分为单位，除以 100 即为美元。`costs` 区块中的数据可以通过[`cost` 命令](./cost.md)进一步拆分；控制面板显示总览数值，`cost` 则提供归因明细。

> **提示：** `budgets.pausedAgents` 和 `budgets.pausedProjects` 可快速提示你自动化是否因预算触发而停止。工作似乎停滞时，请先检查这两项。

---

## 输出格式

默认情况下，命令会以简洁、易读的格式打印摘要对象，适合在终端中快速查看。

需要程序处理时，请传入 `--json` 并解析结果。JSON 输出就是原始摘要，因此上表中的所有字段都可用：

```sh
paperclipai dashboard get --company-id <company-id> --json
```

将输出传给 `jq` 可提取所需内容。以下是一些实用的单行命令：

```sh
# Month-to-date budget utilization as a percentage
paperclipai dashboard get -C <company-id> --json | jq '.costs.monthUtilizationPercent'

# How many approvals are waiting on a human
paperclipai dashboard get -C <company-id> --json | jq '.pendingApprovals'

# Agents currently in an error state
paperclipai dashboard get -C <company-id> --json | jq '.agents.error'

# Failed runs over the reported activity window
paperclipai dashboard get -C <company-id> --json | jq '[.runActivity[].failed] | add'
```

退出码、`--json` 行为以及如何串联 CLI 调用等脚本约定，详见[输出和脚本](./output-and-scripting.md)。

---

## 适用场景

`dashboard get` 是在终端中查看 UI 控制面板的等效方式：返回全公司的摘要数据，供人工阅读或脚本处理。你不想打开浏览器，或希望通过自动化（或已配置的 AI 操作员）轮询公司健康状态时，可以使用它。

实用的监控循环如下：

1. 运行 `paperclipai dashboard get -C <company-id>` 获取概要状态。
2. 如果 `pendingApprovals` 非零，使用[`approval list`](./approval.md)继续处理。
3. 如果 `agents.error` 或 `budgets.pausedAgents` 非零，使用 [`agent`](./agent.md) 和 [`activity list`](./activity.md) 排查原因。
4. 如果 `costs.monthUtilizationPercent` 正接近预算上限，使用[`cost`](./cost.md)拆分费用。

控制面板告诉你*是否*需要深入检查；范围更明确的命令则告诉你*发生了什么*。

> **注意：** `dashboard get` 读取的是公司当前状态，并非实时数据流。要观察正在执行的心跳运行，请使用 [`run`](./run.md) 检查子命令；要跟踪审计记录，请使用 [`activity`](./activity.md)。

---

## 另请参阅

- [通用选项](./common-options.md)——所有客户端命令共享的身份验证、上下文和 API 地址解析规则。
- [输出和脚本](./output-and-scripting.md)——`--json` 约定和 CLI 调用串联方式。
- [费用命令](./cost.md)——按智能体、模型、服务提供商、付款方或项目拆分概要支出。
- [审批命令](./approval.md)——审核并处理控制面板统计的待处理审批。
- [活动命令](./activity.md)——记录智能体和运行状态变化的审计日志。
- [智能体命令](./agent.md)——检查控制面板统计的智能体。
- [运行命令](./run.md)——检查 `runActivity` 序列所依据的心跳运行。
