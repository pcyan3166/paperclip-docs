---
paperclip_version: v2026.824.0
seo_title: 待处理事项 API
seo_description: 读取看板的优先级决策队列，即决策页面背后的动态流，一次调用即可查看审批、智能体问题和受阻工作。
---

# 待处理事项

读取看板当前的决策队列。这是 **Decisions** 页面背后的动态流：按优先级统一展示需要人员处理的工作，包括审批、问题、受阻工作、失败的运行、预算提醒和智能体错误。

如果你正在构建运维视图，或需要为看板用户提供简明、实时的待处理事项列表，请使用此 API。它是只读动态流；人员采取的操作仍由相应的审批、交互、任务或提醒负责。

如果你还没有阅读，请先查看 [API 概览](./overview.md)，了解基础 URL、身份验证和公司范围规则。

---

## 读取队列

```
GET /api/companies/{companyId}/attention
```

此路由要求调用方有权访问指定公司和看板。看板调用方还必须具有用户上下文，否则会返回 `403` 和 `"Board user context required"`。

默认情况下，动态流会排除当前用户已忽略或暂缓的事项。呈现这些隐藏记录时，请添加 `includeDismissed=true`。

```bash
curl -s \
  "http://localhost:3100/api/companies/{companyId}/attention?includeDismissed=true" \
  -H "Authorization: Bearer <token>"
```

### 查询参数

| 参数 | 说明 |
|---|---|
| `includeDismissed` | 设为 `true` 时，包含当前用户已忽略或暂缓的记录。 |
| `activitySince` / `activityUntil` | 以 ISO 时间戳限定各事项的 `activityAt` 范围。 |
| `queue` | 只返回属于指定键值的[决策队列](./decisions.md)事项。 |
| `sort` | `activity`（默认值）或 `decide`。 |
| `cursor` | 从上一页的 `nextCursor` 继续读取。 |
| `limit` | 1 到 100 之间的整数，默认值为 50。 |

时间边界格式错误时会返回 `400 Bad Request`，并附带 `"activitySince must be an ISO timestamp"`（`activityUntil` 同理）；如果起始时间晚于结束时间，则返回 `"activitySince must be before or equal to activityUntil"`。其他验证消息包括 `"sort must be 'activity' or 'decide'"`、`"limit must be an integer between 1 and 100"`；游标格式错误或由不同排序方式生成时，会返回 `"Invalid attention cursor"`。

响应为 `AttentionFeed`，包含以下顶层字段：

| 字段 | 内容 |
|---|---|
| `companyId` | 拥有此动态流的公司。 |
| `generatedAt` | Paperclip 生成此视图的时间。 |
| `totalCount` | 过滤后的完整动态流中的事项数量，不受分页影响。 |
| `decideNowCount` | 根据决策截止时间统计今天到期的事项数。看板侧边栏徽标显示的就是此数量。 |
| `nextCursor` | 将此值作为 `cursor` 传入以读取下一页；到达末页时为 `null`。 |
| `countsBySourceKind` | 按待处理事项来源类别分组的计数。 |
| `items` | 当前页按优先级排列的 `AttentionItem` 记录。 |

每个事项会说明需要立即关注的原因（`whyNow`）、应打开的主题（`subject`）、建议采取的操作（`decisionVerbs`）、严重程度（`severity`）以及活动时间（`activityAt`）。它还包含 `entryRule` 和 `exitRule`，便于运维人员了解事项为何出现，以及满足什么条件后会消失。如有相关信息，`relatedIssue`、`project`、`workspace` 和 `detail` 会提供上下文，无需再发起查询。

事项还带有其侧边状态及少量来源信息：

