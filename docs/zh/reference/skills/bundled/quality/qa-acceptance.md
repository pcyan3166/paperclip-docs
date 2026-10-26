---
paperclip_version: v2026.618.0
seo_title: QA 验收技能
seo_description: 编写审核者无需询问作者即可执行和判断的验收标准：主流程、边界情况、错误状态和性能限制。
---

# QA 验收

> 为功能变更制定 QA 验收标准和手动验证计划，覆盖主流程、边界情况、错误状态、性能限制，并明确通过/失败的证据。

编写审核者可以直接在运行中的应用上执行的验收标准，使其无需询问作者即可判断通过或失败。内容需涵盖主流程、边界情况、错误状态、性能限制以及明确的通过/失败证据。验收标准就是约定；自动化测试负责验证正确性，QA 则负责功能层面的行为。

此技能是**内置**目录技能，属于内置基础套件。如何安装、审计、更新、分配和重置目录技能，请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

## 目录元数据

| 字段 | 值 |
|---|---|
| Catalog id | `paperclipai:bundled:quality:qa-acceptance` |
| Canonical key | `paperclipai/bundled/quality/qa-acceptance` |
| Catalog path | `catalog/bundled/quality/qa-acceptance` |
| Kind | `bundled` |
| Category | `quality` |
| Slug | `qa-acceptance` |
| Entrypoint | `SKILL.md` |
| Trust level | `markdown_only` |
| Compatibility | `compatible` |
| Default install | `false` |
| Recommended roles | `qa`、`engineer`、`product` |
| Requires | — |
| Tags | `qa`、`acceptance`、`validation`、`testing` |
| Files | 1 |
| Content hash | `sha256:32372dacaf62e93454b9855968c4eec96456ba78b509f450b3dfaa48e31ef356` |
| Package | `@paperclipai/skills-catalog@0.3.1` |

## 文件清单

| 路径 | 类型 | 字节数 |
|---|---|---:|
| `SKILL.md` | `skill` | 3,861 |

## 完整技能定义

以下代码块是此技能完整且权威的 `SKILL.md`，也是智能体在运行时加载的确切文件。可使用代码块右上角的控件复制或下载为 `SKILL.md`。

````markdown skill-source
---
name: qa-acceptance
description: Produce QA acceptance criteria and a manual validation plan for a feature change — golden path, edge cases, error states, performance limits, and explicit pass/fail evidence.
key: paperclipai/bundled/quality/qa-acceptance
recommendedForRoles:
  - qa
  - engineer
  - product
tags:
  - qa
  - acceptance
  - validation
  - testing
---

# QA Acceptance

Write acceptance criteria that a reviewer can run against the running app and decide pass or fail without asking the author. The criteria are the contract — automated tests cover correctness, QA covers feature-level behavior.

## When to use

- A feature change is heading to QA and needs a written validation plan.
- A reviewer is asked to verify a PR that touches user-visible behavior.
- An incident postmortem requires a regression check before reopen-prevention.
- A release candidate needs a pre-cut smoke pass.

## When not to use

- The change is unit-test-only (utility refactor, internal naming). Acceptance criteria are unnecessary churn.
- You are asked to write tests against API contracts. Use contract testing, not feature QA.

## Acceptance criteria format

Each criterion is a single, independently-verifiable statement:

```md
- **Given** <starting state>, **when** <action>, **then** <observable outcome>.
```

Example:

```md
- **Given** a CSV export with 0 rows, **when** the user clicks Export, **then** the file downloads with only the header row and the UI shows "Exported 0 rows".
```

Avoid criteria that combine multiple `when`s or `then`s. Split them.

## What every plan must cover

1. **Golden path.** The most common successful flow, end to end.
2. **Empty and minimum states.** Zero items, one item, missing optional inputs.
3. **Boundary inputs.** Max length strings, max numeric values, unicode, RTL text where applicable.
4. **Error states.** Network failure, permission denied, validation failures, conflict (409), not found (404).
5. **Concurrency and ordering.** Two users acting at once, race against background jobs, refresh during mutation.
6. **Performance envelope.** The largest realistic input the change must handle without UI hangs or timeouts.
7. **Backward compatibility.** Existing data, existing URLs, persisted user preferences continue to work.
8. **Telemetry and audit.** Events, logs, or activity entries the change is supposed to emit.

If a section is genuinely not applicable, write "N/A: <why>" — do not silently omit.

## Evidence

Each criterion needs evidence on the verification pass:

- Screenshot or short clip for UI behavior.
- Copied console / network output for API behavior.
- Log snippet or activity row for telemetry.
- Timing measurement for performance criteria.

