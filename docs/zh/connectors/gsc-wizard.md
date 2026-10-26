---
paperclip_version: v2026.1023.0
seo_title: GSC Wizard 搜索控制台连接器
seo_description: 让智能体通过 GSC Wizard 读取 Google Search Console、Bing Webmaster 和分析数据，并将 URL 提交操作保留在您的审核之后。
---

# GSC Wizard

智能体可以通过 GSC Wizard 读取您站点的搜索性能数据——包括您在该处连接的属性的 Google Search Console、Bing Webmaster 和分析数据。它适用于诸如“本周哪些页面失去了点击量？”之类的 SEO 任务。

## 连接前准备

- 一个 GSC Wizard 账户，其中已连接您的搜索和分析属性。
- 如果您计划使用密钥，请先决定智能体需要只读还是读写访问权限。您在 GSC Wizard 中创建密钥时选择此权限。

## 连接 GSC Wizard

1. 打开 **Connectors** 并选择 **GSC Wizard**。
2. 阅读主按钮上方的访问说明行。它说明了连接属于谁以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或切换方法，请选择 **Change**。
3. 选择一种方法：
   - **Sign in with GSC Wizard** — 完成浏览器登录。Paperclip 会自动注册其客户端。
   - **Use a GSC Wizard MCP key** — 为预期账户在 GSC Wizard 中创建一个密钥，选择只读或读写权限，并将其粘贴到 **Your GSC Wizard key** 字段中。

## 选择访问权限

浏览器登录请求 `mcp:read` 和 `mcp:write` 范围。使用密钥时，您在创建时选择的权限是上限；Paperclip 无法稍后提高该权限，因此如果智能体只需要报告，请选择只读。

大多数工具读取报告，但有些工具可以提交 URL 进行索引或以其他方式更改您站点的索引状态。这些更改发生在 Paperclip 之外，且难以快速撤销。在 **Permissions** 选项卡上将它们设置为 **Ask first** 或 **Off**，并将读取操作保留为 **Allowed**。参见 [设置操作权限](action-permissions.md)。

由于一个 GSC Wizard 账户可以包含来自多个提供商的多个站点，请要求智能体在每次查询中指明站点和日期范围。这可防止报告静默地来自错误的属性。

## 试用

```txt
Using GSC Wizard, list the properties you can see, then give me a seven-day performance summary (clicks, impressions, average position) for our main site. Do not submit any URLs.
```

将数字与相同日期的 Search Console 进行核对。匹配的报告可确认凭据和属性。

> **注意：** 示例任务，非记录的测试结果。

## 故障排除与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 缺少某个站点 | 该站点未在 GSC Wizard 内部连接，或账户无法查看它 | 在 GSC Wizard 中连接该属性 |
| 数字与 Search Console 不匹配 | 站点或日期范围不同 | 在任务中明确指明两者 |
| URL 提交被拒绝 | 密钥为只读，或操作设置为 **Off** | 创建读写密钥，或检查 **Permissions** 选项卡 |
| **Needs attention** | 授权或密钥已被撤销 | 选择 **Reconnect** |

限制：每个连接对应一个 GSC Wizard 账户。密钥的权限在创建时固定。可用操作取决于 GSC Wizard 托管服务器暴露的内容。

## 相关指南

- [Windsor.ai](windsor-ai.md) — 来自多个来源的营销、分析、CRM 和仓库数据。
- [设置操作权限](action-permissions.md)
- [GSC Wizard MCP documentation](https://www.gscwizard.com/claude-search-console-mcp.html)

## 来源

- [GSC Wizard definition](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/gsc-wizard.json) — 方法、范围、密钥字段和警告。
