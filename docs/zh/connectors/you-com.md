---
paperclip_version: v2026.1005.0
seo_title: You.com 连接器
seo_description: 让智能体通过 You.com 进行网络搜索。选择浏览器登录、API 密钥或无密钥免费配置，然后运行快速搜索测试并进行故障排查。
---

# You.com

智能体可以通过 You.com 搜索网络——当任务需要来自公司外部的最新信息，而不仅仅是 Paperclip 中已有的内容时，这非常有用。

## 连接前须知

您有三种选择，其中一种完全不需要任何配置：

- 一个 You.com 账户，用于浏览器登录。
- 一个来自 you.com/platform 的 You.com API 密钥，如果您需要更高的调用速率或配额。
- 两者都不需要——无密钥免费配置无需账户即可使用。

## 连接 You.com

1. 打开 **Connectors** 并选择 **You.com**。
2. 阅读主按钮上方的访问说明行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择其他身份、缩小智能体范围或使用其他登录方法，请选择 **Change**。
3. 选择连接方式：
   - **Sign in with You.com** — 完成浏览器登录。Paperclip 会自动注册其客户端。
   - **Use an API key** — 将您的密钥粘贴到 **You.com API key** 字段中。当浏览器登录不适用时使用此方法。
   - **Use the free profile** — 无需账户即可连接。You.com 将此限制为具有自身速率限制的缩减版、只读工具集。

这三种方式都连接到 You.com 托管的 MCP 服务器，因此您的智能体看到的搜索操作来自该服务器。可以先用免费配置试用连接器，再决定是否需要密钥。

## 选择访问权限

此连接器从公共网络读取数据；它不会访问您自己的数据。这使其成为共享风险较低的连接器之一。

仍有两件事值得决定：

- **谁使用配额。** 每个拥有访问权限的智能体都消耗同一账户的速率限制——或者免费配置的速率限制，后者更严格。如果一个繁忙的智能体不断触及限制，请为其提供独立的连接，或改用 API 密钥。
- **智能体如何处理结果。** 搜索结果属于外部内容。智能体应将其视为需要核查的信息，而非需要遵循的指令。有关如何将其他连接器中的后续操作保持在 **Ask first** 状态，请参阅 [设置操作权限](action-permissions.md)。

## 试用

```txt
Search the web with You.com for the latest release notes of a tool we use, and summarise the top three results with their links.
```

打开智能体返回的链接。带有可核查来源的搜索可以确认连接状态及智能体的权限。

> **Note:** 示例任务，非记录的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 在大量使用后搜索失败 | You.com 速率限制，尤其是在免费配置下 | 等待并重试，或使用 API 密钥连接 |
| 预期的操作缺失 | 免费配置暴露的是缩减版、只读工具集 | 使用登录或 API 密钥重新连接 |
| API 密钥被拒绝 | 密钥复制不完整或已被撤销 | 创建新密钥并重新连接 |
| **Needs attention** | 授权已过期或被撤销 | 选择 **Reconnect** |

限制：每个连接对应一个 You.com 账户或配置。可用的操作取决于 You.com 托管服务器所暴露的内容。

## 相关指南

- [OpenAI](openai.md)、[OpenRouter](openrouter.md)、[Grok](xai.md) — 其他 AI 提供商连接器。
- [连接器访问机制说明](access-model.md)
- [设置操作权限](action-permissions.md)
- [You.com MCP server documentation](https://you.com/docs/build-with-agents/mcp-server)
