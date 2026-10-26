---
paperclip_version: v2026.1012.0
seo_title: 聘用你的首位智能体
seo_description: 了解智能体的角色、AI 系统、预算和唤醒规则。完成首位智能体的配置，并排查常见问题。
---

# 聘用你的首位智能体

智能体不只是“一个 AI”，而是一套配置：它承担特定角色，由特定 AI 系统驱动，在预算范围内运行，并遵循明确的唤醒与工作规则。

聘用智能体时，你是在告诉 Paperclip：由哪个 AI 系统运行它、它在公司里承担什么角色，以及它需要遵守哪些限制。Claude、Codex 等 AI 系统本身运行在 Paperclip 之外；Paperclip 是管理这些系统的控制层。

CEO 始终是你创建的首位智能体。它负责读取公司目标、提出策略、创建任务，并向下属委派工作。CEO 开始运行之前，公司里的工作不会推进。

---

## 开始之前

你需要：

- 已创建的公司（参见[创建你的第一家公司](./your-first-company.md)）
- 可用于支付 AI 服务费用的方式：登录 Claude 或 ChatGPT 订阅，或使用 Anthropic API 密钥（供 Claude Code 使用）或 OpenAI API 密钥（供 Codex 使用）。获取密钥的方法见[安装指南](./installation.md)。
- **使用 Claude Code 时：** 在 Mac 上安装好 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)。

---

1. **打开 Agents（智能体）页面并点击“New Agent（新建智能体）”**

   在侧边栏点击 **Agents（智能体）**。如果这是你的首位智能体，列表会为空，并提示你创建智能体。点击 **New Agent（新建智能体）**。

   ![空的智能体列表，其中有“新建智能体”按钮](../../../user-guides/screenshots/light/agents/agents-list-empty.png)

2. **为智能体命名**

   系统会打开名为 **Meet your next agent（认识你的下一位智能体）** 的对话框，顶部显示两个步骤：**1. Name（名称）** 和 **2. Adapter（适配器）**。

   在 **Agent name（智能体名称）** 中输入名称（例如“CEO”），然后点击 **Choose adapter（选择适配器）**。

   ![“Meet your next agent”对话框，显示智能体名称输入框](../../../user-guides/screenshots/light/agents/new-agent-name-role.png)

   这里不需要选择角色。每家公司的首位智能体都是 **CEO**，Paperclip 会自动设置。CEO 直接向董事会（也就是你）汇报。之后聘用的智能体会先采用通用角色并向 CEO 汇报；创建后可在各智能体的 **Harness / Runtime（运行环境）** 页面更改这两项设置。

