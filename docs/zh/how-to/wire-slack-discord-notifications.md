---
paperclip_version: v2026.1005.0
seo_title: 接入 Slack 或 Discord 通知
seo_description: 将审批请求、受阻的高优先级问题及预算超支情况推送至频道，使董事会无需监控 UI 即可掌握决策动态。
---

# 为审批接入 Slack/Discord 通知

将审批请求、受阻的高优先级问题及预算超支情况推送至 Slack 或 Discord 频道，使董事会无需监控 UI 即可掌握决策动态。两者的配置流程相同：一个 Paperclip 例程按预定计划触发一个通知智能体，该智能体与“最后已见”游标进行差异比对，并向 Webhook URL 发送结构化消息。

从配置到通知生效：约 15 分钟。

---

## 架构

```txt
   ┌──────────────────┐   schedule trigger    ┌────────────────────┐
   │ Paperclip routine│──────(every 60s)────▶│ Notifier agent     │
   └──────────────────┘                       │  ─ poll API        │
                                              │  ─ diff vs cursor  │
                                              │  ─ format payload  │
                                              └─────────┬──────────┘
                                                        │ HTTPS POST
                                                        ▼
                                          ┌─────────────────────────┐
                                          │ Slack/Discord webhook   │
                                          └─────────────────────────┘
```

Paperclip 目前不主动推送出站 Webhook——例程与智能体的组合*本身*就是推送机制。这使得所有活动部件集中在一处：你可以查看通知智能体的运行历史，通过重新运行例程来重放遗漏的事件，并通过更新一个密钥来轮换频道 Webhook。有关底层模型，请参阅 [心跳与例程](../guides/projects-workflow/routines.md)。

