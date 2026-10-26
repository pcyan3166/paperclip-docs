---
paperclip_version: v2026.1023.0
seo_title: 选择智能体适配器
seo_description: 比较用于运行智能体的适配器——Claude Code、Codex、OpenCode、HTTP webhook 等——并为每种角色选择合适的连接方式。
---

# 智能体适配器

在 Paperclip 中创建智能体时，最先配置的事项之一就是它的适配器。适配器连接 Paperclip 与实际运行智能体的 AI 系统，告诉 Paperclip 如何启动智能体、如何向它分配工作，以及如何接收运行结果。

可以把它想成不同国家使用的电源转换插头。Paperclip 的任务系统和组织架构就像电力标准，各地都相同；但插头需要与所连接的 AI 运行环境相匹配。Claude Code 需要一种配置，OpenAI Codex 需要另一种配置，而自定义的云端智能体还需要第三种配置。

没有适配器的智能体只是数据库中的一条记录；配好适配器后，它才能成为团队中实际工作的成员。

---

## 适配器对比

| 适配器 | 适用场景 | 需要准备 |
|---|---|---|
| `claude_local` | 大多数用户；在 Mac 上运行 Claude | 安装 Claude Code，并准备 Anthropic API 密钥 |
| `codex_local` | OpenAI 用户；在 Mac 上运行 Codex | 安装 Codex CLI，并准备 OpenAI API 密钥 |
| `gemini_local` | Gemini 用户；在 Mac 上运行 Gemini CLI | 安装 Gemini CLI，并准备 Google 凭据 |
| `opencode_local` | 需要使用多个服务提供商并切换模型 | 安装 OpenCode CLI，并准备相应 API 密钥 |
| `cursor` | 使用 Cursor Agent CLI，并需要可恢复的本地运行 | 安装 Cursor Agent CLI 并完成认证 |
| `paperclip_runner` | 使用原生会话和 Paperclip 任务工具 | 启用 Runner、准备执行环境和兼容的服务凭据 |
| `pi_local` | 需要 Pi 内置工具集的 Pi 用户 | 安装 Pi CLI，并准备相应 API 密钥 |
| `grok_local` | xAI 用户；运行 Grok Build CLI | 安装 Grok CLI，并准备 xAI API 密钥或 SuperGrok 登录 |
| `hermes_local` | 需要持久记忆、30 多种工具和 80 多种技能 | 安装 Hermes Agent（Python 3.10+） |

对大多数刚开始使用 Paperclip 的用户来说，**`claude_local`** 是合适的选择。它会在你的 Mac 上直接运行 Anthropic Claude.ai 背后的 Claude，只需要安装 Claude Code，并通过环境变量配置 API 密钥。

![新建智能体对话框的第二步“选择适配器”，显示每种可用适配器的卡片](../../../user-guides/screenshots/light/agents/adapter-type-dropdown.png)

> **说明：** 在 Paperclip Cloud 上，新建智能体选择器提供 Claude、Codex、OpenCode 和 Grok。设置 Grok 智能体时，可连接 xAI 订阅或 API 密钥。

