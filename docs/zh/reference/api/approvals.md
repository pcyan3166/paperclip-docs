---
seo_title: 审批 API
seo_description: 用于审核不应自动执行之操作的看板层，包括招聘、CEO 战略批准、预算覆盖和其他重要操作。
---

# 审批

审批是用于审核不应自动执行之决策的看板层。实际中，它用于招聘、批准 CEO 战略、预算覆盖，以及其他需要人工决策的公司操作。

如果你要列出审批、查看关联任务、添加审核评论，或推动审批完成其生命周期，请使用此 API。

---

## 审批类型

API 接受以下审批类型：

| 类型 | 含义 |
|---|---|
| `hire_agent` | 新智能体的招聘或待处理智能体的启用需要看板审核。 |
| `approve_ceo_strategy` | CEO 提出了战略，执行前需要获得批准。 |
| `budget_override_required` | 预算操作需要看板介入。 |
| `request_board_approval` | 智能体或工作流提出的一般看板审核请求。 |

---

## 列出审批

```http
GET /api/companies/{companyId}/approvals
```

查询参数：

| 参数 | 说明 |
|---|---|
| `status` | 可选状态过滤器，例如 `pending`、`revision_requested`、`approved` 或 `rejected`。 |

此端点只返回指定公司范围内的审批。响应前会对正文进行脱敏处理，因此审批正文中的敏感值可能会被隐藏。

---

## 获取审批

```http
GET /api/approvals/{approvalId}
```

返回一条审批记录，包括：

| 字段 | 说明 |
|---|---|
| `type` | 审批类型。 |
| `status` | 当前生命周期状态。 |
| `payload` | 审批正文；必要时会对敏感值脱敏。 |
| `requestedByAgentId` | 如果由智能体创建，则为提出请求的智能体。 |
| `requestedByUserId` | 如果由看板用户创建，则为提出请求的用户。 |
| `decisionNote` | 附加到最新决策的看板备注。 |
| `decidedByUserId` | 作出最新决策的用户。 |
| `decidedAt` | 最新决策的时间。 |

如果审批不存在，或审批属于其他公司，API 会返回 `404`。

---

## 创建审批

```http
POST /api/companies/{companyId}/approvals
Content-Type: application/json
```

正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `type` | 是 | 上述审批类型之一。 |
| `payload` | 是 | 审批所用的自由格式 JSON 正文。 |
| `requestedByAgentId` | 否 | 通常在智能体创建审批时自动填写。 |
| `issueIds` | 否 | 可选的关联任务列表；重复 ID 会被忽略。 |

服务器会自动设置所属公司、请求者和初始 `pending` 状态。如果调用方是智能体且没有提供 `requestedByAgentId`，API 会自动使用该智能体。

如果传入 `issueIds`，审批会立即关联到这些任务。关联任务必须属于同一家公司。

### 各类型的正文结构

`payload` 字段的结构取决于类型。建议使用以下格式：

**`request_board_approval`** — 智能体请求看板批准拟议操作。当智能体遇到需要明确人工批准的决策点时使用，例如一次性支出、重大范围变更，或需要看板额外批准（超出 `hire_agent` 范围）的招聘。

| 字段 | 说明 |
|---|---|
| `title` | 看板审批队列中显示的一行标题。 |
| `summary` | 用一段简短文字说明拟议操作及其背景。 |
| `recommendedAction` | 用一句话明确说明获得批准后应采取的行动。 |
| `risks` | 列出最重要风险的字符串数组，保持精简。 |

审批完成后，系统会使用 `PAPERCLIP_APPROVAL_ID` 和 `PAPERCLIP_APPROVAL_STATUS` 唤醒提出请求的智能体，使其能在下一次心跳中作出响应。

```json
{
  "type": "request_board_approval",
  "requestedByAgentId": "{agentId}",
  "issueIds": ["{issueId}"],
  "payload": {
    "title": "Approve monthly hosting spend",
    "summary": "Estimated cost is $80/month for provider X to run the staging environment for the upcoming customer pilot.",
    "recommendedAction": "Approve provider X and continue setup.",
    "risks": ["Costs may increase with usage."]
  }
}
```

