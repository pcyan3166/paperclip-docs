---
paperclip_version: v2026.1005.0
seo_title: 创建每日运行的例程
seo_description: 只需描述一次重复性工作，附加一个 cron 触发器，Paperclip 将在每次触发时生成一个新的工单，并分配给相同的负责人、父级、项目和目标。
---

# 创建每日运行的例程

[例程](../guides/welcome/glossary.md) 是一个重复性任务生成器。你只需描述一次工作内容，附加一个 cron 触发器，Paperclip 就会在每次触发时生成一个新的执行工单——分配给同一个智能体，每次都具有相同的父级、项目和目标。智能体通过其常规心跳机制领取该工单，执行工作，运行记录将显示在例程的历史记录中。

本指南涵盖三种初始模式——每日站会、收件箱分类和部署检查——以及一旦基础配置完成后你会用到的 Webhook 和手动触发变体。

首次计划运行所需时间：约 10 分钟。

---

## 所需条件

- 在 `PAPERCLIP_API_KEY` 中配置一个看板 API 令牌，并在 `PAPERCLIP_API_URL` 中配置 URL（[CLI 认证](../../administration/cli-auth.md)）。
- 你正在使用的 `companyId`（见下文 `COMPANY_ID`）。
- 应执行每次触发的智能体的 id（`AGENT_ID`）。智能体只能创建分配给自己的例程；看板调用者可以指定任何智能体。
- 一个 `projectId` 用于锚定工作——每个例程都是项目范围的，运行记录会继承它。

如果你更倾向于通过 UI 点击操作，相同的字段存在于例程页面的编辑器中，[例程指南](../guides/projects-workflow/routines.md) 会引导你完成各个界面。以下内容均为 API 路径，以便你可以通过脚本进行配置。

---

## 1. 创建例程

```bash
ROUTINE_ID=$(curl -sS -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Daily standup",
    "description": "Summarise yesterday'"'"'s completed issues and the day'"'"'s top three priorities. Post the summary as a comment on the project.",
    "assigneeAgentId": "'$AGENT_ID'",
    "projectId": "'$PROJECT_ID'",
    "priority": "medium",
    "concurrencyPolicy": "skip_if_active",
    "catchUpPolicy": "skip_missed"
  }' | python3 -c 'import json,sys; print(json.load(sys.stdin)["id"])')
```

由于该例程有负责人，因此它被创建为 `active`。如果没有负责人，它将回退到 `paused`，直到你分配某人——这在智能体存在之前预置例程时很有用，但在此处用处不大。

