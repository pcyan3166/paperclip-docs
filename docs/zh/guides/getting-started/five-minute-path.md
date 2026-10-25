---
seo_title: 五分钟快速上手：创建你的首位智能体
seo_description: 安装 Paperclip、创建公司、聘用 CEO 智能体并批准它的首个策略，大约五分钟即可让 AI 公司开始运转。
paperclip_version: v2026.1012.0
---

# 五分钟快速上手

这是从 Paperclip 已经启动，到智能体为你完成工作的最短路径。安装完成后，整个流程约需 5 分钟：创建公司、聘用 CEO 智能体、批准它的首个策略，然后观察任务出现在看板上。

![Paperclip 仪表板，显示实时智能体卡片、预算事件横幅、月度支出摘要卡片和近期活动](../../../user-guides/screenshots/light/dashboard/dashboard-overview.png)

---

## 简要说明

Paperclip 公司是一个独立的 AI 组织，包含一个目标、一支智能体团队、一个任务看板和一份预算。CEO 是你聘用的首位智能体。它会读取公司的目标并提出策略；你批准策略后，它便开始创建任务，并在看板上推进任务。

接下来几分钟里，你会接触到公司、智能体、任务和心跳这四个概念。完整说明见[关键概念](../welcome/key-concepts.md)，开始之前不必先读完。

---

## 开始前：安装 Paperclip

安装 Paperclip，并准备好 AI 服务提供商的密钥。这是一次性准备，不计入五分钟；根据你已有的环境，大约需要 3–10 分钟。

- 准备一台装有 Node.js 24 或更高版本的电脑；终端安装只需一条命令。
- 准备 [Anthropic](https://console.anthropic.com) 的 API 密钥（用于 `claude_local`），或 [OpenAI](https://platform.openai.com) 的 API 密钥（用于 `codex_local`）。安装指南会介绍如何获取密钥。
- 使用 `claude_local` 适配器时，同一台电脑上还需安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)。请在开始本流程前安装，因为第 2 步会用到它。

> **费用提示：** 智能体调用 API 会产生费用。体验产品可预留 5–20 美元；活跃公司的费用通常为每月 20–100 美元。启用心跳前，请为智能体和公司设置预算；达到预算上限时，Paperclip 会自动暂停智能体。

[查看安装步骤 →](./installation.md)

---

## 五分钟流程

Paperclip 启动后还需三步。总计约 5–11 分钟。

| 步骤 | 页面 | 完成后你将拥有 | 大约用时 |
|------|------|----------------|----------|
| 1 | [创建你的第一家公司](./your-first-company.md) | 一个已命名的公司（目标稍后添加） | 1–2 分钟 |
| 2 | [聘用你的首位智能体](./your-first-agent.md) | 一个状态为 `idle` 的已配置 CEO 智能体 | 2–4 分钟 |
| 3 | [观察智能体工作](./watching-agents-work.md) | 已触发一次心跳、已提交策略审批，并在看板上看到首批任务 | 2–5 分钟 |

请按顺序阅读这些步骤。每一步都刻意写得很短。

---

## 流程结束时你会看到什么

完成第 3 步后，你将拥有：

- **Companies（公司）** 页面上的一个公司，其中包含你设定的目标。
- **Agents（智能体）** 页面上的 CEO 智能体。它的状态为 `idle`，且已启用心跳。
- CEO 创建的一项待处理 **Strategy（策略）** 审批。
- 批准策略后，**Issues（问题/任务）** 页面上会出现若干 `todo` 或 `backlog` 任务。
- **Agents → CEO → Runs（运行记录）** 下的一份运行记录，完整展示 CEO 首次心跳期间所做的事情。

至此，一个能自主工作的公司就已运行。接下来可以聘用更多智能体、调整目标，或亲自分派任务。

---

## 接下来可以做什么

完成这三个步骤后：

- **聘用第二位智能体**——CEO 可以通过[审批](../../../guides/day-to-day/approvals.md)流程申请聘用下属。角色设计请参阅[智能体](../../../guides/org/agents.md)，汇报关系请参阅[组织结构](../../../guides/org/org-structure.md)。
- **设置实际工作流程**——阅读[任务](../../../guides/day-to-day/issues.md)了解任务生命周期；阅读[成本与预算](../../../guides/day-to-day/costs.md)了解费用控制；阅读[心跳与例程](../../../guides/projects-workflow/routines.md)了解定时运行。
- **连接代码仓库**——[执行工作区](../../../guides/projects-workflow/workspaces.md)指南会介绍如何让智能体在实际项目中工作。

如果页面上的界面元素让你感到陌生，可以快速查阅[关键概念](../welcome/key-concepts.md)和[术语表](../welcome/glossary.md)。

[创建你的第一家公司 →](./your-first-company.md)
