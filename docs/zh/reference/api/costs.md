---
paperclip_version: v2026.1023.0
seo_title: 成本 API
seo_description: 公司范围的支出端点，回答四个问题：支出了多少、由谁产生、相对预算如何，以及离公司上限还有多远。
---

# 成本

需要回答以下四个问题时，可以使用此 API：

- 公司支出了多少？
- 哪些代理、模型、提供商或项目产生了这些支出？
- 预算是否即将发出警告或硬性阻止工作？
- 除令牌支出外，还记录了哪些非令牌财务事件？

如果需要验证账本本身是否准确——例如仍有运行等待记账、使用量没有可靠价格，或提供商账单对不上——请使用本页末尾的[会计](#会计)端点。

以下所有端点都限定在公司范围内。

> **金额。** 每个以数字形式返回分值金额的响应（例如 `costCents` 或 `spendCents`）也可能包含以 `Exact` 结尾的精确字符串字段（例如 `costCentsExact: "12.3456789"`）。字符串保留到百分之一美分的小数点后 7 位，因此不会抹去每次请求产生的小额费用。对账时请使用 `Exact` 字段；显示金额时请使用数字字段。

---

## 上报用量

### 成本事件

`POST /api/companies/{companyId}/cost-events`

这是记录令牌支出的主要端点。适配器通常会在心跳或其他 AI 操作结束后调用此端点。

必填字段：

- `agentId`
- `provider`
- `model`
- `costCents`
- `occurredAt`

常见的可选字段：

- 如果支出来自特定任务，传入 `issueId`。
- 如果支出明确归属某项目，传入 `projectId`。
- 如果可以将支出归因到更高层级的目标，传入 `goalId`。
- 如果支出来自特定运行，传入 `heartbeatRunId`。
- `billingCode`：自定义会计标签。
- 如果收费方不同于 `provider`，传入 `biller`。
- 如果要区分 `metered_api`、`subscription_included`、`subscription_overage`、`credits`、`fixed` 或 `unknown`，传入 `billingType`。
- 使用 `costStatus` 标记用量如何定价：提供商上报金额时用 `reported`；根据令牌数和公开费率表计算时用 `estimated`；已记录令牌用量但没有可用金额时用 `unpriced`。
- 令牌层级数据使用 `inputTokens`、`cachedInputTokens` 和 `outputTokens`（范围为 0 至 `2147483647` 的整数）。
- `idempotencyKey`：最多 200 个字符，避免重试上报时重复计费。
- `providerRequestId`：最多 250 个字符，提供商自己的请求 ID，方便之后与导入的发票条目匹配。
- `pricingProvenance`：价格来源信息，包括 `source`（`provider_reported`、`provider_invoice`、`operator`、`rate_card` 或 `unknown`），以及可选的 `version`、`evidence`、`serviceTier`、`contextTier`（`short` 或 `long`）和每百万令牌费率 `inputCentsPerMillion`、`cachedInputCentsPerMillion`、`cacheWriteCentsPerMillion`、`outputCentsPerMillion`。

实现中的规则：

- 代理必须属于该公司。
- 看板用户可以上报公司内任何代理的成本。
- 通过代理身份认证的调用只能上报该代理自身的成本。
- 省略 `biller` 时，默认使用 `provider`。
- 省略 `billingType` 时，默认为 `unknown`。
- 省略 `costStatus` 时，默认为 `reported`。
- `costCents` 可以是数字或十进制字符串（例如 `"0.0425"`），不能为负数。小于一美分的金额也会保留。
- `occurredAt` 必须是 ISO 日期时间字符串。

> **已定价、估算和未定价的用量。** 大多数成本事件都有真实的 `costCents` 金额，因此状态为 `reported`。有些运行没有提供商上报价格，但模型有 Paperclip 发布的费率；此类记录标记为 `estimated`，实际提供商账单可能不同。当代理运行使用了令牌，但 Paperclip 完全无法确定其美元成本时（例如本地 CLI 指向 OpenAI 或 OpenRouter 以外的 OpenAI 兼容端点），用量会标记为 `unpriced`，而不是记录成零成本。这样既保留令牌数，也明确金额未知而非确实为零。未定价用量可能会阻止受预算控制的工作；参阅[达到阈值时的行为](#达到阈值时的行为)。

接收事件后，Paperclip 会：

- 存储该事件。
- 重新计算代理和公司的 `spentMonthlyCents`。
- 评估预算策略，处理软警告和硬性限制。
- 写入活动日志。

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -X POST "http://localhost:3100/api/companies/company-1/cost-events" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "agentId": "agent-1",
    "issueId": "issue-1",
    "projectId": "project-1",
    "heartbeatRunId": "run-1",
    "provider": "anthropic",
    "biller": "anthropic",
    "billingType": "metered_api",
    "model": "claude-sonnet-4-20250514",
    "inputTokens": 15000,
    "cachedInputTokens": 2000,
    "outputTokens": 3000,
    "costCents": 12,
    "occurredAt": "2026-04-15T12:30:00.000Z"
  }'
```
<!-- tab: JavaScript -->
```js
await fetch("http://localhost:3100/api/companies/company-1/cost-events", {
  method: "POST",
  headers: {
    Authorization: "Bearer <token>",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    agentId: "agent-1",
    projectId: "project-1",
    heartbeatRunId: "run-1",
    provider: "anthropic",
    billingType: "metered_api",
    model: "claude-sonnet-4-20250514",
    inputTokens: 15000,
    cachedInputTokens: 2000,
    outputTokens: 3000,
    costCents: 12,
    occurredAt: "2026-04-15T12:30:00.000Z",
  }),
});
```
<!-- tab: Python -->
```python
import requests

