---
paperclip_version: v2026.1023.0
seo_title: Tavily 网络搜索连接器
seo_description: 通过您的 Tavily 账户为智能体提供网络搜索和页面提取功能。包括浏览器登录、使用的 Tavily 密钥、HTTPS 回调规则以及故障排除方法。
---

# Tavily

智能体可以通过 Tavily 搜索公共网络并提取页面内容，使用您自己的 Tavily 账户。这适用于有边界的调研任务，在这些任务中，您希望智能体引用每个事实的来源。

> **注意：** Tavily 出现在每个实例的目录中；无需启用实验性设置。

## 连接前准备

- 一个具有 API 密钥和一定使用额度的 Tavily 账户。您无需粘贴密钥——登录过程会自动为您选择密钥——但搜索操作会消耗该密钥的额度。
- 您的 Paperclip 实例需要一个公共 HTTPS 地址。Tavily 拒绝在登录后将请求重定向回本地的 `http://` 地址，因此仅运行在您自己机器上的 Paperclip 无法完成此设置。

## 连接 Tavily

1. 打开 **连接器** 并选择 **Tavily**。
2. 阅读主按钮上方的访问说明行。它说明了连接所属的身份以及哪些智能体可以使用它。若要选择不同身份或限制智能体范围，请选择 **更改**。
3. 选择 **继续前往 Tavily** 并在 Tavily 的授权界面中批准访问权限。

Paperclip 会自动向 Tavily 注册其客户端，并请求 `openid` 和 `offline_access` 权限范围。无需在开发者控制台中进行任何配置。

> **注意：** Tavily 文档中也记录了将 API 密钥放入服务器地址的做法。请勿使用该路径；登录方式可确保密钥不出现在 URL 中。

## 选择访问权限

Tavily 服务器仅提供读取操作：搜索网络和提取页面内容。没有会更改您 Tavily 账户中任何内容的操作，这使得该连接器成为风险较低的共享连接器之一。

仍有两件事值得决定：

- **谁的额度。** 在登录时，如果您的账户拥有名为 `mcp_auth_default` 的密钥，Tavily 将选择该密钥。否则，它可能会使用您的个人默认密钥或找到的第一个密钥。请检查连接正在使用哪个密钥及其额度，如果您希望控制这一点，请在连接之前创建一个 `mcp_auth_default` 密钥。
- **智能体如何处理结果。** 搜索结果属于外部内容。要求智能体使用聚焦的查询、引用来源 URL，并将检索到的内容与其推断的内容分开。调研并不构成联系公司或在其网站上填写表格的许可。参见 [设置操作权限](action-permissions.md)。

## 试用

```txt
Search the web with Tavily for three recent articles about a topic we care about, and list them with their links and a one-line summary each.
```

打开链接。您可以检查的结果将确认登录状态和智能体的权限。

> **注意：** 这是示例任务，而非记录在案的测试结果。

## 故障排除与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 登录在开始前被拒绝 | Paperclip 通过本地 HTTP 访问 | 为 Paperclip 使用公共 HTTPS 地址，然后重新连接 |
| 搜索因会话或请求错误而失败 | Tavily 的会话已过期 | 要求智能体重试；Paperclip 会启动新会话，但不会自动重放失败的调用 |
| 使用量来自意外的密钥 | Tavily 在登录时选择了默认密钥 | 在 Tavily 中创建一个 `mcp_auth_default` 密钥并重新连接 |
| 搜索停止工作 | 密钥的额度已用完 | 在 Tavily 中充值 |
| **需要关注** | 授权已被撤销或过期 | 选择 **重新连接** |

限制：每个连接对应一个 Tavily 账户，仅支持浏览器登录，且仅支持公共网络内容。

## 相关指南

- [Exa](exa.md) 和 [并行搜索](parallel-search.md) — 无需密钥的网络搜索连接器。
- [Telem.AI](telem.md) — 使用一个密钥跨多个提供商进行搜索。
- [连接器访问机制](access-model.md)
- [设置操作权限](action-permissions.md)
- [Tavily MCP 文档](https://docs.tavily.com/documentation/mcp)

## 来源

- [Tavily 定义](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/tavily.json) — 方法、权限范围、HTTPS 回调规则及密钥选择警告。
- [权限审查](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/doc/connections/tool-method-permission-reviews.json) — 只读操作范围。
