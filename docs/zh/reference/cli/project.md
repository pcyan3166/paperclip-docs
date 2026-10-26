---
paperclip_version: v2026.609.0
seo_title: CLI 项目命令
seo_description: 管理用于归集工作的项目，包括状态、负责人代理、关联目标以及其下 issue 继承的执行默认值。
---

# 项目命令

项目用于归集公司内相关工作，包含状态、可选的负责人代理、一个或多个关联目标，以及其下 issue 会继承的执行默认值。需要在终端而非 UI 中查看或调整公司项目时，可以使用这些命令，包括设置 JSON 格式的 `env` 绑定和执行工作区策略；后者决定代理的工作如何在服务器端运行。

所有 `project` 子命令都接受通用客户端标志（`--data-dir`、`--api-base`、`--api-key`、`--context`、`--profile`、`--json`）。公司范围由每条命令单独通过 `-C, --company-id <id>` 指定，而不是全局继承；需要时请显式传入。

---

## 快速一览

```sh
paperclipai project list --company-id <company-id>
paperclipai project get <project-id-or-shortname> [--company-id <company-id>]
paperclipai project create --company-id <company-id> --name "Launch Site" [--goal-ids <id1,id2>] [--lead-agent-id <agent-id>]
paperclipai project update <project-id-or-shortname> --status in_progress [--company-id <company-id>]
paperclipai project delete <project-id-or-shortname> --yes [--company-id <company-id>]
```

`get`、`update` 和 `delete` 接受项目 UUID 或短名称作为项目引用。使用短名称时，请提供 `--company-id`，以便服务器在正确的公司范围内解析。

> **提示：** 编写脚本时，可以在任意子命令后添加 `--json`。`list` 的人类可读输出会为每个项目打印一行摘要；JSON 输出则包含完整项目记录。

---

## 列出项目

```sh
paperclipai project list --company-id <company-id>
paperclipai project list --company-id <company-id> --json
```

`list` 限定在公司范围内，必须提供 `--company-id`。默认输出会为每个项目打印一行，显示最常用于排查的信息：

| 字段 | 含义 |
|---|---|
| `id` | 项目 UUID。 |
| `name` | 项目显示名称。 |
| `status` | 项目当前状态。 |
| `urlKey` | 项目稳定的短名称/URL 键。 |
| `goalIds` | 逗号分隔的关联目标 ID。 |
| `leadAgentId` | 项目负责人代理（如果已设置）。 |

没有项目时，`list` 会输出空结果。脚本需要完整记录时，请使用 `--json`。

---

## 获取项目

```sh
paperclipai project get <project-id-or-shortname>
paperclipai project get launch-site --company-id <company-id>
paperclipai project get <project-id> --json
```

`get` 按 ID 或短名称返回一个项目。必须提供项目引用参数。指定 `--company-id` 会添加 `companyId` 查询参数；服务器需要该参数才能明确解析短名称。因此，传入短名称而非 UUID 时务必同时指定公司 ID。

---

## 创建项目

```sh
paperclipai project create --company-id <company-id> --name "Launch Site"
```

