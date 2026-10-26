---
paperclip_version: v2026.1023.0
seo_title: Gauge AI 可见性连接器
seo_description: 允许智能体读取 Gauge 的 AI 可见性、SEO 和流量数据并运行内容工作流，同时将发布操作设置为“先询问”，以确保任何 CMS 上线前都经过人工确认。
---

# Gauge

智能体可以与您公司的 Gauge 组织进行交互——检查 AI 答案引擎如何提及和引用您的品牌，研究关键词和流量，并运行 Gauge 的内容工作流。

> **警告：** Gauge 内容工具可以发布到您在 Gauge 中连接的 CMS，包括直接发布到您的实时站点。在任何智能体无人值守运行之前，请将发布操作设置为**先询问**。

## 连接前准备

- 一个 Gauge 账户。
- 确定此连接所属的 Gauge 组织。每个连接仅对应一个组织：浏览器登录使用您在 Gauge 同意界面上选择的组织，而 API 密钥则使用创建该密钥的组织。

## 连接 Gauge

1. 打开**连接器**并选择**Gauge**。
2. 阅读主按钮上方的访问说明行。它说明了连接所属的身份以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或切换方法，请选择**更改**。
3. 选择一种方法：
   - **使用 Gauge 登录** — 完成浏览器登录，并在 Gauge 的同意界面上选择组织。Paperclip 会自动注册其客户端，因此无需在开发者控制台中进行任何设置。
   - **使用 API 密钥** — 在 Gauge 中，打开**设置 → 集成 → API 密钥**，为 Paperclip 生成一个密钥，并将其粘贴到**您的 Gauge 密钥**字段中。

两种方法都提供相同的**读写**能力：读取 AI 可见性、SEO 和流量数据，并运行内容工作流。

## 选择访问权限

浏览器登录仅向 Gauge 请求 `mcp` 范围。组织信息来自您在同意界面上的选择，而非额外的身份权限范围。

API 密钥的工作方式不同，在选择它之前值得了解这一点：

- Gauge API 密钥没有范围限制。密钥可以访问创建它的组织中的每个工具，包括发布功能。
- 使用密钥发出的请求归属于组织，而非个人。
- Paperclip 无法缩小现有密钥的权限范围。请仅为 Paperclip 生成一个单独的密钥，以便您可以独立删除它。

写入工具默认启用，因此**权限**选项卡上的操作列表是您决定智能体可以执行哪些操作的地方。读取可见性和关键词报告的风险较低。任何发布内容的操作至少应设置为**先询问**，以便人工审核者在内容发布到您的站点之前查看每次发布内容。参见[设置操作权限](action-permissions.md)。

## 试用

```txt
Using Gauge, summarise how often AI answer engines mentioned our brand over the last month and list the three most-cited pages. Do not create or publish any content.
```

将数字与 Gauge 仪表板进行比较。您可以核对的报告可以在不更改任何内容的情况下确认凭据和所选组织。

> **注意：** 这是示例任务，而非记录在案的测试结果。

## 故障排除与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 数据来自错误的组织 | 在同意时选择了不同的组织，或密钥来自另一个组织 | 重新连接并选择正确的组织，或使用在该组织中创建的密钥 |
| 内容未经审核即上线 | 发布操作被设置为**允许** | 在**权限**选项卡上将发布操作设置为**先询问**或**关闭** |
| API 密钥被拒绝 | 密钥已在 Gauge 中删除或复制不完整 | 在**设置 → 集成 → API 密钥**下生成新密钥并重新连接 |
| **需要关注** | 授权或密钥已被撤销 | 选择**重新连接** |

限制：每个连接对应一个 Gauge 组织。API 密钥无法缩小权限范围。可用的操作取决于 Gauge 托管服务器暴露的内容。

## 相关指南

- [连接器访问机制](access-model.md)
- [设置操作权限](action-permissions.md)
- [响应连接器审核请求](review-requests.md) — 当智能体等待“先询问”发布时您看到的内容。
- [Gauge MCP 文档](https://docs.withgauge.com/help/mcp)

## 来源

- [Gauge 定义](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/gauge.json) — 方法、范围、密钥字段、能力和警告。
- [画廊文案](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/ui/src/lib/app-gallery-copy.ts) — 目录卡片文本。
