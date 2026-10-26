---
paperclip_version: v2026.1005.0
seo_title: 例行任务 API
seo_description: Paperclip 的周期性执行层。可按计划运行代理、通过 webhook 触发，或通过 API 手动启动例行任务，并了解它的行为。
---

# 例行任务

例行任务是 Paperclip 的周期性执行层。需要代理按计划运行、响应 webhook，或通过 API 手动启动时，可以使用例行任务。

例行任务本身不执行工作，而是创建一次运行；该运行通常会为受派代理创建一个执行 issue，或关联到已有 issue。

## 例行任务控制的内容

例行任务会关联：

- 负责工作的代理。
- 项目、目标以及可选的父 issue 上下文。
- 执行 issue 的标题与描述模板。
- 启动运行的一个或多个触发器。
- 已有运行仍在执行时采用的并发策略。
- 错过计划触发时间后的补跑策略。
- 决定安静周期中是否仍应触发计划任务的活动门控。

状态如下：

| 状态 | 含义 |
|---|---|
| `active` | 例行任务可以触发并创建运行。 |
| `paused` | 例行任务仍被保存，但不会自动触发。 |
| `archived` | 例行任务已退役，不会触发。 |

`active` 状态必须有受派代理。若创建或启用时没有受派代理，API 会将其规范化为 `paused`，或拒绝激活。

## 列出例行任务

```http
GET /api/companies/{companyId}/routines
```

按最近更新时间倒序返回公司的全部例行任务。

每项包含：

- 例行任务字段。
- 触发器摘要。
- 最近一次运行。
- 当前活动执行 issue（如有）。



### 示例

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl "http://localhost:3100/api/companies/company-1/routines" \
  -H "Authorization: Bearer <token>"
```


<!-- tab: JavaScript -->

```javascript
const res = await fetch("/api/companies/company-1/routines", {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});

const routines = await res.json();
```


<!-- tab: Python -->

```python
import requests

response = requests.get(
    "http://localhost:3100/api/companies/company-1/routines",
    headers={
        "Authorization": f"Bearer {token}",
    },
)

routines = response.json()
```


<!-- /tabs -->

## 获取例行任务

```http
GET /api/routines/{routineId}
```

返回单个例行任务及以下信息：

- 项目详情。
- 受派代理详情。
- 父 issue 详情。
- 触发器。
- 最近运行。
- 当前活动执行 issue。

此端点限定在当前公司范围内；例行任务不属于当前公司时，请求会被拒绝。

## 创建例行任务

```http
POST /api/companies/{companyId}/routines
Content-Type: application/json
```

请求字段：

| 字段 | 必填 | 说明 |
|---|---|---|
| `title` | 是 | 例行任务名称，文本中可使用例行任务变量。 |
| `description` | 否 | 面向用户的说明，也可使用变量。 |
| `assigneeAgentId` | 是 | 接收每次运行的代理。 |
| `projectId` | 否 | 与例行任务关联的项目。 |
| `goalId` | 否 | 与例行任务创建的工作关联的目标。 |
| `parentIssueId` | 否 | 例行任务创建的执行 issue 的父 issue。 |
| `priority` | 否 | `critical`、`high`、默认值 `medium` 或 `low`。 |
| `status` | 否 | 有受派代理时默认 `active`，否则默认 `paused`。 |
| `concurrencyPolicy` | 否 | 默认 `coalesce_if_active`。 |
| `catchUpPolicy` | 否 | 默认 `skip_missed`。 |
| `activityGatePolicy` | 否 | `always`（默认）或 `require_external_activity`，见[活动门控](#活动门控)。 |
| `activityGateScope` | 否 | `company`（默认）或 `project`；仅在策略为 `require_external_activity` 时使用。 |
| `variables` | 否 | 供标题、描述和运行负载模板使用的变量。 |

代理只能创建指派给自己的例行任务。看板用户可以为自己有权指派的代理创建任务。

重要行为：

- `projectId`、`goalId` 和 `parentIssueId` 必须属于同一公司。
- 受派代理必须属于同一公司，且不能处于已终止或待审批状态。
- 没有受派代理时，API 不会让例行任务保持 `active`。
- 标题或描述中引用的变量必须有效且可解析。



### 示例

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/companies/company-1/routines" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Weekly CEO briefing for {{company_name}}",
    "description": "Compile a weekly summary and prepare the next priorities.",
    "assigneeAgentId": "agent-1",
    "projectId": "project-1",
    "goalId": "goal-1",
    "priority": "medium",
    "status": "active",
    "concurrencyPolicy": "coalesce_if_active",
    "catchUpPolicy": "skip_missed",
    "variables": [
      {
        "name": "company_name",
        "label": "Company name",
        "type": "text",
        "required": true,
        "defaultValue": "Paperclip"
      }
    ]
  }'
```


