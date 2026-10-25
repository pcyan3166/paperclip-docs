---
paperclip_version: v2026.1005.0
seo_title: Shopify 连接器
seo_description: 为智能体提供商店的公开目录、政策及购物车工具。这是面向购物者的店面界面，而非用于访问订单或客户数据的 Admin API。
---

# Shopify

智能体可以搜索 Shopify 商店的产品和政策，并操作购物车，其方式与购物者助手相同。

> **警告：** 这是 Shopify 面向购物者的界面，而非 Admin API 访问权限。它**无法**触及商家订单、客户信息、库存管理或 Shopify Admin 中的任何内容——即使是针对商家自己的商店也是如此。如果您需要管理后台数据，此连接器并非正确的途径。

无需登录。这些是公开端点，因此智能体能看到的内容与任何访问店面的访客所能看到的内容一致。

## 连接前准备

- 商店的永久 `your-store.myshopify.com` 域名。自定义域名并非端点，即使那是客户使用的地址。
- 一个**公开**的店面。Shopify 在选定套餐之前会将试用店面保持为私有状态，而受密码保护的店面对所有访问者返回授权错误——Paperclip 无法使用商家的 Admin 会话来绕过此限制。

要使店面公开：选择一个 Shopify 套餐，然后在 Shopify Admin 中打开 **Online Store → Preferences**，将店面可见性设置为公开，并移除密码保护。

## 选择服务器

Shopify 提供两种服务器，Paperclip 同时暴露这两种选项：

| 选项 | 适用场景 |
| --- | --- |
| **Shopify UCP commerce** | 推荐选择。Shopify 当前的目录、购物车和结账工具 |
| **Storefront policies and compatibility tools** | Shopify 的兼容性服务器，适用于智能体主要需要店面政策和常见问题解答搜索的场景 |

> **注意：** 在 UCP 选项下，Paperclip 会自动提供 Shopify 所需的智能体配置文件。目前，在 Paperclip 自身的生产配置文件建立之前，它发送的是 Shopify 文档中记录的托管配置文件示例。

## 连接 Shopify

1. 确认店面已公开。
2. 打开 **Connectors** 并选择 **Shopify**。
3. 阅读主按钮上方的访问说明行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他登录方法，请选择 **Change**。
4. 检查服务器选项，并选择 **Change** 以选择其他选项。
5. 将 **Store domain** 输入为裸永久主机——`your-store.myshopify.com`，不带 `https://` 且末尾无路径。Paperclip 会验证格式并拒绝任何非 `myshopify.com` 主机的输入。
6. 完成设置。

## 选择访问权限

由于端点是公开的，该连接不授予任何特权访问权限：边界即为店面本身。连接不会暴露任何购物者原本无法找到的内容。

需要关注的是写入操作。购物车和结账操作是针对商店的真实操作：

> **警告：** `cancel-cart`、`cancel-checkout` 和 `complete-checkout` 被归类为破坏性操作。完成结账即视为购买。除非智能体确实需要这些功能，否则请将其保持为 **Off**，并至少使用 **Ask first**。

产品和政策查询属于读取操作，保持为 **Allowed** 是安全的。参见 [Set action permissions](action-permissions.md)。

一个连接对应一个商店。若要连接第二个商店，请再次连接 Shopify。

## 试用

查找一个您可以在店面上自行确认的产品：

```txt
Search the store for a product called "canvas tote" and tell me its price and whether it is in stock. Do not add anything to a cart.
```

预期结果应与公开产品页面匹配。首次检查仅限于产品或政策查询——任何与购物车相关的操作都会启动真实的商务流程。

> **注意：** 此为示例任务，非记录测试结果。请替换为商店中的实际产品。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 设置因授权错误而失败 | 店面无法公开访问——它受密码保护，或商店未使用提供公开店面的套餐 | 确认未登录访客可以加载店面。使店面公开是关于您商店的真实商业决策，而非可轻率对待的设置步骤；如果它应保持私有，则此连接器不可用于该商店 |
| 商店域名被拒绝 | 输入了自定义域名或完整 URL | 使用裸 `your-store.myshopify.com` 主机 |
| 智能体无法看到订单或客户 | 这是预期行为——这不是 Admin API 访问 | 此处无需修复；此连接器不涵盖管理后台数据 |
| 结果中缺少产品 | 它们未发布，或对店面的销售渠道不可用 | 在 Shopify Admin 中发布它们 |
| 政策搜索无结果 | 商店中未填写政策 | 在 Shopify Admin 中添加它们 |
| 结账操作意外完成 | 结账工具是真实的商务操作 | 将其设置为 **Off** 或 **Ask first** |

限制：每个连接对应一个商店，仅限公开店面，无 Admin API。适用 Shopify 自身的速率限制。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/shopify/)

- [Stripe](stripe.md) — 支付数据，具有其自身的边界。
- [Set action permissions](action-permissions.md)
- [How connector access works](access-model.md)
- [Shopify Storefront MCP documentation](https://shopify.dev/docs/apps/build/storefront-mcp/servers/storefront)
