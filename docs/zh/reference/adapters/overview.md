---
paperclip_version: v2026.1023.0
seo_title: 适配器概览
seo_description: 适配器将 Paperclip 控制平面连接到执行实际工作的运行时。从这里比较可选项，并为聘用的每个智能体选择适配器。
---

# 适配器概览

适配器将 Paperclip 控制平面连接到实际执行工作的运行时。需要选择适配器、了解 Paperclip 对适配器的要求，或开发新的适配器包时，请阅读本节。

---

## 适配器的作用

每个适配器都负责以下核心工作：

1. 启动或调用底层运行时。
2. 传递智能体的公司、任务和唤醒上下文。
3. 捕获结果、会话状态和用量元数据。
4. 在运行开始前验证环境。
5. 可选地提供自定义界面执行记录解析器和技能同步行为。

> **注意：** Paperclip 负责编排智能体。适配器决定运行时如何启动、如何保存状态，以及如何解释其输出。

---

## 选择适配器

| 使用场景 | 从这里开始 |
|---|---|
| 在本机运行 Claude Code | [Claude Code](./claude-code.md) |
| 在本机运行 OpenAI Codex CLI | [Codex](./codex.md) |
| 在本机运行 Gemini CLI | [Gemini CLI](./gemini-cli.md) |
| 在本机运行 Cursor Agent CLI | [Cursor Local](./cursor-local.md) |
| 使用服务提供商/模型路由的 OpenCode CLI | [OpenCode](./opencode.md) |
| 使用内置工具集的 Pi CLI | [Pi](./pi.md) |
| 具备持久记忆和 30 多种工具的 Hermes Agent | [Hermes](./hermes.md) |
| 在本机运行 Grok Build CLI | [Grok Local](./grok-local.md) |
| 在本机运行 Kimi Code CLI | [Kimi Code](./kimi-local.md) |
| 通过 WebSocket 网关连接 OpenClaw | [OpenClaw Gateway](./openclaw-gateway.md) |
| 自定义 shell 命令或脚本 | [Process](./process.md) |
| 你控制的 webhook 或云服务 | [HTTP](./http.md) |
| 独立 npm 包或本地插件 | [外部适配器](./external-adapters.md) |
| 从头开发新的适配器包 | [创建适配器](./creating-an-adapter.md) |
| 开发自定义执行日志解析器 | [适配器界面解析器契约](./adapter-ui-parser.md) |

如果你刚开始配置，最常见的路径是：

1. 如果智能体运行在与 Paperclip 相同的计算机上，请选择本地适配器。
2. 如果运行时只是一个命令，请选择 `process`。
3. 如果运行时位于 API 或 webhook 后面，请选择 `http`。
4. 如果你希望独立管理版本并安装运行时，请使用外部适配器插件。

---

## 内置适配器

这些适配器随 Paperclip 一起发布，并始终可在主机中使用：

| 适配器 | 类型键 | 界面可用性 | 适用场景 |
|---|---|---|---|
| [Claude Code](./claude-code.md) | `claude_local` | 可选（推荐） | 支持会话持久化、技能同步和结构化执行记录解析的 Claude Code 运行。 |
| [Codex](./codex.md) | `codex_local` | 可选（推荐） | 支持会话持久化和受管理 `CODEX_HOME` 的 Codex CLI 运行。 |
| [Gemini CLI](./gemini-cli.md) | `gemini_local` | 可选 | 支持会话恢复和本地技能同步的 Gemini CLI 运行。 |
| [Cursor Local](./cursor-local.md) | `cursor` | 可选 | 使用 `--resume` 保持会话连续性并输出结构化流的 Cursor Agent CLI 运行。 |
| [OpenCode](./opencode.md) | `opencode_local` | 可选 | 支持服务提供商/模型路由和 `--session` 恢复的 OpenCode CLI 运行。 |
| [Pi](./pi.md) | `pi_local` | 可选 | 使用内置工具集并支持服务提供商/模型路由的 Pi CLI 运行。 |
| [Hermes](./hermes.md) | `hermes_local` | 可选 | 具备持久记忆、30 多种工具、80 多项技能和多服务提供商路由的 Hermes Agent 运行。 |
| [Grok Local](./grok-local.md) | `grok_local` | 可选 | 使用 `--resume` 保持会话连续性、流式输出推理内容，并将技能暂存到 `.claude/skills` 的 Grok Build CLI 运行。 |
| [Paperclip Runner](./paperclip-runner.md) | `paperclip_runner` | 实验性；受实例设置控制 | 原生运行生命周期、持久会话恢复，以及经过验证支持的 Codex、OpenCode、Claude、Grok、Cursor 和 Pi 运行时。 |
| [Kimi Code](./kimi-local.md) | `kimi_local` | 可选 | 通过共享 ACP 引擎运行 Kimi Code CLI；支持无界面 CLI 回退、会话恢复和按次运行注入技能。 |
| [OpenClaw Gateway](./openclaw-gateway.md) | `openclaw_gateway` | **即将推出**（请使用 OpenClaw 邀请流程） | 通过 WebSocket 网关协议访问的远程 OpenClaw 实例。 |
| [Process](./process.md) | `process` | **即将推出**（仅 API/导入可用） | Shell 命令、脚本和自定义本地运行时。 |
| [HTTP](./http.md) | `http` | **即将推出**（仅 API/导入可用） | 通过类似 webhook 的调用方式连接你自己的服务。 |

