---
seo_title: 决策 API
seo_description: 当代理知道下一步该做什么、但无权独自执行时，如何提出问题、给出选项并等待人工决定。
---

# 决策

有时代理已经判断出下一步应该做什么，却没有权限自行执行。这时它可以创建一个**决策**来征求意见：写下问题，列出简短的选项，并附上每个选项会执行的确切变更。你选定一个选项后，Paperclip 会替你执行相应变更。

整个交互都会被记录。决策会存储下来，选项经过签名以防被篡改；效果逐项执行，每项都有审计记录；事后也能查询结果。开放决策还会作为 `decision` 来源出现在[关注事项](./attention.md)动态中，与其他需要你处理的项目显示在同一处。

本页也介绍**决策队列**和**分流**——这两个辅助机制可将关注事项动态整理成具名分类，并为事项设置决策截止时间。

决策限定在公司范围内，每个请求都会检查公司访问权限。如果你刚接触此 API，请先阅读 [API 概览](./overview.md)，了解基础 URL、认证和错误代码约定；本页以这些内容为基础，不再重复说明。

> **代理提议，人来决策。** 创建决策需要携带运行上下文的代理令牌——看板令牌会返回 `403 Forbidden`，响应体为 `{ "error": "Agent run context required" }`。作出决策和忽略决策需要看板权限。看板或提议该决策的代理都可以取消它。

---

## 决策的结构

决策包含 `title`、Markdown 格式的 `body` 和 1 到 8 个 `options`。每个选项都是一个带标签的选择，并包含一组**效果**——选择该选项后由 Paperclip 执行的具体变更。

| 效果 `type` | 作用 |
|---|---|
| `comment_on_issue` | 在目标 issue 上发布 `bodyMarkdown`。 |
| `create_issue` | 根据 `draft` 创建新 issue。 |
| `update_issue_status` | 设置目标 issue 的 `status`，并可附带 `comment`。 |
| `assign_issue` | 将目标 issue 指派给 `assigneeAgentId` 或 `assigneeUserId`，并可附带 `comment`。 |
| `cancel_issue_tree` | 取消目标 issue 及其所有后代，并留下 `reasonComment`。 |
| `resolve_blocker` | 从目标 issue 的阻塞项中移除 `removeBlockedByIssueIds`。 |

每项效果都需要指定 `targetIssueId` 以及 `strict` 或 `lenient` 形式的 `staleness`。这是安全控制：创建决策时，Paperclip 会将每个引用的 issue 快照保存到 `targetSnapshots`；如果目标在此期间发生变化，`strict` 效果就会拒绝执行。`cancel_issue_tree` 始终使用 `strict`，而包含此效果的选项必须设置 `style: "destructive"`。

选项还可以收集少量有类型的输入。最多可设置 4 个 `inputs` 字段；其值会替换效果评论和原因文本中写入的 `{{input.<id>}}` 占位符。

### 状态和执行状态

| `status` | 含义 |
|---|---|
| `open` | 等待看板回应。 |
| `decided` | 有人选择了一个选项或忽略了决策。 |
| `expired` | `expiresAt` 已过期，或严格模式的目标已不存在。 |
| `cancelled` | 提议决策的代理或看板撤回了决策。 |

决策选定后，`executionStatus` 会显示效果执行情况：执行期间为 `running`，之后变为 `succeeded`、`partial` 或 `failed`。

### 签名

已存储决策的选项和目标快照都受 HMAC 签名（`signedSpec`，版本为 `decision-spec-v1`）保护。Paperclip 在允许你作出或忽略决策前会重新验证签名，确保从代理提议到你回复期间，没有人能更改该决策将执行的操作。签名不匹配时会返回 `403 Forbidden`，响应体为 `{ "error": "Decision signature verification failed" }`。

如果设置了 `PAPERCLIP_DECISION_SIGNING_SECRET`，签名密钥会从该环境变量读取（至少 32 个字符）；否则服务器会生成 `decision-signing.key` 文件，并像其他秘密文件一样以 `0600` 权限保存。

---

## 提议决策

```
POST /api/companies/{companyId}/decisions
```

