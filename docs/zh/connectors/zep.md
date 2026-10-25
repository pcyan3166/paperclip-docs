---
paperclip_version: v2026.1012.0
seo_title: Zep 记忆连接器
seo_description: 将 Zep Memory MCP 与您的工作身份连接，选择智能体访问权限，并使用由您的 Zep 管理员授权的个人记忆或共享图谱。
---

# Zep

您可以让智能体从您连接的 Zep 身份中回忆决策和偏好，包括该身份被允许使用的共享图谱。请使用您的 Zep 管理员配置的工作账户登录。

## 连接前准备

- 在 **设置 → 实验性功能** 下启用 **记忆连接器**。该功能默认关闭；请参阅 [记忆连接器](memory-connectors.md)。
- 需要一个已启用 Memory MCP 的 Zep 项目、可用的 MCP 席位，以及由其管理员配置的 Google Workspace 或企业 OIDC。
- 分配给该工作身份对该项目及您所需的任何共享图谱的访问权限。

Zep 项目 API 密钥无法为此托管 Memory MCP 端点进行身份验证。

## 连接 Zep

1. 打开 **连接器** 并选择 **Zep**。
2. 阅读访问摘要。此方法使用您的个人登录；选择 **更改** 以限制哪些智能体可以使用它。
3. 使用分配的工作身份完成 **使用 Zep 登录**。
4. 审查 **权限** 和提供方的 **智能体指令**。

Paperclip 使用托管 MCP 端点 `https://api.getzep.com/mcp`。您无需为此流程注册单独的 OAuth 应用。

## 选择访问权限

Zep 控制该身份的记忆和共享图谱访问权限。Paperclip 控制哪些符合条件的智能体可以调用暴露的工具，以及每个操作是 **允许**、**先询问** 还是 **关闭**。

活动操作默认为 **允许**。如果您希望审查保存的记忆，请对写入操作选择 **先询问**。不要使用其他图谱或身份作为缺少访问权限的变通方法；请直接在 Zep 中授权所需的上下文。

您可以编辑或禁用连接的回忆和保存指导，而无需移除其工具。请参阅 [已保存的连接指令](connection-instructions.md)。

## 试用

要求一个符合条件的智能体在已连接身份的记忆或特定授权的共享图谱中查找一个已知的决策。要求它返回源上下文，而不添加或更改记忆，然后将结果与 Zep 进行比较。

> **注意：** 这是建议的测试，而非记录的实时结果。最近添加的剧集（episodes）可能在由其衍生的图谱完成处理之前即可用。

## 故障排除与限制

| 问题 | 检查项 |
| --- | --- |
| 目录中缺少 Zep | 启用 **记忆连接器**。 |
| 登录没有项目访问权限 | 与项目管理员检查分配的身份、MCP 席位和身份提供商设置。 |
| 共享图谱不可用 | 检查该图谱在 Zep 中的授权情况。 |
| 缺少最近的记忆 | 检查源剧集以及图谱处理是否已完成。 |

关闭实验性设置会保留已保存的连接继续运行，并允许重新连接。Paperclip 不会自动将对话上传到 Zep。

## 相关指南

- [记忆连接器](memory-connectors.md)
- [连接器访问权限的工作原理](access-model.md)
- [Zep Memory MCP 文档](https://help.getzep.com/memory-mcp-server)

## 来源

- [Zep 定义](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/app-definitions/zep.json) — 个人 OAuth 登录、先决条件、端点和默认指导。
- [功能默认值](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/feature-catalog.ts) — 实验性可用性。
