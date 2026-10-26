---
paperclip_version: v2026.609.0
seo_title: CLI 概览
seo_description: Paperclip CLI 可启动实例，并以功能完整的终端方式管理公司，具备与界面相同的能力且支持脚本化。
---

# CLI 概览

Paperclip CLI（`paperclipai`）可以启动 Paperclip 实例，并以功能完整的终端方式管理公司。Paperclip 以界面为先，大多数操作员主要使用网页界面；而当终端更适合执行任务时，可使用 CLI，例如安装和配置实例、自动化和 CI、脚本化或批量操作，以及在没有浏览器的无头环境中运行。建议从本页开始了解：这里介绍 CLI 的两个组成层、身份验证方式，以及各参考页面和指南所在位置。

阅读时请记住：CLI 和 UI 是同一控制平面及同一数据的两个操作界面。本文档中的操作也都可以在界面中完成；CLI 只是让你能从 shell 中执行。如果你还想让 AI 使用 CLI、完全不打开浏览器地操作公司，可了解可选的自主运行工作流——这只是 CLI 的一种用途，并非 CLI 的前提。

---

## 两个层面

CLI 的所有功能都属于以下两个层面之一，它们的前置条件不同。

| 层面 | 功能 | 连接对象 | 是否需要凭据？ |
|---|---|---|---|
| **设置** | 引导初始化、诊断、配置并运行本地实例 | 本地配置文件和本地服务器进程 | 否 |
| **控制平面** | 管理公司、智能体、任务、目标、运行、预算等 | Paperclip 服务器 API | 是 |

**设置命令**负责启动服务器并确保其健康。它们操作磁盘上的配置（默认位于 `~/.paperclip`）和本地进程。主要命令包括 `onboard`、`doctor`、`configure`、`env`、`allowed-hostname`、`db:backup` 和 `run`（不带子命令）。详见[设置命令](./setup-commands.md)以及介绍 `env-lab` 和 `worktree` 工具的[开发环境](./dev-environments.md)参考。

