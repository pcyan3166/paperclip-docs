---
seo_title: Fireflies 连接器
seo_description: 通过浏览器登录或 API 密钥连接 Fireflies 以读取会议数据。包含访问限制、独立的例程 Webhook 以及未经验证的设置。
---

# Fireflies

Fireflies 允许智能体访问已连接账户可用的会议转录、摘要和待办事项。Paperclip 提供浏览器登录或 API 密钥两种方式。

## 连接前准备

- 一个具有您希望智能体使用的会议访问权限的 Fireflies 账户。
- 对于密钥方法，需要一个来自 Fireflies **Settings → Developer Settings** 的 API 密钥。

## 连接 Fireflies

> **未经验证的设置：** 此流程遵循固定的 Paperclip 定义和 Fireflies 的文档。它尚未通过实时 Fireflies 连接进行测试。以下每个步骤均未经过验证。

1. **未经验证：** 打开 **Connectors** 并选择 **Fireflies**。
2. **未经验证：** 选择 **Sign in with Fireflies** 进行浏览器授权，或选择 **Use an API key** 使用您控制的密钥。
3. **未经验证：** 在 **Access** 中，选择身份以及可以使用该身份的智能体。
4. **未经验证：** 完成浏览器登录，或在 **Fireflies API key** 中粘贴密钥。两种方法均使用 `https://api.fireflies.ai/mcp`；Paperclip 将密钥作为 `Authorization: Bearer` 标头发送。
5. **未经验证：** 完成连接检查，并在要求智能体读取会议之前检查操作列表。

## 选择访问权限

已连接账户的会议访问权限决定了可以访问哪些数据。Paperclip 的智能体选择和操作权限决定了谁可以通过连接器调用暴露的操作。

每个工具初始状态均为 **Allowed**。对于您希望审查或阻止的操作，请使用 **Ask first** 或 **Off**。会议文本可能包含机密信息；请选择可以处理预期会议的智能体。

可选的摘要就绪 Webhook 在例程的 **Triggers** 选项卡中单独配置。其签名密钥与 Fireflies API 密钥是分开的。连接此账户不会配置例程触发器。

## 尝试使用

```txt
Find a meeting I can access and report its title and date. Do not upload, edit, or delete anything.
```

将结果与 Fireflies 进行比较，并检查连接器调用。请使用内容适合所选智能体的会议。

> **未经验证的检查：** 示例任务，非记录的测试结果。

## 故障排除与限制

| 问题 | 检查项 |
| --- | --- |
| 未找到会议 | 确认已连接账户可以在 Fireflies 中看到已处理的转录。 |
| API 密钥身份验证失败 | 确认 Developer Settings 中的密钥，并使用正确的密钥重新连接。 |
| 例程未收到摘要事件 | 检查其独立的触发器和 Webhook 签名密钥。 |
| 缺少预期操作 | 检查连接列表并使用 **Refresh actions**。 |

连接器不会授予账户无法访问的会议的访问权限。适用提供商限制。在依赖该连接之前，请在您的实例中测试登录和读取操作。

## 相关指南

- [连接器概述](https://paperclip.ing/product/connectors/fireflies/)

- [连接器访问权限的工作原理](access-model.md)
- [设置操作权限](action-permissions.md)
- [验证与故障排除](verify-and-troubleshoot.md)
- [Fireflies 服务器配置](https://docs.fireflies.ai/getting-started/mcp-configuration)

## 来源

- [Paperclip 连接器定义](https://github.com/paperclipai/paperclip/blob/3166e93a7eee315e3bfbda622e080044ec5c343d/packages/shared/src/app-definitions/fireflies.json#L18) — 在固定的产品版本下的方法名称、身份验证、端点和连接器特定限制。提供商设置文档链接如上。
