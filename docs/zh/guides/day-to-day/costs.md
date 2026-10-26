---
paperclip_version: v2026.1023.0
seo_title: 成本与预算：支出和限额
seo_description: 跟踪 API 支出、订阅费用和决策模型成本，并在 Audit 的 Costs 和 Budgets 标签中设置由 Paperclip 自动执行的预算限额。
---

# 成本与预算

> **警告：**AI 智能体会调用真实 API，并产生真实费用。智能体每次工作——每次心跳、每项任务、读取或撰写每条评论——都会通过 Anthropic 或 OpenAI 等提供商发送和接收文本；提供商按 token（大致相当于词）向你收费。这不是 Paperclip 收取的费用，而是你直接支付给 AI 提供商的成本。开始大量运行智能体前，请阅读本指南。

Paperclip 的预算系统旨在避免意外支出。你设置限额，平台会自动执行——智能体不会超出你允许的金额。

所有相关功能都位于侧边栏的 **Audit** 下。Audit 页面有五个标签：**Activity**、**Runs**、**Costs** 和 **Budgets**、**Timeline**；本指南介绍其中两个：

- **Costs**——查看资金花在哪里。该标签包含 **Overview**、**Providers**、**Billers**、**Finance**、**Decisions** 和 **Fast responses** 视图，共享日期范围选择器，并在顶部显示五项主要指标。
- **Budgets**——查看用于限制支出的预算，以及达到预算限额时生成的事件。

本指南会逐一介绍各视图，再说明成本的计算方式、预算如何保护你，以及如何逐步调整支出。（旧版 `/costs` 和 `/budgets` 链接仍然有效，会重定向到 Audit。）

---

## 成本如何计算

智能体每次执行心跳时都会产生 API 调用。调用会将智能体上下文（身份、任务、工作指令）发送给 AI 提供商，并接收响应（智能体的推理过程和下一步操作）。提供商根据 token 数量收费，包括输入 token（发送的内容）和输出 token（返回的内容）。

Paperclip 会记录每次调用：调用智能体、使用模型、token 数量和确切美元成本。记录按每个智能体的日历月汇总，每月 UTC 时间 1 日重置。

> **注意：**一个 token 大致相当于一个词，但准确地说，它是略小于词的文本片段。忙于编码或写作的智能体每月可能处理 100,000–500,000 个 token。按 Anthropic 常见定价计算，中等活跃度的工作者智能体每月大约花费 $3–$15；但实际费用会随模型和每项任务需要的上下文量而显著变化。

### 为什么美元金额比 token 数量看起来低

将智能体的 token 数量乘以提供商标价，通常会得到比 Paperclip 报告更高的金额。这种差距通常来自提示缓存，而不是计算错误。

智能体每次心跳都会重复发送许多相同文本，例如身份、常驻指令和当前任务。提供商会缓存这些重复上下文，缓存 token 的收费远低于新 token。你可以在 **Providers** 视图的每模型行中查看输入 token 和缓存输入 token 的分别统计。

Paperclip 记录的是**经缓存调整后的成本**：即提供商在应用缓存折扣后实际收取的费用，而非根据原始 token 数量推算的估值。适配器单独报告折扣后的金额时，该金额会记入成本账本、计入预算并汇总到 Costs 标签的各项总计中。适配器只报告单一成本数字时，Paperclip 会将其视为实际收费金额。

实用原则：界面中的美元金额就是实际计费金额。运行使用了大量缓存输入 token、但费用较低是正常现象——说明智能体在重复利用上下文，而不是每次都为相同内容重新付费。若要详细分析某次运行，请结合 token 列和美元金额一起查看，不要尝试从其中一个推算另一个。

### 估算、未定价和仍在记账的成本

并非每次运行都能立即获得明确价格。Paperclip 会标出不确定情况，而不是悄悄将其四舍五入或记为零：

- **Estimated（估算）**——提供商未报告价格，但 Paperclip 知道模型公开定价，因此按 token 数计算了价格。此类行会显示 **Estimated** 或 **Partially estimated** 徽标；悬停可查看估算了多少笔收费。提供商最终账单可能略有差异。
- **Unpriced（未定价）**——运行消耗了 token，但没有可靠价格。例如，智能体 CLI 指向 OpenAI 兼容端点，但该端点并非 OpenAI 或 OpenRouter；Paperclip 不会将 OpenAI 定价套用到其他服务。token 会保留，但成本会标记为未知，绝不会记为 `$0`。
- **Awaiting accounting（等待记账）**——运行已结束，但成本尚未记录。本地 CLI 智能体会在运行过程中保存用量，因此即使运行崩溃或超时，只要 Paperclip 后续完成同步，费用仍会被记录。

所选日期范围存在上述情况时，**Overview** 的指标卡片下方会出现一行简短、中性的说明，例如 *3 run costs unavailable*。该提示消失前，请将标签中的总计视为**已知**支出。这是信息提示，不一定代表错误；多数情况下无需你采取行动。

点击提示可按智能体查看受影响条目。每个智能体名称链接到其配置；每个条目显示发生时间、**View run** 链接和说明状态的标签：

