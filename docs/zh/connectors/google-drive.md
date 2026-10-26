---
paperclip_version: v2026.1005.0
seo_title: Google Drive 连接器
seo_description: 允许智能体搜索和读取 Drive 文件，并可选择创建或复制文件。不暴露删除和共享功能。包含群组、作用域和读取测试。
---

# Google Drive

> **警告：** **Google 验证待定。** Paperclip 尚未完成 Google 应用验证。在授权过程中，您可能会看到未验证应用的警告。如果 Google 提供 **Advanced**（高级）选项以继续前往 Paperclip，在审查所请求的访问权限后，您可以选择继续。并非所有账户都提供此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果您无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：** 在验证待定期间，Google Workspace 连接器以及任何已保存的 Google 账户会暂时从 **Connectors**（连接器）页面隐藏。这仅将其从列表中隐藏。现有的 Google 连接将继续运行，其访问权限和权限保持不变，需要 Google 服务的智能体仍可在其任务中通过连接卡片向您请求该服务。

智能体可以搜索 Drive，读取文件元数据和内容，并在创建连接时添加新文件或复制现有文件。

删除、移动、重命名以及更改文件共享对象的操作**不**由该连接器暴露，无论属于哪个能力组。

> **警告：** 在工具运行之前，Google Drive 需要 Google Workspace Developer Preview 注册。Google 必须注册用于登录的 Workspace 电子邮件，以及如果您自带 OAuth 客户端，则需注册拥有该 OAuth 客户端的 Cloud 项目。请先在 [Google Workspace Developer Preview](https://developers.google.com/workspace/preview) 处申请。

## 连接前准备

- 一个能够打开您希望智能体使用的文件的 Google Workspace 账户。
- 该账户的 Developer Preview 注册，并已由 Google 确认。
- 如果您的实例未注册 Paperclip Cloud，您将需要拥有自己的 Google OAuth 客户端，并启用 **Drive API** 和 **Drive MCP API**。[Set up your own Google OAuth app](google-setup.md) 是完整的流程——请在开始此处操作之前完成。

## 选择能力组

| 群组 | 智能体可以执行的操作 | 请求的作用域 |
| --- | --- | --- |
| **仅读取** | 搜索文件，读取元数据和内容，列出最近文件，读取文件的权限 | `drive.readonly` |
| **读取与创建** | 上述操作，外加创建新文件和复制现有文件 | `drive.readonly`, `drive.file` |

能力组在连接的生命周期内固定不变；若要更改，需建立新的连接。

## 连接 Google Drive

1. 打开 **Connectors** 并选择 **Google Drive**。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他登录方法，请选择 **Change**（更改）。
3. 检查能力组。设置从可以执行更改的群组开始；若要仅读取连接，请选择 **Change** 并在 **What should Paperclip be able to do?**（Paperclip 应该能够做什么？）下选择该群组。当您的实例提供时，Paperclip 使用 **Connect with Paperclip**（通过 Paperclip 连接）。若要改用您自己的客户端，请选择 **Use your own Google OAuth app**（使用您自己的 Google OAuth 应用）并提供来自 [Set up your own Google OAuth app](google-setup.md) 的客户端 ID 和密钥。
4. 使用已注册的 Workspace 账户完成 Google 的同意屏幕。

## 选择访问权限

文件访问范围来自 Google。连接可以看到授权账户已经能够打开的内容：其拥有的文件、与其共享的文件，以及其作为成员的共享云盘内容。Paperclip 没有文件或文件夹选择器，因此缩小访问权限意味着在 Drive 中更改共享设置，或使用权限更有限的账户进行授权。

已审查的操作：

| 操作 | 能力组 |
| --- | --- |
| `search-files`, `list-recent-files` | 两者 |
| `get-file-metadata`, `read-file-content`, `download-file-content` | 两者 |
| `get-file-permissions` | 两者 |
| `create-file`, `copy-file` | 仅限 **读取与创建** |

有两个值得注意的区别。元数据和内容是独立的操作，因此智能体可以列出它无法有效解析的文件。并且 `get-file-permissions` 读取共享列表——它不更改该列表。

> **注意：** “读取与创建”能力组的 `drive.file` 作用域仅覆盖应用本身创建或打开的文件，因此可以创建和复制文件，但不能编辑任意现有文件。连接器的指导原则是文件创建和复制应获得批准；请将它们保持在 **Ask first**（先询问）状态。

[连接器访问机制说明](access-model.md) 涵盖了身份和智能体选择。

## 尝试使用

```txt
Search my Google Drive for a document containing "quarterly plan" and tell me its title, owner, and last modified date. Do not open or change anything else.
```

预期结果是您能在 Drive 中自行找到的文件的元数据。这在不写入任何内容的情况下确认了凭据、账户和智能体的权限。

> **注意：** 这是示例任务，而非记录的测试结果。

## 故障排除与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 在同意屏幕之前 Google 拒绝 | Developer Preview 注册未完成 | 完成注册并重试 |
| 未提供 **Connect with Paperclip** | 实例未注册 Paperclip Cloud，或 Cloud 未提供 Drive 配置文件 | 使用您自己的 Google OAuth 应用 |
| 搜索未找到预期的内容 | 文件未与授权账户共享，或位于其不属于的共享云盘中 | 共享该文件或将该账户添加到共享云盘 |
| 文件已列出但内容无法读取 | 文件类型没有可提取的文本，或它是指向外部项目的链接 | 检查文件类型；元数据和内容是独立的能力 |
| 缺少创建或复制功能 | 连接是以 **仅读取** 方式建立的 | 建立 **读取与创建** 连接 |
| 智能体无法删除或重新共享文件 | 预期行为——这些操作未暴露 | 在 Drive 中执行 |
| **Needs attention**（需要关注） | Google 令牌已过期或被撤销 | 选择 **Reconnect**（重新连接） |

限制：一个连接覆盖一个 Google 账户。不支持删除、移动、重命名或共享更改。不支持编辑任意现有文件；若要编辑文档，请使用 [Google Docs](google-docs.md)、[Google Sheets](google-sheets.md) 或 [Google Slides](google-slides.md)。适用 Developer Preview。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/google-drive/)

- [Google Docs](google-docs.md)、[Google Sheets](google-sheets.md)、[Google Slides](google-slides.md) — 编辑特定文件类型内的内容。
- [Google Workspace 搜索](google-workspace-search.md) — 跨 Gmail、Drive、Calendar 和 Chat 的单一只读搜索。
- [连接器访问机制说明](access-model.md)
- [Google Drive API MCP reference](https://developers.google.com/workspace/drive/api/reference/mcp)
