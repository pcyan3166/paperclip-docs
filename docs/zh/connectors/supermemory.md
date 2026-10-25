---
paperclip_version: v2026.1012.0
seo_title: Supermemory 连接器
seo_description: 通过浏览器登录连接 Supermemory，选择工作区及其可访问的标签，并在 Paperclip 中审查记忆操作和已保存的智能体指引。
---

# Supermemory

您可以授权智能体在 Supermemory 工作区中搜索和保存记忆。在浏览器登录期间，请选择与工作匹配的读取或写入访问权限以及可选标签。

## 连接前准备

- 在 **设置 → 实验性功能** 下启用 **记忆连接器**。该功能默认关闭；请参阅 [记忆连接器](memory-connectors.md)。
- 一个具有访问权限的 Supermemory 账户，用于智能体将使用的工作区。
- 关于此连接应访问哪些空间及可选标签的决定。

托管的 MCP 连接使用浏览器登录。开发者 API 密钥是独立的凭据，此设置不使用该凭据。

## 连接 Supermemory

1. 打开 **连接器** 并选择 **Supermemory**。
2. 阅读访问摘要。此方法使用您的个人登录；选择 **更改** 以限制哪些智能体可以使用它。
3. 完成 **使用 Supermemory 登录**，并审查提供商提供的工作区、读取或写入访问权限以及可选标签。
4. 打开 **权限** 并审查操作和 **智能体指令**。

Paperclip 连接到 `https://mcp.supermemory.ai/mcp`。对于此流程，您无需注册单独的 OAuth 应用。

## 选择访问权限

Supermemory 的同意控制决定了此连接可访问的数据。Paperclip 的操作开关无法扩大该同意范围。智能体提供的标签不能替代提供商的授权。

活动操作默认状态为 **允许**，包括写入和具备删除能力的操作。`add_memory` 操作可以保存或遗忘信息，因此 Paperclip 将其归类为破坏性操作。如果您希望审查或阻止这些更改，请将其设置为 **先询问** 或 **关闭**。

默认指引告知智能体在授权的工作区和标签范围内进行回忆和保存。您可以在 **智能体指令** 下编辑或禁用该指引；请参阅 [已保存的连接指令](connection-instructions.md)。

## 尝试使用

要求符合条件的智能体在已同意的工作区和标签内搜索一条已知的、无害的记忆，且不保存或遗忘任何内容。将结果与 Supermemory 进行比较，并检查连接器调用。

> **注意：** 这是建议的测试，而非记录的实时结果。只读同意并不能确保记忆写入功能可用。

## 故障排查与限制

| 问题 | 检查项 |
| --- | --- |
| 目录中缺少 Supermemory | 启用 **记忆连接器**。 |
| 写入被拒绝 | 检查提供商同意和 Paperclip 操作权限。 |
| 标签被拒绝 | 在 Supermemory 中重新授权预期的标签；不要扩大查询范围以绕过同意。 |
| 搜索无结果 | 检查工作区、标签以及是否存在相关记忆。 |

关闭实验性设置后，已保存的连接将继续运行，并允许重新连接。Paperclip 不会自动将任务对话复制到 Supermemory。

## 相关指南

- [记忆连接器](memory-connectors.md)
- [设置操作权限](action-permissions.md)
- [Supermemory MCP 文档](https://supermemory.ai/docs/supermemory-mcp/mcp)

## 来源

- [Supermemory 定义](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/app-definitions/supermemory.json) — 个人 OAuth 登录、端点、同意指引和默认指令。
- [功能默认值](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/feature-catalog.ts) — 实验性可用性。
