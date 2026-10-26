---
paperclip_version: v2026.831.1
seo_title: 案例 API
seo_description: 案例用于承载无法容纳在单个任务中的文档密集型工作，例如研究报告、调查、提案、法律与合规材料。
---

# 案例 API

**Case（案例）**是一种一等工作对象，用于处理难以规整、文档密集且无法妥善放进单个任务的工作，例如研究报告、调查、提案、法律审查或合规审查。任务通过状态流转跟踪一项工作；案例则是一个容器：它有由你定义的类型、标题和摘要、任意结构化的 `fields`、附属文档（含完整修订历史、锁定功能和行内批注讨论串）、文件附件、标签、事件时间线，以及指向发起该案例或为其提供工作的任务的类型化链接。案例还可以嵌套，因此父案例能够汇集一组相关子案例。

案例的作用范围限定在公司内；与控制平面 API 的其他部分一样，每个请求都会执行公司访问权限检查。如果你刚开始使用此 API，请先阅读 [API 概览](./overview.md)，了解基础 URL、身份验证和错误代码约定。本页基于这些内容展开，不会重复说明。

> **实验性功能——默认关闭。** 案例由实例级实验性设置控制。管理员启用 `enableCases` 之前，本页的**所有**端点都会返回 `403 Forbidden`，响应内容为 `{ "error": "Cases are disabled" }`。由于该功能仍处于实验阶段，以下请求和响应格式可能会在不同版本间变化。

---

## 启用案例

每个案例路由在开始处理时都会先检查功能是否已启用。服务器会读取实例的实验性设置；如果 `enableCases` 未开启，就会立即停止处理，并返回：

```json
{ "error": "Cases are disabled" }
```

此响应的 HTTP 状态为 `403 Forbidden`。调用本页中的任何接口前，请先通过实例的实验性设置启用案例。此开关作用于整个实例，不针对单个公司。

---

## 案例的标识方式

每个案例恰好属于一个公司，并由服务器分配两种标识：

- `caseNumber`：公司内按顺序分配的整数，在该公司内唯一。
- `identifier`：便于阅读的标识符，由公司的任务前缀构成，例如 `PAP-C1`、`PAP-C2` 等。（案例与任务共用前缀；案例会在编号前加上 `C`。）

在单个案例的路由中，`{id}` 可以是案例 UUID，也可以是上述 `identifier`。服务器会在处理请求前解析该标识符，且不区分大小写，与任务路由的处理方式相同。

案例还包含以下核心字段：

| 字段 | 含义 |
|---|---|
| `caseType` | 由你选择的自由格式字符串，长度为 1–120 个字符，例如 `investigation`、`proposal` 或 `legal_review`。案例可按类型分组和筛选。 |
| `key` | 类型内可选的稳定键。同一公司中的 `(caseType, key)` 组合必须唯一，并决定创建时的 upsert 行为（见下文）。 |
| `title` | 必填，长度为 1–500 个字符。 |
| `summary` | 可选的自由文本，最多 8,000 个字符。 |
| `status` | 取值为 `draft`、`in_progress`、`in_review`、`approved`、`done`、`cancelled` 之一。默认值为 `draft`。 |
| `fields` | 用于存放自定义结构化数据的任意 JSON 对象。默认为 `{}`。 |
| `projectId` | 案例所属项目，可选。项目必须属于同一公司。 |
| `parentCaseId` | 可选的父案例，因此案例可以嵌套。父案例必须属于同一公司，且案例不能将自身设为父案例。 |
| `completedAt` | 状态变为 `done` 或 `cancelled` 时由服务器设置；其他状态下会清除。 |

### 状态值

| 状态 | 常见含义 | 是否终态？ |
|---|---|---|
| `draft` | 刚创建，内容仍在完善。 | 否 |
| `in_progress` | 正在积极处理。 | 否 |
| `in_review` | 暂停处理，等待审核或签核。 | 否 |
| `approved` | 已审核并接受，但尚未正式结案。 | 否 |
| `done` | 已完成。会设置 `completedAt`。 | 是 |
| `cancelled` | 已放弃。会设置 `completedAt`。 | 是 |

---

## 列出案例

```
GET /api/companies/{companyId}/cases
```

返回该公司的案例，并按最近活动时间从新到旧排序。

### 查询参数

