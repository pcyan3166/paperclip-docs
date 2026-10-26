---
paperclip_version: v2026.1005.0
seo_title: Google Workspace 搜索连接器
seo_description: 跨 Gmail、Drive、Calendar 和 Chat 的一次性只读搜索。何时使用它而非单应用连接器，它请求的权限，以及测试方法。
---

# Google Workspace 搜索

> **警告：** **Google 验证待定。** Paperclip 尚未完成 Google 应用验证。在授权过程中，您可能会看到未验证应用的警告。如果 Google 提供 **Advanced**（高级）选项以继续前往 Paperclip，在审查所请求的访问权限后，您可以选择继续。并非所有账户都提供此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果您无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：** 在验证待定期间，Google Workspace 连接器及任何已保存的 Google 账户将从 **Connectors**（连接器）页面暂时隐藏。这仅将其从列表中隐藏。现有的 Google 连接将继续运行，其访问权限和权限保持不变，需要 Google 服务的智能体仍可在其任务中通过连接卡片向您请求该服务。

这是一次跨 Gmail、Drive、Calendar 和 Chat 的只读搜索，因此智能体可以在不知道答案位于哪个应用的情况下回答“我们在哪里讨论过这个？”。

它搜索所有四个应用。无法将其连接为仅针对子集。

> **警告：** 此连接同时请求对 Gmail、Drive、Calendar 和 Chat 的读取访问权限。这比任何单应用连接器都更广泛。如果智能体只需要一个应用，请连接该应用。

## 何时使用此连接器而非应用连接器

| 您的需求 | 使用 |
| --- | --- |
| 在不知道内容位于哪个应用的情况下查找某物 | 此连接器 |
| 在某个特定应用中读取、编辑或创建内容 | [Gmail](gmail.md)、[Google Drive](google-drive.md)、[Google Calendar](google-calendar.md) 或 [Google Chat](google-chat.md) |
| 尽可能窄的访问权限 | 单应用连接器，配合其只读组 |

此连接器仅进行搜索。它返回带有链接的结果；它不读取完整文档、创建草稿或更改事件。需要对查找到的内容采取操作的智能体还需要相关的应用连接器。

## 连接前准备

- 一个 Google Workspace 账户，您愿意使其邮件、文件、日历和聊天可被搜索。
- 该账户的 Google Workspace Developer Preview 注册，并经 Google 确认。请在 [Google Workspace Developer Preview](https://developers.google.com/workspace/preview) 处申请。
- 如果没有加入 Paperclip Cloud，您需要拥有自己的 Google OAuth 客户端，并启用 **Gmail API**、**Drive API**、**Calendar API**、**Chat API** 和 **Workspace MCP API**——共五个 API，比任何单应用连接器所需的都多。[Set up your own Google OAuth app](google-setup.md) 是完整的操作程序。

## 连接 Google Workspace 搜索

1. 打开 **Connectors** 并选择 **Google Workspace Search**。
2. 阅读主按钮上方的访问权限说明行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他登录方法，请选择 **Change**（更改）。
3. 当您的实例提供时，Paperclip 使用 **Connect with Paperclip**（通过 Paperclip 连接）。若要使用自己的客户端，请选择 **Use your own Google OAuth app**（使用您自己的 Google OAuth 应用）并提供来自 [Set up your own Google OAuth app](google-setup.md) 的客户端 ID 和密钥。
4. 使用已注册的 Workspace 账户完成 Google 的同意屏幕。

一个能力组，**Search Workspace**（搜索 Workspace），请求 `gmail.readonly`、`drive.readonly`、`calendar.readonly` 和 `chat.messages.readonly`。只有一个操作，`search-corpus`。

## 选择访问权限

搜索覆盖的范围是授权账户在那四个应用中已经能看到的内容——其自己的邮件、与其共享的文件、其列表中的日历以及其所属的空间。Paperclip 不会缩小此范围，也没有按应用或按文件夹的过滤器。

由于范围较广，智能体选择的选择比平时更重要。建议优先选择 **Just agents I pick**（仅我选择的智能体）。在此处使用 **Organization identity**（组织身份）会让符合条件的智能体在负责人属于凭据适用人员范围的运行中搜索已连接的账户——这通常不是您想要的。

[连接器访问机制说明](access-model.md) 涵盖了身份和智能体选择。

## 试用

搜索一个您知道存在且能在结果中识别出的内容：

```txt
Search my Google Workspace for "renewal contract" and tell me which app each result came from.
```

预计会有少量结果，每个结果都通过链接标识其来源应用。将每个结果与已知项目进行比较，并检查连接器调用。一次查询仅验证返回的结果；它不能证明所有四个服务都成功进行了搜索。

> **注意：** 这是示例任务，而非记录的测试结果。请使用一个您知道出现在您自己的邮件或文件中的短语。

## 故障排除与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| Google 在同意屏幕前拒绝 | Developer Preview 注册未完成 | 完成注册并重试 |
| 未提供 **Connect with Paperclip** | 实例未加入 Paperclip Cloud，或 Cloud 未提供搜索配置文件 | 使用您自己的 Google OAuth 应用 |
| 结果始终只来自一个应用 | 查询在一个语料库中匹配，或账户在其他应用中内容很少 | 尝试一个您知道出现在其他地方的短语 |
| 找到结果但智能体无法读取整个项目 | 预期行为——此连接器进行搜索，不读取 | 添加相关的应用连接器 |
| 您需要一个专门用于 Drive 的连接 | 此连接请求跨多个 Workspace 服务的访问权限 | 使用 [Google Drive](google-drive.md) 并审查其权限和操作 |
| **Needs attention**（需要关注） | Google 令牌已过期或被撤销 | 选择 **Reconnect**（重新连接） |

限制：只读且仅搜索，使用一个已授权账户。结果取决于 Google 的搜索工具及该账户的访问权限。适用 Developer Preview。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/google-workspace-search/)

- [Gmail](gmail.md)、[Google Drive](google-drive.md)、[Google Calendar](google-calendar.md)、[Google Chat](google-chat.md)
- [连接器访问机制说明](access-model.md)
- [Google universal search MCP guide](https://developers.google.com/workspace/guides/universal-search-mcp)