`create` 限定在公司范围内，必须提供 `--company-id` 和 `--name`。其余选项均为可选项，并直接映射到项目记录。

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 创建项目所属的公司，必填。 |
| `--name <name>` | 项目名称，必填。 |
| `--description <text>` | 自由文本描述。 |
| `--status <status>` | 初始项目状态。 |
| `--goal-ids <csv>` | 以逗号分隔的关联目标 ID 列表。 |
| `--goal-id <id>` | 已弃用的单个目标 ID。建议使用 `--goal-ids`。 |
| `--lead-agent-id <id>` | 项目负责人代理。 |
| `--target-date <date>` | 目标日期/预计完成日期。 |
| `--color <value>` | 项目颜色。 |
| `--env-json <json>` | 作为 JSON 对象的项目 `env` 绑定。参见[JSON 字段：`env` 和执行工作区策略](#json-字段-env-和执行工作区策略)。 |
| `--execution-workspace-policy-json <json>` | 作为 JSON 对象的执行工作区策略。 |

关联目标应使用 `--goal-ids`；`--goal-id` 仅为兼容旧的单目标模型而保留。CLI 会在发送请求前按服务器的创建项目 schema 校验请求体，因此超出范围的值会在本地触发清晰错误，不必等服务器往返后才发现。

```sh
paperclipai project create \
  --company-id <company-id> \
  --name "Launch Site" \
  --description "Public marketing launch" \
  --status planned \
  --goal-ids <goal-id-1>,<goal-id-2> \
  --lead-agent-id <agent-id> \
  --color "#3b82f6"
```

---

## 更新项目

```sh
paperclipai project update <project-id-or-shortname> --status in_progress
paperclipai project update launch-site --company-id <company-id> --name "Launch v2"
```

`update` 接受项目引用（UUID 或短名称），只更新你传入的字段；省略的字段保持不变。与 `get` 一样，按短名称引用项目时请提供 `--company-id`，以确保解析到正确的公司。

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 按短名称查找项目时使用的公司 ID。 |
| `--name <name>` | 新项目名称。 |
| `--description <text\|null>` | 新描述；传入 `null` 可清除。 |
| `--status <status>` | 新项目状态。 |
| `--goal-ids <csv>` | 用此逗号分隔列表替换已关联的目标 ID。 |
| `--goal-id <id\|null>` | 已弃用的单个目标 ID；传入 `null` 可清除。 |
| `--lead-agent-id <id\|null>` | 新负责人代理；传入 `null` 可清除。 |
| `--target-date <date\|null>` | 新目标日期；传入 `null` 可清除。 |
| `--color <value\|null>` | 新颜色；传入 `null` 可清除。 |
| `--env-json <json\|null>` | 替换 `env` 绑定；传入 `null` 可清除。 |
| `--execution-workspace-policy-json <json\|null>` | 替换执行工作区策略；传入 `null` 可清除。 |
| `--archived-at <iso8601\|null>` | 归档时间戳；传入 `null` 可取消归档。 |

### 清除字段与省略字段的区别

可空字段遵循同一规则：传入字面值 `null` 可清除字段；省略标志则保持字段不变。适用于 `--description`、`--goal-id`、`--lead-agent-id`、`--target-date`、`--color`、`--env-json`、`--execution-workspace-policy-json` 和 `--archived-at`。

```sh
# Clear the lead agent and unarchive in one call
paperclipai project update <project-id> --lead-agent-id null --archived-at null

# Archive a project
paperclipai project update <project-id> --archived-at 2026-06-01T00:00:00Z
```

> **注意：** `--goal-ids` 会替换项目关联目标的整个集合。请传入你希望保留的完整列表，而不是增量项。

---

## 删除项目

```sh
paperclipai project delete <project-id-or-shortname> --yes
paperclipai project delete launch-site --company-id <company-id> --yes
```

`delete` 会移除项目。未提供 `--yes` 时命令会拒绝运行，并立即报告“Deletion requires --yes.”。按短名称引用项目时，请传入 `--company-id`。

> **警告：** 提供 `--yes` 后，项目会立即删除。删除前请使用 `project get` 确认引用对应正确项目；使用短名称时尤其要仔细核实。

---

<a id="json-字段-env-和执行工作区策略"></a>
## JSON 字段：`env` 和执行工作区策略

两个项目字段接受 CLI 传入的原始 JSON 配置：`env` 绑定（`--env-json`）和执行工作区策略（`--execution-workspace-policy-json`）。在 `create` 和 `update` 命令中，CLI 都会使用 `JSON.parse` 解析传入字符串，并将解析结果转发给 API。JSON 格式无效时，会在发出请求前于本地报错 `Invalid JSON: ...`；在 `update` 中传入字面值 `null` 则会清除字段。

这些字段会影响代理工作的实际运行方式。请记住，代理实际执行工作是在服务器端的 Paperclip 运行时和适配器中进行的；CLI 设置默认值，服务器负责执行这些配置。

### `--env-json`

`env` 绑定声明项目中的 issue 在执行时继承的环境变量。由秘密信息支持的条目引用一个具名 secret，而不是直接嵌入值，从而避免明文出现在项目记录中：

```sh
paperclipai project create \
  --company-id <company-id> \
  --name "Ops" \
  --env-json '{"OPENAI_API_KEY":{"kind":"secret","secretName":"openai-api-key"}}'
```

### `--execution-workspace-policy-json`

执行工作区策略控制如何为项目中的 issue 配置执行工作区，例如是否启用策略，以及新工作默认使用哪种模式：

```sh
paperclipai project update <project-id> \
  --execution-workspace-policy-json '{"enabled":true,"defaultMode":"shared_workspace"}'
```

要移除先前设置的策略，请传入 `null`：

```sh
paperclipai project update <project-id> --execution-workspace-policy-json null
```

> **提示：** 可将这些 JSON 值保存在文件中再传入，或用能够保证 JSON 有效的工具生成。CLI 会在发送前解析字符串，因此格式错误的对象会立即被拒绝，不会造成难以理解的服务器报错。

---

## 另请参阅

- [目标命令](./goal.md)——使用 `--goal-ids` 管理项目关联的目标。
- [Issue 命令](./issue.md)——项目内的工作对象，会继承项目的执行默认值。
- [代理命令](./agent.md)——可指定为项目负责人的代理。
- [工作区命令](./workspace.md)——检查和操作执行工作区策略配置的工作区。
- [通用选项](./common-options.md)——所有命令共用的客户端标志。
- [输出与脚本](./output-and-scripting.md)——`--json` 输出的行为及其使用方式。