requests.post(
    "http://localhost:3100/api/companies/company-1/cost-events",
    headers={"Authorization": "Bearer <token>"},
    json={
        "agentId": "agent-1",
        "projectId": "project-1",
        "heartbeatRunId": "run-1",
        "provider": "anthropic",
        "billingType": "metered_api",
        "model": "claude-sonnet-4-20250514",
        "inputTokens": 15000,
        "cachedInputTokens": 2000,
        "outputTokens": 3000,
        "costCents": 12,
        "occurredAt": "2026-04-15T12:30:00.000Z",
    },
)
```
<!-- /tabs -->

> **提示：** 如果已知产生支出的 issue、项目或运行，请传入对应 ID。之后查看细分报表时，这些信息会很有用。

### 按缓存调整成本

提示缓存意味着提供商实际收取的运行费用通常远低于原始令牌数所暗示的金额。Paperclip 记录的是实际计费金额，而非名义金额，因此账本中的金额会与发票相符。

适配器通过执行结果提供以下字段：

- `costUsd`：适配器报告的运行成本，单位为美元。
- `cacheAdjustedCostUsd`：应用提示缓存折扣后、提供商实际收取的金额，单位为美元。适配器单独提供缓存调整金额时会设置此字段。

Paperclip 会将二者解析为一个最终计费金额：

- `cacheAdjustedCostUsd` 是有限且大于等于零的数字时，使用该值作为计费金额。
- 否则，如果 `costUsd` 是有限且大于等于零的数字，则使用该值作为计费金额；提供商上报的 `costUsd` 视为已考虑缓存调整。
- 如果都不是，则该运行没有可用的计费金额。

Paperclip 会将解析出的金额转换为该运行成本事件中的 `costCents`。下文的所有摘要、分项和预算策略都使用此数值。此值也用于决定 `costStatus`：如果运行有令牌用量但无法解析成本，就会标记为 `unpriced`，而非真实零成本。

运行记录会保留这两个金额。心跳运行的 `usageJson` 会在适配器提供 `costUsd` 时记录该字段；如果解析出了缓存调整金额，也会记录 `cacheAdjustedCostUsd`，以及令牌计数、`provider`、`biller`、`model`、`costStatus` 和 `billingType`。可通过以下任一端点读取：

- `GET /api/companies/{companyId}/heartbeat-runs`
- `GET /api/heartbeat-runs/{runId}`

> **注意：** `cacheAdjustedCostUsd` 和 `costUsd` 的单位是美元；成本事件中的 `costCents` 单位是美分。将运行与账本条目对账时，切勿混淆单位。

---

## 读取支出

### 谁有权读取支出

本节所有读取端点都要求调用者能访问公司，并具有读取公司范围数据的权限（`company_scope:read`）。缺少该权限时会返回 `403`，错误信息为 `Costs are outside this actor's authorization boundary`。

<a id="choosing-a-date-range"></a>
### 选择日期范围

摘要、细分报表和财务读取端点共用相同的日期筛选项：

- `from`：可选起始时间，可为带偏移量的 ISO 日期时间（`2026-04-01T00:00:00.000Z`）或普通日期（`2026-04-01`）。
- `to`：可选结束时间，格式相同。
- `period`：可选；`all` 表示全部历史数据，或 `month`。

返回的数据范围取决于提供的参数：

