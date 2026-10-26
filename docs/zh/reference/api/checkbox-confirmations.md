---
paperclip_version: v2026.1012.0
seo_title: 复选框确认 API
seo_description: 一种任务讨论串交互：要求人员从已知选项列表中勾选任意子集，然后通过一次操作确认或拒绝整个请求。
---

# 复选框确认

复选框确认是一种任务讨论串交互，要求看板用户或其他用户从已知选项列表中选择任意子集，然后确认或拒绝整个请求。它会在任务讨论串中显示为卡片，响应则以结构化数据返回：包括已接受的选项 ID，或附带可选原因的拒绝结果。

使用下方交互路由并设置 `kind: request_checkbox_confirmation` 来创建。API、CLI、插件辅助函数和 UI 使用相同的正文验证规则。此功能自 v2026.609.0 起提供。

![任务讨论串中的复选框确认卡片：包含标题和提示、五个选项且其中三项已勾选、折叠的详情区域、选择摘要，以及自定义的接受和拒绝按钮。](../../../user-guides/screenshots/light/interactions/checkbox-confirmation-card.png)

所有路由都挂载在 `/api` 下，并使用 `Authorization: Bearer {token}` 进行身份验证。

| 方法 | 路径 | 用途 |
|---|---|---|
| POST | `/api/issues/{issueId}/interactions` | 创建交互。 |
| GET | `/api/issues/{issueId}/interactions` | 列出任务的交互。 |
| POST | `/api/issues/{issueId}/interactions/{interactionId}/accept` | 接受交互，并提交选中的选项 ID。 |
| POST | `/api/issues/{issueId}/interactions/{interactionId}/reject` | 拒绝交互，可附带原因。 |

交互的 `respond` 路由用于 `ask_user_questions`；复选框确认只能通过 `accept` 或 `reject` 解决。解决后，交互会被封存，后续响应将被拒绝。

---

## 创建复选框确认

```
POST /api/issues/{issueId}/interactions
```

请求正文：