创建单个决策。这是代理端点：调用方必须使用同时包含代理 ID 和运行 ID 的代理令牌，并且该运行必须关联到 issue。成功时返回 `201 Created` 和已存储的决策。

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `title` | 是 | 1–500 个字符。 |
| `body` | 是 | 最多 100,000 个字符。 |
| `options` | 是 | 1–8 个选项；ID 必须唯一。 |
| `ruleKey` | 否 | 最多 240 个字符，可为 null。统计端点会用它对决策分组。 |
| `inputs` | 否 | 最多 4 个字段，可为 null；ID 必须唯一。 |
| `expiresAt` | 否 | 默认设为 7 天后。必须是未来时间且不超过 30 天。 |
| `idempotencyKey` | 否 | 1–500 个字符，可为 null；在公司范围内唯一。 |
| `continuationPolicy` | 否 | `none`（默认）或 `wake_origin_agent`。 |
| `metadata` | 否 | 任意对象。 |

系统会拒绝未知字段。

每个选项都需要 `id`（1–120 个字符）和 `label`（1–240 个字符），可选 `description`（最多 2,000 个字符）及 `style`（`default`、`primary` 或 `destructive`），并最多包含 10 项 `effects`。每个输入需要 `id`（1–120 个字符）和 `label`（1–240 个字符），可选 `placeholder`、`required` 和 `maxLength`（最多 20,000）。

如果决策解决后应立即唤醒代理，请设置 `continuationPolicy: "wake_origin_agent"`。Paperclip 随后会发送一次心跳唤醒，并附带 `issueId`、`decisionId` 和 `outcome`（值为 `decided`、`expired` 或 `cancelled`）。该通知采用至少一次投递语义。

**来源关系会经过验证，不会被盲目信任。** 运行 ID 必须属于当前公司中的调用代理，否则会返回 `403 Forbidden`，响应体为 `{ "error": "Decision provenance requires the origin run" }`。如果该运行上下文没有 issue，则会返回 `422 Unprocessable Entity`，响应体为 `{ "error": "Origin run is not issue-scoped" }`。

**代理不能提议超出自身权限范围的操作。** 系统会对选项涉及的每个 issue 检查两次：可见性需要 `issue:read`；执行效果需要 `issue:comment` 或 `issue:mutate`。检查失败时会返回 `403 Forbidden`，响应体分别为 `{ "error": "Decision target is outside the origin visibility boundary" }` 或 `{ "error": "Decision effect exceeds the origin authority boundary" }`。

其他可能的响应：

| 状态 | 请求体 | 情况 |
|---|---|---|
| `409 Conflict` | `{ "error": "Decision idempotency key already used with a different payload" }` | 该 key 已用于不同内容。重放完全相同的请求体会返回原决策。 |
| `422 Unprocessable Entity` | `{ "error": "All referenced issues must exist in the company" }` | 某个效果引用了公司中不存在的 issue。 |
| `422 Unprocessable Entity` | `{ "error": "expiresAt must be within 30 days" }` | 过期时间已过去或超过 30 天。 |
| `429 Too Many Requests` | `{ "error": "Open decision cap reached" }` | 该代理的开放决策过多。默认上限为 50，可通过 `PAPERCLIP_DECISIONS_OPEN_CAP` 设置。 |

成功创建后会记录一条 `decision.created` 活动。

### 示例

```bash
curl -sS -X POST \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/companies/{companyId}/decisions" \
  -d '{
    "title": "Ship the migration, or hold for a rollback plan?",
    "body": "The schema change is ready but has no rollback path.",
    "ruleKey": "migration-gate",
    "options": [
      {
        "id": "ship",
        "label": "Ship it",
        "style": "primary",
        "effects": [
          {
            "type": "update_issue_status",
            "targetIssueId": "3a9d0c14-77b2-4f8e-b0e6-9d5c1e2f7a44",
            "staleness": "strict",
            "status": "done"
          }
        ]
      },
      {
        "id": "hold",
        "label": "Hold for a rollback plan",
        "effects": [
          {
            "type": "comment_on_issue",
            "targetIssueId": "3a9d0c14-77b2-4f8e-b0e6-9d5c1e2f7a44",
            "staleness": "lenient",
            "bodyMarkdown": "Holding: {{input.reason}}"
          }
        ]
      }
    ],
    "inputs": [
      { "id": "reason", "label": "Why?", "required": true, "maxLength": 500 }
    ]
  }'
```

