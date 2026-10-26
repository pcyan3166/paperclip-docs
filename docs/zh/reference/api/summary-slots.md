---
paperclip_version: v2026.720.0
seo_title: 摘要槽位 API
seo_description: UI 中始终显示项目或工作区当前简要状态的区域，由智能体撰写摘要，无需手动维护。
---

# 摘要槽位

**摘要槽位**是 UI 中用于持续展示当前简要状态的区域，可以位于项目顶部、项目工作区顶部或工作区总览页中。你无需亲自撰写摘要：提交生成请求后，Paperclip 内置的 **Summarizer** 智能体会生成摘要并写入槽位，同时保留修订历史。

可以把槽位理解为带标签的架子，而不是文档。架子由两个要素标识：它描述的对象（例如某个项目），以及它在页面上的位置（例如页眉）。Summarizer 最近写入的内容会放在架子上；每次生成都会追加一个修订版本，而不是覆盖旧内容。

摘要槽位限定在公司范围内，每次请求都会检查公司访问权限。如果你刚开始使用 API，请先阅读 [API 概览](./overview.md)，了解基础 URL、身份验证和错误代码约定；本页会沿用这些规则，不再重复说明。

> **实验功能，默认关闭。** 摘要槽位受实例级实验性设置控制。运维人员开启 `enableSummaries` 前，本页**所有**端点都会返回 `404 Not Found` 和 `{ "error": "Summaries are not enabled" }`。由于该功能仍处于实验阶段，不同发行版本之间的请求和响应结构可能会变化。

---

## 启用摘要功能

本页中的每个路由都会先检查实例是否启用了摘要功能。服务器会读取实例的实验性设置；如果 `enableSummaries` 不是 `true`，就会立即返回：

```json
{ "error": "Summaries are not enabled" }
```

HTTP 状态码为 `404 Not Found`。调用本页任何端点之前，都要先在实例实验性设置中开启摘要功能。这是整个实例范围的开关，而非单个公司的开关。

