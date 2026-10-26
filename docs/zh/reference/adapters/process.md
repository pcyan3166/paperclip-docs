---
paperclip_version: v2026.1012.0
seo_title: Process 适配器
seo_description: 在 Paperclip 主机上运行任意命令。如果你的智能体实际上是脚本、shell 命令或自定义运行时，这是合适的选择。
---

# Process

`process` 适配器会在与 Paperclip 相同的计算机上运行任意命令。如果你的智能体只是一个脚本、shell 命令，或一个已具备自行处理工作的自定义运行时，请使用此适配器。

> **信息：** `process` 是 Paperclip 运行时使用的内置内部适配器。目前它在智能体配置中的适配器类型下拉菜单里显示为 **“即将推出”**，无法手动选择。若要立即使用，请通过 API 或导入公司导出包的方式配置智能体。

---

## 适用场景

- 希望将本地脚本作为智能体运行。
- 已有自定义命令行工作流程。
- 希望 Paperclip 将公司上下文注入进程并收集进程输出。

## 不适用场景

- 运行时是长期运行的服务或远程 webhook。请改用 [HTTP](./http.md)。
- 需要内置会话持久化或特定服务商 CLI 集成。请改用一种本地适配器。
- 需要比纯 stdout/stderr 捕获更丰富的执行记录解析器。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `command` | 是 | 可执行命令或脚本路径。 |
| `args` | 否 | 命令参数，可以是字符串数组或以空格分隔的字符串。 |
| `cwd` | 否 | 进程工作目录。通常必须为绝对路径。 |
| `env` | 否 | 传递给进程的额外环境变量。 |
| `timeoutSec` | 否 | 执行超时秒数。`0` 表示不设超时。 |
| `graceSec` | 否 | 强制停止前的宽限时间。服务器适配器默认值为 `15`。 |

> **注意：** Process 适配器不会另行发明更高级的协议。它会启动命令、传输日志，并返回 stdout/stderr 和退出元数据。

---

## 工作原理

1. Paperclip 构建运行时环境并注入标准的 `PAPERCLIP_*` 变量。
2. 适配器解析用于日志和执行的命令。
3. Paperclip 使用已配置的参数启动子进程。
4. 进程退出、超时或失败时，此次运行结束。
5. stdout 和 stderr 会随执行结果返回，以供界面渲染。

如果进程以非零状态退出，执行结果会返回错误消息和原始输出载荷。

---

<a id="connection-instructions"></a>
## 连接说明

连接可以携带供智能体使用的说明，例如如何使用 Honcho 或 Mem0 等记忆服务提供商。当智能体获得授权使用的任一连接启用了说明时，Paperclip 会将其交给你的进程，使模型能够实际读取。

进程通过环境变量 `PAPERCLIP_CONNECTION_INSTRUCTIONS_FILE` 获取说明：

- 有适用说明时，该变量包含私有 JSON 文件的路径（仅进程所有者可读），文件中包含 `{ "text": "...", "digest": "..." }`。
- 没有适用说明时，该变量会设为空字符串，而不是路径。Paperclip 始终会覆盖此变量，因此不会泄漏从主机或 `env` 配置继承的值。

`text` 是可直接使用的 Markdown。请读取文件并将 `text` 附加到发送给模型的提示词中；不要传递整个对象。`digest` 是 `text` 的 SHA-256 十六进制哈希，可用于识别不同运行间说明的变化。

```python
import json, os

path = os.environ.get("PAPERCLIP_CONNECTION_INSTRUCTIONS_FILE", "")
extra = ""
if path:
    with open(path) as f:
        extra = json.load(f)["text"]
prompt = base_prompt + ("\n\n" + extra if extra else "")
```

请在进程运行期间读取此文件。Paperclip 会在启动前创建该文件，并在进程退出后立即删除。

用户如何编写和编辑说明，请参阅[已保存的连接说明](../../connectors/connection-instructions.md)。

---

## 环境测试

**Test Environment**（测试环境）按钮会检查：

- 已配置 `command`。
- 工作目录已存在或可以创建。
- 可在目标计算机上解析该命令。

如果找不到命令，适配器就不会运行。

---

## 示例

```json
{
  "adapterType": "process",
  "adapterConfig": {
    "command": "python3",
    "args": ["scripts/run-agent.py", "--company-id", "company-1"],
    "cwd": "/Users/me/projects/paperclip-workspace",
    "env": {
      "OPENAI_API_KEY": {
        "type": "secret_ref",
        "secretId": "secret-id",
        "version": "latest"
      }
    },
    "timeoutSec": 300,
    "graceSec": 15
  }
}
```

---

## 实用说明

- 如果希望完全控制运行时，请使用 Process 适配器。
- 如果脚本需要回调 Paperclip，请使用注入的 `PAPERCLIP_API_URL` 和 `PAPERCLIP_API_KEY`。
- 尽可能让命令保持确定性，以便更轻松地调试心跳。

---

## 后续阅读

- [创建适配器](./creating-an-adapter.md)
- [外部适配器](./external-adapters.md)
