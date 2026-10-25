---
paperclip_version: v2026.1023.0
seo_title: Windsor.ai 营销数据连接器
seo_description: 允许智能体通过 Windsor.ai 查询营销、分析、CRM 和仓库数据，并阻止任何更改广告活动、预算或帖子的操作。
---

# Windsor.ai

智能体可以通过一个连接查询您在 Windsor.ai 中连接的数据——包括广告、分析、CRM 和仓库数据源。这是一种快速回答“上周的广告活动表现如何？”的方法，无需为每个广告平台单独配置连接器。

> **注意：** Windsor.ai 出现在每个实例的目录中；无需启用实验性设置。

> **警告：** Windsor.ai 的服务器还暴露了某些已连接的广告和商务平台的写入操作：广告活动、预算、帖子和商品列表。在 Windsor.ai 中连接数据源并不意味着智能体应当修改这些数据。请将这些操作保持为 **Off（关闭）** 或 **Ask first（先询问）**。

## 连接前准备

- 拥有一个 Windsor.ai 账户，且已链接所需的数据源。仅链接任务所需的源账户。
- 对于浏览器登录，Paperclip 必须可通过公共 HTTPS 地址访问。Windsor.ai 拒绝本地 HTTP 回调；在本地实例上，请改用 API 密钥。

## 连接 Windsor.ai

1. 打开 **Connectors（连接器）** 并选择 **Windsor.ai**。
2. 阅读主按钮上方的访问行。它说明了该连接属于谁以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或切换方法，请选择 **Change（更改）**。
3. 选择一种方法：
   - **Sign in with Windsor.ai（使用 Windsor.ai 登录）** — 选择 **Continue to sign in（继续登录）** 并完成浏览器登录。Paperclip 会自动注册其客户端。
   - **Use a Windsor.ai API key（使用 Windsor.ai API 密钥）** — 为预期的 Windsor.ai 组织创建密钥，并将其粘贴到 **Your Windsor.ai key（您的 Windsor.ai 密钥）** 字段中。

## 选择访问权限

浏览器登录请求 `create`、`delete` 和 `offline_access` 作用域。Windsor.ai 要求 `delete` 作用域才能打开会话，即使您只想读取数据也是如此，因此批准登录会授予提供商级别的删除权限。如果这对您的账户不可接受，请勿批准。Paperclip 也无法提高或缩小 API 密钥的权限。

Paperclip 的操作列表是缩小权限范围的地方：

| 操作类型 | 建议设置 |
| --- | --- |
| 读取源字段和报告数据 | **Allowed（允许）** |
| 更改广告活动、预算、帖子或商品列表 | **Off（关闭）**，或如果由人工审核每一项，则设为 **Ask first（先询问）** |

在连接后立即在 **Permissions（权限）** 选项卡中设置这些项。参见 [Set action permissions](action-permissions.md)。

当智能体报告结果时，请要求它们指明源账户和指标定义，因为同一指标在不同平台上可能含义不同。

## 试用

```txt
Using Windsor.ai, list the connected data sources, then pull clicks and impressions from our Search Console source for the last five days. Name the source account in your answer. Do not change anything.
```

将数字与源平台进行核对。像这样的有限读取可以确认凭据以及哪些源已链接。

> **注意：** 这是示例任务，而非记录的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 本地实例上浏览器登录失败 | Windsor.ai 拒绝本地 HTTP 回调 | 使用公共 HTTPS 地址，或使用 API 密钥连接 |
| 缺少数据源 | 未在 Windsor.ai 中链接 | 在 Windsor.ai 中链接源账户 |
| 智能体更改了广告活动或预算 | 写入操作被设置为 **Allowed（允许）** | 将其设置为 **Off（关闭）**，并在广告平台上审查该更改 |
| **Needs attention（需要关注）** | 授权或密钥已被撤销 | 选择 **Reconnect（重新连接）** |

限制：每个连接对应一个 Windsor.ai 组织。登录始终包含提供商的 `delete` 作用域。可用操作取决于 Windsor.ai 托管服务器所暴露的内容。

## 相关指南

- [GSC Wizard](gsc-wizard.md) — 来自 Search Console 和 Bing Webmaster 的搜索性能。
- [Set action permissions](action-permissions.md)
- [Windsor.ai MCP](https://mcp.windsor.ai/)

## 来源

- [Windsor.ai definition](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/windsor-ai.json) — 方法、作用域、回调要求、密钥字段和警告。
