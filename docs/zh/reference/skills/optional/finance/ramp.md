---
paperclip_version: v2026.707.0
seo_title: Ramp 技能
seo_description: 通过对 Ramp 官方发布的智能体指令进行轻量封装，让智能体从 Paperclip 配置并使用 Ramp，而非依赖内置副本。
---

# Ramp

> 在 Paperclip 中获取并遵循 Ramp 发布的智能体操作指南，涵盖账户开户、卡片和支出控制；所有财务操作均受 Paperclip 审批关卡和默认拒绝的主机允许列表保护。

当公司希望智能体从 Paperclip 配置或使用 [Ramp](https://ramp.com) 时，可使用此技能。此技能仅对 Ramp 自己发布的智能体指令进行轻量封装：它不会将 Ramp 操作指南的副本内置到智能体中，而是在任务开始时获取 Ramp 当前的指令，并在 Paperclip 治理规则下执行。这能让智能体始终使用 Ramp 的最新指南，同时确保未经 Paperclip 审批，不会转移资金、创建账户或安装第三方工具。

此技能是**可选**目录技能，可按需安装。如何安装、审计、更新、分配和重置目录技能，请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

## 适用场景

- 公司希望申请 Ramp 账户或完成开户流程。
- 智能体需要处理 Ramp 卡片、支出控制或报告。
- 你希望在 Paperclip 监督下遵循 Ramp 的官方配置指南（get-started、apply-to-ramp、incorporate-with-ramp）。

### 不适用场景

- 任务与 Ramp 无关——此技能只封装 Ramp 自己的操作指南。
- 你希望智能体自动转移资金或提交法律/财务文件——这些操作始终需要人工批准；没有批准时，此技能会默认拒绝执行。

## 工作方式

此技能严格限定指令来源，以及智能体可自行执行的操作。

**始终获取实时版本。** 任务开始时，智能体会拉取 Ramp 当前的指令，而不依赖记忆或复制的版本，从而遵循 Ramp 最新的官方操作指南。

**只信任 Ramp 自己的主机。** 获取内容仅限允许列表中的来源，包括 Ramp 的 `get-started` 技能、操作指南目录、技能索引和 Ramp CLI 仓库。镜像站、URL 缩短服务、搜索摘要、用户粘贴的替代链接或未固定版本的第三方仓库均会被拒绝。由于公开操作指南目录在同一主机上混合了 Official 和 Community 条目，且没有来源标记，此技能只将目录用于发现内容：只会自动继续官方配置流程，以及用户或任务明确指定的操作指南；来源不明确的内容会被视为不可信示例。如果无法确认来源，技能就会停止。

**所有获取到的指令都从属于 Paperclip。** Ramp 的操作指南仅作为参考，不能覆盖系统、公司、智能体或任务指令。

## 强制审批关卡

即使 Ramp 的操作指南表示用户可以继续，此技能也不会自动批准支出或法律/财务操作。智能体执行以下操作前必须获得 Paperclip 审批：

- 申请 Ramp 账户或提交公司开户资料。
- 注册公司、设立实体、启动 EIN 相关流程、接受法律协议或提交任何备案文件。
- 通过从网络管道执行的 shell 安装程序安装或更新 Ramp CLI。
- 安装第三方浏览器自动化工具、MCP 服务器、CLI 或连接器，或向其验证身份、授予凭据；操作指南中引用的工具也包括在内。
- 代表用户登录 Ramp、连接 Ramp 账户，或在运行可能暴露公司财务数据时授权连接器。

审核者可以在随附的目录条目中查看来源模型和这些安全规则，因此安装前即可了解智能体获准执行的操作。如何申请并完成审批，请参阅[审批](../../../../guides/day-to-day/approvals.md)。

## 目录元数据

| 字段 | 值 |
|---|---|
| 目录 ID | `paperclipai:optional:finance:ramp` |
| 规范键 | `paperclipai/optional/finance/ramp` |
| 目录路径 | `catalog/optional/finance/ramp` |
| 类型 | `optional` |
| 类别 | `finance` |
| Slug | `ramp` |
| 入口文件 | `SKILL.md` |
| 信任级别 | `markdown_only` |
| 兼容性 | `compatible` |
| 默认安装 | `false` |
| 建议角色 | `finance`、`operations`、`founder`、`engineer` |
| 依赖项 | — |
| 标签 | `ramp`、`finance`、`spend`、`approvals`、`agent-cards` |

## 另请参阅

- [可选技能](../../optional.md)——完整的自选目录索引。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、分配和故障排查。
- [审批](../../../../guides/day-to-day/approvals.md)——智能体如何请求受审批限制的操作，以及人工如何批准。