---

## 提议决策包

```
POST /api/companies/{companyId}/decision-bundles
```

当一项工作同时引出多个相关问题时，代理可以将它们打包提议。决策包为一组相关决策提供共享的 `title` 和 `summary`，使动态视图能为同组决策显示一个共同标题；不过每个决策仍需单独作答。

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `title` | 是 | 1–500 个字符。 |
| `summary` | 是 | 最多 100,000 个字符。 |
| `decisions` | 是 | 1–50 个决策，每个都使用上方的创建请求体。 |

系统会拒绝未知字段。运行上下文要求与创建单个决策相同。成功时返回 `201 Created`，包含决策包及其 `decisions` 数组。

---

## 列出决策

```
GET /api/companies/{companyId}/decisions
```

按创建时间从新到旧返回公司的决策。此路由需要看板访问权限。

### 查询参数

| 参数 | 说明 |
|---|---|
| `status` | `open`、`decided`、`expired` 或 `cancelled`。 |
| `bundleId` | 只返回此决策包中的决策（UUID）。 |
| `targetIssueId` | 只返回效果涉及此 issue 的决策（UUID）。 |
| `originAgentId` | 只返回该代理提议的决策（UUID）。 |
| `limit` | 不超过 100 的正整数，默认 50。 |

无效查询会返回 `400 Bad Request`，请求体为 `{ "error": "Invalid decision filters", "details": { ... } }`。

每行还会附带两个额外字段，使列表视图不必再发起额外请求。每行都包含 `targetChanged`——这是一个从目标 issue ID 映射到布尔值的对象；如果某个 issue 自快照之后发生过变化，其值为 `true`，UI 可据此将无法执行的选项显示为灰色。状态不再是 `open` 的行还包含 `executions`，按 `effectIndex` 排序。

---

## 读取决策统计

```
GET /api/companies/{companyId}/decisions/stats
```

此端点提供反馈循环。它按 `ruleKey` 分组，统计某类提议有多少被接受、拒绝或过期，可用于调整代理提议时遵循的规则。看板和代理令牌都可以调用。

### 查询参数

| 参数 | 必填 | 说明 |
|---|---|---|
| `groupBy` | 是 | 必须为 `ruleKey`。 |
| `originAgentId` | 否 | 限定到单个代理（UUID）。 |
| `since` | 否 | 只统计在此时间或之后创建的决策。 |

系统会拒绝未知参数；查询无效时返回 `400 Bad Request`，请求体为 `{ "error": "Invalid decision stats filters", "details": { ... } }`。代理只能查看自己的数据：传入其他代理的 `originAgentId` 会返回 `403 Forbidden`，请求体为 `{ "error": "Agents may only read their own decision stats" }`；代理请求始终会自动限定到该代理自身。

响应示例：

```json
{
  "groupBy": "ruleKey",
  "filters": { "originAgentId": null, "since": null },
  "totals": { "proposed": 12, "accepted": 7, "rejected": 3, "expired": 2 },
  "groups": [
    {
      "ruleKey": "migration-gate",
      "proposed": 5,
      "accepted": 3,
      "rejected": 1,
      "expired": 1,
      "chosenOptions": [{ "optionId": "ship", "count": 3 }]
    }
  ]
}
```

计数含义如下：`proposed` 统计范围内的所有决策；`accepted` 是已作出选择而非被忽略的决策；`rejected` 是被明确忽略的决策；`expired` 单独统计。已取消的决策只计入 `proposed`。`chosenOptions` 只统计被接受的结果。

---

## 获取决策

```
GET /api/decisions/{id}
```

返回单个决策及其 `executions` 数组。看板和代理令牌都可以调用，但代理只能读取自己提议的决策；否则返回 `403 Forbidden`，响应体为 `{ "error": "Only the origin agent may read this decision" }`。

决策不存在，或其所属公司不在调用方的访问范围内时，都会返回 `404 Not Found`，响应体为 `{ "error": "Decision not found" }`；这两种情况使用相同响应。

每条执行记录都会告知你 `effectIndex`、`effectType`、`targetIssueId`、`status`（`claimed`、`executed`、`failed` 或 `skipped`）、`result`、`error`、`activityLogId` 和 `executedAt`。

---

## 作出决策