- **不传参数**——当前 UTC 日历月，从每月 1 日到现在。与月预算使用的时间范围一致。
- **传入 `from` 和/或 `to`**——完全按指定范围查询，起止时间均包含在内。
- **`period=all`**——查询所有历史数据。不能与 `from` 或 `to` 同时使用。

`period` 未知、日期无法解析或 `from` 晚于 `to` 时，服务器会返回 `400`。

> **注意：** 过去省略日期会返回全部历史数据；现在改为返回当前月。如果脚本依赖旧行为，请添加 `period=all`。

### 公司摘要

`GET /api/companies/{companyId}/costs/summary`

返回所选日期范围内的公司总支出、当前公司预算和预算使用率，并告知金额数据是否完整。

响应字段：

- `spendCents`（及 `spendCentsExact`）——范围内已知支出。
- `budgetCents`。
- `utilizationPercent`。
- `eventCount`——范围内成本事件数。
- `estimatedEventCount`——其中按费率表估算、而非由提供商上报价格的事件数。
- `unpricedEventCount`——没有可靠价格的用量（包含在订阅中的用量不计入）。
- `pendingRunCount`——已完成但尚未记录成本的运行数。
- `unknownBillingEventCount`——尚未确定按 API 还是订阅计费的用量事件数。这些费用仍计入 `spendCents`。
- `pricingComplete`——只有在不存在未定价用量和待处理运行时才为 `true`，表示 `spendCents` 已涵盖全部金额。

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl "http://localhost:3100/api/companies/company-1/costs/summary?from=2026-04-01T00:00:00.000Z&to=2026-04-30T23:59:59.999Z" \
  -H "Authorization: Bearer <token>"
```
<!-- tab: JavaScript -->
```js
const res = await fetch(
  "http://localhost:3100/api/companies/company-1/costs/summary?from=2026-04-01T00:00:00.000Z&to=2026-04-30T23:59:59.999Z",
  { headers: { Authorization: "Bearer <token>" } },
);
const summary = await res.json();
```
<!-- tab: Python -->
```python
import requests

