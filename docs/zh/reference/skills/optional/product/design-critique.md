---
paperclip_version: v2026.618.0
seo_title: 产品设计评审技能
seo_description: 对屏幕、流程或组件进行结构化评审，按影响程度排序列出待改进项，并说明理由。
---

# 产品设计评审

> 对产品设计进行结构化评审，检查用户任务是否清晰、信息层级、可供性、错误状态、无障碍性和一致性，重点说明要改什么、按什么顺序改，以及为什么要改。

对屏幕、流程或组件进行一轮结构化评审。输出应是设计师或工程师可以据此行动的改进清单，按影响程度排序并附上理由；不要重新设计，也不要只给形容词。

此技能是**可选**目录技能，可按需安装。如何安装、审计、更新、分配和重置目录技能，请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

## 适用场景

- 设计师或工程师请求你评审某个屏幕、原型或线上 UI。
- 某项功能即将发布，有人希望进行最后一轮 UX 检查。
- 怀疑某个流程导致用户流失，希望在埋点研究前先获得初步评估。

### 不适用场景

- 用户想要重新设计——那是设计项目，不是设计评审。
- 工作仍处于早期阶段，没有具体工件可供评审。
- 你不了解用户要完成的任务——先询问用户任务。

## 目录元数据

| 字段 | 值 |
|---|---|
| 目录 ID | `paperclipai:optional:product:design-critique` |
| 规范键 | `paperclipai/optional/product/design-critique` |
| 目录路径 | `catalog/optional/product/design-critique` |
| 类型 | `optional` |
| 类别 | `product` |
| Slug | `design-critique` |
| 入口文件 | `SKILL.md` |
| 信任级别 | `markdown_only` |
| 兼容性 | `compatible` |
| 默认安装 | `false` |
| 建议角色 | `designer`、`product`、`engineer` |
| 依赖项 | — |
| 标签 | `design`、`product`、`ux`、`review` |
| 文件数 | 1 |
| 内容哈希 | `sha256:429f94df398a0697042b5bbe4755b1ff1a230aa5f41d99118ad37493ac65d21c` |
| 软件包 | `@paperclipai/skills-catalog@0.3.1` |

## 文件清单

| 路径 | 类型 | 字节数 |
|---|---|---:|
| `SKILL.md` | `skill` | 4,851 |

## 完整技能定义

下方代码块是此技能完整且权威的 `SKILL.md`，也是智能体运行时实际加载的文件。可以使用代码块右上角的控件进行复制，或将其下载为 `SKILL.md`。

````markdown skill-source
---
name: design-critique
description: Give a structured product design critique — user job clarity, hierarchy, affordance, error states, accessibility, and consistency — focused on what to change, in what order, and why.
key: paperclipai/optional/product/design-critique
recommendedForRoles:
  - designer
  - product
  - engineer
tags:
  - design
  - product
  - ux
  - review
---

# Product Design Critique

A structured critique pass for a screen, flow, or component. The output is a prioritized list of changes a designer or engineer can act on — not adjectives. Critique is not redesign; recommend, do not rebuild.

## When to use

- A designer or engineer asks for feedback on a screen, mock, or live UI.
- A feature is shipping and someone wants a final UX read.
- A flow is suspected of causing user drop-off and you want a pre-research read before instrumentation.

## When not to use

- The user wants a redesign. That is a design project, not a critique.
- The work is so early that no concrete artifact exists. Sketch with them instead of critiquing air.
- You have no context on the user job. Ask for it first; design critique without user context devolves into taste.

## Pre-critique context

Before opening a screen, get:

- **Who is the user.** Specific role and competence, not "users".
- **What job they are doing on this screen.** One sentence.
- **What success looks like.** What the user can do after this screen that they could not before.
- **Where this screen sits in the larger flow.** What precedes and follows.

If any of these is missing, ask. Critique without these is opinion.

## The pass (in order)