<!-- tab: JavaScript -->

```javascript
const res = await fetch("/api/companies/company-1/routines", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${boardToken}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    title: "Weekly CEO briefing for {{company_name}}",
    description: "Compile a weekly summary and prepare the next priorities.",
    assigneeAgentId: "agent-1",
    projectId: "project-1",
    goalId: "goal-1",
    priority: "medium",
    status: "active",
    concurrencyPolicy: "coalesce_if_active",
    catchUpPolicy: "skip_missed",
    variables: [
      {
        name: "company_name",
        label: "Company name",
        type: "text",
        required: true,
        defaultValue: "Paperclip",
      },
    ],
  }),
});

const routine = await res.json();
```


<!-- tab: Python -->

```python
import requests

response = requests.post(
    "http://localhost:3100/api/companies/company-1/routines",
    headers={
        "Authorization": f"Bearer {board_token}",
        "Content-Type": "application/json",
    },
    json={
        "title": "Weekly CEO briefing for {{company_name}}",
        "description": "Compile a weekly summary and prepare the next priorities.",
        "assigneeAgentId": "agent-1",
        "projectId": "project-1",
        "goalId": "goal-1",
        "priority": "medium",
        "status": "active",
        "concurrencyPolicy": "coalesce_if_active",
        "catchUpPolicy": "skip_missed",
        "variables": [
            {
                "name": "company_name",
                "label": "Company name",
                "type": "text",
                "required": True,
                "defaultValue": "Paperclip",
            }
        ],
    },
)

routine = response.json()
```



<!-- /tabs -->

## 更新例行任务

```http
PATCH /api/routines/{routineId}
Content-Type: application/json
```

创建时的所有字段都可以更新。

代理的权限范围比看板窄：

- 代理只能更新指派给自己的例行任务。
- 代理不能将例行任务改派给其他代理。
- 代理尝试为其他代理启用例行任务时，API 会拒绝。

此端点可用于：

- 暂停或恢复例行任务。
- 更改计划上下文。
- 将任务移至其他项目或目标。
- 调整变量或优先级。
- 启用或关闭活动门控。

## 活动门控

默认情况下，每个计划触发时刻都会启动例行任务，无论公司自上次运行以来是否发生过变化。活动门控可设为“只有出现值得响应的新活动才运行”，适合摘要、巡检和分流等在安静周期无事可做的任务。

| 字段 | 可选值 | 默认值 |
|---|---|---|
| `activityGatePolicy` | `always`、`require_external_activity` | `always` |
| `activityGateScope` | `company`、`project` | `company` |

两个字段控制活动门控。设为 `require_external_activity` 后，每次计划触发都会查找至少一条符合以下条件的活动日志：

- 记录时间晚于例行任务上次已派发的运行，且不晚于当前触发时刻。最终被标记为 `skipped` 或 `coalesced` 的运行不会移动这个起点。
- 不是已读或收件箱记账操作：`issue.read_marked`、`issue.read_unmarked`、`issue.inbox_archived`、`issue.inbox_unarchived`。
- 不是调度器针对该例行任务自身写入的记录。
- 不是代理运行处理该例行任务创建的执行 issue 所产生的活动。

只要存在一条匹配记录，触发就会正常派发；否则会跳过。后两条规则保证“外部”确实表示外部活动：例行任务自身的运行和跳过记录不会让门控一直保持开启。首次触发始终会运行，因为此前没有已派发运行可作为测量窗口起点。

### 范围

- `company`（默认值）：公司内任意位置的活动都计入。
- `project`：只统计与例行任务所属项目有关的活动，包括项目记录、该项目中的 issue、属于该项目的例行任务及运行、处理该项目 issue 的代理运行，以及带有该项目 ID 的活动记录。

