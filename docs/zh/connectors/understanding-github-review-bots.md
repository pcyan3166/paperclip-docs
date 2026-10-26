---
paperclip_version: v2026.1023.0
seo_title: 理解 GitHub PR 审查机器人
seo_description: 了解为何安装审查机器人、选择其运行时机以及在合并前要求通过其检查是三个独立的决策。
---

# 理解 GitHub PR 审查机器人

> **实验性功能：** GitHub 审查机器人拥有独立的实验性设置项 **GitHub 审查机器人**，必须由实例管理员开启。该功能默认关闭，且独立于 **聊天连接器**。关闭此功能会隐藏设置和管理界面；已存在的机器人将继续运行。设置和行为可能会发生变化。现有的 GitHub 工具连接不会自动启用此功能。

机器人可以审查您的 PR，留下评论，并给出 3/5 的评分，而不会阻止任何人合并代码。这可能会让人感到意外：如果审查未通过，为什么 GitHub 仍然允许合并？

缺失的一环在于：**将机器人设为必需的 PR 检查需要单独的 GitHub 仓库设置**。GitHub 允许您获取机器人的反馈，而不必承诺将该反馈作为合并的必要条件。

**安装机器人、决定其运行时机以及在合并前要求通过其检查，是三个独立的选择。**

以下是它们在 Paperclip 中的运作方式：

1. **安装应用：** 授予机器人访问您仓库的权限，并允许 GitHub 向其发送通知。
2. **选择触发器：** 告诉 Paperclip 在哪些通知情况下应触发智能体进行审查。
3. **（可选）要求通过其检查：** 告诉 GitHub 合并操作是否必须等待通过的结果。

在第二个和第三个选择之间还存在一种转换：智能体的“3/5”或“5/5”评分必须转换为 GitHub 能够理解的检查结果。我们也将对此进行说明。

假设我们要为一个将 PR 合并到 `main` 的仓库设置机器人。在启用所有功能之前，我们希望先理解每个选择。完整的入门流程请参阅 [设置 GitHub 审查机器人](github-review-bot-setup.md)——一个基于 Storybook 的机器人实操教程。

## 1. 安装 GitHub 应用赋予机器人访问权限和事件通知

首先，机器人需要一种参与您仓库的方式。您在选定的仓库上安装 GitHub 应用。其权限允许机器人读取代码、发布评论和审查，以及发布检查。

这确立了机器人被允许执行的操作。这并不意味着“审查每个 PR”，也不意味着“没有这个机器人任何人都不能合并”。这些是我们稍后做出的选择。

该应用还有一个指向 Paperclip 的 **Webhook URL**。Webhook 仅仅是 GitHub 在发生某些事件时向该地址发送 HTTP 通知。这是 Paperclip 无需反复询问 GitHub 是否有变化即可获知您 PR 状态的方式。

涉及的通知如下：

| GitHub 上发生的事件 | Paperclip 接收到的 Webhook |
| --- | --- |
| 有人在 Issue 或 PR 的主对话中评论 | `issue_comment` |
| 有人在 PR 审查线程的内联评论中评论 | `pull_request_review_comment` |
| 有人编写或编辑 Issue 描述 | `issues` |
| 有人编写或编辑 PR 描述 | `pull_request` |
| 有人提交带有摘要的 PR 审查 | `pull_request_review` |
| 有人打开一个 PR | `pull_request`，动作参数为 `opened` |
| 有人向现有 PR 推送新提交 | `pull_request`，动作参数为 `synchronize` |

Paperclip 的设置会将应用订阅到所有五种事件类型，因此提及（mention）可以在评论、内联审查评论、Issue 或 PR 描述，或审查摘要中生效。

例如，您写下 `@your-bot please review this PR`。GitHub 将该评论发送到应用的 Webhook，Paperclip 识别出文本中的授权请求。GitHub 没有单独的机制来理解 AI 审查应该做什么；这种解释属于 Paperclip 的职责。

