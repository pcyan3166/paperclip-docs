---
paperclip_version: v2026.609.0
seo_title: CLI 例行任务命令
seo_description: 管理定期和事件驱动的工作：例行任务创建的请求体、状态、修订历史以及触发器。
---

# 例行任务命令

例行任务是 Paperclip 中定期或由事件驱动的工作。一个例行任务包含描述待创建工作的请求体、状态、修订历史以及一个或多个触发器；触发器可以是计划任务、Webhook 或公开 API 端点。需要让 AI 设置自动运行的自动化（例如每晚摘要、将外部事件转换成任务的 webhook）、检查例行任务执行情况，或在紧急情况下暂停所有工作时，请使用这些命令。单数形式的 `routine` 命令组调用控制平面 REST API；复数形式的 `routines` 命令组是用于本地数据库维护的应急入口。

> **注意：** 与 Paperclip 中其他工作对象一样，例行任务实际创建的工作由服务器端运行。这些命令通过 API 创建、配置、触发和观察例行任务，本身不会执行任务。参阅[工作在哪里运行](../../guides/welcome/key-concepts.md)。

---

## 两个命令组

此处有两个命令组，不可互换。

| 命令组 | 调用对象 | 适用情况 |
|---|---|---|
| `routine`（单数） | 控制平面 REST API | 日常操作：列出、创建、更新、触发和检查例行任务及其触发器。 |
| `routines`（复数） | 直接操作已配置的本地数据库 | 仅用于本地维护：暂停本地实例中某个公司的所有例行任务。 |

`routine` 命令接受标准客户端标志（`--api-base`、`--api-key`、`--context`、`--profile`、`--json` 和 `--data-dir`）。唯一的 `routines disable-all` 命令完全不调用 API，而是直接打开本地 Paperclip 数据库并修改例行任务状态。

---

## `routine list`

列出公司中的例行任务。此命令限定在公司范围内，因此需要公司 ID。

```sh
paperclipai routine list --company-id <company-id>
paperclipai routine list --company-id <company-id> --project-id <project-id>
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 要列出例行任务的公司（必填；可回退到上下文中的公司）。 |
| `--project-id <id>` | 只列出关联到指定项目的例行任务。 |

---

## `routine get`

按 ID 获取一个例行任务，包括其当前请求体、状态和触发器。

```sh
paperclipai routine get <routine-id>
```

---

## `routine create`

在公司中创建例行任务。定义以 JSON 请求体提供，使 CLI 与 API schema 保持同步——API 接受的字段都可以在这里传入。

```sh
paperclipai routine create --company-id <company-id> --payload-json '{
  "name": "Nightly status digest",
  "description": "Summarize the day's closed issues",
  "status": "active"
}'
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 创建例行任务所属的公司（必填）。 |
| `--payload-json <json>` | JSON 字符串形式的例行任务定义（必填）。 |

> **提示：** 建议将 JSON 请求体构建在变量或文件中再传入，而不是直接手动输入。这样可以重复执行创建操作，之后也容易将请求体与 `routine get` 的结果进行差异比较。

---

## `routine update`

部分更新已有例行任务。以 JSON 请求体只传入要更改的字段——它沿用 `create` 的 schema，可用于暂停例行任务（`{"status":"paused"}`）、重命名或重写请求体。

```sh
paperclipai routine update <routine-id> --payload-json '{"status": "paused"}'
```

| 标志 | 用途 |
|---|---|
| `--payload-json <json>` | JSON 字符串形式的待更新字段（必填）。 |

每次更新都会创建一个新修订版本，因此下文的修订历史命令很有用。

---

## 修订版本

例行任务的编辑都有版本记录。每次 `update`（以及服务器记录到该例行任务上的触发器变更）都会产生一个修订版本，可通过下列命令列出或回滚。

### `routine revisions`

按时间倒序列出例行任务的修订历史。

```sh
paperclipai routine revisions <routine-id>
```

### `routine revision:restore`

恢复到某个先前的修订版本。它不会回退历史，而是根据选定版本创建新的最新修订，将该版本设为当前定义，因此回滚操作本身也会留下审计记录。

```sh
paperclipai routine revision:restore <routine-id> <revision-id>
```

两个参数均为位置参数：先传例行任务 ID，再传从 `routine revisions` 获取的修订版本 ID。

---

## 运行

一次例行任务*运行*表示例行任务被触发一次，即计划、webhook 或手动触发器所产生的执行。

### `routine runs`

列出某个例行任务最近的运行。可用它确认计划是否确实触发，或排查 webhook 为什么没有产生工作。

```sh
paperclipai routine runs <routine-id>
paperclipai routine runs <routine-id> --limit 50
```

| 标志 | 用途 |
|---|---|
| `--limit <n>` | 限制返回的运行数量。省略时使用服务器默认值。 |

### `routine run`

不受计划时间限制，立即触发例行任务。可选请求体会与运行输入合并，适合测试、一次性调用或触发已配置但尚无计划的例行任务。

```sh
paperclipai routine run <routine-id>
paperclipai routine run <routine-id> --payload-json '{"reason": "manual smoke test"}'
```

| 标志 | 用途 |
|---|---|
| `--payload-json <json>` | JSON 字符串形式的运行输入。默认为 `{}`。 |

> **注意：** `routine run` 通过 API 请求启动运行。实际是否执行以及何时执行，仍受例行任务的并发和补运行设置（见下文）以及公司的常规心跳限制控制。

---

## 触发器

触发器用于启动例行任务，一个例行任务可以配置多个触发器。共有三种类型，应根据驱动任务的方式选择：