你还需要为公司配置并启用内置 Summarizer 智能体。否则生成请求无法启动。有关如何配置，请参见下文[请求生成摘要](#request-a-summary)和[内置智能体](./built-in-agents.md)页面。

---

## 槽位的标识方式

每个槽位由三项信息寻址，它们共同构成每个请求的路径和查询参数：

| 部分 | 位置 | 含义 |
|---|---|---|
| `scopeKind` | 路径 | 摘要描述的对象类型。 |
| `scopeId` | 查询字符串（写入时也可放在请求正文） | 对象的具体 UUID。 |
| `slotKey` | 路径 | 对象上的槽位名称。 |

### 范围类型

| `scopeKind` | 描述对象 | 是否需要 `scopeId`？ |
|---|---|---|
| `project` | 单个项目。 | 是，项目 UUID。 |
| `project_workspace` | 单个项目工作区。 | 是，工作区 UUID。 |
| `workspaces_overview` | 公司级工作区总览。 | 否。 |

两种方向的规则都很严格：`project` 或 `project_workspace` 槽位缺少 `scopeId` 时会被拒绝；而 `workspaces_overview` 槽位如果**包含** `scopeId`，也会被拒绝，并返回 `workspaces_overview summary slots must not include scopeId`。这两种情况都会返回 `422 Unprocessable Entity`、正文 `{ "error": "Invalid summary slot selector" }` 和验证详情。

范围目标也必须属于你的公司。将 `project` 或 `project_workspace` 槽位指向其他公司的对象时，会返回 `404 Not Found` 和 `{ "error": "Summary target not found" }`。

### 槽位键

目前只有一个槽位键：`header`，也就是出现在页面顶部的摘要。保留此字段是为了将来添加更多槽位时无需改变这些路由的结构。

### 槽位字段

读取槽位时，响应包含槽位记录本身和它指向的文档。槽位包含以下字段：

| 字段 | 含义 |
|---|---|
| `id` | 槽位 UUID。 |
| `companyId`、`scopeKind`、`scopeId`、`slotKey` | 上文所述的槽位标识信息。 |
| `documentId` | 保存摘要文本的文档；首次写入前为 `null`。 |
| `status` | `idle`、`generating` 或 `failed`。 |
| `failureReason` | `status` 为 `failed` 时的自然语言说明；其他状态下为 `null`。 |
| `generatingIssueId` | 当前正在执行的生成任务；没有任务时为 `null`。 |
| `lastGeneratedAt` | 上次写入摘要的时间。 |
| `lastGeneratedByAgentId` | 撰写摘要的智能体。 |
| `lastModel` | Summarizer 报告本次写入时使用的模型。 |
| `createdAt`、`updatedAt` | 时间戳。 |

### 状态值

| 状态 | 含义 |
|---|---|
| `idle` | 当前没有生成任务。槽位中可能已有摘要，也可能尚未生成摘要。 |
| `generating` | 生成任务正在执行。 |
| `failed` | 生成任务结束，但没有生成摘要。`failureReason` 会说明原因。 |

如果生成任务在从未写入摘要的情况下进入 `done` 或 `cancelled`，槽位就会变为 `failed`。服务器会自动设置 `failureReason`，内容为“摘要生成任务 {label} 在写入摘要前已取消。”或“摘要生成任务 {label} 已结束，但没有写入摘要。”这样槽位就不会永远停留在 `generating` 状态。

---

## 读取摘要槽位

```
GET /api/companies/{companyId}/summary-slots/{scopeKind}/{slotKey}
```

返回槽位当前状态。对于 `project` 和 `project_workspace` 范围，请传入 `?scopeId={uuid}`；对于 `workspaces_overview`，省略此参数。

响应包含三部分：

- `slot`：上文描述的槽位记录；如果从未有人使用此槽位，则为 `null`。
- `document`：摘要文档，包含 `body`（供你呈现的 Markdown）、`title`、`latestRevisionId` 和 `latestRevisionNumber`。尚未写入摘要时为 `null`。
- `generatingIssue`：正在执行的生成任务的精简视图（`id`、`identifier`、`title`、`status`、`assigneeAgentId`）；没有正在运行的任务时为 `null`。

全新槽位不算错误。此时返回 `{ "slot": null, "document": null, "generatingIssue": null }`，你可以呈现空状态并提供“生成”按钮。

### 示例

```bash
curl -sS \
  -H "Authorization: Bearer {token}" \
  "https://paperclip.example.com/api/companies/{companyId}/summary-slots/project/header?scopeId={projectId}"
```

---

## 修订历史

```
GET /api/companies/{companyId}/summary-slots/{scopeKind}/{slotKey}/revisions
```

返回槽位过去生成的摘要，按时间从新到旧排列，最多返回 20 个修订版本。每个版本包含 `revisionNumber`、`title`、`body`、`changeSummary`、`createdAt`，以及负责生成的 `createdByAgentId` 和 `createdByRunId`，便于展示摘要随时间的变化。

槽位已存在但尚无文档时，会返回该槽位和空的 `revisions` 数组。如果槽位从未存在，`slot` 为 `null`，`revisions` 也为空。

---

<a id="request-a-summary"></a>
## 请求生成摘要

```
POST /api/companies/{companyId}/summary-slots/{scopeKind}/{slotKey}/generate
```

此操作相当于“刷新此摘要”按钮。它本身不会写入摘要，而是创建任务、分配给 Summarizer 并唤醒该智能体。Summarizer 写入后，摘要会稍后出现。

**调用权限。** 生成摘要有意设计为人工操作：只有看板运维人员可以触发，智能体不能发起。智能体调用方会收到 `403 Forbidden` 和 `{ "error": "Only board operators can generate summaries." }`。看板运维人员需要 `tasks:assign` 权限（隐式本地看板用户和实例管理员跳过此检查）；没有此权限时，响应为 `403 Forbidden` 和 `{ "error": "Missing permission: tasks:assign" }`。

**Summarizer 必须就绪。** 如果公司尚未配置并启用内置 `summarizer` 智能体，请求会失败，返回 `422 Unprocessable Entity`、`{ "error": "Summarizer built-in agent is not configured" }`，以及一个 `details` 对象，其中包含 `code: "summarizer_not_configured"` 和该内置智能体当前的 `status`。

**请求正文。** 正文可选，唯一接受的字段是 `scopeId`，也可以使用查询字符串传入此值。未知字段会被拒绝。

**响应。**

| 字段 | 含义 |
|---|---|
| `slot` | 槽位；此时 `status: "generating"`，且已设置 `generatingIssueId`。 |
| `generatingIssue` | 新创建或找到的任务（`id`、`identifier`、`title`、`status`、`assigneeAgentId`）。 |
| `alreadyGenerating` | 如果已有生成任务正在执行，且本请求加入了现有任务而非新建任务，则为 `true`。 |

**状态代码。** 新启动的生成请求返回 `202 Accepted`。如果请求被合并到已有运行中，则返回 `200 OK`，因此重复点击“刷新”不会排入两个摘要任务。

服务器会在后台预先生成一份范围快照，其中包含受阻、审核中、进行中和最近完成的任务，并将其放入分配给 Summarizer 的任务描述。这样智能体就能依据范围有限且限定在公司内的视图工作，而无需遍历整个任务列表。

### 示例

```bash
curl -sS -X POST \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/companies/{companyId}/summary-slots/project/header/generate" \
  -d '{ "scopeId": "{projectId}" }'
```

---

## 写入摘要

```
PUT /api/companies/{companyId}/summary-slots/{scopeKind}/{slotKey}
```

Summarizer 会在运行结束时调用此端点。本页记录此端点，是为了说明槽位会发生什么变化；你通常不需要自行调用。**只有 Summarizer 内置智能体**可以写入摘要，而且必须通过为此特定槽位创建的生成任务所对应的运行来写入。

请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `markdown` | 是 | 摘要正文，长度为 1–200,000 个字符。以 `markdown` 格式存储。 |
| `title` | 否 | 最多 200 个字符，或为 `null`。 |
| `changeSummary` | 否 | 附加到新修订版本的备注，最多 1,000 个字符，或为 `null`。 |
| `baseRevisionId` | 否 | 本次写入所基于的修订 ID，用于冲突检测。 |
| `generationIssueId` | schema 中非必填，实际必需 | 本次写入所属的生成任务。 |
| `model` | 否 | 智能体实际使用的模型，最多 200 个字符。保存为槽位的 `lastModel`。 |
| `scopeId` | 否 | 可替代查询字符串中的 `scopeId`。 |

未知字段会被拒绝。

### 写入必须满足的规则

以下任何一项不满足，都会返回 `403 Forbidden`：

- 调用方必须是携带该公司 `summarizer` 内置标记的智能体，否则返回 `{ "error": "Only the Summarizer built-in agent may write summaries" }`。
- 写入必须标明所属任务；缺少 `generationIssueId` 时返回 `{ "error": "Summary writes must identify the active generation task" }`。
- 指定任务必须是槽位当前正在等待的任务；否则返回 `{ "error": "Summary write does not match the active generation task" }`。无法加载任务时返回 `{ "error": "Linked generation task not found" }`。
- 该任务必须针对此特定槽位，否则返回 `{ "error": "Generation task does not target this summary slot" }`。
- 任务必须分配给当前调用智能体，否则返回 `{ "error": "Generation task is not assigned to this agent" }`。
- 写入必须来自该任务的运行，否则返回 `{ "error": "Summary write must run from the linked generation task" }`。

此外还有两种冲突，都会返回 `409 Conflict`：

- 如果有更新的生成任务在此期间接管了槽位，会返回 `{ "error": "Summary generation was superseded by a newer task" }`。
- 如果提供了 `baseRevisionId`，但该 ID 已不再是最新修订版本，会返回 `{ "error": "Summary was updated by someone else" }`，并提供 `currentRevisionId`，供写入方重新定位基准。

### 成功写入的效果

写入会追加新修订版本，而不是覆盖旧文本；随后将槽位状态恢复为 `idle`，清除 `failureReason` 和 `generatingIssueId`，并写入 `lastGeneratedAt`、`lastGeneratedByAgentId` 和 `lastModel`。响应包含已更新的 `slot`、`document` 和新的 `revision`。

---

## 活动日志

两个会更改数据的路由都会在公司活动日志中记录 `entityType: "summary_slot"` 条目：

| 操作 | 记录时机 |
|---|---|
| `summary_slot.generate_requested` | 有人请求生成摘要时。详情包含范围、槽位键、`generatingIssueId` 和 `alreadyGenerating`。 |
| `summary_slot.write` | Summarizer 写入修订版本时。详情包含范围、槽位键、`documentId`、`revisionId` 和 `revisionNumber`。 |

如何读取日志，请参见[活动记录](./activity.md)页面。

---

## 后续阅读

- [内置智能体](./built-in-agents.md)：了解如何配置 Summarizer，以及其生命周期状态的含义。
- [任务](./issues.md)：`POST .../generate` 创建的生成任务属于普通任务。
- [目标与项目](./goals-and-projects.md)：了解摘要槽位可以限定到哪些项目和工作区。
- [API 概览](./overview.md)：查看基础 URL、身份验证、公司范围和通用错误代码表。
