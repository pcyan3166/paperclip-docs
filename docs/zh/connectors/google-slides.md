---
paperclip_version: v2026.1005.0
seo_title: Google Slides 连接器
seo_description: 允许智能体读取 Google Slides 演示文稿并可选地更新它们。能力组、编辑涵盖范围、读取测试及故障排查。
---

# Google Slides

> **警告：** **Google 验证待定。** Paperclip 尚未完成 Google 应用验证。在授权过程中，您可能会看到未验证应用的警告。如果 Google 提供 **Advanced**（高级）选项以继续前往 Paperclip，在审查所请求的访问权限后，您可以选择继续。并非所有账户都提供此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果您无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：** 在验证待定期间，Google Workspace 连接器以及任何已保存的 Google 账户将从 **Connectors**（连接器）页面暂时隐藏。这仅将其从列表中隐藏。现有的 Google 连接将继续运行，其访问权限和权限保持不变，需要 Google 服务的智能体仍可在其任务中通过连接卡片向您请求该服务。

智能体可以读取 Google Slides 演示文稿的幻灯片和内容，并在编辑连接下对其进行更新。

> **警告：** Google Slides 在授权前需要 Google Workspace 开发者预览版注册。Google 必须注册登录的 Workspace 电子邮件，如果您使用自己的 OAuth 客户端，还必须注册拥有该 OAuth 客户端的 Cloud 项目。请先在 [Google Workspace Developer Preview](https://developers.google.com/workspace/preview) 处申请。

## 连接前准备

- 一个 Google Workspace 账户，该账户已能打开您希望智能体使用的演示文稿，且已确认完成开发者预览版注册。
- 如果没有加入 Paperclip Cloud，则需要您自己的 Google OAuth 客户端，并启用 **Drive API**、**Slides API** 和 **Slides MCP API**。[Set up your own Google OAuth app](google-setup.md) 是完整的操作程序——请在开始此处操作之前完成。

## 选择能力组

| 组 | 智能体可以执行的操作 | 请求的权限范围 |
| --- | --- | --- |
| **仅读取** | 读取演示文稿的幻灯片和内容 | `presentations.readonly` |
| **读取与编辑** | 上述操作，外加更新演示文稿 | `presentations` |

两个组都不请求 Google Drive 访问权限。仅 Slides 权限范围就足以读取和编辑该账户可以打开的演示文稿，通过演示文稿 ID 进行。使用早期版本建立的连接请求了更多权限。如果在此更改之前建立的 **Connect with Paperclip** 连接停止工作，请选择 **Reconnect**（重新连接）以使用较小的权限集进行登录。使用您自己的 OAuth 应用的连接将保留其先前的授权，直到您重新连接它。

该组在连接的生命周期内是固定的。

## 连接 Google Slides

1. 打开 **Connectors** 并选择 **Google Slides**。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。要选择不同的身份、缩小智能体范围或使用其他登录方法，请选择 **Change**（更改）。
3. 检查能力组。设置从可以执行更改的组开始；如果要仅读取连接，请选择 **Change** 并在 **What should Paperclip be able to do?**（Paperclip 应该能够做什么？）下选择它。当您的实例提供时，Paperclip 使用 **Connect with Paperclip**。如果要使用您自己的客户端，请选择 **Use your own Google OAuth app** 并提供来自 [Set up your own Google OAuth app](google-setup.md) 的客户端 ID 和密钥。
4. 使用已注册的 Workspace 账户完成 Google 的同意屏幕。

## 选择访问权限

演示文稿的可达性来自 Google：连接可以打开授权账户可以打开的内容。Paperclip 中没有文件选择器。

已审查的操作：

| 操作 | 组 |
| --- | --- |
| `read-presentation` | 两者 |
| `update-presentation` | 仅限 **读取与编辑** |

对这里的编辑含义要有现实的认识。写入能力仅通过一个操作 `update-presentation` 提供，它通过 Slides API 应用更改，而不是驱动 Slides 编辑器。由此产生两个后果，其中只有第一个是能力声明：

- **支持哪些编辑取决于连接器定义的操作结构。** 不要假设该操作暴露了每个 Slides API 请求。在依赖某项具体编辑功能之前，请检查该操作当前的输入参数结构。
- **它无法提供的是设计判断。** 即使 API 接受更改，智能体在看不见结果的情况下放置元素，也不会像人那样安排幻灯片。将智能体的编辑视为初稿，之后由某人在 Slides 中打开。

> **注意：** 连接器的指导原则是演示文稿更新应经过批准。请将 `update-presentation` 保持在 **Ask first**（先询问）状态。

## 尝试使用

通过 URL 而非标题来识别演示文稿，并对照您可以打开的演示文稿检查答案：

```txt
Read this presentation — https://docs.google.com/presentation/d/YOUR_DECK_ID/edit
— and list the title of each slide in order. Do not change it.
```

将标题与演示文稿进行比较。

> **警告：** 不要使用“名为 X 的演示文稿”进行测试。已审查的操作是 `read-presentation`，它接受演示文稿标识符——此连接器没有演示文稿搜索工具，因此基于标题的请求测试的是发现功能而非您的连接。标题搜索属于 [Google Drive](google-drive.md) 或 [Google Workspace 搜索](google-workspace-search.md)。

如果您想确认编辑功能，请在副本上操作，而不是在即将由某人展示的演示文稿上操作。

> **注意：** 这是说明性任务，而非记录的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| Google 在同意屏幕前拒绝 | 开发者预览版注册未完成 | 完成注册并重试 |
| 未提供 **Connect with Paperclip** | 实例未加入 Paperclip Cloud，或 Cloud 未提供 Slides 配置文件 | 使用您自己的 Google OAuth 应用 |
| 找不到演示文稿 | 它未与授权账户共享 | 在 Google Drive 中共享；无需重新连接 |
| 缺少 `update-presentation` | 连接是以 **仅读取** 建立的 | 建立 **读取与编辑** 连接 |
| 编辑已落地但看起来不对 | Slides API 不涵盖每种布局和设计功能 | 在 Google Slides 中调整 |
| **Needs attention**（需要关注） | Google 令牌已过期或被撤销 | 选择 **Reconnect** |

限制：一个连接覆盖一个 Google 账户。创建演示文稿不属于此连接器——请使用带有创建组的 [Google Drive](google-drive.md)。评论和修订历史未暴露。适用开发者预览版。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/google-slides/)

- [Google Drive](google-drive.md) — 查找和创建文件。
- [Google Docs](google-docs.md)
- [连接器访问机制说明](access-model.md)
- [Google Slides API MCP reference](https://developers.google.com/workspace/slides/api/reference/mcp)