| 触发器类型 | 触发时机 | 说明 |
|---|---|---|
| `schedule` | 到达 cron/重复计划设定的时间。 | 常见的“每晚/每小时/每周运行”触发器。计划重叠或错过时段后的行为由并发与补运行策略控制。 |
| `webhook` | 外部系统向触发器的 webhook URL 发送 POST 请求。 | 带有可轮换的秘密值，可用于将外部事件转换为 Paperclip 工作。 |
| `api`（公开） | 调用方通过 `publicId` 触发公开端点。 | 使用 `routine trigger:fire` 触发。公开 ID 可分享，而无需暴露例行任务 ID。 |

触发器的精确结构（类型、计划表达式、webhook 配置、并发策略和补运行策略）由 `trigger:create` 和 `trigger:update` 上的 JSON 请求体设置。CLI 会将请求体直接传给 API，因此触发器 schema 是可用字段的权威依据。

### 并发与补运行

对于计划触发的例行任务，有两项设置最重要，都在触发器请求体中设置：

- **并发**决定例行任务尚未完成时，下一个计划时间到达后如何处理。可以允许运行重叠，也可以将其串行化，避免慢运行叠加在自身之上。
- **补运行**决定错过的计划时段如何处理，例如实例停机期间错过的时段。可以在调度器恢复后触发所有遗漏执行，也可以跳过遗漏项，直接等下一个到期时段。

请审慎选择。例如汇总“上次运行以来”数据的摘要任务通常需要串行执行且不补运行，以避免生成多个重叠积压任务；幂等的数据抓取任务则可以接受补运行。部署或停机恢复后，可运行 `routine runs <routine-id>` 确认触发情况。

### `routine trigger:create`

为例行任务添加新的触发器。参数是例行任务 ID，请求体是触发器定义。

```sh
paperclipai routine trigger:create <routine-id> --payload-json '{
  "kind": "schedule",
  "schedule": "0 7 * * *"
}'
```

| 标志 | 用途 |
|---|---|
| `--payload-json <json>` | JSON 字符串形式的触发器定义。默认为 `{}`。 |

### `routine trigger:update`

部分更新已有触发器。此命令的参数是*触发器* ID，而非例行任务 ID。可用它修改计划、调整并发/补运行策略，或启用/禁用触发器。

```sh
paperclipai routine trigger:update <trigger-id> --payload-json '{"schedule": "0 */2 * * *"}'
```

| 标志 | 用途 |
|---|---|
| `--payload-json <json>` | JSON 字符串形式的待更新字段（必填）。 |

### `routine trigger:delete`

从例行任务中移除触发器。

```sh
paperclipai routine trigger:delete <trigger-id>
```

### `routine trigger:rotate-secret`

轮换 webhook 触发器的秘密值。秘密泄漏或按计划定期轮换时应执行。轮换后旧值会失效，因此请在同一次变更中更新向 webhook 发送请求的调用方。

```sh
paperclipai routine trigger:rotate-secret <trigger-id>
```

### `routine trigger:fire`

按 `publicId` 触发公开 `api` 触发器。这是提供给外部调用方的端点：它使用公开 ID，而非例行任务 ID，因此可分享而不会泄露内部 ID。

```sh
paperclipai routine trigger:fire <public-id>
paperclipai routine trigger:fire <public-id> --payload-json '{"event": "deploy.finished"}'
```

| 标志 | 用途 |
|---|---|
| `--payload-json <json>` | 与此次触发一同传递的请求体。默认为 `{}`。 |

---

## `routines disable-all`

这是本地维护命令，也就是“全部停止”开关。它会直接修改已配置的本地 Paperclip 数据库，暂停某个公司的所有未归档例行任务，不经过 API。本地实例运行异常、需要立即停止所有计划任务和 webhook 驱动的工作时使用。

```sh
paperclipai routines disable-all --company-id <company-id>
paperclipai routines disable-all --company-id <company-id> --json
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 要暂停例行任务的公司（必填；可回退到 `PAPERCLIP_COMPANY_ID`）。 |
| `-c, --config <path>` | Paperclip 配置文件路径；不使用默认配置时指定。 |
| `-d, --data-dir <path>` | Paperclip 数据目录根路径，用于将状态与 `~/.paperclip` 隔离。 |
| `--json` | 输出原始结果对象，而非人类可读摘要。 |

具体行为如下：

- 从本地数据库读取该公司的所有例行任务。
- 将状态**不是** `paused` 或 `archived` 的例行任务设为 `paused`。
- 不修改已经暂停或已归档的例行任务。

摘要会报告四个计数：例行任务总数、新暂停的数量、原本已暂停的数量和已归档的数量：

```text
Paused 3 routine(s) for company <company-id> (1 already paused, 0 archived).
```

使用 `--json` 时会返回相同计数，字段为 `companyId`、`totalRoutines`、`pausedCount`、`alreadyPausedCount` 和 `archivedCount`。

> **警告：** `disable-all` 操作的是**本地**实例数据库（如果配置使用嵌入式模式，它会启动嵌入式 PostgreSQL 集群）。它不能用来暂停远程控制平面中的例行任务；远程操作请逐项运行 `routine update <routine-id> --payload-json '{"status":"paused"}'`。目前没有批量重新启用命令；问题解决后请逐个使用 `routine update` 恢复例行任务。

---

## 另请参阅

- [运行命令](./run.md)——检查和控制心跳运行，包括例行任务生成的运行。
- [Issue 命令](./issue.md)——例行任务创建的任务；可按 `originKind=routine_execution` 筛选。
- [通用选项](./common-options.md)——所有 `routine` 命令接受的客户端标志。
- [输出与脚本](./output-and-scripting.md)——使用 `--json` 将例行任务数据传给其他工具。
- [例行任务与自动化](../../guides/projects-workflow/routines.md)——自动化设计端到端指南。
