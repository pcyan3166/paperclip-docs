---
paperclip_version: v2026.609.0
seo_title: CLI 成本、财务与预算命令
seo_description: 查看公司的支出、记录手动财务事件，并设置代理支出上限。
---

# 成本、财务与预算命令

需要查看公司支出、记录手动财务事件或设置代理支出护栏时，使用这些命令。它们在终端中提供与 UI 相同的成本视图和预算控制，适用于生成支出报告、将预算接入自动化流程或无头环境操作（也包括配置好的 AI 操作员）。

命令分为三个顶层组：`cost`（读取支出并记录成本事件）、`finance`（记录和汇总财务事件）以及 `budget`（读取预算状态并编写控制支出的策略）。所有读取操作都不会修改数据；只有 `event:create`、`policy:upsert`、`company:update`、`agent:update` 和 `incident:resolve` 会改变状态。

> **注意：** 此处大多数命令都限定在公司范围内。使用 `-C, --company-id <id>` 显式指定公司，或者让 CLI 从所选上下文配置文件解析公司。`budget company:update` 和 `budget incident:resolve` 必须指定公司才能运行。另外两个命令通过位置参数确定作用范围，不接受 `--company-id`：`cost issue <issue-id>`（服务器根据 issue 解析公司）和 `budget agent:update <agent-id>`（代理 ID 直接定位预算）。

---

## 通用选项

此页中的每条命令都接受标准客户端标志。完整列表请参阅[通用选项](./common-options.md)，`--json` 用法请参阅[输出与脚本](./output-and-scripting.md)。

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 目标公司。公司范围命令接受此选项；除非上下文配置文件已选定公司，否则必须提供。`cost issue` 和 `budget agent:update` 按位置参数确定范围，不接受此选项。 |
| `--api-base <url>` | 覆盖 API 基础 URL。 |
| `--api-key <token>` | 覆盖凭据。 |
| `--context <path>` | 使用指定上下文文件，而不是 `~/.paperclip/context.json`。 |
| `--profile <name>` | 选择具名角色配置文件。 |
| `--data-dir <path>` | 覆盖本地数据目录。 |
| `--json` | 输出原始 JSON，而非格式化表格。使用脚本调用这些命令时应启用此选项。 |

> **提示：** 读取成本低且幂等。可以在监控循环中使用 `--json` 调用 `cost summary`、`budget overview` 和 `cost window-spend`，再将输出传入自己的仪表板或告警系统。

---

## 查看成本

`cost` 组会按你关心的维度，汇总报告公司累计支出。以下子命令均读取公司的成本汇总数据；选择能回答你问题的细分方式即可。

```sh
paperclipai cost summary --company-id <company-id>
paperclipai cost by-agent --company-id <company-id>
paperclipai cost by-agent-model --company-id <company-id>
paperclipai cost by-provider --company-id <company-id>
paperclipai cost by-biller --company-id <company-id>
paperclipai cost by-project --company-id <company-id>
paperclipai cost window-spend --company-id <company-id>
paperclipai cost quota-windows --company-id <company-id>
```

| 命令 | 用途 |
|---|---|
| `cost summary` | 查看公司的总支出概览，从这里开始。 |
| `cost by-agent` | 查看哪些代理在产生支出，用于发现支出失控的代理。 |
| `cost by-agent-model` | 按所用模型细分每个代理的支出，是调整模型选择时最细致的视图。 |
| `cost by-provider` | 按上游服务提供商汇总支出。 |
| `cost by-biller` | 按计费对象汇总支出。 |
| `cost by-project` | 按项目汇总支出，以便将费用归因到产生费用的工作。 |
| `cost window-spend` | 查看当前滚动预算窗口内的支出，这是预算策略实际检查的数值。 |
| `cost quota-windows` | 查看已配置的配额窗口及其周期和限额。 |

> **提示：** 预算告警触发时，最快的排查方式是先运行 `cost window-spend` 查看超出限额的程度，再运行 `cost by-agent` 找出来源。

### 查看单个 issue 的成本

要查看单个 issue 从头到尾产生的成本，请传入其 ID 或人类可读标识符（例如 `PAP-39`）：

```sh
paperclipai cost issue <issue-id>
```

这是唯一一个在命令行上不限定公司范围的 `cost` 读取命令——服务器会根据 issue ID 解析公司。在复查高成本任务、判断支出是否物有所值时可以使用它。

### 记录成本事件

代理运行时的大多数成本都会由服务器端运行时自动记录。如果需要补记运行时未捕获的支出（例如归属于该公司的单笔外部费用），可以直接提交成本事件：

```sh
paperclipai cost event:create --company-id <company-id> --payload-json '{...}'
```

必须提供 `--payload-json`，且其值必须是描述成本事件的有效 JSON 对象。CLI 会解析该值，并将其原样 POST 到公司的 `cost-events` 端点，因此请求体结构应符合服务器的成本事件 schema。

> **警告：** `--payload-json` 会通过 `JSON.parse` 解析。JSON 格式错误时，命令会在发送请求前失败。请在 shell 中用单引号括起整个值，以保留其中的双引号。

---

## 记录和读取财务信息

`cost` 跟踪模型和运行时支出；`finance` 组则跟踪更广泛的财务事件，包括公司需要记账的收入、费用和其他资金变动，并提供汇总结果。

