---
paperclip_version: v2026.1005.0
seo_title: Gemini CLI 适配器
seo_description: 在 Paperclip 主机上运行 Google Gemini CLI，让会话跨心跳恢复，并为每个智能体配置沙盒。
---

# Gemini CLI

`gemini_local` 会在与 Paperclip 相同的计算机上运行 Google Gemini CLI。如果你需要支持会话恢复、可配置沙盒，并将 Paperclip 技能注入 Gemini 技能目录的本地智能体，请使用此适配器。

---

## 适用场景

- 主机上已安装 Gemini CLI。
- 希望智能体在多次心跳之间恢复同一个 Gemini 会话。
- 希望 Paperclip 在本地管理技能，同时不污染项目目录。
- 希望通过 API 密钥或 Gemini CLI 登录进行身份验证。

## 不适用场景

- 智能体运行在 webhook 或远程 API 后面。请使用 [HTTP](./http.md)。
- 只需要命令运行器或脚本。请使用 [Process](./process.md)。
- 尚未安装 Gemini CLI，或无法访问目标工作目录。

> **在 Docker 中运行？** 官方 Paperclip 镜像会预装 Gemini CLI 和其他本地 CLI，因此 `gemini_local` 可在容器中直接使用，无需手动安装。镜像还会设置 `GEMINI_SANDBOX=false`，以便在容器中安全使用。你仍需提供凭据——请参阅 [Docker](../deploy/docker.md)。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `cwd` | 否 | 智能体的绝对工作目录。实际使用中建议设置。省略后，适配器会回退到当前进程工作目录。有权限时 Paperclip 会创建该路径。 |
| `engine` | 否 | Gemini 的运行方式：`auto`（默认，运行 ACP）、`acp`（始终使用 Agent Client Protocol）或 `cli`（始终使用经典 Gemini CLI）。请参阅 [ACP 引擎](#acp-引擎)。 |
| `model` | 否 | Gemini 模型 ID。默认为 `auto`。常见选择包括 `gemini-3.1-pro-preview`、`gemini-3.8-flash`、`gemini-2.5-pro` 和 `gemini-2.5-flash`。请参阅[模型](#模型)。 |
| `promptTemplate` | 否 | 执行时使用的提示词模板。 |
| `instructionsFilePath` | 否 | 添加到提示词前面的 Markdown 指令文件。 |
| `sandbox` | 否 | 启用 Gemini 沙盒模式。否则适配器会传入 `--sandbox=none`。 |
| `yolo` | 否 | 无人值守批准模式的便捷开关。 |
| `approvalMode` | 否 | Gemini 批准模式的高级控制选项。 |
| `command` | 否 | 默认为 `gemini`。 |
| `extraArgs` | 否 | 追加到 Gemini 调用中的额外 CLI 参数。 |
| `env` | 否 | 传递给运行时的环境变量。支持密钥引用。 |
| `helloProbeTimeoutSec` | 否 | 就绪探测的超时时间。 |
| `timeoutSec` | 否 | 执行超时秒数。在本地和 SSH 目标上，`0` 表示适配器不设置墙钟超时。在沙盒目标上，`0` 或未设置时采用沙盒默认值 14,400 秒；使用正数覆盖该值，使用负数禁用适配器超时。 |
| `graceSec` | 否 | 强制停止前的宽限时间。 |

> **注意：** Gemini CLI 会在就绪检查中使用 `--output-format stream-json`；当已保存会话的 cwd 仍与当前 cwd 匹配时，会使用 `--resume` 恢复会话。对于非交互式执行，提示词通过 `--prompt` 传递（不会通过 stdin 传递）；适配器还会为 Gemini CLI 子进程设置适用于无头运行的终端和浏览器环境，避免无人值守执行等待浏览器身份验证或彩色终端提示而停滞。

---

## ACP 引擎

Gemini 可通过 ACP 或经典 Gemini CLI 两种引擎运行，由 `engine` 字段选择：

- **`auto`（默认）——ACP。** Paperclip 通过 Agent Client Protocol（ACP）运行 Gemini。如果主机不满足 ACP 前置条件，执行会失败并显示说明原因的设置错误——Paperclip 不会自行切换引擎。
- **`acp`——始终使用 ACP。** 强制使用 Agent Client Protocol 路径。
- **`cli`——始终使用 Gemini CLI。** 强制使用经典 CLI 包装器，完全跳过 ACP。

ACP 可提供更丰富的结构化实时执行记录：会话身份、带上下文窗口用量的状态、Assistant 和思考过程的令牌增量，以及会随进度更新并最终折叠为单张卡片的工具调用信息。你在查看沙盒执行的实时输出时，额外细节尤为有用。

当引擎解析为 ACP（`auto` 或 `acp`）时，以下额外字段适用：

| 字段 | 默认值 | 说明 |
|---|---|---|
| `agentCommand` | `gemini --acp` | 可选，用于覆盖 Gemini ACP 服务器命令。 |
| `mode` | `persistent` | `persistent` 会在多次执行之间保留 ACP 会话状态；`oneshot` 则每次运行都重新开始。 |
| `nonInteractivePermissions` | `deny` | ACP 智能体在交互式会话外请求输入时如何处理——拒绝请求（`deny`）或令执行失败（`fail`）。 |
| `stateDir` | 由 Paperclip 管理 | 可选的 ACP 会话状态目录。默认位于 Paperclip 按公司和智能体划分的存储目录中。 |
| `warmHandleIdleMs` | `0` | 两次运行之间保持 ACP 进程热运行的时间，单位为毫秒。`0` 表示每次运行后关闭进程，但仍保留持久会话状态。 |

> **请注意：** 旧版独立 `acpx_local` 适配器的能力现在由 ACP 提供。该适配器已退役——请选择 `gemini_local`（或 `claude_local` / `codex_local`），并将 `engine` 保持为 `auto`，即可默认使用 ACP。

### 在沙盒环境中使用 ACP

如果智能体在 Paperclip 沙盒环境中运行，可以将 `engine` 保持为 `auto`。如果沙盒提供 Paperclip 双向进程会话，Paperclip 就会继续使用 ACP 引擎及其结构化实时执行记录；无需在适配器配置中添加单独的桥接设置。

只运行一次性命令的环境无法承载 ACP 会话，因此此类环境中的 `auto` 执行会失败并显示设置错误，而不会切换到 Gemini CLI。如果你希望使用 CLI 路径（例如在此类环境中），请设置 `engine: "cli"`。

---

## 模型

在 `model` 字段中选择 Gemini 模型 ID。可选项包括：

- `auto`（适配器默认值——由 Gemini CLI 选择模型）
- `gemini-3.1-pro-preview`
- `gemini-3.1-pro-preview-customtools`
- `gemini-3.8-flash`
- `gemini-3.7-flash`
- `gemini-3.6-flash`
- `gemini-3.5-flash`
- `gemini-3.5-flash-lite`
- `gemini-3.1-flash-lite`
- `gemini-3-flash-preview`
- `gemini-2.5-pro`
- `gemini-2.5-flash`
- `gemini-2.5-flash-lite`

列表中已移除较旧的 `gemini-2.0-flash` 和 `gemini-2.0-flash-lite`。你仍可以输入 Gemini CLI 接受的任何模型 ID。

---

## 会话持久化

适配器会在多次心跳之间保存 Gemini 会话 ID，并在下次唤醒时恢复会话。

如果工作目录发生变化，适配器会启动新会话，而不是尝试恢复旧会话。

如果 Gemini 报告未知会话错误，Paperclip 会自动使用新会话重试。

如果恢复的会话超出模型令牌预算（Gemini 会报告输入超出最大令牌数），适配器会将旧会话视为无法恢复，并改用新会话自动重试一次，而不是让执行失败。长期运行的智能体在对话历史最终超出上下文窗口时，可能会遇到这种情况。

---

## 无头身份验证

Gemini CLI 要求在 `~/.gemini/settings.json` 中持久化一种身份验证方法，否则会拒绝非交互式执行；单独设置环境变量 `GEMINI_DEFAULT_AUTH_TYPE` 并不满足此要求。为避免无头执行停在交互式身份验证方式提示上，当适配器管理运行时主目录且检测到 API 密钥（`GEMINI_API_KEY` 或 `GOOGLE_API_KEY`）时，会为你预先选择 API 密钥身份验证方式。它会在 `settings.json` 中写入 `selectedAuthType: "gemini-api-key"`，并且不会修改已有的 `settings.json`。

这种预选仅适用于由 Paperclip 配置受管理主目录的运行（远程执行目标）。普通本地运行使用你真实的主目录，其中已有 Gemini 身份验证配置。

## 技能注入

Gemini CLI 会将 Paperclip 技能符号链接到 `~/.gemini/skills`。

适配器不会覆盖现有用户技能，只会公开此次执行需要的 Paperclip 管理技能。

---

## 环境测试

**Test Environment**（测试环境）按钮会检查：

- Gemini CLI 已安装且可执行。
- 工作目录是可用的绝对路径。
- 可通过 `GEMINI_API_KEY`、`GOOGLE_API_KEY`、Google 账户登录或 Gemini CLI 身份验证完成身份验证。
- hello 探测可以运行 `gemini --output-format json "Respond with hello."`。

除非你设置了 `approvalMode` 或关闭 `yolo`，否则 hello 探测会使用真实执行时相同的无人值守 `yolo` 批准模式运行。

测试还会检测身份验证和配额错误，因此安装检查通过并不自动意味着该账户仍可执行任务。

---

## 示例

```json
{
  "adapterType": "gemini_local",
  "adapterConfig": {
    "cwd": "/Users/me/projects/paperclip-workspace",
    "model": "gemini-2.5-pro",
    "instructionsFilePath": "/Users/me/projects/paperclip-workspace/INSTRUCTIONS.md",
    "promptTemplate": "You are the product engineer for this company. Stay focused on the task.",
    "sandbox": false,
    "yolo": true,
    "env": {
      "GEMINI_API_KEY": {
        "type": "secret_ref",
        "secretId": "secret-id",
        "version": "latest"
      }
    },
    "helloProbeTimeoutSec": 10,
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
