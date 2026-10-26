---
paperclip_version: v2026.1023.0
seo_title: 设置 GitHub 连接器
seo_description: 针对两种 GitHub 路由的分步设置指南——使用托管身份或细粒度令牌访问仓库工具，以及用于从 Issue 进行聊天的 GitHub App。
---

# 设置 GitHub 连接器

由于 GitHub 执行两项互不相关的任务，因此分为两个独立的设置流程。请选择您需要的一项：

- [将 GitHub 设置为智能体工具](#set-up-github-as-an-agent-tool) — 智能体读取并操作仓库。
- [设置来自 GitHub 的聊天](#set-up-chat-or-a-pr-review-bot-from-github) — 用户通过 Issue 和 Pull Request 评论与智能体协作。

关于每种路由的管理范围，以及 shell Git 和 `gh` 的例外情况，请参阅 [GitHub](github.md)。在为智能体授予专用身份之前，请先阅读该例外说明。

<a id="set-up-github-as-an-agent-tool"></a>
## 将 GitHub 设置为智能体工具

### 1. 首先决定身份

决定*哪个*身份以及*哪些*仓库是值得放慢脚步仔细考虑的部分，因为 GitHub 端的范围是限制智能体能力的真正边界。

| 身份 | 适用场景 |
| --- | --- |
| **我的 GitHub 账户** | 您希望智能体以您的身份，在您的工作环境中操作，而您是责任人 |
| **智能体的专用账户** | 智能体应在提交、评论和 Pull Request 中以自身身份出现。需要连接管理器权限 |

对于任何推送代码的操作，专用账户是更好的选择。其操作可追溯，且撤销它不会干扰人员自身的访问权限。

### 2. 启动连接并选择凭据路径

1. 在侧边栏中选择 **Connectors**（连接器）。
2. 找到 **GitHub** 并选择 **Connect**（连接）。
3. 阅读主按钮上方的访问说明行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要更改其中任何一项，请选择 **Change**（更改）：在 **Connect GitHub as**（以...身份连接 GitHub）中回答步骤 1 中确定的身份，并选择 **Any agent**（任何智能体）或 **Only agents I choose**（仅我选择的智能体）。对于专用身份，智能体问题已预先确定——Paperclip 会询问 **Which agent owns this GitHub account?**（哪个智能体拥有此 GitHub 账户？）。
4. 检查凭据路径。当提供托管路径时，设置将从该路径开始；令牌路径作为替代方案位于 **Change**（更改）之下：

| 路径 | 说明 | 提供条件 |
| --- | --- | --- |
| **Connect GitHub** | Paperclip 的托管 GitHub App 授权 | 仅当实例已注册 Paperclip Cloud 且 Cloud 提供了 GitHub 连接器配置文件时 |
| **Personal access token (advanced)** | 您创建并控制的细粒度令牌 | 始终提供 |

**从此时起，这两个分支不共享步骤。** 请遵循 3a 或 3b，不要同时执行。

### 3a. 托管路径 — 在 GitHub 中授权并选择仓库

1. 选择 **Continue to GitHub**（继续前往 GitHub）并进行授权。
2. 在 GitHub 的安装界面上，选择智能体应访问的具体仓库。除非确实有此意图，否则避免选择 **All current and future repositories**（所有当前和未来仓库）——Paperclip 会在仓库行中标记此类安装，因为它会随着组织的增长而自动扩大范围。

返回 Paperclip 后，身份卡片将显示 **Accessible GitHub repositories**（可访问的 GitHub 仓库）。若要稍后更改列表，请使用 **Add More Repos on GitHub**（在 GitHub 上添加更多仓库）或 **Configure access on GitHub**（在 GitHub 上配置访问权限），然后选择 **Refresh access**（刷新访问权限）以便 Paperclip 重新读取安装信息。

此分支是唯一能为 shell Git 和 `gh` 生成持久身份的路径。

### 3b. 令牌路径 — 创建令牌，然后粘贴

此分支没有授权重定向，也没有 GitHub 安装界面。**仓库范围存在于令牌中**，因此您在创建令牌时设置范围，而不是在 Paperclip 中事后设置。

1. 在 GitHub 中，前往 **Settings**（设置） → **Developer settings**（开发者设置） → **Personal access tokens**（个人访问令牌） → **Fine-grained tokens**（细粒度令牌），并选择 **Generate new token**（生成新令牌）。直接链接：[github.com/settings/personal-access-tokens/new](https://github.com/settings/personal-access-tokens/new)。
2. 将 **Resource owner**（资源所有者）设置为拥有仓库的账户或组织。如果是组织，令牌在生效前可能需要组织所有者的批准——请预期这一延迟，而不是假设令牌已损坏。
3. 在 **Repository access**（仓库访问权限）下，选择 **Only select repositories**（仅选择仓库）并精确选择智能体应使用的仓库。
4. 设置 **Expiration**（过期时间）。在此处，令牌过期是一项功能；请记录日期，因为连接将在该日期停止工作。
5. 授予与智能体预期操作相匹配的仓库权限，不多不少。只读的 **Contents**（内容）、**Metadata**（元数据）、**Issues**（问题）和 **Pull requests**（拉取请求）是合理的起点；请有意地、逐个添加写入权限。
6. 生成令牌并复制它。GitHub 仅显示一次。它以 `github_pat_` 开头。
7. 返回 Paperclip，将其粘贴到 **Your GitHub key**（您的 GitHub 密钥）字段中并完成操作。此时连接已完成——无需进一步授权。

> **注意：** 由于权限是您在令牌上勾选的那些，此路径是两种路径中更容易审计的一种。它无法提供的是持久的 shell 身份：为此，请使用托管路径。

若要稍后更改仓库范围，请在 GitHub 中编辑令牌或发行新令牌并重新连接。**Refresh access**（刷新访问权限）会重新读取安装信息，在此处不适用。

### 4. 设置操作权限

在 **Permissions**（权限）选项卡中，逐项检查列表：读取操作设为 **Allowed**（允许）；写入和破坏性操作设为 **Ask first**（先询问）或 **Off**（关闭），直到您观察过智能体的工作表现。

当您在专用 GitHub 身份上更改权限时，Paperclip 会显示 shell 警告。请按照原文阅读：shell Git 和 `gh` 使用该账户，不受这些开关的限制。

### 5. 在 GitHub 上设置真正的护栏

由于 shell 不受操作权限的限制，对于具有推送权限的智能体，重要的控制措施位于 GitHub 端：

- 对于托管 App，授予 **selected repositories**（选定仓库）权限。对于 PAT，在 GitHub 中限制仓库选择和令牌权限。
- 保护默认分支：要求通过 Pull Request，并要求由智能体以外的人员进行审查。
- 要求合并前通过状态检查。

Paperclip 的 [execution policy](../guides/power/execution-policy.md) 也可以要求在 Paperclip Issue 上设置审查阶段，但 Paperclip 审查阶段和 GitHub 分支保护规则是独立的控制措施。如果合并很重要，请同时设置两者。

### 6. 通过读取操作进行验证

以您打算使用的智能体身份，要求它读取预期仓库中的一个已知文件。比较内容并检查任务的连接器调用。分别检查托管 App 安装设置或，对于 PAT，检查其仓库选择和权限。成功的读取验证了该操作，而非整个访问边界。

不要通过推送、Pull Request 或合并来验证。

<a id="set-up-chat-or-a-pr-review-bot-from-github"></a>
## 设置来自 GitHub 的聊天或 PR 审查机器人

> **实验性功能：** 在开始此路径之前，请在实例的实验性设置中启用 **GitHub review bots**（GitHub 审查机器人）。它与 **Chat connectors**（聊天连接器）是分开的。

审查机器人拥有 **Connectors**（连接器）页面上的独立条目 **GitHub Code Review Bot**（GitHub 代码审查机器人），与用于仓库工具的 **GitHub** 条目分开。选择它将直接跳转到选择智能体。

请遵循 [Set up a GitHub review bot](github-review-bot-setup.md) 进行完整设置，包括可选的 Storybook 审查者。它涵盖了选择智能体、从 GitHub 创建和安装 App、仓库访问、个人账户链接以及真实的 PR 测试。新机器人以仅提及模式启动；如果您希望机器人仅响应提及，请保持 **Run automatically**（自动运行）关闭。

如果您正在决定审查是否应自动运行或阻止合并，请先阅读 [Understanding GitHub PR review bots](understanding-github-review-bots.md)。仅安装 App 本身既不会自动运行审查，也不会阻止合并。

## 故障排查

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 缺少托管工具选项 | 未注册 Paperclip Cloud，或未宣传 GitHub 连接器配置文件 | 使用 **Personal access token (advanced)**（个人访问令牌（高级）） |
| 细粒度令牌被拒绝，或无法访问任何内容 | 它指向未批准的组织，或其仓库选择为空 | 检查组织个人访问令牌设置下是否有待批准的请求，并确认令牌选择的仓库 |
| 令牌连接在特定日期停止工作 | 令牌已过期 | 发行具有相同范围的新令牌并重新连接 |
| 聊天设置中未显示 Webhook URL | 实例未配置公共基础 URL | 请管理员进行配置 |
| **Connectors**（连接器）中缺少 **GitHub Code Review Bot** | 实例的 **GitHub review bots**（GitHub 审查机器人）已关闭 | 请管理员启用 |
| 仓库数量少于预期 | 安装未包含这些仓库 | 在 GitHub 上修复，然后 **Refresh access**（刷新访问权限） |
| *"You don't have permission to reconnect this identity."*（您没有权限重新连接此身份。） | 该身份属于他人，或属于另一个智能体 | 联系其所有者，或使用您自己的身份 |
| 工具调用正常但 `git push` 被拒绝 | GitHub 上的账户权限或分支保护 | 在 GitHub 上检查这两项 |
| GitHub App 日志中 Webhook 传递失败 | Webhook URL 或密钥不匹配 | 在 Paperclip 中重新生成密钥并更新 App |
| 评论未创建任务 | App 未安装在该仓库上，或未订阅这两个评论事件 | 在那里安装并检查事件 |

更多详情见 [验证连接器并修复故障连接器](verify-and-troubleshoot.md)。

## 相关指南

- [理解 GitHub PR 审查机器人](understanding-github-review-bots.md)
- [设置 GitHub 审查机器人](github-review-bot-setup.md)

- [GitHub](github.md)
- [将智能体连接到 GitHub 仓库并让其打开 PR](../how-to/connect-agent-to-github.md)
- [为人员和智能体使用独立账户](separate-accounts.md)
- [设置操作权限](action-permissions.md)
