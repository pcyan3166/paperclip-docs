---
paperclip_version: v2026.1012.0
seo_title: 支出前需董事会批准
seo_description: 限额是被动响应式的。对于大额、离散的支出——例如每月 40 美元的 SaaS 订阅——让智能体先向人类请示，而不是在达到 80% 时才发出警告。
---

# 智能体支出前需董事会批准

预算限额是被动响应式的。它们在 80% 时发出警告，在 100% 时硬性停止——这对于限制失控情况很有用，但对于*“未经我同意，不要订阅这个每月 40 美元的 SaaS”*这类需求则毫无用处。对于需要人类在资金流动**之前**批准的大额、离散支出，你需要一个审批门：智能体停止操作，发布 `request_board_approval`，并在继续之前等待你的决定。

本指南在一个测试公司上端到端地配置该模式。实现可用的审批往返流程所需时间：约 5 分钟。

---

## 1. 审批与硬性限额——何时使用哪种

| 关注点 | 硬性预算限额 (HT1) | 支出审批 (本指南) |
|---|---|---|
| 触发条件 | 累计支出跨越百分比阈值 | 智能体决定某项操作超出策略 |
| 方向 | 被动——在资金支出后触发 | 主动——在操作发生前触发 |
| 粒度 | 按范围（公司、智能体、项目）每月聚合 | 按决策，提案包含在负载中 |
| 停止对象 | 整个范围暂停（不再有心跳） | 仅停止特定支出；智能体继续处理其他任务 |
| 行动者 | 董事会在 **Audit → Budgets** 下解决事件 | 董事会在审批页面批准/拒绝 |

**两者并用**。限额是你的安全网——它捕捉那些绕过契约的情况。审批门是契约：任何超过 $X 的金额、任何经常性支出、任何不在预批准供应商列表内的支出，都需要董事会在智能体提交前做出决定。

> **已超过硬性限额？** 这会自动生成 `budget_override_required` 审批——参见 [设置月度预算并强制执行](./set-monthly-budget.md) 了解该流程。本指南描述的审批门是智能体发起的 `request_board_approval`，这是一种不同的模式。

---

## 2. 将其作为智能体的契约

Paperclip 不会自动检测“这段代码即将产生支出”。OpenAI 客户端上没有拦截器。审批门存在于智能体自己的剧本中——通常是 `HEARTBEAT.md` 或专用技能——它包含两个部分。

**第一部分：规则。** 在智能体的指令中添加一条简短、机械化的条款，描述什么行为越界：

```md
## Spend gate

Before any of the following, you MUST stop and create a `request_board_approval` on the
current issue. Do not commit, sign up, charge a card, or change a paid plan until you have
been woken with `PAPERCLIP_APPROVAL_STATUS=approved` for that approval.

- Any one-time spend ≥ $25
- Any new recurring subscription, regardless of cost
- Switching a model that increases per-token cost by more than 2x
- Any payment to a third party not already on the approved-vendors list

If the approval is rejected, abandon the spend and comment on the source issue with the
reason. Do not retry without new direction.
```

**第二部分：辅助程序。** 大多数智能体通过一个微小的共享例程来解决规则，该例程创建审批、将问题标记为阻塞并退出。具体形式取决于你的运行时——如果你编写了 webhook 处理器，则是 bash；如果你编写了自定义脚本，则是 Python 或 TypeScript；如果你使用 `claude_local`，则是技能文件。第 3 节展示了端到端的形式。

> **为什么是指令而非代码？** 真正的支出拦截器必须包装智能体可能使用的每个 HTTP 客户端，加上触及凭据的文件系统写入，加上 shell 调用。Paperclip 不在该路径上。将审批门放在智能体的指令中，使其与智能体决定的其他所有内容处于同一层面——可在运行记录中审计，可在 Instructions 选项卡中编辑，与智能体遵循的其他所有例程形式相同。

---

## 3. 提交审批 (`request_board_approval`)

智能体使用 `type: request_board_approval` 向公司审批路由发布请求。负载是董事会看到的提案：