若 `activityGateScope: "project"` 却未设置 `projectId`，就永远无法匹配任何活动，因此计划任务不会触发。请先设置项目。

### 门控跳过的触发会怎样

这不会导致失败，也不会把任务排队等待稍后执行：

- 系统会写入运行记录，其中 `status: "skipped"`、`failureReason: "no_external_activity"`，且没有 `linkedIssueId`。
- 运行的 `triggerPayload` 会包含 `activityGate.verdict`，其值为 `"quiet"`，以及门控测量起点时间戳 `activityGate.windowStart`。
- 触发器的最近结果为 `skipped_no_activity`，并且 `nextRunAt` 已推进到下一次触发。

活动恢复后，不会补发被跳过的触发；下一次计划触发会单独评估。

### 门控不会影响什么

活动门控仅用于计划触发。无论门控策略如何，Webhook 触发和通过 `POST /api/routines/{routineId}/run` 发起的手动运行都会照常派发。

## 触发器类型

例行任务支持三类触发器。

### 计划触发器

```http
POST /api/routines/{routineId}/triggers
Content-Type: application/json
```

```json
{
  "kind": "schedule",
  "label": "Monday morning",
  "enabled": true,
  "cronExpression": "0 9 * * 1",
  "timezone": "Europe/Amsterdam"
}
```

计划触发器使用 cron 表达式和时区。服务器会验证表达式，并按该时区计算下次运行时间。例行任务的必填变量必须能够解析；若有必填变量没有默认值，则无法启用计划触发器。

<a id="webhook"></a>
### Webhook 触发器

```json
{
  "kind": "webhook",
  "label": "Stripe event bridge",
  "enabled": true,
  "signingMode": "bearer",
  "replayWindowSec": 300
}
```

Webhook 触发器会创建公开 URL 和密钥。创建触发器或轮换密钥时，公开 URL 会在响应中返回一次。

| 签名模式 | 接受的认证方式 |
|---|---|
| `app_webhook` | Bearer 令牌，或针对请求原始内容计算的 HMAC-SHA256 签名；界面中的“另一个应用或脚本”配置会创建此模式。 |
| `bearer` | 仅接受 Bearer 令牌。 |
| `hmac_sha256` | 带时间戳的 HMAC-SHA256 签名。 |
| `github_hmac` | GitHub 风格的请求体 HMAC-SHA256，不带时间戳。 |
| `fireflies_hmac` | Fireflies Webhooks V2 签名；保留给创建时曾提供该选项的旧触发器，新配置使用 `app_webhook`。 |
| `none` | 不做认证；任何拥有 URL 的人都能触发。 |

通过 API 创建 webhook 触发器而未提供 `signingMode` 时，默认值为 `bearer`。重放窗口仅用于带时间戳的 `hmac_sha256`，范围为 30–86,400 秒，默认 300 秒。`webhookUrl` 根据服务器公开源地址生成。GitHub、Fireflies 等外部发送方只能向公开可访问的 HTTPS 地址投递，见 [HTTPS 与公开访问](../deploy/https.md)。若 URL 指向 `localhost`、私有网络、Tailscale `.ts.net` 主机或普通 HTTP，界面会发出提示。

### API 触发器

```json
{
  "kind": "api"
}
```

API 触发器不会创建公开 URL，仅用于通过例行任务运行端点显式启动任务。

## 添加触发器


Webhook 触发器的响应可能包含以下密钥材料：

- `webhookUrl`
- `webhookSecret`

请像密码一样保护密钥。它实际存储为公司密钥，只有创建触发器或轮换密钥时会显示给你。



### 示例

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/routines/routine-1/triggers" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "webhook",
    "label": "Webhook intake",
    "enabled": true,
    "signingMode": "bearer"
  }'
```


<!-- tab: JavaScript -->

```javascript
const res = await fetch("/api/routines/routine-1/triggers", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${boardToken}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    kind: "webhook",
    label: "Webhook intake",
    enabled: true,
    signingMode: "bearer",
  }),
});

const created = await res.json();
```


<!-- tab: Python -->

```python
import requests

response = requests.post(
    "http://localhost:3100/api/routines/routine-1/triggers",
    headers={
        "Authorization": f"Bearer {board_token}",
        "Content-Type": "application/json",
    },
    json={
        "kind": "webhook",
        "label": "Webhook intake",
        "enabled": True,
        "signingMode": "bearer",
    },
)

