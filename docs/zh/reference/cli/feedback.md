---
paperclip_version: v2026.609.0
seo_title: CLI 反馈命令
seo_description: 检查、汇总和导出针对公司智能体输出及工作记录的赞成与反对反馈。
---

# 反馈命令

如果你想检查、汇总或提取针对公司工作（例如智能体输出、评论及审核者投票的其他对象）记录的赞成/反对反馈，可使用这些命令。从“大家实际如何评价智能体的工作？”快速生成可分享的材料，最直接的方式是运行 `paperclipai feedback report` 查看概要，再运行 `paperclipai feedback export` 将完整资料包写入磁盘。`trace` 和 `bundle` 子命令则按 ID 获取单条反馈记录，适用于脚本流程。

Paperclip 将反馈记录为**反馈轨迹**。每条轨迹记录针对一个目标的一次投票（包括 `vote`、`status`、所属任务、投票对象摘要和可选原因），并附带 `payloadSnapshot` 以及可下载的底层文件**资料包**。这些命令通过 API 读取轨迹；CLI 不会自行计算投票，只会查看服务器已经保存的记录。

---

## 命令摘要

| 命令 | 用途 |
|---|---|
| `feedback report` | 在终端显示公司的反馈轨迹报告，包括汇总和逐条详情。 |
| `feedback export` | 将投票记录和原始轨迹资料包写入文件夹及 `.zip` 归档。 |
| `feedback trace <traceId>` | 按 ID 获取一条反馈轨迹。 |
| `feedback bundle <traceId>` | 获取单条反馈轨迹的资料包（轨迹及其附带文件）。 |

以上四个子命令都接受[通用选项](./common-options.md)中介绍的客户端标志：`--data-dir`、`--api-base`、`--api-key`、`--context`、`--profile` 和 `--json`。`report` 和 `export` 按公司限定范围，另外还接受 `-C, --company-id <id>`。

> **注意：** `feedback report` 和 `feedback export` 按以下顺序解析公司：`-C/--company-id`、`PAPERCLIP_COMPANY_ID`、CLI 上下文默认公司。如果均未设置，命令会回退到 API 返回的**第一家**公司。实例中有多家公司时，请始终传入 `--company-id`，确保导出的是目标公司。

---

## 筛选反馈轨迹

`report` 和 `export` 共享相同的筛选标志。它们会先筛选从 `/api/companies/<company-id>/feedback-traces` 读取的轨迹，然后再呈现报告或写入文件。

| 标志 | 用途 |
|---|---|
| `--target-type <type>` | 只包含针对指定目标类型的轨迹，例如评论或工作成果。 |
| `--vote <vote>` | 只包含指定投票值（`up` 或 `down`）的轨迹。 |
| `--status <status>` | 只包含处于指定导出状态的轨迹（见下表）。 |
| `--project-id <id>` | 只包含关联到指定项目的轨迹。 |
| `--issue-id <id>` | 只包含关联到指定任务的轨迹。 |
| `--from <iso8601>` | 只包含在此时间戳或之后创建的轨迹。 |
| `--to <iso8601>` | 只包含在此时间戳或之前创建的轨迹。 |
| `--shared-only` | 只包含符合分享/导出条件的轨迹。 |

准备将反馈发送到实例外部时，请同时使用时间范围和 `--shared-only`，这样只会导出服务器标记为可分享且日期符合要求的轨迹。

### 轨迹状态值

每条轨迹都带有导出 `status`。报告会汇总以下四种状态，以显示每条反馈在分享流程中的位置：

| 状态 | 含义 |
|---|---|
| `pending` | 已记录，等待导出/分享。 |
| `sent` | 已分享/导出到实例外。 |
| `local_only` | 保留在当前实例中，不允许导出。 |
| `failed` | 尝试导出/分享失败。 |

---

## 报告

`feedback report` 会读取匹配的轨迹并打印终端报告，包括服务器 URL 和公司名称标题、汇总区块，以及每条轨迹的详情区块。

```sh
paperclipai feedback report --company-id <company-id>
```

汇总会统计赞成票、反对票、附有文字原因的反对票和轨迹总数，并按导出状态细分。详情区块会为每条轨迹显示所属任务的引用和标题、轨迹短 ID、状态、创建日期、目标标签、投票对象摘录，以及（如果有）投票原因。

可以使用任意筛选标志缩小报告范围。例如，只查看某项目最近一周的反对票：

```sh
paperclipai feedback report \
  --company-id <company-id> \
  --vote down \
  --project-id <project-id> \
  --from 2026-05-25T00:00:00Z
```

如需在报告中内联显示每条轨迹的完整原始负载，请添加 `--payloads`：

```sh
paperclipai feedback report --company-id <company-id> --payloads
```

