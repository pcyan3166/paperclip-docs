---
paperclip_version: v2026.1023.0
seo_title: Airtable 连接器
seo_description: 允许智能体读取和更新 Airtable 记录。涵盖Airtable Base（数据库）和 Table（数据表）的访问范围、企业白名单机制、一项不改变任何数据的读取测试，以及故障排查。
---

# Airtable

智能体可以处理您的 Airtable 基础表——查找记录、读取字段，以及创建或更新行。

## 连接前准备

- 一个具有访问权限的 Airtable 账户，用于访问您希望智能体使用的基础表。
- 在企业版计划中，管理员可能需要先将客户端加入白名单，之后任何用户才能进行连接。如果授权被拒绝或一直处于待定状态，通常就是此原因所致。

## 连接 Airtable

1. 打开 **Connectors**（连接器）并选择 **Airtable**。
2. 阅读主按钮上方的访问说明行。该行说明了此连接采用的登录身份，以及哪些智能体可以使用它。若要选择其他身份、缩小智能体范围或使用其他登录方法，请选择 **Change**（更改）。
3. 选择 **Continue to sign in**（继续登录）并完成浏览器登录，在 Airtable 询问时选择要共享的内容。

Paperclip 会自动向 Airtable 注册其客户端，因此无需在开发者控制台中进行任何设置。登录需要您的 Paperclip 地址为 HTTPS，或为本地 `localhost` 地址。

### 改用个人访问令牌

如果浏览器登录不适合您——例如，您希望连接的访问范围由令牌固定，而非由个人的登录状态决定——请选择 **Change**（更改）并选择 **Use a personal access token**（使用个人访问令牌）。

1. 在 [airtable.com/create/tokens](https://airtable.com/create/tokens) 创建令牌。仅授予令牌那些您计划允许的操作实际所需的权限范围（scopes）和基础表。
2. 将其粘贴到 **Your Airtable key**（您的 Airtable 密钥）字段中，然后选择 **Connect**（连接）。令牌以 `pat` 开头。

Paperclip 将令牌作为机密信息存储，并在每次调用时发送给 Airtable；智能体永远无法看到它。令牌本身以及 Airtable 用户在每个基础表上的角色都会限制连接可以执行的操作，因此使用范围狭窄的令牌是一个真正的边界。企业管理员可能仍需要将 MCP 访问加入白名单。

> **提示：** 在首次尝试写入操作时，请使用一次性或开发用的基础表。

## 选择访问权限

Airtable Base（数据库）和 Table（数据表）的访问范围来自 Airtable：即授权账户能够打开的内容，加上您在 Airtable 授权屏幕上选择的内容——或者，如果使用个人访问令牌，则是您授予令牌的权限范围和基础表。Paperclip 自身没有基础表选择器，因此请在那里进行限制，或通过授权一个属于较少工作区的账户来限制。

浏览器登录会向 Airtable 请求 `data.records:read`、`data.records:write`、`schema.bases:read`、`schema.bases:write`、`data.recordComments:read`、`data.recordComments:write` 和 `workspacesAndBases:read`。这包括更改记录和基础表架构，因此除非您在 **Permissions**（权限）选项卡中将其设置为只读，否则这不是一个只读连接。

Airtable 基础表通常将参考数据与操作记录混合在一起，而智能体无法区分二者。记录的写入和删除至关重要：

> **警告：** 在 Airtable 中删除记录很难撤销，而在大型基础表中，智能体更新错误字段的情况可能难以察觉。在您观察智能体工作之前，请将写入操作保持为 **Ask first**（先询问），并将删除操作保持为 **Off**（关闭）。Airtable 自身的修订历史是恢复路径。

读取操作可以保持为 **Allowed**（允许）。参见 [设置操作权限](action-permissions.md)。

## 尝试使用

```txt
In the "Customers" base, find the record for Northwind and tell me its status and owner fields. Do not change anything.
```

将字段值与基础表进行对比。读取一条您已知其值的记录，可以在不改变数据的情况下确认凭据、基础表共享设置以及智能体的权限——而且与摘要不同，具体的字段值是可以检查的。

然后打开连接的活动日志，确认 Airtable 调用列在其中。仅凭答案本身并不能证明智能体读取了该记录，而非推断出该结果。

> **注意：** 这是示例任务，而非记录在案的测试结果。请替换为您自己账户中的基础表和记录。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 授权被拒绝或一直处于待定状态 | 企业管理员尚未将客户端加入白名单 | 请 Airtable 管理员将其加入白名单 |
| 缺少某个基础表 | 在授权屏幕上未共享该基础表，或账户无法打开它 | 重新连接并包含该基础表，或在 Airtable 中获取访问权限 |
| 结果中缺少某个字段 | 最可能的原因是该字段类型未被服务器暴露，或 Airtable 权限将其隐藏 | 以授权账户身份在 Airtable 中打开该记录并进行对比。如果您在那里能看到该字段，则问题在于工具界面而非权限 |
| 写入操作被拒绝 | Airtable 拒绝了该操作——字段验证、必填字段，或视图或表格限制 | 在更改任何内容之前，先阅读操作原始响应中提供方的错误文本；它比本表能更精确地指明原因 |
| 记录被意外更改 | 写入操作被设置为 **Allowed**（允许） | 从 Airtable 的修订历史中恢复，然后收紧设置 |
| 使用个人访问令牌的连接无法访问某个基础表 | 该基础表未添加到令牌中，或令牌缺少权限范围 | 在 Airtable 中编辑令牌，或创建新令牌并重新连接 |
| **需要关注** | 授权已在 Airtable 中撤销，或令牌已被删除 | 选择 **Reconnect**（重新连接） |

限制：每个连接仅支持一个 Airtable 账户或令牌。Paperclip 内部没有基础表或表格过滤器。Airtable 的 API 速率限制适用，因此大型扫描速度较慢。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/airtable/)

- [Google Sheets](google-sheets.md) — 用于电子表格数据，具有更严格的权限范围选项。
- [设置操作权限](action-permissions.md)
- [连接器访问机制说明](access-model.md)
- [Using the Airtable MCP server](https://support.airtable.com/articles/9897799762-using-the-airtable-mcp-server)
