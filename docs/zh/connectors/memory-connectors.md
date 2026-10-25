---
paperclip_version: v2026.1012.0
seo_title: Paperclip 中的记忆连接器
seo_description: 选择一个实验性记忆提供商，启用其设置，并帮助智能体回忆有用的上下文，同时保持访问权限和已保存的指导由您掌控。
---

# 记忆连接器

您可以为智能体提供一个跨任务回忆偏好、决策和早期工作的地方。Paperclip 通过与其他工具连接器相同的访问和操作控制，连接到 **Mem0**、**Zep**、**Supermemory**、**Cognee** 和 **Honcho**。

这些连接器处于实验阶段，默认关闭。打开 **设置 → 实验性功能** 并启用 **记忆连接器**，然后在 **连接器** 中找到该提供商。

关闭此设置会隐藏目录设置并阻止建立新的精选连接。您已保存的连接将继续运行，您可以重新连接或轮换其凭据，而无需重新打开该设置。自定义 MCP 服务器保持其常规行为。

## 选择提供商

| 提供商 | 连接方式 | 准备事项 |
| --- | --- | --- |
| [Mem0](mem0.md) | API 密钥 | 一个项目密钥以及您希望智能体使用的记忆标识符。 |
| [Zep](zep.md) | 浏览器登录 | 一个已配置的 Memory MCP 项目、一个分配的席位，以及由其管理员授权的工作身份。 |
| [Supermemory](supermemory.md) | 浏览器登录 | 在同意过程中要授权的 workspace 和可选标签。 |
| [Cognee](cognee.md) | 云 API Base URL 和密钥 | 一个活动的云 workspace 及其租户 URL。 |
| [Honcho](honcho.md) | API 密钥和 workspace | 一个组织密钥以及智能体应使用的 workspace。 |

您在每个连接的常规 **权限** 页面上管理所有连接。没有单独的记忆页面、自动对话上传或后台记忆同步：智能体通过调用提供商的工具来保存和检索上下文。

## 审查智能体可以执行的操作

活动操作默认为 **允许**，包括记忆写入和删除。若要审查智能体保存的内容，请将写入操作更改为 **先询问**。如果您不希望智能体遗忘已存储的信息，请将删除操作设置为 **关闭**。

Paperclip 将检索分类为读取，将存储和更新分类为写入，将删除或重置分类为破坏性操作。Supermemory 的 `add_memory` 也可以遗忘信息，因此整个操作被分类为破坏性操作。

提供商的凭据、同意和访问规则决定连接可以访问哪些数据。用户标识符、数据集名称、workspace 参数或标签本身并不是由 Paperclip 强制执行的新的隔离边界。当工作必须保持分离时，请使用单独的凭据或提供商访问控制。

## 告知智能体何时记忆

所有五个提供商都在 **智能体指令** 下包含可编辑的指导。它鼓励相关的回忆和简洁、持久的记忆，而不复制整个对话或存储凭据。

您可以编辑文本或关闭 **告知智能体使用 {provider}**。关闭它会保留工具访问权限和您保存的文本。[已保存的连接指令](connection-instructions.md) 解释了指导何时到达智能体以及如何恢复默认设置。

## 检查连接

从预期提供商上下文中的一个已知、无害的记忆开始。要求一个符合条件的智能体在不保存或删除任何内容的情况下检索它，然后将结果与提供商进行比较。

空结果可能意味着上下文错误、存储为空或索引尚未完成。这并不能证明凭据是错误的。如果您需要写入测试，请在测试空间中使用一次性数据，并首先有意选择写入权限。

> **注意：** 建议的检查，而非记录的实时结果。

## 相关指南

- [连接器访问如何工作](access-model.md)
- [设置操作权限](action-permissions.md)
- [已保存的连接指令](connection-instructions.md)
- [实验性功能](../../experimental/overview.md)

## 来源

- [功能默认值](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/feature-catalog.ts) 和 [连接设置服务](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/server/src/services/tool-access.ts) — 默认关闭的设置和新设置门控。
- [记忆工具分类](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/server/src/services/tool-access.ts) — 审查过的操作风险。