> **信息：** 智能体配置中的适配器类型下拉菜单当前将 `openclaw_gateway`、`process` 和 `http` 标记为 **“即将推出”**。它们在运行时已完全可用，只是目前还不能从界面手动选择。直接界面选择功能推出前，请通过 API 或导入的公司导出包进行配置。

---

## 原生 Paperclip Runner

如果你需要原生运行生命周期和内置任务工具，请选择 **Paperclip Runner**。然后选择其 **Harness**；如果使用 **ACP agents**，还要选择对应的 **ACP agent**。现有直接适配器仍使用各自的运行路径；启用此功能不会迁移它们。

自托管实例默认开启 `enableNativeRunner`，由 Cloud 管理的实例则默认关闭。**ACP agents** 现在还包括经过验证支持的 **Cursor** 和 **Pi** 运行时。Cursor 使用通过 `paperclipai runtime setup cursor` 准备的固定版本运行时、明确指定的模型，以及 Agent、Plan 或 Ask 会话模式；Pi 使用 `paperclipai runtime setup pi` 和明确指定的 `provider/model` ID。Grok 使用单独固定版本的原生运行时。请参阅 [Paperclip Runner](./paperclip-runner.md)、[Cursor](./cursor-local.md#cursor-on-paperclip-runner) 和 [Grok](./grok-local.md#grok-build-on-paperclip-runner)。

---

## 外部适配器

外部适配器需要单独安装，并在启动时从适配器插件存储中加载。安装后，它们与内置适配器的行为相同，但拥有独立的软件包，可独立管理版本。

请从 Board 界面安装，或调用 `POST /api/adapters/install`。

请参阅：

- [外部适配器](./external-adapters.md)
- [适配器界面解析器契约](./adapter-ui-parser.md)
- [创建适配器](./creating-an-adapter.md)

---

## 常见概念

| 概念 | 重要性 |
|---|---|
| `cwd` | 适配器的工作目录。大多数本地适配器都要求使用绝对路径。 |
| `env` | 传递给运行时的环境变量。敏感值最好使用密钥引用。 |
| 会话状态 | 允许适配器在下一次心跳时恢复同一对话或命令状态。 |
| 技能 | 适配器专属逻辑，用于让运行时访问 Paperclip 技能。 |
| `testEnvironment()` | 适配器就绪检查。界面会在保存或运行适配器前调用它。 |
| 界面解析器 | 将 stdout 转换为结构化执行记录条目，供执行查看器显示。 |

> **提示：** 如果不确定先看哪一页，请从与你当前使用的运行时相匹配的适配器页面开始；只有需要打包或扩展适配器时，才继续查看外部或自定义适配器文档。

---

## 反馈粒度

想在智能体工作时查看其思考过程吗？你能在执行记录中看到多少实时细节，取决于所选适配器。

所有适配器都会将 stdout 实时传输到执行日志，Paperclip 也会在智能体工作时实时显示日志——即使执行目标是沙盒，日志也会增量读取并逐渐填充。不同之处在于输出的*结构*：适配器发出的事件流越丰富，执行记录能呈现的内容就越多，超出纯文本输出的部分也越多。

以下是大致分级，按信息丰富程度从高到低排列：

- **ACP 引擎——完整的结构化事件流。** 当 `claude_local`、`codex_local`、`gemini_local` 或 `kimi_local` 通过 Agent Client Protocol 运行时——例如使用默认的 `engine: auto`，或明确指定 `engine: acp`——每个有意义的时刻都会生成一条 JSONL 事件：会话身份、状态（进度文本及上下文窗口用量）、Assistant 和思考过程的令牌增量、工具调用进度中的标题及状态更新、结果停止原因摘要，以及错误。执行记录会将这些内容渲染为实时更新的消息、思考、工具和状态区块；同一工具调用的重复状态更新会合并到一张工具卡片中，而不会逐条堆叠。
- **CLI 包装器（使用 `engine: cli` 的 `claude_local` / `codex_local` / `gemini_local`，以及 `cursor`、`opencode_local` 等）——CLI 自身的事件流。** 这些适配器会解析各 CLI 的流式 JSON 输出，包括 Assistant 文本、工具调用和结果，以及最终的用量/费用摘要。CLI 自身会输出多少细节，你就能看到多少。
- **Paperclip Runner——原生生命周期和结构化活动。** 原生运行路径会通过自身的执行记录路径呈现服务提供商活动和 Paperclip 工具。不同服务提供商支持的能力和用量统计有所不同；Cursor 和原生 Grok 不会提供权威的按次运行费用总额。
- **通用适配器（`process`、`http`）——纯文本输出。** 你只能看到 stdout 和 stderr 行，没有结构化执行记录。

如果运行的是沙盒工作器，请将 `engine` 保持为 `auto`，这样在沙盒提供 Paperclip 双向进程会话时，适配器会使用 ACP。只运行一次性命令的沙盒以及非沙盒远程目标会回退到 CLI 路径。无论哪种情况，沙盒执行日志都会实时传输；丰富的事件流则会在远程执行进行时，让执行记录和状态栏更有用。

---

## 后续阅读

- [Paperclip Runner](./paperclip-runner.md)
- [Claude Code](./claude-code.md)
- [Codex](./codex.md)
- [Gemini CLI](./gemini-cli.md)
- [Cursor Local](./cursor-local.md)
- [OpenCode](./opencode.md)
- [Pi](./pi.md)
- [Hermes](./hermes.md)
- [Grok Local](./grok-local.md)
- [Kimi Code](./kimi-local.md)
- [OpenClaw Gateway](./openclaw-gateway.md)
- [Process](./process.md)
- [HTTP](./http.md)
- [外部适配器](./external-adapters.md)
