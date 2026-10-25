---
paperclip_version: v2026.1023.0
seo_title: Arcade 连接器
seo_description: 连接在 Arcade 网关中选择的工具，选择智能体访问权限，并检查身份验证和操作权限。设置未经过验证。
---

# Arcade

Arcade 允许智能体访问您在 Arcade 网关中选择的工具。该网关可以将来自多个服务器的工具组合在一个 URL 之后。

Arcade 在每个实例上均可用；无需启用实验性设置。

## 连接前准备

- 一个 Arcade 账户和一个包含您希望暴露的工具的网关。
- 网关 URL 及其所有者选择的身份验证模式。

Arcade 支持浏览器登录和基于请求头的身份验证。请遵循您网关的设置；仅凭 URL 并不能证明无需身份验证。

## 连接 Arcade

> **未经验证的设置：** 此流程遵循固定的 Paperclip 定义和 Arcade 的文档。它尚未通过实时 Arcade 连接进行测试。以下每个步骤均未经过验证。

1. **未经验证：** 在 Arcade 中，创建或选择一个网关，并选择智能体应使用的工具。复制其 URL，格式为 `https://api.arcade.dev/mcp/YOUR-GATEWAY-SLUG`。
2. **未经验证：** 打开 **Connectors**（连接器）并选择 **Arcade**。
3. **未经验证：** 阅读主按钮上方的访问行。选择 **Change**（更改）以选择身份或限制可使用该身份的智能体。
4. **未经验证：** 粘贴网关 URL。如果要求登录，请登录；或者在选择 **Change**（更改）后，在 **Authentication**（身份验证）下输入所需的令牌和请求头。基于请求头的网关可能同时需要授权令牌和最终用户标识符；请使用网关所有者指定的值。
5. **未经验证：** 选择 **Connect Arcade**（连接 Arcade），然后在允许智能体工作之前检查该连接的操作列表。

## 选择访问权限

工具来自您的 Arcade 账户，并在您连接时显示在 Paperclip 中。Arcade 控制网关暴露哪些工具。Paperclip 控制智能体可以调用的已暴露操作；它不配置网关底层的应用程序账户。

每个工具初始状态均为 **Allowed**（允许）。在需要时，在 **Permissions**（权限）中将操作设置为 **Ask first**（先询问）或 **Off**（关闭）。在 **Refresh actions**（刷新操作）之后以及更改网关的工具选择之后，请审查该列表。

## 查看已连接的应用程序账户

打开已保存网关的 **Permissions**（权限）页面，在账户发现可用时，可查看 **Connected apps**（已连接的应用程序）和 **Refresh Arcade**（刷新 Arcade）。该列表仅限于配置的 Arcade 用户和此网关暴露的工具。

如果同步需要单独的凭据，请在网关菜单中使用 **Set up account sync**（设置账户同步）。为相同的 Arcade 项目输入 **Project API key**（项目 API 密钥）及其 **Arcade user ID**（Arcade 用户 ID），然后选择 **Save and sync**（保存并同步）。该密钥单独存储，仅用于账户同步；您的网关可以在没有它的情况下正常工作。

账户显示在 **Connectors**（连接器）中的其应用程序卡片下，标记为 **Managed by Arcade**（由 Arcade 管理）。使用 **Open in Arcade**（在 Arcade 中打开）在上游进行管理。Paperclip 访问权限保留在已保存的网关上，并覆盖其暴露的应用程序。

成功的网关设置并不能证明特定应用程序已获得授权。智能体在首次使用应用程序时会检查应用程序访问权限，并且可能仍需您协助进行提供商授权。

## 尝试使用

要求一个符合条件的智能体执行您的网关暴露的一个只读查找操作。将结果与源应用程序进行比较，并在 Paperclip 中检查连接器调用。

> **未经验证的检查：** 这是一个建议的测试，而非记录的结果。请使用您自己的连接中存在的一个操作；在首次检查时不要发送消息或更改记录。

## 故障排除和限制

| 问题 | 检查项 |
| --- | --- |
| 链接检查失败 | 在 Arcade 中确认完整的网关 URL 及其所需的身份验证模式。 |
| 缺少某个工具 | 检查网关的工具选择，然后使用 **Refresh actions**（刷新操作）。 |
| 工具无法访问应用程序 | 在 Arcade 中检查上游应用程序授权。 |

可用的工具取决于网关。成功的连接并不能证明每个上游应用程序都已获得授权。

## 相关指南

- [连接器概述](https://paperclip.ing/product/connectors/arcade/)

- [连接器访问权限如何工作](access-model.md)
- [设置操作权限](action-permissions.md)
- [连接自定义 MCP 服务器](custom-mcp-servers.md)
- [Arcade 网关文档](https://docs.arcade.dev/en/operate/governance/mcp-gateways)

## 来源

- [Paperclip 连接器定义](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/packages/shared/src/app-definitions/arcade.json#L16) — 在固定的产品版本下的方法名称、身份验证、端点和连接器特定的限制。提供商设置文档链接在上方。
- [目录和账户分组](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/ui/src/pages/apps/Browse.tsx) — 过滤器、分组的账户行和网关菜单。
- [已连接的应用程序列表](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/ui/src/pages/apps/app-detail/ConnectedAggregatorApps.tsx) — 刷新控件和验证状态。
