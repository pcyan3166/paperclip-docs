---
seo_title: 执行策略：强制执行审查关卡
seo_description: 执行器尝试关闭任务时，运行时会拦截并转入审查和审批流程。介绍三层策略、修改循环和升级处理。
---

# 执行策略

Paperclip 的执行策略系统让任务流转更加可靠。它不依赖智能体记得把工作交给别人审查，而是由**运行时自动强制执行**审查和审批阶段：执行者尝试关闭任务时，运行时会拦截状态转换，并将工作转给相应的审查者或审批者。

本页介绍何时使用执行策略、三层强制机制如何组合，以及如何通过界面和 API 配置策略。

---

## 三层策略

| 层级 | 用途 | 范围 |
|---|---|---|
| **必须发表评论** | 每次智能体运行都必须在任务上发表评论 | 运行时约束，始终启用 |
| **审查阶段** | 审查者检查质量并可要求修改 | 按任务配置，可选 |
| **审批阶段** | 经理或相关人员作最终确认 | 按任务配置，可选 |

一项任务可以只配置审查、只配置审批、按顺序同时配置两者，或两者都不配置（仍须满足必须发表评论的基本要求）。

---

## 标准流程：审查 → 审批

```
┌────────┐   执行者       ┌───────────┐   审查者       ┌───────────┐   审批者       ┌──────┐
│ todo   │──完成工作────▶│ in_review │──批准───────▶│ in_review │──批准───────▶│ done │
│(编码者)│               │   (QA)    │              │  (CTO)    │              │      │
└────────┘               └───────────┘              └───────────┘              └──────┘
```

1. 创建任务时，通过 `executionPolicy` 指定审查阶段（例如 QA）和审批阶段（例如 CTO）。
2. 执行者在 `in_progress` 状态下处理任务。
3. 执行者尝试将任务状态改为 `done`，运行时会拦截：
   - 状态改为 `in_review`，而不是 `done`。
   - 任务重新分配给第一位审查者。
   - `executionState` 将审查阶段设为 `pending`。
4. 审查者附上评论并将任务状态改为 `done`，以表示批准。
   - 系统会创建结果为 `approved` 的决策记录。
   - 任务仍为 `in_review`，并重新分配给审批者。
   - `executionState` 前进到审批阶段。
5. 审批者附上评论并将状态改为 `done`，以表示批准。
   - `executionState.status` 变为 `completed`。
   - 任务最终进入 `done` 状态。

---

## 要求修改后的循环

```
┌───────────┐   审查者要求       ┌─────────────┐   执行者         ┌───────────┐
│ in_review │──修改────────────▶│ in_progress │──重新提交──────▶│ in_review │
│   (QA)    │                   │   (编码者)  │                │   (QA)    │
└───────────┘                   └─────────────┘                └───────────┘
```

1. 审查者发表评论，说明需要修改之处，并将任务转换为 `done` 以外的任意状态（通常是 `in_progress`）。
2. 运行时会自动：
   - 将状态设为 `in_progress`。
   - 重新分配给原执行者（记录在 `returnAssignee` 中）。
   - 将 `executionState.status` 设为 `changes_requested`。
3. 执行者完成修改后，再次将状态改为 `done`。
4. 运行时会把任务送回**同一位审查者**，而不是从策略起点重新开始。
5. 循环持续进行，直至审查者批准。

---

## 循环无法结束时：审查轮数上限与升级处理

要求修改的循环本应逐步收敛，但有时不会。例如双方都是智能体，审查者的成功标准已偏离到执行者无法满足，工作便会被无限退回；每轮都会消耗一次运行，却没有进展。

因此，Paperclip 会统计修改轮数，并在达到上限时将审查交给人工处理。

**计数方式。** `executionState.changesRequestedCount` 记录当前阶段中由**智能体发起**的连续“要求修改”轮数。执行者重新提交后会继续累计；阶段获批时计数会重置为 `0`。

**人工决定不计入。** 如果是*人类*要求修改，计数会重置为 `0`。上限用于阻止无人监督的智能体互相退回，不会限制你自己的审查。因此，只要流程中有人类参与，循环实际上不受此上限约束。

**达到上限时的处理。** 默认上限为 `3`。连续第三次由智能体作出的“要求修改”决定会与前两次不同：决定仍会连同审查者评论一起记录，但运行时不会把工作退回执行者，而是：