summary = requests.get(
    "http://localhost:3100/api/companies/company-1/costs/summary",
    headers={"Authorization": "Bearer <token>"},
    params={
        "from": "2026-04-01T00:00:00.000Z",
        "to": "2026-04-30T23:59:59.999Z",
    },
).json()
```
<!-- /tabs -->

<a id="breakdown-views"></a>
### 支出细分视图

这些端点使用与摘要端点相同的日期筛选项：

- `GET /api/companies/{companyId}/costs/by-user`
- `GET /api/companies/{companyId}/costs/by-agent`
- `GET /api/companies/{companyId}/costs/by-agent-model`
- `GET /api/companies/{companyId}/costs/by-provider`
- `GET /api/companies/{companyId}/costs/by-biller`
- `GET /api/companies/{companyId}/costs/by-project`

用途如下：

- `by-user` 用于了解支出是代表哪些人员产生的。
- `by-agent` 用于查看总体成本较高的代理。
- `by-agent-model` 用于发现哪个代理/模型组合消耗了大量令牌。
- `by-provider` 用于比较 Anthropic、OpenAI 及其他已接入提供商。
- 当提供商名称与实际收费实体不同时，使用 `by-biller`。
- `by-project` 可将成本关联到项目工作，而不仅仅是某个代理。

`by-agent` 中，不属于某个代理的费用（例如后台[决策模型](./decision-models.md)请求）会作为 `agentId: null` 的行返回。UI 会将它标记为 **Paperclip 服务**。

`by-agent` 和 `by-agent-model` 的行包含 `eventCount`（该组账本事件数，不是不同运行的数量）和 `estimatedEventCount`，可用来判断总额中有多少属于估算值。

### 按用户查看成本

`GET /api/companies/{companyId}/costs/by-user`

每次运行都会记录代表其执行工作的人员，即责任用户。此端点按该用户汇总成本事件。响应包含两个字段：

- `activeUserCount`——公司的有效人类成员数；不包括内置本地看板登录用户。
- `rows`——每位用户一行，按支出排序。每个有效成员都会显示，即使支出为零。

每行包含：

- `userId`、`userName`、`userImage`。**未归因**行的 `userId` 为 `null`，包含未记录用户的支出（包括记在本地看板登录名下的支出）。已经不再有效的成员仍会在自己的行中保留历史支出；如果无法找到其姓名，则标记为 `Former user`。
- `runCount`——范围内包含成本事件的不同运行数量。
- `eventCount`、`estimatedEventCount`、`unpricedEventCount`。
- `costCents`、`costCentsExact`。
- `inputTokens`（不包括缓存输入）、`cachedInputTokens`、`outputTokens`。

未归因支出有单独一行，因此所有行的总和始终等于公司总额。

### 成本缺口

`GET /api/companies/{companyId}/costs/data-quality`

摘要显示存在未定价、待处理或计费方式未知的用量时，此端点会说明相关运行和事件。成本页面的可展开成本缺口行由此端点提供数据。日期筛选项与摘要端点相同。

响应包含：

- `events` 和 `runs`——每类最多返回最近的 25 条记录。每项包含 `id`、`runId`、`agentId`、`agentName`、`occurredAt`、`provider`、`model` 和 `reason`。
- `hasMoreEvents` 和 `hasMoreRuns`——列表被截断时为 `true`；请缩小日期范围以查看其余记录。
- `canRetry`——你是否有权限调用下方的重试端点。

`reason` 可能为：

- `missing_price`——用量没有可靠价格。
- `missing_model`——已记录令牌但未提供模型。
- `unknown_billing`——运行没有确认按 API 还是订阅计费。
- `waiting_for_receipt`——运行已停止，最终用量尚未送达。
- `receipt_unavailable`——运行停止已有一段时间，但最终用量数据仍未到达；仍会接受延迟收到的使用凭据。
- `retryable`——已保存的用量可以尝试记账。

详情不会包含提供商错误或凭据。

`POST /api/companies/{companyId}/costs/retry-accounting`

```json
{ "runId": "<run uuid>" }
```

立即尝试记录某次运行已保存的用量，而不必等待自动重试。它不会重新运行代理；同一运行重复记账是安全的。响应为 `{ "accounted": … }`。只有查看权限的看板成员会收到 `403`，错误为 `Viewers cannot retry cost recording`。

### 订阅

`GET /api/companies/{companyId}/costs/subscriptions`

在 API 支出旁报告每月订阅费用。它接受与摘要相同的日期筛选项，但只有令牌总数受日期范围影响；费用显示的是 `asOf` 所代表的**当前**每月承诺金额。

顶层字段：

- `accounts`——你有权查看的订阅账户列表（自己的个人账户、由你管理的共享账户，以及你是受众成员的共享账户）。
- `monthlyTotals`——按货币分别汇总的条目，包含 `currency`、`amountCents` 和 `estimatedCount`。不同货币不会合并。
- `activeCount`、`unknownPriceCount`、`unidentifiedAccountCount`。
- `api`、`subscription`、`unknown`、`unattributedSubscription`——范围内按计费类型拆分的公司用量。每项都有 `inputTokens`、`cachedInputTokens`、`outputTokens`、`costCents`、`eventCount`、`estimatedEventCount` 和 `unpricedEventCount`。`unknown` 包含点数、固定价格和无法识别的计费类别。
- `canRefresh`——你是否可以触发账户发现。

每个账户包括 `id`、`provider`、`name`、`ownerUserId`、`ownerName`、`shared`、`identityVerified`、`detectedPlan`、`observedAt`、`lastCheckedAt`、`refreshStatus`（`ok` 或 `unavailable`）、`canEdit`、范围内的 `usage`、使用该账户的 `agents`，以及以下 `price` 字段：

- `revision`、`plan`、`amountCents`、`monthlyCents`、`currency`、`effectiveAt`。
- `cadence`——`month` 或 `year`（年费金额会除以 12 得到 `monthlyCents`）。
- `status`——`active`、`ended` 或 `excluded`。
- `source`——`catalog`（公开标价，因此是估算）、`user` 或 `unknown`。

`POST /api/companies/{companyId}/costs/subscriptions/refresh`

在后台检查你可以查看的已连接订阅账户，以获取账户身份和套餐信息。会立即以 `202` 和 `{ "status": "accepted" }` 响应。它只使用你已有权限使用的凭据。检查任务过多时返回 `429`，错误为 `Subscription account checks are busy. Try again shortly.`。查看者会收到 `403`。

`PATCH /api/companies/{companyId}/costs/subscriptions/{subscriptionId}`

记录你实际支付的价格。请求体：

- `expectedRevision`——你读取到的 `price.revision`。如果其他人在此期间修改了价格，会返回 `409` 和 `This subscription changed. Reload it before saving.`。
- `plan`——1–100 个字符。
- `amountCents`——数字或十进制字符串；价格未知时可设为 `null`。
- `currency`——`USD`、`EUR`、`GBP`、`CAD`、`AUD` 或 `CHF`。
- `cadence`——`month` 或 `year`。
- `status`——`active`、`ended` 或 `excluded`。

响应为 `{ "id": …, "revision": … }`。每次更新都会新增一个自当前时间生效的价格修订版本，较早版本会保留。所有者可以编辑自己的个人账户；共享账户则需要管理 AI 连接的权限。其他调用者会收到 `403` 和 `Only subscription owners or managers of shared accounts can edit prices`。此处设置的 `status` 不会取消提供商那边的任何服务。

`POST /api/companies/{companyId}/costs/subscriptions/{subscriptionId}/link`

```json
{ "targetId": "<subscription uuid>", "expectedRevision": 3, "targetRevision": 1 }
```

将同一提供商中的两个账户标记为同一项订阅或席位，以便费用只计算一次，并合并用量。路径中的账户会合并到 `targetId`，并保留目标账户的价格。你必须同时有权编辑两个账户，否则返回 `403`。如果两个 ID 指向同一账户、来源提供商不同，或读取之后任一修订版本发生变化，会返回 `409`。

### 快速变化的支出

`GET /api/companies/{companyId}/costs/window-spend`

返回最近 `5h`、`24h` 和 `7d` 的滚动支出。无需自行选择日期范围即可发现支出突然增长。

---

## 预算控制

### 设置公司预算

`PATCH /api/companies/{companyId}/budgets`

这是设置公司月预算的最简单方式。实现过程中还会在后台同步相应的公司预算策略。

<a id="set-agent-budget"></a>
### 设置代理预算

`PATCH /api/agents/{agentId}/budgets`

设置指定代理的月预算，并同步对应的代理预算策略。

两个端点的请求体相同：

```json
{ "budgetMonthlyCents": 5000 }
```

公司和代理月预算均按 UTC 日历月计算。发送 `0` 会关闭对应策略，而不是将预算上限设为零美元。

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -X PATCH "http://localhost:3100/api/companies/company-1/budgets" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{ "budgetMonthlyCents": 100000 }'
```
<!-- tab: JavaScript -->
```js
await fetch("http://localhost:3100/api/companies/company-1/budgets", {
  method: "PATCH",
  headers: {
    Authorization: "Bearer <token>",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ budgetMonthlyCents: 100000 }),
});
```
<!-- tab: Python -->
```python
import requests

requests.patch(
    "http://localhost:3100/api/companies/company-1/budgets",
    headers={"Authorization": "Bearer <token>"},
    json={"budgetMonthlyCents": 100000},
)
```
<!-- /tabs -->

