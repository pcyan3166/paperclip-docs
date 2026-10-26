---
paperclip_version: v2026.1023.0
seo_title: 连接器访问机制说明
seo_description: 适用的访问控制取决于连接器的类型。本文解释了工具连接的四个关卡，以及管理渠道和模型的规则。
---

# 连接器访问机制说明

连接服务与允许智能体使用该服务并非同一行为。Paperclip 将二者分离——但**如何**分离取决于您查看的连接器类型。请从这里开始。

## 首先：这是哪种类型的连接器？

四种形态，具有真正不同的控制机制。阅读错误的章节是导致产生错误预期的最常见原因。

| 形态 | 示例 | 访问控制依据 |
| --- | --- | --- |
| **应用工具** | Notion, Jira, Stripe, Supabase | 下方的四个关卡。这是目录中的大多数，本页其余部分均针对此类型 |
| **消息渠道** | Slack 聊天, Discord, Telegram, Microsoft Teams, iMessage Photon, GitHub 代码审查机器人, AgentMail | **无操作列表，无逐操作开关。** 范围取决于应用安装位置及关联的发送者。请参阅连接器自身的页面 |
| **模型提供商** | Anthropic, OpenAI, OpenRouter, Grok | **完全无工具。** 这些提供商提供模型运行所需的凭据。无需许可任何操作；关键在于凭据共享、责任用户及智能体资格。请参阅 [Anthropic](anthropic.md) 获取完整规则 |
| **混合用途** | [GitHub](github.md), [Slack](slack.md) | 两个独立的连接，拥有独立的凭据——一个是工具，一个是渠道。各自遵循上表中对应行的规则 |

> **警告：** 不要将某一行的假设带入另一行。在渠道或模型连接上，“将写入设置为先询问”毫无意义，且无法通过 **Permissions**（权限）选项卡缩小渠道的范围。

### 渠道拥有的控制机制

渠道没有操作开关，但并非不受管控。每个渠道都有一个 **Access**（访问）选项卡，其中包含 **External identity access**（外部身份访问），该选项卡上的一个开关决定了当 Paperclip 无法识别的某人发送消息时发生的情况：

| **允许未关联人员** | 效果 |
| --- | --- |
| **开启** | 发送者被视为受限访客。其任务仅在隔离的工作区和沙箱环境中运行，若不可用则 Paperclip 拒绝请求。访客无法批准、雇佣、支出、管理访问权限或重新分配智能体 |
| **关闭** | 仅与 Paperclip 人员关联的发送者才能启动工作 |

已关联的身份以其当前的 Paperclip 用户身份行事，拥有该人员的权限。

[iMessage Photon](imessage-photon.md) 是例外：它完全不允许未关联的发送者，因此在那里进行关联是必需的设置步骤，而非策略选择。

## 工具连接的四个关卡

本页其余部分描述 **应用工具** 连接。它们将访问权限分为四个决策，由不同的人在关键时刻做出，且每次工具调用都会对照这四个关卡进行检查。

```txt
      credential                   who it belongs to              which agents           which actions
  ┌──────────────────┐         ┌──────────────────────┐      ┌──────────────────┐   ┌──────────────────┐
  │ the connection   │────────▶│ the grant            │─────▶│ agent access     │──▶│ action           │
  │ token or key,    │         │ personal, org, or    │      │ any agent, or    │   │ Allowed /        │
  │ server URL,      │         │ dedicated to one     │      │ just the ones    │   │ Ask first /      │
  │ transport        │         │ agent                │      │ you pick         │   │ Off              │
  └──────────────────┘         └──────────────────────┘      └──────────────────┘   └──────────────────┘
```

调用必须通过所有四个关卡。仅持有凭据是不够的；仅作为被允许的智能体也是不够的。

## 连接

连接是指存储的凭据加上访问服务所需的一切：传输方式、服务器 URL 以及身份验证方式（`oauth`、`api_key` 或 `none`）。工具连接使用 `mcp_remote`、`rest_api` 或 `local_stdio`；`chat_sdk` 和 `runtime_auth` 传输方式属于上述的渠道和模型形态，这就是为什么操作关卡不适用于它们的原因。

每个连接都携带一个公司范围的标识——类似于 `uid` 的 `google-sheets/finance-sheet-1a2b3c4d`——该标识在重命名后依然有效。Paperclip 的其余部分引用的是该标识，这就是为什么在 UI 中重命名连接永远不会破坏指向它的策略。

密钥永远不会成为您可以读回的连接记录的一部分。凭据值作为密钥存储并被引用；API 返回的是元数据。

## 授权：凭据属于谁

