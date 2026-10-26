---
paperclip_version: v2026.1023.0
seo_title: 决策训练 API
seo_description: 记录人员的判断过程，包括回答智能体问题、批准或拒绝高风险操作，并将其作为智能体之后可学习的信号。
---

# 决策训练

公司中的人员每天都会作出智能体可以学习的判断，例如有人在讨论串中回答智能体的问题、有人批准或拒绝高风险操作，或有人决定执行阶段应如何继续。**决策训练**允许你保留这些时刻。指定一个已经发生的决策后，服务器会将决策者当时可见的所有内容冻结为不可变快照；你可以添加说明决策**原因**的备注。最终，这些内容会成为可导出的带标签训练示例。

你可以把它视为给良好判断过程加书签：快照是输入；备注和记录的结果构成标签。

决策训练示例限定在公司范围内，每次请求都会检查公司访问权限。每个示例还遵守其来源任务的权限：如果你无法读取该任务（例如你无权查看某个私有任务），就不能捕获、预览、列出、导出或读取源自该任务的示例。如果你刚开始使用 API，请先阅读 [API 概览](./overview.md)，了解基础 URL、身份验证和错误代码约定；本页不再重复说明。

> **只有人类可以写入。** 创建、编辑和删除示例都要求调用方是已登录的看板人类用户。使用智能体 token 调用这些路由会收到 `403 Forbidden` 和 `{ "error": "Decision training writes require a human user" }`。这是刻意设计的：此功能旨在记录人类判断。

---

## 三种决策类型

捕获示例时，需要告诉服务器你指的是哪类决策。恰好有三种：

| `sourceKind` | 指向的内容 |
|---|---|
| `interaction` | 智能体在任务讨论串中提出的问题或请求，以及它如何得到解决。 |
| `approval` | 某项任务上的审批请求，无论已批准、拒绝或仍待处理。 |
| `execution_decision` | 针对任务上某个执行阶段记录的决策。 |

无论选择哪种类型，都要提供具体决策的 `sourceId`，以及该决策所在任务的 `issueId`。三者必须匹配。服务器会在指定公司和任务范围内查找决策，因此配对错误时会返回 `404 Not Found`，并根据类型附带 `{ "error": "Decision interaction not found" }`、`{ "error": "Decision approval not found" }` 或 `{ "error": "Execution decision not found" }`。`issueId` 不属于此公司时会返回 `{ "error": "Issue not found" }`。

---

## 哪些内容会被冻结：快照

最重要的概念是**截止点**。快照不是“任务目前的样子”，而是“作出决策那一刻任务的样子”。该时刻之后创建的所有内容都会被排除，因此训练示例绝不会意外包含未来信息。

对于已解决的 `interaction` 或 `approval`，截止点是解决或作出决定的时刻。对于 `execution_decision`，截止点是决策创建的时刻。如果交互或审批仍处于待处理状态，还没有作出决策的时刻，因此截止点设为捕获时间，`decisionOutcome` 返回 `null`。

截止点及之前的所有内容都会复制到带版本号的 `snapshot` 对象中：

| 快照字段 | 内容 |
|---|---|
| `version` | 快照格式版本，目前为 `1`。 |
| `retention` | 此快照采用的数据保留规则：`policy`，以及 `commentDeletion: "redact"` 和 `issueDeletion: "cascade"`。 |
| `capturedAt` | 执行捕获的 ISO 时间戳。 |
| `cutoff` | `at`（ISO 时间戳）、`lastCommentId` 和 `commentCount`。 |
| `issue` | 完整任务记录。 |
| `comments` | 截止点及之前创建的所有任务评论，按时间从旧到新排列。 |
| `runs` | 与该任务关联且在截止点及之前开始并最后更新的智能体运行。 |
| `decision` | 决策本身的 `kind`、`payload`、`actor` 和 `outcome`。 |
| `code` | 尽力收集的代码上下文，详见下文。 |

### 代码上下文

Paperclip 会尝试记录作出决策时针对的是哪段代码，以便训练示例能够有意义地重放。`code` 对象包含 `repoUrl`、`ref`、`commitSha` 和 `resolution`，用于说明对该提交判断的置信程度：