- 保持阶段为 `pending`，任务为 `in_review`；
- 将**任务责任人**设为该阶段参与者——优先使用 `responsibleUserId`，若不存在则使用 `createdByUserId`——并把任务分配给该人员；
- 通过你已经使用的关注和审查界面显示这项待处理审查。

此后由你处理：可以批准、亲自要求修改（计数会重置并将任务交还执行者），或重新界定工作范围。

**升级后会保持锁定。** 一旦审查升级，只有被升级到的人工审查者才能推进流程。其他人尝试推进阶段或重新分配时，会收到 `Only the escalated reviewer can advance the current execution stage` 错误。如果任务状态或负责人发生偏移，运行时会将其恢复。这样可以防止无关更新悄悄取消升级处理。

**任务没有责任人时**（`responsibleUserId` 和 `createdByUserId` 均不存在），行为不变：工作仍按原流程交还执行者，但计数会继续增加，因此仍可看出循环次数。

### 调整轮数上限

在任务的 `executionPolicy` 上设置 `maxReviewRounds` 可覆盖默认值：

```json
{
  "stages": [
    { "type": "review", "participants": [{ "type": "agent", "agentId": "qa-agent-id" }] }
  ],
  "maxReviewRounds": 5
}
```

该值必须是 `1` 至 `50` 的正整数。省略此字段或设为 `null` 时，服务器默认值为 `3`。设为 `1` 可让智能体第一次要求修改时就交由人工处理。

此功能推出前创建的任务会读取为 `changesRequestedCount: 0`，无需迁移数据。

---

## 策略示例

**仅审查：**

```json
{
  "stages": [
    { "type": "review", "participants": [{ "type": "agent", "agentId": "qa-agent-id" }] }
  ]
}
```

执行者完成 → 审查者批准 → 任务完成。

**仅审批：**

```json
{
  "stages": [
    { "type": "approval", "participants": [{ "type": "user", "userId": "manager-user-id" }] }
  ]
}
```

执行者完成 → 审批者签字确认 → 任务完成。

**多位审查者或审批者：**每个阶段都支持多位参与者。运行时会选择其中一位处理，并排除原执行者，以避免自己审查自己的工作。

---

## 必须发表评论的基本保障

无论是否设置了审查阶段，每次与任务关联的智能体运行结束时都必须发表评论。此要求由运行时强制执行：

1. 运行结束后，运行时检查智能体是否在本次运行期间发表评论。
2. **没有评论：** `issueCommentStatus` 变为 `retry_queued`，并以 `missing_issue_comment` 原因再次唤醒智能体一次。
3. **重试后仍没有评论：** `issueCommentStatus` 变为 `retry_exhausted`，不再重试并记录失败。
4. **已发表评论：** `issueCommentStatus` 变为 `satisfied`，并关联评论 ID。

这样可以防止智能体不留任何记录就直接关闭工作。

### 运行级跟踪字段

| 字段 | 说明 |
|---|---|
| `issueCommentStatus` | `satisfied`、`retry_queued` 或 `retry_exhausted` |
| `issueCommentSatisfiedByCommentId` | 满足要求的评论 ID |
| `issueCommentRetryQueuedAt` | 安排再次唤醒的时间戳 |

---

## 数据模型

### 执行策略（任务字段：`executionPolicy`）

```ts
interface IssueExecutionPolicy {
  mode: "normal" | "auto";
  commentRequired: boolean;       // 始终为 true，由运行时强制执行
  stages: IssueExecutionStage[];  // 有顺序的审查/审批阶段列表
  maxReviewRounds?: number | null; // 1–50；null 使用服务器默认值 3
}

interface IssueExecutionStage {
  id: string;                     // 自动生成的 UUID
  type: "review" | "approval";
  approvalsNeeded: 1;             // 暂不支持多重批准
  participants: IssueExecutionStageParticipant[];
}

interface IssueExecutionStageParticipant {
  id: string;
  type: "agent" | "user";
  agentId?: string | null;
  userId?: string | null;
}
```

### 执行状态（任务字段：`executionState`）

```ts
interface IssueExecutionState {
  status: "idle" | "pending" | "changes_requested" | "completed";
  currentStageId: string | null;
  currentStageIndex: number | null;
  currentStageType: "review" | "approval" | null;
  currentParticipant: IssueExecutionStagePrincipal | null;
  returnAssignee: IssueExecutionStagePrincipal | null;
  completedStageIds: string[];
  lastDecisionId: string | null;
  lastDecisionOutcome: "approved" | "changes_requested" | null;
  changesRequestedCount?: number;  // 本阶段由智能体连续发起的修改轮数
}
```

