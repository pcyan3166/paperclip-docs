---
paperclip_version: v2026.609.0
seo_title: CLI 输出与脚本
seo_description: 将 paperclipai 接入脚本、CI 步骤或 AI 操作循环：了解两种输出模式、退出码以及可靠解析结果的方法。
---

# 输出与脚本

将 `paperclipai` 接入脚本、CI 步骤或 AI 操作循环，而不是人工查看输出时，请参阅本页。内容包括两种输出模式（`--json` 和人类可读格式）、CLI 如何通过退出码和 `stderr` 表示成功与失败，以及捕获 ID、通过管道传给 `jq` 和以非交互方式运行命令的实用方法。

这里介绍的行为都来自共享客户端层 `cli/src/commands/client/common.ts`，所有控制平面命令都会经过这一层。规则是一致的：如果命令接受 `--json`，输出方式就与其他命令相同；失败方式也相同。

---

## 两种输出模式

每条客户端命令都通过同一个共享渲染器输出结果。渲染器只有两种模式，由 `--json` 标志选择。

| 模式 | 标志 | 输出 |
|---|---|---|
| 人类可读 | （默认） | 适合在终端阅读的紧凑逐行格式。 |
| 机器可读 | `--json` | API 原始结果，以两个空格缩进格式化为 JSON。 |

> **提示：** 使用 CLI 编写脚本或将输出通过管道传给其他工具时，始终添加 `--json`。人类可读格式面向人工查看，不是稳定接口——列顺序和截断方式可能随版本改变。JSON 是 API 返回的数据，脚本应解析它。

### `--json` 的输出内容

指定 `--json` 后，CLI 会对 API 结果执行 `JSON.stringify(data, null, 2)`，并且不会向 `stdout` 输出其他内容。没有横幅、标签或颜色代码，只有可以直接通过管道传递的 JSON 文档。

```sh
paperclipai issue get PAP-39 --json
```

```json
{
  "id": "244c0c2c-8416-43b6-84c9-ec183c074cc1",
  "identifier": "PAP-39",
  "title": "Implement caching layer",
  "status": "in_progress",
  "priority": "high"
}
```

### 人类可读模式的输出内容

不指定 `--json` 时，渲染器会根据结果的结构调整输出格式：

- **数组**：每项输出一行。每个对象项会展平为 `key=value` 行；渲染器优先输出 `identifier`、`id`、`name`、`status`、`priority`、`title` 和 `action`（若存在，按此顺序），再追加其余标量字段。嵌套对象不会显示在该行中，较长字符串会截断至约 90 个字符。空数组输出 `(empty)`。
- **单个对象**：以美化的 JSON 输出，与 `--json` 一样使用双空格缩进。因此读取单条记录的 `get` 命令在人类可读模式和 `--json` 模式中看起来几乎一样。
- **标量**（字符串、数字或布尔值）：直接输出其值。`null` / `undefined` 输出 `(null)`。

```sh
paperclipai issue list --company-id <company-id> --status todo,in_progress
```

```text
identifier=PAP-39 id=244c0c2c... status=in_progress priority=high title=Implement caching layer
identifier=PAP-41 id=9b1f0a2c... status=todo priority=medium title=Add request tracing
```

用 `key=value` 显示列表方便快速浏览，但会省略嵌套字段并截断字符串，因此**不要**解析这种格式。需要从列表中提取值时，请切换到 `--json` 并使用 JSON 解析器。

---

## 退出码与错误行为

CLI 遵循标准 shell 约定：命令成功时以 `0` 退出；失败时以非零退出码退出，并将错误写入 `stderr`。

| 结果 | 退出码 | 流 | 格式 |
|---|---|---|---|
| 成功 | `0` | `stdout` | 渲染后的结果（指定 `--json` 时为 JSON）。 |
| API 错误 | `1` | `stderr` | `API error <status>: <message>[ details=<json>]`。 |
| 其他错误 | `1` | `stderr` | 错误消息文本。 |

