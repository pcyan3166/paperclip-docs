---
paperclip_version: v2026.1023.0
seo_title: 已识别但未列出的提供商
seo_description: 存在于 Paperclip 目录中但未在连接器列表中显示的提供商定义，以及那些通过自身审批机制限制访问的提供商。
---

# Paperclip 识别但不列出的提供商

这是目录背后库存的参考文档。如果您正在寻找要设置的连接器，请从 [Connectors](../../connectors.md) 开始——本页面上的所有内容要么被排除在 **Connectors** 列表之外，要么被提供商阻止，因此目前都无法从目录中进行设置。

这里容易混淆两个不同的概念，因此将它们分开列出。**被扣留（withheld）** 的提供商拥有完整的定义，Paperclip 仍然识别它，但它被故意排除在面向客户的列表之外。**提供商门控（provider-gated）** 的提供商则是因为提供商本身不允许独立注册的客户端连接，因此不予提供。

> **注意：** 已针对 Paperclip **v2026.1023.0** 版本进行检查。目录可见性可能会在版本发布之间发生变化；识别并不保证可用性。现有连接仍需独立验证。

## 从 Connectors 列表中扣留

Paperclip 将识别和可见性作为两件事处理：一个定义可以保留在目录中，同时从商店中隐藏。应用程序源代码中关于隐藏集合的注释给出了原因——将两者分开意味着将提供商从列表中移除并不一定需要破坏某人已经保存的连接。

在此提交版本中我们可以确认的是：以下定义存在，Paperclip 仍然解析它们，并且它们被过滤出您浏览的列表。我们并没有连接或运行其中任何一个，因此之前保存的连接是否仍然有效在此处未经验证。如果您有一个这样的连接，[Verify a connector and fix a broken one](verify-and-troubleshoot.md) 是查明情况的方法，答案具体取决于您的实例。

这些提供商都没有连接器页面，也无法从目录中添加。

截至 Paperclip v2026.1023.0，Bitly、Context7 和 Make 已退出此列表并重新回到目录中——参见 [Bitly](bitly.md)、[Context7](context7.md) 和 [Make](make.md)。ClickUp、Klaviyo 和 monday.com 已加入此列表：它们的定义存在，但登录尚未端到端验证，因此它们仍被排除在您浏览的列表之外。

| 提供商 | Slug |
| --- | --- |
| beehiiv | `beehiiv` |
| Brex | `brex` |
| Candid | `candid` |
| ClickUp | `clickup` |
| Coda | `coda` |
| Egnyte | `egnyte` |
| Embat | `embat` |
| Kernel | `kernel` |
| Klaviyo | `klaviyo` |
| Local Falcon | `local-falcon` |
| Manufact | `manufact` |
| monday.com | `monday` |
| O'Reilly | `oreilly` |
| PlanetScale | `planetscale` |
| Razorpay | `razorpay` |
| Sanity | `sanity` |
| Similarweb | `similarweb` |
| Ticket Tailor | `ticket-tailor` |
| TickTick | `ticktick` |
| Xero | `xero` |

## Composio：重新列出，旧连接已退役

Composio 曾位于上述扣留列表中。它现在作为通过 Composio Connect 的常规远程 MCP 连接重新回到 **Connectors** 列表中——参见 [Connect apps through an MCP aggregator](mcp-aggregators.md)。

通过较旧的 Composio 代理（broker）建立的连接则是另一回事。该代理已退役，因此这些连接不再运行：它们显示为不健康状态，其工具不再提供给智能体，对它们的调用会被拒绝。Paperclip 会在连接本身中告诉您该怎么做：

> *"此旧版 Composio 连接不再受支持。请从 Connectors 添加一个新的 Composio MCP 连接，然后移除此连接。现有的凭据和权限不会迁移。"*

实际操作中：从 **Connectors** 重新连接 Composio，在新连接上设置访问和操作权限，然后移除旧连接。没有任何内容会自动迁移。

## 由提供商门控

这些提供商被记录为被阻止，是因为提供商方面存在要求。这是他们的要求，而非 Paperclip 的要求，必须由他们解除或授予——Paperclip 中没有设置可以绕过此限制。

| 提供商 | 提供商的要求 |
| --- | --- |
| G2 (`g2`) | G2 必须启用跨应用程序令牌内省（cross-application token introspection），独立注册的客户端才能连接。没有客户侧的申请流程。 |
| Vercel (`vercel`) | Vercel 审核并批准 MCP 客户端。批准发生在 Vercel 和客户端发布者之间，因此操作员无法为自己的实例申请。 |
| monday.com (`monday`) | monday.com 在自动客户端注册期间拒绝 Paperclip 的回调地址。monday.com 必须将回调地址加入白名单，登录才能工作；账户管理员可能也需要批准。它也被扣留在上述列表中。 |
| Zomato (`zomato`) | Zomato 限制第三方客户端，并要求将每个重定向 URI 加入白名单。这两个步骤都在 Zomato 一侧进行。 |

如果您现在需要其中之一，不依赖于提供商客户端审批的路径在 [Can't find your app?](../../connectors.md#cant-find-your-app) 中——包括提供商允许的自定义服务器、您已拥有的自动化，或工作区和 CLI。

## 相关

- [Connectors](../../connectors.md) — 您可以实际从中设置的目录。
- [Connect a custom MCP server](custom-mcp-servers.md) — 适用于没有目录条目的服务的路径。
- [Verify a connector and fix a broken one](verify-and-troubleshoot.md) — 检查您已有的连接是否仍然有效。