```
POST /api/decisions/{id}/decide
```

选择一个选项并让 Paperclip 执行其效果。此操作需要看板访问权限。响应包含该决策和执行记录。

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `optionId` | 是 | 1–120 个字符，必须与此决策中的某个选项匹配。 |
| `inputValues` | 否 | 输入 ID 到字符串的映射对象；每个字符串最多 20,000 个字符。 |
| `idempotencyKey` | 否 | 1–500 个字符，可为 null，使重试安全。 |

系统会拒绝未知字段。

效果按顺序执行，每项都有独立事务、执行记录和审计日志。如果 `strict` 效果的目标已经改变，该效果会以 `target_changed` 原因记录为 `skipped`，而不是导致整个决策失败。因此，出现部分成功是正常且可见的结果，不会被悄悄隐藏。所有效果均执行后，`executionStatus` 变为 `succeeded`；部分执行时为 `partial`；一个也未执行时为 `failed`。

重试是安全的。重复使用相同的 `idempotencyKey`，或使用相同的 `optionId` 和输入值重新发送请求，会继续已有结果，而不会重复作出决策。不同用户重放请求时会返回 `403 Forbidden`，响应体为 `{ "error": "Decision replay belongs to a different user" }`。

其他响应：

| 状态 | 请求体 | 情况 |
|---|---|---|
| `403 Forbidden` | `{ "error": "Decision signature verification failed" }` | 已存储规范与签名不匹配。 |
| `409 Conflict` | `{ "error": "decision_already_resolved", "code": "decision_already_resolved", ... }` | 决策已先被其他人处理。 |
| `409 Conflict` | `{ "error": "decision_expired", "code": "decision_expired", ... }` | 决策已过期。Paperclip 会在作出响应时将其标记为 `expired`。 |
| `422 Unprocessable Entity` | `{ "error": "Unknown optionId" }` | 此决策中不存在该选项 ID。 |
| `422 Unprocessable Entity` | `{ "error": "Input {id} is required" }` / `{ "error": "Input {id} is too long" }` | 必填字段为空，或值超出 `maxLength`。 |

作出决策会记录 `decision.decided` 活动；每项效果还会分别记录 `decision.effect_executed`、`decision.effect_failed` 或 `decision.effect_skipped`。

### 示例

```bash
curl -sS -X POST \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/decisions/{id}/decide" \
  -d '{
    "optionId": "hold",
    "inputValues": { "reason": "No rollback path yet." },
    "idempotencyKey": "7c1f4a90-6a2f-4c1d-8f0c-6d0b8b0f5f21"
  }'
```

---

## 忽略决策

```
POST /api/decisions/{id}/dismiss
```

拒绝该提议，不执行任何操作。此操作需要看板访问权限，可选传入最多 20,000 个字符的 `reason`。系统会拒绝未知字段。

忽略也是一种明确答复，而非没有回复——统计端点会将其计为 `rejected`，因此它能提供有用的反馈。如果决策中有一个没有效果的选项，该选项会被记录为选择结果；否则，Paperclip 会保存 `chosenOptionId: "dismissed"`、`executionStatus: "succeeded"` 和 `metadata.dismissed: true`。无论哪种情况，都会记录 `decision.dismissed` 活动。

---

## 取消决策

```
POST /api/decisions/{id}/cancel
```

撤回一个开放决策。请求体为空。看板令牌可以取消其有权访问的公司中的任意决策；代理只能取消自己提议的决策，否则返回 `403 Forbidden`，响应体为 `{ "error": "Only the origin agent may cancel" }`。

重复取消已取消的决策属于无操作，会返回同一记录，因此重试安全。取消其他已解决的决策会返回 `409 Conflict` 和 `decision_already_resolved`。取消操作会记录 `decision.cancelled` 活动；如果决策配置了后续唤醒策略，还会发送相应唤醒通知。

> 上述每个写入路由都会严格校验请求体。格式错误的请求体会返回 `400 Bad Request`，响应体为 `{ "error": "Validation error", "details": [ ... ] }`。

---

## 决策队列

队列是关注事项动态中具名的分类。它不是给每类工作再添加一个状态字段，而是一种**辅助记录**：通过 `(sourceKind, sourceId)` 指向关注事项，同时不改动其关联审批、交互或运行。一个事项可以同时存在于多个队列中；将其从队列移除不会影响底层工作。

