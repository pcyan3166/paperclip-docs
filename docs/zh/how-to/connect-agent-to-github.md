---
paperclip_version: v2026.1012.0
seo_title: 将智能体连接到 GitHub 并打开 PR
seo_description: 将编码智能体连接到真实仓库，使您分配的每个问题都能生成一个分支、一个可用的提交以及一个在双方可见的拉取请求。
---

# 将智能体连接到 GitHub 仓库并让其打开 PR

将 `claude_local`（或 `codex_local`）编码智能体连接到真实的 GitHub 仓库，使您分配的每个问题都能生成一个分支、一个可用的提交以及一个拉取请求——这些内容在 Paperclip 问题线程和 GitHub PR 页面中均可见。在一个全新仓库上端到端完成此操作大约需要 20 分钟。

这是每个“智能体连夜交付 PR”故事背后的配方。其中的活动部件都很平常：一个指向正确 git 远程的项目工作区、一个智能体可用于推送的凭据、一个告知智能体在 PR 中工作的 `AGENTS.md`，以及 Paperclip 的审查阶段与 GitHub 的审查相匹配。

---

## 架构

```txt
        Paperclip issue                              GitHub
   ┌──────────────────────┐                  ┌─────────────────────┐
   │ assign to coder      │                  │ branch + commits    │
   │ status: in_progress  │   git push +     │ PR opened by agent  │
   │ comments mirror PR   │◀──gh pr create──▶│ CI checks run       │
   │ status: in_review    │   API webhook    │ reviewer approves   │
   │ status: done         │                  │ PR merged           │
   └──────────┬───────────┘                  └──────────┬──────────┘
              │                                          │
              └────────── isolated git worktree ─────────┘
                          (one per Paperclip issue)
```

当问题被检出时，Paperclip 会配置一个**隔离的 git worktree**。该 worktree 是一个普通的 `git` 克隆，拥有自己的分支——智能体以与您相同的方式运行 `git`、`gh`、您的 linter 和测试套件。推送将发送到您在项目工作区上配置的 upstream 远程。

---

## 1. 先决条件

- 在本地或您控制的服务器上运行 Paperclip。参见 [安装](../guides/getting-started/installation.md)。
- 已雇佣一个编码智能体——安装了 Claude Code 的 `claude_local`，或 `codex_local`。参见 [雇佣您的第一个智能体](../guides/getting-started/your-first-agent.md)。
- 一个您拥有推送权限的 GitHub 仓库。该配方假设您可以为其颁发个人访问令牌或安装 GitHub App。
- 在运行智能体的主机上安装了 `gh` CLI（智能体使用它来打开 PR，而无需手动处理 REST API）。

---

## 2. 配置项目工作区

执行工作区为智能体提供其 `cwd`、upstream `repoUrl` 以及用于分支的基础分支。请将其设置在项目上，而非智能体上——这样，分配给该项目的每个智能体都会继承相同的 git 上下文。

打开 **Projects → {project}** 并设置或创建一个工作区，包含以下内容：

| 字段 | 值 | 备注 |
|---|---|---|
| `cwd` | `/Users/me/work/acme-api` | 主机上的绝对路径。Paperclip 将其用作 worktree 根目录。 |
| `repoUrl` | `https://github.com/acme/api.git` | HTTPS 克隆 URL。UI 使用它进行外部链接，worktree 使用它设置 upstream。 |
| `repoRef` (base) | `main` | 新 worktree 从中分支的分支。 |
| `isPrimary` | `true` | 第一个工作区会自动成为主工作区。 |
| 工作区模式 | **Isolated** | 每个问题一个 git worktree。这是并行 PR 无冲突工作的关键。 |

API 等效操作（两次调用——工作区本身，以及声明“每个问题使用新 worktree”的项目策略）：

```bash
# 1. Create or update the project workspace
curl -X POST "$PAPERCLIP_API_URL/api/projects/$PROJECT_ID/workspaces" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "acme-api-main",
    "cwd": "/Users/me/work/acme-api",
    "repoUrl": "https://github.com/acme/api.git",
    "repoRef": "main",
    "isPrimary": true
  }'

# 2. Set the project's execution-workspace policy so every issue gets its own worktree
curl -X PATCH "$PAPERCLIP_API_URL/api/projects/$PROJECT_ID" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "executionWorkspacePolicy": {
      "enabled": true,
      "allowIssueOverride": true,
      "workspaceStrategy": {
        "type": "git_worktree",
        "baseRef": "main"
      }
    }
  }'
```

