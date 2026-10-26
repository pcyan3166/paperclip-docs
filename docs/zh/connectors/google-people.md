---
paperclip_version: v2026.1005.0
seo_title: Google People 连接器
seo_description: 允许智能体查找联系人和 Workspace 目录个人资料。仅读，在任何能力组中均无写入权限。账户要求、查找测试及故障排查。
---

# Google People

> **警告：** **Google 验证待处理。** Paperclip 尚未完成 Google 应用验证。在授权过程中，您可能会看到未验证应用的警告。如果 Google 提供 **Advanced**（高级）选项以继续前往 Paperclip，在审查所请求的访问权限后，您可以选择继续。并非所有账户都提供此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果您无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：** 在验证待处理期间，Google Workspace 连接器以及任何已保存的 Google 账户将从 **Connectors**（连接器）页面暂时隐藏。这仅将其从列表中隐藏。现有的 Google 连接将继续运行，其访问权限和权限保持不变，需要 Google 服务的智能体仍可在其任务中通过连接卡片向您请求该服务。

智能体可以搜索您的 Google 联系人，搜索您组织的 Workspace 目录，并读取您自己的个人资料。

此连接器为仅读。没有写入能力组，因此智能体无法创建、编辑或删除联系人。

> **警告：** Google People 在授权前需要 Google Workspace Developer Preview 注册。Google 必须注册用于登录的 Workspace 电子邮件，如果您使用自己的客户端，还必须注册拥有该 OAuth 客户端的 Cloud 项目。请先在 [Google Workspace Developer Preview](https://developers.google.com/workspace/preview) 处申请。

## 连接前准备

- 一个已确认 Developer Preview 注册的 Google 账户。
- 对于目录搜索，需要一个 Google Workspace 账户。目录的可用性取决于 Workspace 账户及其管理员的设置——个人 Google 账户有联系人，但没有组织目录。
- 如果没有加入 Paperclip Cloud，则需要您自己的 Google OAuth 客户端，并启用 **People API** 和 **People MCP API**。[配置您自己的 Google OAuth 应用](google-setup.md) 是完整的配置流程，请在开始本指南前完成。

## 连接 Google People

1. 打开 **Connectors** 并选择 **Google People**。
2. 阅读主按钮上方的访问权限说明行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择其他身份、缩小智能体范围或使用其他登录方法，请选择 **Change**（更改）。
3. 当您的实例提供时，Paperclip 使用 **Connect with Paperclip**（通过 Paperclip 连接）。若要改用您自己的客户端，请选择 **Use your own Google OAuth app**（使用您自己的 Google OAuth 应用），并提供来自 [Set up your own Google OAuth app](google-setup.md) 的客户端 ID 和密钥。
4. 完成 Google 的同意屏幕。

有一个能力组，**Read contacts**（读取联系人），请求 `directory.readonly`、`userinfo.profile` 和 `contacts.readonly`。

## 选择访问权限

有两个不同的人员来源，了解答案来自哪个来源是值得的：

| 来源 | 涵盖内容 | 操作 |
| --- | --- | --- |
| 您的联系人 | 授权账户上的个人联系人列表 | `search-contacts` |
| 组织目录 | 受您的管理员目录设置约束的 Workspace 同事 | `search-directory-people` |
| 您自己的个人资料 | 授权账户本身的个人资料 | `get-user-profile` |

目录的可达范围由 Google 决定，而非 Paperclip。Workspace 管理员可以限制或禁用目录共享，在这种情况下，即使授权成功，目录搜索也可能返回很少或没有结果。

> **警告：** 此连接器返回关于真实个人的个人数据——姓名、电子邮件地址以及您的目录暴露的其他任何信息。请优先选择 **Just agents I pick**（仅我选择的智能体）而非 **Any agent**（任何智能体），并给智能体提供具体的查找指令，而不是枚举目录的指令。

[连接器访问机制说明](access-model.md) 涵盖了身份和智能体选择。

## 尝试使用

查找一个您可以验证的人，而不是列出所有人：

```txt
Look up "Priya Raman" in our Google directory and tell me the email address on
that contact.
```

使用全名，而不是名字——名字通常会匹配多个人，然后您是在检查智能体的消歧能力，而不是连接本身。预期结果是您已经知道的那个地址。

如果仍然有多个人匹配，请询问每个结果来自哪个来源：联系人和目录是不同的，知道哪个来源回答了问题可以告诉您连接实际达到了什么范围。

> **注意：** 这是示例任务，而非记录的测试结果。请替换为一个您可以验证其地址的同事。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| Google 在同意屏幕前拒绝 | Developer Preview 注册未完成 | 完成注册并重试 |
| 未提供 **Connect with Paperclip** | 实例未加入 Paperclip Cloud，或 Cloud 未提供 People 配置文件 | 使用您自己的 Google OAuth 应用 |
| 目录搜索无结果，但联系人正常 | 账户不是 Workspace 账户，或管理员已限制目录共享 | 询问您的 Workspace 管理员关于目录可见性的问题 |
| 目录结果中缺少某位同事 | 他们超出了管理员配置的共享目录范围 | Paperclip 中无需修复 |
| 智能体无法添加或编辑联系人 | 预期行为——此连接器为仅读 | 在 Google Contacts 中操作 |
| **Needs attention**（需要关注） | Google 令牌已过期或被撤销 | 选择 **Reconnect**（重新连接） |

限制：在所有能力组中均为仅读。一个连接覆盖一个 Google 账户。联系人组和标签，以及其他人的私人联系人列表，均不暴露。适用 Developer Preview。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/google-people/)

- [Google Workspace 搜索](google-workspace-search.md) — 跨 Gmail、Drive、Calendar 和 Chat 的单一仅读搜索。
- [连接器访问机制说明](access-model.md)
- [验证连接器并修复故障连接器](verify-and-troubleshoot.md)
- [Google People API MCP reference](https://developers.google.com/people/api/mcp)