队列路由接受看板和代理令牌。读取需要 `decision_queue:read` 权限；创建、更新和修改队列成员需要 `decision_queue:manage`。无权访问时会返回 `403 Forbidden`，并附带授权说明。

这些路由中的 `sourceKind` 必须是以下某一种关注事项来源：`approval`、`decision`、`issue_thread_interaction`、`join_request`、`recovery_action`、`productivity_review`、`blocker_attention`、`review`、`failed_run`、`budget_alert` 或 `agent_error_alert`。

### 列出预置规则

```
GET /api/companies/{companyId}/decision-queue-seed-rules
```

返回 Paperclip 可自动填充的内置队列及其对应信号。这是一份静态目录，可以在公司尚无队列时先了解自动填充能做什么。

| 队列 `key` | 标题 | 数据来源 |
|---|---|---|
| `prs` | PR | `issue_has_pull_request_work_product`——issue 含有 `pull_request` 工作成果的关注事项。 |
| `plans` | 计划 | `plan_document_confirmation`——关联到 issue 计划文档的待处理 `request_confirmation` 交互。 |
| `questions` | 问题 | `ask_user_questions`——待处理的 `ask_user_questions` 交互。 |

每项的结构为 `{ key, title, description, rules: [{ key, signal, description }] }`。

构建关注事项动态时会执行自动填充：匹配事项出现时，Paperclip 会在队列缺失时创建队列、添加事项，并记录 `decision_queue.seeded` 和 `decision_queue_item.seeded` 活动。预置队列的默认值为 `seedRulesEnabled: true`；通过更新路由关闭此项后，Paperclip 将停止向队列添加内容，但保留队列和现有事项。

### 列出队列

```
GET /api/companies/{companyId}/decision-queues
```

按更新时间从新到旧返回公司的队列。每个队列包含以下字段：

| 字段 | 内容 |
|---|---|
| `id` / `companyId` / `key` | 标识信息。`key` 在公司内唯一。 |
| `title` / `description` | 显示文本。 |
| `createdByType` | `agent`、`user` 或 `system`。 |
| `createdByAgentId` / `createdByUserId` / `createdByRunId` | 创建者或来源运行。 |
| `retentionDays` | 保留天数；不设置时为 `null`。 |
| `seedRules` / `seedRulesEnabled` | 此队列使用的自动填充规则及是否启用。 |
| `itemCount` | **你能查看的**事项数量；计数会遵循你对各来源的读取权限。 |
| `createdAt` / `updatedAt` | 时间戳。 |

### 创建队列