| 字段 | 内容 |
|---|---|
| `rank` | 事项在完整优先级动态流中的位置，从 1 开始。 |
| `queues` | 此事项所属的决策队列，每项为 `{ key, title }`。 |
| `decideBy` | `today`、`this_week`、`whenever`、`YYYY-MM-DD` 日期，或 `null`。 |
| `decideByAttribution` | 设置此期限的人；未设置时为 `null`。 |
| `snoozedUntil` | 分诊暂缓结束的时间；未暂缓时为 `null`。 |
| `expiresAt` | 底层工作过期的时间；不会过期时为 `null`。 |
| `ruleKey` | 提出此决策时依据的规则；未指定时为 `null`。 |
| `originAgentName` | 创建此事项的智能体名称（如有）。 |
| `inlineResolvable` | 是否可以直接在列表行中解决此事项，而无需打开主题。 |
| `trainingExampleId` | 如果你已将此事项保存为[决策训练](./decision-training.md)示例，则会设置此字段。 |

### 来源和严重程度

`sourceKind` 可能为以下值之一：

- `approval`
- `decision`
- `issue_thread_interaction`
- `join_request`
- `recovery_action`
- `productivity_review`
- `blocker_attention`
- `review`
- `failed_run`
- `budget_alert`
- `agent_error_alert`

`severity` 可能为 `critical`、`high`、`medium` 或 `low`。

Paperclip 会先对底层信号去重，再排列动态流。默认使用 `sort=activity` 时，排序依次优先考虑近期活动、严重程度、来源优先级和稳定的去重键。使用 `sort=decide` 时，动态流会优先显示有明确期限的事项（按截止时间由近到远），然后是标记为 `whenever` 的事项，最后是尚未分诊的事项；同一组内依次按 `expiresAt`、严重程度及上述活动顺序排序。

请将它视为决策队列，而不是公司所有事件的完整历史记录。

### 动态流中的决策

`decision` 事项表示智能体尚未解决的提案：提出一个问题，附带一组简短选项，并准确说明每个选项会进行哪些更改。这类记录可以直接在列表中处理；`expiresAt` 表示提案自行失效的时间。完整记录及答复路由见[决策 API](./decisions.md)。

---

<a id="agent-addressed-issue-thread-interactions"></a>
## 指定给智能体的任务讨论串交互

`issue_thread_interaction` 事项是智能体在任务讨论串中提出的卡片，例如问题、确认请求或一组评判意见。过去，此类卡片都需要等待看板处理。现在，卡片可以**指定给某个智能体**，并由**符合公司治理规则的智能体**解决；这会改变有权答复的人，也会影响卡片是否出现在此动态流中。

