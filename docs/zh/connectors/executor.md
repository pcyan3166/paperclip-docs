---
paperclip_version: v2026.1012.0
seo_title: Executor 连接器
seo_description: 连接托管的 Executor 端点并管理其暴露的操作。包含上游策略边界以及明确未经验证的设置。
---

# Executor

Executor 为已配置的集成提供统一的前置端点。Paperclip 连接到该端点；Executor 处理上游连接及其自身策略。

Executor 在每个实例上均可用；无需启用实验性设置。

## 连接前准备

- 一个 Executor 部署，其中包含您打算暴露的集成和连接。
- 一个可访问的远程端点 URL 及其所需的身份验证详情。

请使用您部署的 URL。Paperclip 的连接器定义不提供默认服务器 URL。

## 连接 Executor

> **未经验证的设置：** 此流程遵循固定的 Paperclip 定义和 Executor 的 MCP Proxy 文档。它尚未通过实时 Executor 连接进行测试。以下每个步骤均未经验证。

1. **未经验证：** 在 Executor 中配置预期的集成、凭据和策略。获取该部署的远程端点 URL。
2. **未经验证：** 打开 **Connectors**（连接器）并选择 **Executor**。
3. **未经验证：** 阅读主按钮上方的访问行。选择 **Change**（更改）以选择身份或限制可使用该连接身份的**智能体**范围。
4. **未经验证：** 粘贴端点 URL。如果需要，请登录，或者根据您部署的说明，在 **Authentication**（身份验证）下添加令牌或标头，方法是先选择 **Change**（更改）。
5. **未经验证：** 选择 **Connect Executor**（连接 Executor）并检查暴露的操作列表。

诸如 `executor mcp` 之类的本地命令不是远程 URL。关于远程连接器和**适配器**级别本地进程之间的区别，请参阅 [连接自定义 MCP 服务器](custom-mcp-servers.md)。

## 选择访问权限

工具来自您的 Executor 账户，并在您连接时显示在 Paperclip 中。可访问的集成取决于 Executor 的配置。

每个工具初始状态均为 **Allowed**（允许）。根据需要，在 **Permissions**（权限）中将暴露的操作设置为 **Ask first**（先询问）或 **Off**（关闭）。Executor 的上游策略是另一层控制；Paperclip 的批准不会覆盖 Executor 的拒绝。如果某个暴露的操作捆绑了多个上游操作，Paperclip 管理该调用，而不是每个内部操作。

## 查看已连接的应用账户

当您的部署暴露受支持的账户清单时，其 **Permissions**（权限）页面会列出 **Connected apps**（已连接应用）并提供 **Refresh Executor**（刷新 Executor）。Paperclip 读取部署的连接和集成清单；它不执行生成的代码来猜测连接了什么。

账户显示在其应用卡片下，标记为 **Managed by Executor**（由 Executor 管理）。未知集成仍保留自身身份，不会被套用推测出的品牌标识。如果无法发现，Paperclip 会说明这一点；仅成功访问工具并不能验证账户的授权。

可选的 **Console URL**（控制台 URL）告诉 Paperclip **Open in Executor**（在 Executor 中打开）应发送您到哪里。请使用您受信任的 HTTPS 集成页面，包括其工作区或自托管路径。您可以在设置期间设置，或从已保存网关的 **Console URL** 菜单项中设置。

尽管其目录过滤器暂时隐藏，Executor 在 **Connectors**（连接器）中仍然可用。网关权限仍然管理其集成中的调用，账户管理仍保留在 Executor 中。

## 尝试使用

要求一个符合条件的**智能体**执行您配置的集成中的一个只读查找。将结果与源系统进行比较，并检查连接器调用。

> **未经验证的检查：** 建议的测试，而非记录的结果。请使用连接中可见的操作，并避免对上游系统进行更改。

## 故障排除和限制

| 问题 | 检查项 |
| --- | --- |
| 无法访问端点 | 检查部署可用性以及 Paperclip 是否能访问提供的 URL。 |
| 登录或标头失败 | 使用该部署的身份验证设置。 |
| 缺少某个集成 | 检查其在 Executor 中的配置和连接，然后在 Paperclip 中 **Refresh actions**（刷新操作）。 |
| 调用被拒绝 | 检查 Paperclip 权限和 Executor 的上游策略。 |

Executor 部署版本可能暴露不同的接口。此页面不保证固定的工具列表或与每个部署版本的兼容性。

## 相关指南

- [连接器概述](https://paperclip.ing/product/connectors/executor/)

- [连接器访问如何工作](access-model.md)
- [设置操作权限](action-permissions.md)
- [连接自定义 MCP 服务器](custom-mcp-servers.md)
- [Executor MCP Proxy 文档](https://executor.sh/docs/mcp-proxy)

## 来源

- [Paperclip 连接器定义](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/app-definitions/executor.json#L16) — 在固定的产品版本下的方法名称、身份验证、端点和连接器特定限制。提供商设置文档链接在上方。
- [目录和账户分组](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/pages/apps/Browse.tsx) — 过滤器、分组的账户行和网关菜单。
- [已连接应用列表](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/pages/apps/app-detail/ConnectedAggregatorApps.tsx) — 刷新控件和验证状态。
