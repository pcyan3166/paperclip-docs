---
paperclip_version: v2026.1023.0
seo_title: Cursor Local 适配器
seo_description: 在 Paperclip 主机上运行 Cursor Agent CLI，使聊天会话可跨心跳继续，并在执行视图中输出结构化流。
---

# Cursor Local

`cursor` 会在与 Paperclip 相同的计算机上运行 Cursor Agent CLI。如果你希望 Cursor 聊天会话跨心跳恢复，并在执行日志中提供结构化流输出，请使用此适配器。本页还介绍单独的 [Paperclip Runner 上的 Cursor 路径](#cursor-on-paperclip-runner)。现有 `cursor` 智能体会继续使用 CLI 适配器，直到你更改其配置。

---

## 适用场景

- 你已经在本地使用 Cursor Agent CLI。
- 希望 Paperclip 在多次心跳之间通过会话恢复（`--resume`）运行 Cursor。
- 希望在执行日志中看到结构化流输出（`--output-format stream-json`）。

## 不适用场景

- 需要类似 webhook 的外部调用。请使用 [OpenClaw Gateway](./openclaw-gateway.md) 或 [HTTP](./http.md)。
- 只需要运行一次性的 shell 命令。请使用 [Process](./process.md)。
- 尚未安装 Cursor Agent CLI，或无法通过 `PATH` 找到它。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `cwd` | 否 | 绝对工作目录。建议设置。有权限时会创建该目录；否则回退到进程工作目录。 |
| `model` | 否 | Cursor 模型 ID。默认为 `auto`。常见选择包括 `auto`、`composer-2.5`、`claude-opus-5-5`、`gpt-5.6-sol`、`composer-1.5`、`gpt-5.3-codex`、`opus-4.6-thinking`、`sonnet-4.6`、`gemini-3-pro` 和 `grok`。 |
| `mode` | 否 | 作为 `--mode` 传递给 Cursor 的执行模式。可选 `plan` 或 `ask`。留空时使用普通自主运行模式。 |
| `promptTemplate` | 否 | 执行提示词模板。 |
| `instructionsFilePath` | 否 | 添加到执行提示词前面的 Markdown 指令文件绝对路径。 |
| `command` | 否 | 默认为 `agent`。仅在需要覆盖默认可执行文件路径时设置。 |
| `extraArgs` | 否 | 追加到 Cursor 调用命令中的额外 CLI 参数。 |
| `env` | 否 | 环境变量。支持密钥引用。 |
| `timeoutSec` | 否 | 执行超时秒数。`0` 表示不设超时。 |
| `graceSec` | 否 | 强制停止前等待 SIGTERM 的宽限时间。 |

---

## 会话持久化

Cursor Local 会保存 Cursor 会话 ID；下次心跳时，如果保存的会话 `cwd` 与当前 `cwd` 匹配，就会通过 `--resume` 恢复会话。若已保存的 cwd 不再匹配，则会启动新会话。

---

## 执行详情

- 运行命令格式为 `agent -p --output-format stream-json ...`。
- 提示词通过 stdin 传递给 Cursor。
- Cursor 界面解析器会将结构化流输出解析为执行记录条目。

---

## 模型

适配器接受的常见模型 ID：

```
auto
composer-2.5, composer-1.5, composer-1
claude-opus-5-5, claude-fable-5-1, claude-sonnet-5
gpt-5.6-sol, gpt-5.6-terra, gpt-5.6-luna
gemini-3.8-flash
muse-spark-1.3
grok-4.7, grok-4.6, grok-4.5
gpt-5.3-codex, gpt-5.3-codex-high, gpt-5.3-codex-xhigh
gpt-5.2, gpt-5.2-codex, gpt-5.2-codex-high
gpt-5.1-codex-max, gpt-5.1-codex-mini
opus-4.6, opus-4.6-thinking, opus-4.5, opus-4.5-thinking
sonnet-4.6, sonnet-4.6-thinking, sonnet-4.5, sonnet-4.5-thinking
gemini-3.1-pro, gemini-3-pro, gemini-3-flash
grok, kimi-k2.5
```

`auto` 是安全的默认值——Cursor 会为工作选择合适的模型。

---

## 示例

```json
{
  "adapterType": "cursor",
  "adapterConfig": {
    "cwd": "/Users/me/projects/paperclip-workspace",
    "model": "auto",
    "promptTemplate": "You are the engineering lead. Work carefully and report progress.",
    "timeoutSec": 300,
    "graceSec": 15
  }
}
```

---

<a id="cursor-on-paperclip-runner"></a>
## 在 Paperclip Runner 上运行 Cursor

如果你希望 Cursor 使用原生 Runner 生命周期和 Paperclip 任务工具，请选择这条实验性路径。它使用 `paperclip_runner` 适配器和固定版本的 ACP 运行时，而不是 `PATH` 上的 `agent` 可执行文件。

### 准备执行主机

在运行 Paperclip 的同一操作系统账户下，于智能体执行所在的主机上运行：

```sh
paperclipai runtime setup cursor
```

此命令会下载并验证固定版本的运行时，不会调用模型。支持 macOS ARM64、macOS x64 和 Linux x64。自定义执行镜像也需要准备对应运行时。请参阅[运行时命令](../cli/runtime.md)。

安装运行时不会对 Cursor 进行身份验证。请通过智能体的密钥支持环境变量提供 `CURSOR_API_KEY` 或 `CURSOR_AUTH_TOKEN`。原生路径使用私有运行时状态，不会借用个人的交互式 Cursor 登录。

### 配置智能体

1. 如果实例尚未启用，请在 Experimental 设置中启用 **Paperclip Runner**。
2. 选择 **Paperclip Runner**，将 **Harness** 设为 **ACP agents**，并将 **ACP agent** 设为 **Cursor**。
3. 明确填写 Cursor 模型 ID；此路径没有默认模型。
4. 选择 **Cursor mode**，检查 ACP 权限，然后保存。
5. 分配工作前，使用 **Test Environment** 检查运行环境是否就绪。

```json
{
  "adapterType": "paperclip_runner",
  "adapterConfig": {
    "provider": "acpx",
    "acpxAgent": "cursor",
    "model": "composer-2.5",
    "acpxSessionMode": "agent",
    "acpxPermissionMode": "approve-all"
  }
}
```

示例中的模型只是一个选择项，并不能证明你的 Cursor 账户可以使用它。开始工作前会检查模型资格和身份验证。

| Cursor 模式 | 配置值 | 行为 |
|---|---|---|
| **Agent** | `agent` | Cursor 默认自主会话模式。 |
| **Plan** | `plan` | 原生计划模式。接受计划不会自动切换到 Agent 模式或安排实施工作。 |
| **Ask** | `ask` | 原生提问/研究模式。 |

会话模式独立于 `acpxPermissionMode` 和任务工作模式。更改它不会扩大公司权限，也不会将执行环境变为操作系统沙盒。接受原生计划后，即使计划运行已成功，任务仍可能等待你的下一条消息。

### 当前限制

- Cursor 使用 Paperclip 的问题工具获取用户输入；不要假设 Cursor CLI 的所有交互功能都能在 Paperclip 中使用。
- 原生加载会恢复由 Runner 拥有的会话。无法使用服务提供商原生的分叉和恢复方法，也不支持实时引导；后续消息会通过控制器队列处理。
- 部分原生计数器可能以诊断通知的形式显示，不能据此确认权威令牌总数或单次运行的美元费用。
- Runner 会拒绝使用环境中现存的 MCP、hook 和插件执行配置。请通过 Paperclip 连接允许使用的工具，不要依赖项目发现机制。
- 运行时验证只能确认已安装并可就绪，不能证明你有权使用特定模型，也不能保证付费执行会成功。

有关功能默认状态、服务提供商选项和权限模式，请参阅 [Paperclip Runner](paperclip-runner.md)。

---

## 后续阅读

- [Paperclip Runner](./paperclip-runner.md)
- [运行时命令](../cli/runtime.md)
- [创建适配器](./creating-an-adapter.md)
- [适配器界面解析器契约](./adapter-ui-parser.md)
