---
paperclip_version: v2026.1005.0
seo_title: Google Docs 连接器
seo_description: 允许智能体读取 Google Docs 文档，并可选择更新它们。涵盖能力组、文档范围、编辑涵盖的内容、读取测试以及故障排查。
---

# Google Docs

> **警告：** **Google 验证待处理。** Paperclip 尚未完成 Google 应用验证。在授权过程中，您可能会看到未验证应用的警告。如果 Google 提供 **Advanced**（高级）选项以继续前往 Paperclip，在审查所请求的访问权限后，您可以选择继续。并非所有账户都提供此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果您无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：** 在验证待处理期间，Google Workspace 连接器及任何已保存的 Google 账户将从 **Connectors**（连接器）页面暂时隐藏。这仅将其从列表中隐藏。现有的 Google 连接将继续运行，其访问权限和权限保持不变，需要 Google 服务的智能体仍可在其任务中通过连接卡片向您请求该服务。

智能体可以读取 Google Docs 文档的文本和结构，并在编辑连接中更新它们。

> **警告：** Google Docs 在授权前需要 Google Workspace Developer Preview 注册。Google 必须注册登录的 Workspace 电子邮件，如果您使用自己的 OAuth 客户端，还必须注册拥有该 OAuth 客户端的 Cloud 项目。请先在 [Google Workspace Developer Preview](https://developers.google.com/workspace/preview) 处申请。

## 连接前准备

- 一个能够打开您希望智能体使用的文档的 Google Workspace 账户。
- 该账户的 Developer Preview 注册，并已由 Google 确认。
- 如果没有加入 Paperclip Cloud，您需要拥有自己的 Google OAuth 客户端，并启用 **Drive API**、**Docs API** 和 **Docs MCP API**。[Set up your own Google OAuth app](google-setup.md) 是完整的操作程序——请在开始此处操作之前完成。

## 选择能力组

| 组 | 智能体可以执行的操作 | 请求的权限范围 |
| --- | --- | --- |
| **Read only**（仅读取） | 读取文档文本和结构 | `documents.readonly` |
| **Read & edit**（读取与编辑） | 上述操作，外加更新文档 | `documents` |

这两个组都不请求 Google Drive 访问权限。仅凭 Docs 权限范围就足以读取和编辑该账户可以打开的文档（通过文档 ID）。使用早期版本建立的连接请求了更多权限。如果在此更改之前建立的 **Connect with Paperclip** 连接停止工作，请选择 **Reconnect**（重新连接）以使用较小的权限集进行登录。使用您自己的 OAuth 应用的连接在重新连接之前将保留其先前的授权。

该组在连接的生命周期内是固定的。

## 连接 Google Docs

1. 打开 **Connectors**（连接器）并选择 **Google Docs**。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。要选择不同的身份、缩小智能体范围或使用其他登录方法，请选择 **Change**（更改）。
3. 检查能力组。设置从可以执行更改的组开始；如果要仅读取连接，请选择 **Change** 并在 **What should Paperclip be able to do?**（Paperclip 应该能够做什么？）下选择它。当您的实例提供时，Paperclip 使用 **Connect with Paperclip**。如果要使用自己的客户端，请选择 **Use your own Google OAuth app** 并提供来自 [Set up your own Google OAuth app](google-setup.md) 的客户端 ID 和密钥。
4. 使用已注册的 Workspace 账户完成 Google 的同意屏幕。

## 选择访问权限

文档可达性来自 Google：连接可以打开授权账户可以打开的内容。Paperclip 中没有文档选择器。

已审查的操作：

| 操作 | 组 |
| --- | --- |
| `read-doc` | 两者 |
| `update-doc` | 仅限 **Read & edit** |

这是一个刻意保持较小的接口：两个操作，而不是一个 Docs 编辑器。`update-doc` 通过 Docs API 应用更改，因此它能表达的内容就是该 API 接受的内容——在计划依赖于特定类型编辑的工作之前，请在 **Permissions**（权限）选项卡上检查该操作的架构。确定不存在的是超出这两个操作之外的任何内容：这里没有用于评论、建议或修订历史的单独工具。

> **注意：** 连接器的指南是文档更新应经过批准。请将 `update-doc` 保持为 **Ask first**（先询问）；编辑会落入其他人可能正在使用的真实文档中。

[连接器访问机制说明](access-model.md) 涵盖了身份和智能体选择。

## 尝试使用

通过 URL 而非标题来识别文档，并选择其中您已知的一项内容。替换两者：

```txt
Read this Google Doc — https://docs.google.com/document/d/YOUR_DOC_ID/edit — and
tell me the text of its first heading. Do not edit it.
```

预期结果是您自己可以看到的标题。

> **警告：** 不要使用“标题为 X 的文档”进行测试。此处已审查的操作是 `read-doc`，它需要一个文档标识符——此连接器上没有文档搜索工具。基于标题的请求会使智能体猜测或因与您设置无关的原因失败。如果您确实需要标题搜索，请使用 [Google Drive](google-drive.md) 或 [Google Workspace 搜索](google-workspace-search.md)。

如果您想确认编辑功能，请在为此目的创建的草稿文档上进行操作，而不是在实时文档上操作。

> **注意：** 这是示例任务，而非记录的测试结果。请替换一个您可以打开的文档 URL 或 ID，以及一个您已知的值。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| Google 在同意屏幕前拒绝 | Developer Preview 注册未完成 | 完成注册并重试 |
| 未提供 **Connect with Paperclip** | 实例未加入 Paperclip Cloud，或 Cloud 未提供 Docs 配置文件 | 使用您自己的 Google OAuth 应用 |
| 找不到文档 | 它未与授权账户共享 | 在 Google Drive 中共享；无需重新连接 |
| `update-doc` 缺失 | 连接是以 **Read only** 建立的 | 建立一个 **Read & edit** 连接 |
| 编辑未产生预期的格式 | 更新通过 Docs API 进行，该 API 不涵盖每个编辑器功能 | 在 Google Docs 中完成格式设置 |
| **Needs attention**（需要关注） | Google 令牌已过期或被撤销 | 选择 **Reconnect** |

限制：一个连接覆盖一个 Google 账户。创建文档不属于此连接器的一部分——请使用带有创建组的 [Google Drive](google-drive.md)。评论、建议和修订历史未公开。适用 Developer Preview。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/google-docs/)

- [Google Drive](google-drive.md) — 查找和创建文件。
- [连接器访问机制说明](access-model.md)
- [验证连接器并修复故障连接器](verify-and-troubleshoot.md)
- [Google Docs API MCP reference](https://developers.google.com/workspace/docs/api/reference/mcp)