> **建议：** Grok Build 也可以通过实验性 **Paperclip Runner** 适配器（`paperclip_runner`）运行，此时选择 **Grok Build** 作为服务提供商。执行环境中必须在固定路径安装 Grok Build。详见 [Paperclip Runner 上的 Grok Build](../../reference/adapters/grok-local.md#grok-build-on-paperclip-runner)。

### 明确选择 Paperclip Runner

如需使用原生生命周期，请选择 **Paperclip Runner**，再选择服务提供商：**Codex**、**OpenCode**、**Claude Managed**、**AWS AgentCore**、**ACP agents** 或 **Grok Build**。在 **ACP agents** 下，可选择 Grok Build、Claude、Cursor 或 Pi；GitHub Copilot 标记为等待资格验证。已有的直接适配器智能体会继续使用当前运行环境，直到你更改适配器。

此功能处于实验阶段：自托管实例默认开启 `enableNativeRunner`，Cloud 托管实例默认关闭。详见 [Paperclip Runner](../../reference/adapters/paperclip-runner.md)。

### 默认情况下，智能体会在全自动模式运行

智能体通常会在无人旁观时工作，因此没有人可以回答“我可以运行此命令吗？”之类的提示。正因如此，Claude Code、Codex、OpenCode 和 Gemini CLI 等编程适配器默认以全自动模式启动，可直接使用工具（包括连接服务中的 MCP 工具），不会停下来请求批准。

你可以在各智能体的适配器配置中降低自动程度。每种适配器的参考页面会说明对应开关：

- Claude Code：`dangerouslySkipPermissions`（参见 [Claude Code 权限](../../reference/adapters/claude-code.md#permissions)）
- Codex：`dangerouslyBypassApprovalsAndSandbox`（参见 [Codex 权限](../../reference/adapters/codex.md#permissions)）
- OpenCode：`dangerouslySkipPermissions`（参见 [OpenCode](../../reference/adapters/opencode.md)）

> **提示：** “全自动”指跳过运行过程中的确认提示，不代表智能体可以随意操作公司资源。公司权限、预算和审批要求依然生效。

---

## Claude Code（`claude_local`）

`claude_local` 适配器会在你的 Mac 上直接使用 Anthropic 的命令行 AI 工具 Claude Code 运行智能体。Paperclip 触发心跳时，会将上下文和任务交给 Claude Code，等待运行完成，然后读取 Claude 的工作结果。

**实际体验：** 智能体由 Claude 驱动，可以在任务工作区内读取和写入文件、浏览网页及运行终端命令。它可以完成实际编程、撰写文档、研究等多种工作。

### 前置条件

- 在 Mac 上安装 Claude Code（[claude.ai/code](https://claude.ai/code)）
- 准备 Anthropic API 密钥（[console.anthropic.com](https://console.anthropic.com)）

### 配置字段

**Working directory（工作目录，已弃用）**

新建智能体时不会显示此字段。它只会在编辑已有工作目录配置的旧版本地智能体时出现，作为旧配置的回退选项。新智能体会从任务所属项目的工作区获取文件夹，因此请配置[项目工作区](../projects-workflow/workspaces.md)，而不要为每位智能体指定一个文件夹。如果旧智能体仍显示此字段，可以保持原样；等其工作都迁入项目后，也可以清除此字段。

**Model（模型）**

选择使用哪种 Claude 模型。留空时，Paperclip 会使用 `claude-opus-5`。常见选项：

- `claude-opus-5-5` 或 `claude-opus-5`——能力最强、推理效果最佳，费用也最高。适合 CEO 或负责复杂策略的智能体。
- `claude-sonnet-5-5` 或 `claude-sonnet-5`——速度快、能力强、费用较低。适合处理日常任务的执行人员。使用 Sonnet 5.5 需要 Claude Code `2.1.284` 或更高版本。

不确定时，可为执行人员选择 Sonnet，为 CEO 选择 Opus。模型旁的 **effort（思考强度）** 选项会随模型不同而改变；较新的 Opus、Sonnet 5 和 5.5、Fable 模型会提供 `xhigh` 和 `max`。详见 [Claude Code——推理强度](../../reference/adapters/claude-code.md#reasoning-effort)。

**Environment variables（环境变量）**

智能体表单中有 **Environment variables（环境变量）** 区域。在此添加 `ANTHROPIC_API_KEY`，可以输入明文值，也可以引用机密信息。如果不确定变量名，使用标准名称 `ANTHROPIC_API_KEY` 即可。

**Timeout (sec)（超时秒数）**

设置一次心跳最多可运行多久，超过后 Paperclip 会中止运行。已有智能体可在 **Harness / Runtime（运行环境）** 页面的 **Run Policy → Advanced Run Policy（运行策略 → 高级运行策略）** 中找到此项。初始值为 `0`，代表不设超时。设置超时后，请留出足够时间；复杂编程任务可能需要 10 分钟甚至更久，限制过低会让智能体在任务中途停止。

![智能体 Harness / Runtime 页面的适配器卡片显示 Claude Code 类型、模型和思考强度，下方为配置卡片顶部](../../../user-guides/screenshots/light/agents/claude-local-config-filled.png)

### 常见错误

**“Claude Code not found（找不到 Claude Code）”**——尚未安装 Claude Code，或它不在系统 `PATH` 中。请从 [claude.ai/code](https://claude.ai/code) 安装，然后在智能体 **Harness / Runtime（运行环境）** 页面底部的 **Test your agent（测试智能体）** 卡片中点击 **Run test（运行测试）**，确认 Paperclip 能找到它。

**“API key invalid（API 密钥无效）”**——确认环境变量名称与配置完全一致，并确认密钥以 `sk-ant-` 开头。Anthropic 密钥区分大小写。

**“Timeout（超时）”**——心跳运行时间超过超时设置。请增加超时值，或把任务拆成更小的部分，让每次心跳能更快完成。

---

## Codex（`codex_local`）

`codex_local` 适配器使用 Mac 上的 OpenAI Codex CLI 运行智能体。其工作方式与 `claude_local` 相同，只是使用 OpenAI 模型而非 Anthropic 模型。

### 前置条件

- 在 Mac 上安装 OpenAI Codex CLI
- 准备 OpenAI API 密钥（[platform.openai.com](https://platform.openai.com)）

### 配置字段

配置字段与 `claude_local` 大致相同，主要是选择模型和设置环境变量，不过这里使用 OpenAI 的选项。与 Claude Code 一样，**Working directory（已弃用）** 只会在编辑已有目录设置的旧智能体时出现。

Codex 的 **Model（模型）** 示例：

- `gpt-5.6-sol`——默认模型，也是常规起点。
- `gpt-6.1-sol`——最新 Codex 模型，推理强度最高支持 `ultra`。
- `gpt-6-astra`、`gpt-6-sol` 或 `gpt-6-luna`——GPT-6 系列，提供额外的推理强度级别。
- `o4-mini`——速度快、成本低，适合日常任务。

完整列表见 [Codex——模型](../../reference/adapters/codex.md#models)。

![Codex 本地适配器配置表单](../../../user-guides/screenshots/light/agents/codex-local-config.png)

---

## 其他本地适配器

OpenCode 适配器 `opencode_local` 使用 OpenCode 运行智能体。OpenCode 是灵活的开源 AI 终端工具，支持多个服务提供商。你可以通过 `adapterConfig.model` 字段选择服务提供商和模型，格式为 `provider/model`。

**适用场景：** 你想轻松切换不同服务提供商，或使用不能直接通过 `claude_local` 或 `codex_local` 使用的模型。

配置方式与其他本地适配器相似，但增加了一个采用 `provider/model` 格式的 **Model（模型）** 字段，例如 `anthropic/claude-opus-4-6` 或 `openai/gpt-5.3-codex`。

---

## 你可能会看到的其他适配器

有三种内置适配器在智能体配置下拉菜单中标记为 **Coming soon（即将推出）**，目前不能直接从界面中选择。它们在运行时中功能完整；在界面选择器开放前，可以通过 API 或导入公司导出包来配置。

- `openclaw_gateway`——通过 WebSocket gateway 协议连接远程 OpenClaw 实例。常规方式是在公司设置页面使用 OpenClaw 邀请提示流程。详见 [OpenClaw Gateway](../../reference/adapters/openclaw-gateway.md)。
- `http`——以 webhook 方式调用你自己的服务，面向开发自定义智能体集成的开发者。详见 [HTTP](../../reference/adapters/http.md)。
- `process`——运行任意本地命令或脚本。详见 [Process](../../reference/adapters/process.md)。

也可以安装外部适配器插件；加载后会出现在下拉菜单中。

---

## HTTP Webhook（`http`）

> **信息：** `http` 适配器当前不能从智能体配置下拉菜单中选择。现阶段请通过 API 或导入公司导出包进行配置。

`http` 适配器会连接你控制的 Web 服务器或云函数。心跳触发时，Paperclip 会将包含智能体上下文和任务的 HTTP 请求发送到你的端点，由你的端点处理工作并返回结果。

**适用场景：** 智能体运行在云端、另一台机器上，或构建在支持 webhook 调用的平台上。

### 配置字段

**Webhook URL（`url`）**

心跳触发时 Paperclip 要调用的完整 URL。该 URL 必须可公开访问，或能够从 Paperclip 实例连接。它是智能体 **Harness / Runtime（运行环境）** 页面 **Configuration（配置）** 卡片中唯一与 HTTP 专属的字段。

**Headers（`headers`）**

额外请求标头，通常包含一个机密令牌，供你的服务器验证请求是否来自 Paperclip。请通过 API 或公司导出包在适配器配置中设置；表单不会显示此字段。

**Timeout（`timeoutMs`）**

Paperclip 等待响应的最长时间，单位为毫秒；超时后会将心跳视为失败。通过 API 或公司导出包设置。

![智能体使用 HTTP 适配器时的 Harness / Runtime 页面，HTTP 被选为适配器类型](../../../user-guides/screenshots/light/agents/http-adapter-config.png)

> **说明：** HTTP 适配器面向开发自定义智能体集成的开发者。如果你是在本地使用标准 AI 服务提供商，`claude_local` 或 `codex_local` 会简单得多。

---

## Shell Process（`process`）

> **信息：** `process` 适配器当前不能从智能体配置下拉菜单中选择。现阶段请通过 API 或导入公司导出包进行配置。

`process` 适配器会在运行 Paperclip 的同一台机器上执行命令。心跳触发时，Paperclip 会运行你指定的命令，并通过标准输入或环境变量传递智能体上下文。

**适用场景：** 自定义脚本、本地自动化，或使用没有专用适配器的工具构建智能体。

### 配置字段

**Command（命令）**

可执行命令及其参数。运行 Paperclip 的机器必须能够访问此命令。

**Working directory（工作目录）**

运行命令的目录。

**Environment variables（环境变量）**

提供给进程的额外环境变量，例如 API 密钥和路径。

---

## 获取 API 密钥

如果你还没有 API 密钥，请按以下步骤操作：

<!-- tabs: Anthropic（Claude）, OpenAI -->

<!-- tab: Anthropic（Claude） -->

1. 前往 [console.anthropic.com](https://console.anthropic.com) 登录或创建账户。
2. 在左侧边栏点击 **API Keys**。
3. 点击 **Create Key**。
4. 输入容易识别的名称（例如“Paperclip”）。
5. 立即复制密钥：它以 `sk-ant-` 开头，且只显示一次。

请将密钥存放在安全位置（密码管理器很适合）。配置适配器时使用环境变量名称 `ANTHROPIC_API_KEY`。

> **注意：** 创建密钥后请立即复制。Anthropic 只显示一次；如果丢失，需要创建新密钥并更新适配器配置。

<!-- tab: OpenAI -->

1. 前往 [platform.openai.com](https://platform.openai.com) 登录或创建账户。
2. 点击右上角的个人资料图标，然后点击 **API keys**。
3. 点击 **Create new secret key**。
4. 输入名称（例如“Paperclip”），然后点击 **Create secret key**。
5. 立即复制密钥：它以 `sk-` 开头，且只显示一次。

请将密钥保存在安全位置。配置适配器时使用环境变量名称 `OPENAI_API_KEY`。

> **注意：** 创建密钥后请立即复制。OpenAI 只显示一次；如果丢失，需要创建新密钥并更新适配器配置。

<!-- /tabs -->

---

## 已完成

现在你已经了解适配器的用途，以及如何配置最常见的几种适配器。下一篇指南介绍执行工作区，也就是智能体执行文件类任务时使用的隔离代码环境。

[执行工作区 →](../projects-workflow/workspaces.md)