我们的连接器不要求仓库中存在 GitHub Actions 工作流文件。Paperclip 接收事件并在普通的 Paperclip 任务中运行指定的智能体。机器人的应用身份提供其受治理的 GitHub 工具。

GitHub 安装访问权限和 Paperclip 中启用的仓库是独立的控制项：应用需要在 GitHub 上拥有访问权限，且该仓库必须在 Paperclip 中为此机器人启用。

请参阅 [GitHub 的 Webhook 文档](https://docs.github.com/en/webhooks/webhook-events-and-payloads)。

## 2. Paperclip 的配置决定哪些通知启动审查

现在机器人可以获知仓库的活动。下一个选择是哪些通知实际上应启动工作。

这个选择很重要，因为您可能只希望在需要帮助时才获得帮助。或者您可能希望每个符合条件的 PR 在打开时立即被审查。这两种设置使用相同的 GitHub 应用和 Webhook；Paperclip 的配置改变了通知到达后发生的情况。

您可以选择：

- **仅提及：** 必须有人明确要求机器人进行审查。
- **自动审查：** 审查新 PR、更新的提交或其他选定事件。
- **两者兼有：** 自动审查，同时也接受再次审查的请求。

打开 PR、重新打开 PR、标记为准备审查以及推送新提交是独立可配置的事件。自动审查还遵守配置的作者、草稿、分支、标签和仓库限制。

假设我们以 **仅提及** 模式打开我们的示例 PR。GitHub 仍然通知 Paperclip PR 已打开，但 Paperclip 不会启动自动审查。当您添加 `@your-bot please review` 时，该授权请求会启动指定的智能体。

如果您启用 **新 PR 的自动审查**，打开同一个 PR 就会在无人提及机器人的情况下启动审查。您可以单独启用对新推送的审查，或者要求对这些后续评估进行新的提及。

授权的手动请求可以绕过自动调度过滤器。它不能绕过仓库限制、排除的文件或请求者的权限。

在普通 Issue 中的提及会启动任务对话；PR 评分检查针对的是特定的 PR 提交。普通的后续讨论不会改变评分。重复的审查请求会继续 PR 现有的 Paperclip 任务。

<a id="3-the-agents-score-becomes-a-github-check-result"></a>
## 3. 智能体的评分成为 GitHub 检查结果

此时智能体已完成审查。假设它发现了一个问题并给我们的 PR 打了 **3/5**。GitHub 如何处理这个数字？

单独来看，什么也不会发生。**GitHub 没有内置的 5/5 概念。** 它不会阅读机器人的评论并推断“3/5”应该阻止合并。该评分属于 Paperclip 的评估格式和评分策略。

为了使结果能被 GitHub 的合并规则使用，Paperclip 将评估转换为名为 **Paperclip Review** 的 **检查运行（check run）**。这与您在构建或测试结果中看到的 GitHub 对象类型相同。

智能体提交包含已审查提交、评分、发现、理由和覆盖范围的结构化评估。Paperclip 验证结果并将其与配置的阈值进行比较。智能体不直接选择任意的通过检查结果。

当最低评分为 5 时：

```text
Agent submits a complete assessment: 3/5
    → Paperclip validates it
    → Paperclip reports failure to GitHub

Agent submits a complete assessment: 5/5
    → Paperclip validates it
    → Paperclip reports success to GitHub
```

因此，您可以在 PR 上同时看到一条写着“3/5”的评论和一个红色的 **Paperclip Review** 检查。评论向人类解释结果。检查为 GitHub 提供其规则可用的结果。该检查附加到智能体审查的确切提交上。

| 审查情况 | 检查报告的内容 |
| --- | --- |
| 等待智能体 | 排队中 (Queued) |
| 智能体正在审查 | 进行中 (In progress) |
| 完成的评估满足阈值 | 成功 (Success) |
| 完成的评估低于阈值 | 失败 (Failure) |
| 不完整的评估或无法完成的执行 | 需要关注的非通过结果 |
| 报告模式下的完整评估 | 中性 (Neutral)；低分不会导致检查失败 |

阈值默认为 5/5，可设置为 1–5。不完整的评估无法通过。报告模式对于在不强制评分的情况下提供反馈很有用；它不能替代必需的评分阈值。GitHub 可以接受中性检查作为满足必需检查。如果低分必须阻止合并，请使用数值阈值，而不是报告模式。请参阅 [GitHub 的必需检查故障排除指南](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/troubleshooting-required-status-checks)。

对于我们的 Storybook 示例，配置的审查指令要求智能体生成并构建故事，渲染页面，并查找可见的 `oogabooga`。找到该单词的完整运行得 5/5；未找到匹配页面的完整运行得 3/5。构建失败会导致评估不完整。在 5/5 阈值下，只有第一种结果通过。

请参阅 [GitHub 的 Checks API 指南](https://docs.github.com/en/rest/guides/using-the-rest-api-to-interact-with-checks)。

<a id="4-githubs-repository-rules-decide-whether-the-check-blocks-merging"></a>
## 4. GitHub 的仓库规则决定检查是否阻止合并

我们的示例 PR 现在有一个红色的 **Paperclip Review** 检查。我们还有一个选择：该结果是否应阻止合并？

**机器人可以发布失败的检查而不阻止合并。** GitHub 允许检查提供可选反馈。仓库所有者决定哪些检查重要到需要在代码到达 `main` 等分支之前被要求通过。

这就是为什么安装机器人或启用自动审查不会创建合并要求。您可能希望先尝试审查者并检查其反馈，然后再让整个团队的合并依赖于它。

为了使结果成为强制性的，仓库管理员配置一条规则，其含义为：**“合并到 `main` 的 PR 需要通过的 Paperclip Review 检查。”**

典型的设置如下：

1. 安装并配置机器人，然后让它发布第一个 **Paperclip Review** 检查。
2. 打开仓库的 **设置 → 规则 → 规则集 (Settings → Rules → Rulesets)**。
3. 创建或编辑针对您合并目标分支（如 `main`）的分支规则集。
4. 启用 **要求状态检查通过 (Require status checks to pass)** 并添加 **Paperclip Review**。
5. 在可用时选择机器人应用作为预期来源，以确保来自不同集成的结果不满足该要求。
6. 将规则集的强制实施设置为 **Active**，并有意选择绕过权限。

经典的分支保护规则也可以要求该检查。组织规则可能已经对仓库施加了要求。当您安装或配置机器人时，Paperclip 不会自动更改这些 GitHub 规则。

**要求通过检查本身不会启动智能体。** 它告诉 GitHub 等待结果。Paperclip 中的触发器配置仍然决定智能体何时运行。

例如，您可以要求通过检查，同时保持机器人处于仅提及模式。这是一个有效的设置：在 PR 能够满足合并规则之前，必须有人明确要求审查。启用自动审查消除了手动启动步骤；它不会使合并规则更强。

如果没有人触发必需的审查，缺失的结果将阻止受该规则约束的用户进行合并。如果连接器收到 PR 事件，但在其配置的评分策略下不允许自动执行，它可以发布 **需要授权的手动审查 (Authorized manual review required)**。然后，授权的提及可以启动审查。

GitHub 计划和仓库可见性会影响这些规则是否可用。验证规则是否处于活动状态并适用于目标分支；仅可见的检查不能证明合并强制执行。配置的绕过权限仍然适用。

请参阅 [GitHub 规则集的可用规则](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
和 [创建仓库规则集](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository)。

## 四种有用的组合

我们现在可以将两个行为选择并排展示。这就是为什么“机器人审查 PR”本身并不能告诉您它是自动运行还是其结果是强制性的：

| 审查触发器 | GitHub 要求 Paperclip Review？ | 发生的情况 |
| --- | --- | --- |
| 仅提及 | 否 | 当有人请求时进行可选审查。其失败的检查本身不会阻止合并。 |
| 自动 | 否 | 自动反馈。其失败的检查本身不会阻止合并。 |
| 仅提及 | 是 | 合并等待直到有人请求审查且必需检查通过。 |
| 自动 | 是 | 自动运行审查。合并等待必需检查通过。 |

其他仓库规则，包括正式的审查要求，可能在上述任何场景中独立阻止合并。

## 当有人再次推送或再次提及机器人时会发生什么？

假设智能体给我们的 PR 打了 3/5，且必需检查失败。您修复了问题并推送了另一个提交。

GitHub 现在需要对该新代码进行评估。检查结果属于它所评估的提交：即使之前的版本已通过，该结果也不会确立新版本是合格的。

接下来发生什么取决于我们之前做出的触发器选择：

- **启用自动推送审查：** 推送通知启动新审查。
- **禁用推送审查：** 您写下 `@your-bot please review again` 来启动它。

智能体审查新提交，提交完整的 5/5 评分，Paperclip 发布成功的检查。PR 现在已满足 **此** 合并要求。其他必需检查或人工批准可能仍未完成。

重复的审查提及也可以请求对同一提交进行新的评估。对话在同一个 Paperclip 任务中继续，保留审查历史并更新一个当前摘要。旧的执行不能覆盖最新头部的评估。

请参阅 [GitHub 状态检查](https://docs.github.com/en/pull-requests/reference/status-checks)。

## 正式的批准和请求更改审查是独立的

GitHub 还支持正式的 PR 审查：**批准 (Approve)**、**请求更改 (Request changes)** 和仅评论审查。这些参与 GitHub 的审查策略，该策略独立于必需的状态检查。

这是另一个“审查”一词可能令人困惑的地方。我们一直在讨论智能体评估代码并产生检查。点击 GitHub 的 **批准** 按钮——或者让机器人通过 API 执行该操作——是一个独立的操作。

您可能希望智能体的评估是强制性的，同时保留人工批准。要求通过机器人的检查并单独要求人工审查，可以让您表达这两种期望。

因此，同一个 PR 上可能出现三样东西：

| 项目 | 含义 |
| --- | --- |
| 写着“5/5”的摘要评论 | 机器人对其评估的解释。 |
| 成功的 **Paperclip Review** 检查 | 经过验证的评估满足了该提交的配置评分策略。 |
| 正式的 **已批准 (Approved)** 审查 | 机器人在单独启用的权限下明确执行了 GitHub 的批准操作。 |

5/5 不会自动提交正式批准。正式批准不会使失败的检查变绿。两者都不能替代其他必需检查或人工审查要求。

Paperclip 的 **APPROVE** 和 **REQUEST_CHANGES** 权限是可单独配置的，且默认均关闭。启用某项操作允许智能体通过受治理的工具请求该操作；它不会自动执行该操作。

请参阅 [GitHub 的受保护分支和审查要求](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)。

## 自动必需审查者的实用设置

- 在目标仓库上安装应用，并在 Paperclip 中启用它们。
- 启用新 PR 和更新提交的审查，以及授权的重复提及。
- 将最低评分设置为 5/5。
- 在 GitHub 中，为目标分支要求 **Paperclip Review**，在可用时将机器人应用作为其预期来源。
- 除非您特别希望执行该独立操作，否则保持正式机器人批准关闭。
- 独立配置任何人工批准要求。

通过该设置，打开 PR 会启动智能体，评估变为检查，GitHub 等待该检查通过。推送会启动对新代码的新评估，提及允许您明确要求另一次审查。每个部分都有自己的职责，您有意识地选择每一项。

## 自己动手试试

[设置 GitHub 审查机器人](github-review-bot-setup.md) 通过一个完整的 Storybook 示例进行讲解：选择智能体，连接并安装其应用，链接您的账户，配置提示词，并观察失败的检查变绿。它还展示了使结果成为合并要求的可选 GitHub 规则。