| `resolution` | 含义 |
|---|---|
| `exact` | 提交来自直接产生该决策的那次运行。 |
| `nearest_run` | 该次运行没有提交记录，因此使用截止点之前最近一次有提交记录的运行。 |
| `workspace` | 回退使用执行工作区或项目工作区元数据。 |
| `none` | 无法确定提交。 |

---

## 数据保留：来源发生变化时的处理方式

快照不可变，但仍需遵守删除请求。两项规则都记录在每个快照的 `retention` 块中，策略名称为 `scrub_deleted_comments_v1`：

- **删除评论时进行脱敏。** 如果之后删除了已纳入快照的任务评论，服务器会将快照中的该条评论正文改为空字符串、写入 `deletedAt`，并添加原因为 `source_comment_deleted` 的 `retentionRedaction`。快照其余内容不变。
- **删除任务时级联删除。** 删除任务后，其决策训练示例也会一并删除。

---

## 创建示例

```
POST /api/companies/{companyId}/decision-training
```

将决策捕获为训练示例。成功时返回 `201 Created` 和已存储的示例。

请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `sourceKind` | 是 | `interaction`、`approval` 或 `execution_decision` 之一。 |
| `sourceId` | 是 | 决策 UUID。 |
| `issueId` | 是 | 决策所属任务的 UUID。 |
| `notes` | 否 | 对该决策的说明，最多 100,000 个字符；默认 `""`。 |

未知字段会被拒绝。

**每人对每个决策只能创建一个示例。** 服务器会对 `(sourceKind, sourceId, createdByUserId)` 强制唯一。如果你已训练过此决策，再次尝试会返回 `409 Conflict` 和 `{ "error": "This decision is already trained by this user" }`。同事仍可以独立训练同一个决策；约束以作者为单位，因此多人可以对同一时刻给出不同标签。

