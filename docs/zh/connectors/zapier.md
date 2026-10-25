---
paperclip_version: v2026.1012.0
seo_title: Zapier 连接器
seo_description: 通过 Zapier MCP 服务器连接数千款应用。生成的 URL 即为凭据——如何安全设置、选择暴露的操作以及进行故障排查。
---

# Zapier

Zapier 允许智能体访问您已在 Zapier 账户中连接的应用。您在 Zapier 中构建一个 MCP 服务器，选择其暴露的操作，并将生成的 URL 提供给 Paperclip。

> **危险：** 生成的 Zapier URL 包含其自身的访问令牌。它是凭据，而不仅仅是一个地址——任何拥有该 URL 的人都可以调用服务器暴露的操作。请像对待密码一样对待它：切勿将其粘贴到工单、聊天消息、截图或代码提交中。

Zapier 在所有实例上均可用；无需启用任何实验性设置。

## 连接前准备

- 一个 Zapier 账户，其中已连接您希望访问的应用。
- 在 Zapier 中创建的一个 MCP 服务器，并已在其上启用您希望暴露的具体操作。

智能体可以调用的操作由 Zapier 决定，而非 Paperclip。这是连接前需要正确配置的主要事项：启用了广泛操作的服务器会赋予智能体广泛的访问权限，而 Paperclip 无法在 Zapier 所宣传的范围之外进一步缩小权限。

## 连接 Zapier

1. 在 Zapier 中创建一个 MCP 服务器，并仅启用智能体应具备的操作。请从最小可用集合开始。
2. 复制服务器生成的完整连接 URL。
3. 打开 **Connectors**（连接器）并选择 **Zapier**。
4. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他登录方法，请选择 **Change**（更改）。
5. 粘贴完整的 **MCP server URL**（MCP 服务器 URL）并选择 **Connect Zapier**（连接 Zapier）。Paperclip 会验证其能否访问该服务器，然后将 URL 存储为机密信息。

请完全按照 Zapier 生成的方式粘贴 URL，包括令牌部分。截断的 URL 会导致检查失败。

## 选择访问权限

存在两个独立的层级，了解何时使用哪一层级很有必要：

| 层级 | 控制位置 | 用途 |
| --- | --- | --- |
| 哪些操作存在 | Zapier，在 MCP 服务器上 | 完全移除能力 |
| 智能体是否可调用已暴露的操作 | Paperclip，在 **Permissions**（权限）选项卡上 | 要求审批，或将其关闭 |

在 Zapier 中更改已暴露的操作集无需重新连接——之后请在 Paperclip 中使用 **Refresh actions**（刷新操作）以重新读取列表。

> **警告：** 在此连接中，您在 Zapier 中暴露的操作一旦刷新，即在 Paperclip 中变为 **active**（活动状态），受现有策略管辖，而非等待审批。扩大 Zapier 侧的范围即为访问决策；请在每次刷新后审查 Paperclip 列表。

Zapier 中任何可以编辑该 MCP 服务器的人都可以扩大智能体的访问范围，而无需触及 Paperclip。请保持可编辑人员列表精简，并在审查连接时一并审查该列表。

由于 Zapier 操作通常作用于真实系统——发送消息、创建记录——其中大多数被归类为写入操作。活动操作初始状态为 **Allowed**（允许）；如果您希望在了解智能体如何使用它们的同时审查调用，请将其更改为 **Ask first**（先询问）。参见 [Set action permissions](action-permissions.md)。

## 试用

首次检查时，请故意选择一个只读操作——例如在已连接的应用中查找记录或列出最近的项目——并以您打算使用的智能体身份运行它。

```txt
Use Zapier to look up the most recent row in my tracking spreadsheet and tell me what it says. Do not create or send anything.
```

只读操作可在不对外部系统执行操作的情况下，确认 URL、服务器及智能体的权限。避免使用发送消息或创建记录的操作进行验证；Zapier 操作通常触及真实目的地。

> **注意：** 此为示例任务，而非记录的测试结果。请替换为您自己的服务器所暴露的操作。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| **Connect Zapier** 失败 | URL 不完整，或服务器已在 Zapier 中被删除 | 从 Zapier 再次复制完整 URL |
| 操作列表为空 | Zapier MCP 服务器上未启用任何操作 | 在 Zapier 中启用操作，然后 **Refresh actions** |
| 您在 Zapier 中启用的操作缺失 | Paperclip 尚未重新读取列表 | 使用 **Refresh actions** |
| 您在 Zapier 中暴露的操作缺失 | Paperclip 的列表副本已过期 | 使用 **Refresh actions**，然后确认新操作的设置——它将在现有策略下以活动状态到达，而非处于关闭状态 |
| 操作在 Zapier 内部失败 | Zapier 中的底层应用连接已过期 | 在 Zapier 中重新连接该应用 |
| 您怀疑 URL 已泄露 | URL 是承载凭据 | 在 Zapier 中重新生成服务器 URL 并在 Paperclip 中重新连接 |

限制：操作界面即为 Zapier 服务器所暴露的内容，Paperclip 无法查看或限制 Zap 内部发生的任何超出其所调用操作的行为。Zapier 自身的套餐限制和任务配额适用。轮换 URL 意味着需要重新连接。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/zapier/)

- [Connect your own MCP server](custom-mcp-servers.md) — 适用于您自行运行的服务器。
- [Set action permissions](action-permissions.md)
- [How connector access works](access-model.md)
- [Zapier MCP quickstart](https://docs.zapier.com/mcp/quickstart)

## 来源

- [Connection setup](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/features/connections/ConnectionSetupFlow.tsx) — 单屏访问选择和设置行为。
- [Remote MCP setup](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/features/connections/remote-mcp/RemoteMcpConnectionSetup.tsx) — 特定于提供方的连接控制。
