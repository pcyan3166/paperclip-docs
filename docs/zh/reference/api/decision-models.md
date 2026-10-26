---
paperclip_version: v2026.1023.0
seo_title: 决策模型 API
seo_description: 配置公司用于小模型决策的共享 OpenAI 或 OpenRouter 连接，运行会产生费用的设置测试，并读取请求历史。
---

# 决策模型

Paperclip 的一些可选功能会向小型快速模型提出简短的结构化问题，而不是唤醒完整智能体，例如询问是/否概率、从短列表中选择一项，或按量表给出评分。**决策模型**是公司选择用于支付此类请求费用的唯一共享 API 连接。

使用此 API 可以：

- 查看和更改公司使用的连接；
- 检查决策功能是否可用；
- 运行固定的设置测试；
- 列出近期请求及其费用。

没有用于提出自定义问题的通用端点，只有 Paperclip 内置功能会发送决策请求。在应用中，这些设置位于 **Company Settings** 的 **Decision model** 区域；相关请求历史位于 **Audit → Costs** 下的 **Decisions** 视图。参见[成本与预算](../../guides/day-to-day/costs.md#decisions)。

所有路径均位于 `/api/companies/{companyId}/decision-model` 下。如果你刚开始使用 API，请先阅读 [API 概览](./overview.md)，了解基础 URL、身份验证和错误约定。

> **注意：这不是决策队列。** 本页介绍小模型请求。由智能体提出、等待人员答复的决策见[决策 API](./decisions.md)。

---

## 支持的服务提供商

| 服务提供商 | 模型 |
|---|---|
| OpenAI | `gpt-6-luna`（GPT-6 Luna） |
| OpenRouter | `typesafe/jev-1.13`（Jev 1.13） |

连接必须是以上服务提供商之一提供的公司共享 API 密钥，并且使用该服务提供商自己的端点。不能使用个人账户、订阅登录或自定义端点。

---

## 各类调用方的权限

| 端点 | 调用方 |
|---|---|
| `GET /` | 有权访问公司的看板用户 |
| `PUT /` | 有权管理 AI 连接的看板用户 |
| `GET /availability` | 任何有权访问公司的调用方 |
| `POST /test` | 有权管理 AI 连接的看板用户 |
| `GET /history` | 有权访问公司且可以读取公司范围数据（`company_scope:read`）的调用方 |

无权管理连接的调用方在调用 `PUT /` 或 `POST /test` 时会收到 `403` 和 `Manage connections permission is required`。

---

## 读取设置

`GET /api/companies/{companyId}/decision-model`

响应包含：

- `canManage`：你是否可以更改设置。
- `settings`：当前配置；无权管理时为 `null`。
- `choices`：可供你选择的兼容共享连接；无权管理时为空。每个选项都包含连接详情和一个 `decisionModel` 名称。

`settings` 包含：

- `enabled`：是否启用决策功能。
- `connectionId` 和 `grantId`：所选连接及其凭据授权；未设置时为 `null`。
- `allowBackground`：没有责任人时，后台功能是否可以对该连接计费。
- `provider`：`openai`、`openrouter` 或 `null`。
- `model`：模型 ID 或 `null`。
- `companyId`。

---

## 更改设置

`PUT /api/companies/{companyId}/decision-model`

```json
{
  "enabled": true,
  "connectionId": "<connection uuid>",
  "grantId": "<grant uuid>",
  "allowBackground": true
}
```

- `enabled`：必填。
- `connectionId` 和 `grantId`：必填，必须同时设置，或同时为 `null`。
- `allowBackground`：可选。

没有连接时不能启用决策功能；请求会因 `Select a connection before enabling decisions` 验证失败。响应为更新后的 `settings` 对象。

首次为公司配置决策模型时，`allowBackground` 默认**开启**。关闭后，它会保持关闭状态，即使之后更改连接，或在后续更新中省略此字段也一样。由后台代付发起的请求会归因于 **Paperclip services**，而不是某个人。

将 `enabled` 设为 `false` 会阻止新请求，但不会取消或删除已经发送的请求。如果所选连接被删除，设置会保留，但请求会被阻止，直到你选择其他连接。

---

## 检查可用性

`GET /api/companies/{companyId}/decision-model/availability`

返回 `{ "available": true }`，或者返回 `{ "available": false, "reason": "…" }`，其中 `reason` 可能为以下值之一：

- `not_configured`
- `disabled`
- `connection_unavailable`
- `incompatible_connection`
- `access_denied`
- `responsible_user_missing`
- `background_disabled`
- `budget_blocked`

这是对配置和当前权限的快速本地检查，不会联系服务提供商。因此，即使 `available: true`，也不能保证服务提供商可访问，或预算一定会允许请求。

---

## 运行设置测试

`POST /api/companies/{companyId}/decision-model/test`

向已配置的模型发送一组固定的三个示例问题：是/否问题、选择题和评分题。你不能提供自定义提示词、模型或身份。**此操作会产生真实的 API 费用。**

响应为以下形式之一：

- `{ "status": "succeeded", "invocationId", "answers", "usage" }`
- `{ "status": "failed", "invocationId", "errorCode", "usage" }`
- `{ "status": "unavailable", "reason" }`，其中 `reason` 来自上方的可用性列表。

`usage` 包含 `inputTokens`、`outputTokens`、`costCents`（十进制字符串或 `null`）和 `costStatus`（`reported`、`estimated` 或 `unpriced`）。

每个答案都包含 `type`：

- `boolean`：`probability`，范围为 0 到 1。
- `choice`：所选的 `choice`。
- `score`：按顺序排列的评判标准给出的 `score`，从零开始计数。

如果服务提供商提供了 `probabilities` 和 `confidence`，答案也可能包含这两个字段。测试答案不会保存，但用量和费用会保存。

请求时限为 30 秒，并且不会重试。超时请求的费用会保持未解决状态，其预算预留也会继续占用。运维人员可以使用[更正费用](./costs.md#correct-a-charge)来结算，包括记录已验证的零费用。

---

## 读取历史记录

`GET /api/companies/{companyId}/decision-model/history`

接受与 [Costs API](./costs.md#choosing-a-date-range) 相同的 `from`、`to` 和 `period` 过滤器，另有 `limit`，范围为 `1` 到 `500`，默认值为 `100`。与成本报告不同，省略日期时不会限定为当前月份，而是返回历史上最近的请求。结果按时间倒序排列。

每条记录包含：

- `id`、`feature`、`startedAt`、`finishedAt`、`durationMs`；
- `actorType`、`responsibleUserId`、`userName`；
- `agentId`（始终包含）；
- `issueId`、`issueIdentifier`、`runId`（只有你有权查看相关任务时才包含任务和运行关联）；
- `connectionId`、`provider`、`model`；
- `status` 和 `errorCode`；
- `inputTokens`、`outputTokens`、`costCents`、`costStatus`。

设置测试事件会记录 `feature: "settings.test"`。Paperclip 不会保留输入内容、问题、答案或服务提供商的原始响应，只保留这些元数据。

---

## 成本与预算

决策请求属于普通支出：

- 每次请求都会记录在成本账本中，并计入公司预算，以及适用于该工作运行的智能体和项目预算。
- 没有关联智能体的请求会在 [`costs/by-agent`](./costs.md#breakdown-views) 中显示为 `Paperclip services` 条目（`agentId: null`）。
- 成本会保留到小于一美分的精度。OpenRouter 使用其报告的成本；OpenAI 的成本根据固定费率估算。没有可靠价格时，成本为未知，绝不会记为零。

---

## 后续阅读

- [成本与预算](../../guides/day-to-day/costs.md#decisions)：在应用中配置决策模型并查看 Decisions 视图。
- [成本 API](./costs.md)：查看其他支出和预算端点。