**控制平面命令**是服务器 API 的 HTTP 客户端。它们需要知道服务器地址（[API 基础地址解析](#cli-如何查找服务器)）和你的身份（[凭据](#身份类型和凭据)）。这是 CLI 的主要部分，涵盖公司、项目、目标、任务、智能体、提示、运行、例程、审批、活动、控制面板、费用、工作区、访问、适配器、资源、技能、密钥、云端、插件和反馈。

> **注意：** `run` 有意复用为两种用途。`paperclipai run` **不带子命令**时会初始化并启动本地实例（设置操作）。`paperclipai run <subcommand>`（`list`、`live`、`get`、`events`、`log`、`cancel` 等）通过 API 检查和控制心跳**运行**（控制平面操作）。同一个动词对应两个不同概念，详见[run 参考](./run.md)。

---

## 工作在哪儿运行

在编写 CLI 脚本前，最重要的是理解这一点：**CLI 不会运行模型。** 智能体实际完成的工作（LLM 调用、编码、研究）在服务器端的 Paperclip 运行时及其适配器中执行。CLI 只负责触发和观察工作。

调用 `agent wake` 或 `heartbeat run` 时，CLI 会向服务器发送 POST 请求（`/api/agents/:id/wakeup`）；服务器运行适配器，CLI 再将运行事件和日志流式传回终端。你是在调度工作，而不是执行计算。

唯一例外是 `agent local-cli <agent>`。它会配置本地环境，让真人或本地 AI（Claude、Codex）在你自己的机器上以该智能体身份运行，并通过 API 报告结果。完整说明见[关键概念](../../guides/welcome/key-concepts.md)。

---

## 身份类型和凭据

每个控制平面调用都以*某个身份*发出。Paperclip 只有两种身份类型，CLI 配置文件会记录你使用哪一种。

| 身份类型 | 权限 | 常见用途 |
|---|---|---|
| **控制面板操作员** | 在整个实例范围内拥有完整控制面板授权 | 初始化、构建组织、生成凭据和调度所有工作 |
| **智能体** | 精确限定在一个公司和一个智能体中 | 智能体处理自己的工作，或本地 CLI 以该智能体身份运行 |

凭据类型也与这两种身份一一对应。

| 凭据 | 创建方式 | 属性 |
|---|---|---|
| **控制面板 API 令牌** | `token board create` | 可命名、支持过期和撤销，并在服务器端审计 |
| **智能体 API 密钥** | `token agent create` | 限定在一个公司和一个智能体中；明文仅在创建时显示**一次** |

获取首个凭据的方式取决于实例模式：

- **`local_trusted`**——CLI 可在不使用浏览器的情况下创建并批准自己的身份验证挑战；服务器将回环地址视为隐式控制面板身份（“local-board”）。这是无头初始化方式。
- **`authenticated`**——首个凭据来自 `auth bootstrap-ceo`（会生成一次性数据库邀请）或浏览器控制面板认领流程。之后，控制面板令牌可以在非交互环境中生成更多控制面板令牌和智能体密钥。

凭据命令见[身份验证](./authentication.md)，无需浏览器的流程见[安装](./installation.md)。

### 快速身份验证方式

- **`paperclipai connect`**——交互式向导（仅限 TTY）。解析 API 基础地址、检查健康状态、以控制面板身份登录、让你选择身份类型/公司/智能体、生成令牌、保存包含身份类型信息的配置文件，并打印 shell 环境变量命令。如果你正在终端前操作，从这里开始。
- **`paperclipai auth login`**——设备代码/浏览器批准流程，会按 API 基础地址保存控制面板凭据。适合非交互式控制面板登录。
- **`paperclipai agent local-cli <agent-id>`**——将 CLI 交给 AI 的一站式命令：创建长期有效的智能体 API 密钥，将 Paperclip 技能安装到 `~/.codex/skills` 和 `~/.claude/skills`，并打印 `PAPERCLIP_API_URL`、`PAPERCLIP_COMPANY_ID`、`PAPERCLIP_AGENT_ID` 和 `PAPERCLIP_API_KEY` 的导出命令。

```sh
paperclipai connect
# or, for an AI operator on this machine:
paperclipai agent local-cli <agent-id> -C <company-id>
```

身份验证流程中的选择判断见[CLI 身份验证](../../administration/cli-auth.md)。

---

## CLI 如何查找服务器

控制平面命令按以下确切顺序解析 API 基础地址。第一个能提供 URL 的来源会胜出。

1. `--api-base <url>`
2. `PAPERCLIP_API_URL`
3. 所选上下文配置文件的 `apiBase`
4. 本地 Paperclip 配置中的服务器端口
5. `http://localhost:3100`

> **提示：** 命令无法连接时，错误会包含实际尝试的 URL，并提示检查 `GET /api/health`。大多数“无法连接”问题都是 API 基础地址解析到了错误位置；首先查看错误中的 URL。

---

## 上下文配置文件

为避免一直重复输入相同标志，CLI 会将默认值存储在 `~/.paperclip/context.json` 中。配置文件包含身份类型信息：控制面板配置文件会记录 `persona=board`；智能体配置文件会记录 `persona=agent`、`agentId` 和 `agentName`。

特别重要的是，配置文件保存的是 `apiKeyEnvVarName`——即环境变量的*名称*，而非明文令牌。密钥保留在环境变量中；配置文件只记住应从何处读取。

```sh
paperclipai context set --api-base http://localhost:3100 --company-id <company-id>
paperclipai context set --api-key-env-var-name PAPERCLIP_API_KEY
export PAPERCLIP_API_KEY=...
paperclipai context show
paperclipai context use default
```

完整标志列表见[通用选项](common-options.md)，配置文件管理方式见[身份验证](./authentication.md)。

---

## 通用标志

大多数控制平面命令共享以下标志，熟悉一次即可。

| 标志 | 用途 |
|---|---|
| `--data-dir <path>` | 将所有本地状态（配置、上下文、数据库、日志、存储、密钥）与 `~/.paperclip` 隔离。适用于测试实例和工作树。 |
| `--api-base <url>` | 指向远程或非默认服务器。 |
| `--api-key <token>` | 不使用配置文件，直接进行身份验证。 |
| `--context <path>` | 读取和写入指定上下文文件。 |
| `--profile <name>` | 选择要使用的上下文配置文件。 |
| `--json` | 输出机器可读格式，供脚本处理。 |

按公司限定范围的命令还接受 `--company-id <id>`（部分命令如 `run` 提供简写 `-C`）。脚本和 `--json` 模式的示例见[输出和脚本](output-and-scripting.md)。

---

## 最快捷的操作流程

**全新本地安装：**

```sh
paperclipai onboard
paperclipai run
```

**实例已经运行，只想开始操作：** 跳过初始化，连接后即可使用。

```sh
paperclipai connect
paperclipai company list
```

**将操作交给 AI：** 一次性配置智能体凭据和技能。

```sh
paperclipai agent local-cli <agent-id> -C <company-id>
```

---

## 参考文档索引

每个命令组都有对应页面。

**设置和实例**

- [安装](installation.md)——获取二进制文件和仓库内开发别名。
- [设置命令](./setup-commands.md)——`onboard`、`doctor`、`configure`、`env`、`allowed-hostname`、`db:backup`、`run`。
- [开发环境](./dev-environments.md)——`env-lab` 和 `worktree` 工具。
- [通用选项](common-options.md)——共享标志、配置文件和 data-dir。
- [输出和脚本](output-and-scripting.md)——`--json` 和自动化模式。

**身份**

- [身份验证](./authentication.md)——`connect`、`auth`、`token`、`context`。
- [访问](./access.md)——访问控制。

**组织**

- [公司](./company.md) · [项目](./project.md) · [目标](./goal.md) · [任务](./issue.md) · [智能体](./agent.md)

**驱动和观察工作**

- [提示](./prompt.md)——向智能体移交任务。
- [运行](./run.md)——检查和控制心跳运行。
- [例程](./routine.md)——定期执行的计划工作。
- [审批](./approval.md) · [活动](./activity.md) · [控制面板](./dashboard.md) · [费用](./cost.md)

**平台和扩展**

- [工作区](./workspace.md) · [适配器](./adapter.md) · [资源](./asset.md) · [技能](./skills.md) · [密钥](./secrets.md) · [云端](./cloud.md) · [插件](./plugin.md) · [反馈](./feedback.md)

---

## 操作指南

如果你更想按流程学习，而不是查阅参考手册，以下产品指南从界面角度介绍相同内容；其中每个操作都对应上方的 CLI 命令组：

- 入门：[安装](../../guides/getting-started/installation.md) · [创建第一家公司](../../guides/getting-started/your-first-company.md) · [聘用第一个智能体](../../guides/getting-started/your-first-agent.md)
- 日常使用：[控制面板](../../guides/day-to-day/dashboard.md) · [任务](../../guides/day-to-day/issues.md) · [审批](../../guides/day-to-day/approvals.md) · [费用和预算](../../guides/day-to-day/costs.md)
- 组织：[组织结构](../../guides/org/org-structure.md) · [委派](../../guides/org/delegation.md) · [项目](../../guides/projects-workflow/projects.md) · [目标](../../guides/projects-workflow/goals.md) · [例程](../../guides/projects-workflow/routines.md)
- 凭据：[CLI 身份验证](../../administration/cli-auth.md)

从终端管理公司时，请结合上方的命令参考和这些指南使用。

---

## 后续步骤

- 初次使用？先阅读[安装](installation.md)，然后通过[身份验证和令牌](./authentication.md)连接。
- 想要无头运行？[安装](installation.md)介绍了无需浏览器的 `onboard` → `run` 流程。
- 正在构建公司？先阅读[创建第一家公司](../../guides/getting-started/your-first-company.md)，再使用[公司](./company.md)、[项目](./project.md)和[任务](./issue.md)命令开展工作。
