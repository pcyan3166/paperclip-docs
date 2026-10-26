---
seo_title: 活动记录 API
seo_description: Paperclip 的审计记录：发生了什么变化、由谁更改、涉及哪个对象以及何时发生。提供底层相同日志的两种读取方式，分别用于审计和调试。
---

# 活动记录

活动记录是 Paperclip 的审计日志。你可以用它回答：发生了什么变化、由谁更改、哪个对象发生了变化，以及变化何时发生？

底层日志可以通过两种方式读取。普通公司动态流（`/activity`）是按时间倒序排列的轻量列表，便于快速查看。更丰富的审计动态流（`/audit/agent-actions`）为 UI 的统一活动页面提供数据，增加了游标分页、范围切换、归因信息（涉及哪个智能体、哪次运行以及代表谁操作）和 CSV 导出。如果你需要查看单个任务或某次心跳运行的历史记录，请使用下方对应的端点。

---

## 端点

## 列出公司活动

```
GET /api/companies/{companyId}/activity
```

返回公司的活动动态流，最新记录排在最前。

查询参数：

| 参数 | 说明 |
|---|---|
| `agentId` | 按精确的操作方智能体 ID 过滤 |
| `entityType` | 按精确的实体类型过滤 |
| `entityId` | 按精确的实体 ID 过滤 |

注意事项：

- 过滤条件必须精确匹配，不支持模糊搜索。
- 此端点不提供分页。
- 公司动态流会过滤隐藏的任务，但仍会显示非任务活动。
- 此端点适合监控公司整体活动，不适合全文搜索。

> **提示：** 如果动态流内容太多，可以先用 `entityType=issue` 或 `entityId=<id>` 缩小范围。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl "http://localhost:3100/api/companies/company-1/activity?entityType=issue&agentId=agent-1" \
  -H "Authorization: Bearer <your-token>"
```

<!-- tab: JavaScript -->

```javascript
const url = new URL("/api/companies/company-1/activity", "http://localhost:3100");
url.searchParams.set("entityType", "issue");
url.searchParams.set("agentId", "agent-1");

const res = await fetch(url, {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});

const activity = await res.json();
```

<!-- tab: Python -->

```python
import requests

url = "http://localhost:3100/api/companies/company-1/activity"
params = {
    "entityType": "issue",
    "agentId": "agent-1",
}
headers = {
    "Authorization": f"Bearer {token}",
}

response = requests.get(url, params=params, headers=headers)
activity = response.json()
```

<!-- /tabs -->

## 审计动态流（智能体操作）

```
GET /api/companies/{companyId}/audit/agent-actions
```

返回支撑统一活动页面的丰富审计动态流，支持游标分页。记录按时间倒序排列，并包含普通动态流没有的归因信息：执行操作的智能体、心跳运行，以及负责的用户（如果记录本身未注明用户，则从运行记录中解析）。任务、评论和文档记录会补充任务标识符、标题和简短评论摘录。

查询参数：

| 参数 | 类型 | 说明 |
|---|---|---|
| `actorScope` | `agents` \| `all` | 要包含的操作方。`agents`（默认值）只返回关联了智能体的记录；`all` 返回所有操作方类型。 |
| `agentId` | 字符串（UUID） | 只筛选一个执行操作的智能体。 |
| `responsibleUserId` | 字符串 | 筛选代表某位用户执行的所有操作。 |
| `runId` | 字符串（UUID） | 筛选某次心跳运行。 |
| `entityType` | 字符串 | 精确实体类型，例如 `issue`。 |
| `entityId` | 字符串 | 精确实体 ID。 |
| `action` | 字符串 | 按操作名称前缀匹配，例如 `issue` 会匹配 `issue.updated`、`issue.comment_added` 等。 |
| `actorType` | `agent` \| `user` \| `system` \| `plugin` | 精确的操作方类型。 |
| `from` | 日期 | 纳入记录的最早 `createdAt` 时间。 |
| `to` | 日期 | 纳入记录的最晚 `createdAt` 时间。 |
| `cursor` | 字符串 | 上一次响应中 `nextCursor` 返回的不透明游标。 |
| `limit` | 整数 | 每页数量，范围为 1–200，默认值为 `50`。 |

每次响应包含 `{ items, nextCursor, accessTier }`。分页时将 `nextCursor` 作为 `cursor` 传回；最后一页的 `nextCursor` 为 `null`。

### 两级访问权限

此端点有两个访问级别，响应中的 `accessTier` 会标明当前级别：

- **`actorScope=all` 对所有公司成员开放。** 他们可以看到所有操作方类型（`agent`、`user`、`system`、`plugin`），但敏感归因字段 `agentId`、`runId`、`responsibleUserId` 和 `details` 会被清空为 `null`，响应包含 `accessTier: "basic"`。这些调用方使用归因过滤器（`agentId`、`responsibleUserId`、`runId`、`entityType`、`entityId`、`action`、`actorType`、`from`、`to`）会收到 `403`。
- **拥有 `audit:view_agent_actions` 权限的成员**可以读取完整记录、使用所有过滤器，且响应包含 `accessTier: "full"`。无论访问级别如何，默认范围 `actorScope=agents` 始终要求此权限。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl "http://localhost:3100/api/companies/company-1/audit/agent-actions?actorScope=all&limit=50" \
  -H "Authorization: Bearer <your-token>"
```

