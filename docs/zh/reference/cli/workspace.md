---
paperclip_version: v2026.609.0
seo_title: CLI 工作区、组织和环境命令
seo_description: 在终端中检查公司的结构和执行环境：组织架构、智能体配置以及运行实际发生的位置。
---

# 工作区、组织和环境命令

需要从终端检查公司的*结构*和*执行环境*时，可使用这些命令，包括组织架构、各角色对应的智能体配置、心跳运行实际工作的执行工作区、为其提供支持的环境和租约，以及各项目的工作区。调试运行、配置环境或确认公司使用了哪些基础设施时，可以使用这些底层检查和控制命令；日常任务流程则使用其他命令。

这些命令都是 Paperclip API 的读取和控制封装。它们会**触发**并**观察**服务器端状态；实际执行（运行时服务、运行时命令、环境探测）由 Paperclip 运行时完成，而不是由 CLI 完成。

> **注意：** 此处的所有命令都接受通用客户端标志：`--data-dir`、`--api-base`、`--api-key`、`--context`、`--profile` 和 `--json`。详见[通用选项](./common-options.md)。按公司限定范围的子命令也接受 `-C, --company-id <id>`；按 ID 或项目限定的子命令则将 ID 作为位置参数。

---

## 组织架构（`org`）

使用 `org` 获取公司的组织架构，可返回结构化数据，也可生成可嵌入文档或状态更新的图表。

```sh
paperclipai org get --company-id <company-id>
paperclipai org svg --company-id <company-id> --out org.svg
paperclipai org png --company-id <company-id> --out org.png
```

| 命令 | 返回内容 |
|---|---|
| `org get` | 结构化 JSON 数据格式的组织架构。 |
| `org svg` | 以 SVG 图像形式呈现的组织架构。 |
| `org png` | 以 PNG 图像形式呈现的组织架构。 |

`get` 按公司限定范围，和其他读取命令一样支持 `--json`。`svg` 和 `png` 会获取二进制图像数据：

- 传入 `--out <path>` 时，字节会写入指定文件，命令则输出简短的 `{ out, bytes }` 摘要。
- 不传 `--out` 时，原始图像字节流会写入标准输出——请自行将其通过管道写入文件（`paperclipai org png -C <company-id> > org.png`）或传给其他工具。

> **提示：** 需要分享组织汇报结构快照时，使用 `org svg` / `org png`。编写脚本并需要底层数据时，使用 `org get`。

---

## 智能体配置（`agent-config`）

`agent-config list` 会返回公司的智能体配置摘要，集中显示每个智能体解析后的适配器/模型设置。在调整环境或聘用智能体前，可用此命令审计当前使用的配置。

```sh
paperclipai agent-config list --company-id <company-id>
```

此命令按公司限定范围且只读。需要比较不同公司的配置，或将配置传给其他工具时，可以搭配 `--json` 使用。

---

## 执行工作区（`workspace`）

执行工作区是服务器端的沙盒，心跳运行会在其中完成工作。使用 `workspace` 命令可列出工作区、检查某个工作区、确认是否可以安全关闭、查看其操作历史、更新设置，或控制其运行时。

```sh
paperclipai workspace list --company-id <company-id>
paperclipai workspace get <execution-workspace-id>
paperclipai workspace close-readiness <execution-workspace-id>
paperclipai workspace operations <execution-workspace-id>
paperclipai workspace update <execution-workspace-id> --payload-json '{...}'
paperclipai workspace runtime-service <execution-workspace-id> start --payload-json '{...}'
paperclipai workspace runtime-command <execution-workspace-id> run --payload-json '{...}'
```

| 命令 | 参数 | 用途 |
|---|---|---|
| `list` | `--company-id <id>` | 列出公司的所有执行工作区。 |
| `get` | `<id>` | 获取单个执行工作区。 |
| `close-readiness` | `<id>` | 检查工作区是否可以安全关闭（不会丢失正在执行的工作）。 |
| `operations` | `<id>` | 列出工作区的操作历史。 |
| `update` | `<id> --payload-json <json>` | 使用 JSON 请求体修改工作区。 |
| `runtime-service` | `<id> <action> [--payload-json <json>]` | 控制工作区中的运行时服务。 |
| `runtime-command` | `<id> <action> [--payload-json <json>]` | 在工作区中运行运行时命令。 |

对于 `runtime-service` 和 `runtime-command`，位置参数 `<action>` 的取值为 `start`、`stop`、`restart` 或 `run`。`--payload-json` 标志用于通过 JSON 请求体传递运行时目标；省略时默认为 `{}`。

```sh
paperclipai workspace runtime-service <execution-workspace-id> restart --payload-json '{"service":"dev-server"}'
paperclipai workspace runtime-command <execution-workspace-id> run --payload-json '{"command":"npm test"}'
```

> **提示：** 拆除或关闭工作区之前，运行 `workspace close-readiness <id>`。它会告诉你现在关闭是否会中断活动工作。

> **警告：** `runtime-service stop` / `restart` 会操作正在运行的运行时进程，可能有运行正在依赖它们。请先用 `workspace get` 和 `workspace operations` 确认没有关键工作正在执行。

---

## 环境（`environment`）

