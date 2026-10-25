---
paperclip_version: v2026.1012.0
seo_title: Composio 连接器
seo_description: 连接 Composio Connect 或已配置的会话 URL。了解应用授权、聚合器权限以及未经验证的设置步骤。
---

# Composio

Composio 允许智能体通过 Composio Connect 发现和使用应用。应用授权在 Composio 中完成，而访问选择在 Paperclip 中进行。

Composio 在每个实例上均可用；无需启用实验性设置。

## 连接前准备

- 一个 Composio 账户以及您打算授权的应用的访问权限。
- 对于外部配置的会话，需要其所有者提供的 URL 和必需的请求头。

## 连接 Composio

> **未经验证的设置：** 此流程遵循固定的 Paperclip 定义和 Composio 的文档。尚未通过实时 Composio 连接进行测试。以下每个步骤均未经验证。

1. **未经验证：** 打开 **连接器** 并选择 **Composio**。
2. **未经验证：** 阅读主按钮上方的访问行。选择 **更改** 以选择身份或缩小可使用该身份的智能体范围。
3. **未经验证：** 选择 **连接 Composio** 以使用默认的 `https://connect.composio.dev/mcp` 端点并完成浏览器登录。对于外部配置的会话，打开 **重用现有会话** 并在 **更改** 下提供其 URL 和必需的请求头。
4. **未经验证：** 完成连接检查并检查在 Paperclip 中暴露的操作。
5. **未经验证：** 当 Composio 请求对上游应用进行授权时，在完成该独立的浏览器流程之前，检查该应用及请求的访问权限。

请勿用传统的服务器创建配方替代 Composio Connect。Composio 现在为 SDK 创建的端点记录会话；请使用您实际拥有的端点的说明。

## 选择访问权限

工具来自您的 Composio 账户，并在您连接时显示在 Paperclip 中。Composio Connect 暴露了可以发现和执行上游应用的操作。Paperclip 中针对执行操作的设置控制该暴露的调用；它不会为嵌套在其中的每个操作提供单独的权限开关。

每个工具初始状态均为 **允许**。在 **权限** 中审查广泛的执行和连接管理操作，并根据需要选择 **先询问** 或 **关闭**。同时在 Composio 中限制上游应用的授权。

## 通过已保存的账户连接应用

对于没有原生设置的应用，在其目录卡片上选择 **连接**，如果提供多个提供商，则选择 Composio。选择一个已保存的 **Composio 账户** 并选择 **继续**。若要保存另一个网关，选择 **连接新账户…** 并选择 **连接新账户**。

Paperclip 会检查该应用是否已处于活动状态。如果需要授权，请使用 **在 Composio 中连接 {app}**，完成托管登录，返回并选择 **我已连接**。Paperclip 在报告已连接之前会再次检查该账户。此流程不会创建任务，也不会启动智能体运行。

过期或不确定的交接不会静默重试。**获取新链接** 允许您显式请求另一个链接。重用已保存的网关会保留其访问和操作设置；它不会为此单一应用添加新的智能体授权或创建隔离的权限。

## 查看和刷新应用账户

网关的 **权限** 页面列出 **已连接的应用**，并在首次加载时启动检查。**刷新 Composio** 会刷新该已保存的网关。相同的操作也位于 **连接器** 的菜单中；浏览操作也会在后台检查支持的公共 Composio 目录。

导入的账户显示在其应用卡片下，标记为 **由 Composio 管理** 并附带已保存的网关名称。失败的检查会将最后已知的账户保留为 **未验证**；过期的账户显示 **需要登录**。这些是对 Composio 授权的观察结果，而非新的 Paperclip 连接或授权。

使用 **在 Composio 中打开** 来管理导入的账户。它将打开 Composio Connect 的应用区域，该区域与平台开发者项目是分开的。从 Paperclip 中移除已保存的网关不会删除其在 Composio 中的应用账户。

## 尝试使用

要求一个符合条件的智能体在您已授权的应用中发现一个只读查找操作。仅批准查找操作，将其结果与应用进行比较，并检查连接器调用。

> **未经验证的检查：** 建议的测试，而非记录的结果。仅发现操作并不能证明应用执行已获授权。

## 故障排除和限制

| 问题 | 检查项 |
| --- | --- |
| 应用要求登录 | 完成其独立的 Composio 授权；Paperclip 访问权限不会授权该应用。 |
| 授权链接已过期 | 通过 Composio 请求新链接。 |
| 配置的会话拒绝访问 | 与会话所有者核实其 URL 和必需的请求头。 |
| 应用操作失败 | 在 Composio 中检查应用连接，如有需要则重新授权。 |

连接器的范围取决于上游账户和所选端点。移除智能体的 Paperclip 访问权限不会删除这些上游授权。

## 相关指南

- [连接器概述](https://paperclip.ing/product/connectors/composio/)

- [连接器访问权限如何工作](access-model.md)
- [设置操作权限](action-permissions.md)
- [重新授权、撤销或断开连接](reauthorize-and-disconnect.md)
- [Composio Connect 文档](https://docs.composio.dev/docs/composio-connect)
- [会话端点](https://docs.composio.dev/docs/sessions-via-mcp)

## 来源

- [Paperclip 连接器定义](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/app-definitions/composio.json#L20) — 在固定的产品版本下的方法名称、身份验证、端点以及特定于连接器的限制。提供商设置文档链接见上文。
- [目录和账户分组](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/pages/apps/Browse.tsx) — 过滤器、分组的账户行以及网关菜单。
- [已连接应用列表](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/pages/apps/app-detail/ConnectedAggregatorApps.tsx) — 刷新控件和验证状态。