授权回答了“这是谁的账户？”这一问题。Paperclip 支持三种类型。可用的选项和 UI 标签取决于连接器和方法；此表描述的是数据模型，而非每个设置屏幕都提供的三个选项：

| 设置选择 | 授权类型 | 凭据策略 | 含义 |
| --- | --- | --- | --- |
| **仅我** | `user` | `per_user` | 您的账户。智能体仅在您是责任人的运行中使用它。 |
| **组织内的任何人** | `organization` | shared | 公司的一个账户。符合条件的智能体会在运行责任人属于该凭据授权人员范围时使用它。 |
| **为智能体专用的账户** | `agent` | `per_agent` | 属于一个智能体的账户，无论谁启动了运行，始终由该智能体使用。 |

您在连接之前不必回答这个问题。设置会在主按钮上方的一行中说明它将使用的身份，**Change**（更改）会打开这些选择。当方法允许时，新连接默认使用组织身份；仅支持个人或专用智能体访问的方法则默认使用其中之一。当智能体从任务中请求连接时，设置默认从 **仅我** 开始，且仅限该智能体。

创建个人连接对任何活跃成员均可用。创建组织或专用智能体授权是管理操作——如果您缺乏权限，Paperclip 将以 *“Only a connection manager can share this credential with the organization”*（仅连接管理器可以将此凭据与组织共享）为由拒绝请求，并在服务器端强制执行，而非信任浏览器。

AgentMail 是设置例外：其新密钥默认对公司内所有人及所选收件箱所有者开放。重用密钥会保留其现有授权。

组织授权拥有自己的适用人员范围，在连接器的身份卡上设置：**公司内的任何人**，或 **我选择的人员** 并附带命名成员列表。该受众决定了共享凭据将支持哪些人的运行。

<a id="scopes-recorded-on-a-sign-in-grant"></a>
### 登录授权上记录的权限范围

当您通过提供商登录进行连接时，授权还会记录 OAuth 权限范围。并非每个提供商都会说明其实际授予的内容，因此记录会注明列表的来源。如果您通过 API 读取授权，请查看 `providerTenant.oauth` 下方：

- `scopeSource: "provider"` — 提供商在颁发令牌时声明了这些权限范围。
- `scopeSource: "requested_fallback"` — 提供商未说明任何内容，因此列表仅为 Paperclip 请求的内容。请将其视为未知，而非狭窄授权的证明。
- `requestedScopes` — Paperclip 在此授权上请求的内容。
- `unrequestedScopes` — 提供商声明但 Paperclip 从未请求的权限范围。除非提供商报告了更多内容，否则此项为空。

这些字段是记录，而非控制。权限范围列表不决定智能体可以调用哪些操作——操作权限才决定，如下所述。在此记录存在之前创建的授权保留其权限范围列表，但没有来源。

[为人员和智能体使用独立账户](separate-accounts.md) 介绍了如何在它们之间进行选择。

## 智能体访问：哪些智能体可以使用

独立于授权，每个连接都说明了哪些智能体可以使用它。在 **Which agents can use this connection?**（哪些智能体可以使用此连接？）下有两个选项——在连接时位于 **Change**（更改）之后，在连接后位于 **Permissions**（权限）选项卡上：

- **任何智能体** — *“在公司范围内可用。”*
- **仅我选择的智能体** — *“仅对选定的智能体可用。”* 然后您选择智能体。

新的工具连接通常默认为 `all_agents`；为请求智能体进行的设置会将访问权限缩小到该智能体。方法和已保存的连接可以携带更具体的访问权限。这不会绕过审批：创建连接本身是一个经过授权和审计的操作，且每个单独的操作仍有其自身的权限。

对于专用智能体身份，问题缩小为 *“哪个智能体拥有此 GitHub 账户？”*——身份和智能体是同一个决策。

## 已保存连接的指导

某些连接还会向符合条件的智能体的任务上下文贡献可编辑的指令。五个实验性记忆提供商包含默认值；已保存的指令也可与其他连接一起使用。

指导从不授予访问权限或移除审批要求。关闭它保留工具访问权限，且连接在传递其启用的指导之前，仍必须拥有可访问的工具和当前有效的授权。请参阅 [已保存连接指令](connection-instructions.md)。

## 操作权限：允许哪些调用

连接后，Paperclip 读取服务的操作列表，并为每个条目赋予三种状态之一：

| 状态 | UI 中的含义 |
| --- | --- |
| **Allowed**（允许） | *“无需审批即可运行。”* |
| **Ask first**（先询问） | *“人类必须批准每次调用。”* |
| **Off**（关闭） | *“智能体无法运行此操作。”* |