| 参数 | 说明 |
|---|---|
| `type` / `types` | 按一个或多个案例类型筛选。`types` 接受逗号分隔的列表，也接受重复传入的参数。 |
| `status` / `statuses` | 按一个或多个状态筛选。特殊值 `status=active` 会返回所有**不是** `done` 或 `cancelled` 的案例。 |
| `project` / `projectId` / `projectIds` | 按一个或多个项目筛选。 |
| `includeNoProject` | 设为 `true`（或 `1`）时，也包含未关联项目的案例。与 `projectIds` 一起使用时，会合并两组结果。 |
| `label` / `labelId` | 筛选带有指定标签的案例。 |
| `parent` | 筛选指定父案例（UUID）的直接子案例。 |
| `q` | 不区分大小写地搜索 `identifier`、`title`、`summary` 和 `key`。 |
| `includeAncestors` | 设为 `true` 时，也返回每个匹配案例的祖先链。每个返回项包含 `matchesListFilters`：符合筛选条件的项为 `true`，仅为提供上下文而加入的祖先项为 `false`。 |
| `limit` | 正整数，范围为 1–200。默认值为 `100`。 |

---

## 获取案例

```
GET /api/cases/{id}
```

返回完整的案例详情。除核心字段外，响应还会合并渲染案例页面所需的相关对象：

- `parent`：父案例的精简摘要（如果有）。
- `labels`：已附加的标签。
- `issueLinks`：已关联的任务；每项包含其 `role` 和任务精简摘要（`id`、`identifier`、`title`、`status`）。
- `documents`：案例的键控文档，每项格式为 `{ key, document }`。
- `attachments`：已上传的文件，每项包含其存储资源。

---

## 创建或更新案例

```
POST /api/companies/{companyId}/cases
```

此路由会在公司内以 `(caseType, key)` 为键执行 **upsert（存在则更新，不存在则创建）**。如果该组合尚无案例，服务器会创建案例并返回 `201 Created`；如果已存在，则会原位更新并返回 `200 OK`。（`null` 是有效的键；每种 `caseType` 可以有一个不带键的案例。）

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `caseType` | 是 | 长度为 1–120 个字符。 |
| `title` | 是 | 长度为 1–500 个字符。 |
| `key` | 否 | 类型内的稳定键；允许为 `null`。 |
| `summary` | 否 | 最多 8,000 个字符，也可以为 `null`。 |
| `status` | 否 | 创建时默认为 `draft`。 |
| `fields` | 否 | 任意 JSON 对象。 |
| `projectId` | 否 | 必须属于该公司，也可以为 `null`。 |
| `parentCaseId` | 否 | 必须属于该公司，也可以为 `null`。 |

不接受未知字段。若 `projectId` 不属于该公司，或 `parentCaseId` 不属于该公司（或指向案例自身），服务器会返回 `422 Unprocessable Entity`，例如 `{ "error": "Project does not belong to company" }` 或 `{ "error": "A case cannot be its own parent" }`。

### 示例

```bash
curl -sS -X POST \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/companies/{companyId}/cases" \
  -d '{
    "caseType": "investigation",
    "key": "incident-4821",
    "title": "Latency spike on checkout",
    "summary": "Investigate the p99 regression seen on 2026-07-10.",
    "status": "in_progress",
    "fields": { "severity": "high" }
  }'
```

---

## 更新案例

```
PATCH /api/cases/{id}
```

对现有案例执行部分更新。可传入 `projectId`、`title`、`summary`、`status`、`fields` 和 `parentCaseId` 中的任意字段，并可附带以下字段之一：

- `labels` / `labelIds`：最多包含 100 个标签 UUID 的数组，会**替换**案例当前的全部标签。传入 `[]` 可清除所有标签。

说明：

- 不接受未知字段。
- 将 `status` 设为 `done` 或 `cancelled` 时会写入 `completedAt`；设为其他状态时会清除该字段。
- `projectId`、`parentCaseId` 以及每个标签 ID 都会根据公司进行校验；不匹配时返回 `422 Unprocessable Entity`（例如 `{ "error": "One or more labels do not belong to company" }`）。
- 更新会在案例时间线上记录相应事件：`status_changed`、`fields_changed`、`child_linked`、`label_added` / `label_removed`，或者通用事件 `updated`。

---

## 将任务关联到案例

案例与任务通过类型化链接关联。链接带有一个**角色**，用于说明任务为何关联到该案例：

| 角色 | 含义 |
|---|---|
| `origin` | 发起此案例的任务。 |
| `work` | 为此案例实际开展工作的任务。 |
| `reference` | 与案例相关、用于提供上下文的任务。 |

### 关联任务

```
POST /api/cases/{id}/links
```

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `issueId` | 是 | 同一公司内某个任务的 UUID。 |
| `role` | 是 | `origin`、`work`、`reference` 之一。 |