所有错误都由同一个处理器路由，因此各命令的行为一致。API 错误（Paperclip 服务器返回 HTTP 错误响应）会以 `API error <status>: <message>` 格式打印；服务器返回结构化详细信息时，会附加 `details=<json>`，例如检出冲突可能显示 `API error 409: ...`，认证失败可能显示 `API error 401: ...`。其他错误（错误标志、缺少公司 ID、连接被拒绝等）会直接打印普通错误消息。两种情况的退出码都是 `1`。

结果写入 `stdout`，错误写入 `stderr`，因此可以分别处理：

```sh
# Capture JSON output, let errors flow to the terminal
result=$(paperclipai issue get PAP-39 --json) || exit 1

# Discard human chatter, keep only errors
paperclipai issue list --company-id <company-id> --json 2>/tmp/pc-errors.log
```

> **注意：** 退出码非零始终表示命令没有完成，请检查退出状态。在 `paperclipai ... --json | jq ...` 这样的管道中，默认看到的是 `jq` 的退出状态，而不是 CLI 的状态；在 bash 中使用 `set -o pipefail`，让上游失败时整个管道也失败。

### 连接错误

如果 CLI 无法访问 API，错误消息会包含尝试访问的 URL，并提示在该地址检查 `GET /api/health`。这类问题通常是 API 基础 URL 解析（见下文）指向了没有监听服务器的地址。完整解析顺序请参阅[通用选项](common-options.md)。

---

## 以非交互方式运行

对脚本或自主代理而言，首要原则是绝不让命令停下来等待提示。两种情况可能触发交互式流程，都可以避免。

**1. 交互式看板重新认证。** 命令遇到 `401`，或需要看板/实例管理员权限的 `403` 时，CLI 可能尝试启动交互式看板登录来恢复访问。只有在 `stdin` 和 `stdout` 都是 TTY，且没有显式提供 `--api-key` 时才会这样做。在脚本中，输出通常经过管道传递或重定向，因此 `stdout` 不是 TTY，CLI 会跳过恢复流程并直接返回 API 错误。要确保在任何环境中都不会进入交互流程，请始终显式提供凭据：

```sh
paperclipai issue list \
  --company-id <company-id> \
  --api-base https://paperclip.example.com \
  --api-key "$PAPERCLIP_API_KEY" \
  --json
```

传入 `--api-key`（或设置 `PAPERCLIP_API_KEY`）会完全禁用交互恢复流程，因此凭据有问题时命令会立即失败，而不是一直等待。

**2. 确认提示。** 破坏性命令会在 TTY 中要求确认；不使用 TTY 时则需要显式标志才能继续。对于支持 `--yes` 的命令（例如 `skills remove`、`skills reset`），请传入该标志；有明确要求的命令则需提供完整确认标志——例如 `company delete` 要求同时提供 `--yes` 和 `--confirm <id>`，且服务器还会执行额外限制。具体流程请参阅[公司](./company.md)。

### 脚本中的凭据解析顺序

API 密钥按以下顺序解析：`--api-key`、`PAPERCLIP_API_KEY`，然后是上下文配置文件 `apiKeyEnvVarName` 指向的环境变量。用于自动化时，优先采用前两种方式，这样执行就不依赖机器上存在配置文件。公司 ID 按以下顺序解析：`--company-id`（如 `run` 等命令也支持别名 `-C`）、`PAPERCLIP_COMPANY_ID`，然后是配置文件默认值。公司范围命令如果找不到公司 ID，会明确提示你传入 `--company-id` 或设置相应环境变量。

适用于 CI、与配置文件无关的调用方式如下：

```sh
export PAPERCLIP_API_URL="https://paperclip.example.com"
export PAPERCLIP_API_KEY="<token>"
export PAPERCLIP_COMPANY_ID="<company-id>"

paperclipai issue list --status todo --json
```

> **提示：** 在 CI 或共享主机上运行 CLI 时，可使用 `--data-dir <path>` 将所有本地状态与 `~/.paperclip` 隔离，避免不同作业间泄露上下文、配置和凭据。参阅[通用选项](common-options.md)。