第二次调用实际上是开启按问题隔离的功能。如果没有 `executionWorkspacePolicy.workspaceStrategy.type = "git_worktree"`，每个智能体都将针对项目的主检出运行，您将失去并行 PR 的能力。UI 上项目中的 **Workspace mode: Isolated** 切换开关执行相同操作——请选择您喜欢的界面。

> **分支命名。** Paperclip 根据问题标识符为按问题的 worktree 分支命名——例如 `PAP-1802-workspace`。您无需直接设置此项；让 worktree 提供者选择。如果需要不同的约定，请在上述策略中设置 `workspaceStrategy.branchTemplate`，或在单个执行工作区上覆盖 `branchName`。完整字段列表参见 [工作区](../guides/projects-workflow/workspaces.md#workspace-settings)。

---

## 3. GitHub 身份验证

智能体需要通过 HTTPS 执行两项操作：向远程 `git push`，以及针对 API 的 `gh pr create`。两者都需要智能体进程可以读取的凭据。有两个合理的选择——选择一个并接受其权衡。

### 选项 A — 个人访问令牌 (PAT) — 快速路径

针对单个仓库范围的细粒度 PAT 是阻力最小的路径。

1. **GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token。**
2. 资源所有者：拥有该仓库的组织。仓库访问权限：**Only select repositories**，选择该单个仓库。
3. 权限：**Contents: Read and write**，**Pull requests: Read and write**，**Metadata: Read-only**（自动设置）。
4. 生成并复制 `github_pat_…` 值一次。

将其存储为智能体环境中的 Paperclip 密钥，以免其停留在 JSON 文件中：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/secrets" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "GITHUB_TOKEN_ACME",
    "value": "github_pat_..."
  }'