无法读取来源任务时，创建会被拒绝，并返回 `404 Not Found` 和 `{ "error": "Decision training source not found" }`。[预览示例](#预览示例)路由也会返回相同错误。

创建成功后，会记录 `decision_training.created` 活动。

### 示例

```bash
curl -sS -X POST \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/companies/{companyId}/decision-training" \
  -d '{
    "sourceKind": "approval",
    "sourceId": "8f1c2b7e-5f3a-4d61-9a5e-2c0b7d4e1a93",
    "issueId": "3a9d0c14-77b2-4f8e-b0e6-9d5c1e2f7a44",
    "notes": "Rejected because the migration had no rollback path. Always ask for one on schema changes."
  }'
```

---

## 预览示例

```
POST /api/companies/{companyId}/decision-training/preview
```

想在确认前查看将被冻结的确切内容吗？预览会执行相同的捕获流程并返回结果，**但不会保存任何内容**。

请求正文接受 `sourceKind`、`sourceId` 和 `issueId`，与创建时相同，但不包含 `notes`。未知字段会被拒绝。

响应如下：

```json
{
  "cutoffAt": "2026-07-14T09:21:03.118Z",
  "decisionOutcome": "rejected",
  "snapshot": { "version": 1, "...": "..." }
}
```

预览是只读操作，但由于会暴露相同的捕获决策状态，因此与写入操作一样，只允许人类用户调用。

---

## 列出示例

```
GET /api/companies/{companyId}/decision-training
```

返回公司的示例，按时间从新到旧排列；只包含调用方有权读取其来源任务的示例。每行是 `{ example, issueTitle, issueIdentifier }`，因此无需针对每个任务再查询一次即可呈现列表。

### 查询参数

| 参数 | 说明 |
|---|---|
| `project` | 只返回任务属于指定项目（UUID）的示例。 |
| `kind` | 按来源类型过滤：`interaction`、`approval` 或 `execution_decision`。 |
| `author` | 按创建示例的用户 ID 过滤。 |
| `q` | 对示例的 `notes` 以及任务的 `title` 和 `identifier` 进行不区分大小写的搜索，最多 500 个字符。 |

查询无效时会返回 `400 Bad Request` 和 `{ "error": "Invalid decision training query", "details": { ... } }`。

---

## 获取示例

```
GET /api/decision-training/{id}
```

按 UUID 返回一个示例，包括完整的 `snapshot`、`notes` 和 `notesHistory`。如果 ID 不是 UUID，或者示例属于你无权访问的公司或任务，响应都会相同：`404 Not Found` 和 `{ "error": "Decision training example not found" }`。这样路由不会泄露你无权查看的示例是否存在。

读取会记录为 `decision_training.read` 活动。

### 示例字段

| 字段 | 含义 |
|---|---|
| `id` | 示例 UUID。 |
| `companyId` | 所属公司。 |
| `sourceKind` / `sourceId` | 捕获此示例的决策。 |
| `issueId` | 决策所在的任务。 |
| `cutoffAt` | 快照被冻结的时刻。 |
| `notes` | 当前备注文本。 |
| `notesHistory` | 之前的备注版本，每项为 `{ author, at, body }`。 |
| `decisionOutcome` | 记录的决策结果；捕获时仍待处理则为 `null`。 |
| `retentionPolicy` | 所应用的数据保留策略：`scrub_deleted_comments_v1`。 |
| `snapshot` | 上文说明的冻结状态。 |
| `createdByUserId` | 捕获示例的人类用户。 |
| `createdAt` / `updatedAt` | 时间戳。 |

---

## 更新备注

```
PATCH /api/decision-training/{id}
```

快照永不更改，但你可以修改说明。此路由只更新 `notes`。

请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `notes` | 是 | 最多 100,000 个字符。 |

未知字段会被拒绝。**只有作者可以编辑。** 其他人尝试修改你的示例时会收到 `403 Forbidden` 和 `{ "error": "Only the example author can change decision training examples" }`。

旧备注永远不会丢失；每次实际更改都会将上一个正文以 `{ author, at, body }` 形式加入 `notesHistory`，并记录 `decision_training.notes_updated` 活动。提交完全相同的文本属于空操作：示例原样返回，不会新增历史记录或活动。

---

## 删除示例

```
DELETE /api/decision-training/{id}
```

彻底删除示例。返回 `204 No Content`，且正文为空。

与编辑一样，只有作者可以删除，并且调用方必须是人类用户。删除后会记录 `decision_training.deleted` 活动。

---

## 导出为 JSONL

```
GET /api/companies/{companyId}/decision-training/export.jsonl
```

将训练流水线指向此端点。它以换行符分隔的 JSON 格式导出公司的示例，顺序和内容与列表路由相同：按时间从新到旧排列，并且仅包含调用方有权读取其来源任务的示例。响应的 content type 为 `application/x-ndjson`。

每行是一个完整 JSON 对象，且恰好包含三个键：

| 键 | 内容 |
|---|---|
| `retentionPolicy` | 示例的数据保留策略，例如 `scrub_deleted_comments_v1`。 |
| `state` | 完整的冻结 `snapshot`，也就是示例的输入。 |
| `label` | `{ "outcome": ..., "notes": ... }`，即记录的 `decisionOutcome` 和作者备注。 |

单行内容如下（此处为了便于阅读进行了排版；导出文件中每个对象只占一行）：

```json
{
  "retentionPolicy": "scrub_deleted_comments_v1",
  "state": { "version": 1, "capturedAt": "...", "issue": {}, "comments": [], "runs": [], "decision": {}, "code": {} },
  "label": { "outcome": "rejected", "notes": "Rejected because the migration had no rollback path." }
}
```

只要有示例，响应正文就会以换行符结束；没有示例时，正文完全为空。注意，导出不应用任何过滤条件，始终返回公司中的所有示例。若要获取过滤后的子集，请使用列表路由。

每次导出都会记录为 `decision_training.exported` 活动，其中包括导出的示例数量和 ID，因此总有记录可以追踪是谁取走了数据。

### 示例

```bash
curl -sS \
  -H "Authorization: Bearer {token}" \
  "https://paperclip.example.com/api/companies/{companyId}/decision-training/export.jsonl" \
  -o decision-training.jsonl
```

---

## 后续阅读

- [任务](./issues.md)：查看决策训练示例所属任务，以及可供捕获的讨论串交互来源。
- [审批](./approvals.md)：了解使用 `sourceKind: "approval"` 捕获的审批请求。
- [活动记录](./activity.md)：查看本页所述的 `decision_training.*` 活动。
- [API 概览](./overview.md)：查看基础 URL、身份验证、公司范围和通用错误代码表。