```bash
ISSUE_ID="<the-issue-driving-the-spend>"

curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/approvals" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- <<JSON
{
  "type": "request_board_approval",
  "issueIds": ["$ISSUE_ID"],
  "payload": {
    "title": "Approve Vercel Pro for the staging deploy",
    "summary": "Hosting the staging environment for the customer pilot. Free tier hits the build-minutes cap at the third deploy of the day; Pro removes that ceiling.",
    "recommendedAction": "Approve a Vercel Pro subscription on the team plan billed to the company card.",
    "estimatedCostCents": 2000,
    "cadence": "monthly",
    "vendor": "Vercel",
    "risks": [
      "Recurring charge — needs explicit cancellation if we drop the pilot.",
      "Build minutes still capped per project; very large monorepos can hit the soft cap."
    ],
    "alternativesConsidered": [
      "Stay on free tier and hand-trigger deploys — viable but slows the feedback loop to the customer."
    ]
  }
}
JSON
```

关于该负载的几点说明：

- **`title`** 是董事会在队列中看到的一行摘要。使其足够具体，以便无需打开详细视图即可做出决定。
- **`summary`** 是一段话——解释*为什么*，而不是*是什么*。推荐操作涵盖“是什么”。
- **`recommendedAction`** 是批准后应发生的确切事情。董事会批准的是操作，而非解释。
- **`risks`** 是一个短数组。保持精简；董事会无法采取行动的风险分析是噪音。
- **其他任何**你放入负载的内容（`estimatedCostCents`、`vendor`、`cadence`、`alternativesConsidered`）会出现在详细视图的 **See full request** 下。将此用于你的智能体一致填充的结构化字段——它们使后续审计更容易。

`issueIds` 将审批链接回触发它的问题。董事会使用它从审批导航到工作；智能体使用它在唤醒时找到要恢复的问题。

提交后，智能体应停放源问题并退出：

```bash
APPROVAL_ID=$(curl -s -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/approvals" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d @approval.json | jq -r .id)

curl -X PATCH "$PAPERCLIP_API_URL/api/issues/$ISSUE_ID" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "X-Paperclip-Run-Id: $PAPERCLIP_RUN_ID" \
  -H "Content-Type: application/json" \
  -d "{
    \"status\": \"blocked\",
    \"comment\": \"Waiting on board approval [${APPROVAL_ID}](/PAP/approvals/${APPROVAL_ID}) before continuing.\"
  }"
```

`blocked` 在此处是正确的，即使没有其他问题是阻塞器——解除阻塞者是董事会的决定，而非同级任务。当审批解决时，智能体在唤醒处理器中自行解除阻塞（第 5 节）。

> **幂等性。** 心跳可能重新触发。在发布新审批之前，智能体应检查是否已存在针对相同 `issueId` 且仍处于 `pending`/`revision_requested` 状态的审批。最便宜的检查是列表调用：
>
> ```bash
> curl "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/approvals?status=pending,revision_requested" \
>   -H "Authorization: Bearer $PAPERCLIP_API_KEY" | jq '[.[] | select(.payload.title=="Approve Vercel Pro for the staging deploy")] | length'
> ```
>
> 如果计数非零，不要重新提交；只需退出。

---

## 4. 董事会端体验

审批出现在 **Approvals → Pending** 中，带有黄色徽章。

![带有待处理 request_board_approval 行的审批队列](../../user-guides/screenshots/light/approvals/approvals-list.png)

打开它。详细视图在顶部显示三个结构化字段（标题、摘要、推荐操作），风险以项目符号列表形式显示，一个 **See full request** 展开器用于智能体包含的任何额外负载字段，以及一个 **Comments** 线程用于不需要完整修订请求的来回沟通。

决策按钮位于底部——每个审批都有相同的三个按钮：

![批准、拒绝和请求修订按钮](../../user-guides/screenshots/light/approvals/approve-reject-revision-buttons.png)

- **Approve** — 智能体以 `PAPERCLIP_APPROVAL_STATUS=approved` 唤醒并继续支出。
- **Reject** — 智能体以 `PAPERCLIP_APPROVAL_STATUS=rejected` 唤醒并根据其指令放弃支出。
- **Request revision** — 审批进入 `revision_requested` 状态，智能体*不会*自动唤醒（参见第 5 节末尾的注意事项）。该按钮不接受备注，因此请在 **Comments** 框中解释你想要什么（或通过下方的 API 发送 `decisionNote`）。

