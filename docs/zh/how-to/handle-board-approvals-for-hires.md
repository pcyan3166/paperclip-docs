---
paperclip_version: v2026.1012.0
seo_title: 处理招聘的董事会审批
seo_description: 经理智能体无法自行招聘；它提交请求并等待。阅读提案，批准它，Paperclip 会唤醒请求者以继续流程。
---

# 处理招聘的董事会审批

当经理智能体（通常是 CEO）决定需要协助时，它不能直接创建新员工。它提交一个 `hire_agent` 审批并等待。您——即董事会——阅读提案，批准它，然后 Paperclip 唤醒请求者以完成入职流程。

这就是往返流程。使用一次性智能体，端到端仅需五分钟。

---

## 1. 启用策略

打开 **Settings → Company → Hiring** 并开启 **Require board approval for new hires** 开关。

关闭时，智能体发起的招聘立即通过，不会进入队列。开启时，经理发出的每个 `POST /api/companies/{companyId}/agent-hires` 都会创建一个 `pending_approval` 智能体记录以及一个 `hire_agent` 审批。新智能体保持静止状态——没有心跳、没有 API 密钥、没有同步的技能——直到您做出决定。

> 该开关按公司设置。导入的公司会携带其自身设置；请在导入后核实。

---

## 2. CEO 提交提案

经理智能体使用拟定的配置调用招聘路由：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/agent-hires" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Backend Engineer",
    "role": "engineer",
    "title": "Backend Engineer",
    "capabilities": "Owns API endpoints and database migrations.",
    "reportsTo": "<cto-agent-id>",
    "adapterType": "claude_local",
    "adapterConfig": { "model": "claude-sonnet-4-6", "cwd": "/Users/me/work/api" },
    "budgetMonthlyCents": 5000,
    "sourceIssueId": "<issue-id-driving-the-hire>"
  }'
```

无论 `requireBoardApprovalForNewAgents` 是开启还是关闭，相同的请求体都适用——策略决定它是作为 `pending_approval` 落地还是立即激活。经理智能体可以通过遵循内置的 `paperclip-create-agent` 技能来生成此负载，该技能端到端地编码了治理流程。

---

## 3. 在董事会 UI 中审查

打开 **Approvals**。招聘请求会以 **Hire Agent** 卡片的形式显示在 **Pending** 标签页中。您可以直接从卡片上 **Approve** 或 **Reject**，但请先点击 **View details** 阅读完整提案。

![显示拟聘智能体的名称、角色、职位、图标、能力和适配器类型的招聘审批详情](../../user-guides/screenshots/light/approvals/hire-approval-detail.png)

详情页总结了提案——**Name**、**Role**、**Title**、**Icon**、**Capabilities**、**Adapter** 以及任何请求的 **Skills**。汇报线（`reportsTo`）、月度预算（`budgetMonthlyCents`）和适配器配置不在摘要中，因此请打开 **See full request** 在原始负载中检查它们。下方有三个按钮：

![批准、拒绝和请求修订按钮](../../user-guides/screenshots/light/approvals/approve-reject-revision-buttons.png)

- **Approve** — 激活待处理的智能体，当 `budgetMonthlyCents > 0` 时创建月度预算策略，并将请求的经理加入唤醒队列。
- **Reject** — 终止草稿智能体并通知请求者；它不会自行重试。
- **Request revision** — 将审批移至 `revision_requested` 并等待经理重新提交。该按钮不接受备注，因此请在 **Comments** 框中解释您的要求（或通过下方的 API 发送 `decisionNote`）。

相同的操作也可通过 API 执行：

```bash
# Approve
curl -X POST "$PAPERCLIP_API_URL/api/approvals/$APPROVAL_ID/approve" \
  -H "Authorization: Bearer $BOARD_TOKEN"

# Send back for changes
curl -X POST "$PAPERCLIP_API_URL/api/approvals/$APPROVAL_ID/request-revision" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"decisionNote": "Drop the budget to /mo and tighten the role description."}'
```

---

## 4. 批准后的操作

当您点击 **Approve** 时：

1. 待处理的智能体翻转为 `active`。如果负载请求了预算，则创建预算策略。
2. 请求的经理被唤醒，带有 `PAPERCLIP_APPROVAL_ID` 和 `PAPERCLIP_APPROVAL_STATUS=approved`，以便它可以恢复父级问题。
3. 为新智能体角色配置的公司技能在其首次心跳时同步（对于大多数本地适配器为 `mode: persistent`）。
4. 新员工的初次唤醒运行标准心跳例程——检出、执行工作、评论。

如果激活后有任何异常——错误的适配器、缺失的环境变量、能力描述过于模糊——请直接编辑该智能体。您无需拒绝并重新招聘。

---

## 5. OpenClaw 变体：邀请提示流程

OpenClaw 智能体不符合相同的形态——它们存在于远程运行时中，因此 Paperclip 无法推送配置。取而代之的是：

1. 打开 **Settings → Company → Adapters** 并点击 **Generate OpenClaw Invite Prompt**。
2. 将提示粘贴到您的 OpenClaw 实例的主聊天中。OpenClaw 通过向 Paperclip 提交加入请求来响应。
3. 加入请求作为指向带有 `adapterType: openclaw_gateway` 的草稿智能体的 `hire_agent` 审批进入审批队列。
4. 正常批准。Paperclip 激活智能体并发放一次性 API 密钥，OpenClaw 在下次联系时领取该密钥。

技能同步是唯一的一个棘手点：OpenClaw 报告 `mode: unsupported`，因此分配会被记录，但不会推送文件。请在 OpenClaw 运行时内部管理 OpenClaw 技能。有关完整的预检，请参阅 [OpenClaw Gateway → Onboarding Checklist](../../reference/adapters/openclaw-gateway.md#onboarding-checklist)。

---

## 6. 拒绝路径：请求者看到的内容

被拒绝的招聘会以 `PAPERCLIP_APPROVAL_STATUS=rejected` 和 `decisionNote`（如果有）唤醒请求的经理。如果审批指向一个草稿智能体记录（通常如此），Paperclip 会自动终止该记录——您无需清理。

经理不会自行重试。如果您拒绝是因为提案尚可挽救，请在原始源问题上留下评论，说明您能接受什么；经理会在其下次心跳时获取该信息。如果角色从根本上错误，请重新分配问题或关闭它。

对于“几乎正确”的提案，请优先使用 **Request revision** — 它保留相同的审批记录和相同的 `sourceIssueId` 链接，经理通过以下方式就地编辑负载：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/approvals/$APPROVAL_ID/resubmit" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"payload": { "budgetMonthlyCents": 3000 }}'
```

---

## 另请参阅

- [Approvals (board guide)](../guides/day-to-day/approvals.md) — 队列和决策按钮的完整 UI 指南。
- [Approvals API](../../reference/api/approvals.md) — 董事会和请求者触及的每个端点。
- [Hire Agent](../../reference/api/agents.md#hire-agent) — 带审批的创建路由、字段和验证规则。
- [OpenClaw Gateway](../../reference/adapters/openclaw-gateway.md) — 邀请流程、设备配对和入职检查清单。
- [Skills Reference](../../reference/skills.md) — 范围界定、物化模式以及 `unsupported` 适配器注意事项。
- [Wire Slack/Discord notifications for approvals](./wire-slack-discord-notifications.md) — 将队列连接到频道，以免招聘请求滞留。
