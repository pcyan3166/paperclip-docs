---
paperclip_version: v2026.1023.0
seo_title: Bitly 连接器
seo_description: 允许智能体读取 Bitly 链接、二维码和点击分析数据，同时保持链接删除、Sites 发布和自定义域名更改处于关闭状态。
---

# Bitly

智能体可以操作您的 Bitly 账户——查找短链接和二维码、读取点击分析数据，并在您允许的情况下创建和管理链接。

Bitly 出现在每个实例的目录中；无需启用实验性设置。

> **警告：** Bitly 的工具列表包括永久删除短链接、Bitly Sites 的发布与删除，以及自定义域名的更改。删除短链接会导致所有持有该链接的人无法再进行重定向。在需要之前，请将写入和破坏性操作保持为 **Off**（关闭）。

## 连接前准备

- 一个 Bitly 账户，以及您希望智能体使用的工作区。

## 连接 Bitly

1. 打开 **Connectors**（连接器）并选择 **Bitly**。
2. 阅读主按钮上方的访问说明行。它说明了连接所属的身份以及哪些智能体可以使用它。若要选择其他身份、缩小智能体范围或切换方法，请选择 **Change**（更改）。
3. 选择一种方法：
   - **Sign in with Bitly**（使用 Bitly 登录）——完成浏览器登录，并仅选择您打算使用的工作区。Paperclip 会自动注册其客户端。
   - **Use an API key**（使用 API 密钥）——在 Bitly 的 **Settings → API**（设置 → API）下生成一个令牌，并将其粘贴到 **Your Bitly key**（您的 Bitly 密钥）字段中。

## 选择访问权限

这两种方法都具有广泛的权限。浏览器登录请求 `mcp.all` 范围，该范围涵盖 Bitly 的整个工具列表：链接和二维码、分析数据、群组、自定义域名以及 Bitly Sites。API 令牌没有文档记录的缩小范围的方法，因此请将其视为同等情况。Paperclip 无法缩小其中任何一种的权限。

这将实际的控制权交给了 Paperclip 的操作列表。请按以下方式配置这些操作：

| 操作类型 | 建议设置 |
| --- | --- |
| 读取链接、二维码和点击分析数据 | **Allowed**（允许） |
| 创建或更新链接和二维码 | **Ask first**（先询问） |
| 删除链接或 Sites，发布 Sites，更改自定义域名 | **Off**（关闭） |

在连接后立即在 **Permissions**（权限）选项卡上设置这些项。参见 [设置操作权限](action-permissions.md)（设置操作权限）。

## 试用

```txt
Using Bitly, show me the five short links with the most clicks in the last 30 days and their destination URLs. Do not create, change, or delete anything.
```

将计数与 Bitly 仪表板进行核对。像这样的读取操作可以在不更改任何内容的情况下确认凭据和工作区。

> **注意：** 这是示例任务，而非记录在案的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 登录完成但连接失败 | 旧版本上的令牌交换问题 | 更新 Paperclip，然后选择 **Reconnect**（重新连接） |
| 出现来自其他工作区的链接，或缺少预期的链接 | 登录时选择了不同的工作区 | 重新连接并选择预期的工作区 |
| 某个链接停止重定向 | 删除操作被设置为 **Allowed**（允许） | 将破坏性操作设置为 **Off**（关闭）；在 Bitly 中重新创建该链接 |
| API 令牌被拒绝 | 令牌已重新生成或复制不完整 | 在 **Settings → API**（设置 → API）下创建新令牌并重新连接 |
| **Needs attention**（需要关注） | 授权或令牌已被撤销 | 选择 **Reconnect**（重新连接） |

限制：每个连接对应一个 Bitly 账户。无法从 Paperclip 缩小令牌和 `mcp.all` 授权的权限。可用的操作取决于 Bitly 托管服务器所暴露的内容。

## 相关指南

- [连接器访问机制说明](access-model.md)（连接器访问机制说明）
- [设置操作权限](action-permissions.md)（设置操作权限）
- [Bitly MCP quickstart](https://dev.bitly.com/bitly-mcp/overview/quickstart/)（Bitly MCP 快速入门）

## 来源

- [Bitly definition](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/bitly.json)（Bitly 定义）——方法、范围、密钥字段和警告。
- [App catalog visibility](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions.ts)（应用目录可见性）——Bitly 列在目录中。
- [Tool connection service](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/server/src/services/tool-access.ts)（工具连接服务）——OAuth 令牌交换请求 JSON 响应。