相同的操作也可在 API 上执行：

```bash
# Approve
curl -X POST "$PAPERCLIP_API_URL/api/approvals/$APPROVAL_ID/approve" \
  -H "Authorization: Bearer $BOARD_TOKEN"

# Reject with a reason — the reason lands on the approval and is visible to the agent on wake
curl -X POST "$PAPERCLIP_API_URL/api/approvals/$APPROVAL_ID/reject" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"decisionNote": "Wait until we have customer commitment before paid hosting."}'

# Send back for changes
curl -X POST "$PAPERCLIP_API_URL/api/approvals/$APPROVAL_ID/request-revision" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"decisionNote": "Same intent, but try the Hobby tier first ($0). Resubmit with that as the action."}'
```

有关完整的 UI 演练，包括队列过滤器和评论线程，参见 [Approvals (board guide)](../guides/day-to-day/approvals.md)。

---

## 5. 解决时的智能体唤醒

当你批准或拒绝时，Paperclip 使用两个额外的环境变量唤醒请求智能体：

| 变量 | 值 |
|---|---|
| `PAPERCLIP_APPROVAL_ID` | 已解决审批的 UUID。 |
| `PAPERCLIP_APPROVAL_STATUS` | `approved` 或 `rejected`。 |

智能体的心跳首先查看这些变量，然后再检查收件箱：

```python
import os, requests

API   = os.environ["PAPERCLIP_API_URL"]
TOKEN = os.environ["PAPERCLIP_API_KEY"]
RUN_ID = os.environ.get("PAPERCLIP_RUN_ID", "")
H = {"Authorization": f"Bearer {TOKEN}", "Content-Type": "application/json"}
RH = {**H, "X-Paperclip-Run-Id": RUN_ID}

def heartbeat():
    approval_id = os.environ.get("PAPERCLIP_APPROVAL_ID")
    status      = os.environ.get("PAPERCLIP_APPROVAL_STATUS")

    if approval_id:
        return handle_approval(approval_id, status)

    # ... normal inbox-driven heartbeat ...

def handle_approval(approval_id, status):
    # 1. Re-fetch the approval (env vars are convenience; payload + decisionNote live on the record)
    approval = requests.get(f"{API}/api/approvals/{approval_id}", headers=H).json()

    # 2. Find the issue this approval was gating
    issues = requests.get(f"{API}/api/approvals/{approval_id}/issues", headers=H).json()
    if not issues:
        return  # approval wasn't linked to an issue; nothing to resume
    issue_id = issues[0]["id"]

    if status == "approved":
        # 3a. Do the gated spend, then move the issue forward
        provision_vercel_pro(approval["payload"])
        requests.patch(
            f"{API}/api/issues/{issue_id}",
            headers=RH,
            json={
                "status": "in_progress",
                "comment": f"Approval [{approval_id}](/PAP/approvals/{approval_id}) approved — provisioned Vercel Pro and resumed.",
            },
        )

    elif status == "rejected":
        # 3b. Abandon the spend; comment with the board's reason
        note = approval.get("decisionNote") or "No reason provided."
        requests.patch(
            f"{API}/api/issues/{issue_id}",
            headers=RH,
            json={
                "status": "in_progress",
                "comment": f"Approval [{approval_id}](/PAP/approvals/{approval_id}) rejected: {note}\n\nAbandoning the spend. Will not retry without new direction.",
            },
        )
```

关于唤醒的两个重要细节：

- **唤醒仅覆盖 `approved` 和 `rejected`。** `request-revision` 不会触发 `PAPERCLIP_APPROVAL_ID`。董事会的 `decisionNote` 存储在审批上；智能体在其下一次正常心跳时得知此事（或者如果你也在源问题上留下评论，则会更快触发唤醒——参见 [Debug a stuck heartbeat](./debug-stuck-heartbeat.md) 了解完整的唤醒原因集合）。
- **重新获取审批和链接的问题。** 环境变量仅携带 id 和裁决。负载、`decisionNote` 和链接问题列表都存在于 API 上，自智能体提交请求以来可能已发生变化——特别是在修订轮次之后。