### 预算概览

`GET /api/companies/{companyId}/budgets/overview`

需要看板级预算健康概况时使用此端点。返回：

- 当前策略。
- 活动中的预算事件。
- 已暂停代理和项目的数量。
- 待审批数量。

每项策略摘要还会提供对应时间窗口的 `unpricedEventCount` 和 `pendingRunCount`、`reservationCents` 以及 `unpricedUsagePolicy`，帮助你了解为何某个范围在未达到预算上限时仍受阻。

### 创建或更新策略

`POST /api/companies/{companyId}/budgets/policies`

这是通用预算策略 API。需要为公司、代理或项目设置策略，而不仅是设置公司月预算时，请使用此端点。

请求字段：

- `scopeType` 和 `scopeId`——必填；指定策略适用的公司、代理或项目。
- `amount`——以整数美分表示的上限。创建策略时必填，更新时可选；`0` 会关闭预算执行。
- `reservationCents`——可选的单次运行预留金额（见[预算预留](#预算预留)）；可以是大于等于零的数字或十进制字符串。
- `unpricedUsagePolicy`——可选，`block` 或 `allow`（见下文）。
- `metric`、`windowKind`、`warnPercent`（1–99）、`hardStopEnabled`、`notifyEnabled`、`isActive`——可选。

实现采用的重要默认值：

- `metric` 默认为 `billed_cents`。
- 公司和代理范围的 `windowKind` 默认为 `calendar_month_utc`。
- 项目范围的 `windowKind` 默认为 `lifetime`。
- `warnPercent` 默认为 `80`。
- `hardStopEnabled` 默认为 `true`。
- `notifyEnabled` 默认为 `true`。
- `unpricedUsagePolicy` 默认为 `block`。
- `reservationCents` 默认为 `0`。
- `isActive` 默认为 `true`。

这些默认值适用于首次创建策略。更新现有策略时，省略字段会保留其当前值，因此只改 `unpricedUsagePolicy` 或 `reservationCents` 时无需重复发送其他字段。

### 预算预留

预算预留是 Paperclip 在运行开始前从预算中暂时扣留的估算金额。它可以避免多个运行同时启动，并且每个运行单独看都未超限，但合计却突破预算上限。

- 在每项策略上用 `reservationCents` 设置。默认值 `0` 表示不预留金额。
- 运行开始时，Paperclip 会在所有适用的活动硬停止策略中，取最大的 `reservationCents` 作为预留金额。
- 如果已记录支出加上现有预留金额将达到上限，运行就不会启动，原因是 `Available budget is reserved by unfinished runs`。
- 成本记录完成后会释放预留。超时或服务器重启都不会释放，直到记账完成为止。

预留只控制工作能否启动，不能限制提供商最终收取的实际运行费用；实际费用可能高于估算值。

### 预算事件

`POST /api/companies/{companyId}/budget-incidents/{incidentId}/resolve`

允许的操作：

- `keep_paused`
- `raise_budget_and_resume`

对于软（警告）事件，`keep_paused` 会关闭该警告；该范围原本就不会被暂停。无论是哪种事件，`raise_budget_and_resume` 都会提高上限。

提高预算时，必须提供大于当前观测支出的新 `amount`。在以下情况中，提高预算仍会被拒绝，并返回 `422`：

- 该范围内已完成运行仍等待记账（`Completed runs must finish accounting before resuming`）。
- 该范围存在未定价用量且策略设为 `block`（`Resolve unpriced usage or explicitly allow it in the budget policy before resuming`）。

> **警告：** 硬性预算限制会暂停受影响的范围，并取消该范围中的工作。只有新预算高于当前观测支出且会计数据完整时，提高预算才会恢复该范围。

### 达到阈值时的行为

- 达到警告阈值时，如果启用了通知，服务会创建软事件。软事件只发出警告，不会暂停任何内容。
- 达到 100% 时，如果启用了硬性限制，服务会创建硬事件。
- 如果暂时无法确认支出金额，启用硬性限制的策略还会在达到 100% 前阻止新工作：例如预算窗口内的已完成运行仍在等待记账，或窗口中有未定价用量且 `unpricedUsagePolicy` 设为 `block`。如果希望工作继续，并将未定价用量视为未知，可将该策略设为 `allow`。
- 硬性限制策略会暂停受影响的公司、代理或项目，并取消该范围中的工作。
- 预算窗口按 UTC 时区每月第一天重置。
- 项目策略默认使用生命周期窗口，除非显式选择其他窗口类型。

---

## 财务事件

这些端点用于记录令牌成本事件以外的会计事件，例如点数、退款、平台费用、手动调整及其他非原始令牌用量的财务记录。

### 创建财务事件

`POST /api/companies/{companyId}/finance-events`

此端点仅限看板使用。

`amountCents` 可以是数字或十进制字符串，不能为负数。`currency` 可以是任意三字母代码（存储时会转换为大写，默认 `USD`）。可能会重试同一写入时，请发送 `idempotencyKey`（最多 200 个字符）。

### 读取财务数据

- `GET /api/companies/{companyId}/costs/finance-summary`
- `GET /api/companies/{companyId}/costs/finance-by-biller`
- `GET /api/companies/{companyId}/costs/finance-by-kind`
- `GET /api/companies/{companyId}/costs/finance-events`

财务事件列表端点支持：

- `from`
- `to`
- `limit`：1–500，默认 100。

常见事件类型包括 `inference_charge`、`platform_fee`、`credit_purchase`、`credit_refund` 和 `manual_adjustment`。API 还会保存 `direction`、`amountCents`、`currency`、`estimated` 和可选元数据。

Paperclip 不会进行货币换算：

- `finance-summary` 的主要汇总字段 `debitCents`、`creditCents`、`netCents` 和 `estimatedDebitCents` 均以美元表示（`currency: "USD"`），同时通过 `currencies` 数组逐币种提供汇总。
- `finance-by-biller` 和 `finance-by-kind` 的每行都带有自己的 `currency`。
- 从[提供商成本报告](#导入提供商成本报告)导入的金额不会计入这些汇总，因为它们与发票和运行估算值重叠。`finance-summary` 会通过 `providerReportedCents` 和 `providerReportedCentsExact` 单独报告，不要将它们与其他汇总相加。

---

<a id="accounting"></a>
## 会计

这些端点用于检查成本账本是否完整，以及金额是否与提供商实际计费一致。它们仅供看板使用：调用者必须是有权访问公司的看板用户。代理不能调用，即使查询自己的运行也不行。

### 会计健康状态

`GET /api/companies/{companyId}/accounting/health`

从此处开始检查。它会返回简要状态：

- `pendingRunCount` 和 `oldestPendingAt`——已完成但尚未记录成本的运行。
- `unpricedEventCount`——没有可靠价格的用量。
- `pendingCancellationCount`——因预算停止但尚未完成取消的工作。
- `heldReservationCents`——[预算预留](#预算预留)当前扣留的总额。
- `items`——最多 100 条待处理运行和最多 100 条未定价事件。每项包含 `runId`、`costEventId`（仅未定价事件有）、`agentId`、`state`、`lastError`、`attempts`、`since` 和 `lastAttemptAt`。`state` 可能为：
  - `waiting_for_receipt`——该运行的用量报告尚未送达。
  - `retryable`——记账失败，可以重试。
  - `unpriced`——该事件需要价格。

### 重试运行记账

`POST /api/companies/{companyId}/accounting/retry`

```json
{ "runId": "<run uuid>" }
```

重新尝试记录某个运行的成本。响应为 `{ "accounted": … }`。再次失败时，错误会保存到运行记录中，并在健康状态中以增加后的 `attempts` 计数显示。

### 检查和修复汇总数据

`GET /api/companies/{companyId}/accounting/inspect`

Paperclip 会在单条成本事件之外维护公司和代理的月累计支出，以及代理生命周期总支出。`inspect` 会从头重新计算这些汇总并列出所有不一致项。响应包含 `companyId`、`checkedAt`、`fingerprint` 和 `findings`。每项问题包含 `kind`、关联的 `entityId`、`actual`、`expected` 和 `repairable`：

- `company_projection`、`agent_projection`、`runtime_projection`——汇总值与事件不匹配，可以修复。
- `missing_receipt`、`missing_acknowledgement`、`receipt_mismatch`、`legacy_runtime`——运行记录不完整或与用量报告不符，需要进一步调查；修复操作不会修改它们。

`POST /api/companies/{companyId}/accounting/repair`

```json
{ "fingerprint": "<64-character fingerprint from inspect>", "reason": "Month-end check" }
```

修复操作会将所有可修复的汇总重置为重新计算的值，并返回新的检查结果。`reason` 为必填项（最多 1,000 个字符），会写入活动日志。如果检查后数据发生变化，fingerprint 就不再匹配，操作会返回 `409`；请重新检查后再重试。

<a id="import-provider-invoices"></a>
### 导入提供商发票

`POST /api/companies/{companyId}/accounting/invoices`

导入提供商发票到 Paperclip，以便与已记录金额比较。请求体：

- `biller`——发票签发方（最多 150 个字符）。
- `externalId`——发票自身的 ID（最多 200 个字符）。
- `currency`——三字母货币代码。
- `lines`——1–2,000 条明细，每条包括：
  - `externalId`——发票内唯一的明细 ID。
  - `kind`——`inference`（默认）、`fee` 或 `credit`。
  - `amountCents`——大于等于零的数字或十进制字符串。
  - `occurredAt`——带时区偏移量的 ISO 日期时间。
  - 可选 `costEventId`、`runId`、`providerRequestId` 和 `model`，用于匹配已记录费用。
  - 可选 `pricing`——与成本事件中的 `pricingProvenance` 结构相同。

成功时返回 `201` 和已存储发票。每条发票明细还会转换为财务事件：`inference` 明细作为费用、`fee` 明细作为平台费、`credit` 明细作为退款；如果匹配费用已在财务账本中，或某条明细可能匹配多个费用，则不会新建财务事件。重复导入同一发票会返回现有记录；用相同发票 ID 导入不同内容会返回 `409`。

`GET /api/companies/{companyId}/accounting/invoices` 会列出最近导入的 100 张发票。

### 对账发票

`GET /api/companies/{companyId}/accounting/invoices/{invoiceId}`

返回 `invoice` 及其 `lines`。每条明细会显示 `amountCents`、匹配费用的 `matchedEventId` 和 `recordedCents`、二者差值 `differenceCents` 以及 `status`：

- `matched`——发票和账本金额一致。
- `difference`——已匹配，但金额不同（或已记录费用尚未定价）。
- `unmatched`——找不到匹配的已记录费用。
- `ambiguous`——有多个费用符合，或多条发票明细指向同一费用。
- `non_inference`——费用或点数明细，不参与此类比较。
- `unsupported_currency`——发票不是 USD，无法与 USD 账本比较。

<a id="correct-a-charge"></a>
### 更正费用

`POST /api/companies/{companyId}/accounting/events/{eventId}/adjustments`

用于修复已记录成本事件的金额，例如为未定价费用补充价格，或按已对账发票的 `difference` 调整金额。请求体：

- `idempotencyKey`——必填，最多 200 个字符。
- `expectedCents`——你认为当前记录的金额。如果你查看后费用发生变化，请求会返回 `409`，以便重新审查。
- `correctedCents`——更正后的金额。
- `reason`——必填，最多 1,000 个字符。
- `pricing`——必填，说明新价格的来源（与 `pricingProvenance` 结构相同）。
- `invoiceLineId`——可选，将更正与导入的发票明细关联。该明细必须是同一收费方的 USD `inference` 明细、在对账中匹配此事件、金额正好为 `correctedCents`，且此前未被使用。

成功时返回 `201` 和更正记录。Paperclip 会保留提供商原始金额，更新事件的有效成本，重新计算月初至今的支出，并再次检查预算。如果 `pricing.source` 是 `rate_card`，事件的 `costStatus` 变为 `estimated`，否则变为 `reported`，因此更正未定价费用后会将其从未定价计数中移除。

`GET /api/companies/{companyId}/accounting/events/{eventId}/adjustments` 会按时间从新到旧列出某项事件的更正。每项包含 `previousCents`、`correctedCents`、`previousStatus`、`previousPricing`、`pricing`、`reason`、`actorId` 和 `createdAt`。

### 导入提供商成本报告

`POST /api/companies/{companyId}/accounting/provider-costs/import`

直接从 OpenAI 或 Anthropic 组织级成本报告导入每日总额。此操作会将已存储的管理员凭据发送给提供商，因此除看板访问权限外，还需要公司所有者/管理员、实例管理员或本地可信安装权限。其他调用者会收到 `403`，错误为 `Company admin access required to import provider billing.`。

开始前，请将提供商管理员密钥保存为公司秘密，并在通过[秘密 API](./secrets.md) 创建或更新秘密时设置其 `providerMetadata` 字段，标记该秘密用于计费：

```json
{ "providerBilling": { "provider": "openai", "accountId": "<organization id>" } }
```

Paperclip 只会使用标记了相同提供商和账户的公司秘密。

请求体：

- `provider`——`openai` 或 `anthropic`。
- `secretId`——已标记的秘密 ID。
- `accountId`——提供商组织 ID（只可包含字母、数字、`_` 和 `-`）。
- `scopeIds`——1–25 个唯一 OpenAI 项目 ID 或 Anthropic workspace ID。
- `from` 和 `to`——普通日期。`to` 不包含在范围内；范围最多 31 天，且必须在今天（UTC）或之前结束，因此只能导入已完成日期的数据。

响应为 `{ "daysRead": …, "eventsCreated": … }`。每天及每个项目/workspace 都会转换为财务事件。重复导入相同日期时只记录差额，并以更正形式记为借项或贷项。只接受 USD 报告。如果其他人导入了更新的数据，而导入期间你的数据仍在加载，请求会失败并提示重新获取报告。

---

## 内部诊断

`GET /api/companies/{companyId}/costs/quota-windows`

此端点仅供看板使用，为成本页面的订阅配额面板提供数据。公司必须存在，且调用者必须通过看板认证。

端点会为调用者有权查看的 AI 连接中已关联的每个 Anthropic 或 OpenAI 订阅账户返回一项。读取使用的是账户自身保存的凭据，而不是运行 Paperclip 的机器上的登录状态。每项包含 `provider`、`accountKey`（账户及凭据版本的稳定 ID，绝不包含凭据本身）、`accountLabel`、`ok`、`windows`，以及读取成功时的 `capturedAt`。读取失败时会返回 `ok: false` 和 `errorFamily`，例如 `credentials_unavailable`、`authentication_required`、`permission_denied` 或 `provider_unavailable`，并给出通用消息，而不暴露提供商的原始错误。用户没有此类账户时，Anthropic 和 OpenAI 各返回一条占位条目，字段为 `errorFamily: "credentials_unavailable"`。

---

## 实用的读取顺序

需要了解公司的支出时，可以从以下步骤开始：

1. `GET /costs/summary`
2. 如果有代理使用订阅，运行 `GET /costs/subscriptions`。
3. `GET /costs/by-agent`
4. `GET /costs/by-project`
5. `GET /budgets/overview`
6. `GET /costs/window-spend`

通常这套顺序能帮你判断问题来自单个代理、单个项目、提供商组合，还是实际预算策略。如果摘要显示 `pricingComplete: false`，请从 `GET /costs/data-quality` 开始查找受影响的运行。如果某个范围在预算上限之下仍被阻止，再检查 `GET /accounting/health`。