1. **Clarity of the user job.**
   - Within 3 seconds of opening, is it obvious what this screen is for?
   - Does the primary action match the user's actual job, or a designer's preferred path?

2. **Visual hierarchy.**
   - The most important thing on the screen should be the most prominent (size, weight, position, color).
   - Secondary actions should look secondary. Tertiary should be findable but not loud.
   - Headings should chunk content into the right groups for the task.

3. **Affordance and signifiers.**
   - Clickable things look clickable.
   - Disabled things look disabled and explain why on hover/focus.
   - Drag, scroll, or swipe interactions are discoverable, not hidden.

4. **States.**
   - Empty state (no data) is designed, not a blank rectangle.
   - Loading state communicates progress, not just spins.
   - Error states say what went wrong and what to do next, in the user's words.
   - Success state confirms without celebrating banal actions.

5. **Inputs and forms.**
   - Labels visible, not just placeholders.
   - Validation runs at the right time (on blur, not on every keystroke unless the user is in a known-format field).
   - Required fields marked.
   - Field order matches the user's mental order, not the database order.

6. **Accessibility.**
   - Sufficient color contrast (WCAG AA at minimum; AAA where reasonable).
   - Focus order is logical for keyboard navigation.
   - Interactive elements are reachable without a mouse.
   - Critical information is not color-only (icons, text, position back it up).
   - Touch targets at least 44×44 px on mobile.

7. **Consistency.**
   - Tokens, components, and patterns match the rest of the product.
   - "Borrowed" patterns from other products are intentional, not accidental drift.

8. **Copy.**
   - Buttons are verbs that name the outcome ("Save changes" beats "Submit").
   - Microcopy explains, does not decorate.
   - Tone matches the product voice.

9. **Edge cases.**
   - Long content (long names, many items, RTL languages).
   - Tiny content (one item, zero items).
   - Slow network and offline behavior.
   - Permissions denied.

## Output format

Group findings by severity, then by category. Each finding is one issue and one suggested fix.

```md
## Design critique: <screen name>

### Must-fix (blocks ship)
- **<category>:** <one-line issue>. **Try:** <one-line suggestion>.

### Should-fix (before broader rollout)
- **<category>:** <one-line issue>. **Try:** <one-line suggestion>.

### Nice-to-fix (when there's room)
- **<category>:** <one-line issue>. **Try:** <one-line suggestion>.

### Strengths to keep
- <one-line thing the design got right>
```

Always include the "strengths to keep" section. It is not flattery — it is signal to the designer about what not to change in the next round.

## Anti-patterns

- "I would do it differently" without saying what or why. That is preference, not critique.
- Long critiques that bury must-fix items under nice-to-haves.
- Suggesting net-new features under the guise of a critique.
- Ignoring user context and grading on taste.
- Treating a critique as approval. State approval explicitly if asked; otherwise critique is feedback, not sign-off.
````

## 中文译文

# 产品设计评审

对页面、流程或组件进行结构化评审。输出应是按优先级排列、设计师或工程师可以执行的改进清单，而不是形容词。评审不是重新设计：给出建议，不要直接重做。

## 适用场景

- 设计师或工程师希望你评审页面、原型或线上 UI。
- 功能即将发布，需要做最后一次 UX 检查。
- 怀疑某个流程导致用户流失，希望在埋点研究前先检查一遍。

## 不适用场景

- 用户要求重新设计。这属于设计项目，不是评审。
- 工作尚处于早期，没有具体产物。应先与用户一起画草图，不要对空想进行评审。
- 不了解用户要完成的工作。先询问；缺少用户上下文的设计评审会退化为个人品味判断。

## 评审前的上下文

查看页面前，应了解：

- **谁是用户。** 具体角色和能力水平，不要只写“用户”。
- **用户在此页面要完成什么。** 用一句话说明。
- **成功是什么样。** 用户离开此页面时能完成什么以前做不到的事。
- **页面在完整流程中的位置。** 页面之前和之后发生什么。