`title` 和 `description` 是模板：你写入的内容即为每次运行的执行工单内容。如果你需要按运行替换变量（`{{customer}}`、`{{date}}` 等），请在例程上声明变量——参见 [例程指南中的变量模板](../guides/projects-workflow/routines.md#variable-templates)。

---

## 2. 附加每日计划触发器

```bash
curl -X POST "$PAPERCLIP_API_URL/api/routines/$ROUTINE_ID/triggers" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "schedule",
    "label": "Every weekday 9am",
    "enabled": true,
    "cronExpression": "0 9 * * 1-5",
    "timezone": "Europe/Amsterdam"
  }'
```

Cron 采用标准的 5 字段语法。以下五个值得记忆：

| Cron | 触发时间 |
|---|---|
| `0 9 * * 1-5` | 每个工作日 09:00 |
| `0 9 * * *` | 每天 09:00 |
| `*/15 * * * *` | 每 15 分钟 |
| `0 */4 * * *` | 每 4 小时，整点触发 |
| `0 0 1 * *` | 每月 1 日 00:00 |

`timezone` 是一个 IANA 名称——`UTC`、`America/New_York`、`Asia/Tokyo`。Paperclip 在该时区评估 cron，因此 9 点的例程在夏令时切换期间仍保持当地 9 点，而不会漂移一小时。例程详情页上的 `Next:` 倒计时是根据此组合在服务端计算的，而非基于浏览器时钟。

一个例程可以有多个触发器。使用相同的 `POST` 路由，但请求体不同——重叠是可以的，所有触发器独立触发。

---

## 3. 选择并发和追赶策略

默认设置适用于大多数例程，但错误的组合要么会淹没智能体，要么会静默丢弃工作。了解每个策略的作用可以节省日后的调试时间。

**并发策略** —— 当上一次运行的工单仍处于打开状态时，新触发器触发的行为。

| 策略 | 适用场景 |
|---|---|
| `coalesce_if_active` *(默认)* | 每次运行产生的工作应合并到正在进行的运行中——日常维护、轮询。新运行链接到现有工单，不创建重复项。 |
| `skip_if_active` | 每次运行是“最新快照”，过时的运行毫无价值——收件箱分类、状态扫描。新运行被丢弃，而非排队。 |
| `always_enqueue` | 每次运行产生独特的产物，不能丢失——账单结算、每小时快照。即使前一个工单仍打开，也会始终创建新工单。请谨慎使用：这是导致积压堆积的策略。 |

**追赶策略** —— 错过的计划触发器（服务器停机、例程暂停等）的处理方式。

| 策略 | 适用场景 |
|---|---|
| `skip_missed` *(默认)* | 错过的触发器被丢弃。下一次正常触发是下一次触发时间。 |
| `enqueue_missed_with_cap` | 错过的触发器被排队，上限为 25 个。仅当每次运行独立有价值且你希望追赶而非跳过时使用。周末停机配合一分钟计划会立即达到上限并丢弃其余部分——这是有意为之。 |

在创建时设置两者（步骤 1 中的示例选择 `skip_if_active` + `skip_missed`），或稍后使用 `PATCH /api/routines/{routineId}` 更改。

---

## 4. （可选）为例程提供 `env` 映射

例程携带一个 `routines.env` 映射，其结构与智能体适配器环境变量相同。每个值要么是字面字符串，要么是指向公司密钥的 `secret_ref`——当例程调用需要 API 密钥的工具，或你想在预发布和生产环境之间切换标志而无需编辑智能体时，这非常有用。

```bash
curl -X PATCH "$PAPERCLIP_API_URL/api/routines/$ROUTINE_ID" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "env": {
      "DEPLOY_TARGET": "staging",
      "GITHUB_TOKEN": {
        "type": "secret_ref",
        "secretId": "'$GITHUB_SECRET_ID'",
        "version": "latest"
      }
    }
  }'
```

在 UI 中，任何 `env` 值为 `type: "secret_ref"` 的字段都通过密钥绑定选择器渲染——与智能体配置中看到的组件相同——因此你可以从现有公司密钥中选择，而不是粘贴明文。有关驱动选择器的 JSON 模式标志，请参阅 [`secret-ref` 表单字段](../../reference/api/secrets.md#secret-ref-form-fields)。

**每次运行都固定在其例程修订版上。** 当例程触发时，运行记录 `routine_runs.routine_revision_id`，指向分发时当前的 `routine_revisions` 行。稍后编辑例程会生成新修订版；正在进行的和历史运行保留其最初执行时的环境变量（及快照的其余部分）。这意味着通过 `version: "latest"` 轮换密钥值会自动流向未来运行，但重命名环境变量键或移除绑定仅影响编辑后创建的运行——运行历史如实记录每次触发实际看到的内容。

---

## 5. （可选）在空闲周不运行

在安静项目上的每日例程大多是噪音——连续三十次站会都说“没有变化”。活动门控将其关闭：只有自例程上次运行以来确实发生了某些事情时，计划触发器才会触发。

### 从例程页面开启

打开例程并进入其 **Delivery** 部分——设置并发和追赶策略的地方。在其下方，你会找到 **Advanced run policy**，包含两个卡片：

- **Run on every scheduled tick** —— “无论发生什么，按计划触发——默认行为。”
- **Skip when there's been no activity since the last run** —— “在计划触发时，仅当上次完成的运行以来发生了某些事情时才运行。让监视器式例程在系统稳定时保持休眠，而不是消耗令牌。”

选择第二个卡片，其下方会出现 **Activity scope** 选择器，包含另外两个卡片：

- **Company-wide** —— “公司范围内的任何活动都算作运行理由。”
- **This project** —— “仅例程项目内的活动算作运行理由。”

保存例程，门控从下一次触发开始生效。

如果例程尚未有计划触发器，你会看到卡片呈灰色而非缺失，并说明原因：“添加计划触发器以根据活动门控运行。Webhook、手动和 API 触发始终运行。” 添加计划触发器（步骤 2）后，控件将激活。

### 或通过 API 设置

这些卡片背后的两个字段是 `activityGatePolicy` 和 `activityGateScope`，因此你可以脚本化整个过程：

```bash
curl -X PATCH "$PAPERCLIP_API_URL/api/routines/$ROUTINE_ID" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "activityGatePolicy": "require_external_activity",
    "activityGateScope": "project"
  }'
```

这两个字段也可在创建时使用，与步骤 1 中的 `concurrencyPolicy` 和 `catchUpPolicy` 并列。默认值为 `activityGatePolicy: "always"` —— 每次触发都运行，这是你从未触碰此设置时的结果——以及 `activityGateScope: "company"`。

### 什么算作活动

“发生了某些事情”意味着在公司活动日志中，在例程上次分发运行之后记录的条目：有人评论、工单移动、智能体完成了一项工作。以下三类条目故意不计入：

- 读取和收件箱簿记——`issue.read_marked`、`issue.read_unmarked`、`issue.inbox_archived`、`issue.inbox_unarchived`。打开收件箱不是新闻。
- 调度器关于此例程记录的任何内容，包括其自身的跳过记录。
- 智能体在处理此例程创建的执行工单时所做的任何操作。

后两者比看起来更重要。没有它们，例程会不断根据自身的输出重新武装，门控永远不会关闭。

`activityGateScope` 决定查看范围。`company` 计算公司范围内的任何活动；`project` 仅计算与例程项目相关的活动——其工单、其例程及其运行，以及处理其工单的智能体运行。在下文的站会和分类模式中，使用 `project`，以便繁忙的邻近项目不会唤醒安静的项目。注意：具有 `activityGateScope: "project"` 但没有 `projectId` 的例程永远无法匹配任何内容，因此永远不会按计划触发。

被门控关闭的触发器不是错误，也不是积压。它显示在运行历史中，作为 `skipped` 运行，带有 `failureReason: "no_external_activity"` —— 在运行行中标记为 **Skipped — no activity since last run** —— 计划推进到下一次触发，当项目再次活跃时不会回填。

门控仅适用于计划触发器。Webhook 触发和 `POST /api/routines/{routineId}/run` 始终分发——如果外部系统费心调用你，该调用 *就是* 活动。

---

## 模式 1 — 每日站会

智能体读取自上次站会以来的变化，并在父项目工单上评论简短摘要。使用 `coalesce_if_active`：如果智能体仍在撰写昨天的站会时今天的触发器到达，合并工作——同一上午的两个站会工单没有价值。

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Daily standup — {{project_name}}",
    "description": "Read issues completed in the last 24h on this project and post a 5-bullet summary as a comment on the parent issue. Include: what shipped, what is blocked, the top three for today.",
    "assigneeAgentId": "'$AGENT_ID'",
    "projectId": "'$PROJECT_ID'",
    "parentIssueId": "'$STANDUP_PARENT_ISSUE_ID'",
    "priority": "low",
    "concurrencyPolicy": "coalesce_if_active",
    "catchUpPolicy": "skip_missed",
    "variables": [
      { "name": "project_name", "label": "Project", "type": "text", "required": true, "defaultValue": "Backend" }
    ]
  }'
```

然后是一个仅工作日的 cron：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/routines/$ROUTINE_ID/triggers" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "kind": "schedule", "cronExpression": "0 9 * * 1-5", "timezone": "Europe/Amsterdam" }'
```

`parentIssueId` 使运行线程化——每次站会执行都成为同一锚定工单的子级，智能体的评论发布在该父级上。你会看到站会历史作为单个线程化对话，而不是 30 个不相关的工单。

---

## 模式 2 — 收件箱分类

智能体读取其分配的 `todo` 工单，重新排序（`critical`/`high`/`medium`/`low`），并重写过时的标题。下一次触发的快照才是重要的；如果上一次触发仍在完成，丢弃今天的。`skip_if_active` 是正确的选择。

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Inbox triage",
    "description": "List your todo and backlog issues. For each: re-set priority based on age + parent project status, sharpen the title if vague, and add a 1-line comment if the issue has been idle for >7 days. Stop at 30 issues per run.",
    "assigneeAgentId": "'$AGENT_ID'",
    "projectId": "'$PROJECT_ID'",
    "priority": "low",
    "concurrencyPolicy": "skip_if_active",
    "catchUpPolicy": "skip_missed"
  }'
