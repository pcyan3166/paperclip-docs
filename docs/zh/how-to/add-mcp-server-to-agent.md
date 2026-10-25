---
seo_title: 为智能体添加 MCP 服务器
seo_description: 挂载一个模型上下文协议（MCP）服务器，让单个智能体获取新工具——查询 Postgres、搜索向量索引、提交问题——而无需修改 Paperclip。
---

# 为智能体的工具包添加 MCP 服务器

将模型上下文协议（MCP）服务器挂载到特定的 Paperclip 智能体上，使其获取新工具——读取本地 Postgres、查询向量索引、提交 GitHub 问题——而无需修改 Paperclip 本身。对于本地服务器，在一个全新的智能体上端到端完成大约需要 10 分钟；对于远程服务器，大约需要 20 分钟。

心智模型保持不变：Paperclip 是控制平面，适配器启动运行时，运行时才是与 MCP 通信的实体。因此，接线工作位于适配器和运行时层——Paperclip 的职责是限定哪个智能体获取哪个适配器配置，并在运行查看器中展示由此产生的工具列表。

---

## 什么是 MCP

[模型上下文协议](https://modelcontextprotocol.io)（另见：[术语表条目](../guides/welcome/glossary.md#m)）是一个开放标准，用于通过小型 JSON-RPC 契约将语言模型运行时连接到外部工具和数据源。MCP 服务器通过 `stdio`（本地）或 HTTP/SSE（远程）暴露一组工具（以及可选的资源提示）。支持 MCP 的运行时——如 Claude Code 或 Hermes 智能体——会列出这些工具，并允许智能体在运行过程中调用它们。

对于 Paperclip 而言，MCP 是赋予智能体某种能力的方式，这种能力不适合用 [技能](../../reference/skills.md)（它是 Markdown 捆绑包，而非可执行文件）来实现，且你不想教智能体将其作为原始 HTTP 调用。如果工具已经支持 MCP，请优先选择此路径。

> **当前的适配器支持情况。** [Claude Code](../../reference/adapters/claude-code.md) 和 [Hermes](../../reference/adapters/hermes.md#tools) 是目前两个拥有文档记录的 MCP 路径的适配器。Claude Code 继承 Claude Code 的 MCP 客户端（在 Claude Code 层面配置）；Hermes 通过其 `toolsets` 字段暴露 MCP。Codex、Cursor Local、Gemini CLI、OpenCode 和 Pi 遵循与 Claude Code 相同的模式——适配器继承其 CLI 自带的任何 MCP 支持——但该路径尚未在此处记录。

---

## 架构

```txt
   ┌────────────────────┐    Paperclip checkout    ┌──────────────────────┐
   │ Paperclip control  │─────────────────────────▶│  Adapter (claude_/   │
   │ plane (this agent) │                          │  hermes_local)       │
   └────────────────────┘                          └──────────┬───────────┘
                                                              │ launches
                                                              ▼
                                                  ┌──────────────────────┐
                                                  │ Runtime (Claude Code │
                                                  │ or Hermes Agent)     │
                                                  └──────────┬───────────┘
                                                             │ MCP JSON-RPC
                                          ┌──────────────────┴──────────────────┐
                                          ▼                                     ▼
                                ┌──────────────────┐                ┌──────────────────────┐
                                │ Local MCP server │                │ Remote MCP server    │
                                │ (stdio process)  │                │ (HTTP/SSE + OAuth)   │
                                └──────────────────┘                └──────────────────────┘
```

智能体从不直接说 MCP——其运行时才说。这意味着 MCP 服务器配置是每运行时独立的，而*范围限定*（哪个智能体看到哪个服务器）取决于智能体的适配器配置和运行时的配置文件位置，而不是 Paperclip 层面的开关。

---

## 1. 先决条件

- 在本地或您控制的服务器上运行 Paperclip。参见 [安装](../guides/getting-started/installation.md)。
- 已使用受支持的适配器之一雇佣了一个智能体。参见 [雇佣您的第一个智能体](../guides/getting-started/your-first-agent.md)。
- 对于 Claude Code：在运行智能体的主机上安装了 [Claude Code CLI](https://docs.claude.com/en/docs/claude-code)。
- 对于 Hermes：安装了 [Hermes 智能体](https://github.com/NousResearch/hermes-agent)（`pip install hermes-agent`）。
- 您想要挂载的 MCP 服务器。[Anthropic 的目录](https://github.com/modelcontextprotocol/servers) 和 [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers) 是两个最好的起点。

---

## 2. 本地 MCP 服务器 (stdio)

本地 stdio 服务器是最简单的情况：运行时生成的子进程，通过 stdin/stdout 上的 JSON-RPC 进行通信。对于在与智能体相同主机上运行的任何内容，请使用此方式——文件系统读取器、Postgres 桥接器、向量存储代理。

### 路径 A — Claude Code

Claude Code 将 MCP 服务器配置保存在三个可能的范围中（优先级：当同名定义两次时，local > project > user）：

| 范围 | 存储位置 | 谁可以看到 |
|---|---|---|
| **Local** *(默认)* | 当前项目路径下的 `~/.claude.json` | 仅您自己，且仅当从该项目运行 Claude Code 时。不提交到 git。 |
| **Project** | `<repo-root>/.mcp.json` | 任何使用 `cwd` 运行 Claude Code 的人。提交到源代码控制。 |
| **User** | `~/.claude.json` 用户级 | 该 OS 用户运行的所有 Claude Code 运行，跨越所有项目。 |

对于 Paperclip，**优先使用 project 范围**：Claude Code 的 `cwd` 已将智能体固定到特定的工作目录，且该目录中的 `.mcp.json` 随项目一起分发，因此指向相同 `cwd` 的其他贡献者和其他智能体将获得相同的服务器。

使用 Claude CLI 添加服务器（在主机上运行，在智能体的 `cwd` 中）。所有 `claude mcp add` 选项位于服务器名称之前，然后 `--` 将名称与传递给 MCP 服务器的命令 + 参数分隔开：

```bash
cd /Users/me/projects/paperclip-workspace      # whatever cwd the agent uses
claude mcp add --transport stdio --scope project filesystem -- \
  npx -y @modelcontextprotocol/server-filesystem /Users/me/projects/paperclip-workspace
```

这会在项目根目录写入一个 `.mcp.json`，看起来像这样：

```json
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/me/projects/paperclip-workspace"
      ],
      "env": {}
    }
  }
}
```

> **为什么范围对 Paperclip 很重要。** 共享项目工作区的两个智能体——例如 CTO 和程序员都指向同一个仓库——都将看到 project 范围的 MCP 服务器。这几乎总是您想要的。如果您需要一个 MCP 服务器仅对*一个*智能体可见，请使用每智能体 `cwd`（使项目文件对该智能体唯一），或使用 [local 范围](#path-a--claude-code) 并仅在运行该智能体的心跳主机上手动添加。

通过运行一次性探针来验证服务器是否已加载，使用与心跳相同的环境。Claude Code 的 `claude mcp list` 读取智能体在运行时将看到的相同配置：

```bash
cd /Users/me/projects/paperclip-workspace
claude mcp list
```

对于健康的 stdio 服务器，输出看起来像 `filesystem: npx -y @modelcontextprotocol/server-filesystem … - ✓ Connected`（HTTP/SSE 服务器获得 `(HTTP)` / `(SSE)` 标签），对于损坏的服务器则为 `… - ✗ Failed to connect`。如果显示失败，则命令不在 `PATH` 上——在分配任何依赖它的任务给智能体之前，先修复此问题。

### 路径 B — Hermes

Hermes 智能体具有内置的 MCP 客户端，由适配器的 `toolsets` 字段控制。在智能体的适配器配置上启用它，然后在 `~/.hermes/config.yaml` 中将 Hermes 指向实际的服务器。

适配器配置（Paperclip 侧）：

```json
{
  "adapterType": "hermes_local",
  "adapterConfig": {
    "model": "anthropic/claude-sonnet-4",
    "toolsets": "terminal,file,mcp",
    "persistSession": true
  }
}
```

在 `toolsets` 中包含 `mcp` 是告诉 Hermes 启动其 MCP 客户端的方式。如果没有它，Hermes 将忽略其 YAML 中配置的任何服务器——当您想要暂时进行无 MCP 运行时很有用。

Hermes 配置（`~/.hermes/config.yaml`，在运行智能体的主机上）：

```yaml
mcp_servers:
  filesystem:
    command: npx
    args:
      - "-y"
      - "@modelcontextprotocol/server-filesystem"
      - "/Users/me/projects/paperclip-workspace"
```

> **Hermes 服务器注册是主机范围的。** YAML 是每 OS 用户的，而不是每智能体的。如果您在同一主机上运行多个 Hermes 智能体并希望每个智能体有不同的 MCP 服务器，请为每个智能体使用不同的 OS 用户，并在该用户下运行心跳进程——这与 Hermes 用于其会话内存的隔离模式相同。有关技能的并行模式，参见 [Hermes → 技能集成](../../reference/adapters/hermes.md#skills-integration)。

---

## 3. 远程 MCP 服务器 (HTTP/SSE + OAuth)

远程服务器运行在其他地方——托管的 GitHub MCP、供应商的分析端点、您团队的内部 MCP 网关。它们通过 HTTP（可流式 HTTP）或 SSE 使用相同的 JSON-RPC 契约，大多数生产级别的服务器通过 OAuth 2.1 和 [PKCE](https://datatracker.ietf.org/doc/html/rfc7636) 进行身份验证。

Paperclip 的棘手之处在于 **Paperclip 无头运行运行时**——对于 `claude_local`，那是 `claude --print …`。OAuth 的重定向和粘贴流程至少需要一次交互式 shell 来完成身份验证。可行的模式是：

1. 以*运行心跳的相同 OS 用户*身份交互式添加远程服务器。
2. 走一遍 OAuth 流程。运行时在用户的配置目录中持久化令牌并自动刷新它们。
3. 后续的心跳无头运行——令牌已在磁盘上。

### 路径 A — Claude Code

```bash
# As the OS user the heartbeat runs as:
cd /Users/me/projects/paperclip-workspace
claude mcp add --transport http --scope user github https://api.githubcopilot.com/mcp/

# Then start an interactive Claude session once to complete OAuth:
claude
> /mcp
# Pick "github" from the list, follow the browser redirect, paste the code back.
```

交互式运行后，`~/.claude.json` 包含 GitHub MCP 的可刷新 OAuth 令牌。下一个 Paperclip 心跳会自动拾取该服务器。使用 `claude mcp get github` 进行验证——流程完成后它报告 `OAuth: configured`。

> **此处用户范围是正确的。** 远程 MCP 令牌与授权它们的 OS 用户绑定，因此用户范围（而非项目）反映了现实。不要将 OAuth 令牌提交到 project 范围的 `.mcp.json`——它们不属于 git。

### 路径 B — Hermes

Hermes 的 MCP 客户端支持相同的 HTTP 和 SSE 传输。带有 bearer 身份验证的远程服务器的 YAML 形式（如果供应商提供长期 API 令牌，则完全跳过 OAuth）：

```yaml
mcp_servers:
  github:
    transport: http
    url: https://api.githubcopilot.com/mcp/
    headers:
      Authorization: "Bearer ${GITHUB_MCP_TOKEN}"
```

`${GITHUB_MCP_TOKEN}` 从智能体的环境中解析。将令牌作为 Paperclip 密钥放入智能体的 `env` 块中——切勿将其烘焙到 YAML 中：

```json
"env": {
  "GITHUB_MCP_TOKEN": {
    "type": "secret_ref",
    "secretId": "<secret-id>",
    "version": "latest"
  }
}
```

对于完整的 OAuth（无静态令牌），遵循与 Claude Code 相同的一次性交互式模式——运行 `hermes chat` 一次以完成浏览器流程，然后让后续心跳使用持久化的令牌。

---

## 4. 每智能体范围限定规则

*谁看到哪些 MCP 工具* 的层次结构源自运行时的配置文件优先级，而不是 Paperclip 开关。内化这一点，其余部分就是机械性的：

| 服务器注册位置 | 哪些智能体可以看到 |
|---|---|
| Project `.mcp.json`（已提交；Claude `--scope project`） | 每个 `cwd` 为该项目根目录的智能体。 |
| Local — 项目路径下的 `~/.claude.json`（Claude `--scope local`，默认） | 仅该 OS 用户，且仅当从该 `cwd` 运行时。 |
| User — `~/.claude.json` 用户级（Claude `--scope user`）或 `~/.hermes/config.yaml` | 在该主机上由该 OS 用户运行的每个智能体。 |
| 适配器级配置（例如通过 `HERMES_CONFIG` 指向的每智能体 YAML 中的 `mcp_servers`） | 仅该一个智能体。 |

当同一服务器名称出现在两个范围中时，Claude Code 的优先级是 **local > project > user**——更具体的定义获胜。

实用的经验法则：

- **一个 MCP 服务器，仓库上的所有程序员：** project `.mcp.json`，已提交。审查员和程序员都能获得它。
- **一个 MCP 服务器，仅针对此智能体：** 给智能体一个唯一的 `cwd`，并将服务器放在*该*项目文件中。或者在专用 OS 用户下运行该智能体。
- **一个 MCP 服务器，个人实验：** local 范围（`claude mcp add` 的默认值）或通过 `env` 覆盖的 `HERMES_CONFIG=~/.hermes/agent-X.yaml`。

Paperclip 侧的 CLI 命令名称不变——没有 `paperclipai mcp add`，因为事实来源是运行时的配置。依赖运行时。

---

## 5. 调试：智能体实际拥有哪些工具？

按速度顺序查看三个地方。

### a. 在与心跳相同的 shell 中运行运行时的“list”命令

这是最有用的检查。从智能体的 `cwd`，使用与心跳相同的 `env`：

```bash
# Claude Code
claude mcp list                 # all configured servers + connection status
claude mcp get github           # one server's full details, incl. OAuth status

# Hermes
hermes mcp list
```

`claude mcp list` 每个服务器打印一行：健康为 `<name>: <command-or-url> (<TRANSPORT>) - ✓ Connected`，损坏为 `… - ✗ Failed to connect`。`claude mcp get <name>` 添加解析后的范围、完整的传输配置，以及——对于 OAuth 服务器——是否配置了凭据。如果心跳看到的列表与您的 shell 不同，则环境或 `cwd` 已漂移——有关环境差异工作流，参见 [调试卡住的心跳](./debug-stuck-heartbeat.md)。

### b. 阅读智能体的运行记录

在 Paperclip 中打开智能体并点击最新的运行（智能体 → `<agent>` → 运行）。记录显示智能体进行的每次工具调用，包括 MCP 工具调用（它们以其 `mcp__<server>__<tool>` 命名空间显示）。如果您的工具不在记录中，且智能体有理由使用它，则服务器可能未加载。

此视图即将增加的功能将在运行开始时显示*可用*工具列表（而不仅仅是已调用的）；当该功能发布时，我们将在下方添加截图。目前，“有什么可用？”最好通过 `mcp list` 和一次明确要求智能体“列出您的工具”的快速运行来回答。

### c. 用单行任务进行探针

分配给智能体一个任务，其整个内容为：

```txt
List the MCP tools you have access to. Return a one-line description of each.
Do not call any of them — just enumerate.
```

这迫使运行时将其工具列表实例化到记录中，而不产生任何副作用。如果您期望的工具缺失，您已将故障缩小为“配置未加载”而非“智能体拒绝调用它”。

---

## 6. 实战示例：GitHub MCP 服务器

往返：一个 Claude Code 智能体，给定问题标题和描述，通过 [GitHub MCP 服务器](https://github.com/github/github-mcp-server) 在正确的仓库上打开 GitHub 问题。

### 设置

```bash
# 1. Get a GitHub PAT (or use an existing GH App token) with `issues: write`.
#    Store it as a Paperclip secret bound to the agent.
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/secrets" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "name": "GITHUB_MCP_TOKEN", "value": "github_pat_..." }'

# 2. Reference the secret on the agent's adapter config:
#    "env": {
#      "GITHUB_MCP_TOKEN": { "type": "secret_ref", "secretId": "<id>", "version": "latest" }
#    }

# 3. Add the GitHub MCP server at user scope (so the OAuth-less PAT auth is host-wide).
claude mcp add --transport http --scope user github https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer $GITHUB_MCP_TOKEN"
```

### 验证

```bash
claude mcp list
# github: https://api.githubcopilot.com/mcp/ (HTTP) - ✓ Connected
```

### 从 Paperclip 任务驱动

创建任务并将其分配给智能体：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/issues" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "File a Github issue on acme/api",
    "description": "Open an issue on github.com/acme/api titled \"Reject signup when orgId is null\" with the body \"Add a 422 response from POST /signup if orgId is missing.\" Use the GitHub MCP tool — do not shell out to gh.",
    "assigneeAgentId": "'"$AGENT_ID"'"
  }'
```

在下一个心跳上，智能体使用 `owner=acme`、`repo=api`、标题和正文调用 `mcp__github__create_issue`。记录显示调用和响应。Paperclip 问题移动到 `done`，GitHub 问题 URL 粘贴到评论线程中。

相同的模式可以推广——将 GitHub MCP 替换为 Linear MCP、Postgres MCP、自定义内部 MCP——智能体将在下一个心跳上获取新工具包，而无需重新部署任何内容。

---

## 7. 故障排除

**`mcp list` 显示服务器，但智能体从未调用它。**
运行时看到工具；智能体认为它不相关。要么工具描述太模糊（改进服务器的工具模式），要么智能体的指令未提及该能力。在 `AGENTS.md` 中添加明确提示：“当您需要提交问题时，使用 GitHub MCP 服务器（`mcp__github__*` 工具）而不是 `gh` 或 curl。”

**`claude mcp list` 显示 `✗ Failed to connect`。**
在相同的 shell 中直接运行服务器命令——失败消息会更具体。最常见的原因：二进制文件不在心跳用户的 `PATH` 上，服务器期望的工作目录不存在，或必需的环境变量（API 密钥、数据库 URL）未设置。

**Paperclip 下的工具列表与终端不同。**
心跳的 `cwd` 或 `env` 与您的 shell 不同。比较它们——从智能体的运行内部与从您自己的 shell 运行 `pwd && env | sort`。最常见的差异是 `PATH`：图形启动的进程有时继承比终端会话更精简的 `PATH`，因此 `npx` 或 `node` 缺失。

**远程 MCP 服务器在几小时后返回 `401`。**
OAuth 刷新失败。交互式重新运行 `claude` 以重新授权。如果运行时本应自动刷新但未刷新，请向运行时项目提交错误报告——而不是向 Paperclip——因为令牌缓存位于运行时层。

**共享项目的两个智能体看到彼此的 MCP 服务器。**
这是设计使然——project 范围是共享的。将每智能体服务器移动到 user 范围，或将项目拆分为每智能体 `cwd`s。

**我移除的服务器仍被列出。**
Claude Code 短暂缓存解析后的配置。重启任何长期运行的心跳会话（或只需等待下一个心跳——Paperclip 为 `claude_local` 每次运行启动新的 CLI 调用）。Hermes 默认恢复同一会话；通过切换 `persistSession: false` 一次来强制全新启动。

有关更深的心跳级调试——智能体未唤醒，运行时根本无法启动——参见 [调试卡住的心跳](./debug-stuck-heartbeat.md)。

---

## 另请参阅

- [Claude Code](../../reference/adapters/claude-code.md) — 托管 Claude Code 的适配器，包括会话持久化和技能同步。
- [Hermes — 工具](../../reference/adapters/hermes.md#tools) — `toolsets` 字段，包括 `mcp`。
- [技能参考](../../reference/skills.md) — 当技能捆绑包是比 MCP 服务器更合适的原语时。
- [将智能体连接到 GitHub 仓库](./connect-agent-to-github.md) — PR 驱动的对应部分；将此指南与之配对，用于打开问题*和* PR 的程序员。
- [调试卡住的心跳](./debug-stuck-heartbeat.md) — 当智能体的工具列表看起来不正确时的第一站。
- [MCP 规范](https://modelcontextprotocol.io) — 上游协议文档。
- [MCP 服务器目录](https://github.com/modelcontextprotocol/servers) — 官方和社区服务器，随时可挂载。