环境描述工作区*在哪里以及如何*运行，包括运行时服务和命令所使用的底层基础设施（SSH 主机、容器、能力）。工作区通过租约声明使用某个环境。可以使用这些命令盘点环境、探测健康状态、创建或更新环境，以及查看其租约。

```sh
paperclipai environment list --company-id <company-id>
paperclipai environment capabilities --company-id <company-id>
paperclipai environment create --company-id <company-id> --payload-json '{...}'
paperclipai environment get <environment-id>
paperclipai environment leases <environment-id>
paperclipai environment lease <lease-id>
paperclipai environment update <environment-id> --payload-json '{...}'
paperclipai environment delete <environment-id>
paperclipai environment probe <environment-id>
paperclipai environment probe-config --company-id <company-id> --payload-json '{...}'
```

| 命令 | 参数 | 用途 |
|---|---|---|
| `list` | `--company-id <id>` | 列出公司的环境。 |
| `capabilities` | `--company-id <id>` | 报告公司可使用的环境能力。 |
| `create` | `--company-id <id> --payload-json <json>` | 根据 JSON 请求体创建环境。 |
| `get` | `<environment-id>` | 获取单个环境。 |
| `leases` | `<environment-id>` | 列出针对某环境持有的所有租约。 |
| `lease` | `<lease-id>` | 根据租约自身的 ID 获取单项环境租约。 |
| `update` | `<environment-id> --payload-json <json>` | 使用 JSON 请求体修改环境。 |
| `delete` | `<environment-id>` | 删除环境。 |
| `probe` | `<environment-id>` | 探测现有环境的可达性/健康状态。 |
| `probe-config` | `--company-id <id> --payload-json <json>` | 在创建环境**之前**探测候选配置。 |

请注意这两个不同的租约命令：`environment leases <environment-id>` 会列出某个环境上的所有租约；`environment lease <lease-id>` 则通过租约自己的 ID 查找**单项**租约（请求路径为 `/api/environment-leases/{leaseId}`）。如果你正在检查某个环境并想知道谁在使用它，请使用 `leases`；如果已从其他地方获得租约 ID，则使用 `lease`。

```sh
# Validate a config before committing it, then create the environment.
paperclipai environment probe-config --company-id <company-id> --payload-json '{"kind":"ssh","host":"build-01.internal"}'
paperclipai environment create --company-id <company-id> --payload-json '{"name":"build-01","kind":"ssh","host":"build-01.internal"}'
paperclipai environment probe <environment-id>
```

> **提示：** `probe-config` 是试运行路径，可检查候选配置而不保存任何内容。`probe` 用于验证已存在的环境。先探测，再创建。

---

## 项目工作区（`project-workspace`）

项目工作区限定在单个项目中。它们通过项目 ID（对于单工作区操作，还需要工作区 ID）定位，而不是通过公司 ID。使用这些命令可以列出、创建、更新和删除单个项目关联的工作区，并控制这些工作区。

```sh
paperclipai project-workspace list <project-id>
paperclipai project-workspace create <project-id> --payload-json '{...}'
paperclipai project-workspace update <project-id> <workspace-id> --payload-json '{...}'
paperclipai project-workspace delete <project-id> <workspace-id>
paperclipai project-workspace runtime-service <project-id> <workspace-id> restart --payload-json '{...}'
paperclipai project-workspace runtime-command <project-id> <workspace-id> run --payload-json '{...}'
```

| 命令 | 参数 | 用途 |
|---|---|---|
| `list` | `<project-id>` | 列出项目的工作区。 |
| `create` | `<project-id> --payload-json <json>` | 创建项目工作区。 |
| `update` | `<project-id> <workspace-id> --payload-json <json>` | 更新项目工作区。必须提供工作区 ID。 |
| `delete` | `<project-id> <workspace-id>` | 删除项目工作区。 |
| `runtime-service` | `<project-id> <workspace-id> <action> [--payload-json <json>]` | 控制项目工作区中的运行时服务。 |
| `runtime-command` | `<project-id> <workspace-id> <action> [--payload-json <json>]` | 在项目工作区中运行运行时命令。 |

与执行工作区一样，`runtime-service` 和 `runtime-command` 的 `<action>` 取值为 `start`、`stop`、`restart` 或 `run`，且 `--payload-json` 默认值为 `{}`。

```sh
paperclipai project-workspace list <project-id>
paperclipai project-workspace delete <project-id> <workspace-id>
paperclipai project-workspace runtime-command <project-id> <workspace-id> run --payload-json '{"command":"pnpm build"}'
```

> **注意：** `update` 必须提供工作区 ID 位置参数；省略时命令会拒绝调用。`create` 只接受项目 ID，因为此时工作区尚不存在。

---

## 另请参阅

- [通用选项](./common-options.md)——上述命令都接受的客户端标志。
- [输出和脚本](./output-and-scripting.md)——使用 `--json` 和管道处理结果。
- [项目命令](./project.md)——管理拥有项目工作区的项目。
- [开发环境](./dev-environments.md)——环境和 SSH 测试装置的整体配置流程。
- [运行命令](./run.md)——检查在这些工作区中执行的心跳运行。
- [适配器命令](./adapter.md)——`agent-config list` 所汇总的适配器/模型配置。
