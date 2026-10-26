---
paperclip_version: v2026.1023.0
seo_title: Context7 库文档连接器
seo_description: 让智能体通过 Context7 的托管服务器查询当前公开的库文档和代码示例。无需账户、密钥或登录。
---

# Context7

智能体可以通过 Context7 查询公开库的最新文档和代码示例。当任务依赖于库*当前*的工作方式，而非模型训练时的方式时，Context7 很有帮助。

Context7 出现在每个实例的目录中；无需启用实验性设置。

Context7 是只读且公开的。它无法查看您的代码或仓库，也无法更改任何地方的内容。这使其成为最容易尝试的连接器之一。

## 连接前准备

无需任何准备。无需创建账户，无需粘贴密钥，也无需登录。Paperclip 连接到 Context7 位于 `https://mcp.context7.com/mcp` 的公开托管服务器，无需凭据。

Context7 提供可提升速率限制并解锁私有仓库的 API 密钥。此连接不使用密钥，因此您将获得匿名层级：仅限公开文档，且速率限制较低。

## 连接 Context7

1. 打开 **Connectors** 并选择 **Context7**。
2. 阅读主按钮上方的访问行。它说明了连接所属对象以及哪些智能体可以使用它。选择 **Change** 以缩小智能体范围。
3. 选择 **Connect**。

这就是全部设置。连接完成后，操作列表将显示在 **Permissions** 选项卡上。

## 选择访问权限

此处关于风险的决定很少：Context7 的操作仅读取公开文档，且默认状态为 **Allowed**。

唯一值得思考的是**查询中包含的内容**。Context7 会接收每次查询的文本。智能体应仅询问关于公开库的问题，并排除机密、个人信息以及您自己代码的片段。如果您有处理敏感材料的智能体，在选择智能体时可以将它们排除在此连接之外。参见 [Share a connector with people and agents](share-access.md)。

## 尝试使用

```txt
Use Context7 to look up the current React documentation for cleaning up an effect in useEffect, and show me the recommended pattern with a short example.
```

将答案与库自身的文档进行比较。引用当前文档的答案可以确认连接正常工作。

> **注意：** 这是示例任务，而非记录的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 多次请求后查询失败或变慢 | 匿名速率限制 | 分散查询时间，或将工作分散到更少的智能体上 |
| 找不到私有仓库或内部库 | 此连接是公开且无密钥的 | 为私有文档使用其他来源；此方法不接受密钥 |
| 智能体找不到某个库 | Context7 未对其进行索引，或名称存在歧义 | 使用库的确切包名进行询问 |
| 智能体将代码粘贴到查询中 | 智能体将其包含在查询文本中 | 通过其指令提醒智能体，将查询限制在公开问题上 |

限制：仅限公开文档。由于没有 API 密钥，速率限制较低，且无法访问私有仓库。操作内容取决于 Context7 托管服务器所暴露的内容。

## 相关指南

- [Hugging Face](hugging-face.md) — 另一个公开、无密钥的查询连接器。
- [连接器访问机制说明](access-model.md)
- [设置操作权限](action-permissions.md)
- [Context7 API guide](https://context7.com/docs/api-guide)

## 来源

- [Context7 definition](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/context7.json) — 公开方法、服务器 URL 和警告。
- [Catalog visibility](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions.ts) — Context7 可连接并列入目录；默认操作策略。