缺少任一项时都应先询问。没有这些信息的评审只是意见。

## 依次进行评审

1. **用户任务是否清晰。**
   - 打开页面 3 秒内，用户是否能明白页面用途？
   - 主要操作是否符合用户实际要完成的任务，而不是设计师偏好的路径？
2. **视觉层级。**
   - 页面最重要的内容应最突出（尺寸、字重、位置、颜色）。
   - 次要操作应显得次要；第三级操作要容易找到，但不能过于醒目。
   - 标题应按任务需要将内容划分为恰当的组。
3. **可供性与提示符。**
   - 可点击元素看起来应可点击。
   - 禁用元素应显得已禁用，并在悬停或获得焦点时说明原因。
   - 拖动、滚动或滑动等交互应容易发现，而不是隐藏起来。
4. **状态。**
   - 空状态（无数据）应经过设计，而不是一块空白矩形。
   - 加载状态应传达进度，而不是只显示旋转图标。
   - 错误状态应使用用户能理解的话说明发生了什么以及下一步怎么做。
   - 成功状态应给予确认，但不要为平常操作过度庆祝。
5. **输入和表单。**
   - 标签应始终可见，不能只依赖占位文字。
   - 验证时机应合理（例如失焦时验证；已知格式字段除外，不要每敲一个字符就验证）。
   - 必填字段应明确标出。
   - 字段顺序应符合用户的思维顺序，而不是数据库顺序。
6. **无障碍。**
   - 颜色对比度至少达到 WCAG AA；合理时应达到 AAA。
   - 键盘导航的焦点顺序应符合逻辑。
   - 交互元素应无需鼠标也能操作。
   - 关键信息不能只靠颜色表示；应由图标、文字或位置等共同传达。
   - 移动端触控目标至少为 `44×44 px`。
7. **一致性。**
   - 设计令牌、组件和模式应与产品其他部分一致。
   - 借鉴其他产品的模式应出于明确目的，而不是意外产生的偏差。
8. **文案。**
   - 按钮使用能说明结果的动词（“保存更改”优于“提交”）。
   - 微文案用于解释，而不是装饰。
   - 语气应符合产品风格。
9. **边界情况。**
   - 内容很长（名称很长、项目很多、RTL 语言）。
   - 内容很少（一个项目、零个项目）。
   - 网络缓慢或离线。
   - 权限被拒绝。

## 输出格式

按严重程度分组，再按类别归类。每条发现只指出一个问题并给出一个建议修复方案。

```md
## 设计评审：<页面名称>

### 必须修复（阻止发布）
- **<类别>：** <一行描述问题>。**建议：** <一行修复建议>。

### 应当修复（扩大发布前）
- **<类别>：** <一行描述问题>。**建议：** <一行修复建议>。

### 可以修复（有余力时）
- **<类别>：** <一行描述问题>。**建议：** <一行修复建议>。

### 应保留的优点
- <一行说明设计做对的地方>
```

始终包含“应保留的优点”部分。这不是恭维，而是告诉设计师下一轮不要改掉哪些有效设计。

## 反面模式

- 只说“我会用另一种做法”，却不说明具体做法或原因。这只是偏好，不是评审。
- 评审过长，把必须修复的问题埋在可选建议下面。
- 借评审之名建议新增功能。
- 忽略用户上下文，仅凭个人品味打分。
- 把评审当成批准。如果用户明确要求批准，就明确说明；否则评审只是反馈，不是签核。

## 另请参阅

- [可选技能](../../optional.md)——按类别整理的全部可选目录技能。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、分配和故障排查。
- [应用内置目录](../../../skills.md#3-app-shipped-catalog)——内置和可选目录技能的版本管理与更新方式。
- [技能指南](../../../../guides/org/skills.md)——界面操作指南和内置目录概览。