> **提示：** `--payloads` 会在每条轨迹下输出完整的 `payloadSnapshot` JSON，内容会很长。只有在需要精确排查记录了什么时才使用，不适用于日常查看。

需要程序处理时，传入 `--json`。命令会输出包含 `apiBase`、`companyId`、`summary` 和完整 `traces` 数组的结构化对象，而不是格式化的终端报告：

```sh
paperclipai feedback report --company-id <company-id> --json
```

---

## 导出

`feedback export` 会读取匹配的轨迹，获取每条轨迹的资料包，并将全部内容写入文件夹及同级的 `.zip` 归档。需要将反馈从终端导出用于审核、归档或交接时，请使用此命令。

```sh
paperclipai feedback export --company-id <company-id> --out ./feedback/acme
```

省略 `--out` 时，命令会在当前目录写入 `./feedback-export-<timestamp>`。输出目录必须不存在文件；命令拒绝写入非空目录，以免覆盖先前的导出内容。

### 导出内容

导出文件夹结构便于人工阅读，也可传给其他工具处理：

| 路径 | 内容 |
|---|---|
| `index.json` | 清单：导出时间、服务器 URL、公司 ID、摘要（计数、`uniqueIssues`、排序后的任务标识符列表）以及写入文件列表。 |
| `votes/` | 每条轨迹一个 JSON 文件，包含扁平化投票记录（投票、目标、状态、同意版本、时间戳和原因）。 |
| `traces/` | 每条轨迹一个 JSON 文件，包含完整轨迹对象。 |
| `full-traces/<issue>-<trace>/` | 每条轨迹一个目录，包含 `bundle.json` 和该轨迹资料包中的所有文件，按原始相对路径写入。 |
| `<output-dir>.zip` | 保存（不压缩）`index.json` 和以上所有文件的 ZIP 归档；归档根目录使用导出文件夹名称。 |

文件名由任务标识符（例如 `PAP-39`）和轨迹或投票 ID 的短片段组成，因此无需打开文件也能浏览导出内容。

### 按公司和任务导出

导出始终**限定在公司范围内**：先获取一家公司的轨迹，再按轨迹所属的**任务**对文件分组。清单中的 `summary.uniqueIssues` 和 `summary.issues` 会准确列出导出涉及的任务；`votes/`、`traces/` 和 `full-traces/` 中的文件名也包含任务标识符。因此，一次导出既能回答“这家公司的工作总体评分如何？”，也能回答“哪些任务收到了反馈？”，无需再进行第二轮处理。

仅导出某一任务中可分享的反馈：

```sh
paperclipai feedback export \
  --company-id <company-id> \
  --issue-id <issue-id> \
  --shared-only \
  --out ./feedback/<issue-id>
```

传入 `--json` 时，命令会输出 `companyId`、解析后的 `outputDir`、`zipPath` 和导出 `summary`，而不是格式化摘要：

```sh
paperclipai feedback export --company-id <company-id> --json
```

---

## 轨迹

`feedback trace` 根据 ID 获取单条反馈轨迹。如果手头已有报告、导出结果或其他工具提供的轨迹 ID，并希望查看原始记录，可使用此命令。

```sh
paperclipai feedback trace <trace-id>
paperclipai feedback trace <trace-id> --json
```

此命令直接读取 `/api/feedback-traces/<trace-id>`，不限定公司；轨迹 ID 本身就能唯一标识该记录。

---

## 资料包

`feedback bundle` 会获取单条轨迹的**资料包**，包括轨迹和随其捕获的底层文件。其内容与 `feedback export` 写入各个 `full-traces/` 目录的内容相同，只是一次只获取一条轨迹。

```sh
paperclipai feedback bundle <trace-id>
paperclipai feedback bundle <trace-id> --json
```

如果需要查看单张投票背后的完整上下文（即实际接受投票的文件），但不需要导出整个公司的数据，请使用 `bundle`。

---

## 脚本注意事项

- 将输出传给 `jq` 或其他工具时，所有子命令都应使用 `--json`。约定见[输出和脚本](./output-and-scripting.md)。
- `report --json` 返回与 `export --json` 相同结构的 `summary`，因此可以先用 `report` 试运行筛选条件，再用 `export` 将结果写入磁盘。
- 因为 `export` 会拒绝写入非空的 `--out` 目录，脚本每次运行都应生成新路径，或使用默认的时间戳目录。

---

## 另请参阅

- [任务命令](./issue.md)——反馈轨迹关联到任务；使用这些命令检查被投票的工作。
- [公司命令](./company.md)——反馈限定在公司范围内；导出和可移植性功能见此处。
- [通用选项](./common-options.md)——所有反馈子命令接受的共享客户端标志。
- [输出和脚本](./output-and-scripting.md)——自动化所用 `--json` 输出的结构。