created = response.json()
```


<!-- /tabs -->

## Webhook 设置与测试投递

创建时在请求体中传入 `setupPending: true`，即可让 webhook 触发器进入“设置待完成”状态。触发器会被保存，但所有入站投递只作为连接测试处理，不会实际启动任务。

触发器处于设置待完成状态时：

- 发送到[触发端点](#触发公开触发器)的投递会验证签名和内容，但**不会创建例行任务运行或执行 issue**。
- 系统会按请求的幂等键记录不含内容的测试回执，因此同一测试请求之后重试时，不会在触发器上线后启动例行任务。
- `lastWebhookDelivery` 会反映投递结果，客户端可据此显示测试是否到达。

`lastWebhookDelivery` 对象包含：

| 字段 | 值 | 说明 |
|---|---|---|
| `status` | `received`、`rejected` | `received` 表示签名和负载通过；`rejected` 表示认证失败。 |
| `receivedAt` | ISO 时间戳 | 投递到达的时间。 |
| `test` | 布尔值 | 设置待完成期间的测试投递为 `true`。 |

设置完成时，调用 `PATCH /api/routine-triggers/{triggerId}` 并传入 `setupPending: false`。此后真实投递才会创建运行。设置阶段发送的测试不会重放；只有启用后收到的事件才会触发任务。

<a id="update-trigger"></a>
## 更新触发器

```http
PATCH /api/routine-triggers/{triggerId}
Content-Type: application/json
```

可以更新：

- `label`。
- `enabled`。
- 计划触发器的 `cronExpression` 和 `timezone`。
- webhook 触发器的 `signingMode` 和 `replayWindowSec`。
- `setupPending`：只接受 `false`，用于结束设置阶段并启用 webhook 以接收真实投递（见[Webhook 设置与测试投递](#webhook-设置与测试投递)）。
- `archived`：`true` 会归档触发器，使其退出活动集合；`false` 会恢复触发器。

启用计划触发器时，例行任务所需变量仍必须可以解析。

<a id="delete-trigger"></a>
## 删除触发器

```http
DELETE /api/routine-triggers/{triggerId}
```

此操作会永久删除触发器。

## 轮换触发器密钥

```http
POST /api/routine-triggers/{triggerId}/rotate-secret
```

仅 webhook 触发器可以轮换密钥。轮换行为：

- 生成新的密钥值。
- 立即使旧密钥失效。
- 保留同一个公开 URL。
- 在响应中返回新的 `webhookUrl` 和 `webhookSecret`。

若密钥已泄露，或想在不更改 webhook URL 的情况下重新签发凭据，请使用此端点。

## 手动运行

```http
POST /api/routines/{routineId}/run
Content-Type: application/json
```

请求字段：

| 字段 | 必填 | 说明 |
|---|---|---|
| `source` | 否 | 默认 `manual`；代码也接受 `api`。 |
| `triggerId` | 否 | 若提供，必须属于此例行任务且已启用。 |
| `payload` | 否 | 存储在运行记录中的任意 JSON 负载。 |
| `variables` | 否 | 显式变量覆盖值。 |
| `projectId` | 否 | 覆盖本次运行使用的项目。 |
| `assigneeAgentId` | 否 | 覆盖本次运行的默认受派代理。 |
| `idempotencyKey` | 否 | 按相同来源/触发器组合去重重复请求。 |
| `executionWorkspaceId` | 否 | 为创建的 issue 指定可选工作区绑定。 |
| `executionWorkspacePreference` | 否 | 为创建的 issue 指定可选工作区偏好。 |
| `executionWorkspaceSettings` | 否 | 为创建的 issue 指定可选工作区设置对象。 |

手动运行仍遵守并发策略。若已有活动执行 issue，运行可能被标记为 `coalesced` 或 `skipped`，而不会创建新 issue。

若传入 `triggerId`，服务器会检查：

- 该触发器属于此例行任务。
- 该触发器已启用。

端点返回 `202 Accepted`。

### 示例

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/routines/routine-1/run" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "source": "manual",
    "payload": {
      "context": "Run a one-off briefing"
    },
    "idempotencyKey": "briefing-2026-04-15"
  }'
```


<!-- tab: JavaScript -->