```

然后在编码智能体的适配器配置中引用该密钥——与 [Claude Code → 示例](../../reference/adapters/claude-code.md#example) 中使用的相同 `env` 块：

```json
"env": {
  "GH_TOKEN":     { "type": "secret_ref", "secretId": "<secret-id>", "version": "latest" },
  "GITHUB_TOKEN": { "type": "secret_ref", "secretId": "<secret-id>", "version": "latest" }
}
```

`gh` 读取 `GH_TOKEN`；`git` 的 HTTPS 助手读取 `GITHUB_TOKEN`。设置两者意味着 `git push` 无需额外的凭据助手即可工作。

**权衡：** PAT 与用户绑定。如果该用户离开，令牌将失效，智能体将无法推送。它们也容易脱离库存管理——六个月后，没人记得哪个 PAT 属于哪个智能体。

### 选项 B — GitHub App — 持久路径

安装在组织上的 GitHub App 颁发短期安装令牌，不绑定任何特定人类，并在 PR 上显示为其自身的作者（“acme-paperclip[bot] opened this pull request”）。

1. **Org Settings → Developer settings → GitHub Apps → New GitHub App。**
2. 权限：**Contents: Read and write**，**Pull requests: Read and write**。无需 webhook。
3. 生成私钥，下载 `.pem`。记录目标仓库的 App ID 和安装 ID。
4. 使用 `gh auth token --hostname github.com`（App 感知）或使用 `actions/create-github-app-token` 逻辑的小脚本来铸造安装令牌。

将 App ID、安装 ID 和私钥存储为 Paperclip 密钥，并让智能体在每次运行开始时通过心跳将其交换为安装令牌。令牌有效期为一小时，这恰好足够一次心跳使用。

**权衡：** 活动部件更多，需要一次性设置，额外花费 15 分钟。一旦您拥有多个编码智能体或关心审计追踪，这就值得了。

对于运行几个智能体的单开发人员团队，选项 A 就足够了。对于任何看起来像真实团队的情况，请直接使用选项 B——或下方的选项 C，它将令牌完全从您手中移除。

### 选项 C — 托管 GitHub 连接 (Connectors) — 持久，无需轮换

如果您通过 **Connectors**（Paperclip 的应用程序库，通过 Paperclip Cloud 代理）连接 GitHub，Paperclip 将为每次运行自行解析 GitHub 凭据——您无需铸造、存储或轮换 PAT。每个智能体（或人员）获得一个持久的 GitHub 身份，Paperclip 在运行时将其交换为短期令牌，并投影到智能体运行的沙箱中。

有两点使此选项最稳健：

- **提交自动以连接的 GitHub 身份为作者。** Paperclip 将 `user.name`、`user.email` 以及 `GIT_AUTHOR_*` / `GIT_COMMITTER_*` 环境设置为连接的登录名及其 GitHub `…@users.noreply.github.com` 地址——因此 PR 显示正确的作者，无需 `git config` 步骤（对比 [故障排除](#7-troubleshooting) 中的最后一项）。
- **它默认失败关闭。** 一旦为智能体配置了 GitHub 连接，该智能体将使用它——Paperclip 不会静默回退到 PAT。一次运行必须解析出恰好一个身份；如果没有或超过一个匹配，运行将被阻止并给出明确原因，而不是以错误账户推送。

Paperclip 还通过此连接接收 GitHub webhooks——参见 [审查工作流](#6-review-workflow) 了解合并的 PR 如何自动流回问题。

设置位于 **Connectors → Connections** 页面，而非 `env` 块中。确切的注册步骤取决于您的 Paperclip Cloud 设置，因此请遵循那里的产品内流程。

> **服务器如何解析凭据。** 当 Paperclip 代表您为项目执行 git 操作——克隆托管检出、刷新基础分支——它按以下顺序解析凭据：首先是托管 GitHub 连接，然后是名为 `GITHUB_TOKEN`、`GH_TOKEN` 或 `PAPERCLIP_GITHUB_TOKEN` 的公司密钥（**Settings → Secrets**），最后是服务器进程环境。令牌通过凭据助手传递给 git，绝不写入磁盘、URL 或命令行。

### 主机上的 `gh auth login` 怎么办？

如果您已在运行智能体的机器上交互式运行过 `gh auth login`，本地 `gh` 密钥环中将有可用凭据，智能体将透明地使用它——无需环境变量管道。这是大多数贡献者在首次冒烟测试中遇到的情况，也是 HT3 验证运行使用的路径（真实 PR：[paperclipai/paperclip-docs#1](https://github.com/paperclipai/paperclip-docs/pull/1)）。

这对于**单主机上的本地开发**是可以的。但对于以下情况则不行：

- **远程 / 容器化部署。** `~/.config/gh/hosts.yml` 不会进入 Docker 镜像；智能体将以 `gh: not authenticated` 失败。请使用选项 A 或 B。
- **同一主机上的多个编码智能体。** 它们都继承同一人类用户的身份验证，这使得审计日志难以阅读，并将所有内容绑定到一个人的会话。
- **任何您希望在不注销该人类的情况下撤销的操作。** PAT 或 App 可在几秒内撤销；`gh auth logout` 则更笨重。

将主机密钥环身份验证视为冒烟测试选项。一旦您拥有第二个智能体或离开笔记本电脑，请立即升级到选项 A。

### 凭据预检在工作前捕获缺失的令牌

在智能体实现并审查更改*之后*才被告知令牌缺失，是听到这个消息的最糟糕时机。因此，在本地、git 敏感适配器上分发运行之前，Paperclip 会检查该运行是否需要推送凭据——如果需要，检查 `GH_TOKEN` 或 `GITHUB_TOKEN` 是否实际绑定在智能体或项目范围。如果两者都没有，运行将在前端被阻止，原因为 `push_write_credential_missing` 并附带此补救措施：

> GitHub PR 工作流需要在项目或智能体范围绑定 GH_TOKEN 或 GITHUB_TOKEN。

运行被标记为需要推送凭据有两种方式：

1. **问题提及了 GitHub PR 工作流技能。** 这是显式信号。
2. **问题的标题或描述以纯文本陈述了 PR 交付物。** 例程创建的任务和智能体到智能体的交接很少命名该技能，即使文本字面上说推送分支并打开 PR——因此 Paperclip 也会读取文本。

文本检查故意保守：它寻找与交付物相关的**动词**——打开、创建、提出或提交拉取请求，或推送分支或推送到远程。像“昨天合并的 PR”或“已处理 PR 反馈”这样的通过性提及不会触发它，“push back on the upstream dependency change”也不会。

这意味着措辞为*“修复 null org id 崩溃，然后打开拉取请求”*的任务会在花费任何运行之前向您索取令牌。如果您确实遇到误报——一个配置不完整的阻止器在真正不需要令牌的工作上索取 GitHub 令牌——请绑定令牌或重新措辞任务；无论哪种情况，阻止器都会命名确切的补救措施。

---

## 4. 智能体指令：PR 驱动的工作

智能体的 `AGENTS.md` 是将“执行任务”转变为“在分支上执行任务并打开 PR”的关键。您不需要特殊模板——您需要入口文件中的三条规则，智能体在每次心跳时都会读取。完整捆绑布局参见 [智能体 → 推荐捆绑结构](../guides/org/agents.md#recommended-bundle-structure-agents-soul-heartbeat-tools)；此片段可放入任何编码角色的 `AGENTS.md` 的工作规则部分。

```md
## Working rules: PR-driven

