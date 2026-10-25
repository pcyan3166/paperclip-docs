---
paperclip_version: v2026.1012.0
seo_title: Honcho 记忆连接器
seo_description: 通过 API 密钥和工作区连接 Honcho，为选定的智能体提供访问权限，并在它们回忆或保存记忆时保持对等方和会话上下文的清晰。
---

# Honcho

您可以让智能体在 Honcho 中回忆和保存对话上下文。在 Paperclip 中选择工作区，然后在任务中指定对等方或会话，当该上下文重要时进行标识。

## 连接前准备

- 在 **设置 → 实验性功能** 下启用 **记忆连接器**。默认情况下此项处于关闭状态；请参阅 [记忆连接器](memory-connectors.md)。
- 来自 [Honcho 仪表板](https://app.honcho.dev) 的组织信息和 API 密钥。
- 智能体应使用的工作区 ID。

## 连接 Honcho

1. 打开 **连接器** 并选择 **Honcho**。
2. 阅读主按钮上方的访问权限说明行。选择 **更改** 以选择个人凭据或限制哪些智能体可以使用该凭据。
3. 输入 **Honcho API 密钥** 和必需的 **Honcho 工作区**，然后完成设置。
4. 打开 **权限** 并审查暴露的操作和 **智能体指令**。

Paperclip 连接到 `https://mcp.honcho.dev`。对于声明了 `workspace_id` 参数的工具，Paperclip 会提供您配置的工作区，而不是允许智能体替换它。

## 选择访问权限

Honcho 的 API 密钥和提供商访问规则决定了可以访问哪些数据。保存的工作区提供调用上下文；这并不意味着该密钥本身无法访问其他工作区。

活动操作默认状态为 **允许**，包括写入和具备删除能力的工具。在 **权限** 中将写入操作设置为 **先询问**，并将不需要的操作设置为 **关闭**。

默认指令告知智能体使用配置的工作区以及任务中的对等方或会话，并在保存前澄清缺失的记忆目标。您可以编辑或禁用该指导；请参阅 [已保存的连接指令](connection-instructions.md)。

没有保存工作区的旧连接将保留其工具和手动工作区参数。在您完成 **Honcho 工作区** 配置之前，它不会收到固定的连接指导。更改保存的工作区也会使等待来自前一个工作区参数的批准失效。

## 试用

为符合条件的智能体提供工作区预期的对等方和会话上下文，并要求它检索一条已知的、无害的消息，而不创建或更改任何内容。将结果与 Honcho 进行比较，并检查连接器调用。

> **注意：** 这是建议的测试，而非记录的实时结果。

## 故障排除与限制

| 问题 | 检查项 |
| --- | --- |
| Honcho 未出现在目录中 | 启用 **记忆连接器**。 |
| 新设置无法完成 | 除了密钥外，还需输入必需的 **Honcho 工作区**。 |
| 旧连接有工具但没有指令 | 完成其保存的工作区配置。 |
| 检索返回错误的上下文 | 检查配置的工作区以及任务中指定的对等方或会话。 |
| 调用失败 | 检查密钥及其提供商端的访问权限。 |

关闭实验性设置会使已保存的连接继续运行，并允许重新连接。Paperclip 不会根据智能体的身份创建自动记忆范围，也不会后台上传对话。

## 相关指南

- [记忆连接器](memory-connectors.md)
- [连接器访问权限工作原理](access-model.md)
- [Honcho MCP 文档](https://honcho.dev/docs/v3/guides/integrations/mcp)

## 来源

- [Honcho 定义](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/app-definitions/honcho.json) — 密钥、工作区字段、端点和默认指令。
- [连接指令上下文](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/connection-instructions.ts) 和 [托管工作区参数](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/server/src/services/honcho-connection.ts) — 必需配置和托管工作区参数。