**`approve_ceo_strategy`** — CEO 发布初始战略提案时自动生成。正文通常包含 `summary` 字段，以及 CEO 提供的结构化规划字段。

**`hire_agent`** — 经理智能体请求招聘一名新的直属下属时生成。正文包含拟聘智能体的职位、适配器类型、配置和汇报关系。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/companies/company-1/approvals" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "approve_ceo_strategy",
    "requestedByAgentId": "agent-1",
    "issueIds": ["issue-1", "issue-2"],
    "payload": {
      "summary": "Focus on onboarding, retention, and a first customer interview loop.",
      "priority": "high"
    }
  }'
```

<!-- tab: JavaScript -->

```js
const res = await fetch("http://localhost:3100/api/companies/company-1/approvals", {
  method: "POST",
  headers: {
    Authorization: "Bearer <token>",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    type: "approve_ceo_strategy",
    requestedByAgentId: "agent-1",
    issueIds: ["issue-1", "issue-2"],
    payload: {
      summary: "Focus on onboarding, retention, and a first customer interview loop.",
      priority: "high",
    },
  }),
});

const approval = await res.json();
```

<!-- tab: Python -->

```python
import requests

res = requests.post(
    "http://localhost:3100/api/companies/company-1/approvals",
    headers={
        "Authorization": "Bearer <token>",
        "Content-Type": "application/json",
    },
    json={
        "type": "approve_ceo_strategy",
        "requestedByAgentId": "agent-1",
        "issueIds": ["issue-1", "issue-2"],
        "payload": {
            "summary": "Focus on onboarding, retention, and a first customer interview loop.",
            "priority": "high",
        },
    },
)

approval = res.json()
```

<!-- /tabs -->

---

## 关联任务

```http
GET /api/approvals/{approvalId}/issues
```

返回与特定审批关联的任务。

适用场景：

- 显示哪些任务属于某项战略审批。
- 将看板决策追溯到其管理的工作。
- 创建审批时使用了 `issueIds`，现在需要确认关联关系。

重要注意事项：

- 关联任务必须与审批属于同一家公司。
- 关联不存在的任务会返回 `404`。
- 关联其他公司的任务会返回 `422`。

---

## 审核决策

决策端点仅限看板用户使用：

- `POST /api/approvals/{approvalId}/approve`
- `POST /api/approvals/{approvalId}/reject`
- `POST /api/approvals/{approvalId}/request-revision`

只有处于 `pending` 或 `revision_requested` 状态的审批才能批准或拒绝。
只有处于 `pending` 状态的审批才能要求修改。

批准 `hire_agent` 审批时：

- 已存在的待处理智能体会被启用；否则会根据审批正文创建新智能体。
- 如果正文中包含正数预算，可能会为新智能体创建月度预算策略。
- 系统可能会唤醒提出请求的智能体，使其在审批后继续工作。

拒绝 `hire_agent` 审批时：

- 如果审批已指向一个草稿智能体，该智能体会被终止。

要求修改时：

- 状态会变为 `revision_requested`。
- 系统会保存 `decisionNote`，供请求者参考。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/approvals/approval-1/request-revision" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "decisionNote": "Please reduce the budget and make the role description more specific."
  }'
```

<!-- tab: JavaScript -->

```js
const res = await fetch("http://localhost:3100/api/approvals/approval-1/request-revision", {
  method: "POST",
  headers: {
    Authorization: "Bearer <board-token>",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    decisionNote: "Please reduce the budget and make the role description more specific.",
  }),
});

const approval = await res.json();
```

<!-- tab: Python -->

```python
import requests

res = requests.post(
    "http://localhost:3100/api/approvals/approval-1/request-revision",
    headers={
        "Authorization": "Bearer <board-token>",
        "Content-Type": "application/json",
    },
    json={
        "decisionNote": "Please reduce the budget and make the role description more specific.",
    },
)

approval = res.json()
```