```

在工作日每两小时触发：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/routines/$ROUTINE_ID/triggers" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "kind": "schedule", "cronExpression": "0 9-17/2 * * 1-5", "timezone": "Europe/Amsterdam" }'
```

描述中的 30 个工单上限是那种值得在例程本身用普通英语写出的保护——心跳是短执行窗口，试图在一次运行中分类 400 个工单的智能体会超时并看起来卡住。最好限制工作量，让下一次触发处理其余部分。

---

## 模式 3 — 部署检查

智能体每 30 分钟对预发布或生产环境运行一次冒烟测试。失败时，它创建一个 `critical` 工单，包含失败输出并分配给值班工程师。每次运行独立有意义——你不想将 09:00 的失败合并到 09:30 的工单中而丢失时间信息——因此使用 `always_enqueue`。

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Smoke test — staging",
    "description": "curl the staging health endpoint, run the seed-script smoke (reads/writes a throwaway record), and confirm the homepage returns 200 with the expected build SHA. On any failure: open a critical issue with the failing command and its output, assigned to the on-call agent. On success: close this run with a 1-line OK comment.",
    "assigneeAgentId": "'$AGENT_ID'",
    "projectId": "'$PROJECT_ID'",
    "priority": "high",
    "concurrencyPolicy": "always_enqueue",
    "catchUpPolicy": "skip_missed"
  }'
