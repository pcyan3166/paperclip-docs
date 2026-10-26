---
seo_title: You.com 连接器
seo_description: 通过浏览器登录、API 密钥或无密钥免费配置文件连接 You.com。了解工具集缩减、提供商限制及未经验证的设置。
---

# You.com

You.com 将智能体连接到其托管的 Web 智能服务器。您可以选择浏览器登录、API 密钥或功能缩减的无密钥免费配置文件。

## 连接前准备

- 用于浏览器登录的 You.com 账户，或用于密钥方法的来自 `you.com/platform` 的 API 密钥。
- **使用免费配置文件** 无需账户，该配置文件提供功能缩减的只读工具集，并受提供商速率限制。

## 连接 You.com

> **未经验证的设置：** 此流程遵循固定的 Paperclip 定义和 You.com 的文档。尚未通过实时 You.com 连接进行测试。以下每个步骤均未经验证。

1. **未经验证：** 打开 **连接器** 并选择 **You.com**。
2. **未经验证：** 选择 **使用 You.com 登录**、**使用 API 密钥** 或 **使用免费配置文件**。
3. **未经验证：** 在 **访问** 页面，选择身份以及允许使用该身份的智能体。
4. **未经验证：** 完成浏览器登录，将密钥粘贴到 **You.com API 密钥** 中，或对于免费配置文件继续操作而不提供凭据。登录和密钥方法使用 `https://api.you.com/mcp`；免费方法使用 `https://api.you.com/mcp?profile=free`。Paperclip 将 API 密钥作为 `Authorization: Bearer` 标头发送。
5. **未经验证：** 完成检查并查看所选配置文件的动作列表。

产品服务器与 You.com 的文档搜索服务器是分开的。请为此连接器使用上述端点。

## 选择访问权限

每个工具初始状态均为 **允许**。在 **权限** 中，可将任何暴露的动作设置为 **先询问** 或 **关闭**。只读调用仍可能将搜索词或请求的 URL 发送给提供商。

免费配置文件提供的工具少于经过身份验证的访问。请检查已连接配置文件的列表，而不要假设所有 You.com API 都作为动作暴露。提供商的速率限制和账户套餐权限仍然适用。

## 试用

```txt
Search the web for the official Paperclip documentation and report the source URL. Do not send private company information in the query.
```

检查返回的来源以及 Paperclip 中的连接器调用。

> **未经验证的检查：** 示例任务，非记录的测试结果。

## 故障排除与限制

| 问题 | 检查项 |
| --- | --- |
| 免费配置文件中缺少某个工具 | 可能需要经过身份验证的访问；请检查提供商的配置文件文档。 |
| 密钥被拒绝 | 检查密钥在 You.com 中的有效性和产品访问权限，然后重新连接。 |
| 请求达到速率限制 | 检查 You.com 中当前配置文件的限制。 |
| 仅出现文档搜索 | 确认连接使用的是产品端点，而非文档服务器。 |

切换凭据不会移除提供商限制。此页面不承诺固定的配额或固定的工具列表。

## 相关指南

- [连接器概述](https://paperclip.ing/product/connectors/youcom/)

- [连接器访问工作原理](access-model.md)
- [设置动作权限](action-permissions.md)
- [验证与故障排除](verify-and-troubleshoot.md)
- [You.com 服务器文档](https://you.com/docs/build-with-agents/mcp-server)

## 来源

- [Paperclip 连接器定义](https://github.com/paperclipai/paperclip/blob/3166e93a7eee315e3bfbda622e080044ec5c343d/packages/shared/src/app-definitions/youcom.json#L19) — 在固定的产品版本中，包含方法名称、身份验证、端点及连接器特定的限制。提供商设置文档链接见上文。
