---
paperclip_version: v2026.1005.0
seo_title: Kimi Code 适配器
seo_description: 在 Paperclip 主机上将 Moonshot Kimi Code CLI 作为本地编程智能体运行。默认使用共享 ACP 引擎，也可选择无头 CLI 路径。
---

# Kimi Code

`kimi_local` 会在与 Paperclip 相同的计算机上运行 Moonshot 的 Kimi Code CLI（`kimi`）。如果你需要能够输出结构化实时执行记录、跨心跳恢复会话，并注入 Paperclip 技能（同时不污染自己的 Kimi 技能目录）的本地编程智能体，请使用此适配器。

Kimi Code 默认使用与 Claude、Codex 和 Gemini 相同的共享 ACP 引擎。你也可以将其切换到无头 CLI 路径。

---

## 适用场景

- 主机上已经运行 Kimi Code CLI。
- 需要通过 `-r <session_id>` 在多次心跳之间保持会话连续性。
- 希望 Paperclip 将技能注入 Kimi 技能目录，而不修改智能体工作区。

## 不适用场景

- 运行时位于 webhook 或远程 API 后面。请使用 [HTTP](./http.md) 或 [OpenClaw Gateway](./openclaw-gateway.md)。
- 只需要 shell 命令或脚本。请使用 [Process](./process.md)。
- 尚未安装 Kimi Code CLI，或 Paperclip 主机无法执行该 CLI。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `cwd` | 否 | 智能体进程的默认绝对工作目录。有权限时 Paperclip 会创建该路径；省略后回退到当前进程工作目录。 |
| `engine` | 否 | 运行 Kimi 的执行路径：适配器默认使用 ACP 引擎（`kimi acp`）；ACP 前置条件不可用时会因设置错误而失败。设置为 `acp` 或 `cli` 可指定执行路径。请参阅 [ACP 引擎](#acp-引擎)。 |
| `model` | 否 | Kimi 模型别名（`provider/model`）。默认为 `kimi-code/kimi-for-coding`。请参阅[模型](#模型)。 |
| `effort` | 否 | 思考力度（`low`、`medium`、`high`、`max`）。仅适用于 CLI 路径。请参阅[思考力度](#思考力度)。 |
| `promptTemplate` | 否 | 执行时使用的提示词模板。 |
| `instructionsFilePath` | 否 | 添加到执行提示词前面的 Markdown 指令文件绝对路径。本地运行时，同一目录中的其他文件（`HEARTBEAT.md`、`SOUL.md`、`TOOLS.md`）也会通过 `--add-dir` 开放读取。 |
| `command` | 否 | 默认为 `kimi`。 |
| `extraArgs` | 否 | 追加到 Kimi 调用中的额外 CLI 参数。 |
| `env` | 否 | 传递给运行时的 `KEY=VALUE` 环境变量。支持密钥引用。 |
| `timeoutSec` | 否 | 执行超时秒数。 |
| `graceSec` | 否 | 强制停止前的 SIGTERM 宽限时间，单位为秒。 |

> **注意：** 适配器会设置适用于无头执行的环境变量（`CI=1`、`NO_COLOR=1`、`KIMI_CODE_NO_AUTO_UPDATE=1`），避免无人值守执行等待交互式提示或更新预检。

---

## ACP 引擎

Kimi Code 有两种执行路径，由 `engine` 字段选择：

- **ACP（默认）。** Paperclip 通过 Agent Client Protocol（`kimi acp`）运行 Kimi，提供结构化实时执行记录——会话身份、带上下文窗口用量的状态、Assistant 和思考过程的令牌增量，以及会随进度更新并最终折叠为单张卡片的工具调用信息。Claude、Codex 和 Gemini 使用的也是同一套机制。
- **CLI（选择启用）。** 设置 `engine: "cli"` 使用无头 CLI 路径。此模式会通过 `kimi -p` 运行，并为非交互式执行传入 `--output-format stream-json`；提示词作为参数传递，而不是通过 stdin 传递。

要使用 ACP，请将 `engine` 留空或设置为 `engine: "acp"`——缺少前置条件时执行会因设置错误停止，而不会切换执行路径。要始终使用无头 CLI 路径，请设置 `engine: "cli"`。

---

## 模型

在 `model` 字段中选择 Kimi 模型别名。已知选项包括：

- `kimi-code/kimi-for-coding` — K2.8 Preview（适配器默认值）
- `kimi-code/kimi-for-coding-highspeed` — K2.7 Coding Highspeed
- `kimi-code/k3` — K3
- `kimi-code/k3-256k` — K3（256K）

你也可以输入不在列表中的模型别名；Paperclip 会将无法识别的值原样传递给 Kimi CLI。

---

## 思考力度

`effort` 字段会将 Paperclip 推理力度值映射为 Kimi 支持的思考力度。Kimi 提供 `low`、`high` 和 `max`——**没有 `medium` 级别**，因此 `medium` 会映射为 `high`。

仅在 **CLI 路径**（`engine: cli`）中应用思考力度，此时适配器会通过 `KIMI_MODEL_THINKING_EFFORT` 将其传给支持推理力度的模型——`kimi-code/kimi-for-coding`（K2.8 Preview）、`kimi-code/k3` 和 `kimi-code/k3-256k`。对于不声明支持推理力度的模型，该设置会被忽略；默认 ACP 引擎路径也不会传递此值。如果需要控制思考力度，请固定使用 `engine: "cli"`。

---

## 会话持久化

已保存会话的工作目录与当前 `cwd` 匹配时，Kimi 会通过 `-r <session_id>` 恢复会话。会话 ID 从末尾的 `session.resume_hint` 元事件中获取。

如果存在以下位置提示，会话编解码器也会保留它们：

- `cwd`
- `workspaceId`
- `repoUrl`
- `repoRef`

如果工作目录发生变化，适配器会启动新会话，而不是复用旧会话。

---

## 技能注入

本地运行时，Paperclip 会通过 `--skills-dir` 将所需技能指向每次运行专用的受管理目录，确保技能可靠加载，同时不污染你自己的 `~/.kimi-code/skills` 目录。远程运行时则会将技能同步到远程技能目录。

Kimi 技能目录默认为 `~/.kimi-code/skills`；设置 `KIMI_CODE_HOME` 后则为 `$KIMI_CODE_HOME/skills`。通过 CLI 手动安装 Paperclip 技能时，它们会被放入该目录。

---

## 身份验证

Kimi Code 支持以下三种身份验证方式：

- **`kimi login`**——OAuth 设备登录流程，与在 Paperclip 之外使用 Kimi Code 时相同。
- **在 Kimi 的 `config.toml` 中配置服务提供商。**
- **环境变量组合 `KIMI_MODEL_NAME` + `KIMI_MODEL_API_KEY`**——在适配器的 `env` 中设置后，可直接针对指定模型和密钥运行。

---

## 环境测试

**Test Environment**（测试环境）按钮会检查 Kimi Code CLI 是否已安装并可执行、工作目录是否可用，以及开始真实心跳前身份验证是否就绪。如果探测失败，请修正命令或身份验证路径。

---

## 示例

```json
{
  "adapterType": "kimi_local",
  "adapterConfig": {
    "cwd": "/Users/me/projects/paperclip-workspace",
    "model": "kimi-code/kimi-for-coding",
    "instructionsFilePath": "/Users/me/projects/paperclip-workspace/INSTRUCTIONS.md",
    "env": {
      "KIMI_MODEL_API_KEY": {
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

## 后续阅读

- [创建适配器](./creating-an-adapter.md)
- [适配器界面解析器契约](./adapter-ui-parser.md)
- [外部适配器](./external-adapters.md)
