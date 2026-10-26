---
paperclip_version: v2026.1023.0
seo_title: Grok Local 适配器
seo_description: 在 Paperclip 主机上将 xAI Grok Build CLI 作为本地编程智能体运行，并在每次心跳间恢复同一个 Grok 会话。
---

# Grok Local

`grok_local` 会在与 Paperclip 相同的计算机上运行 xAI Grok Build CLI。如果你需要能在多次心跳之间恢复同一 Grok 会话的本地编程智能体，并希望 Paperclip 自动将技能暂存到 Grok 原生发现路径，请使用此适配器。

---

## 适用场景

- 运行 Paperclip 的计算机上已安装并登录 Grok CLI。
- 需要通过 `--resume` 在多次心跳之间恢复会话的本地编程智能体。
- 希望 Paperclip 将受管理的指令和技能暂存到执行工作区，并使用 Grok 的原生发现路径（`Agents.md` 和 `.claude/skills`）。

## 不适用场景

- 智能体运行在 webhook 或远程端点后面。请改用 [HTTP](./http.md) 或 [OpenClaw Gateway](./openclaw-gateway.md)。
- 只需要运行一次性脚本，而不需要编程智能体循环。请使用 [Process](./process.md)。
- 主机尚未安装 Grok CLI，或尚未登录。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `cwd` | 否 | 智能体进程的默认绝对工作目录。有权限时 Paperclip 会创建此路径。 |
| `instructionsFilePath` | 否 | Markdown 指令文件的绝对路径（通常为 `AGENTS.md`）。安全时，Paperclip 会将其暂存到执行工作区并命名为 `Agents.md`；否则会回退到 `--rules @file`。 |
| `promptTemplate` | 否 | 执行时使用的提示词模板。 |
| `model` | 否 | Grok 模型 ID。默认为 `grok-build`。请参阅[模型](#模型)。 |
| `permissionMode` | 否 | 通过 `--permission-mode` 传递的 Grok 权限模式。**无默认值**——留空时 Paperclip 完全不传递权限模式标记。（Grok 1.0+ 会将 `dontAsk` 设为默认拒绝模式，且该设置会覆盖 `--always-approve`，因此强制使用它会破坏无人值守执行；除非有明确理由，否则请留空。） |
| `alwaysApprove` | 否 | 添加 `--always-approve`，使无人值守执行不会因提示而停滞。默认为 `true`；这才是无人值守执行策略，而不是权限模式。 |
| `disableWebSearch` | 否 | 传入 `--disable-web-search`，使执行不会调用 Grok 网页搜索。默认为 `true`。 |
| `reasoningEffort` | 否 | 通过 `--reasoning-effort` 传递的 Grok 推理力度：`low`、`medium` 或 `high`。`grok-4.7` 和 `grok-4.6` 还接受 `xhigh`。 |
| `maxTurns` | 否 | 一次执行的最大智能体轮次数。 |
| `command` | 否 | 默认为 `grok`。只有在主机其他位置安装 Grok 时才覆盖此值。 |
| `extraArgs` | 否 | 追加到 Grok 调用中的额外 CLI 参数。 |
| `env` | 否 | 传递给 Grok 的环境变量。支持密钥引用。 |
| `timeoutSec` | 否 | 执行超时秒数。 |
| `graceSec` | 否 | SIGTERM 宽限时间，单位为秒。 |

> **注意：** 执行会使用 `grok --single` 和 `--output-format streaming-json`。流式推理通道现在会在不同思考内容之间保留换行，因此实时 Working 面板不会再把内容连成一长串文本。

---

## 模型

模型选择器提供以下 Grok 模型 ID：

- `grok-build`（适配器默认值）
- `grok-4.7`
- `grok-4.6`
- `grok-4.5`

智能体表单中的推理力度选项取决于所选模型。如果切换到不支持当前推理力度的模型（例如从 `grok-4.7` 的 `xhigh` 切换到 `grok-4.5`），表单会清除此值并回退到 Auto。

---

## 会话持久化

Grok Local 会保存 Grok 会话 ID，并在下次心跳时通过 `--resume <sessionId>` 恢复——但只有已保存会话的 `cwd` 与当前 `cwd` 匹配时才会恢复。如果目录已经移动，适配器会启动新会话。

会话编解码器会保留与其他本地适配器相同的位置提示：

- `cwd`
- `workspaceId`
- `repoUrl`
- `repoRef`

> **提示：** 保存适配器之前，可以在主机上运行 `grok models`，确认已完成身份验证并查看可用模型。

---

## 身份验证

Grok Local 支持两种身份验证模式，具体模式仅取决于执行环境中是否存在 `XAI_API_KEY`：

- **API 密钥。** 设置 `XAI_API_KEY`（通常在 `env` 中以密钥引用方式提供），适配器就会使用该密钥运行。此方式按用量计费，Paperclip 会显示 xAI 报告的单次运行费用。
- **订阅（SuperGrok）。** 不设置 `XAI_API_KEY` 时，Grok 会使用已登录账户完成身份验证。订阅方式不会产生单次运行的美元费用。

订阅执行会使用哪个登录凭据，取决于智能体的运行位置：

- **本地执行**会使用主机自身的 `grok login`（保存在 `~/.grok`），直到公司 Grok 主目录中存在可用的 `auth.json`（例如沙盒设备登录后生成的文件）。有了公司凭据后，Paperclip 会将执行指向该公司的 Grok 主目录（`GROK_HOME`），避免不同公司的登录信息互相共享。如果你已自行设置 `GROK_HOME`（无论是在主机环境还是智能体 `env` 中），且公司登录信息不可用，Paperclip 会保留你设置的值。
- **沙盒和远程执行**始终使用公司的 Grok 主目录，绝不会回退到主机登录信息。
- **受管理的 AI 连接**会使用连接指定的 Grok 主目录。

> **提示：** 如果主机上有你不希望智能体继承的 `XAI_API_KEY`，可以在该智能体的 `env` 中将 `XAI_API_KEY` 设为空值。此空值覆盖会清除继承的密钥，并切换到订阅身份验证。

### 使用订阅账户登录

如果智能体运行在没有可用 Grok 登录凭据的 Paperclip 沙盒环境中，[环境测试](#环境测试)会报告缺少身份验证；Paperclip 也可以为你启动交互式设备登录。Grok 会输出一个 `https://accounts.x.ai/oauth2/device` URL 和一个简短的一次性代码；打开 URL 并确认代码后，Paperclip 会将生成的凭据保存到该公司的 Grok 主目录，供之后的心跳复用。在本地或 SSH 主机上，也可以直接在该计算机上运行 `grok login`。

---

## 环境测试

**Test Environment**（测试环境）按钮会检查工作目录是否可用、`grok` 命令是否可执行，然后运行 `grok models` 和一行 hello 探测。测试会报告 Grok 是否已通过身份验证、列出发现的模型，并在你配置的 `model` 不在列表中时发出警告。

如果 Grok 尚未登录，测试会发出警告而不是失败；在沙盒环境中，它还会提供启动上文所述设备登录的选项。

---

## 技能注入

Paperclip 会将为智能体启用的运行时技能暂存到执行工作区中的 `.claude/skills`。Grok 会自动将它们识别为项目技能，无需额外配置。

如果你提供 `instructionsFilePath`，Paperclip 会优先将其暂存到工作区并命名为 `Agents.md`，以便 Grok 原生加载。如果这样做不安全（例如工作区中已有不属于 Paperclip 的 `Agents.md`），适配器就会回退到 `--rules @file`。

---

## 示例

```json
{
  "adapterType": "grok_local",
  "adapterConfig": {
    "cwd": "/Users/me/projects/paperclip-workspace",
    "model": "grok-build",
    "reasoningEffort": "medium",
    "instructionsFilePath": "/Users/me/projects/paperclip-workspace/AGENTS.md",
    "env": {
      "XAI_API_KEY": {
        "type": "secret_ref",
        "secretId": "secret-id",
        "version": "latest"
      }
    },
    "timeoutSec": 300,
    "graceSec": 15,
    "maxTurns": 200
  }
}
```

---

<a id="grok-build-on-paperclip-runner"></a>
## 在 Paperclip Runner 上运行 Grok Build

`grok_local` 并非运行 Grok 的唯一方式。Grok Build 也可通过 **Paperclip Runner**（`paperclip_runner`）运行，这是 Paperclip 的实验性原生运行器；它通过 Agent Client Protocol（ACP）与 Grok 通信，而不是包装 CLI。如果你需要 Runner 提供的持久会话（即服务器重启后执行仍能继续，并恢复同一 Grok 对话）以及内置 Paperclip 工具，请选择此方式。

现有 `grok_local` 智能体会继续使用 `grok_local` 运行。它们不会自动迁移；是否切换由你为每个智能体单独决定。

### 设置

1. 确认已在 Experimental 设置中启用 **Paperclip Runner**。此功能在自托管实例上默认开启，在由 Cloud 管理的实例上默认关闭。
2. 创建或编辑智能体，并将 **Paperclip Runner** 选为适配器。
3. 将 **Harness** 设为 **Grok Build**。

保存的配置如下。新的 Grok Runner 智能体默认使用 `grok-4.7` 模型：

```json
{
  "adapterType": "paperclip_runner",
  "adapterConfig": {
    "provider": "acpx",
    "acpxAgent": "grok",
    "model": "grok-4.7"
  }
}
```

### 在智能体运行位置安装 Grok Build

Runner 不会替你下载 Grok，也不会使用 `PATH` 上找到的 `grok`。它要求执行环境在以下确切路径中安装 Grok Build 1.0.13：

```
/opt/paperclip/providers/grok/1.0.13/grok
```

- 由 Paperclip 标准 Runner 镜像创建的 **Daytona 沙盒**已包含此程序。
- **自定义沙盒镜像和本地主机**需要你自行安装。

Grok 启动前，Runner 会检查该二进制文件的校验和。如果文件缺失或不匹配，执行会停止并报告所需路径和版本。仅支持 Linux x64 和 macOS ARM64；Intel Mac 执行目标会被拒绝。**Test Environment** 会确认运行时已安装并通过验证；模型访问权限则会在智能体实际运行时检查。

### 登录

Runner 上的 Grok 使用公司的 Grok 连接。通常使用订阅登录。如果你明确选择公司的 `XAI_API_KEY` 密钥，则会改为按用量收取 API 费用。订阅登录不会自动回退到 API 密钥，因此订阅过期时不会悄悄开始产生费用。远程执行绝不会借用主机自己的 Grok 登录信息。

### 权限

Grok 的 Runner **ACPX permission mode**（ACPX 权限模式，`acpxPermissionMode`）默认为 **Full auto (approve all)**（完全自动，全部批准，`approve-all`），因此智能体无需其他设置即可无人值守工作。完全自动仍受公司权限、受管理的审批流程和执行环境边界约束。

如果你选择了限制更严格的 **Automatic Paperclip actions**（自动执行 Paperclip 操作，`approve-paperclip`）或 **Allow Paperclip reads**（允许读取 Paperclip 数据，`approve-reads`），这些模式会保留；不过在这些模式下 Grok 无法自动批准 Paperclip 工具调用——调用会暂停并等待审批。**Deny all**（全部拒绝，`deny-all`）会直接拒绝 Grok 的权限请求。

### 原生路径限制

Grok 原生 Runner 的用量统计尚未通过资格验证。缺少令牌或美元费用总数并不代表执行免费；上文的计费说明针对的是 `grok_local` 和所选身份验证方式。原生会话加载支持恢复执行，但不支持实时引导，后续消息会通过控制器队列处理。

支持的任务模式和接入要求请参阅 [Paperclip Runner](./paperclip-runner.md)。

---

## 后续阅读

- [Paperclip Runner](./paperclip-runner.md)
- [Grok 连接](../../connectors/xai.md)
- [适配器概览](./overview.md)
- [创建适配器](./creating-an-adapter.md)
- [适配器界面解析器契约](./adapter-ui-parser.md)
