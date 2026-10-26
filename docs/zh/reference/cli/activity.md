---
paperclip_version: v2026.609.0
seo_title: CLI 活动命令
seo_description: 在终端中读取公司的审计记录：按时间顺序记录谁对哪个实体执行了什么操作，可用于监控和调试。
---

# 活动命令

活动日志是公司的审计记录，按时间顺序记录谁对哪个实体执行了什么操作。监控运行中的公司、还原智能体执行过的操作，或将外部系统接入与 Paperclip 界面相同的记录时，可以使用这些命令。它与界面中的活动动态完全一致，只是可从终端访问，便于编写脚本、导出或无界面监控。

`activity list` 和 `activity issue` 只读。此处唯一会修改数据的命令是 `activity create`，通常很少需要使用它——服务器会在正常工作（例如更新任务、锁定文档和处理恢复操作）时自动写入活动记录。只有当某个外部事件确实应进入公司活动记录时，才使用 `activity create`。

三个子命令都接受[通用客户端选项](./common-options.md)（`--api-base`、`--api-key`、`--context`、`--profile`、`--data-dir`、`--json`）。`activity list` 和 `activity create` 按公司限定范围，必须提供 `--company-id`；`activity issue` 则根据任务本身解析所属公司。

> **提示：** 如果要编写命令脚本或将输出传给其他工具，请添加 `--json`。不加此选项时，`list` 会逐行输出紧凑的内联记录，方便人工快速浏览。

---

## activity list

列出公司的活动日志条目。

```sh
paperclipai activity list --company-id <company-id>
paperclipai activity list --company-id <company-id> --agent-id <agent-id>
paperclipai activity list --company-id <company-id> --entity-type issue --entity-id <issue-id>
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | **必填。** 要查询其活动日志的公司。 |
| `--agent-id <id>` | 仅显示归属于指定智能体的条目。 |
| `--entity-type <type>` | 按实体类型筛选，例如 `issue`。 |
| `--entity-id <id>` | 按指定实体 ID 筛选。 |

筛选条件可以组合使用：传入 `--entity-type issue --entity-id <issue-id>` 可将日志限定为单个任务；再加上 `--agent-id`，即可只查看某个智能体对该任务执行的操作。

默认（非 JSON）输出中的每项都是一条内联记录，包含最常用于筛选或比对的字段：

| 字段 | 含义 |
|---|---|
| `id` | 活动条目自身的 ID。 |
| `action` | 发生的操作，例如 `issue.updated` 或 `issue.document_locked`。 |
| `actorType` | 操作者类型，例如智能体、用户或系统。 |
| `actorId` | 操作者的 ID。 |
| `entityType` | 操作涉及的对象类型。 |
| `entityId` | 操作涉及的对象 ID。 |
| `createdAt` | 记录写入时间。 |

没有匹配条目时，命令会输出空结果，而不会报错。使用 `--json` 可获取完整条目对象，包括服务器保存的所有负载内容。

---

## activity create

在公司活动日志中创建条目。请求正文以原始 JSON 字符串提供，服务器会按照 `CreateActivity` 负载进行验证。

```sh
paperclipai activity create --company-id <company-id> \
  --payload-json '{"action":"deploy.completed","entityType":"project","entityId":"<project-id>"}'
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | **必填。** 要将条目记录到的公司。 |
| `--payload-json <json>` | **必填。** 描述活动条目的 JSON 字符串。 |

CLI 会在发送前使用 `JSON.parse` 解析 `--payload-json`，因此 JSON 语法必须有效。请在 shell 中使用单引号包住整个字符串，确保其中的双引号原样保留。哪些字段可用于 `CreateActivity` 负载，由服务器决定；格式无效时，服务器会返回请求错误，CLI 会直接显示该错误。

> **注意：** 正常操作几乎不需要此命令。任务、文档和恢复操作发生变化时，Paperclip 会自行写入活动记录。仅在需要将外部事件（例如部署、事故或手动里程碑）补入同一审计记录时，才使用 `activity create`。

---

## activity issue

列出单个任务的活动记录；任务可通过 ID 或人类可读标识符（例如 `PAP-39`）指定。

```sh
paperclipai activity issue <issue-id>
paperclipai activity issue PAP-39 --json
```

此命令将任务作为位置参数，并从所选配置文件和任务本身解析其余上下文（包括所属公司），因此**不需要**提供 `--company-id`。无需筛选整个公司的日志，这是快速回答“这个任务发生过什么？”的简便方式。

`activity issue <id>` 和 `activity list --company-id <id> --entity-type issue --entity-id <id>` 的功能有所重叠；如果手头已有任务标识符，优先使用 `activity issue`，因为它更简洁，并且可以直接接受 `PAP-39` 这样的可读标识符。

---

## 另请参阅

- [任务命令](issue.md)——创建、更新、评论任务，以及检出相关工作；这些任务的历史会显示在此处。
- [控制面板命令](dashboard.md)——查看与活动动态并列的界面公司概览状态。
- [通用选项](./common-options.md)——所有客户端命令共享的标志。
- [输出和脚本](./output-and-scripting.md)——了解 `--json` 输出的结构，以及如何将其传给其他工具。
