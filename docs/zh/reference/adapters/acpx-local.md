---
paperclip_version: v2026.722.0
seo_title: ACPX 本地适配器（已退役）
seo_description: ACPX 已退役。它把 ACP 当作独立的智能体选项，但 ACP 实际上是执行方式的细节；本页介绍替代方案。
---

# ACPX Local（已退役）

> **此适配器已退役。** 创建智能体时不再能选择 `acpx_local`。ACP 现在是各执行框架适配器的一项能力，而不再是单独的适配器；因此请选择与你的执行框架相匹配的适配器，并让 ACP 自行处理执行：
>
> [Claude Code](./claude-code.md) · [Codex](./codex.md) · [Gemini CLI](./gemini-cli.md)

## 替代方案

过去，ACPX 是唯一能够面向 Claude、Codex 或自定义 ACP 服务器的适配器。这使 ACP 看起来像智能体的独立*选项*，但实际上 ACP 是所选执行框架的一种执行*能力*。

现在，`claude_local`、`codex_local` 和 `gemini_local` 都原生支持 ACP。将 `engine` 保持为默认的 `auto`，当主机支持 ACP 时就会使用 ACP；不支持时则自动回退到 CLI 路径。如果必须使用 ACP，且缺少前置条件时应立即停止执行，请设置 `engine: "acp"`。

这些适配器各自的配置部分都会说明 ACP 字段：`agentCommand`、`mode`、`nonInteractivePermissions`、`stateDir` 和 `warmHandleIdleMs`。

## 在 Windows 上原生运行

如果你使用 Windows，这是个好消息：现在本地 ACP 智能体也能在 Windows 上运行，与 Linux 一样。过去，内嵌的 ACPX 引擎会先将每个本地智能体命令包装成自动生成的 Bash 脚本，然后再启动。这意味着它只能在有 Bash shell 的环境中工作。现在这个包装层已移除。Claude、Codex、Gemini 和任何自定义 ACP 适配器都会在你所用的平台上作为原生进程启动，中间不再需要 shell 脚本。

启用此功能无需更改任何设置。将 `engine` 设为 `auto`（默认值）或 `acp`，选择与你的执行框架相匹配的适配器后，执行在 Windows 和 Linux 上的启动方式就相同。

以下是底层的更改，以确保原生运行路径正常工作：

- **环境变量通过 ACPX 会话选项传递。** 智能体环境（包括 Paperclip 为某次执行注入的值）不再写入 shell 脚本，而是作为会话选项的一部分传给 ACPX 运行时，并在进程启动时应用。
- **在进程内部捕获智能体写入 stderr 的输出。** 引擎会直接读取子进程的 stderr，而不是依赖 shell 重定向，因此各个平台的执行日志仍会显示诊断信息。
- **在 Windows 上优先使用 `.cmd` 启动脚本。** 引擎从 `node_modules/.bin` 解析捆绑的 `npm`/`pnpm` 二进制文件时，在 Windows 上会先查找 `.cmd` 启动脚本（例如 `pnpm.cmd`），然后才回退到不带扩展名的名称——因为这些工具在 Windows 上实际就是这样安装的。
- **符号链接失败时回退到复制文件。** Paperclip 会通过符号链接将运行时技能和配置文件暂存到目标位置。在 Windows 上，创建符号链接可能因 `EPERM` 权限错误而失败；遇到这种情况，引擎会改为复制文件，确保设置流程仍能成功。

## 如果你之前使用 ACPX 智能体

**你无需执行任何操作。** 实例升级时，现有 `acpx_local` 智能体已自动迁移。迁移会读取每个智能体的旧 `agent` 字段，并移动到对应适配器：

| 旧 `adapterConfig.agent` 值 | 新适配器 | 新配置 |
|---|---|---|
| `codex` | `codex_local` | `engine: "acp"` |
| 其他任何值（包括未设置） | `claude_local` | `engine: "acp"` |

对于 Codex 智能体，旧的 `effort`、`reasoningEffort` 和 `thinkingEffort` 字段会合并为单一的 `modelReasoningEffort` 值。

## 如果错误中仍出现 `acpx_local`

Paperclip 会有意保留已退役适配器的注册信息，这样过时的记录会显示明确错误，而不会悄悄回退到 `process` 适配器。如果某次执行日志显示 `acpx_local` 已退役，说明该智能体记录早于迁移；请打开智能体设置，将其切换为 `claude_local` 或 `codex_local`，并将 `engine` 设置为 `acp`。

## 相关内容

- [Claude Code](./claude-code.md) — Claude 执行框架，默认使用 ACP。
- [Codex](./codex.md) — Codex 执行框架，默认使用 ACP。
- [Gemini CLI](./gemini-cli.md) — Gemini 执行框架，默认使用 ACP。
- [适配器概览](./overview.md) — 当前可选的完整适配器列表。
- [创建适配器](./creating-an-adapter.md) — 如果内置适配器都不适用，可自行开发。
