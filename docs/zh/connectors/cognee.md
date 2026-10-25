---
paperclip_version: v2026.1012.0
seo_title: Cognee 云记忆连接器
seo_description: 使用租户 URL 和 API 密钥连接活跃的 Cognee 云工作区，然后通过 Paperclip 选择智能体如何回忆、保存或遗忘记忆。
---

# Cognee

您可以为智能体在 Cognee 云工作区中提供共享记忆。Paperclip 使用内置的云客户端来暴露三个操作：`remember`、`recall` 和 `forget`。

## 连接前准备

- 在 **设置 → 实验性功能** 下启用 **记忆连接器**。该功能默认关闭；请参阅 [记忆连接器](memory-connectors.md)。
- 一个拥有订阅的活跃 Cognee 云工作区。
- 来自 [Cognee 的 API 密钥页面](https://platform.cognee.ai/api-keys) 的 **API 基础 URL** 和 API 密钥。

此连接无需安装本地 MCP 运行时。内置客户端也适用于公共部署。

## 连接 Cognee

1. 打开 **连接器** 并选择 **Cognee**。
2. 阅读主按钮上方的访问权限说明。选择 **更改** 以选择个人凭据或限制可使用该凭据的智能体范围。
3. 输入 **API 基础 URL** 和 **Cognee API 密钥**，然后完成设置。
4. 审查 **权限** 和 **智能体指令**。

在表单中使用形式为 `https://your-tenant.aws.cognee.ai` 的 HTTPS 租户源，不包含 API 路径、查询参数或自定义端口。Paperclip 会验证该源并安全地保存这两个值。设置过程会在完成前检查云访问权限。

## 选择访问权限

| 操作 | 分类 | 功能描述 |
| --- | --- | --- |
| `recall` | 读取 | 检索相关记忆，可选择使用命名数据集或会话。 |
| `remember` | 写入 | 将文本存储在数据集或会话中。 |
| `forget` | 破坏性 | 删除数据集，或在明确请求时删除所有拥有的记忆。 |

活跃操作默认状态为 **允许**。如果您希望限制访问范围，可对保存的记忆使用 **先询问**，对删除操作使用 **关闭**。

云密钥决定了可访问的数据范围。数据集或会话名称是调用的上下文，而非由 Paperclip 强制执行的新的隔离边界。当工作必须保持分离时，请使用提供商权限或单独的凭据。

您可以在 **智能体指令** 下编辑或禁用伴随此连接的指导说明。请参阅 [已保存的连接指令](connection-instructions.md)。

## 试用

要求符合条件的智能体使用 `recall` 在命名数据集中处理一个已知的无害事实，而不调用 `remember` 或 `forget`。将答案与 Cognee 中的该数据集进行比较。

> **注意：** 这是建议的测试，而非记录的实时结果。索引是异步运行的，因此写入后立即回忆可能会在数据集就绪前返回 HTTP 409 错误。

## 故障排查与限制

| 问题 | 检查项 |
| --- | --- |
| 目录中缺少 Cognee | 启用 **记忆连接器**。 |
| 租户 URL 被拒绝 | 复制 API 基础 URL，仅使用其 HTTPS `*.aws.cognee.ai` 源。 |
| 设置失败 | 检查工作区订阅和 API 密钥。 |
| 回忆提示索引未完成 | 等待 Cognee 完成处理，然后重试检索。 |
| 缺少更广泛的 Cognee 包中的某个工具 | 此连接仅暴露上述三个经过审查的操作。 |

关闭实验性设置后，已保存的连接将继续运行，并允许凭据轮换。Paperclip 不会自动上传对话或暴露 Cognee 的更广泛管理工具。

## 相关指南

- [记忆连接器](memory-connectors.md)
- [设置操作权限](action-permissions.md)
- [Cognee 云文档](https://docs.cognee.ai/cognee-cloud/connections/cloud-mcp)

## 来源

- [Cognee 定义](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/app-definitions/cognee.json) — 凭据字段和默认指令。
- [内置云客户端](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/server/src/services/cognee-connection.ts) 和 [工具连接服务](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/server/src/services/tool-access.ts) — 经过审查的工具和设置行为。
