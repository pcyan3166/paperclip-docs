---
paperclip_version: v2026.1023.0
seo_title: Webflow 连接器
seo_description: 让智能体处理 Webflow 站点和 CMS 集合。角色如何限制访问范围，CMS 内容与发布的区别，读取测试，以及故障排查。
---

# Webflow

智能体可以处理您的 Webflow 站点——读取站点结构并操作 CMS 集合及其项目。

实际应用场景是 CMS 内容：智能体起草、检查或更新集合项目。Webflow 还在同一端点暴露了用于画布、样式和布局的 Designer 工具，但它们附带一个通常不适合智能体工作的先决条件——请参阅 [Designer 工具](#designer-tools)。

## 连接前准备

- 一个具有您希望智能体使用的站点访问权限的 Webflow 账户。
- 您的工作区和站点角色决定了连接可以触及的范围。受限的角色意味着受限的连接，这通常正是您所需要的。

## 连接 Webflow

1. 打开 **Connectors** 并选择 **Webflow**。
2. 阅读主按钮上方的访问说明行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择其他身份、缩小智能体范围或使用其他登录方法，请选择 **Change**。
3. 选择 **Continue to Webflow** 并完成浏览器登录，在 Webflow 询问时授权您想要的站点。

Paperclip 会自动注册其客户端，因此无需在开发者控制台中进行任何配置。

## 选择访问权限

站点访问范围由 Webflow 决定：即授权账户在其工作区和站点角色下可访问的站点，加上您已授权的内容。Paperclip 没有站点选择器。

最重要的区别在于更改内容与使其上线之间：

| 操作 | 效果 |
| --- | --- |
| 读取站点、集合和项目 | 不产生任何更改 |
| 创建或更新 CMS 项目 | 更改 Webflow 中的内容 |
| 发布 | 使更改在实时站点上可见 |

> **警告：** 发布是具有重大后果的操作。一次发布可能会将站点上的所有待处理更改推送到实时站点，而不仅仅是智能体正在处理的那个项目——包括同事已暂存但尚未完成的编辑。请将发布操作保持为 **Off**，并让人工在 Webflow 中执行发布。

有关设置这些权限的每一项，请参阅 [设置操作权限](action-permissions.md)。

<a id="designer-tools"></a>
## Designer 工具

Webflow 的 MCP 服务器确实暴露了 Designer 工具——画布、样式和布局——它们由本连接器使用的同一端点提供服务。**它们并非不可用；而是有条件的。**

此处需要使用配套应用。Webflow 指出，“MCP Companion App 必须保持打开状态并在 Webflow Designer 中运行，Designer API 工具才能正常工作”，该应用需从 Designer 打开时的 Apps 面板启动。关闭它，画布操作将停止工作。

这一先决条件不适合无人值守的智能体工作，因为它要求人员在整个过程中保持 Designer 打开。请将 Designer 工作规划为由人员驱动的任务，并使用此连接器处理 CMS 内容。

> **注意：** 我们尚未通过 Paperclip 运行 Webflow Designer 工具。这里引用的是提供商自身的要求，并非 Paperclip 的实测结果——因此如果您尝试使用，请将其视为未经验证，并在一个无关紧要的站点上开始。Webflow 的 [MCP getting started](https://developers.webflow.com/mcp/reference/getting-started) 是当前工具界面的权威参考。

## 试用

```txt
List the CMS collections on the marketing site and tell me how many items are in the blog collection. Do not create, change, or publish anything.
```

与 Webflow 仪表板进行对比。读取集合结构可在不触及内容或实时站点的情况下确认凭据和站点范围。

> **注意：** 这是示例任务，而非记录的测试结果。请替换为您自己账户中的站点和集合。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 缺少某个站点 | 账户的工作区或站点角色未包含该站点，或未对其进行授权 | 在 Webflow 中调整角色，或重新连接并授权该站点 |
| 缺少某个 CMS 字段 | 某些字段类型未暴露，或该字段位于不同的集合中 | 在 Webflow 中检查集合架构 |
| 更改在实时站点上不可见 | CMS 更改需要发布 | 当更改准备好时，从 Webflow 进行发布 |
| 站点意外发布 | 发布操作被设置为 **Allowed** | 将其设置为 **Off**；在 Webflow 中审查已上线的内容 |
| Designer 工具失败或不可用 | MCP Companion App 未在 Webflow Designer 中打开；它必须保持打开状态 Designer 工具才能工作 | 从 Apps 面板打开它，或直接在 Designer 中执行设计工作 |
| **需要关注** | 授权已被撤销 | 选择 **Reconnect** |

限制：每个连接仅限一个 Webflow 账户。Paperclip 内部没有站点过滤器。Designer 工具需要在 Designer 中保持打开的伴随应用，因此不适合无人值守工作。Webflow 套餐的限制适用于 CMS 项目数量和 API 速率。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/webflow/)

- [Wix](wix.md) — 另一个网站平台连接器。
- [设置操作权限](action-permissions.md)
- [连接器访问机制说明](access-model.md)
- [Webflow MCP getting started](https://developers.webflow.com/mcp/reference/getting-started)