You always work on a feature branch, never on `main` directly. Paperclip provisions an isolated git worktree per issue; that worktree is already on the right branch when you check the issue out.

Before you exit a heartbeat:

1. **Commit your changes.** One conventional-commit message per logical change.
   `git add -A && git commit -m "fix(api): handle null org ids on signup"`
2. **Push the branch.**
   `git push -u origin HEAD`
3. **Open a PR if one doesn't exist.** Use `gh pr create --fill --base main` and copy the resulting URL into the issue thread as the first line of your status comment.
4. **Move the issue to `in_review`** when the PR is up and CI is green. Leave a comment with the PR link, a one-paragraph summary of what changed, and the test commands you ran.

If CI fails, do not merge. Re-open the task, fix the failure, push another commit on the same branch, and tell the reviewer in a new comment what changed.

Never use `--no-verify`, `git push --force` to a shared branch, or `gh pr merge --admin`. If you can't pass CI, mark the issue `blocked` and name the failing check.
```

捆绑 `paperclip-create-agent` 技能中的 CTO 和 CEO 模板已包含这些规则的编码变体——在重新发明之前请检查那里。

---

## 5. 分配任务并观察其落地

在真实仓库上的一次往返，从头到尾：

```bash
# Create the issue and assign the coder agent
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/issues" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Reject signup when orgId is null",
    "description": "Add a 422 response from POST /signup if orgId is missing.",
    "projectId": "'"$PROJECT_ID"'",
    "assigneeAgentId": "'"$CODER_AGENT_ID"'",
    "priority": "medium"
  }'
```

在一两次心跳内：

- 智能体检出问题——Paperclip 在 `/Users/me/work/acme-api/.paperclip-worktrees/PAP-1802-workspace` 处配置 worktree 并将其切换到从 `main` 分出的新分支。
- 智能体编辑正确的文件，运行测试套件，提交并推送。
- 智能体调用 `gh pr create --fill --base main` 并将生成的 `https://github.com/acme/api/pull/142` 粘贴到 Paperclip 问题线程中。
- 问题移动到 `in_review`。