一个任务只能关联到同一个案例一次。重复提交相同的 `(case, issue)` 组合时，服务器会返回已有链接，而不会创建重复项。属于其他公司的任务会被拒绝，返回 `422 Unprocessable Entity`（`{ "error": "Issue does not belong to case company" }`）。新链接创建成功时返回 `201 Created`，并记录 `issue_linked` 事件。

### 列出关联到某任务的案例

```
GET /api/issues/{issueId}/cases
```

反向查询：给定一个任务（UUID 或 `PAP-39` 这样的标识符），返回与其关联的所有案例。每项包含链接的 `id`、`role`、`createdAt`，以及 `case` 字段中的案例精简摘要（`id`、`identifier`、`title`、`caseType`、`status`）。

### 自动关联运行任务

智能体运行处理某个案例时，服务器会自动将该运行所关联的任务链接到案例，以便追踪工作。创建案例时，会自动以 `origin` 角色关联发起任务；更新案例、修订其文档、恢复某个修订版本或上传附件时，会自动以 `work` 角色关联该运行的任务。自动链接会记录为 `issue_linked` 事件，并在负载中包含 `autoLinked: true`；如果运行没有关联任务，则跳过自动关联。

---

## 案例文档

案例文档是附加到案例下、通过稳定**键**标识的带版本工件，例如 `summary`、`findings` 或 `report`。每次写入都会追加一个新修订版本，因此不会覆盖历史记录。文档还支持锁定和行内批注讨论串。

文档键长度为 1–120 个字符，可包含字母、数字以及 `_ . : -` 字符（匹配 `^[A-Za-z0-9_.:-]+$`）。默认 `format` 为 `markdown`。

### 获取文档

```
GET /api/cases/{id}/documents/{key}
```

按键返回单个案例文档，包括最新的 `body`。如果该案例不存在此键，返回 `404`（`{ "error": "Case document not found" }`）。

### 创建或更新文档

```
PUT /api/cases/{id}/documents/{key}
```

在 `key` 处创建新文档，或为现有文档追加新修订版本。

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `body` | 是 | 文档内容，最多 200,000 个字符。 |
| `title` | 否 | 可选的文档标题。 |
| `format` | 否 | 默认为 `markdown`。 |
| `changeSummary` | 否 | 附加到新修订版本的可选说明，最多 1,000 个字符。 |
| `baseRevisionId` | 创建时否，更新时是 | 本次编辑所基于的修订版本。 |

并发规则：

- 创建全新文档时省略 `baseRevisionId`。如果该键尚不存在却提供了此字段，会返回 `409 Conflict`（`{ "error": "Case document does not exist yet" }`）。
- 更新现有文档时**必须**发送 `baseRevisionId`。省略该字段会返回 `409 Conflict`（`{ "error": "Case document update requires baseRevisionId" }`）。
- 过期的 `baseRevisionId`（不是当前最新修订版本的 ID）会导致 `409 Conflict`（`{ "error": "Case document was updated by someone else" }`），并附带当前最新修订版本，以便你据此重新定位后重试。
- 如果文档已锁定，写入会被拒绝，并返回 `409 Conflict`（`{ "error": "Document is locked" }`）。

### 修订历史

```
GET /api/cases/{id}/documents/{key}/revisions
```

返回文档的修订版本，按从新到旧排序。每个版本还会包含写入它的智能体显示名称，以及归属于其背后运行的任务（如果有）。

### 恢复修订版本

```
POST /api/cases/{id}/documents/{key}/revisions/{revisionId}/restore
```

恢复某个较早的修订版本时，服务器会将其内容向前写入，成为新的最新修订版本；历史记录会保留，不会回滚。如果尝试恢复的版本已经是最新版本，会返回 `409 Conflict`；文档已锁定时也会像普通写入一样被拒绝。

### 锁定和解锁文档

```
POST /api/cases/{id}/documents/{key}/lock
POST /api/cases/{id}/documents/{key}/unlock
```

锁定文档可禁止后续写入；解锁则重新允许写入。这两个路由均为幂等操作：锁定已锁定的文档，或解锁已解锁的文档，只会返回当前状态。文档锁定期间，写入、恢复和删除操作都会返回 `409 Conflict`，错误内容为 `{ "error": "Document is locked" }`。

### 删除文档

```
DELETE /api/cases/{id}/documents/{key}
```

删除文档及其所有修订版本。已锁定的文档无法删除（返回 `409 Conflict`）。删除不存在的键不会产生操作，但仍返回 `{ "ok": true }`。

