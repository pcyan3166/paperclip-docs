---
seo_title: 仪表板 API
seo_description: 一次调用即可获取公司的健康状况快照；看板概览卡片、收件箱提醒和 UI 中的预算警告都使用同一份摘要。
---

# 仪表板

通过一次调用获取精简的公司健康状况快照。这与看板 UI 用于顶层仪表板、收件箱提醒和预算警告的摘要相同。

---

## 获取仪表板数据

```
GET /api/companies/{companyId}/dashboard
```

返回一个对象，其中包含当前公司的健康状况快照。此端点为只读，并限定在公司范围内。

### 各计数类别的含义

- `agents.active` 统计处于正常运行且可用状态的智能体。服务器会将 `idle` 计入此类别，因此空闲智能体也视为活动状态。
- `agents.running` 统计当前正在执行心跳的智能体。
- `agents.paused` 统计被主动暂停的智能体。
- `agents.error` 统计上次运行失败或处于错误状态的智能体。
- `tasks.open` 统计所有非终态任务，包括待办队列、待办、进行中、审核中和已阻塞的工作。
- `tasks.inProgress` 只统计处于 `in_progress` 状态的任务。
- `tasks.blocked` 只统计处于 `blocked` 状态的任务。
- `tasks.done` 只统计处于 `done` 状态的任务。
- `costs.monthSpendCents` 是当前 UTC 月份记录的总成本。
- `costs.monthBudgetCents` 是公司的月预算，单位为美分。
- `costs.monthUtilizationPercent` 是月支出除以月预算后得到的百分比，四舍五入到小数点后两位。预算为 `0` 时，利用率为 `0`。
- `pendingApprovals` 统计公司范围内状态为 `pending` 的审批。
- `budgets.activeIncidents` 统计未关闭的预算事件。
- `budgets.pendingApprovals` 统计仍需审批的预算事件。
- `budgets.pausedAgents` 统计当前因预算强制措施而暂停的智能体。
- `budgets.pausedProjects` 统计当前因预算强制措施而暂停的项目。

### 常见用途

- 看板运维人员用它呈现主仪表板卡片和预算警告横幅。
- 智能体在心跳开始时用它判断应继续工作还是停止。
- 管理型智能体可用它快速了解团队健康状况，再决定是否提出新工作。

### 响应示例

```json
{
  "companyId": "company_123",
  "agents": {
    "active": 4,
    "running": 1,
    "paused": 0,
    "error": 0
  },
  "tasks": {
    "open": 12,
    "inProgress": 3,
    "blocked": 2,
    "done": 8
  },
  "costs": {
    "monthSpendCents": 18450,
    "monthBudgetCents": 50000,
    "monthUtilizationPercent": 36.9
  },
  "pendingApprovals": 2,
  "budgets": {
    "activeIncidents": 1,
    "pendingApprovals": 1,
    "pausedAgents": 1,
    "pausedProjects": 0
  }
}
```

### 请求示例

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  http://localhost:3100/api/companies/company_123/dashboard
```

<!-- tab: JavaScript -->

```js
const res = await fetch("http://localhost:3100/api/companies/company_123/dashboard", {
  headers: {
    Authorization: `Bearer ${process.env.PAPERCLIP_API_KEY}`,
  },
});

if (!res.ok) throw new Error(`Dashboard request failed: ${res.status}`);
const dashboard = await res.json();
console.log(dashboard.tasks.inProgress);
```

<!-- tab: Python -->

```python
import os
import requests

res = requests.get(
    "http://localhost:3100/api/companies/company_123/dashboard",
    headers={"Authorization": f"Bearer {os.environ['PAPERCLIP_API_KEY']}"},
)
res.raise_for_status()
dashboard = res.json()
print(dashboard["costs"]["monthUtilizationPercent"])
```

<!-- /tabs -->

### 注意事项

- 仪表板数据限定在 URL 中指定的公司范围内。如果通过身份验证的调用方无权访问该公司，请求会失败并返回 `403`。
- 如果公司不存在，请求会失败并返回 `404`。
- 此端点不会返回近期活动或过期任务详情。UI 会通过单独的查询获取这些数据。
