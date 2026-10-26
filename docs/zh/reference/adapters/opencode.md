---
paperclip_version: v2026.1023.0
seo_title: OpenCode 适配器
seo_description: 在 Paperclip 主机上运行 OpenCode，使用 OpenCode 自身的服务提供商和模型路由格式，并在多次心跳间恢复会话。
---

# OpenCode

`opencode_local` 会在与 Paperclip 相同的计算机上运行 OpenCode。如果你需要使用 OpenCode 的 `provider/model` 格式进行服务提供商/模型路由，并希望会话跨心跳恢复，请使用此适配器。

---

## 适用场景

- 你已经在本地使用 OpenCode。
- 希望使用服务提供商/模型路由，例如 `anthropic/claude-sonnet-4-5` 或 `openai/gpt-5.2-codex`。
- 希望 Paperclip 在多次心跳之间通过 `--session` 恢复 OpenCode 会话。

## 不适用场景

- 运行时位于 webhook 或 API 后面。请使用 [OpenClaw Gateway](./openclaw-gateway.md) 或 [HTTP](./http.md)。
- 只需要运行一次性的 shell 命令或脚本。请使用 [Process](./process.md)。
- 计算机上尚未安装 OpenCode CLI。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `cwd` | 否 | 绝对工作目录。建议设置。有权限时会创建该目录；否则回退到进程工作目录。 |
| `model` | **是** | `provider/model` 格式的 OpenCode 模型 ID（例如 `openai/gpt-5.2-codex`、`anthropic/claude-sonnet-4-5`）。 |
| `variant` | 否 | 通过 `--variant` 传递的服务提供商专属推理/配置变体。接受 `minimal`、`low`、`medium`、`high`、`xhigh`、`max`。 |
| `dangerouslySkipPermissions` | 否 | 注入临时运行时配置并设为 `permission=allow`，使所有工具和连接无需审批提示即可运行。Paperclip 无人值守执行时默认为 `true`。 |
| `promptTemplate` | 否 | 执行提示词模板。 |
| `instructionsFilePath` | 否 | 添加到执行提示词前面的 Markdown 指令文件绝对路径。 |
| `command` | 否 | 默认为 `opencode`。只有需要使用不同的可执行文件路径时才覆盖此值。 |
| `extraArgs` | 否 | 追加到 OpenCode 调用中的额外 CLI 参数。 |
| `env` | 否 | 环境变量。支持密钥引用。 |
| `timeoutSec` | 否 | 执行超时秒数。`0` 表示不设超时。 |
| `graceSec` | 否 | 强制停止前的 SIGTERM 宽限时间。 |

> **注意：** 对于 `opencode_local` 智能体，Paperclip 要求明确填写 `model` 值。请使用 `opencode models` 查看 `provider/model` 格式的有效选项。

---

## 会话持久化

已保存会话的 `cwd` 与当前 `cwd` 匹配时，OpenCode 会通过 `--session` 恢复会话。如果目录已经移动，就会启动新会话。

---

## 执行详情

- 运行命令格式为 `opencode run --format json ...`。
- 模型选择通过 CLI 标记 `--model` 传递。
- OpenCode 会在任务准备好的工作区中运行：Paperclip 通过 `--dir` 传递工作区路径。如果你的 `extraArgs` 在任何 `--` 之前已经设置了 `--dir`，则会保留你设置的值。
- 如果运行超时，会以 `OpenCode execution timed out` 错误失败。
- 系统会自动设置 `OPENCODE_DISABLE_PROJECT_CONFIG=true`，避免 OpenCode 将配置写入项目目录。
- 无人值守运行时（默认开启）如果启用 `dangerouslySkipPermissions`，Paperclip 会将现有 `opencode` 配置复制到临时 `XDG_CONFIG_HOME`，在该副本的 `opencode.json` 中设置 `permission=allow`，并让此次运行使用该副本。这会让 OpenCode 对所有工具和连接启用全自动模式，使无人值守执行不再因审批提示而停滞，同时不会修改真实配置。临时副本中你原配置的所有 `permission` 规则都会被替换；如果需要 OpenCode 遵守这些规则，请将 `dangerouslySkipPermissions` 设为 `false`。下文的环境覆盖变量（`PAPERCLIP_OPENCODE_PROVIDERS`、`PAPERCLIP_OPENCODE_SMALL_MODEL`）也会写入同一个临时文件，因此只有在开启 `dangerouslySkipPermissions` 且目标在本地运行时才会应用。

---

## 自定义服务提供商和网关

在 Paperclip 服务器（进程环境或智能体 `env`）中设置以下环境变量，可通过你自己的兼容 OpenAI 服务提供商或 LLM 网关路由 OpenCode。Paperclip 会在服务器端读取它们并写入上文所述的临时 `opencode.json`，因此只有开启 `dangerouslySkipPermissions` 且目标在本地运行时才会生效。

### `PAPERCLIP_OPENCODE_PROVIDERS`

此变量采用 OpenCode 的 `provider` 结构，是由服务提供商 ID 到其定义的映射对象。Paperclip 会将这些条目合并到临时 `opencode.json` 的 `provider` 区块中（保留现有服务提供商；同名键会被覆盖）。值不是对象的条目会被跳过，无效 JSON 也会被忽略；这两种情况都会显示执行说明，而不会静默失败。

这样可以添加自定义或远程兼容 OpenAI 的服务提供商（例如提供 `/v1` 接口的欧盟 LLM 网关），然后用常见的 `provider/model` 格式引用其模型。由于 OpenCode 只有在服务提供商的 `models` 映射中存在某模型时，才会解析 `--model provider/model`，因此必须为每个服务提供商明确列出计划使用的模型 ID。

