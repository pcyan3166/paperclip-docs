---
paperclip_version: v2026.1023.0
seo_title: Pi 适配器
seo_description: 在 Paperclip 主机上运行 Pi 编程智能体 CLI，使用内置工具集（read、bash、edit、write、grep、find、ls）和服务提供商配置。
---

# Pi

`pi_local` 会在与 Paperclip 相同的计算机上运行 Pi 编程智能体 CLI。如果你需要 Pi 内置工具集（read、bash、edit、write、grep、find、ls）、服务提供商/模型路由，以及在多次心跳之间恢复会话的能力，请使用此适配器。

---

## 适用场景

- 你已经在本地使用 Pi CLI。
- 希望按 Pi 的 `--provider <name> --model <id>` 格式配置服务提供商和模型路由。
- 希望 Paperclip 在多次心跳之间通过 `--session` 恢复 Pi 会话。
- 需要智能体使用 Pi 的工具集。

## 不适用场景

- 运行时位于 webhook 或 API 后面。请使用 [OpenClaw Gateway](./openclaw-gateway.md) 或 [HTTP](./http.md)。
- 只需要运行一次性的 shell 命令。请使用 [Process](./process.md)。
- 计算机上尚未安装 Pi CLI。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `cwd` | 否 | 绝对工作目录。建议设置。有权限时会创建该目录；否则回退到进程工作目录。 |
| `model` | **是** | `provider/model` 格式的 Pi 模型 ID（例如 `xai/grok-4`）。 |
| `thinking` | 否 | 传递给 Pi 的思考级别。接受 `off`、`minimal`、`low`、`medium`、`high`、`xhigh`。 |
| `promptTemplate` | 否 | 通过 `-p` 标记传递的用户提示词模板。 |
| `instructionsFilePath` | 否 | Markdown 指令文件的绝对路径，通过 `--append-system-prompt` 追加到 Pi 的系统提示词中。 |
| `command` | 否 | 默认为 `pi`。只有需要使用不同的可执行文件路径时才覆盖此值。 |
| `env` | 否 | 环境变量。支持密钥引用。 |
| `timeoutSec` | 否 | 执行超时秒数。`0` 表示不设超时。 |
| `graceSec` | 否 | 强制停止前的 SIGTERM 宽限时间。 |

> **注意：** 对于 `pi_local` 智能体，Paperclip 要求明确填写 `model` 值。请在本机运行 `pi --list-models` 查看有效选项。

---

## 会话持久化

Pi 会将会话存储在 `~/.pi/paperclips/` 下，并在下次心跳时通过 `--session` 恢复。会话按智能体划分，因此多个 Paperclip 智能体可以共用一台主机而互不干扰。

---

## 执行详情

- 智能体指令会通过 `--append-system-prompt` 追加到 Pi 的系统提示词中。
- 用户任务提示词通过 `-p` 传递。
- 所有工具（`read`、`bash`、`edit`、`write`、`grep`、`find`、`ls`）默认启用。
- 如果配置的模型不可用，错误会指出模型名称和 Pi 找到的模型数量，并建议运行 `pi --list-models`，而不是粘贴完整模型列表。

---

## 自定义服务提供商和网关路由

Pi 没有基础 URL 标记或环境变量，因此指向自定义或远程兼容 OpenAI/Anthropic 的端点，唯一方法是在其智能体配置目录中提供 `models.json` 文件。Paperclip 通过环境变量 `PAPERCLIP_PI_PROVIDERS` 为你完成配置。

将 `PAPERCLIP_PI_PROVIDERS` 设置为符合 Pi `models.json` 中 `providers` 结构的 JSON 对象——以服务提供商名称为键，每个条目描述端点地址和提供的模型。Paperclip 会在服务器端读取此变量，将 `{"providers": ...}` 写入临时智能体配置目录中的受管理 `models.json`，并在本次执行期间将 `PI_CODING_AGENT_DIR` 指向该目录。（对于远程执行目标，该目录会发送到沙盒，并将 `PI_CODING_AGENT_DIR` 重新指向沙盒内的副本。）

由于 Pi 会依据注册表中的精确 `(provider, model id)` 匹配来解析 `--provider <name> --model <id>`，每个服务提供商都必须明确列出所提供的模型。

```json
{
  "tensorix": {
    "baseUrl": "http://gateway.example.svc.cluster.local:8080/anthropic",
    "apiKey": "{env:ANTHROPIC_API_KEY}",
    "api": "anthropic-messages",
    "models": [
      {
        "id": "deepseek/deepseek-chat-v3.1",
        "name": "DeepSeek v3.1",
        "reasoning": false,
        "input": ["text"],
        "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 },
        "contextWindow": 128000,
        "maxTokens": 32000
      }
    ]
  }
}
```

行为说明：

- **密钥保持声明式配置。** `apiKey` 可以是字面值，也可以是 `{env:VAR}` 占位符。占位符会在服务器端根据执行 `env` 和 Paperclip 进程环境变量展开，因此网关密钥会写入 `models.json`，确保其值存在。如果占位符无法解析（未设置或为空字符串），会保持原样，交由 Pi 自行尝试解析。
- **受管理目录会替代主机智能体目录。** 凭据通过服务提供商配置（字面密钥或展开后的 `apiKey`）传递，而不是从主机的 `~/.pi/agent` 获取。
- **配置错误会显示出来，而不是静默失败。** JSON 无效、值不是对象，或服务提供商条目本身不是对象时，系统会忽略这些配置并显示诊断说明，而不会等到后续再报含义不明的“找不到模型”错误。未设置变量（或变量为空）是正常的“关闭此功能”状态。

Pi 随后会根据此注册表，将 `--provider <name> --model <id>` 中的服务提供商键（例如 `tensorix`）和你列出的模型 `id`（例如 `deepseek/deepseek-chat-v3.1`）进行匹配，以选择路由。

---

## 示例

```json
{
  "adapterType": "pi_local",
  "adapterConfig": {
    "cwd": "/Users/me/projects/paperclip-workspace",
    "model": "xai/grok-4",
    "thinking": "high",
    "promptTemplate": "You are the engineering lead. Work carefully and report progress.",
    "env": {
      "XAI_API_KEY": {
        "type": "secret_ref",
        "secretId": "xai-api-key",
        "version": "latest"
      }
    },
    "timeoutSec": 300,
    "graceSec": 15
  }
}
```

---

## 后续阅读

- [创建适配器](./creating-an-adapter.md)
- [适配器界面解析器契约](./adapter-ui-parser.md)