<!-- tab: JavaScript -->

```javascript
const url = new URL("/api/companies/company-1/audit/agent-actions", "http://localhost:3100");
url.searchParams.set("actorScope", "all");
url.searchParams.set("limit", "50");

const res = await fetch(url, {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});

const { items, nextCursor, accessTier } = await res.json();
```

<!-- tab: Python -->

```python
import requests

url = "http://localhost:3100/api/companies/company-1/audit/agent-actions"
params = {
    "actorScope": "all",
    "limit": 50,
}
headers = {
    "Authorization": f"Bearer {token}",
}

response = requests.get(url, params=params, headers=headers)
feed = response.json()
```

<!-- /tabs -->

## 将审计动态流导出为 CSV

```
GET /api/companies/{companyId}/audit/agent-actions.csv
```

以 CSV 附件形式流式传输审计动态流。该端点接受与上方动态流相同的过滤参数（忽略 `cursor` 和 `limit`，导出流程会自行分页）。单次导出最多包含 10,000 条记录；历史记录较长时，请缩小日期范围。

此端点始终要求 `audit:view_agent_actions` 权限，不提供基础访问级别。

导出本身也会留下审计记录：Paperclip 会记录 `audit.exported` 活动事件，其中包含导出者、使用的过滤条件以及离开系统的记录数量。

列的顺序为：`createdAt`、`action`、`actorType`、`actorId`、`agentId`、`runId`、`responsibleUserId`、`entityType`、`entityId`、`issueIdentifier`、`issueTitle`、`commentExcerpt`、`documentKey`。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl "http://localhost:3100/api/companies/company-1/audit/agent-actions.csv?from=2026-01-01" \
  -H "Authorization: Bearer <your-token>" \
  -o agent-audit.csv
```

<!-- tab: JavaScript -->

```javascript
const url = new URL("/api/companies/company-1/audit/agent-actions.csv", "http://localhost:3100");
url.searchParams.set("from", "2026-01-01");

const res = await fetch(url, {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});

const csv = await res.text();
```

<!-- tab: Python -->

```python
import requests

response = requests.get(
    "http://localhost:3100/api/companies/company-1/audit/agent-actions.csv",
    params={"from": "2026-01-01"},
    headers={"Authorization": f"Bearer {token}"},
)

with open("agent-audit.csv", "wb") as f:
    f.write(response.content)
```

<!-- /tabs -->

## 创建活动事件

```
POST /api/companies/{companyId}/activity
```

创建新的活动日志条目。此端点仅限看板用户使用。

Paperclip 的大多数路由都会自动写入活动记录，因此通常无需自行调用此端点，除非你正在构建自定义管理集成或记录系统事件。

请求正文：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---:|---|
| `actorType` | `agent` \| `user` \| `system` \| `plugin` | 否 | 默认为 `system` |
| `actorId` | 字符串 | 是 | 自由格式的操作方标签或 ID |
| `action` | 字符串 | 是 | 事件名称，例如 `issue.updated` |
| `entityType` | 字符串 | 是 | 发生变化的对象类型 |
| `entityId` | 字符串 | 是 | 受影响实体的 ID |
| `agentId` | 字符串 \| `null` | 否 | 可选的智能体 UUID |
| `details` | 对象 \| `null` | 否 | 附加 JSON 负载；存储前会进行清理 |

实用说明：

- `actorId` 以文本形式存储。它可以是用户 ID、智能体 ID 或系统标签。
- `details` 以 JSON 形式存储，并可能根据实例日志设置进行脱敏。
- 此端点用于看板侧或系统侧事件，不适用于公开客户端 API。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/companies/company-1/activity" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "actorType": "system",
    "actorId": "nightly-sync",
    "action": "company.report_generated",
    "entityType": "company",
    "entityId": "company-1",
    "details": {
      "source": "scheduled-job",
      "report": "weekly-summary"
    }
  }'
```

