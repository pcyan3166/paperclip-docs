---
paperclip_version: v2026.1005.0
seo_title: Cloudflare 连接器
seo_description: 让智能体能够操作 Cloudflare 账户、区域和资源。了解单一连接可达范围、配置风险、读取测试及修复方法。
---

# Cloudflare

智能体可以通过 Cloudflare 的托管服务器与您的 Cloudflare 账户进行交互——检查区域、DNS 以及您的账户所管理的资源。

> **警告：** 不要假设单一连接可以访问所有 Cloudflare 产品。Cloudflare 发布了一系列功能并会对其进行变更；您的智能体所能获取的是托管服务器向您的凭据暴露的内容。请阅读连接的操作列表，而不是根据 Cloudflare 的产品目录来规划工作。

## 连接前准备

- 一个具有访问权限的 Cloudflare 账户，以便智能体使用所需的区域和资源。
- 如果使用密钥而非浏览器登录，请创建一个仅包含您打算授予的权限的 Cloudflare API 令牌。

Cloudflare 的 API 令牌非常精确——您可以将其范围限定为特定的区域、账户和权限组。这是该连接器上可用的最强控制手段，值得使用，而不是使用账户范围的令牌。

## 连接 Cloudflare

1. 打开 **Connectors** 并选择 **Cloudflare**。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他登录方法，请选择 **Change**。
3. 检查您将如何认证。设置从推荐方法开始；若要使用另一种方法，请选择 **Change**：
   - **Sign in with Cloudflare** — 浏览器登录，携带已登录用户的访问权限。
   - **Use an API key** — 粘贴 Cloudflare API 令牌，携带您赋予它的精确范围。
4. 完成设置。

如果您希望获得狭窄且可审计的边界，请优先选择 API 令牌路径。如果您希望获得最快的设置速度且账户已受限，请优先选择浏览器登录。

## 选择访问权限

可达范围由 Cloudflare 决定：即凭据允许的账户、区域和资源。Paperclip 中没有区域选择器。

有三类值得区分的操作，因为它们带来的后果截然不同：

| 类别 | 操作类型 | 建议设置 |
| --- | --- | --- |
| 读取 | 列出和读取配置或分析数据 | **Allowed** |
| 配置写入 | 更改流量路由、过滤或缓存的方式 | **Ask first** |
| 配置与删除 | 创建或移除资源 | 除非特别需要，否则设为 **Off** |

> **注意：** Cloudflare 的托管端点覆盖广泛的产品表面，其工具列表由提供商决定变更。请将 **Permissions** 选项卡上的实时列表针对您实际建立的连接分为这三类，而不是假设特定的 DNS 或防火墙工具存在——可用性还取决于您令牌上的权限组。

> **警告：** 此处的配置更改会对实时流量生效。DNS 或防火墙编辑可能在几秒钟内使网站离线，而配置新资源可能会在账户上产生费用。请将写入操作置于审批之后。

参见 [Set action permissions](action-permissions.md)。

## 尝试使用

```txt
List the Cloudflare zones this account can see and tell me the DNS record count for one of them. Do not change any record.
```

将区域列表与 Cloudflare 仪表板进行对比，并检查其是否恰好包含您打算让令牌覆盖的区域——不多不少。区域列表既确认了凭据，又同时显示了范围，这是捕捉令牌范围比预期更广的最快方式。

然后确认这些调用出现在连接的活动记录中，以便您知道列表来自 Cloudflare 而非智能体的记忆。

不要在实时区域上通过 DNS 或防火墙更改进行验证。

> **注意：** 这是示例任务，而非记录的测试结果。

## 故障排除与限制

| 问题 | 可能原因 | 修复方法 |
| --- | --- | --- |
| 智能体看到的区域少于预期 | API 令牌的范围限定为特定区域 | 有意扩大令牌范围，或接受该边界 |
| 操作被拒绝 | 令牌缺少该权限组 | 在 Cloudflare 中添加权限，或保持该功能关闭 |
| 预期功能缺失 | Cloudflare 服务器未向此凭据暴露该功能 | 使用 **Refresh actions**；否则该功能不可用 |
| 配置更改破坏了实时流量 | 写入操作被设置为 **Allowed** | 在 Cloudflare 仪表板中回滚，然后将写入操作设置为 **Ask first** |
| 出现意外费用 | 配置操作创建了可计费资源 | 审查 Cloudflare 账单；将配置操作设置为 **Off** |
| **Needs attention** | 令牌已被撤销或登录已过期 | 选择 **Reconnect** |

限制：每个连接仅支持一个 Cloudflare 凭据。Paperclip 内部没有区域过滤器。Paperclip 无法撤销 Cloudflare 配置更改。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/cloudflare/)

- [Netlify](netlify.md) — 另一个托管和部署连接器。
- [Set action permissions](action-permissions.md)
- [Answer a connector review request](review-requests.md)
- [Cloudflare MCP servers](https://developers.cloudflare.com/agents/model-context-protocol/cloudflare/servers-for-cloudflare/)
