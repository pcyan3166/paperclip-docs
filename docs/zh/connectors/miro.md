---
paperclip_version: v2026.1023.0
seo_title: Miro 连接器
seo_description: 让智能体读取和更新 Miro 画板。团队和画板的访问范围，实际可用的画板对象类型，读取测试，以及故障排查。
---

# Miro

智能体可以处理您的 Miro 画板——查找画板、读取其内容，以及添加或更新项目。

## 连接前准备

- 一个 Miro 账户，该账户需拥有您希望智能体使用的团队和画板的访问权限。
- 在企业版计划中，管理员可能会限制允许使用的第三方 MCP 客户端。如果授权被拒绝，这通常是原因所在。

## 连接 Miro

1. 打开 **Connectors** 并选择 **Miro**。
2. 阅读主按钮上方的访问权限说明行。该行说明了连接将以谁的身份登录，以及哪些智能体可以使用它。若要选择其他身份、缩小智能体范围或使用其他登录方法，请选择 **Change**。
3. 选择 **Continue to Miro** 并完成浏览器登录，当 Miro 询问时选择团队。

Paperclip 会自动注册其客户端，因此无需在开发者控制台中进行任何配置。

## 选择访问权限

访问范围（Reach）是指授权 Miro 账户所属的团队及其可以打开的画板。Paperclip 中没有画板选择器。

请务实地看待智能体对画板的操作能力。Miro 的连接操作的是结构化的画板对象，而非人类所体验的画布。暴露哪些对象类型由 Miro 决定，且会发生变化——在围绕特定项目类型规划工作之前，请阅读连接 **Permissions** 选项卡上的实时列表，而不是依赖文档中的任何列表，包括本页面的内容：

- 智能体读取对象及其内容和位置。它不像人类那样“看到”画板，因此仅通过布局传达的空间含义可能会丢失。
- 手绘图形、嵌入内容和交互式小部件与结构化项目并不等同。
- 智能体添加的任何内容都将通过编程方式放置，如果放置位置很重要，则需要人工进行整理。

写入操作会落在同事可能正在使用的画板上。在您了解智能体的行为方式之前，请将其保持在 **Ask first** 状态。参见 [Set action permissions](action-permissions.md)。

## 尝试使用

通过 URL 而非名称来识别画板，并将结果与您可以打开的画板进行核对：

```txt
Read this Miro board — https://miro.com/app/board/YOUR_BOARD_ID/ — and list the
sticky notes grouped by their frame. Do not add or move anything.
```

与画板进行对比。读取您可以自行打开的画板可以确认凭据、团队以及智能体的权限，而无需打乱任何人的工作。

> **注意：** 这是示例任务，并非记录在案的测试结果。请替换为您自己团队中的画板名称。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 授权被拒绝 | 企业管理员限制了第三方 MCP 客户端 | 联系 Miro 管理员允许该客户端 |
| 找不到画板 | 授权账户无法打开该画板，或该画板属于另一个团队 | 共享该画板，或重新连接并选择正确的团队 |
| 画板内容似乎不完整 | 手绘、嵌入内容或小部件内容不是结构化的画板对象 | 这是预期行为；这些内容不作为项目暴露 |
| 智能体误解了画板的布局 | 由空间排列传达的含义未包含在对象数据中 | 为智能体提供明确的上下文，而不是依赖布局 |
| 某项目出现在奇怪的位置 | 编程式放置 | 在 Miro 中移动它，并保持写入操作处于 **Ask first** 状态 |
| **需要关注** | 授权已被撤销 | 选择 **Reconnect** |

限制：每个连接仅限一个 Miro 账户。Paperclip 内部没有画板过滤器。仅支持结构化的画板对象。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/miro/)

- [Set action permissions](action-permissions.md)
- [How connector access works](access-model.md)
- [Verify a connector and fix a broken one](verify-and-troubleshoot.md)
- [Enabling Miro's MCP server](https://help.miro.com/hc/en-us/articles/31625301583890-How-to-enable-Miro-s-MCP-Server-user-guide)