> **提示：** 已经通过 **与智能体聊天** 连接了 Slack？该机器人背后的智能体可以在例程中向其启用的频道或向你发送私信，无需使用 Webhook。将目标位置和定时设置写入例程的指令中。每次运行时，系统会在发送前检查例程负责人的关联 Slack 账户及权限。请参阅 [Slack → 稍后发送消息，或按计划发送](../connectors/slack.md#send-messages-later-or-on-a-schedule)。对于 Discord，或者如果你尚未启用实验性聊天连接器，下方的 Webhook 配置方案仍然是更简单的选择。

---

## 值得推送的内容

克制镜像所有事件的冲动。噪音过多的频道在一周内就会被静音。有三类事件值得占用噪音预算：

| 事件 | 重要性 | 检测方法 |
|---|---|---|
| **待处理审批** | 董事会无法对未见之事做出决策。招聘、CEO 战略及 `request_board_approval` 在解决前都会阻塞智能体工作。 | `GET /api/companies/{companyId}/approvals?status=pending` |
| **受阻的高优先级问题** | 当 `critical` 或 `high` 问题状态变为 `blocked` 时，意味着智能体已放弃并需要人工介入。 | `GET /api/companies/{companyId}/issues?status=blocked&priority=critical,high` |
| **预算超支** | 智能体在预算达到 100% 时会自动暂停。你希望在此发生*之前*获知。 | `GET /api/companies/{companyId}/dashboard` 暴露了每个智能体的预算使用情况。 |

其他所有事件（问题创建、评论发布、智能体唤醒）在仪表板中查看更为有用——不要推送它们。

---

## Slack Webhook 配置

Slack 有两种 Webhook 类型：经典入站 Webhook 和由应用管理的 Webhook。请使用由应用管理的流程——这是 Slack 目前仍在开发的唯一类型。

1. 访问 [api.slack.com/apps](https://api.slack.com/apps) 并点击 **创建新应用 → 从零开始**。
2. 将其命名为 `Paperclip Notifications`，选择工作区并创建。
3. 在左侧边栏中，打开 **入站 Webhook** 并开启 **激活入站 Webhook** 开关。
4. 点击 **向工作区添加新 Webhook**，选择频道（例如 `#paperclip-board`）并授权。
5. 复制 Webhook URL。其格式类似 `https://hooks.slack.com/services/T.../B.../xxxxxxxxxxxx`。

该 URL 本身即为承载令牌——任何拥有它的人都可以向该频道发送消息。切勿将其提交到代码库。将其作为 Paperclip 密钥存储在通知智能体的环境中（参见下方的 [签名与密钥](#signing--secrets)）。

---

## Discord Webhook 配置

Discord Webhook 是频道级别的，而非服务器级别的。你需要在目标频道上拥有 **管理频道** 权限。

1. 打开目标频道的设置（齿轮图标 → **编辑频道**）。
2. **集成 → Webhook → 新建 Webhook**。
3. 将其重命名为 `Paperclip`，并可选地上传头像。
4. **复制 Webhook URL**。格式为：`https://discord.com/api/webhooks/<id>/<token>`。

与 Slack 相同的警告：该 URL 包含身份验证令牌。请将其视为密码对待。

---

## 通知例程

雇佣一个小型智能体，其唯一职责是分发通知。任何具备代码能力的适配器均可使用（`claude_local`、`codex_local` 或自定义 HTTP Webhook）。关键在于它必须能够读取三个环境变量并向两个 URL 发送 POST 请求。

创建一个按计划触发并分配给通知智能体的例程：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Notify board channel",
    "description": "Diff approvals and blocked issues since last run; post to Slack and Discord.",
    "assigneeAgentId": "<notifier-agent-id>",
    "priority": "low",
    "concurrencyPolicy": "skip_if_active",
    "catchUpPolicy": "skip_missed"
  }'
```

然后附加一个 1 分钟的计划触发器：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/routines/<routine-id>/triggers" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "schedule",
    "label": "Every minute",
    "enabled": true,
    "cronExpression": "* * * * *",
    "timezone": "UTC"
  }'
```

`skip_if_active` 加上 `skip_missed` 在此处是合适的组合：如果上一次运行仍在进行中，则跳过本次触发而非堆叠重复项，且在重启后不要尝试补发遗漏的分钟数。

---

## 通知智能体的行为

按顺序执行三件事。其结构简短到足以放入智能体的指令中：

```txt
1. Read PAPERCLIP_NOTIFIER_LAST_SEEN_AT from a tiny KV store
   (or a comment on a "state" issue you own — anything durable).
2. Fetch:
   - GET /api/companies/{COMPANY_ID}/approvals?status=pending
   - GET /api/companies/{COMPANY_ID}/issues?status=blocked&priority=critical,high
   Drop anything with updatedAt <= last-seen-at.
3. For each new event, POST a message to SLACK_WEBHOOK_URL
   and DISCORD_WEBHOOK_URL. On success, write the new max
   updatedAt back to last-seen-at.
```

游标是防止 60 秒例程永远重复发布同一审批请求的关键。如果你使用的是自定义适配器，请将游标持久化存储在你自己的存储中；如果你使用的是 Claude Code 或其他本地适配器，请将其作为 Markdown 注释写入一个专用的 `notifier-state` 问题中，并在下次唤醒时读取。

---

## 消息格式

有用的通知应指明事件、请求者及操作——并提供通往决策页面的一键路径。没有上下文的裸链接是导致频道被静音的原因。

### Slack (Block Kit)

```json
{
  "blocks": [
    {
      "type": "header",
      "text": { "type": "plain_text", "text": "Approval pending: Hire CTO" }
    },
    {
      "type": "section",
      "fields": [
        { "type": "mrkdwn", "text": "*Type*\nhire_agent" },
        { "type": "mrkdwn", "text": "*Requested by*\nCEO" },
        { "type": "mrkdwn", "text": "*Budget*\n$200/mo" },
        { "type": "mrkdwn", "text": "*Adapter*\nclaude_local" }
      ]
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "Need a CTO to own backend platform decisions and review hires below the C-level."
      }
    },
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": { "type": "plain_text", "text": "Review approval" },
          "url": "https://paperclip.example.com/PAP/approvals/<approval-id>",
          "style": "primary"
        }
      ]
    }
  ]
}
```

### Discord (嵌入消息)

```json
{
  "username": "Paperclip",
  "embeds": [
    {
      "title": "Approval pending: Hire CTO",
      "url": "https://paperclip.example.com/PAP/approvals/<approval-id>",
      "color": 2278750,
      "description": "Need a CTO to own backend platform decisions and review hires below the C-level.",
      "fields": [
        { "name": "Type", "value": "hire_agent", "inline": true },
        { "name": "Requested by", "value": "CEO", "inline": true },
        { "name": "Budget", "value": "$200/mo", "inline": true },
        { "name": "Adapter", "value": "claude_local", "inline": true }
      ],
      "footer": { "text": "Paperclip • approvals" }
    }
  ]
}
```

对于受阻问题警报，将标题替换为 `"Blocked: PAP-142 — Migrate billing to Stripe"`，将字段替换为优先级 + 负责人 + 最后评论作者，并将按钮指向 `/PAP/issues/PAP-142`。对于预算超支，标题设为 `"Budget at 92%: backend-engineer"`，字段包含月度上限、本月至今支出，并链接至 `/PAP/agents/backend-engineer/runs`。

---

<a id="signing--secrets"></a>
## 签名与密钥

Slack 和 Discord 的 Webhook URL 是承载令牌——拥有即代表授权。任何拥有该 URL 的人都可以以 `Paperclip` 的身份向频道发送消息。两条规则可覆盖安全态势：

1. **切勿提交 URL。** 将其存储在通知智能体的环境中——对于 `claude_local`，这是智能体配置中的每个智能体环境变量；对于 `http_webhook`，则在接收服务上设置。参见 [环境变量](../reference/deploy/environment-variables.md)。
2. **泄露时轮换。** 在 Slack 中：从应用的入站 Webhook 页面重新生成 Webhook（旧 URL 会立即失效）。在 Discord 中：打开 Webhook 设置并点击 **复制 Webhook URL → 重新生成**。

如果你还想将 Webhook *接收* 到 Paperclip 中（Stripe → Paperclip 例程，GitHub → Paperclip 例程），那是相反的方向，使用 Paperclip 的签名触发模型，支持 `bearer`、`hmac_sha256` 或 `github_hmac` 模式。详见 [例程 API → Webhook 触发器](../reference/api/routines.md#webhook)。

---

<a id="testing-the-loop"></a>
## 测试循环

在将例程设为 `active` 之前，先从你的笔记本电脑对 Webhook 进行试运行：

```bash
curl -X POST "$SLACK_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"text":"Paperclip notifier — wiring check"}'

curl -X POST "$DISCORD_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"content":"Paperclip notifier — wiring check"}'
```

`200 OK` 加上频道中的消息意味着 URL 有效。如果你收到 `400`，则 JSON 格式错误。如果你收到 `403`，则 URL 已被轮换或撤销。

然后手动运行一次例程：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/routines/<routine-id>/run" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "source": "manual" }'
```

从例程详情页面的 **运行** 选项卡中检查生成的问题。首次运行将为每个当前待处理的审批和当前受阻的高优先级问题各发布一条消息，然后写入其游标。下一分钟的触发应为无操作。

---

## 常见故障模式

- **同一审批每分钟重复发布。** 游标未被持久化。在每次运行的顶部打印 `last-seen-at` 并确认其已推进。
- **Slack 返回 `invalid_blocks`。** Block Kit 很严格——不允许未知字段，不允许空的 `fields` 数组，按钮 URL 必须是 HTTPS。使用 [Block Kit Builder](https://app.slack.com/block-kit-builder) 进行验证。
- **Discord 返回 `429 rate limited`。** 你超过了 Discord 每个 Webhook 的限制（约 5 次/2 秒）。改为将多条 `embeds[]` 条目批量放入一条消息的单个嵌入中，而不是发送 N 条独立消息。
- **例程触发但未发布任何内容。** 检查通知智能体的运行历史——`failed` 运行包含异常信息。最常见的原因是智能体缺少环境变量。

---

## 另请参阅

- [审批（董事会指南）](../guides/day-to-day/approvals.md) — 每种审批类型在 UI 中的外观。
- [心跳与例程](../guides/projects-workflow/routines.md) — 并发与补发策略的深入介绍。
- [审批 API](../reference/api/approvals.md) — 通知智能体读取的端点。
- [例程 API](../reference/api/routines.md) — 计划 Cron 和 Webhook 触发器参考。
- [调试卡住的心跳](./debug-stuck-heartbeat.md) — 如果通知停止触发，这是首先应查看的地方。