```sh
paperclipai finance event:create --company-id <company-id> --payload-json '{...}'
paperclipai finance events --company-id <company-id>
paperclipai finance summary --company-id <company-id>
paperclipai finance by-biller --company-id <company-id>
paperclipai finance by-kind --company-id <company-id>
```

| 命令 | 用途 |
|---|---|
| `finance event:create` | 记录财务事件。必须提供 `--payload-json`，请求会 POST 到公司的 `finance-events` 端点。 |
| `finance events` | 列出公司已记录的财务事件。 |
| `finance summary` | 查看公司的综合财务概况。 |
| `finance by-biller` | 按计费对象汇总财务金额。 |
| `finance by-kind` | 按事件类型汇总财务金额。 |

当 AI 操作员完成交易、登记费用或需要记录其他资金事实时，使用 `finance event:create`；使用读取命令将这些记录与成本数据对账。

---

## 预算护栏

`budget` 组用于管理自主支出。预算策略设置运行时在代理工作时强制执行的限额，防止代理在两次人工检查之间悄悄耗尽账户。无头方式运行公司时，这部分尤其重要：应在放开代理工作之前设置预算，而不是事后补设。

```sh
paperclipai budget overview --company-id <company-id>
paperclipai budget policy:upsert --company-id <company-id> --payload-json '{...}'
paperclipai budget company:update --company-id <company-id> --payload-json '{...}'
paperclipai budget agent:update <agent-id> --payload-json '{...}'
paperclipai budget incident:resolve <incident-id> --company-id <company-id>
```

### 读取当前状态

```sh
paperclipai budget overview --company-id <company-id>
```

`budget overview` 是起点：它会返回公司已配置的预算以及当前支出相对预算的情况。修改前先读取一次，修改后再读取一次以确认更改已生效。

### 设置预算策略

```sh
paperclipai budget policy:upsert --company-id <company-id> --payload-json '{...}'
```

`policy:upsert` 会创建预算策略或更新现有策略，并将必需的 `--payload-json` POST 到公司的 `budgets/policies` 端点。服务器会依据该策略判断是否允许某次心跳运行产生支出。可结合 `cost quota-windows` 和 `cost window-spend` 验证窗口和限额是否符合预期。

### 更新公司预算

```sh
paperclipai budget company:update --company-id <company-id> --payload-json '{"...":"..."}'
```

`company:update` 使用 `UpdateBudget` JSON 请求体 PATCH 公司的预算。必须提供 `--payload-json`。可用它调整公司总上限；无论哪个代理产生支出，该上限都适用。

### 更新代理预算

```sh
paperclipai budget agent:update <agent-id> --payload-json '{"...":"..."}'
```

`agent:update` 使用 `UpdateBudget` JSON 请求体 PATCH 单个代理的预算。代理 ID 是位置参数，且必须提供 `--payload-json`。可为值得信任的资深代理提供比新加入代理更大的额度，也可收紧出现过超支情况的代理的额度。

> **注意：** 公司总预算和代理个人预算会共同生效。收紧某个代理的预算不会提高公司上限；所有代理的支出总和仍受公司上限约束。

### 解决预算事件

支出触发预算护栏时，服务器会生成预算事件。在解决之前，受影响的工作会继续受限。决定如何处理后再解决：

```sh
paperclipai budget incident:resolve <incident-id> --company-id <company-id>
paperclipai budget incident:resolve <incident-id> --company-id <company-id> --payload-json '{"...":"..."}'
```

事件 ID 是位置参数。此处 `--payload-json` 为可选项，默认值为 `{}`；只有解决操作需要结构化输入（例如备注或指定处置方式）时，才传入 `ResolveBudgetIncident` 请求体。命令会 POST 到公司的 `budget-incidents/<incident-id>/resolve` 端点。

> **警告：** 解决事件会解除它触发的限制。如果超支仍在继续，应先通过 `budget policy:upsert`、`company:update` 或 `agent:update` 调整预算策略，再解决事件；否则下一次运行时该事件会再次触发。

---

## 实用的预算工作流

无头方式启动公司时，按以下顺序操作更稳妥：

1. `budget policy:upsert`——在任何代理运行前定义窗口和限额。
2. `budget company:update`——设置公司总上限。
3. `budget agent:update <agent-id>`——根据需要为单个代理设置更宽松或更严格的额度。
4. 运行 `budget overview` 和 `cost window-spend`，确认护栏显示符合预期。
5. 公司运行期间，使用 `cost summary`、`cost by-agent` 和 `cost window-spend` 监控。
6. 事件触发后，使用 `cost by-agent` 排查并调整相应预算，然后运行 `budget incident:resolve <incident-id>`。

---

## 另请参阅

- [通用选项](./common-options.md)——此处所有命令都接受的客户端标志。
- [输出与脚本](./output-and-scripting.md)——使用 `--json` 将读取结果接入监控。
- [仪表板命令](./dashboard.md)——公司顶层状态，包括支出视图。
- [活动命令](./activity.md)——预算事件和支出背后的审计记录。
- [代理命令](./agent.md)——`cost by-agent` 和 `budget agent:update` 管理其支出的代理。
- [审批命令](./approval.md)——与预算配合使用、用于高成本操作的看板关卡。