对于完全无法获得 Paperclip 驱动唤醒的 BYO-智能体运行时（[Bring your own agent → Option C](./bring-your-own-agent.md#option-c-custom-script-no-paperclip-adapter) 中的轮询模式），直接轮询审批：`GET /api/approvals/$APPROVAL_ID` 返回当前状态。使用退避循环，并在 `status != "pending"` 时退出。

---

## 6. 验证往返流程

在生产环境中依赖该审批门之前，证明循环是闭合的。使用一次性智能体和低价值审批。

1. **确保智能体的指令命名了审批门。** 打开智能体 → **Instructions**，粘贴第 2 节中规则的最小版本，保存。

2. **触发支出。** 分配给智能体一个应触发审批门的问题（“注册演示 SaaS 账户”）。运行一次心跳——要么让它在预定时间触发，要么点击智能体标题中的 **Run now**。

3. **确认审批已到达。** 打开 **Approvals → Pending**。你应该看到请求，其中包含你的智能体提交的提案负载。确认 `issueIds` 链接到源问题。

4. **从 API 批准。** 使用第 4 节中的 `/approve` 路由针对步骤 3 中的审批 id。智能体应在几秒内被唤醒。

5. **确认智能体已恢复。** 重新打开源问题。状态应回到 `in_progress`（或你的唤醒处理器设置的任何状态），并且应存在一条新评论，引用审批 id 并说明发生了什么。使用 `/reject` 重新运行以确认拒绝分支的行为方式相同。

如果智能体在步骤 4 后未唤醒，最常见的原因是：(a) 审批上未设置 `requestedByAgentId`，且没有智能体最终链接到它（API 为智能体身份验证的调用者自动填充此字段，但董事会身份验证的调用者必须显式传递）；(b) 智能体的适配器不支持带外唤醒（例如 BYO 中的 Option C——切换到轮询）；(c) 心跳处理器在收件箱检查*之后*读取环境变量，并因收件箱为空而静默退出。

---

## 7. 模式与注意事项

**与预算限额结合使用，而非替换它们。** 审批门由智能体自己的指令强制执行。从其他公司导入的新智能体，或忽略其指令的模型，可以绕过审批门。来自 [HT1](./set-monthly-budget.md) 的硬性限额会捕捉绕过行为并暂停范围。两者并用。

**每次支出一个审批，而非每次运行一个。** 如果智能体在前一个审批解决之前重新运行，第 3 节中的幂等性检查可防止重复。没有它，董事会会看到相同的提案两次，智能体必须区分哪个适用。

**使用评论线程进行澄清，而非静默拒绝。** 如果提案*几乎正确*，请在审批上发布评论（`POST /api/approvals/{approvalId}/comments`）而不是拒绝。智能体在其下一次心跳时拾取它并可以调整。将 `request-revision` 保留给智能体需要以结构不同的负载重新提交的情况。

**审计轨迹是自动的。** 每个审批、决策和决策备注都永久存储在审批记录上。`GET /api/approvals/{id}` 是规范来源——在回顾中很有用（“我们为什么注册了 X？”）以及在合规证据中（“这是董事会批准，这是时间戳，这是请求智能体”）。

**通知。** 审批队列本身不会推送到 Slack 或 Discord。如果你的董事会在聊天中，且队列否则会闲置，[配置通知例程](./wire-slack-discord-notifications.md)——与招聘审批通知器形式相同。

---

## 另请参阅

- [Set a monthly budget and enforce it](./set-monthly-budget.md) — 支出控制的被动响应部分。与此审批门并行运行。
- [Approvals (board guide)](../guides/day-to-day/approvals.md) — 队列和决策按钮的完整 UI 演练。
- [Approvals API](../../reference/api/approvals.md) — 此处触及的每个端点，加上生命周期图和重新提交语义。
- [Handle board approvals for hires](./handle-board-approvals-for-hires.md) — 针对 `hire_agent` 审批类型的相同治理流程。
- [Wire Slack/Discord notifications](./wire-slack-discord-notifications.md) — 将审批管道化到频道，以免队列闲置。
- [Debug a stuck heartbeat](./debug-stuck-heartbeat.md) — 当唤醒触发但智能体未拾取审批时的第一站。