| 标签 | 含义 | 可采取的操作 |
|---|---|---|
| **Cost unavailable** | 提供商未为此用量提供可靠价格。 | 无需操作——过去的条目不需要处理。 |
| **Model not reported** | 已记录 token，但没有模型信息，因此无法据此定价。 | 在智能体配置中为未来运行指定明确模型；过去的运行不会重新定价。 |
| **Billing category unknown** | 运行没有注明使用 API 还是订阅计费，但已记录的费用仍会计入。 | 在智能体配置中选择 AI 连接，以便识别未来运行。 |
| **Receiving usage** | 运行已停止，Paperclip 正在等待最终用量数据。 | 无需操作——系统会自动记录。无需重新运行智能体。 |
| **Final usage unavailable** | 运行停止时没有最终用量数据，因此预算核算仍无法确定其成本。 | 无需重试。如果迟到的用量收据之后到达，Paperclip 仍会记录；重新运行智能体无法恢复它。 |
| **Ready to record** | 已保存的用量数据可以记账。Paperclip 会自动重试。 | 点击 **Retry cost recording** 可立即重试。该操作只记录费用，不会重新运行智能体。 |

如果只有计费类别缺失，提示会改为类似 *2 billing types unknown*。每个类别最多显示最近 25 条；选择更短日期范围可以查看更多。只有查看权限的董事会成员可以查看列表，但不能重试记账。

非 API 或订阅计费的用量（例如点数或固定价格套餐）会在指标卡片下单独显示为 *Other billing usage*。这些用量仍会计入推理总额。