可以使用 OpenCode 的 `{env:VAR}` 占位符，避免将密钥写入 JSON。Paperclip 会在服务器端写入文件之前，先根据执行环境变量（然后是进程环境变量）展开占位符；无法解析的占位符会保持原样，交由 OpenCode 自行解析。

```json
{
  "my-gateway": {
    "npm": "@ai-sdk/openai-compatible",
    "name": "My EU Gateway",
    "options": {
      "baseURL": "https://gateway.example.com/v1",
      "apiKey": "{env:MY_GATEWAY_KEY}"
    },
    "models": {
      "gpt-5.2-codex": {},
      "gpt-5.2-codex-small": {}
    }
  }
}
```

设置以上内容后，智能体就可以使用 `"model": "my-gateway/gpt-5.2-codex"`。

> **提示：** OpenCode 默认会使用 `opencode models` 验证已配置模型。如果经网关路由的模型不会出现在该列表中，请设置 `OPENCODE_ALLOW_ALL_MODELS=true`，让 Paperclip 跳过可用性探测（但仍会要求使用 `provider/model` 格式）。

### `PAPERCLIP_OPENCODE_SMALL_MODEL`

该变量通过向临时 `opencode.json` 写入 `small_model`，指定 OpenCode 辅助用的“小型”模型——例如生成会话标题时使用的模型。否则 OpenCode 会回退到每个服务提供商内置的默认值；如果你将服务提供商改为不提供该默认模型的网关，辅助调用就会失败并中止运行。请将此变量设置为网关实际提供的模型，以确保所有调用都使用受支持的模型。

```bash
PAPERCLIP_OPENCODE_SMALL_MODEL=my-gateway/gpt-5.2-codex-small
```

---

## 模型

OpenCode 支持多个服务提供商。常见模型 ID 如下：

| ID | 服务提供商 |
|---|---|
| `openai/gpt-5.2-codex` | OpenAI（默认） |
| `openai/gpt-6-astra` | OpenAI |
| `openai/gpt-6.1-sol` | OpenAI |
| `openai/gpt-6-sol` | OpenAI |
| `openai/gpt-6-luna` | OpenAI |
| `openai/gpt-5.6-sol` | OpenAI |
| `openai/gpt-5.6-terra` | OpenAI |
| `openai/gpt-5.6-luna` | OpenAI |
| `anthropic/claude-opus-5-5` | Anthropic |
| `anthropic/claude-opus-5` | Anthropic |
| `anthropic/claude-fable-5-1` | Anthropic |
| `anthropic/claude-sonnet-5-5` | Anthropic |
| `anthropic/claude-sonnet-5` | Anthropic |
| `google/gemini-3.8-flash` | Google |
| `xai/grok-4.7` | xAI |
| `openai/gpt-5.5` | OpenAI |
| `openai/gpt-5.4` | OpenAI |
| `openai/gpt-5.4-mini` | OpenAI |
| `openai/gpt-5.2` | OpenAI |
| `openai/gpt-5.1-codex-max` | OpenAI |
| `openai/gpt-5.1-codex-mini` | OpenAI |

运行 `opencode models` 查看本机的权威模型列表。

不过，你不只限于该列表中的模型。只有当服务提供商的 `models` 映射中存在模型 ID 时，OpenCode 才会解析 `--model provider/model`。因此，如果打包目录中没有某个 ID（例如目录发布后才推出的模型，或 `openai/gpt-oss-120b:nitro` 这样的路由变体），即使服务提供商支持它，也会被拒绝并报错 `Model not found`。

Paperclip 会将你在 `model` 中填写的值注册到上文所述的临时 `opencode.json` 中；如果相应服务提供商的 `models` 映射里还没有这个模型，就会将其加入。目录中已有的模型会保留全部元数据；你自己的 OpenCode 配置或 `PAPERCLIP_OPENCODE_PROVIDERS` 中显式定义的模型始终优先，因此 Paperclip 不会覆盖它们。添加模型条目时，执行记录会显示一条模型已注册的说明。与本页其他运行时配置一样，只有开启 `dangerouslySkipPermissions` 且目标在本地运行时，此功能才会应用。

执行开始前，Paperclip 会通过 `opencode models` 预先检查配置的模型。如果模型出现在返回列表中，执行会继续。如果列表非空但没有该模型，Paperclip 会刷新 OpenCode 磁盘上的目录一次（`opencode models --refresh`），并在放弃前重新检查——运行时间较长的主机可能会报告过时目录，而服务提供商已经能提供该模型。如果仍然找不到，执行就会被拒绝，并显示 `Configured OpenCode model is unavailable: <model>`，同时列出部分可用 ID，方便你修正 `model` 值。

预检只是尽力而为的保护措施，并非硬性门槛。如果 `opencode models` 无法运行（超时或出错），或未返回任何模型，Paperclip 会记录警告并继续使用你配置的模型，而不会阻止执行，因为真正的调用结果才是最终依据。（如上所述，设置 `OPENCODE_ALLOW_ALL_MODELS=true` 会完全跳过探测。）

---

## 示例

```json
{
  "adapterType": "opencode_local",
  "adapterConfig": {
    "cwd": "/Users/me/projects/paperclip-workspace",
    "model": "anthropic/claude-sonnet-4-5",
    "variant": "high",
    "promptTemplate": "You are the engineering lead. Work carefully and report progress.",
    "env": {
      "ANTHROPIC_API_KEY": {
        "type": "secret_ref",
        "secretId": "anthropic-key",
        "version": "latest"
      }
    },
    "timeoutSec": 300,
    "graceSec": 15,
    "dangerouslySkipPermissions": true
  }
}
```

---

## 后续阅读

- [创建适配器](./creating-an-adapter.md)
- [适配器界面解析器契约](./adapter-ui-parser.md)