- `kind`：`request_checkbox_confirmation`。
- `payload`：复选框正文，完整 schema 见下文。
- `title`：在任务讨论串卡片上显示的标题。
- `summary`：显示在标题下方的一行摘要。
- `idempotencyKey`：可选，但建议设置。参见[幂等性](#idempotency)。
- `continuationPolicy`：`wake_assignee`、`wake_assignee_on_accept` 或 `none`。此类型默认为 `wake_assignee`。参见[后续处理策略](#continuation-policy)。

权限：智能体可以在分配给自己的任务上，或自己曾发表评论的任务上创建交互；看板用户可以在其所属公司中的任何任务上创建交互。

### 示例

```bash
curl -sS -X POST \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/issues/{issueId}/interactions" \
  -d '{
    "kind": "request_checkbox_confirmation",
    "idempotencyKey": "checkbox:{issueId}:cleanup-files:{planRevisionId}",
    "title": "Confirm files to delete",
    "summary": "Pick the files you want removed before I run the cleanup.",
    "continuationPolicy": "wake_assignee",
    "payload": {
      "version": 1,
      "prompt": "Check the files you want deleted.",
      "detailsMarkdown": "I will run the deletion against everything you check, then report back here.",
      "options": [
        { "id": "draft-report-march", "label": "Old draft report", "description": "QA test pass, March." },
        { "id": "tmp-export-2025", "label": "tmp/export-2025.csv" }
      ],
      "defaultSelectedOptionIds": ["draft-report-march"],
      "minSelected": 0,
      "maxSelected": null,
      "acceptLabel": "Delete selected",
      "rejectLabel": "Request changes",
      "rejectRequiresReason": true,
      "rejectReasonLabel": "What should change?",
      "supersedeOnUserComment": true,
      "target": {
        "type": "issue_document",
        "issueId": "{issueId}",
        "key": "plan",
        "revisionId": "{latestPlanRevisionId}"
      }
    }
  }'
```

---

## 正文 schema

`payload` 字段（`RequestCheckboxConfirmationPayload`）包含：

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `version` | 字面值 `1` | 是 | — | Schema 版本，必须传入 `1`。 |
| `prompt` | 字符串，1–1000 个字符 | 是 | — | 显示在复选框列表上方的提示标题。 |
| `detailsMarkdown` | 字符串（最多 20,000 个字符）或 `null` | 否 | `null` | 显示在选项列表上方的 Markdown 背景说明。 |
| `options` | 选项对象数组，1–200 项 | 是 | — | 可选择的选项列表；对象字段见下文。 |
| `defaultSelectedOptionIds` | 字符串数组 | 否 | `[]` | UI 中默认勾选的选项 ID。每个 ID 必须对应 `options` 中的一项；如果设置了 `maxSelected`，数组长度不能超过此值。 |
| `minSelected` | 大于或等于 0 的整数 | 否 | `0` | 最少选择数量。若最终选择的 ID 数少于此值，服务器会拒绝接受请求。此值不能大于选项数量。 |
| `maxSelected` | 大于或等于 0 的整数，或 `null` | 否 | `null`（无上限） | 最多选择数量。设置后，必须大于或等于 `minSelected`，且不能大于选项数量。 |
| `acceptLabel` | 1–80 个字符的字符串，或 `null` | 否 | `null`（使用 UI 默认值） | 接受按钮的标签。 |
| `rejectLabel` | 1–80 个字符的字符串，或 `null` | 否 | `null`（使用 UI 默认值） | 拒绝按钮的标签。 |
| `rejectRequiresReason` | 布尔值 | 否 | `false` | 设为 `true` 时，拒绝操作必须提供非空 `reason`，否则返回 `422`。 |
| `rejectReasonLabel` | 1–160 个字符的字符串，或 `null` | 否 | `null` | 拒绝原因输入框的字段标签。 |
| `allowDeclineReason` | 布尔值 | 否 | `true` | 是否显示原因输入框。 |
| `declineReasonPlaceholder` | 1–240 个字符的字符串，或 `null` | 否 | `null` | 原因输入框中的占位文字。 |
| `supersedeOnUserComment` | 布尔值 | 否 | `false`（由服务器设置） | 设为 `true` 时，之后的看板或用户评论会以 `outcome: "superseded_by_comment"` 解决仍待处理的交互。保持关闭时，即使讨论串中有人继续发言，卡片仍保持待处理状态。 |
| `target` | 目标对象或 `null` | 否 | `null` | 将请求绑定到任务文档修订版本。参见[目标绑定与过期](#target-binding-and-staleness)。 |

选项对象包含：

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `id` | 1–120 个字符的字符串 | 是 | — | 稳定标识符，在正文内唯一。接受后会出现在 `selectedOptionIds` 中。 |
| `label` | 1–120 个字符的字符串 | 是 | — | 复选框旁边显示的选项文本。 |
| `description` | 最多 500 个字符的字符串 | 否 | — | 显示在选项标签下方的补充说明。 |

---

## 接受

```
POST /api/issues/{issueId}/interactions/{interactionId}/accept
```

```json
{ "selectedOptionIds": ["draft-report-march", "tmp-export-2025"] }
```

行为：

- 接受操作要求调用方是看板用户或其他用户。创建该交互的智能体不能接受它。
- 如果省略 `selectedOptionIds`，服务器会回退到正文的 `defaultSelectedOptionIds`。
- 服务器会确认每个 ID 都引用已知选项、去除重复项，并强制执行 `minSelected` 和 `maxSelected` 限制。未知 ID 会导致 `422`。

## 拒绝

```
POST /api/issues/{issueId}/interactions/{interactionId}/reject
```

```json
{ "reason": "Keep the March draft; only delete tmp/export-2025.csv." }
```

如果正文设置了 `rejectRequiresReason: true`，则 `reason` 为必填项；缺少原因或原因为空时会返回 `422`。否则此字段可选。

---

## 结果和结束方式

已解决的交互会携带 `result`（`RequestCheckboxConfirmationResult`）。接受时：

```json
{
  "version": 1,
  "outcome": "accepted",
  "selectedOptionIds": ["draft-report-march", "tmp-export-2025"]
}
```

| `outcome` | 结果字段 | 触发条件 |
|---|---|---|
| `accepted` | `selectedOptionIds`（`minSelected` 为 `0` 时可以为空） | 看板用户或其他用户接受了交互。 |
| `rejected` | `reason`、`commentId`；不包含 `selectedOptionIds` | 看板用户或其他用户拒绝了交互。 |
| `superseded_by_comment` | `commentId` | 交互待处理期间，看板用户或其他用户评论了任务，并且 `supersedeOnUserComment` 为 `true`。 |
| `stale_target` | `staleTarget` | 目标任务文档修订版本已不是最新版本。 |

---

<a id="continuation-policy"></a>
## 后续处理策略

交互的 `continuationPolicy` 控制交互解决后是否唤醒任务负责人：

- `wake_assignee`（此类型的默认值）：看板用户或其他用户解决交互后唤醒负责人，并传递上方的 `result`。
- `wake_assignee_on_accept`：只在接受时唤醒，拒绝时不唤醒。
- `none`：从不唤醒负责人。只有在决策后无需继续任务时才使用。

待处理交互是一种明确的等待流程：创建交互后，来源任务通常会转为 `in_review`，并附加一条评论，说明看板需要作出什么决定。

<a id="target-binding-and-staleness"></a>
## 目标绑定与过期

`target` 复用 `request_confirmation` 的目标 schema，通常如下：

```json
{
  "type": "issue_document",
  "issueId": "{issueId}",
  "key": "plan",
  "revisionId": "{latestRevisionId}"
}
```

也可以包含 `documentId` 和 `revisionNumber`。目标文档出现更新修订版本后，待处理交互会以 `outcome: "stale_target"` 过期；请基于最新修订版本重新创建交互。

<a id="idempotency"></a>
## 幂等性

请使用确定性的 `idempotencyKey`，例如 `checkbox:{issueId}:{decisionKey}:{revisionId}`。临时错误后重试时，这样会复用同一张卡片，而不会堆叠重复项。发生 `stale_target` 或 `superseded_by_comment` 后重新创建时，请在键中加入新的修订 ID。

---

## 如何选择交互类型

复选框确认是四种任务讨论串交互之一：

| 类型 | 适用场景 | 不适用于 |
|---|---|---|
| `request_checkbox_confirmation` | 响应者需要从已知列表（最多 200 项）中选择任意子集，然后接受或拒绝。 | 单纯的是/否决策、自由格式回答或提出新任务。 |
| `request_confirmation` | 与某个目标绑定的单个是/否决策，例如接受计划修订版本。 | 多选项选择。 |
| `ask_user_questions` | 简短结构化表单，包含少量带类型的问题、选项或文本回答。 | 从一个很长的列表中选择许多项目。 |
| `suggest_tasks` | 提出具体任务；接受的项目会成为真正的子任务。 | 由智能体自行执行的选择项。 |

复选框确认的区别在于：它是确认，而不是提问。响应者接受或拒绝整个交互；被选中的 ID 会随接受请求一并提交。

共享交互信封和其他类型见 [Issues API](./issues.md)。