未定价和仍在记账的用量也会影响预算，参见[支出不确定时](#when-spend-is-uncertain)。

除每次请求的推理费用外，还有**账户级**费用——月度订阅费、充值、发票调整和退款等，这些费用无法对应到单次 API 调用。Paperclip 会将其单独记入财务账本，供你对照实际提供商发票和 Paperclip 记录的支出。

---

## Costs 标签：页眉和日期范围

从侧边栏打开 **Audit** 并点击 **Costs** 标签。Costs 中的所有视图共用相同页眉，因此深入各视图前，先了解页眉信息会有所帮助。

页眉标题为 **Costs**。

标题右侧是**日期范围选择器**，提供 **Month to Date**、**7 Days**、**30 Days**、**Year to Date**、**All Time** 和 **Custom** 预设按钮，用于筛选 Costs 标签上的所有图表、表格和指标。Month to Date 和 Year to Date 从 UTC 午夜开始，与月度预算重置所用时间一致。选择 **Custom** 后会出现两个日期输入框，可手动输入起止日期。在填完两个日期之前，多数视图会提示先选择范围再加载数据。

页眉下方是一排五个**指标卡片**：

- **API spend**——所选日期范围内按量付费 API 调用的金额；如果部分价格依据公开定价估算，会显示 **Estimated** 或 **Partially estimated** 徽标，下方列出 API token 数量。没有可靠价格的用量会显示 `—`。
- **Subscriptions**——已连接 AI 订阅的月费，以及智能体在所选范围内通过这些订阅使用的 token 数。参见下文[订阅](#subscriptions)。
- **Budget**——预算被突破时显示活动事件数量；否则，选择 **Month to Date** 时显示使用比例（例如 `62%`）和 `$X of $Y this month`。其他日期范围显示月度限额本身，因为月度上限的使用比例只适用于当前月份。未设置月度上限时显示 `Open`。
- **Recorded charges**——所选范围内账户级财务账本的净额，下方列出借记和贷记。
- **Finance events**——已记录财务事件数量，以及仍未与发票核对、处于估算状态的借记总额。

如果财务账本中有多种货币，页面会说明主要总计仅使用美元；其他货币会单独列出，Paperclip 不会进行币种换算。

这些指标反映所选日期范围，而非历史总额。唯一例外是 **Subscriptions** 中的月费——无论选择什么范围，它都显示**当前**月度承诺金额。

<a id="subscriptions"></a>
### 订阅

如果智能体使用 Claude、ChatGPT/Codex 或 Grok 订阅运行，通常不会产生每次调用的单独费用，但你仍需支付月费。**Subscriptions** 指标卡会将月费与 API 支出并列，便于比较。

Paperclip 根据智能体选择的 AI 连接，判断每次运行使用哪个订阅账户。多个智能体或连接共享同一个付费席位时只计**一次**，因此五个智能体共用一个 $200 套餐时显示 $200，而不是 $1,000。不同席位会分别统计，即使它们属于同一工作区。

指标卡显示：

- 按币种列出的月度总额（Paperclip 不会进行币种换算）。
- 价格来自公开标价而非你手动输入时显示的 **Estimated** 或 **Partially estimated** 徽标。
- 活跃订阅数量和未知价格数量。存在未知价格时，金额标签会显示为 *Known monthly subtotal*。
- 未确认的账户身份数量（如果有）。
- 智能体在所选日期范围内通过订阅使用的输入和输出 token。

点击 **View details** 查看每个账户的所有者（或 **Shared**）、提供商、套餐、月费、范围内使用的 token，以及使用该账户的智能体。标记为 *Estimated · before tax* 的价格来自提供商公开标价。通用套餐（如 OpenAI Pro、未指定层级的 Claude Max、团队或企业席位）无法确定可靠价格，因此在有人填写前会显示 **Price unknown**。

打开 Costs 标签时，Paperclip 会在后台检查已连接账户并读取套餐详情。检查失败时，会显示 *Connected accounts could not be checked. Existing subscription estimates are still shown.* 以及 **Retry account check** 按钮。

#### 输入实际价格

公开标价只是估算。如果你的实际付费金额不同（例如有折扣、使用年付套餐或其他币种），点击账户上的 **Edit price** 并填写：

- **Plan**。
- **Price per billing period**——一个账户或席位的完整账期费用；未知时可留空。
- **Currency**——`USD`、`EUR`、`GBP`、`CAD`、`AUD` 或 `CHF`。
- **Billed**——**Monthly** 或 **Annually**。年费会除以 12。
- **Tracking**——**Active**、**Subscription ended** 或 **Exclude from this organization**。

点击 **Save price**。更改从现在起生效，先前价格会保留为历史记录。在此处结束跟踪不会取消提供商处的订阅；断开连接也不会停止费用，因此实际取消订阅后应将其标记为已结束。

你可以编辑自己的个人账户价格。共享账户需要具备管理 AI 连接的权限。只有查看权限的董事会成员不能编辑价格。

#### 关联重复账户

有时 Paperclip 无法确认账户身份，例如某些 Claude 设置令牌无法读取个人资料信息。未确认账户不会自动按公开标价估算费用，因此凭据更改不会造成费用重复计算；但账户可能会重复显示。如果两个条目实际属于同一订阅或席位，打开其中一个，在 **Same subscription as another account?** 下选择另一个，然后点击 **Link accounts**。Paperclip 随后只计算一次费用、合并两者用量，并保留列表中所选账户的价格。

#### 月费不包括什么

订阅费覆盖整个提供商账户，包括 Paperclip 之外的使用。订阅费与已记录收费和智能体预算分开；它不会计入预算，也不会显示为财务事件。你只能看到有权查看的账户费用：自己的个人账户、由你管理的共享账户，以及将你列为受众成员的共享账户。token 总量仍覆盖公司全部工作。

如果订阅账户在所选范围内记录了额外用量费用（例如超额费用），指标卡下方会显示类似说明：*Subscription accounts also have $X in recorded usage charges for this period, separate from monthly fees.*

---

## Overview（概览）

![Audit Costs 标签的 Overview 视图](../../../user-guides/screenshots/light/costs/overview.png)

打开 Costs 标签时，默认显示 **Overview**。它让你在一个页面查看支出状况，无需先深入某一维度。上方指标卡显示主要支出和预算数据；Overview 会进一步拆解这些数字。

### 按智能体查看

左侧的 **By agent** 卡片会列出所选范围内产生推理事件的每个智能体，包括：

- 智能体姓名和头像；已退役的智能体带有 `terminated` 徽标。
- 右对齐显示所选范围内的总成本；部分成本按公开费率计算时会显示 **Estimated** 或 **Partially estimated** 徽标。
- token 明细：`in <input+cached> (<cached> cached) · out <output>`。
- 运行类型明细：`runs: N api · N sub`，显示按 API 价格计费和由订阅承担的运行数（区别见下方[计费方](#billers)）。

不属于任何智能体的收费（例如后台功能发出的[决策模型](#decisions)请求）会显示在 **Paperclip services** 行中。

某智能体存在每模型明细时，行左侧会显示折叠箭头。点击该行可展开查看每个 `provider / model / billingType` 组合的支出占比、成本、token 数和计费方。这是快速了解“这个智能体为何花费这么多”的方式——可以立即看出是某个模型用量失控，还是多种模型的正常组合。

### 按用户查看

智能体卡片下方是 **By user** 表格。每次运行都会记录服务对象，因此该表格按人员汇总支出；公司有多位成员并希望了解谁的请求带来费用时，这很有用。

每行显示用户、其关联 **Runs** 数量、**Input** 和 **Output** token 以及 **Cost**。请注意：

- 即使某位活跃成员在所选范围内没有支出，也会列出。
- 未记录到具体人员的支出（包括使用内置本地董事会登录运行的内容）会归入 **Unattributed** 行，因此表格始终与公司总额一致。
- 某人的部分用量未定价时，该行会显示未定价费用数（例如 `2 unpriced charges`）。全部用量均未定价时，成本显示 `—`，而不是 `$0.00`。

### 按项目查看

智能体明细右侧是 **By project** 卡片。运行由属于某项目的问题触发时，Paperclip 会将运行成本归到该项目。每行显示项目名称和归属成本总额。没有在项目关联问题中发生的运行会显示为 `Unattributed`。

### 活动预算事件（概览）

任何预算达到警告阈值或硬性停止线时，明细下方最多会显示两张 **Budget incident** 卡片。每张卡片显示作用范围、突破详情和两个处理按钮：**Keep paused** 或 **Raise budget & resume**（带有 **New budget (USD)** 输入框）。每个按钮的含义见[活动事件](#active-incidents)。因支出不确定而非超限导致硬性停止时，卡片会显示 *Accounting is incomplete*，并引导你查看待记账或未定价用量。完整事件列表位于 [Budgets 标签](#budgets-tab)。

### Finance ledger 卡片

Overview 底部是 **Finance ledger** 卡片，显示所选范围内账户级的账本：

- **Debits**——范围内的账户级费用以及事件总数。
- **Credits**——退款、抵扣和返还的额度。
- **Net**——该期间的借记减贷记。
- **Estimated**——尚未由发票确认的估算借记（例如尚未与提供商发票核对的订阅日费用估算）。

各财务事件会列在 [Finance](#finance) 视图中。

---

## Providers（提供商）

![Audit Costs 标签的 Providers 视图](../../../user-guides/screenshots/light/costs/providers.png)

**Providers** 视图按实际执行推理的提供商拆分支出，例如 Anthropic、OpenAI、Google 以及智能体使用的其他模型提供商。想知道“本月向 Anthropic 支付了多少？”或“OpenAI 用量是否增长？”时，应查看此视图。

### 提供商子标签

视图顶部有子标签栏。第一项是 **All providers**，汇总所有提供商的 token 和费用；之后每个在所选范围内产生事件的提供商各占一个子标签。子标签名称包含提供商显示名、token 数和成本，因此无需点击就能查看支出构成。

### 提供商卡片

选择 **All providers** 时，页面会显示提供商卡片网格（宽屏下每行两张）；选择某个提供商时，则单独显示其卡片。每张 **ProviderQuotaCard** 汇总该提供商的数据：

- **Per-model spend**——所选范围内每种模型一行，包含输入 token、缓存输入 token、输出 token 和美元成本。例如，可确认大部分 Anthropic 支出来自 Claude Sonnet，而不是一次误用 Opus。
- **Current-week spend**——单独汇总该提供商本日历周的成本，不受主日期范围选择器影响。适合发现周环比意外变化。
- **Share of budget**——该提供商占公司月度预算的比例。如果按当前消耗速率预测会超过其公平分摊额度，会显示亏空标记；仅在日期范围选择为 **Month to Date** 时显示。

<a id="quota-windows"></a>
### 配额时间窗

在 [AI connections](../../connectors/ai-usage.md) 中连接 Anthropic 或 OpenAI 订阅后，这两家提供商卡片还会显示 **Subscription quota** 区域。Paperclip 会分别读取每个连接的订阅账户，并使用该账户自身保存的登录身份——不会使用运行 Paperclip 的机器当前登录的账户。你只能看到有权使用的账户。

每个账户会以账户名称为标题。在提供商报告的每个用量窗口（例如 5 小时窗口或每周限额）中，显示：

- 时间窗口标签。
- 已使用比例。如果提供商未报告百分比，则会显示 **Usage not reported**，而不是进度条。
- 提供商报告的重置时间。

如果读取暂时失败（例如提供商响应慢），Paperclip 会继续显示最近一次成功读取的结果，并提示 *Showing the last available quota. Updates will resume automatically.*。没有可用旧数据时，会显示 *Subscription quota is currently unavailable. Check usage with your provider.*。账户未连接或登录已过期时，该区域会显示 *Connect or reconnect a subscription in AI connections to view its quota.*，并清除旧数据，避免将过期数值显示为当前状态。

### 提供商窗口支出

Paperclip 还会在配额进度条下显示**窗口支出（window spend）**——提供商在每个当前配额窗口内产生的费用。借此可以区分“5 小时额度已用 80%，因为刚刚出现一波用量”和“已用 80%，且用量一直均匀分布”。

---

<a id="billers"></a>
## Billers（计费方）

![Audit Costs 标签的 Billers 视图](../../../user-guides/screenshots/light/costs/billers.png)

**Billers** 回答的问题与 **Providers** 不同：不是“由谁执行推理？”，而是“由谁向我收费？”。大多数公司中两者相同，但也可能不同。例如通过订阅计划运行 Claude 时，Anthropic 既是计费方也是提供商；通过 API 运行时，Anthropic 仍是计费方，但定价模型不同；或者由网关或转售商代你处理账单关系。

### 计费方子标签

与 Providers 类似，Billers 视图顶部有子标签栏。**All billers** 显示汇总网格；每个计费方子标签单独显示同一张卡片。标签名称包含计费方显示名、token 数和成本。

### 计费方支出卡片

每张 **BillerSpendCard** 会显示：

- 计费方在所选范围内的总支出（美元和 token）。
- 当前日历周向该计费方支付的金额，便于短期核对。
- 该计费方占公司月度预算的比例。
- 此计费方内部按提供商划分的明细——哪些提供商模型通过此计费方收费，以及各自占比。例如可查看某个计费方是否同时支付 Anthropic 和 OpenAI 调用费用。

### 为什么这很重要

Billers 视图有助于核对发票。月底收到 Anthropic 发票时，发票金额应与 Anthropic 计费方卡片一直显示的金额大致一致。不一致时，可从此处开始调查：深入查看计费方卡片、比较提供商和模型明细，并将时间戳与提供商用量仪表板对照。

### 归属

Paperclip 中每笔成本事件都会标记提供商（执行推理的一方）和计费方（向你收费的一方）。简单配置中两者相同，通常无需关注 Billers 视图。使用多个相同提供商的账户、同时采用订阅和 API，或通过集中式网关时，Billers 视图可帮助你保持账务清晰。

---

<a id="finance"></a>
## Finance（财务）

![Audit Costs 标签的 Finance 视图](../../../user-guides/screenshots/light/costs/finance.png)

**Finance** 视图是账户级账本，记录所有非单次推理费用。订阅费、发票核对、充值、退款和调整都会显示在这里。Providers 和 Billers 视图显示智能体做了什么，Finance 视图则显示账户实际发生了什么。

### 财务摘要卡片

视图顶部的 **Finance ledger** 卡片包含四个指标：

- **Debits**——所选范围内借记金额总和及事件数。
- **Credits**——退款、抵扣和返还额度。
- **Net**——所选期间的借记减贷记。
- **Estimated**——Paperclip 已估算但尚未获得发票确认的借记。这些事件最可能随着发票到达而变化。

### 按计费方查看

**By biller** 卡片按向你收费或退款的对象分组显示财务事件。每个计费方行显示计费方名称、所选范围内的借记和贷记总额，以及事件数量。将账目与某个供应商的发票核对时，通常从这里开始。

如果还没有财务事件，卡片会提示你等计费方发票或额度到账后再添加。

### 财务时间线

计费方网格下方的 **Recent financial events** 卡片按时间顺序列出每项财务事件。每行显示事件类型（如订阅借记、额度充值、发票调整）、金额、计费方、时间戳，以及是否为估算或权威金额。想追查具体事件时，可在此找到对应行。

### 财务事件构成

时间线右侧的 **Financial event mix** 卡片不按收费方分组，而是按**事件类型**分组——例如将所有订阅借记、额度和调整分别合并。回答“订阅费与按量付费相比各花了多少？”时很有用。

### 估算值与实际值

Paperclip 将估算借记与权威借记分开，因为提供商发票才是最终依据。发票核对前，Paperclip 对某个订阅日费用的金额只是最佳估算，通常根据套餐价格除以账期长度得出。真实发票到达并导入后，估算标记会清除，金额变为权威值。

顶部的 **Estimated** 指标会显示财务账本中仍处于暂定状态的金额。数值较小时，表示账本大部分已由发票确认；数值较大时，说明还有大量估算支出等待核对。

### 其他货币

主要借记、贷记、净额和估算总额以美元显示。若记录了其他币种的费用，会按各自币种单独列出；Paperclip 不会应用汇率。

### 提供商成本报告和发票

操作人员可以通过[会计 API](../../reference/api/costs.md#accounting) 导入提供商自己的数据：

- **Provider cost reports**——直接从 OpenAI 或 Anthropic 组织获取的每日总额。这些数据与发票以及 Paperclip 自身的运行成本重叠，因此不会加入已记录费用。Finance 视图会单独显示一行：“Provider API cost reports: $X. Report totals overlap invoices and run estimates and are shown separately from recorded charges.” 可用它进行交叉核对。
- **Invoices**——导入发票后，其明细会加入财务账本，并可逐项与已记录费用比较，以便发现和更正差异。

---

<a id="decisions"></a>
## Decisions（决策模型）

部分可选 Paperclip 功能会让小型快速模型回答简短问题（例如“这是否与账单有关？”“应该由哪个团队负责？”），而不是唤醒完整智能体。这些是**决策模型（decision model）**请求；**Decisions** 视图会列出请求及费用。（它与[决策队列](./decisions.md)不同，后者收集需要由**人**作出的决定。）

### 设置决策模型

每家公司需要选择一个共享 API 连接来支付这些请求。新公司默认尚未配置。需要具备管理 AI 连接的权限才能设置。

1. 打开 **Company Settings → General** 并找到 **Decision model** 区域。如果实例管理员启用了 **Experimental fast responses**，该区域会改为显示在 **Company Settings → Connections** 顶部。
2. 选择已有共享 OpenAI 或 OpenRouter API 连接，或点击 **Add connection** 创建连接。Paperclip 在 OpenAI 上使用 GPT-6 Luna，在 OpenRouter 上使用 Jev 1.13。这里只支持公司共享 API key，不支持个人账户、订阅或自定义端点。
3. 保持 **Enable decisions** 开启，并决定是否开启 **Allow company-sponsored background decisions**。首次设置模型时该选项默认开启。开启后，后台功能可以在没有责任人的情况下使用该连接；这些请求会显示为 **Paperclip services**。如果关闭，即使之后更换连接也会保持关闭。
4. 点击 **Save decision model**。
5. 点击 **Run test** 发送一组少量、固定的示例问题。此操作会产生真实 API 费用。答案只会显示一次，不会保存；用量会被记录。

关闭 **Enable decisions** 会阻止新的请求，但不会取消或删除已经发送的请求。

### 查看 Decisions 视图

打开 **Audit → Costs → Decisions**，或在 Decision model 设置中点击 **View usage**。该视图与 Costs 标签的其他区域使用相同日期范围，并列出最近最多 100 个请求，包括：

- **When / feature**——请求开始时间及发起功能（测试按钮对应 **Setup test**）。
- **Responsible**——请求服务的人员；后台请求则显示 **Paperclip services**。
- **Task**——有可访问任务时提供任务链接。
- **Model**、**Status**、**Duration** 和 **Tokens in / out**。
- **Cost**——精确到美分的一小部分，标记为 **Reported** 或 **Estimated**；没有可靠价格时显示 **Unknown**。

Paperclip 不保存问题、输入或答案，只保存上述元数据。

决策请求与其他支出一样计入预算：公司预算，以及其所处理工作的智能体和项目预算。预算阻止请求时，测试会提示 *The company or task budget prevents this request.* 请求超时后，成本会保持未解决状态，预算占用也会保留，直到操作人员通过[会计 API](../../reference/api/costs.md#correct-a-charge) 处理。

<a id="fast-responses"></a>
### Fast responses（快速响应）

**Fast responses** 是实验性功能：有人向智能体发送消息时，小型模型会先发布简短确认，同时智能体开始工作。该功能默认关闭；实例管理员需在实例实验设置中启用 **Experimental fast responses**，之后你可以在 **Company Settings → Connections** 下进行配置。

Costs 标签中的 **Fast responses** 视图会以与 **Decisions** 相同的表格列出这些确认请求，并增加 **Connection** 列，显示每次请求由哪个连接付费。

---

<a id="budgets-tab"></a>
## Budgets 标签

![Audit 页面的 Budgets 标签，显示 Budget control plane 卡片和预算策略卡片](../../../user-guides/screenshots/light/costs/budgets.png)

从侧边栏打开 **Audit** 并点击 **Budgets** 标签。在这里配置保护公司的支出上限。它与 Costs 标签并列，而不是 Costs 的一部分，也不使用日期范围选择器——预算始终按自己的周期运行。可将它视为硬性停止的控制面板：你可以用金额决定任何范围在 Paperclip 拉响紧急刹车前最多支出多少。

### 预算层级

Paperclip 提供两个协同工作的预算保护层级：

**组织预算（Organization budget）**——整个公司每月的总支出上限，所有智能体合计不能超过该金额。刚开始使用时建议设置得保守一些。

**单个智能体预算（Per-agent budget）**——每个智能体自己的月度上限。该限制独立于公司总额执行。即使公司仍有预算，达到 100% 的智能体也会暂停，以免某个智能体支出过高。

第三种范围是**项目预算（project budget）**，用于限制与执行绑定项目的生命周期总支出。适合“一次性工作，整个项目绝不能超过 $200”之类的情况。可在项目自己的 **Budget** 标签中设置。

Paperclip UI 中的所有预算都以美元显示。

### 预算控制面板

Budgets 标签顶部的 **Budget control plane** 卡片包含四项主要指标：

- **Active incidents**——仍未处理的软阈值或硬阈值突破数。
- **Pending approvals**——等待董事会处理的预算覆盖审批数。
- **Paused agents**——因预算突破而无法运行的智能体数。
- **Paused projects**——因预算突破而无法执行的项目数。

四项均为零时，预算状态正常。

### 达到限额后会发生什么

Paperclip 分两个阶段自动执行预算：

```
$0 ─────────────────────── 80% ──────── 100%
   Normal operation         ⚠              🛑
                          Warning        Auto-paused
```

**达到 80%：**Paperclip 会记录警告，让你有机会在硬性停止前介入。警告本身不会暂停智能体。不过，在 v2026.1023.0 中，事件卡片仍使用硬性停止的视觉样式：提供 **Keep paused** 和 **Raise budget & resume**，并显示范围已暂停。截图展示了[后续版本修复](https://github.com/paperclipai/paperclip/pull/16201)中的琥珀色警告卡片。修复发布前，请以策略卡片上的 **Warning** 徽标为准：警告本身不会暂停智能体。

**达到 100%：**智能体会自动暂停，不会再触发心跳。提高预算或等到日历月重置前，智能体会停止所有活动。

自动暂停不会丢失智能体的工作——其正在进行的任务仍会分配给它，并在智能体恢复后继续处理。

<a id="when-spend-is-uncertain"></a>
### 支出不确定时

如果无法信任某个范围当前的支出金额，硬性停止本身也无法可靠执行。因此，即使尚未达到 100%，Paperclip 也会暂停该范围的新工作：

- **已完成运行仍在等待记账。**新工作会等到该笔费用计入后再开始。通常片刻内即可自动处理。
- **预算窗口中有未定价用量。**默认情况下，该范围会保持阻塞，因为 Paperclip 无法判断实际支出距离上限还有多远。

策略卡片会说明原因，例如 *“2 completed runs await accounting.”* 或 *“3 usage events are unpriced. Known spend excludes their unknown cost.”*

如果你愿意将未定价用量当作未知数处理，可打开策略卡片的 **Advanced settings**，关闭 **Block new work when usage has no reliable price**。否则必须先为用量确定价格，才能恢复该范围；操作人员可通过[会计 API](../../reference/api/costs.md#correct-a-charge) 添加价格。

<a id="active-incidents"></a>
### 活动事件

任何预算策略突破后，控制面板下方都会出现 **Active incidents** 区域。每张事件卡片显示：

- 作用范围（公司、智能体或项目）及其名称。
- 策略详情——设置的上限和实际支出。
- 两项操作：
  - **Keep paused**——确认突破并保持范围暂停。适用于希望智能体在本月剩余时间停止工作、等预算周期重置后自然恢复的情况。
  - **Raise budget & resume**——提高策略限额（在 **New budget (USD)** 中输入新上限；必须高于已支出金额），并立即恢复该范围。

如果该范围仍有运行等待记账，或存在设置为阻塞新工作的未定价用量，提高预算会失败。此时会看到 *Could not update the budget. Check any pending runs or unpriced usage shown on this page, then try again.* 请先处理这些事项（参见[支出不确定时](#when-spend-is-uncertain)）。

两种操作都会写入审计记录。

### 按范围查看预算策略

事件区域下方最多有三个分组：**Organization Budgets**、**Agent Budgets** 和 **Project Budgets**；每项已配置策略各有一张卡片。只有存在至少一个策略的分组才会显示。

每张策略卡片显示：

- 适用的范围类型和名称，以及周期——组织和智能体策略使用 **Monthly UTC budget**，项目策略使用 **Lifetime budget**。
- 状态徽标：**Healthy**、**Warning**、**Hard stop** 或 **Paused**。
- **Observed** 已观测支出（含已用限额比例）、**Budget** 上限和软警告阈值。
- **Remaining** 剩余额度和使用条——正常时为绿色，超过警告阈值后变为琥珀色，达到硬限额后变为红色。
- 等待记账的运行或周期内未定价用量。
- **Budget (USD)** 输入框及 **Set budget** 或 **Update budget** 按钮。将预算设为 `0` 会关闭该策略。
- **Advanced settings** 区域，包含两个控件：
  - **Block new work when usage has no reliable price**——默认开启，见[支出不确定时](#when-spend-is-uncertain)。
  - **Reserve per run (USD)**——每次运行开始前从预算中预留的估算金额，通过 **Update reservation** 保存。它可防止多次运行同时启动并合计超限。默认值 `0` 表示关闭。每次未完成运行都会占用预留额；预留金额会使范围超过上限时，新运行会等待先前运行完成。预留只控制工作何时可以开始，实际费用仍可能高于预留金额。

尚无策略时，空状态卡片会显示：“No budget policies yet. Set agent and project budgets from their detail pages, or use the existing organization monthly budget control.” 实际上，各类策略的起点如下：

- **智能体**——通过 API 招聘智能体时指定月度预算，或通过 CLI/API 为已有智能体设置上限（参见[设置月度预算](../../how-to/set-monthly-budget.md)）。
- **项目**——打开项目并点击 **Budget** 标签。即使尚无策略，也可以在那里设置生命周期上限。
- **组织**——使用组织级月度预算控件。

策略创建后，可在此卡片中修改。

### 设置公司预算

1. **打开 Audit → Budgets**。

   点击侧边栏中的 **Audit**，再点击 **Budgets** 标签。公司级策略位于 **Organization Budgets** 下。

   ![Audit → Budgets 中的组织预算策略卡片，显示已观测支出、Budget (USD) 输入框和 Update budget 按钮](../../../user-guides/screenshots/light/costs/company-budget-field.png)

2. **设置月度预算。**

   在 **Budget (USD)** 输入框中填写金额。这是整个公司在一个日历月中可支出的最高金额。

   如果还没有 Organization Budgets 卡片，请先通过 API 设置公司首个预算。[设置月度预算](../../how-to/set-monthly-budget.md)提供了一行调用示例。策略创建后，卡片就会出现。

   > **提示：**建议从较保守的金额开始——小团队每月 $50–100 足以进行试验。了解智能体实际支出后，可以随时提高上限。

3. 点击 **Update budget**（上限当前为零时则点击 **Set budget**）。

### 设置单个智能体预算

每个智能体都应有自己的预算。这既限制了单个智能体带来的风险，也让成本状况更容易理解。

1. **找到智能体预算卡片。**

   点击侧边栏中的 **Audit**，再点击 **Budgets** 标签，并在 **Agent Budgets** 下找到目标智能体。（也可从智能体自己的页面进入：在其侧边菜单 Audit 下点击 **Budgets**。）

   ![Audit → Budgets 中的智能体预算策略卡片，显示已观测支出、月度上限和 Budget (USD) 输入框](../../../user-guides/screenshots/light/costs/agent-budget-field.png)

   没有卡片表示该智能体尚无预算策略。通过 API 招聘并指定月预算的智能体会自动获得策略，但应用内的创建流程不会设置预算。若要为其他智能体设置上限，可使用 CLI 的 `budget agent:update` 命令（参见[成本和预算命令](../../reference/cli/cost.md)）或[设置智能体预算](../../reference/api/costs.md#set-agent-budget) API。[设置月度预算](../../how-to/set-monthly-budget.md)介绍了 API 调用。

   项目的处理方式不同：打开项目并点击 **Budget** 标签，即使还没有策略也可以设置生命周期上限。

2. **设置月度预算。**

   在 **Budget (USD)** 中输入金额。以下是考虑智能体预算时的一些粗略参考：

   | 智能体角色 | 常见月度预算 |
   |---|---|
   | CEO | $30–50——运行频繁，每次运行上下文较长 |
   | 经理（CTO、CMO 等） | $20–40——较活跃，但比 CEO 更聚焦 |
   | 工作者智能体（工程师、写作者等） | $10–25——执行任务，但战略推理较少 |

   这些金额只是起点，并非硬性规则。前两周应观察实际支出，再据此调整。

3. 点击 **Update budget**（上限当前为零时则点击 **Set budget**）。

### 提高预算或恢复已暂停智能体

智能体达到 100% 上限并自动暂停后，有两种选择：

**选项 1：提高智能体月度预算**

1. 打开 **Audit → Budgets**。突破会显示在 **Active incidents** 卡片中；智能体在 **Agent Budgets** 下的策略卡片会标为 **Paused**。
2. 在事件卡片的 **New budget (USD)** 中输入更高金额并点击 **Raise budget & resume**；或者在策略卡片的 **Budget (USD)** 输入框中输入新上限并点击 **Update budget**。

只要支出已完整记账，智能体就会立即恢复，无需等待月底。如果仍有运行等待记账或存在未定价用量，相关问题解决前仍会保持暂停（参见[支出不确定时](#when-spend-is-uncertain)）。

**选项 2：等待月份重置**

每个 UTC 月的 1 日，所有月度预算都会重置为零。纯粹因预算原因暂停的智能体会在周期切换时自动恢复为 active。

> **注意：**如果你不希望智能体在月初自动恢复，请先手动暂停。否则，新月份开始后它会立即重新运行。

### 查看不同阈值下的预算状态

以下示例展示了智能体在 **Audit → Budgets** 中达到不同阶段时的策略卡片：

![约 50% 使用率的智能体预算策略卡片——Healthy 徽标和绿色使用条](../../../user-guides/screenshots/light/costs/agent-budget-50pct.png)

*正常运行——状态为 **Healthy**、使用条为绿色，无需处理。*

![约 80% 使用率的智能体预算策略卡片——琥珀色 Warning 徽标和琥珀色使用条](../../../user-guides/screenshots/light/costs/agent-budget-80pct-warning.png)

*达到警告阈值——徽标变为 **Warning**。这是在硬限额前介入的信号。*

![100% 使用率的智能体预算策略卡片——Paused 徽标、红色满格使用条，并注明提高预算或解除事件前心跳将暂停](../../../user-guides/screenshots/light/costs/agent-budget-100pct-paused.png)

*预算耗尽——卡片显示 **Paused**，智能体自动暂停。提高预算或等到月度重置即可。*

---

## 监控支出

### 从仪表板查看

仪表板上的 **Month Spend** 卡片会显示公司本月已支出金额及其占公司预算的比例（例如 `62% of $100.00 budget`）。如果有任何预算突破，卡片上方会显示红色横幅，列出活动事件数和暂停的智能体数。

每次打开 Paperclip 时都可以查看该卡片。点击它可跳转到 Audit 下的 Costs 标签，其中 **Overview** 会按智能体和项目拆分支出：

![Audit Costs 标签的 Overview，显示日期范围预设、主要指标卡片和 By agent 明细](../../../user-guides/screenshots/light/costs/costs-dashboard-overview.png)

与上次检查相比总额意外增加的智能体值得进一步调查。

### 深入查看某个智能体

在 Costs 标签 Overview 的 **By agent** 卡片中，点击智能体行即可展开。页面会按提供商和模型逐行显示该智能体使用的服务、支出占比、成本、token 数和计费方。

![Audit Costs 标签的 By agent 卡片，展开一个智能体以显示提供商、模型、支出占比、成本和 token 明细](../../../user-guides/screenshots/light/costs/per-run-cost-detail.png)

智能体自己的页面 Overview 中的 **Audit** 区域可直接链接到该智能体的 **Runs**（检查单次心跳）和 **Budgets**（查看上限和使用率）。当你想了解某个智能体为何比预期花费更多时，这些入口很有用。

### 从 Audit 查看

跨智能体的问题，例如“本周哪个提供商收到的钱最多？”“哪个项目消耗的预算最多？”“账本中还有多少金额是估算的？”，应打开 **Audit** 并选择对应标签和视图。在 **Costs** 标签中：总体健康情况查看 Overview；供应商账务查看 Providers/Billers；发票核对查看 Finance；小模型请求查看 Decisions 和 Fast responses。**Budgets** 标签用于管理硬性停止。

---

## 节省成本的建议

**为工作者智能体使用成本较低的模型。**CEO 智能体需要推理战略，可使用最强的可用模型（例如 Claude Opus）。执行常规任务（撰写内容、处理数据、搜索）的工作者智能体通常可以使用更快、更便宜的模型（例如 Claude Sonnet），而质量不会明显下降。可在智能体的适配器配置中设置模型。

**降低后台智能体的心跳频率。**每 15 分钟运行一次的智能体，API 调用量约为每小时运行一次的 4 倍。无需快速响应的智能体使用更长间隔，可以在影响很小的情况下节省费用。

**缩短任务说明。**每次心跳时，智能体都会读取完整任务描述、上下文和对话历史。开放式任务和很长的上下文链会迅速累积成本。使用具体、简洁且有明确完成标准的任务描述，可让上下文保持小而聚焦。

**非工作时间暂停智能体。**如果公司不需要全天候运行，可在每天工作结束时手动暂停智能体，早晨恢复。大多数自动化工作可以在设定的时间窗口内完成。

**关注新智能体的单次运行成本。**新智能体完成前几次心跳后，在 Costs 标签的 **By agent** 卡片中检查支出。如果高于预期，应检查其工作方式：过长的上下文链、过多工具调用或配置不当的提示都可能推高成本。

**用量可预测时优先考虑订阅。**如果智能体工作量稳定，并且能够控制在提供商套餐窗口范围内（例如 Anthropic Pro 订阅），通过订阅计费方路由可能比按量付费 API 更便宜。分配智能体前，使用 Providers 视图的配额窗口确认订阅仍有余量；再比较 **Subscriptions** 和 **API spend** 指标，确认套餐是否划算。

**定期核对财务账目。**由发票确认的金额总是比估算值准确。每月花几分钟通过[会计 API](../../reference/api/costs.md#import-provider-invoices) 导入真实发票，可以保持账本准确，并及时发现价格变化。

---

公司的财务状况现已纳入管理。下一篇指南介绍 Audit 下的 **Activity** 标签——公司中所有操作的完整审计记录，以及如何利用它理解和调试智能体行为。

[审计：活动 →](./activity-log.md)
