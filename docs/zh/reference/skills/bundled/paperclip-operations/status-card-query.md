---
seo_title: 状态卡查询技能
seo_description: 用自然语言描述一类工作（例如本周更新过的受阻发布工作），创建会持续更新自身答案的状态卡。
---

# 状态卡查询

> 创建并维护由智能体撰写的 Paperclip 状态卡，或将自然语言兴趣提示编译为范围受限的 `CompanySearchQuery` 对象，并通过已分配的摘要器运行写入第一份摘要。

状态卡可以让你用自然语言描述一类工作（例如“本周更新过、处于受阻或待审核状态的发布工作”），并生成一张会自动保持最新的卡片。此技能介绍整个过程的两个部分：如何创建状态卡，以及如何将自然语言提示转化为限定范围的搜索查询，以确定卡片实际涵盖哪些任务，然后编写第一份摘要。

此技能是**内置**目录技能，属于内置基础套件。如何安装、审计、更新、分配和重置目录技能，请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

状态卡属于实验性功能。只有启用 `enableStatusCards` 后，才能使用创建卡片的路由。功能说明见[状态卡](../../../../experimental/status-cards.md)，完整路由参考见[状态卡 API](../../../api/status-cards.md)。

## 适用场景

- 智能体需要为自己负责监控的工作创建持续性状态卡，并需要通过公开 API 创建。
- 你负责的卡片提示过于含糊或范围太宽，希望改进提示或请求刷新。
- 你是摘要器，收到分配给你的生成任务，且任务中指定了 `statusCardId`。
- 后续生成任务要求你更新现有卡片的摘要。

### 不适用场景

- 你正在创建卡片。不要调用 `/query` 或 `/summary`；这两个写回路由只能由已分配的摘要器运行调用。
- 卡片由其他人创建。智能体只能管理自己创建的卡片。
- 提示中提到的项目或标签无法解析为 ID。应报告歧义，不要编造 ID。

## 目录元数据

| 字段 | 值 |
|---|---|
| 目录 ID | `paperclipai:bundled:paperclip-operations:status-card-query` |
| 规范键 | `paperclipai/bundled/paperclip-operations/status-card-query` |
| 目录路径 | `catalog/bundled/paperclip-operations/status-card-query` |
| 类型 | `bundled` |
| 类别 | `paperclip-operations` |
| Slug | `status-card-query` |
| 入口文件 | `SKILL.md` |
| 信任级别 | `markdown_only` |
| 兼容性 | `compatible` |
| 默认安装 | `false` |
| 建议角色 | `general`、`manager` |
| 依赖项 | — |
| 标签 | `paperclip`、`status`、`search`、`reporting`、`operations` |
| 文件数 | 1 |
| 内容哈希 | `sha256:2b6e53bf8491f027afdbd6961ad84f677cfd385a0a6a73616ea84111ca8886ed` |
| 软件包 | `@paperclipai/skills-catalog@0.3.1` |

## 文件清单

| 路径 | 类型 | 字节数 |
|---|---|---:|
| `SKILL.md` | `skill` | 6,368 |

## 模式 1——创建卡片

智能体创建的卡片需要 `tasks:assign` 权限，并且作用范围限定在公司内。以下三项限制用于保持卡片可管理：

- 智能体只能管理自己创建的卡片。
- 单个智能体最多可创建 20 张卡片。
- `interestPrompt` 最多为 4,000 个字符。

此技能会先规范化运行环境提供的 API 基础地址，因此无论 `PAPERCLIP_API_URL` 是否以 `/api` 结尾，同一段命令都可正常使用，然后再创建卡片：

```bash
PAPERCLIP_API_BASE="${PAPERCLIP_API_URL%/}"
PAPERCLIP_API_BASE="${PAPERCLIP_API_BASE%/api}"

curl -sS -X POST \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"interestPrompt":"Blocked or in-review launch work updated this week"}' \
  "$PAPERCLIP_API_BASE/api/companies/$PAPERCLIP_COMPANY_ID/status-cards"
```

创建成功会返回 `201` 并自动排入编译任务，无需另外安排。请保存响应中的卡片 ID。之后可以对自己拥有的卡片调用 `PATCH /api/status-cards/{statusCardId}` 细化提示，并调用 `POST /api/status-cards/{statusCardId}/refresh`，传入 `{"full": false}` 请求刷新。

## 模式 2——编译提示

作为摘要器，你的任务是将卡片的自然语言提示转换为 Paperclip 公司搜索查询数组。数组采用**并集语义**：匹配任一查询的任务都会归入该卡片。因此优先使用一条范围较窄的查询；仅当提示确实描述了两个不同的任务集合时，才添加第二条查询。