完整交互生命周期（创建、答复、撤回、过期）见 [Issues API](./issues.md#interactions)。本节介绍决定卡片路由方式的接收方和解决者策略接口。

### 解决者策略

每张卡片都带有一项解决者策略，用来指定可以答复的对象。规范值如下：

| 值 | 对象 |
|---|---|
| `anyone` | 公司中的任何人都可以答复，包括看板用户、任何智能体以及提出问题的智能体。 |
| `not_creator` | 公司中的任何人都可以答复，但不包括创建卡片的智能体及其创建运行。 |
| `human_only` | 只有看板中的人员可以答复；智能体会被拒绝。 |

旧值 `board_or_agents` 和 `board_only` 已弃用，但为兼容性保留一个迁移周期，且在此期间仍可写入。写入时会规范化为标准值：`board_or_agents` → `anyone`，`board_only` → `human_only`；读取记录时始终返回标准值。为保持向后兼容，每条记录还包含 `legacyResolverPolicyAliases`，其中保存请求策略和生效策略对应的别名形式（`anyone` 和 `human_only` 分别映射回 `board_or_agents` 和 `board_only`；`not_creator` 没有别名，因此返回 `null`）。

创建卡片时，以下两个字段决定其路由：

| 字段 | 值 | 含义 |
|---|---|---|
| `resolverPolicy` | `anyone`、`not_creator`、`human_only`（或已弃用的别名） | 创建卡片时请求采用的策略。创建时可选。 |
| `addresseeAgentId` | 智能体 UUID 或 `null` | 可选。将卡片指定给某个智能体，并唤醒该智能体来解决问题。必须引用同一公司中可调用的智能体。 |

服务器会根据**公司治理规则**解析请求的策略，并将结果保存在交互记录中：

| 字段 | 含义 |
|---|---|
| `resolverPolicy` | `requestedResolverPolicy` 的弃用镜像字段，为保持 API 兼容而保留。 |
| `requestedResolverPolicy` | 请求的策略，已规范化：依次采用创建请求中的 `resolverPolicy`、公司的按类型 `defaultPolicy`，或内置的按类型默认值。所有交互类型（`suggest_tasks`、`ask_user_questions`、`request_confirmation`、`request_checkbox_confirmation`、`request_item_verdicts`）的内置默认值都是 `anyone`。未明确指定策略时，默认开放答复。 |
| `effectiveResolverPolicy` | 实际执行的策略。除非某条规则提高了限制程度（见下文），否则等于 `requestedResolverPolicy`。 |
| `resolverPolicyProvenance` | 请求策略的确定方式：`explicit` 表示请求中明确指定，`inherited` 表示继承自治理规则或内置默认值。旧模型下因继承而受到限制的迁移前记录会返回 `legacy_inherited_restriction`。 |
| `effectiveResolverPolicySource` | 生效策略的来源：`requested`、`company_cap` 或 `governed_action`。 |

限制只能沿着 `anyone` → `not_creator` → `human_only` 的方向收紧（从最宽松到最严格）。以下两种规则可以收紧策略：

- **受治理的操作**：携带工具操作的卡片会被强制设为 `human_only`（`effectiveResolverPolicySource` 为 `governed_action`），无论请求的对象是谁。
- **公司上限**：如果公司的按类型治理设置了 `cap`，且该上限比请求的策略更严格，卡片会收紧到该上限（`effectiveResolverPolicySource` 为 `company_cap`）。上限只能缩小允许范围，不能放宽卡片权限。

公司治理配置保存在公司的 `interactionResolverGovernance` 设置中，是按交互类型划分的 `{ defaultPolicy?, cap? }` 映射；每个值都必须是标准策略（或已弃用的别名）。`defaultPolicy` 用于在创建请求未设置 `resolverPolicy` 时提供回退对象；`cap` 是只能收紧的上限。此映射**只能收紧范围**：它可以缩小卡片受众，但不能放宽到超出卡片自身请求的范围。

### 谁可以解决交互

看板用户（人类）始终可以解决卡片，但 `not_creator` 会排除创建卡片的人类用户。**智能体**只有通过以下所有检查后才能解决卡片；检查按以下顺序执行：

- 调用必须携带经过身份验证的智能体运行（包含智能体 ID 和非空运行 ID），否则返回 `422`、错误码 `interaction_run_attribution_required`（`A valid authenticated agent run is required to resolve this issue-thread interaction`）。
- 卡片不能绑定受治理的操作，否则返回 `403`、错误码 `interaction_governed_action_denied`（`This interaction is bound to a governed action that requires independent authorization`）。
- `effectiveResolverPolicy` 不能是 `human_only`，否则返回 `403`、错误码 `interaction_human_only`（`This issue-thread interaction is human-only`）。
- 如果设置了 `addresseeAgentId`，调用智能体必须是该接收方，否则返回 `403`、错误码 `interaction_addressee_mismatch`（`Only the addressed agent or an authorized human may resolve this issue-thread interaction`）。
- 在 `not_creator` 策略下，调用方不能是创建智能体（`createdByAgentId`），也不能是创建运行（`sourceRunId`），否则返回 `403`、错误码 `interaction_creator_excluded`（`This issue-thread interaction requires a resolver other than its creator or creating run`）。对于采用 `not_creator` 策略的人类调用方，对应的拒绝信息是 `This issue-thread interaction requires a resolver other than its creator`。

受治理操作的确认始终仅限人类处理：智能体请求会在受众检查前被拒绝；即使是人类，也必须通过该操作自身独立的授权流程来解决。任务监控器和其他系统运行会视为 `system` 操作方，并始终允许处理。

解决操作的完整拒绝码列表为：`interaction_not_found`、`interaction_run_attribution_required`、`interaction_scope_denied`、`interaction_human_only`、`interaction_creator_excluded`、`interaction_addressee_mismatch`、`interaction_stale_target`、`interaction_superseded`、`interaction_already_resolved`、`interaction_issue_closed`、`interaction_governed_action_denied` 和 `review_policy_denied`。每个拒绝响应都会在其详情中包含导致拒绝的 `effectiveResolverPolicy`。

### 待处理事项动态流的过滤规则

看板动态流会有意隐藏预期由受治理智能体处理的卡片。当 `addresseeAgentId` 指向当前**可调用**的智能体时，`issue_thread_interaction` 事项会**从此动态流中移除**：卡片应由指定智能体处理，而不是由看板处理。以下情况的卡片仍会显示在动态流中：

- `addresseeAgentId` 为 `null`（卡片是为看板提出的）；或
- 指定的智能体已不可调用；此时卡片会回到看板，避免无人处理。

### 撤回与终止过期

指定给智能体的卡片会通过与其他交互相同的管理性结束方式完成，另外还有一种针对接收方的结束方式：

- **撤回**：创建者、当前任务负责人或看板用户可以撤回待处理卡片；其 `result.outcome` 会变为 `withdrawn`。参见[撤回与取消的区别](./issues.md#withdraw-vs-cancel)。
- **任务关闭**：任务状态变为 `done` 或 `cancelled` 时，该任务上所有仍待处理的卡片都会过期，`result.outcome` 为 `issue_closed`。
- **接收方已删除**：指定的智能体被删除时，其待处理卡片会被取消，`result.outcome` 为 `addressee_deleted`（`Cancelled because the addressed agent was deleted`）。

`withdrawn`、`issue_closed` 和 `addressee_deleted` 都属于管理性结束，不代表有人作出了答复或决策。

---

## 忽略或暂缓事项

待处理事项端点不会修改队列。忽略状态属于当前看板用户，应改用收件箱忽略状态路由。

```
GET    /api/companies/{companyId}/inbox-dismissals
POST   /api/companies/{companyId}/inbox-dismissals
DELETE /api/companies/{companyId}/inbox-dismissals/{itemKey}
```

处理待处理事项时，请传入以 `attention:` 开头的 `dismissalKey`。`POST` 正文包含 `itemKey`、可选的 `kind`（`dismiss` 或 `snooze`）；暂缓时还需提供 `snoozedUntil`。`snoozedUntil` 必须是未来的 ISO 时间戳；忽略事项时不能提供此字段。成功创建会返回 `201`；删除同一事项键会恢复该事项，并返回 `204`。

```json
{
  "itemKey": "attention:<attention-dismissal-key>",
  "kind": "snooze",
  "snoozedUntil": "2026-07-13T09:00:00.000Z"
}
```

这些路由要求看板身份验证和看板用户上下文。无效键会返回 `"Unsupported inbox item key"`；无效的暂缓时间会返回 `"snoozedUntil must be an ISO timestamp"` 或 `"snoozedUntil must be in the future"`。

还有另一种不同的暂缓方式。收件箱忽略状态只对你个人生效，并显示在事项的 `dismissal` 字段中。**分诊暂缓**设置在事项来源上，对所有读取动态流的人生效，并显示在 `snoozedUntil` 字段中。如果希望整个公司在某个日期前都看不到某事项，请使用分诊暂缓；如果只是想整理自己的队列，请使用个人忽略状态。分诊暂缓功能见[决策 API](./decisions.md)。

---

## 后续阅读

- [决策](../../guides/day-to-day/decisions.md)：使用内置运维工作流。
- [决策 API](./decisions.md)：了解决策记录、队列和分诊功能。
- [审批](./approvals.md)：处理队列中的审批记录。
- [任务](./issues.md)：处理任务讨论串交互，以及许多待处理事项背后的任务。
