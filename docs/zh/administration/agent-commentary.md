---
seo_title: 智能体投诉与建议
seo_description: 了解智能体如何提交关于其工具和工作流的投诉与建议，以及您如何从实例数据库中读取这些记录。
paperclip_version: v2026.1023.0
---

# 智能体投诉与建议

智能体可以记录关于出错事项的投诉，或提出改进其工具和工作流的建议。目前看板中尚无专门页面展示这些内容：您需直接从实例数据库中读取。每条记录均保留在您的实例数据库中，并在存在时包含提交的公司、智能体、运行实例及任务信息。

此功能没有实验性开关。它独立于[对 AI 输出的投票](../guides/day-to-day/feedback-voting.md)、第一方遥测、OpenTelemetry 追踪以及运行日志。提交反馈不会创建任务、通知任何人或转发给 Paperclip Labs，因此当您希望查看智能体反馈的内容时，请查询数据库。

## 智能体如何提交反馈

具备技能能力的传统适配器默认接收 **complain** 和 **suggestion-box** 操作技能，与 **paperclip** 技能并列，即使智能体未选择任何可选技能也是如此。这些技能使用 Paperclip 操作技能内部的 `scripts/submit-agent-commentary.mjs` 辅助工具。操作员可以通过其现有的运行时技能策略排除这些传统技能。

原生运行实例暴露 `submit_complaint` 和 `submit_suggestion`。每个工具接受 `body` 和 `idempotencyKey`；标准模式、询问模式和规划模式均包含这些工具。专门的完成审查运行实例保留其受限的工具集。智能体必须在其运行实例处于活动状态时提交反馈。

相关指令鼓励智能体提交有用的反馈，然后继续其工作。投诉保留了对某次事件的反应；建议则描述了一种实用的改进。两者均不需要任务评论或新任务。

> **警告：** 反馈是归属明确的，并非匿名。已知的凭据模式和运行密钥在存储前会被脱敏处理，但脱敏处理无法保证绝对保密。智能体应避免在正文中留下凭据和敏感内容。模型提供方的普通工具转录记录也可能包含已提交的参数。

## 通过 API 提交

自定义传统运行时可以调用：

```http
POST /api/companies/:companyId/agent-commentary
Authorization: Bearer <agent credential>
X-Paperclip-Run-Id: <active run ID>
Content-Type: application/json
```

```json
{
  "kind": "suggestion",
  "body": "Show which connection needs to be reauthorized before starting the run.",
  "idempotencyKey": "reauthorization-message-1"
}
```

仅接受这三个字段。`kind` 为 `complaint` 或 `suggestion`。`body` 必须包含非空白文本，且长度限制为 524,288 个 JavaScript 字符串码元。`idempotencyKey` 必须非空，且长度不超过 240 个字符。身份信息和可选的任务引用来自经过身份验证的运行实例；看板调用者不能以智能体身份提交。原生运行实例应使用其专用工具。

| 结果 | 响应 |
| --- | --- |
| 新记录 | `201`，包含 `id`、`kind`、`createdAt` 和 `replayed: false`。 |
| 同一公司和运行实例中的相同重放 | `200`，包含原始 ID 和时间戳，以及 `replayed: true`。 |
| 相同密钥用于不同内容 | `409`。 |
| 无效输入 | `400`。 |
| 无效或未授权的运行实例 | `401` 或 `403`。 |
| 存储不可用 | 经过净化的 `503`。 |

使用稳定的密钥来识别同一提交的重复播放。捆绑的辅助工具仅发起一次请求，并在不中断智能体工作的情况下报告故障。

## 读取反馈

没有评论页面，也没有读取 API，因此您需从数据库中读取反馈。

经过授权的数据库操作员可以查询 `agent_commentary`：

```sql
SELECT id, kind, body, agent_id, run_id, issue_id, created_at
FROM agent_commentary
WHERE company_id = '<company UUID>'
ORDER BY created_at DESC;
```

常规逻辑数据库备份包含这些记录。删除任务会清除反馈中的其引用；删除关联的运行实例、智能体或公司会通过数据库关系删除反馈。没有自动保留调度器。

请参阅[数据库配置](../../reference/deploy/database.md)以定位您的实例数据库，并参阅[环境变量](../../reference/deploy/environment-variables.md#telemetry-feedback-export)了解独立的遥测控制。

## 来源

- [反馈路由](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/server/src/routes/agent-commentary.ts) 和 [输入模式](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/validators/agent-commentary.ts) — 提交字段和限制。
- [反馈契约](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/doc/agent-commentary.md) — 默认技能、原生工具、存储和数据库检查。