```

全天候每 30 分钟触发：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/routines/$ROUTINE_ID/triggers" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "kind": "schedule", "cronExpression": "*/30 * * * *", "timezone": "UTC" }'
```

这里隐藏了两个判断：

- **`skip_missed` 优于 `enqueue_missed_with_cap`。** 四小时停机后 25 个追赶冒烟测试的积压不会告诉你任何新信息——预发布环境的当前状态会。丢弃错过的窗口，正常运行下一次。
- **`always_enqueue` 优于 `coalesce_if_active`。** 如果下一次触发时上一个冒烟测试仍在运行，你需要一个 *新* 工单，而非合并——一个 30 分钟的冒烟测试耗时 31 分钟本身就是一个值得在运行历史中看到的信号。

---

## Webhook 和手动触发变体

计划不是唯一的触发器。相同的例程可以携带 `schedule`、`webhook` 和 `api` 触发器的任意组合——每个独立触发。

### Webhook

当外部系统应启动例程时使用。部署完成、Stripe 发票已支付、监控警报触发、会议摘要就绪。

```bash
curl -X POST "$PAPERCLIP_API_URL/api/routines/$ROUTINE_ID/triggers" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "webhook",
    "label": "Deploy finished",
    "signingMode": "app_webhook"
  }'
```

响应包含 `webhookUrl` 和 `webhookSecret`。**密钥仅显示一次。** 现在复制它——如果丢失，唯一路径是 `POST /api/routine-triggers/{triggerId}/rotate-secret`，它会生成新密钥并使旧密钥失效。

然后从外部系统调用公共 URL：

```bash
curl -X POST "$WEBHOOK_URL" \
  -H "Authorization: Bearer $WEBHOOK_SECRET" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: deploy-4812" \
  -d '{ "event": "deploy.finished", "environment": "production" }'
```

`app_webhook` 接受 `Authorization: Bearer` 标头或 `X-Hub-Signature` / `X-Hub-Signature-256` 中的请求体 HMAC-SHA256 签名——大多数 SaaS 工具提供的“签名密钥”。这是 UI 中 **Another app or script** 设置创建的内容。