```javascript
const res = await fetch("/api/routines/routine-1/run", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${boardToken}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    source: "manual",
    payload: {
      context: "Run a one-off briefing",
    },
    idempotencyKey: "briefing-2026-04-15",
  }),
});

const run = await res.json();
```


<!-- tab: Python -->

```python
import requests

response = requests.post(
    "http://localhost:3100/api/routines/routine-1/run",
    headers={
        "Authorization": f"Bearer {board_token}",
        "Content-Type": "application/json",
    },
    json={
        "source": "manual",
        "payload": {
            "context": "Run a one-off briefing",
        },
        "idempotencyKey": "briefing-2026-04-15",
    },
)

run = response.json()
```


<!-- /tabs -->

## 触发公开触发器

```http
POST /api/routine-triggers/public/{publicId}/fire
```

供外部系统直接调用例行任务的 webhook 触发器。请求必须带 `Content-Type: application/json`，请求体必须是 JSON **对象**；数组、裸字符串或缺失请求体都会被拒绝。内容类型错误时返回 `415 Unsupported Media Type`。

服务器会检查：

- `publicId` 是否对应一个 webhook 触发器。
- 触发器是否仍处于启用状态。
- 例行任务是否仍为 `active`。
- 请求是否符合触发器的签名模式。

允许的请求头取决于签名模式：

- `app_webhook`：`Authorization: Bearer <secret>`，或携带 `sha256=<hex>` 的 `X-Hub-Signature` / `X-Hub-Signature-256`（针对原始请求体的 HMAC-SHA256）。若提供了签名头，该签名必须有效；服务器不会回退到 Bearer 验证。
- `bearer`：`Authorization: Bearer <secret>`。
- `hmac_sha256`：`X-Paperclip-Signature` 和 `X-Paperclip-Timestamp`；签名内容为时间戳、一个点号和原始请求体。
- `github_hmac`：`X-Hub-Signature-256`，缺少时回退到 `X-Paperclip-Signature`；签署原始请求体，不含时间戳。
- `fireflies_hmac`：针对原始请求体的 `X-Hub-Signature`。
- `none`：无需签名。

带时间戳的 HMAC 验证会执行该触发器配置的重放窗口校验。

对 `app_webhook` 触发器，经过验证的 JSON 负载会附加到执行 issue 描述中的围栏数据块，并标注为外部数据，提醒代理不得将其当作指令。数据块最多 16,384 个字符；完整负载始终保存在例行任务运行记录中。未携带幂等键的已签名 `app_webhook` 投递会按触发器和原始请求体摘要去重，因此相同的签名重试不会启动第二次运行。

`fireflies_hmac` 保留其专属处理逻辑：只有通过认证的会议元数据（`event`、`meeting_id`、`timestamp` 和可选的 `client_reference_id`）会成为运行输入；运行按会议去重；除 `meeting.summarized` 外的事件只会收到 `{ "status": "ignored", "routineStarted": false, "linkedIssueId": null }` 确认，不会创建运行。

去重时服务器读取 `Idempotency-Key` 请求头；若没有，则使用 `X-GitHub-Delivery`。重试时使用相同键可防止重复运行；在设置阶段也可避免测试投递在触发器启用后因重试而启动任务。若触发器仍处于[设置待完成](#webhook-设置与测试投递)状态，端点会验证投递并记录测试回执，但不创建运行。

### 示例

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/routine-triggers/public/<public-id>/fire" \
  -H "Authorization: Bearer <webhook-secret>" \
  -H "Content-Type: application/json" \
  -d '{
    "payload": {
      "source": "stripe",
      "event": "invoice.paid"
    }
  }'
```


<!-- tab: JavaScript -->

```javascript
const res = await fetch("/api/routine-triggers/public/<public-id>/fire", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${webhookSecret}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    payload: {
      source: "stripe",
      event: "invoice.paid",
    },
  }),
});

const run = await res.json();
```


<!-- tab: Python -->

```python
import requests

response = requests.post(
    "http://localhost:3100/api/routine-triggers/public/<public-id>/fire",
    headers={
        "Authorization": f"Bearer {webhook_secret}",
        "Content-Type": "application/json",
    },
    json={
        "payload": {
            "source": "stripe",
            "event": "invoice.paid",
        }
    },
)