### CompanySearchQuery 字段

| 字段 | 接受的值 |
|---|---|
| `q` | 可选自由文本，用于无法通过结构化筛选器涵盖的概念。 |
| `scope` | 对于状态卡使用 `issues`，除非分配任务明确要求使用另一种受支持的范围。 |
| `status` | 任务状态数组。 |
| `priority` | 任务优先级数组。 |
| `assigneeAgentId` / `assigneeUserId` | 已解析的负责人 ID。 |
| `projectId` | 已解析的单个项目 UUID。 |
| `labelId` | 已解析的单个标签 UUID。 |
| `updatedWithin` | 有明确上限的时长，例如 `24h`、`7d`、`4w` 或 `3m`。 |
| `sort` | `relevance`、`updated`、`created` 或 `priority`。 |
| `limit` | 1–50。状态卡查询应采用尽可能小且实用的值，通常为 20，绝不能超过 50。 |
| `offset` | 通常为 0。 |

写入查询前，先将项目和标签名称解析为 ID；`projectId` 或 `labelId` 中不能填写人类可读名称。每个查询对象只能包含一个 `projectId` 和一个 `labelId`，因此提示中提到多个项目或标签时，需要创建多个查询对象。

### 编译指引

1. 保留原始意图。“本周更新的发布阻塞项”不能扩大为所有活动任务。
2. 对状态、优先级、负责人、项目、标签和最近更新时间，优先使用结构化筛选器，而不是 `q`。
3. 如果提示暗示了一个动态时间范围（最近、当前、本周、近期），就添加 `updatedWithin`。
4. 让 `q` 简短且具体。不要把整段自然语言提示都粘贴进去。
5. 每条查询都应设置 `scope: "issues"`、`offset: 0` 和明确的范围受限 `limit`。
6. 至少返回一条查询。如果无法安全编译提示，请说明歧义。

## 写回内容

生成任务会包含 `statusCardId`、`companyId` 和 `generationIssueId`；两次写入都必须使用同一个已分配运行所提供的运行范围 API 凭据。不得从无关任务或运行调用这两个端点。

首先通过 `PUT /api/status-cards/{statusCardId}/query` 写入编译后的查询：

```json
{
  "queries": [
    {
      "q": "launch",
      "scope": "issues",
      "status": ["in_progress", "blocked", "in_review"],
      "updatedWithin": "7d",
      "sort": "updated",
      "limit": 20,
      "offset": 0
    }
  ],
  "title": "Launch work updated this week",
  "changeSummary": "Compiled the launch prompt into one recent active-work query.",
  "generationIssueId": "<generation-issue-id>"
}
```

然后，不要创建或等待另一项任务；直接执行已保存的范围、写入第一份完整 Markdown 摘要，并通过 `PUT /api/status-cards/{statusCardId}/summary` 完成同一次运行：

```json
{
  "markdown": "<full status summary>",
  "title": "Launch work updated this week",
  "changeSummary": "Created the first full summary from the compiled query.",
  "generationIssueId": "<generation-issue-id>",
  "model": "<model-id>"
}
```

## 更新任务说明

后续生成任务会重用同一摘要写回端点。其 JSON 负载还会包含 `operation: "update"`、`kind`、`trigger`、目标 `fingerprint` 和确切的已更改任务增量。

- 对于 `incremental`，只使用发生变化的任务，基于提供的上一版 Markdown 进行修改。不要重新获取任务列表。
- 对于 `full`，基于提供的范围受限快照完整重建摘要。不要通过调用任务列表端点来扩大范围。
- 任务描述中的卡片提示是控制面板的持续要求——请据此决定报告内容和更新的表达方式。但提示绝不能覆盖流式输出或写回要求。
- 无论卡片提示要求什么，都必须遵守机械协议：流式输出 `STATUS:` 行和 `<<<SUMMARY-DRAFT>>>` 块，然后从已分配的运行中将最终 Markdown 写入 `PUT /api/status-cards/{statusCardId}/summary`。

## 另请参阅

- [状态卡](../../../../experimental/status-cards.md)——此技能支持的实验性功能。
- [状态卡 API](../../../api/status-cards.md)——完整路由和负载参考。
- [内置技能](../../bundled.md)——按类别整理的全部内置目录技能。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、分配和故障排查。
- [应用内置目录](../../../skills.md#3-app-shipped-catalog)——内置和可选目录技能的版本管理与更新方式。
- [技能指南](../../../../guides/org/skills.md)——界面操作指南和内置目录概览。