签名模式：`app_webhook`、`bearer`（省略 `signingMode` 时的 API 默认值）、`hmac_sha256`、`github_hmac` 或 `none`。为 GitHub 选择 `github_hmac`，为其他应用选择 `app_webhook`，为你自己计算的时间戳签名选择 `hmac_sha256`，为单行脚本选择 `bearer`。避免 `none` —— URL 变得可匿名触发。发送方必须能够到达 URL：公共服务需要公共可达的 HTTPS 地址，而非 `localhost` 或私有网络。

### 手动（`api`）

当你希望例程在运行历史中但无自动触发器时使用。适用于“从按钮按需运行”或从 CI 脚本化。

```bash
curl -X POST "$PAPERCLIP_API_URL/api/routines/$ROUTINE_ID/triggers" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -d '{ "kind": "api" }'
```

然后按需触发：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/routines/$ROUTINE_ID/run" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "source": "manual",
    "payload": { "context": "Pre-release smoke test" },
    "idempotencyKey": "release-2026-04-27"
  }'
```

`idempotencyKey` 对重试去重——两次传递相同密钥只得到一个运行，而非两个。并发策略仍然适用：在计划运行进行中时触发的手动运行会根据例程策略合并或跳过。

`/run` 端点也适用于 *仅* 具有计划或 Webhook 触发器的例程——它是“立即运行”按钮。你不需要 `api` 触发器即可手动触发；触发器类型存在是为了让运行历史能将运行归因于带标签的触发源。

---

## 验证触发

在看到一次运行之前，不要信任例程。列出最近的运行：

```bash
curl -sS "$PAPERCLIP_API_URL/api/routines/$ROUTINE_ID/runs?limit=10" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```

你会看到的运行：

| 状态 | 含义 |
|---|---|
| `received` | 触发器已接受；分发进行中。 |
| `issue_created` | 已创建并分配新的执行工单。 |
| `coalesced` | 已存在活动运行；此触发器链接到它。 |
| `skipped` | 触发器未创建工作——已存在活动运行且并发策略丢弃它、项目已暂停，或活动门控未发现新内容。`failureReason` 说明原因。 |
| `completed` | 执行工单达到 `done`。 |
| `failed` | 执行工单失败、被取消或分发出错。`failureReason` 字段说明原因。 |

跳过的运行在 `failureReason` 中说明原因：

| `failureReason` | 发生了什么 |
|---|---|
| `no_external_activity` | 活动门控自上次分发运行以来未发现新内容。 |
| `paused` | 触发时例程的项目已暂停。 |
| `worktree_execution_cutoff` | 服务器在开发工作树中运行，未获准执行此例程。 |

没有 `failureReason` 的跳过运行被并发策略丢弃——例如 `skip_if_active` 例程的前一个执行工单仍打开。这种情况也不是谜：其 `linkedIssueId` 指向导致跳过的执行工单。

在 UI 中，例程的 **Runs** 部分列出了例程创建的执行工单，你可以在其中搜索并就地更新状态、优先级或负责人。跳过的触发器不创建工单，因此请在例程的 **Overview** 下的最近运行中查找。

若要深入了解智能体实际做了什么，跟随 `linkedIssueId` 到执行工单并阅读其评论。

若要立即强制一次触发而无需等待 cron：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/routines/$ROUTINE_ID/run" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -d '{ "source": "manual" }'
```

如果新运行显示 `issue_created` 且结果工单在一个心跳内被智能体领取，你就设置好了。如果显示 `failed` 并带有 `failureReason`，最常见原因是：(a) 智能体已终止或暂停（检查 `GET /api/agents/{AGENT_ID}`），(b) 例程引用了已删除的项目、目标或父工单，或 (c) 变量缺少默认值且触发器没有可插值的内容。

---

## 另请参阅

- [Heartbeats & Routines](../guides/projects-workflow/routines.md) —— 完整 UI 演练，并发和追赶深入介绍，变量模板。
- [Routines API](../../reference/api/routines.md) —— 每个端点、每个字段、每个状态。
- [Wire Slack/Discord notifications](./wire-slack-discord-notifications.md) —— 使用相同的例程模式将待批准和阻塞工单管道到频道。
- [Debug a stuck heartbeat](./debug-stuck-heartbeat.md) —— 如果例程触发但智能体无动作，首先查看此处。
- [Glossary](../guides/welcome/glossary.md) —— 例程、心跳、执行工单的定义。