每个发现的操作还按风险进行分类，这是“将所有写入设置为先询问”所操作的对象：

- **read**（读取） — 无已知变更。
- **write**（写入） — 创建或更改某些内容。
- **destructive**（破坏性） — 删除、移除或以其他方式不可逆。提供商提供的显式 `destructiveHint` 注释会强制此分类。

对于具有广泛且快速变化目录的提供商，分类故意保守。PostHog 是最清晰的例子：未注释的 PostHog 工具被视为 **write**（写入）而非 read（读取），因为对未知工具静默假设“读取”是应避免的故障模式。其 `exec` 工具被硬性分类为 **destructive**（破坏性）。

有一类连接器存在一个限制，没有任何权限设置可以放宽：

**Google Workspace** 连接器——Gmail、Drive、Docs、Sheets、Slides、Calendar、Chat、People 和 Workspace Search——仅限于您连接的能力组中经过审查的读取和写入操作。Google 添加的不在审查列表上的工具会被关闭，而不是默认允许。

这就是为什么 [Gmail](gmail.md) 无法发送的原因。其草稿组请求 Google 的 `gmail.compose` 权限范围，该范围在 Google 端确实允许发送，但该组的审查写入列表仅包含一个操作，即 `create-draft`。凭据可以发送；但连接器不会提供此功能。同样的机制也是为什么 [Google Drive](google-drive.md) 可以复制和创建文件，但没有删除或共享操作。

[设置操作权限](action-permissions.md) 是操作指南。

## 当发生“先询问”调用时

智能体调用 **Ask first**（先询问）操作时，不会收到错误，也不会收到结果。该调用变为操作请求，智能体等待。

您可以从连接器的 **Review**（审查）选项卡或公司范围的审查队列中回答，在那里它显示为 **Waiting for your OK**（等待您的确认）。有三种回答：

- **Allowed once**（允许一次） — 此次调用运行，其他不变。
- **Always allowed**（始终允许） — 调用运行，且 Paperclip 创建一个 **trust rule**（信任规则），使匹配的调用停止询问。
- **Declined**（拒绝） — 调用不运行。

信任规则在之后可见且可撤销。[回答连接器审查请求](review-requests.md) 涵盖了双方的流程。

## 值得知道的一个例外

逐工具的 **Ask first**（先询问）管理通过 Paperclip 工具网关的工具调用。它不管理 shell（外壳）。

当 GitHub 连接作为专用身份绑定到智能体时，该身份也会交给运行的 shell。Paperclip 在您更改 GitHub 权限时明确说明：*“Shell Git 和 gh 在运行中使用此账户，不受逐工具先询问控制的约束。”* 能够在其工作区中运行 `git push` 或 `gh pr merge` 的智能体，受限于 GitHub 账户能做什么，而非连接器的操作开关。

这不是一般性的沙箱声明，也不是反方向的声明。关于此点的任何内容都不表示其他连接器是沙箱化的，且工具调用的人类审批与拉取请求上的代码审查是不同的事情。[GitHub](github.md) 有详细信息。

## 执行位置

访问决策在调用时解决，而非在设置时解决。工具网关持有智能体的有效策略——连接自身的设置，加上适用的任何工具配置文件和公司工具策略——并在请求到达提供商之前决定允许、询问或拒绝。更改权限在下次调用时生效；无需重新连接。

> **注意：** 自定义 `tools:use` 权限授权——存储的逐智能体或逐人员类型，而非上述的连接身份——严格按字面执行。如果授权的 `scope.allow` 列表命名了不同的工具、连接或应用程序，则不覆盖该调用；具有无法识别的选择器或格式错误的限制的授权不覆盖任何内容。升级后请审查自定义授权：如果智能体失去了之前使用的工具，请先检查其授权。[Tool Gateway reference](../reference/api/tool-gateway.md#explicit-tool-grants) 列出了匹配规则。

## 相关

- [连接器](../connectors.md)
- [与人员和智能体共享连接器](share-access.md)
- [设置操作权限](action-permissions.md)
- [工具网关](../reference/api/tool-gateway.md)
- [信任与低信任审查](../administration/trust-and-low-trust-review.md)

## 来源

- [Connection setup](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/ui/src/features/connections/ConnectionSetupFlow.tsx) — 单屏访问选择和设置行为。
- [Remote MCP setup](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/ui/src/features/connections/remote-mcp/RemoteMcpConnectionSetup.tsx) — 特定提供商的连接控制。
- [Connection grant type](https://github.com/paperclipai/paperclip/blob/abbd88007f059dbadbd3f266febf264577b6ee78/packages/shared/src/types/tool-access.ts) — 记录的 OAuth 权限范围及其来源。
