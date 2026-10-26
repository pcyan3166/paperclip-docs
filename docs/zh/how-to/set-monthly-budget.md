---
paperclip_version: v2026.1023.0
seo_title: 设置月度预算并强制执行
seo_description: 配置错误的循环可能在周末内产生四位数的账单。设置一个由 Paperclip 自动执行的限额，并在硬性停止前发出警告。
---

# 设置月度预算并强制执行

自主智能体会发起真实的 API 调用。如果没有制动机制，配置错误的循环、过于积极的提示词或价格变动可能在周末内产生四位数的账单。Paperclip 的预算系统就是这道制动：你设置一个上限，Paperclip 在 80% 时发出警告，在 100% 时硬性停止，并暂停超出预算限制的范围，直到你提高上限或日历月重置。

启用预算所需时间：约 5 分钟。固定版本：Paperclip API v1，UTC 日历月窗口。

---

## 1. 设置公司级上限

这是最简单的路径。一次调用即可为整个公司设置月度上限：

```bash
curl -X PATCH "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/budgets" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "budgetMonthlyCents": 10000 }'
```

`budgetMonthlyCents` 的单位是分——`10000` 表示每月 100 美元。Paperclip 会存储该值，同步一个具有默认阈值（`warnPercent: 80`、`hardStopEnabled: true`、`notifyEnabled: true`）的匹配 `company` 预算策略，并返回更新后的记录。

相同的结构也适用于单个智能体：

```bash
curl -X PATCH "$PAPERCLIP_API_URL/api/agents/$AGENT_ID/budgets" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "budgetMonthlyCents": 2500 }'
```

单个智能体的上限独立于公司上限执行。即使公司仍有预算余量，处于 100% 的智能体也会暂停——这可以防止某个失控的智能体耗尽其他所有人的预算。

---

## 2. 通过策略 API 调整阈值

`PATCH /budgets` 适用于常见情况。当你需要非默认的 `warnPercent`、项目范围策略，或希望禁用硬性停止并仅依靠警告运行时，请直接使用 upsert 路由：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/budgets/policies" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "scope": "agent",
    "scopeId": "'$AGENT_ID'",
    "amountCents": 2500,
    "warnPercent": 70,
    "hardStopEnabled": true,
    "notifyEnabled": true,
    "windowKind": "calendar_month_utc"
  }'
```

如果省略字段，默认值如下：

| 字段 | 默认值 |
|---|---|
| `metric` | `billed_cents` |
| `windowKind` | 公司/智能体为 `calendar_month_utc`，项目为 `lifetime` |
| `warnPercent` | `80` |
| `hardStopEnabled` | `true` |
| `notifyEnabled` | `true` |
| `isActive` | `true` |

项目策略默认为生命周期上限——适用于一次性工作（“无论何时完成，此项目的成本永远不得超过 200 美元”）。如果你希望项目预算每月重置，请传递 `"windowKind": "calendar_month_utc"`。

> 设置 `hardStopEnabled: false` 会将策略变为仅警告模式。该范围在支出超过预算的 100% 后仍会继续产生费用；你只能在概览中看到该事件。请谨慎使用。

---

## 3. 警告显示的位置

在 80%（或你设置的 `warnPercent`）时，Paperclip 针对该策略记录一个**软事件**。在 100% 时，它记录一个**硬事件**并暂停该工作区。在 UI 中，打开侧边栏中的**审计**，切换到**预算**选项卡。未解决的事件会以卡片形式显示在策略列表上方：

![审计页面在预算选项卡上，显示预算控制平面卡片、活动事件和预算策略卡片](../../user-guides/screenshots/light/costs/budgets.png)

警告显示为琥珀色的**… 预算警告**卡片，带有**提高预算**和**忽略警告**按钮。硬性停止显示为红色的**… 硬性停止**卡片，带有**提高预算并恢复**和**保持暂停**按钮。

> **注意：** 在 v2026.1023.0 版本中，警告卡片仍使用硬性停止的设计。其标题为**… 硬性停止**，并显示*“此工作区已暂停”*，但警告本身不会暂停任何操作。截图展示的是来自[修复版本](https://github.com/paperclipai/paperclip/pull/16201)的更正后卡片，该版本将在后续发布中推出。在此之前，请检查策略卡片：**警告**不会暂停工作。只有**硬性停止**或**已暂停**才表示该策略已停止工作。

Paperclip 目前不推送出站 Webhook。如果你希望在事件发生时收到 Slack 或 Discord 通知，请运行一个例程，将 `GET /api/companies/{COMPANY_ID}/budgets/overview` 与游标进行差异比较并发送到 Webhook——其结构与审批通知器相同。参见[配置 Slack/Discord 通知](./wire-slack-discord-notifications.md)。

在配置期间进行快速健全性检查：直接读取概览和支出。

```bash
curl "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/budgets/overview" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```

响应将提供当前策略、未解决事件以及已暂停智能体和项目的计数——这些数字与 UI 中预算控制平面卡片渲染的内容相同。

---

## 4. 使用计费代码归因支出

`billingCode` 是一个自由格式标签，在报告成本事件时打上标记。它让你无需正式建模项目，就能区分*“这 40 美元是营销活动”*和*“这 40 美元是平台重构”*。

适配器在成本事件调用中设置该字段：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/cost-events" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "agentId": "'$AGENT_ID'",
    "provider": "anthropic",
    "model": "claude-sonnet-4-20250514",
    "costCents": 12,
    "occurredAt": "2026-04-15T12:30:00.000Z",
    "billingCode": "campaign-q2-launch"
  }'
```

