---
paperclip_version: v2026.609.0
seo_title: CLI 目标命令
seo_description: 设置、检查和重组任务与项目最终服务的战略目标，即各项工作的缘由。
---

# 目标命令

目标是公司的战略目的，是任务和项目最终服务的“缘由”。如果你希望从终端设置、检查、重组或结束公司的目标（例如建立目标树、将项目关联到其推进的目标，或将目标标记为已实现），可使用这些命令。目标作用范围限定在公司内，可以嵌套形成层级，也可以由智能体负责。

---

## 目标在模型中的位置

目标位于日常工作之上。项目和任务隶属于目标，而不是反过来：

- 一个**项目**可以关联一个或多个目标——`project create` 接受 `--goal-ids`。项目是推动目标前进的载体。详见[项目命令](./project.md)。
- **任务**可以带有 `goalId`；如果没有，服务器会按以下优先顺序解析目标：任务自身的目标、项目目标、公司的默认目标。详见[任务命令](./issue.md)。
- 目标可以有**父目标**（`--parent-id`），从而建立目标树，例如公司层级目标下设团队或季度子目标。
- 目标可以有**负责人智能体**（`--owner-agent-id`），由该智能体负责实现目标。

目标是整个层级结构的锚点，因此应保持长期有效。通常很少创建新目标；随着目标进展更新状态；几乎不删除目标。

所有目标命令都接受通用客户端选项（`--data-dir`、`--api-base`、`--api-key`、`--context`、`--profile`、`--json`）。集合命令（`list`、`create`）按公司限定范围，接受 `-C, --company-id <id>`；单目标命令（`get`、`update`、`delete`）则直接通过目标 ID 操作。共享标志和 API 地址解析顺序见[通用选项](./common-options.md)。

---

## 列出目标

列出公司的所有目标。

```sh
paperclipai goal list --company-id <company-id>
paperclipai goal list --company-id <company-id> --json
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 要列出其目标的公司。必填（如果已在上下文中选定公司，则可从上下文解析）。 |

默认的人类可读输出会为每个目标打印一行，显示 `id`、`status`、`title`、`level`、`parentId` 和 `ownerAgentId`，足以快速查看目标树。编写脚本并需要完整目标记录时，请使用 `--json`。

> **提示：** 将 `goal list --json` 的输出通过管道传给 `jq`，即可查找某目标的子目标（`jq '[.[] | select(.parentId == "<goal-id>")]'`）或提取公司层级目标（`jq '[.[] | select(.level == "company")]'`）。

---

## 获取目标

根据 ID 获取单个目标。

```sh
paperclipai goal get <goal-id>
paperclipai goal get <goal-id> --json
```

此命令直接操作目标，因此无需传入 `--company-id`。将项目或任务关联到某个目标之前，可用它确认目标当前的 `status`、负责人或父目标。

---

## 创建目标

在公司中创建目标。

```sh
paperclipai goal create --company-id <company-id> --title "Grow revenue 30% this year"
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 创建目标所属的公司。**必填。** |
| `--title <title>` | 目标标题。**必填。** |
| `--description <text>` | 对目标的详细说明。 |
| `--level <level>` | 目标在层级结构中的级别，例如公司级或团队级。 |
| `--status <status>` | 初始状态。 |
| `--parent-id <id>` | 父目标，用于将此目标嵌套在另一个目标之下。 |
| `--owner-agent-id <id>` | 对此目标负责的智能体。 |

只有 `--title` 和 `--company-id` 是必填参数；其他参数均为可选。省略 `level` 或 `status` 时，服务器会应用自己的默认值。命令会打印新建的目标（传入 `--json` 可获取原始记录），请保存返回的 `id`，之后在 `project create --goal-ids` 或任务的 `goalId` 中使用。

嵌套示例——创建公司目标，并由智能体负责其中一个团队子目标：

```sh
# Top-level company goal
paperclipai goal create --company-id <company-id> \
  --title "Reach profitability" --level company

# Sub-goal that rolls up into it, owned by the growth lead
paperclipai goal create --company-id <company-id> \
  --title "Launch paid tier" \
  --parent-id <parent-goal-id> \
  --owner-agent-id <agent-id>
```

> **注意：** 只有在应由某个智能体对目标负责时，才传入 `--owner-agent-id`。所有权表示责任归属，并不表示工作指派——智能体通过任务而非目标接收实际工作。

---

<a id="update-a-goal"></a>
## 更新目标

更新目标字段。所有字段均为可选，只传入要修改的字段即可。

```sh
paperclipai goal update <goal-id> --status achieved
paperclipai goal update <goal-id> --title "Grow revenue 40% this year" --description "Stretch target after Q2 beat"
```

| 标志 | 用途 |
|---|---|
| `--title <title>` | 新标题。 |
| `--description <text\|null>` | 新说明；传入 `null` 可清除说明。 |
| `--level <level>` | 新级别。 |
| `--status <status>` | 新状态；随着目标进展，可据此将目标标记为活动、已实现或已放弃。 |
| `--parent-id <id\|null>` | 更换父目标；传入 `null` 可将目标提升为顶层目标。 |
| `--owner-agent-id <id\|null>` | 更换负责人智能体；传入 `null` 可清除负责人。 |

对于可空字段（`--description`、`--parent-id` 和 `--owner-agent-id`），字面值 `null` 会清除字段；任何其他值都会设置字段。若不删除目标，这是从父目标中移出目标或清除其负责人的唯一方式。

```sh
# Promote a sub-goal to the top level and hand it off
paperclipai goal update <goal-id> --parent-id null --owner-agent-id <new-owner-id>

# Clear an owner entirely
paperclipai goal update <goal-id> --owner-agent-id null
```

此命令不按公司查询目标，因此无需传入 `--company-id`。

---

## 删除目标

删除目标属于受保护操作——必须传入 `--yes`，否则命令会拒绝执行。

```sh
paperclipai goal delete <goal-id> --yes
```

| 标志 | 用途 |
|---|---|
| `--yes` | 确认删除。不传时会报错 “Deletion requires --yes.”。 |

删除目标会改变结构：关联到此目标的项目和引用其 `goalId` 的任务会失去该链接；任何子目标也会失去其父目标。应优先将目标状态更新为 “achieved” 或 “abandoned”，而不是删除，这样可以保留战略历史记录。只有目标系误建时才删除。

> **警告：** `goal delete` 要求明确传入 `--yes`，因为删除目标可能使指向它的项目和任务失去关联。运行前请确认该目标下没有仍在进行的工作。

---

## 另请参阅

- [项目命令](./project.md)——通过 `--goal-ids` 创建关联到目标的项目。
- [任务命令](./issue.md)——为任务设置 `goalId`，以及目标优先级的解析方式。
- [公司命令](./company.md)——选择并检查目标所属的公司。
- [通用选项](./common-options.md)——共享标志和 API 地址解析规则。
- [输出和脚本](./output-and-scripting.md)——使用 `--json` 编写目标数据处理脚本。