```
POST /api/companies/{companyId}/decision-queues
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `key` | 是 | 1–80 个字符；须为 URL 安全的小写 kebab-case 格式，符合 `^[a-z0-9]+(?:-[a-z0-9]+)*$`。 |
| `title` | 是 | 1–120 个字符。 |
| `description` | 否 | 最多 2,000 个字符，可为 null。 |
| `retentionDays` | 否 | 1–3,650 的整数，可为 null。 |

系统会拒绝未知字段。创建成功返回 `201 Created`；如果该 key 已存在，不会报错，而是以 `200 OK` 返回现有队列，因此配置脚本可以重复运行。实际创建队列时会记录 `decision_queue.created` 活动。

### 更新队列

```
PATCH /api/companies/{companyId}/decision-queues/{key}
```

至少传入 `title`、`description`、`retentionDays` 或 `seedRulesEnabled` 之一。系统会拒绝未知字段。key 不存在时返回 `404 Not Found`，请求体为 `{ "error": "Decision queue not found" }`。每次更新都会记录 `decision_queue.updated` 活动。

### 列出队列事项

```
GET /api/companies/{companyId}/decision-queues/{key}/items
```

按创建时间从新到旧返回队列事项，但只包含你有权读取的来源。每项包含 `id`、`companyId`、`queueId`、`sourceKind`、`sourceId`、`addedByType`（`agent`、`user` 或 `system`）、`addedByAgentId`、`addedByUserId`、`addedByRunId`、`responsibleUserId` 和 `createdAt`。

### 添加事项

```
POST /api/companies/{companyId}/decision-queues/{key}/items
```

请求体为 `{ "sourceKind": ..., "sourceId": ... }`，其中 `sourceId` 长度为 1–500 个字符。系统会拒绝未知字段。

队列成员关系遵循可见性规则：只能添加自己有权读取的来源。如果来源不可读，系统返回 `404 Not Found` 和 `{ "error": "Attention source not found" }`，而不是权限错误，从而防止通过此路由探测不可见工作。队列 key 不存在时返回 `404 Not Found`，请求体为 `{ "error": "Decision queue not found" }`。

新增事项返回 `201 Created`；如果事项已在队列中，则以 `200 OK` 返回现有行。添加事项会记录 `decision_queue_item.added` 活动。

### 移除事项

```
DELETE /api/companies/{companyId}/decision-queues/{key}/items/{sourceKind}/{sourceId}
```

成功时返回被移除的事项。来源标识格式错误时返回 `400 Bad Request`，请求体为 `{ "error": "Invalid attention source identity" }`；事项不在队列中时返回 `404 Not Found`，请求体为 `{ "error": "Decision queue item not found" }`。

看板操作员可以清理来源已不存在的事项；代理则仍必须有权读取对应来源。移除事项会记录 `decision_queue_item.removed` 活动。

---

## 分流

分流回答的问题与队列不同：它关注的不是“这件事属于哪个分类”，而是“我需要在什么时候处理它”。与队列事项一样，分流也是按 `(sourceKind, sourceId)` 标识的辅助记录，因此可用于所有关注事项类型，而不会修改其背后的工作。

### 读取分流信息

```
GET /api/companies/{companyId}/decision-triage/{sourceKind}/{sourceId}
```

需要 `decision_queue:read` 权限。如果事项此前从未设置分流信息，则返回 `null`；否则返回分流记录。

| 字段 | 内容 |
|---|---|
| `id` / `companyId` / `sourceKind` / `sourceId` | 标识信息。 |
| `decideBy` | `today`、`this_week`、`whenever`、`YYYY-MM-DD` 日期或 `null`。 |
| `snoozedUntil` | 事项重新出现的时间；未暂缓时为 `null`。 |
| `setByType` | `agent` 或 `user`。 |
| `setByAgentId` / `setByUserId` / `setByRunId` / `responsibleUserId` | 设置者信息。 |
| `version` | 每次写入时递增。 |
| `createdAt` / `updatedAt` | 时间戳。 |

无法读取的来源会返回 `404 Not Found`，请求体为 `{ "error": "Attention source not found" }`。

### 设置分流信息

```
PUT /api/companies/{companyId}/decision-triage/{sourceKind}/{sourceId}
```

需要 `decision_triage:manage` 权限。至少传入以下字段之一：

| 字段 | 说明 |
|---|---|
| `decideBy` | `today`、`this_week`、`whenever` 或 `YYYY-MM-DD` 格式的日期。传入 `null` 可清除。 |
| `snoozedUntil` | 带时区偏移量的 ISO 时间戳。传入 `null` 可清除；时间不得超过五年。 |

系统会拒绝未知字段。省略字段会保留当前值，因此可以调整截止时间而不影响暂缓设置。暂缓时间超过五年时会返回 `422 Unprocessable Entity`，请求体为 `{ "error": "snoozedUntil must be within five years" }`。

对同一来源的写入会串行处理，并递增 `version`，因此多人同时分流同一事项时不会交错覆盖。每次写入都会记录 `decision_triage.updated` 活动，其中包含修改前后的值。

分流信息会直接反映到关注事项动态中：每项会显示 `decideBy` 和 `snoozedUntil`，`sort=decide` 会按截止时间排序，`decideNowCount` 则统计今天到期的事项。排序机制详见[关注事项](./attention.md)。

---

## 后续阅读

- [决策](../../guides/day-to-day/decisions.md)——这些路由背后的操作流程。
- [关注事项](./attention.md)——开放决策、队列和分流信息显示的排序动态。
- [决策训练](./decision-training.md)——将决策保存为带标签的训练示例。
- [Issues](./issues.md)——决策效果所作用的 issue。
- [活动](./activity.md)——记录 `decision.*`、`decision_queue.*` 和 `decision_triage.*` 活动的位置。
- [API 概览](./overview.md)——基础 URL、认证、公司作用域和通用错误代码表。