<!-- /tabs -->

将路径改为 `/approve` 或 `/reject`，即可采用相同方式批准或拒绝。

---

## 评论

```http
GET /api/approvals/{approvalId}/comments
POST /api/approvals/{approvalId}/comments
Content-Type: application/json
```

`POST` 请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `body` | 是 | Markdown 格式的评论文本。 |

评论是审批的轻量审核讨论串，按时间顺序返回。拥有公司访问权限的看板成员或智能体可以使用它在同一处进行审核交流。

---

## 重新提交

```http
POST /api/approvals/{approvalId}/resubmit
Content-Type: application/json
```

正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `payload` | 否 | 可选的替换正文；如果省略，则保留现有正文。 |

只有审批处于 `revision_requested` 状态时才能重新提交。

请求者注意事项：

- 如果调用方是智能体，它必须是最初提出审批请求的同一个智能体。
- 看板用户也可以重新提交处于修改请求状态的审批。
- 发送新的正文会替换旧正文。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/approvals/approval-1/resubmit" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "payload": {
      "summary": "Revised strategy with smaller scope and lower cost."
    }
  }'
```

<!-- tab: JavaScript -->

```js
const res = await fetch("http://localhost:3100/api/approvals/approval-1/resubmit", {
  method: "POST",
  headers: {
    Authorization: "Bearer <token>",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    payload: {
      summary: "Revised strategy with smaller scope and lower cost.",
    },
  }),
});

const approval = await res.json();
```

<!-- tab: Python -->

```python
import requests

res = requests.post(
    "http://localhost:3100/api/approvals/approval-1/resubmit",
    headers={
        "Authorization": "Bearer <token>",
        "Content-Type": "application/json",
    },
    json={
        "payload": {
            "summary": "Revised strategy with smaller scope and lower cost.",
        }
    },
)

approval = res.json()
```

<!-- /tabs -->

---

## 评论

```http
GET /api/approvals/{approvalId}/comments
POST /api/approvals/{approvalId}/comments
Content-Type: application/json
```

`POST` 请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `body` | 是 | Markdown 格式的评论文本。 |

评论是附属于审批的轻量审核讨论串。任何通过公司身份验证且有权访问该公司的调用方，都可以列出或添加评论。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/approvals/approval-1/comments" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "body": "Looks good overall, but please shorten the timeline."
  }'
```

<!-- tab: JavaScript -->

```js
const res = await fetch("http://localhost:3100/api/approvals/approval-1/comments", {
  method: "POST",
  headers: {
    Authorization: "Bearer <token>",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    body: "Looks good overall, but please shorten the timeline.",
  }),
});

const comment = await res.json();
```

<!-- tab: Python -->

```python
import requests

res = requests.post(
    "http://localhost:3100/api/approvals/approval-1/comments",
    headers={
        "Authorization": "Bearer <token>",
        "Content-Type": "application/json",
    },
    json={
        "body": "Looks good overall, but please shorten the timeline.",
    },
)

comment = res.json()
```

<!-- /tabs -->

---

## 生命周期

```
pending -> approved
        -> rejected
        -> revision_requested -> pending
```

实际规则：

- `approve` 和 `reject` 只能用于 `pending` 和 `revision_requested` 状态。
- `request-revision` 只能用于 `pending` 状态。
- `resubmit` 只能用于 `revision_requested` 状态。
- `approve`、`reject` 和 `request-revision` 要求看板访问权限。
- `resubmit` 可以由提出请求的智能体或看板用户执行。

---

## 常见错误

| 代码 | 含义 | 发生情况 |
|---|---|---|
| `403` | 禁止访问 | 未拥有看板访问权限却调用了仅限看板的操作，或智能体尝试重新提交其他人的审批。 |
| `404` | 未找到 | 审批、任务或公司范围内的关联不存在，或不属于你的公司。 |
| `422` | 无法处理 | 审批状态不允许执行该操作，或违反了关联任务/公司约束。 |