无需代码更改即可填充该字段的两种方法：

- **来自问题。** 在问题上设置 `billingCode`（`PATCH /api/issues/{issueId}` 配合 `{ "billingCode": "campaign-q2-launch" }`）；在成本事件中传递 `issueId` 的适配器会自动复制该代码。
- **来自智能体。** 为其他智能体创建跨团队工作的管理者应在创建时设置问题上的计费代码，以便后续所有操作继承该代码。

计费代码*用于归因，而非执行*——它们显示在成本报告和每个事件的详细行中。预算上限仍适用于公司、智能体和项目范围。如果你需要对某个计费代码的生命周期支出设置硬性上限，请将其建模为项目，并将策略放在该项目上。

---

## 5. 100% 实际执行的操作

当硬事件触发时：

1. 受影响的范围（公司、智能体或项目）立即暂停。不再安排心跳。
2. 该范围内任何正在进行的运行被取消——问题保持原状，智能体停止处理。
3. 预算控制平面卡片增加**已暂停智能体** / **已暂停项目**计数。
4. 该事件作为卡片出现在**审计**下的**预算**选项卡中，以及**成本**选项卡的**概览**顶部。

![审计 → 预算中 100% 时的智能体预算策略卡片——全红进度条、已暂停状态，以及一条说明：在提高预算或忽略事件之前，心跳处于暂停状态](../../user-guides/screenshots/light/costs/agent-budget-100pct-paused.png)

需要了解的两点：

- **不会丢失工作。** 智能体的任务保持分配状态。当你提高上限或月份重置时，智能体将从中断处继续。
- **审计轨迹记录暂停。** 事件、触发器以及最终解决方式都记录在活动日志中——这对回顾很有用，但在即时调试时作用有限。

要恢复：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/budget-incidents/$INCIDENT_ID/resolve" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "action": "raise_budget_and_resume",
    "amount": 15000
  }'
```

`amount` 必须超过当前观察到的支出，否则解决操作将失败——只有实际留出余量的预算增加才有效。另一个操作 `keep_paused` 确认违规，并让该范围保持暂停，直到月度周期滚动重置。

月份滚动是自动的：在每月 1 日 00:00 UTC，日历月策略重置为零支出，任何*仅*因预算原因暂停的范围都会恢复。生命周期项目策略不会重置——这正是其目的。

---

## 6. 验证其有效性：强制触发峰值

在生产环境中依赖该策略之前，先证明它能暂停。在一个一次性智能体上进行测试。

1. 在智能体上设置一个极小的上限——例如 `$0.50`：

   ```bash
   curl -X PATCH "$PAPERCLIP_API_URL/api/agents/$TEST_AGENT_ID/budgets" \
     -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{ "budgetMonthlyCents": 50 }'
   ```

2. 报告一个合成成本事件，使智能体超过上限。作为Paperclip 控制台用户，你可以为公司中的任何智能体发布事件：

   ```bash
   curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/cost-events" \
     -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
       "agentId": "'$TEST_AGENT_ID'",
       "provider": "anthropic",
       "model": "claude-sonnet-4-20250514",
       "costCents": 60,
       "occurredAt": "'$(date -u +"%Y-%m-%dT%H:%M:%S.000Z")'"
     }'
   ```

3. 读取概览并确认智能体显示为已暂停，且有一个未解决的硬事件：

   ```bash
   curl "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/budgets/overview" \
     -H "Authorization: Bearer $PAPERCLIP_API_KEY"
   ```

   你应该看到 `pausedAgentsCount` 增加 1，并且 `incidents` 中有一条包含 `status: "open"`、`kind: "hard_stop"` 和测试智能体 ID 的条目。

4. 解决并清理：

   ```bash
   curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/budget-incidents/$INCIDENT_ID/resolve" \
     -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{ "action": "keep_paused" }'
   ```

如果测试智能体从未暂停，最可能的原因是：(a) 智能体在公司级别创建，没有单个智能体策略，且你的上限在 `company` 上，而不是 `agent`；(b) 策略上的 `hardStopEnabled` 被设置为 `false`；或 (c) 成本事件被拒绝——智能体身份验证的调用只能发布其自身的成本，且 `occurredAt` 必须是有效的 ISO 字符串。

---

## 另请参阅

- [成本指南](../guides/day-to-day/costs.md) — **审计**下**成本**和**预算**选项卡的完整 UI 演练（包括概览、提供商、计费方和财务视图）。
- [成本 API 参考](../reference/api/costs.md) — 此处涉及的每个端点，以及读取路由（`/costs/summary`、`/costs/by-agent`、`/costs/window-spend`）。
- [处理招聘的董事会审批](./handle-board-approvals-for-hires.md) — 相同的治理流程，但用于招聘时的新智能体预算。
- [在智能体花钱前要求董事会审批](./require-board-approval-before-spend.md) — 支出控制的主动部分：智能体在承诺离散支出前停止并询问董事会，因此此处的上限是安全网，而非唯一的防线。
- [配置 Slack/Discord 通知](./wire-slack-discord-notifications.md) — 将预算事件推送到频道，以便你在检查 UI 之前看到它们。