### 执行决策（数据表：`issue_execution_decisions`）

```ts
interface IssueExecutionDecision {
  id: string;
  companyId: string;
  issueId: string;
  stageId: string;
  stageType: "review" | "approval";
  actorAgentId: string | null;
  actorUserId: string | null;
  outcome: "approved" | "changes_requested";
  body: string;              // 必填评论，用于说明决策原因
  createdByRunId: string | null;
  createdAt: Date;
}
```

每项决策都会记录参与者、结果、评论和运行 ID。可以按任务查询完整审查历史。

---

## 访问控制

- 只有**当前审查者或审批者**（即执行状态中的 `currentParticipant`）可以推进或拒绝当前阶段。
- 非参与者尝试转换任务状态时，会收到 `422 Unprocessable Entity`。
- 阶段升级给责任人类用户后，只有该用户能推进；其他所有人（包括已配置的阶段参与者）都会被拒绝。
- 批准和要求修改时**都必须附带评论**；空评论或仅包含空格的评论会被拒绝。

---

## API 用法

### 创建任务时设置策略

```sh
POST /api/companies/{companyId}/issues
{
  "title": "Implement feature X",
  "assigneeAgentId": "coder-agent-id",
  "executionPolicy": {
    "mode": "normal",
    "commentRequired": true,
    "stages": [
      { "type": "review",   "participants": [{ "type": "agent", "agentId": "qa-agent-id"  }] },
      { "type": "approval", "participants": [{ "type": "user",  "userId":  "cto-user-id" }] }
    ]
  }
}
```

省略阶段 ID 和参与者 ID 时会自动生成。重复参与者会去重。没有有效参与者的阶段会被移除；如果最终没有有效阶段，策略会设为 `null`。

### 更新现有任务的策略

```sh
PATCH /api/issues/{issueId}
{
  "executionPolicy": { ... }
}
```

如果审查正在进行时移除策略（设为 `null`），执行状态会被清除，任务会交还原执行者。

### 推进阶段（批准）

当前审查者或审批者附上评论并将任务改为 `done`：

```sh
PATCH /api/issues/{issueId}
{
  "status": "done",
  "comment": "Reviewed — implementation looks correct, tests pass."
}
```

运行时会判断此操作是完成整个工作流，还是进入下一阶段。

### 要求修改

附上评论，并将任务转换为 `done` 以外的状态：

```sh
PATCH /api/issues/{issueId}
{
  "status": "in_progress",
  "comment": "Button alignment is off on mobile. Please fix the flex container."
}
```

运行时会自动将任务重新分配给原执行者。

---

## 界面

### 新建任务对话框

创建任务时，负责人选择器旁会出现 **Reviewer（审查者）** 和 **Approver（审批者）** 按钮。点击后会打开参与者选择器，其中包括：

- *No reviewer（无审查者）* / *No approver（无审批者）*（清除选择）
- *Me（我）*（当前用户）
- 全部智能体和董事会用户

选择结果会自动生成 `executionPolicy.stages` 数组。

### 任务属性面板

对于现有任务，属性面板会显示可编辑的 **Reviewer（审查者）** 和 **Approver（审批者）** 字段。每个阶段都可以添加多位参与者。修改后会通过 API 保存到任务的 `executionPolicy` 中。

---

## 设计原则

1. **由运行时强制执行，不依赖提示词。** 智能体不必记得转交工作；运行时会拦截状态转换并按流程转交。
2. **允许迭代，但设置边界。** 审查是一个“要求修改、修改、再次审查”的循环，而非一次性的关卡。重新提交后会回到同一阶段。无人监督的循环仍会结束：智能体达到 `maxReviewRounds` 后，任务会转交人工决定，不会无限消耗运行次数。
3. **角色灵活。** 参与者可以是智能体或用户。并非每个组织都有 QA 团队；审查者/审批者模式也适用于同级审查、经理签字或合规检查。
4. **全过程可审计。** 每项决策都会记录参与者、结果、评论和运行 ID。
5. **保留单任务单次运行约束。** 审查唤醒和评论重试都遵守现有规则：同一任务同一时间只能有一位智能体在运行。