<!-- tab: JavaScript -->

```javascript
const res = await fetch("/api/companies/company-1/activity", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${boardToken}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    actorType: "system",
    actorId: "nightly-sync",
    action: "company.report_generated",
    entityType: "company",
    entityId: "company-1",
    details: {
      source: "scheduled-job",
      report: "weekly-summary",
    },
  }),
});

const created = await res.json();
```

<!-- tab: Python -->

```python
import requests

response = requests.post(
    "http://localhost:3100/api/companies/company-1/activity",
    headers={
        "Authorization": f"Bearer {board_token}",
        "Content-Type": "application/json",
    },
    json={
        "actorType": "system",
        "actorId": "nightly-sync",
        "action": "company.report_generated",
        "entityType": "company",
        "entityId": "company-1",
        "details": {
            "source": "scheduled-job",
            "report": "weekly-summary",
        },
    },
)

created = response.json()
```

<!-- /tabs -->

## 任务活动

```
GET /api/issues/{issueId}/activity
```

返回单个任务的活动历史，最新记录排在最前。

你可以传入以下任意一种标识：

- 原始任务 UUID
- UI 中显示的人类可读标识符，例如 `PAP-475`

路由会先解析标识符，再加载活动记录。因此，在调查某项任务并需要按顺序查看完整历史时，这是最合适的端点。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl "http://localhost:3100/api/issues/PAP-475/activity" \
  -H "Authorization: Bearer <your-token>"
```

<!-- tab: JavaScript -->

```javascript
const res = await fetch("/api/issues/PAP-475/activity", {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});

const issueActivity = await res.json();
```

<!-- tab: Python -->

```python
import requests

response = requests.get(
    "http://localhost:3100/api/issues/PAP-475/activity",
    headers={
        "Authorization": f"Bearer {token}",
    },
)

issue_activity = response.json()
```

<!-- /tabs -->

## 任务关联的运行

```
GET /api/issues/{issueId}/runs
```

返回处理过某个任务的心跳运行记录。

此端点适用于以下场景：

- 活动记录回答“发生了什么”。
- 运行记录回答“涉及哪些心跳运行”。

服务器会使用运行上下文快照和任务活动日志，将运行记录与任务关联起来。因此，即使活动轨迹不完整，此端点仍可能找到相关运行。

响应包含以下运行元数据：

- `runId`
- `status`
- `agentId`
- `adapterType`
- `startedAt`
- `finishedAt`
- `createdAt`
- `invocationSource`
- `usageJson`
- `resultJson`
- `logBytes`

## 运行关联的任务

```
GET /api/heartbeat-runs/{runId}/issues
```

返回与某次心跳运行关联的任务。

注意事项：

- 如果运行不存在，端点会返回空数组。
- 返回任何内容之前，路由都会检查公司访问权限。
- 响应是精简的任务摘要，而不是完整任务记录。

## 活动记录字段

每条活动记录都包含以下字段：

| 字段 | 含义 |
|---|---|
| `companyId` | 此事件所属的公司 |
| `actorType` | `agent`、`user`、`system` 或 `plugin` |
| `actorId` | 操作方的文本标签或 ID |
| `action` | 事件名称 |
| `entityType` | 发生变更的对象类型 |
| `entityId` | 发生变更的实体 ID |
| `agentId` | 可选的操作方智能体 UUID |
| `runId` | 可选的心跳运行 UUID |
| `details` | 包含额外上下文的可选 JSON 负载 |
| `createdAt` | 记录事件的时间 |

> **注意：**公司动态流只追加记录。底层变更发生时会写入事件，之后不会原地编辑这些事件。