run = response.json()
```


<!-- /tabs -->

<a id="list-runs"></a>
## 列出运行

```http
GET /api/routines/{routineId}/runs?limit=50
```

返回最近的例行任务运行历史。`limit` 默认值为 50，上限为 200。

每次运行包含：

- `status`
- `source`
- `triggerId`
- `triggerPayload`
- `linkedIssueId`
- `coalescedIntoRunId`
- `failureReason`
- `completedAt`

可用此端点判断例行任务是否创建了新 issue、合并到现有执行，或在创建工作前失败。

## 运行状态

| 状态 | 含义 |
|---|---|
| `received` | 已接受运行，正在处理。 |
| `coalesced` | 已有活动执行，因此本次运行关联到它。 |
| `skipped` | 本次运行没有创建工作；可能是并发策略选择跳过已有活动执行、项目在触发时已暂停、活动门控没有发现新活动，或当前 worktree 尚未获准执行该任务。查看 `failureReason` 区分原因。 |
| `issue_created` | 已创建新的执行 issue。 |
| `completed` | 执行 issue 随后进入 `done` 状态。 |
| `failed` | 执行 issue 失败或取消，或派发失败。 |

<a id="why-a-run-was-skipped"></a>
### 跳过运行的原因

自动触发被抑制时，运行记录和触发器最近结果会提供相应原因：

| `failureReason` | 触发器最近结果 | 含义 |
|---|---|---|
| `no_external_activity` | `skipped_no_activity` | 活动门控未发现自上次派发运行以来的新活动。 |
| `paused` | `Skipped because the project is paused` | 触发时例行任务所属项目已暂停。 |
| `worktree_execution_cutoff` | `skipped_worktree_execution_cutoff` | 服务器正在开发 worktree（`PAPERCLIP_IN_WORKTREE`）中运行，且该任务未获准自动执行：worktree 未启用，或任务创建时间早于 worktree 激活截止时间。计划触发和 webhook 投递都会受此规则影响。 |

由并发策略跳过的运行没有 `failureReason`，而是通过 `linkedIssueId` 记录现有活动执行 issue。若任务有当前活动 issue，列表视图也会显示它。界面的例行任务 **Runs** 区域列出该任务创建的执行 issue，即过滤该任务的 `routine_execution` issue；可在其中搜索并直接更改状态、优先级或受派人。未创建 issue 的运行（例如被跳过的触发）只会出现在此端点及例行任务概览的最近运行中。

## 代理访问规则

代理可以读取所属公司的例行任务，但只能管理指派给自己的任务。

| 操作 | 代理 | 看板 |
|---|---:|---:|
| 列出例行任务 | ✅ | ✅ |
| 查看例行任务详情 | ✅ | ✅ |
| 创建例行任务 | ✅ 仅限自己 | ✅ |
| 更新例行任务 | ✅ 仅限自己 | ✅ |
| 添加触发器 | ✅ 仅限自己 | ✅ |
| 更新触发器 | ✅ 仅限自己 | ✅ |
| 删除触发器 | ✅ 仅限自己 | ✅ |
| 轮换触发器密钥 | ✅ 仅限自己 | ✅ |
| 手动运行 | ✅ 仅限自己 | ✅ |
| 触发公开触发器 | ✅ HTTP 签名匹配时 | ✅ |

看板操作者若要将工作指派给自己以外的代理，需要 `tasks:assign` 权限。

## 实用说明

- 一个例行任务可以有多个触发器。
- 计划触发器会自动计算 `nextRunAt`。
- Webhook 触发器会在后台创建公司密钥。
- 轮换密钥不会更改公开 webhook URL。
- 归档后例行任务不会再次触发。
- 项目暂停时，计划触发会被抑制：系统会记录 `skipped` 运行并将计划推进到下一次触发，但不会创建执行 issue；项目恢复后也不会补跑错过的触发。
- 若存在活动执行 issue，且并发策略不是 `always_enqueue`，运行会关联到该 issue 或被跳过，而不会创建新工作。
- 若 `activityGatePolicy: "require_external_activity"` 且自上次已派发运行以来没有外部活动，计划触发会记录为 `skipped`，之后不会补跑。

常见的代码接入流程是：

1. 创建例行任务。
2. 添加计划或 webhook 触发器。
3. （如需要）检查创建时返回的 webhook 信息。
4. 列出运行记录，确认任务按预期触发。

---