3. **选择适配器**

   适配器会告诉 Paperclip 如何运行智能体。对话框现在会显示 **Choose an adapter（选择适配器）**，每种适配器各有一张卡片。选择一种，然后点击 **Configure agent（配置智能体）**。

   ![新建智能体对话框的第二步“选择适配器”，显示每种可用适配器的卡片](../../../user-guides/screenshots/light/agents/adapter-type-dropdown.png)

   <!-- tabs: Claude Code (推荐), Codex -->

   <!-- tab: Claude Code (推荐) -->

   **Claude Code** 会在你的 Mac 上直接运行 Claude Code 智能体。它可以处理工作区文件、运行终端命令、编写和编辑代码，并代表你调用 Claude。

   这是 Paperclip 智能体功能最强、也最常用的适配器。

   **前置条件：** 你的 Mac 上必须安装 Claude Code。如果尚未安装，请按照 [Claude Code 安装指南](https://docs.anthropic.com/en/docs/claude-code)操作；这是 Anthropic 的独立产品。安装完成后再回来继续。

   <!-- tab: Codex -->

   **Codex** 会在你的 Mac 上直接运行 OpenAI Codex 智能体。与 Claude Code 一样，它可以处理工作区文件并运行命令，但由 OpenAI 的模型驱动。

   **前置条件：** 需要安装 OpenAI Codex CLI。安装说明请查阅 [OpenAI 文档](https://platform.openai.com)。

   <!-- /tabs -->

4. **连接模型提供商**

   对话框关闭后，你会进入设置页面。页面顶部显示尚未启动的智能体形象，左侧列出 **Connect（连接）**、**Configure（配置）** 和 **Confirmation（确认）** 三个步骤。

   在 **Connect（连接）** 步骤中，选择智能体运行的 **Environment（环境）**（刚开始使用默认值即可），然后连接 Claude 或 OpenAI：你可以登录订阅账户，也可以提供 API 密钥。输入 Paperclip 的密钥会作为机密信息保存，不会以明文出现在智能体设置中。

5. **配置并测试**

   在 **Configure（配置）** 步骤中，选择驱动智能体的 **Model（模型）**。Claude Code 可选择 `claude-opus-4-6`，它能力最强，适合 CEO 等策略型角色；`claude-sonnet-4-6` 更快、成本更低，适合日常工作。Codex 使用默认模型即可开始。某些适配器还提供 **Thinking effort（思考强度）** 设置；不确定时可保留 **Auto（自动）**。

   ![Claude Code 智能体的 Harness / Runtime 页面，显示适配器卡片、模型和思考强度设置](../../../user-guides/screenshots/light/agents/claude-local-config-filled.png)

   *截图展示的是已创建智能体在 **Harness / Runtime（运行环境）** 页面中的模型和思考强度设置；设置完成后也可在此修改。*

   页面底部的 **Test your agent（测试智能体）** 卡片会检查运行环境和模型能否响应。点击 **Run test（运行测试）**。如果显示 **Connection successful（连接成功）**，就可以继续。

6. **完成设置**

   点击 **Finish setup（完成设置）**。Paperclip 会创建智能体，并显示 **Your agent is ready（智能体已就绪）**，其中列出它将使用的适配器、模型和环境。

   点击 **Edit configuration（编辑配置）** 可打开智能体设置；也可以点击 **Assign CEO a Task（为 CEO 分配任务）**（按钮中会显示你的智能体名称），给它第一个工作。你也可以从 **Agents（智能体）** 列表打开它。此时状态应为 `idle`：配置已完成，但尚未运行过心跳。

   ![智能体概览页面，状态为 idle](../../../user-guides/screenshots/light/agents/agent-detail-idle.png)

7. **决定心跳计划**

   新智能体默认关闭定时心跳。收到任务或点击 **Run now（立即运行）** 时，它仍会被唤醒；只是不会按定时计划运行。准备好让 CEO 开始工作后，可在下一篇指南中启用定时器。

   设置位于智能体 **Harness / Runtime（运行环境）** 页面的 **Run Policy（运行策略）** 卡片：打开 **Heartbeat on interval（按间隔运行心跳）**，并设置 **Run heartbeat every … sec（每隔……秒运行一次心跳）**。

   ![Harness / Runtime 页面的运行策略卡片，显示按间隔运行心跳的开关](../../../user-guides/screenshots/light/agents/budget-and-heartbeat-fields.png)

   > **建议：** 初期不要把间隔设得太短，每次心跳都会产生费用。CEO 每小时运行一次（3600 秒）是不错的起点；了解公司的工作节奏后再加快频率。

8. **考虑预算**

   设置流程不会创建支出上限。智能体获得预算策略后，可在 **Audit → Budgets（审计 → 预算）** 中管理；创建预算需使用 CLI 或 API（参见[智能体：预算](../org/agents.md#budgets)）。[成本与预算](../day-to-day/costs.md)介绍了智能体预算与公司预算如何共同生效。

   > **费用提示：** CEO 通常是最活跃的智能体——每次心跳都会运行，而且推理任务比工作型智能体更复杂。建议为 CEO 设置稍高的预算。每月 30–50 美元是合理的起点，之后随时可以调整。

   请记住，智能体预算和公司预算是分开的；两者都会生效。任一上限达到时，智能体都会暂停。

9. **需要时重新测试**

   你可以随时重新运行检查。打开智能体，进入 **Harness / Runtime（运行环境）**，滚动到页面底部的 **Test your agent（测试智能体）** 卡片，然后点击 **Run test（运行测试）**（如果以前测试过，则点击 **Test again（再次测试）**）。

   Paperclip 会检查运行环境（对 Claude Code 而言，会确认已安装且可访问），验证连接，并等待模型响应。

   ![测试智能体卡片显示连接成功](../../../user-guides/screenshots/light/agents/test-environment-success.png)

   测试成功即可开始使用。如果失败，卡片会显示 **Couldn't connect（无法连接）** 或 **Connection needs attention（连接需要处理）**，并附上详细信息：

   ![测试智能体卡片显示连接失败和错误信息](../../../user-guides/screenshots/light/agents/test-environment-fail.png)

   请参阅下方的故障排查部分。

---

## 故障排查

**“Test your agent（测试智能体）”失败**

最常见的两个原因：

- 提供商连接有误——检查输入的 API 密钥（不要包含多余空格），或重新登录订阅账户。对于已创建的智能体，可在 **Harness / Runtime（运行环境）** 页面的 **Environment variables（环境变量）** 中设置 `ANTHROPIC_API_KEY` 或 `OPENAI_API_KEY`。
- 尚未安装 Claude Code，或安装路径与 Paperclip 预期不符——请先确认不通过 Paperclip 也能在 Mac 上独立打开 Claude Code。

**智能体首次心跳后显示 `error` 状态**

在智能体页面的侧栏菜单中，点击 **Audit（审计）** 下的 **Runs（运行记录）**。打开该智能体的运行记录列表，再点击最近一次运行，即可查看完整记录。错误信息会说明原因。

**预算一开始就接近 100%**

这通常是因为模型名称无效，API 返回的错误响应仍计入用量；也可能是关联的 API 密钥没有可用额度。请确认模型名称与 AI 提供商支持的名称完全一致，并在提供商的账单页面检查账户余额。

---

CEO 智能体已配置完成。下一篇指南将介绍如何启用心跳，并观察它首次自主工作的过程。

[观察智能体工作 →](./watching-agents-work.md)
