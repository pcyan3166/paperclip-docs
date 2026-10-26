---
paperclip_version: v2026.1012.0
seo_title: Claude Code 适配器
seo_description: 在 Paperclip 主机上运行 Anthropic Claude Code CLI，并由智能体配置管理会话持久化、技能注入和运行参数。
---

# Claude Code

`claude_local` 会在与 Paperclip 相同的计算机上运行 Anthropic Claude Code CLI。如果你需要本地编程智能体，并希望持久化会话、注入技能且能够完整访问已配置的工作目录，请使用此适配器。

---

## 适用场景

- 你已经在主机上使用 Claude Code。
- 需要本地智能体读取和写入工作目录中的文件。
- 希望 Paperclip 在多次心跳之间恢复同一个 Claude 会话。
- 希望适配器自动将 Paperclip 技能同步到 Claude 的技能路径。

## 不适用场景

- 智能体运行在另一台机器上，或通过 webhook 调用。请改用 [HTTP](./http.md)。
- 只需要运行一次性的脚本或命令。请使用 [Process](./process.md)。
- 尚未安装 Claude Code，或无法通过 `PATH` 找到它。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `cwd` | 否 | 智能体的绝对工作目录。实际使用中建议设置。省略后，适配器会回退到当前进程工作目录。有权限时 Paperclip 会创建此路径。 |
| `engine` | 否 | Claude Code 的运行方式：`auto`（默认，运行 ACP）、`acp`（始终使用 Agent Client Protocol）或 `cli`（始终使用经典 Claude CLI）。请参阅 [ACP 引擎](#acp-引擎)。 |
| `model` | 否 | Claude 模型 ID。常见选择包括 `claude-opus-5-5`、`claude-sonnet-5-5`、`claude-opus-4-8`、`claude-sonnet-5` 和 `claude-fable-5-1`。 |
| `promptTemplate` | 否 | 执行时使用的提示词模板。 |
| `env` | 否 | 传递给 Claude Code 的环境变量。支持密钥引用。 |
| `command` | 否 | 默认为 `claude`。只有需要使用不同的可执行文件路径时才覆盖此值。 |
| `extraArgs` | 否 | 追加到 Claude 调用中的额外 CLI 参数。 |
| `effort` | 否 | 通过 `--effort` 传递的推理力度。可选值取决于模型，请参阅[推理力度](#推理力度)。如果沙盒环境中的 Claude CLI 太旧，无法提供 `--effort` 参数，Paperclip 会移除此参数并提醒你升级环境中的 Claude Code，以恢复推理力度控制。 |
| `chrome` | 否 | 启用时传入 `--chrome`。 |
| `maxTurnsPerRun` | 否 | 限制每次心跳中的智能体轮次数，默认为 `300`。 |
| `dangerouslySkipPermissions` | 否 | 默认为 `true`，因为 Paperclip 会以无头 `--print` 模式运行 Claude。开启后，本地和远程运行都会使用 `--dangerously-skip-permissions`，涵盖内置工具和已连接的工具。请参阅[权限](#权限)。 |
| `timeoutSec` | 否 | 执行超时秒数。在本地和 SSH 目标上，`0` 表示适配器不设置墙钟超时。在沙盒目标上，`0` 或未设置时采用沙盒默认值 14,400 秒；使用正数覆盖该值，使用负数禁用适配器超时。 |
| `graceSec` | 否 | 强制停止前的宽限时间。 |
| `workspaceStrategy` | 否 | 执行工作区策略，例如 `git_worktree`。 |
| `workspaceRuntime` | 否 | 保留供工作区运行时使用的元数据。 |

> **注意：** Claude Code 是无头适配器。与普通 CLI 会话相比，环境测试在这里更为重要，因为 Paperclip 必须确认命令、路径、身份验证模式和模型可以协同工作。

---

<a id="permissions"></a>
## 权限

Paperclip 会在没有人监看的情况下运行 Claude Code，因此遇到权限提示时没有人可以点击“批准”。默认开启 `dangerouslySkipPermissions` 后，Claude 会以全自动模式运行：它可以使用内置工具和来自已连接服务（例如 MCP 服务器）的所有工具，无需暂停并请求许可。

无论智能体运行在 Paperclip 主机还是远程目标上，行为都相同。过去远程运行会使用经过挑选的允许工具列表，而不是完全跳过权限检查；该列表会漏掉已连接的工具以及 Claude 在后续版本中新增的工具，因此现在远程运行和本地运行一样会跳过所有权限检查。

需要了解以下几点：

- **以 root 身份运行的沙盒。** 除非 Claude 确认自己运行在沙盒中，否则它会拒绝完全跳过权限检查。智能体在 Paperclip 管理的沙盒中运行时，Paperclip 会设置 `IS_SANDBOX=1`，使 Claude 在沙盒中接受跳过权限检查。
- **其他 root 进程。** 在受管理沙盒之外，请以非 root 用户运行 Claude。Paperclip 不会悄悄回退到权限更窄的模式——如果 Claude 拒绝启动，执行就会失败，以便你修复设置。

> **提示：** 如果希望 Claude 遇到通常会请求许可的操作时停下来并拒绝，请将 `dangerouslySkipPermissions` 设为 `false`。请记住，在无头执行中，需要批准的工具不会运行。

---

<a id="reasoning-effort"></a>
## 推理力度

智能体表单中的 `effort` 可选值取决于你选择的模型：

| 模型 | 推理力度级别 |
|---|---|
| `claude-opus-5-5`、`claude-opus-5`、`claude-opus-4-8`、`claude-opus-4-7`、`claude-sonnet-5-5`、`claude-sonnet-5`、`claude-fable-5-1`、`claude-fable-5` | `low`、`medium`、`high`、`xhigh`、`max` |
| `claude-opus-4-6`、`claude-sonnet-4-6` | `low`、`medium`、`high`、`max` |
| `claude-haiku-4-5` | 不支持推理力度控制 |
| 其他模型 | `low`、`medium`、`high` |

这些级别同样适用于相应模型的 Bedrock 版本。如果切换到不支持当前所选推理力度的模型，表单会清空该设置，使其回退到 Auto（自动）。

---

## ACP 引擎

Claude Code 可以通过 ACP 或经典 Claude CLI 两种引擎运行，由 `engine` 字段选择：

- **`auto`（默认）——ACP。** Paperclip 通过 Agent Client Protocol（ACP）运行 Claude。如果主机不满足 ACP 前置条件，执行会失败并显示说明原因的设置错误——Paperclip 不会自行切换引擎。
- **`acp`——始终使用 ACP。** 强制使用 Agent Client Protocol 路径。
- **`cli`——始终使用 Claude CLI。** 强制使用经典 CLI 包装器，完全跳过 ACP。

ACP 可提供更丰富的结构化实时执行记录：会话身份、带上下文窗口用量的状态、Assistant 和思考过程的令牌增量，以及会随进度更新并最终折叠为单张卡片的工具调用信息。你在查看沙盒执行的实时输出时，额外细节尤为有用。

当引擎解析为 ACP（`auto` 或 `acp`）时，以下额外字段适用：

| 字段 | 默认值 | 说明 |
|---|---|---|
| `agentCommand` | 软件包本地的 `claude-agent-acp` | 可选，用于覆盖 Claude ACP 服务器命令。 |
| `mode` | `persistent` | `persistent` 会在多次执行之间保留 ACP 会话状态；`oneshot` 则每次运行都重新开始。 |
| `nonInteractivePermissions` | `deny` | ACP 智能体在交互式会话外请求输入时如何处理——拒绝请求（`deny`）或令执行失败（`fail`）。 |
| `stateDir` | 由 Paperclip 管理 | 可选的 ACP 会话状态目录。默认位于 Paperclip 按公司和智能体划分的存储目录中。 |
| `warmHandleIdleMs` | `0` | 两次运行之间保持 ACP 进程热运行的时间，单位为毫秒。`0` 表示每次运行后关闭进程，但仍保留持久会话状态。 |

> **请注意：** 旧版独立 `acpx_local` 适配器的能力现在由 ACP 提供。该适配器已退役——请选择 `claude_local`（或 `codex_local` / `gemini_local`），并将 `engine` 保持为 `auto`，即可默认使用 ACP。

### 在沙盒环境中使用 ACP

如果智能体在 Paperclip 沙盒环境中运行，可以将 `engine` 保持为 `auto`。如果沙盒提供 Paperclip 双向进程会话，Paperclip 就会继续使用 ACP 引擎及其结构化实时执行记录；无需在适配器配置中添加单独的桥接设置。

只运行一次性命令的环境无法承载 ACP 会话，因此此类环境中的 `auto` 执行会失败并显示设置错误，而不会切换到 Claude CLI。如果你希望使用 CLI 路径（例如在此类环境中），请设置 `engine: "cli"`。

---

<a id="model-discovery"></a>
## 模型发现

在智能体配置表单中选择模型时，Claude Code 会通过实时请求 Anthropic API 填充模型下拉列表，而不是使用硬编码列表。因此，上次更新 Paperclip 之后发布的 Claude 模型，也可以立即显示，无需等待 Paperclip 发布新版本。

模型列表生成方式如下：

- **提供了 API 密钥时。** 如果设置了 `ANTHROPIC_API_KEY`，适配器会调用 Anthropic 模型端点（`/v1/models`）。如果你设置了 `ANTHROPIC_BASE_URL`，则使用该地址；否则使用 `https://api.anthropic.com`。适配器会提供 API 返回的所有模型。实时结果会与 Paperclip 内置列表合并并去重，因此你始终能看到至少一组已知可用模型，并且还能看到账户中的新模型。
- **使用 Bedrock 时。** 如果适配器检测到 AWS Bedrock（例如 `CLAUDE_CODE_USE_BEDROCK=1`），它会改为提供带区域前缀的 Bedrock 模型 ID，例如 `us.anthropic.claude-opus-5-5`、`us.anthropic.claude-sonnet-5-5`、`us.anthropic.claude-sonnet-5` 和 `us.anthropic.claude-fable-5-1`。
- **没有密钥或查询失败时。** 如果没有 API 密钥，或请求超时、返回空结果，则只使用 Paperclip 的内置回退列表。模型发现功能不会阻止你保存适配器。

无论采用哪种方式生成列表，下拉菜单都会按照 Claude 应用中的相同顺序显示模型，方便你在预期位置找到所需模型。各系列最新版本优先，并按能力从高到低排列——Fable、Mythos、Opus、Sonnet，然后是 Haiku——接着按系列分组列出较旧版本，每组中新版本优先。模型别名会排在带日期的快照之前；看起来不像 Claude 模型的 ID 会排在列表末尾。

发现的模型会缓存约一分钟（缓存键取决于当前使用的 API 密钥和基础 URL），因此重新打开表单时会立即显示。如果需要最新列表（例如刚获得某个新模型的访问权限），可以使用模型字段的 **refresh**（刷新）控件，绕过缓存重新查询。

> **提示：** `model` 字段仍接受你输入的任何模型 ID。模型发现是为了让你无需记住准确标识符，而不是限制你只能选择列表中的模型。

> **请注意：** 某些较新的模型在 CLI 路径上需要最低 Claude Code 版本：
>
> - Claude Opus 5.5（`claude-opus-5-5`，或 Bedrock 上的 `us.anthropic.claude-opus-5-5`）需要 Claude Code `2.1.280` 或更高版本。
> - Claude Sonnet 5.5（`claude-sonnet-5-5`，或 Bedrock 上的 `us.anthropic.claude-sonnet-5-5`）需要 Claude Code `2.1.284` 或更高版本。
> - Claude Fable 5.1（`claude-fable-5-1`，或 Bedrock 上的 `us.anthropic.claude-fable-5-1`）需要 Claude Code `2.1.251` 或更高版本。
>
> 如果已安装的 CLI 版本较旧，环境测试和执行都会立即因 `claude_cli_version_incompatible` 失败，而不会使用不受支持的二进制文件启动。若要使用该模型，请升级目标主机上的 Claude Code。

---

## 会话持久化

Claude Code 会保存 Claude Code 会话 ID；下次心跳时，如果工作目录仍然匹配，就会恢复该会话。

如果适配器无法恢复先前会话，会自动回退并启动新会话。

会话编解码器还会保留 Claude 会话状态中的重要位置提示，包括：

- `cwd`
- `workspaceId`
- `repoUrl`
- `repoRef`

> **提示：** 如果你在两次心跳之间移动工作目录，Claude Code 会启动新会话，而不会尝试复用旧会话。

### 恢复会话工作区

Paperclip 恢复 `claude_local` 会话时，保存的 `cwd` 是**主机工作区 cwd**——也就是运行 Paperclip 的计算机上的路径——而不是远程沙盒报告的 cwd。这样智能体在远程沙盒中执行时，恢复路径仍然稳定。

心跳信任已保存的 cwd 之前，`isUnsafeSessionWorkspaceCwd` 会检查该路径是否指向少量系统根目录：`/`、`/tmp`、`/var`、`/var/tmp`、`/var/run`、`/usr`、`/etc`、`/proc`、`/sys`、`/dev`、`/run`、`/private` 和 `/private/tmp`。如果保存的 cwd 解析到这些位置之一，Paperclip 就会拒绝该路径，改用智能体主目录工作区，避免智能体在系统目录中任意操作。

工作区恢复还会严格限制复制内容。在 `captureDirectorySnapshot` 期间，所有非目录、非符号链接且非普通文件的内容——包括套接字、FIFO、字符设备、块设备和其他非文件条目——都会跳过。因此，恢复工作区时不会再被意外遗留的设备节点阻断。

最后，声明了 `environment.drivers.register` 能力的插件现在只能从适配器环境中收到少量允许的模型服务提供商 API 密钥，而不是完整环境变量集。驱动程序插件仍能获得与 Anthropic 等服务提供商通信所需的信息，而其他无关密钥则留在主机上。

---

## 技能注入

Claude Code 会创建临时符号链接目录，使 Paperclip 技能可供 Claude 使用，并通过 `--add-dir` 将其传递给 Claude。

如果要在 Paperclip 之外手动使用本地 CLI，请运行：

```sh
pnpm paperclipai agent local-cli claudecoder --company-id <company-id>
```

该命令会将技能安装到 `~/.claude/skills`、创建智能体 API 密钥，并打印运行 Claude 时需要使用的 shell 导出语句，以便 Claude 以该智能体身份运行。

---

## 环境测试

界面中的 **Test Environment** 按钮会在保存或运行适配器前验证 Claude Code。此测试会检查：

- Claude Code 已安装且可执行。
- 工作目录是可用的绝对路径。
- 已通过 `ANTHROPIC_API_KEY`、Bedrock 设置或 Claude 订阅登录配置身份验证。
- hello 探测可以通过 `claude --print - --output-format stream-json --verbose` 命令运行提示词 `Respond with hello.`。

如果测试失败，请先修复命令、路径或身份验证配置，再次尝试。

---

<a id="example"></a>
## 示例

```json
{
  "adapterType": "claude_local",
  "adapterConfig": {
    "cwd": "/Users/me/projects/paperclip-workspace",
    "model": "claude-sonnet-4-6",
    "promptTemplate": "You are the engineering lead. Work carefully and report progress.",
    "env": {
      "ANTHROPIC_API_KEY": {
        "type": "secret_ref",
        "secretId": "secret-id",
        "version": "latest"
      }
    },
    "timeoutSec": 300,
    "graceSec": 15,
    "maxTurnsPerRun": 300,
    "dangerouslySkipPermissions": true
  }
}
```

---

## 后续阅读

- [创建适配器](./creating-an-adapter.md)
- [适配器界面解析器契约](./adapter-ui-parser.md)
- [外部适配器](./external-adapters.md)
