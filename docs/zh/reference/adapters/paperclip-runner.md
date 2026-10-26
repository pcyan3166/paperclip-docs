---
paperclip_version: v2026.1023.0
seo_title: Paperclip Runner 设置与服务提供商
seo_description: 配置 Paperclip Runner 以运行原生智能体会话。选择受支持的服务提供商、设置权限和模型访问，并了解运行时限制。
---

# Paperclip Runner

如果你需要 Paperclip 原生运行生命周期、持久会话恢复和内置任务工具，请使用 `paperclip_runner`。服务提供商与适配器分开选择，然后再配置模型访问权限和执行环境。

Paperclip Runner 目前处于实验阶段。对于自托管实例，`enableNativeRunner` 默认**开启**；对于由 Cloud 管理的实例，默认**关闭**。部署设置可以覆盖这些默认值。启用此开关不会转换现有智能体：引导流程和直接适配器仍使用各自的执行路径。OpenAI Dot 有独立开关，参见 [OpenAI Dot](#openai-dot)。

## 开始之前

- 在 **Settings → Experimental** 中启用了 **Paperclip Runner** 的实例。
- 一个处于活动状态的智能体，以及一个使用标准、规划或询问工作模式的任务。
- 已准备好的执行环境和兼容的模型凭据。
- 所选服务提供商要求的固定运行时版本。

进行源码开发时，启用 Runner 需要 Rust 工具链或 `PAPERCLIP_RUNNER_BINARY`；开发启动器会在需要时构建守护进程。已发布的发行包和执行镜像会提供各自的运行时资源。

## 选择服务提供商

| 服务提供商 | 运行时与设置 |
| --- | --- |
| **Codex** (`codex`) | 原生 Codex app-server 路由，固定使用 Codex `0.160.0`。配置 OpenAI 连接或其他兼容的 Responses 路由。 |
| **OpenCode** (`opencode`) | 原生 OpenCode server 路由，固定使用 `1.18.34`。使用 `provider/model` 格式的模型。 |
| **ACP 智能体 → Claude** (`acpx`, `claude`) | 经过资格验证的 Claude ACP 运行时：Claude Agent SDK `0.3.286`，该 SDK 运行 Claude Code `2.1.286`。配置兼容 Claude 的 AI 连接。 |
| **ACP 智能体 → Grok Build** (`acpx`, `grok`) | 经过资格验证的 Grok Build 运行时。参见 [Grok 设置](grok-local.md#grok-build-on-paperclip-runner)。 |
| **ACP 智能体 → Cursor** (`acpx`, `cursor`) | 固定版本的 Cursor 运行时，需明确指定模型和 Agent、Plan 或 Ask 模式。参见 [Cursor 设置](cursor-local.md#cursor-on-paperclip-runner)。 |
| **ACP 智能体 → Pi** (`acpx`, `pi`) | 经过资格验证的 Pi 1.0 运行时。输入明确的 `provider/model` ID；Pi 使用绑定到智能体环境的公司凭据，例如 `OPENROUTER_API_KEY`、`OPENAI_API_KEY`、`ANTHROPIC_API_KEY` 或 `GEMINI_API_KEY`。参见 [Paperclip Runner 上的 Pi](#pi-on-paperclip-runner)。 |
| **Claude Managed** (`claude_managed`) | 符合公司资格要求的托管智能体配置文件、数据保留确认和会话支出上限。 |
| **AWS AgentCore** (`aws_agentcore`) | 符合公司资格要求的远程智能体配置文件、数据保留确认和预估会话支出上限。 |

GitHub Copilot 会作为资格待确认的 ACP 选项出现，并且在生产运行中被禁用。**Harness** 列表也直接提供 **Grok Build**，选择它会保存相同的 `acpx` 服务提供商，并设置 `acpxAgent: "grok"`。选择任意可执行文件不会创建符合资格要求的配置文件。保存时，历史 ACP Codex 设置会规范化为原生 Codex 服务提供商。

托管服务提供商需要由运维人员预先准备配置文件；不能在服务提供商选择器中粘贴通用模型 API 密钥来完成配置。参见[托管和远程智能体配置文件](../api/agents.md#managed-and-remote-agent-profiles)。

## 配置智能体

1. 创建或编辑智能体，并选择 **Paperclip Runner**。
2. 选择 **Harness**。如果选择 **ACP 智能体**，还要选择 **ACP agent**。
3. 选择模型。编辑已保存的智能体时，为 Codex、OpenCode、Claude 或 Grok 选择兼容的 **Connection**。Cursor 和 Pi 则使用其执行环境中由密钥提供的凭据。
4. 选择执行环境并检查权限。
5. 保存，然后在分配工作前使用 **Test Environment**。

Codex 的最简配置如下：

```json
{
  "adapterType": "paperclip_runner",
  "adapterConfig": {
    "provider": "codex",
    "model": "gpt-5.6-sol",
    "codexPermissionMode": "never"
  }
}
```

凭据绑定需要单独配置。[自定义模型服务提供商](../../connectors/custom-model-providers.md)说明了哪些路由适用于各个 Harness。

## 权限

| 服务提供商 | 配置项 | 支持的行为 |
| --- | --- | --- |
| Codex | `codexPermissionMode` | 只允许 `never`，界面显示为 **Automatic (isolated)**。Paperclip 仍会独立限制工作区、网络和环境。 |
| OpenCode | `opencodePermissionMode` | `allow`（默认值）、`ask` 或 `deny`。 |
| ACP | `acpxPermissionMode` | `approve-all`（默认值）、`approve-paperclip`、`approve-reads` 或 `deny-all`。 |
| 托管服务提供商 | 服务提供商自己的策略 | 按经过认证的配置文件和 Paperclip 策略进行非交互式执行。 |

公司权限、受治理的审批和预算仍然适用。在较严格的 ACP 模式下，Grok 无法自动批准 Paperclip 工具调用；此类请求会等待审批。Cursor 会话模式与 ACP 权限策略是相互独立的设置。

## 会话与任务模式

Runner 会记录原生会话标识，并通过服务提供商支持的会话加载路径进行恢复。恢复时会重新检查已保存的服务提供商、环境、凭据和任务授权；运行持久化并不保证每种服务提供商操作都能在中断后恢复。

支持标准、规划和询问任务。Cursor 另外有 `acpxSessionMode`，可设为 `agent`（默认值）、`plan` 或 `ask`。选择 Plan 或 Ask 会改变 Cursor 原生会话行为，但 Paperclip 的权限和环境边界仍然有效。

<a id="pi-on-paperclip-runner"></a>
### Paperclip Runner 上的 Pi

将 **ACP agents** 选作 **Harness**，再将 **Pi** 选作 **ACP agent**。Pi 接受任意 `provider/model` ID，因此请准确输入所需模型。界面会显示 **Pi thinking level** 字段，可选值为 `off`、`low`（默认值）、`high` 和 `max`；该字段保存为 `piThinkingLevel`。Runner 会在每次会话允许提示模型之前检查此级别；更改级别会启动新会话。如需在主机上安装固定版本的 Pi 运行时，请使用 [`paperclipai runtime setup pi`](../cli/runtime.md#set-up-pi)。Pi 从绑定到智能体执行环境的密钥读取模型凭据，例如 `OPENROUTER_API_KEY`、`OPENAI_API_KEY`、`ANTHROPIC_API_KEY` 或 `GEMINI_API_KEY`；请确保模型 ID 使用你提供的密钥所属服务提供商前缀，例如 `openrouter/…`。

### 后续轮次中的环境变量

为智能体、项目、环境或例程配置的任务环境变量会传递给原生服务提供商进程及其工具命令，包括自定义的 `PAPERCLIP_*` 名称。添加、修改或删除变量时，Runner 会在下一个运行边界替换保留的服务提供商进程，因此下一轮会读取新值。变量未变时，Runner 可以复用预热进程。Runner 的启动路径只传递变量名称；密钥值留在子进程环境中。主机上的环境密钥绝不会被转发。一次原生启动最多携带 128 个已配置变量名。

现有的 `cursor`、`grok_local`、`claude_local`、`codex_local`、`opencode_local` 和 `pi_local` 智能体仍然使用直接适配器，除非你明确切换到 Runner。直接适配器上保存的旧版原生 Runner 开关不会让新的运行改走这条路径。

## 当前限制

- Cursor 需要明确指定模型。其部分原生计数器只是诊断观测值；无法获得权威的单次运行 token 总量和美元成本。
- Grok 的原生 Runner 用量核算尚未通过资格验证。缺少成本数据不代表支出为零。
- 不支持 ACP 实时引导。后续工作会使用控制器队列，而不会声称这是服务提供商原生队列功能。
- 各服务提供商的原生会话能力不同。支持加载会话不代表也支持原生分叉或恢复方法。
- 环境就绪情况与模型访问是两回事。安装检查成功不能证明服务提供商会接受付费运行。

## 在 Runner 上使用 OpenCode

- **回复会在生成时持续流式传输。** 原生 OpenCode 会转发实时到达的文本和推理内容。会话就绪后，排队的聊天消息会立即派发。
- **无需先执行安装步骤。** 服务端将 `opencode-ai@1.18.34` 作为依赖随包提供；即使包管理器跳过了 postinstall 脚本，Runner 也能找到对应的平台包。
- **远程运行使用随附的 Linux 资源。** 在远程沙盒中，Runner 会选择随包提供的 Linux 守护进程和 OpenCode 资源包，因此镜像和 Paperclip 软件包必须来自同一个发行版本。

## 沙盒镜像与运行时版本

在远程沙盒中运行原生 Codex 时，Runner 会在启动前检查其中安装的 Codex 版本。接受从 `0.149.0` 起、但低于 `0.161.0` 的所有稳定版本，因此镜像不必严格匹配 `0.160.0` 的安装固定版本；如果兼容版本与固定版本不同，运行日志会记录此情况。超出此范围的版本或预发布版本会以 `runner_remote_provider_artifact_incompatible` 错误停止运行。错误消息会提示你安装受支持的稳定版 Codex，或将 `PAPERCLIP_RUNNER_REMOTE_CODEX_NPM_SPEC` 设置为固定的软件包。

使用 ChatGPT 登录时，某些 Codex 模型需要较新的 CLI：`gpt-6.1-sol` 需要 `0.159.0` 或更高版本；`gpt-6-sol` 和 `gpt-6-luna` 需要 `0.157.0` 或更高版本。如果新运行所用沙盒中的 Codex 版本过旧，无法支持所选模型，Runner 会在启动前切换到同系列中最接近的旧版模型，或切换到 Runner 默认模型。任务会显示类似 *“由于沙盒中的 Codex 0.156.1 不支持 gpt-6-sol，正在使用 gpt-5.6-sol。”* 的警告；运行日志会记录 `runner.model_fallback` 事件。智能体和任务设置仍保留你所选的模型。若要使用该模型，请更新沙盒中的 Codex CLI。

如果你要自行构建沙盒镜像，升级时请注意以下两点：

- **先部署，再推广镜像。** 推广带有新运行时固定版本的沙盒镜像之前，先升级 Paperclip 控制器和 Runner。较旧的控制器会拒绝新的服务提供商资源包固定版本。
- **为 Sonnet 5.5 重建旧版 Claude Code 镜像。** 使用低于 `2.1.284` 的 Claude Code 构建的镜像，必须先重建，CLI 路径才能使用 Claude Sonnet 5.5。参见 [Claude Code 模型要求](claude-code.md#model-discovery)。

## 在一台机器上运行多个 Runner

沙盒服务提供商以监听器模式启动 Runner 守护进程（`paperclip-runnerd`）时，默认监听端口 `43127`。如果你要在同一台机器上运行多个监听型 Runner，请使用 `--listen-port` 为每个 Runner 分配独立端口，取值范围为 `1` 到 `65535`。Runner 仍绑定到 `0.0.0.0`，只接受对应运行专属路径上的连接，并保持相同的身份验证方式。重新连接的预热 Runner 会保留启动时使用的端口；拒绝其切换端口的请求。

## OpenAI Dot

**OpenAI Dot**（`openai_dot`）是一个实验性 Runner 服务提供商，它连接的是你现有的 Dot（一个在 Paperclip 外部运行的助手），而不是你机器上的编程运行时。Runner 仍然负责分配生命周期、持久等待、Paperclip 工具和完成审核；Dot 自己管理模型和计费。

- 在 **Settings → Experimental** 中开启 **OpenAI Dot** 和 **Assistant connections (MCP)**。两者默认都关闭。Dot 不需要通用的 **Paperclip Runner** 设置；开启 Dot 也不会启用其他 Runner 服务提供商。
- Dot 需要已通过身份验证且拥有公共 HTTPS URL 的实例。分配工作前，先配对 Dot 并验证事件传递。
- 在开关关闭时分配新工作会以 `paperclip_runner_dot_disabled` 失败。
- Dot 不会报告 token 用量或成本，因此 Paperclip 无法强制执行服务提供商支出上限。已知的公司和智能体预算限制仍然适用。

有关设置和配对方法，请参见[外部智能体](../../experimental/external-agents.md)。

## 后续步骤

- [Cursor Local 与原生 Cursor](cursor-local.md)
- [Grok Local 与原生 Grok](grok-local.md)
- [运行时设置命令](../cli/runtime.md)
- [适配器概览](overview.md)