---

## 捕获 ID 并通过管道传递给 jq

大多数脚本都遵循相同流程：运行命令，从 JSON 中提取 ID 或状态，再将其传给下一条命令。`--json` 配合 `jq` 几乎可以满足所有需求。

### 提取单个字段

```sh
# Create an issue and capture its ID for the next step
issue_id=$(paperclipai issue create \
  --company-id <company-id> \
  --title "Ship the export pipeline" \
  --status todo --priority high \
  --json | jq -r '.id')

paperclipai issue comment "$issue_id" --body "Kicking this off." --json
```

`-r` 标志让 `jq` 输出不带引号的原始字符串，适用于 shell 变量和 URL。

### 筛选和重整列表

```sh
# IDs of every todo issue, one per line
paperclipai issue list --company-id <company-id> --status todo --json \
  | jq -r '.[].id'

# Just the identifier and title, as TSV
paperclipai issue list --company-id <company-id> --json \
  | jq -r '.[] | [.identifier, .title] | @tsv'

# Count issues by status
paperclipai issue list --company-id <company-id> --json \
  | jq 'group_by(.status) | map({status: .[0].status, count: length})'
```

### 遍历结果

```sh
# Release every issue currently assigned to a given agent
paperclipai issue list --company-id <company-id> \
  --assignee-agent-id <agent-id> --status in_progress --json \
  | jq -r '.[].id' \
  | while read -r id; do
      paperclipai issue release "$id" --json
    done
```

### 发送 JSON 请求体

数据流也可以反向传递。映射到较宽泛服务器 schema 的命令接受 `--payload-json`；包含正文的技能命令则可从 `stdin` 读取内容：

```sh
# Build a payload with jq, pass it straight in
paperclipai agent create --company-id <company-id> \
  --payload-json "$(jq -nc '{name:"Builder", adapterType:"codex_local"}')"

# Author a skill body from stdin
cat house-style.md | paperclipai skills create \
  --name "House Style" --slug house-style --body-file -
```

> **警告：** 在 shell 中插入请求体变量时要正确加引号。未引用的 `--payload-json` 值中若包含空格或 shell 元字符，会在 CLI 收到前被拆分或破坏。请为字面字符串加单引号，或使用示例中的 `jq -nc` 构建 JSON 并用双引号括住替换结果。

---

## 完整脚本示例

下面的脚本调用具备健壮的处理方式：遇到错误立即停止、不显示交互提示、只解析 JSON，并将输出与诊断信息分开。

```sh
#!/usr/bin/env bash
set -euo pipefail

export PAPERCLIP_API_URL="https://paperclip.example.com"
export PAPERCLIP_API_KEY="<token>"
export PAPERCLIP_COMPANY_ID="<company-id>"

# Find the highest-priority open issue, or exit cleanly if none.
issue_id=$(paperclipai issue list --status todo --json \
  | jq -r 'sort_by(.priority) | reverse | .[0].id // empty')

if [ -z "$issue_id" ]; then
  echo "No open work." >&2
  exit 0
fi

paperclipai issue comment "$issue_id" \
  --body "Picked up by the nightly script." --json >/dev/null

echo "Commented on $issue_id"
```

`set -euo pipefail` 会让脚本在首个命令失败时中止（包括 `jq` 管道内部），因此遇到 `401` 或 `409` 时不会继续使用空数据。每次 CLI 调用都传入 `--json`，不依赖人类可读渲染器。核心原则就是：显式提供凭据、每处都使用 `--json`，并检查退出码。

---

## 另请参阅

- [通用选项](common-options.md)——完整标志列表、API 基础 URL 解析顺序和上下文配置文件。
- [认证](./authentication.md)——看板令牌与代理密钥的区别，以及 `--api-key` 的解析方式。
- [Issue 命令](./issue.md)——大多数脚本会使用的命令组。
- [运行](./run.md)——从终端检查和控制心跳运行。