"Looks good to me" without evidence is not a pass.

## Quarantine and follow-up

- A failing criterion blocks acceptance unless explicitly waived by the owner with a tracked follow-up issue.
- "Known issue" without a linked follow-up is not a waiver.
- If you add a new criterion mid-pass, restart the pass — partial coverage hides regressions.

## Handoff back to the author

Return the validation plan with three sections:

- **Pass.** Criteria that passed, with one-line evidence summaries.
- **Fail.** Criteria that failed, with the exact reproduction.
- **Blocked.** Criteria you could not run, with why.

The author owns turning failures into either fixes or accepted deferrals.

## Anti-patterns

- Acceptance phrased as test plan ("write a Cypress test for X"). Acceptance is what is true after the change ships; tests are how you check.
- Criteria that depend on inspecting implementation details (selectors, query plans). Stay observable.
- Long checklists with no priority. Mark must-pass criteria distinctly from nice-to-have.
- Validation reports that say "passed" with no evidence. Reviewers cannot audit those.
````

## 适用场景

- 功能变更即将进入 QA，需要书面的验证计划。
- 审核者需要验证影响用户可见行为的拉取请求。
- 事故复盘要求在重新开放功能前执行回归检查，以防问题再次出现。
- 发布候选版本需要执行发布前冒烟检查。

### 不适用场景

- 变更仅涉及单元测试（例如工具函数重构或内部命名）。编写验收标准只会增加不必要的工作。
- 用户要求编写 API 合约测试。应使用合约测试，而不是功能 QA。

## 验收标准格式

每条标准都是一项可独立验证的陈述：

```md
- **假设** <起始状态>，**当** <操作>，**那么** <可观察结果>。
```

示例：

```md
- **假设** CSV 导出有 0 行，**当** 用户点击“导出”，**那么** 文件仅包含标题行并下载，且界面显示“已导出 0 行”。
```

避免在同一条标准中组合多个“当”或“那么”条件。应将它们拆开。

## 每份计划都必须覆盖的内容

1. **主流程。** 最常见的成功流程，从头到尾都要覆盖。
2. **空状态和最小状态。** 零条目、一条条目、缺少可选输入。
3. **边界输入。** 最大长度字符串、最大数值、Unicode，以及适用时的 RTL 文本。
4. **错误状态。** 网络故障、权限拒绝、验证失败、冲突（409）、找不到资源（404）。
5. **并发和顺序。** 两个用户同时操作、与后台任务竞态、变更过程中触发刷新。
6. **性能范围。** 此变更必须处理的最大现实输入，且不能造成 UI 卡顿或超时。
7. **向后兼容性。** 现有数据、URL 和已保存的用户偏好是否继续有效。
8. **遥测和审计。** 此变更预期发出的事件、日志或活动记录。

如果某一节确实不适用，请写明 “N/A: <原因>”，不要默默省略。

## 验证证据

验证过程中，每条标准都需要证据：

- UI 行为：截图或简短录屏。
- API 行为：复制控制台或网络输出。
- 遥测：日志片段或活动记录。
- 性能标准：计时数据。

没有证据的“我觉得看起来不错”不算通过。

## 隔离失败项和后续处理

- 验收失败项会阻止验收，除非负责人明确豁免并创建可追踪的后续任务。
- 没有关联后续任务的“已知问题”不构成豁免。
- 如果验证过程中新增了验收标准，应重新开始整轮验证；只覆盖部分内容会掩盖回归问题。

## 向作者交接

将验证计划按以下三个部分返回：

- **通过。** 已通过的标准，并附一行证据摘要。
- **失败。** 未通过的标准，并提供准确的复现步骤。
- **受阻。** 无法执行的标准，并说明原因。

作者负责将失败项转化为修复或经接受的延期事项。

## 反模式

- 将验收写成测试计划（例如“为 X 编写 Cypress 测试”）。验收描述的是变更发布后应满足的事实；测试只是检查方法。
- 依赖检查实现细节（选择器、查询计划）的标准。应描述可观察结果。
- 长清单没有优先级。应明确标记必须通过的标准和可选项。
- 验证报告只说“通过”而没有证据。审核者无法审计这样的报告。

## 另请参阅

- [内置技能](../../bundled.md)——按类别整理的全部内置目录技能。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、分配和故障排查。
- [应用内置目录](../../../skills.md#3-app-shipped-catalog)——内置和可选目录技能的版本管理与更新方式。
- [技能指南](../../../../guides/org/skills.md)——界面操作指南和内置目录概览。