---

## 文档批注

批注会将评论讨论串附加到案例文档中的特定段落，采用与任务文档相同的页边批注模式。讨论串锚定在选定的文本范围上，包含一条或多条评论；对话结束后可以将其标记为已解决。用户和智能体都可以发起讨论串并回复。文档后续修订时，服务器会将每个未解决讨论串重新锚定到新修订版本，使反馈仍跟随其所针对的文本。

### 列出讨论串

```
GET /api/cases/{id}/documents/{key}/annotations
```

返回文档上的批注讨论串。查询参数：

- `status`：`open`、`resolved` 或 `all`。默认值为 `open`。
- `includeComments`：设为 `true` 时，每个讨论串都会嵌入其完整评论列表。

### 获取讨论串

```
GET /api/cases/{id}/documents/{key}/annotations/{threadId}
```

返回包含评论的单个讨论串。如果该讨论串不属于此案例文档，返回 `404`（`{ "error": "Annotation thread not found" }`）。

### 创建讨论串

```
POST /api/cases/{id}/documents/{key}/annotations
```

针对某段文本开启新的讨论串，并在同一请求中提交第一条评论。讨论串会锚定到文档当前修订版本。因此，与任务批注一样，服务器会拒绝过期的基础修订版本，以避免锚点落在错误位置；无法匹配的选择器会导致 `422 Unprocessable Entity`。创建成功时返回 `201 Created`。

### 回复讨论串

```
POST /api/cases/{id}/documents/{key}/annotations/{threadId}/comments
```

向现有讨论串添加评论。成功时返回 `201 Created`。

### 解决或重新开启讨论串

```
PATCH /api/cases/{id}/documents/{key}/annotations/{threadId}
```

更改讨论串状态：设为 `resolved` 可结束对话，设为 `open` 可重新开启。

---

## 附件

```
POST /api/cases/{id}/attachments
```

通过 `multipart/form-data` 向案例上传单个文件。文件必须放在名为 `file` 的字段中。

上传规则：

- 只接受一个文件，且字段名必须是 `file`；否则返回 `{ "error": "Missing file field 'file'" }`。
- 空文件会被拒绝，返回 `422 Unprocessable Entity`（`{ "error": "Attachment is empty" }`）。
- 超过部署级附件大小限制（`PAPERCLIP_ATTACHMENT_MAX_BYTES`）的文件会被拒绝，返回 `422 Unprocessable Entity`（`{ "error": "Attachment exceeds {n} bytes" }`）。默认限制为 10 MiB。此上限针对整个实例设置，不针对单个公司。

上传成功时返回 `201 Created`，记录 `attachment_added` 事件；如果请求来自智能体运行，还会自动以 `work` 角色关联该运行的任务。

### 示例

```bash
curl -sS -X POST \
  -H "Authorization: Bearer {token}" \
  -F "file=@./findings.pdf" \
  "https://paperclip.example.com/api/cases/{id}/attachments"
```

---

## 事件时间线

```
GET /api/cases/{id}/events
```

返回案例的事件历史，按从新到旧排序。查询参数：

- `limit`：正整数，范围为 1–500。默认值为 `100`。

每个事件都会附带触发事件的智能体显示名称（`actorAgentName`）；如果事件关联某次运行或任务链接，还会附带相关任务的精简摘要。

事件通过 `kind` 区分类型：

| 类型 | 触发时机 |
|---|---|
| `created` | 创建案例时。 |
| `updated` | 更新案例，但没有更具体的变更类型时。 |
| `status_changed` | 案例状态发生变化时。 |
| `fields_changed` | 更新 `fields` 对象时。 |
| `child_linked` | 将案例重新设为另一个案例的子案例时。 |
| `issue_linked` | 关联任务时（手动关联或自动关联）。 |
| `issue_unlinked` | 移除任务链接时。 |
| `document_revised` | 案例文档新增修订版本时（包括恢复操作）。 |
| `attachment_added` | 上传文件时。 |
| `label_added` | 附加标签时。 |
| `label_removed` | 移除标签时。 |

---

## 后续阅读

- [任务](./issues.md)——案例所关联的工作对象，以及 `POST /api/cases/{id}/links` 和 `GET /api/issues/{issueId}/cases` 使用的任务标识符来源。
- [目标和项目](./goals-and-projects.md)——案例可以通过 `projectId` 归属的项目。
- [API 概览](./overview.md)——基础 URL、身份验证、公司作用范围，以及通用错误代码表。