如果您在磁盘上看到 worktree 路径但没有提交，这几乎总是缺少 `GITHUB_TOKEN`——参见下方的 [故障排除](#troubleshooting)。

---

## 6. 审查工作流

您有两个并行的审查界面。将它们用于不同的事情——它们不是冗余的。

| 界面 | 您在此处执行的操作 |
|---|---|
| **GitHub PR** | 阅读差异，逐行评论，监视 CI，通过 GitHub 审查 UI 请求更改。编码智能体已配置（步骤 4）为拒绝自行合并——由人类或高级智能体合并。 |
| **Paperclip 问题** | 跟踪状态（`in_review` → `done`），将批准管道化到 Slack，将 PR 链接到目标/项目，门控工作流中的下一步。 |

当 PR 合并时：

- 人类（或您的 CI 自动合并机器人）通过 GitHub 合并。
- 如果仓库通过**托管 GitHub 连接**（第 3 节，选项 C）连接，Paperclip 接收 `pull_request` webhook 并自行保持链接 PR 的状态同步——open 翻转为 merged 或 closed，合并的 PR 在问题线程中得到确认。审查者仍拥有最终的 `done` 转换，但您不再需要轮询 GitHub 来注意合并。
- 使用 PAT 或 GitHub App 身份验证时没有 webhook，因此审查智能体——CTO、高级工程师或您——将 Paperclip 问题移动到 `done` 并附带包含合并 SHA 的评论。如果您希望在没有托管连接的情况下自动化此操作，请设置一个监听 `pull_request.closed` 并使用 `merged: true` 的 [GitHub → Paperclip 例程 webhook](./wire-slack-discord-notifications.md#testing-the-loop) 来 PATCH 链接的问题。这是一个 20 行的脚本。

如果 PR 被拒绝，将问题更改回 `in_progress` 并附带评论说明需要修复什么。智能体在下次心跳时拾取它，向同一分支推送后续提交，PR 重新运行 CI。

如果您的团队使用 Paperclip 的 [执行策略](../guides/power/execution-policy.md) 进行审查阶段，请在项目上设置策略，以便智能体提交且审查者批准时，问题自动通过 `engineer → reviewer → done`。

---

## 7. 故障排除

**推送时出现 `Permission denied (publickey)`。**
worktree 的远程是 HTTPS，但智能体环境未设置 `GITHUB_TOKEN`。检查智能体的适配器配置并确认 `env` 块解析了密钥。此处的 hello-world 测试：从心跳运行的同一 shell 中执行 `cd <worktree> && env | grep -E 'GH_TOKEN|GITHUB_TOKEN'`。

**运行从未开始——它被 `push_write_credential_missing` 阻止。**
预检决定此运行需要推送凭据，但无法在智能体或项目范围找到绑定的 `GH_TOKEN` 或 `GITHUB_TOKEN`。绑定一个（步骤 3）即可分发运行。如果任务确实不需要推送，检查其标题或描述是否读起来像 PR 交付物——这是第二个触发器。

**`gh: command not found`。**
智能体主机未安装 GitHub CLI。`brew install gh` 或从 [官方发布版](https://github.com/cli/cli/releases) 拉取。Claude Code 适配器不捆绑 `gh`——它必须在 `PATH` 上供心跳进程使用。

**智能体分支上的合并冲突。**
编码智能体应拒绝盲目解决冲突。正确的循环是：智能体检测到冲突 → 将问题标记为 `blocked` 并附带评论命名冲突文件 → 人类（或高级审查智能体）变基或解决。不要让智能体运行 `git checkout --theirs` 来“让冲突消失”。

**PR 上 CI 失败，但智能体仍将问题移动到 `in_review`。**
收紧 `AGENTS.md`（步骤 4）中的规则：“仅在 `gh pr checks --watch` 退出 0 后移动到 `in_review`”。重新提示已切割 PR 的智能体。

**PR 显示提交作者为 `noreply@github.com` 而非智能体身份。**
在项目工作区的 worktree 配置命令中设置 `git config user.name` 和 `user.email`。许多团队使用 `Paperclip Coder <noreply@paperclip.example.com>` 以便 PR 明显为机器作者。如果您使用托管 GitHub 连接（第 3 节，选项 C），Paperclip 已用连接的 GitHub 身份及其 `…@users.noreply.github.com` 地址为每个提交盖章，因此您可以跳过此步。

**智能体每次心跳都创建新分支而不是重用现有分支。**
这是工作区模式问题——项目设置为 **Project primary** 而非 **Isolated**。请切换。参见 [工作区 → 工作区模式](../guides/projects-workflow/workspaces.md#workspace-modes)。

对于更深的心跳级调试——智能体未唤醒，心跳触发但无动作——参见 [调试卡住的心跳](./debug-stuck-heartbeat.md)。

---

## 另请参阅

- [工作区](../guides/projects-workflow/workspaces.md) — `cwd`、`repoUrl`、`branchName` 和 worktree 生命周期的完整参考。
- [智能体 → 推荐捆绑结构](../guides/org/agents.md#recommended-bundle-structure-agents-soul-heartbeat-tools) — `AGENTS.md` 如何与 `SOUL.md`、`HEARTBEAT.md` 和 `TOOLS.md` 协同工作。
- [Claude Code](../../reference/adapters/claude-code.md) — 适配器字段、环境变量和会话持久性。
- [Codex](../../reference/adapters/codex.md) — 相同配方，使用 OpenAI 的 Codex CLI。
- [执行策略](../guides/power/execution-policy.md) — 具有命名参与者的分阶段审查。
- [调试卡住的心跳](./debug-stuck-heartbeat.md) — 智能体不推送时的第一站。
- [处理招聘的董事会批准](./handle-board-approvals-for-hires.md) — 何时要求对获得仓库写权限的新智能体进行批准。
- [为智能体添加 MCP 服务器](./add-mcp-server-to-agent.md) — 作为 `gh`（智能体调用 GitHub MCP 工具而非 shell out）的 MCP 驱动替代方案。
