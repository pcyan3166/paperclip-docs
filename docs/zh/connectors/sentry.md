---
paperclip_version: v2026.1005.0
seo_title: Sentry 连接器
seo_description: 让智能体调查 Sentry 错误、发布版本及生产环境问题。涵盖组织与项目访问范围、安全的读取测试以及故障排查。
---

# Sentry

智能体可以调查 Sentry 中的错误——读取问题、堆栈跟踪、发布版本以及生产问题周围的上下文信息。

这是一个适合提供给调试智能体的连接器，因为有用工作几乎完全属于读取操作。

## 连接前准备

- 一个具有访问权限的 Sentry 账户，能够访问您希望智能体调查的组织（organization）和项目（project）。
- 如果您的 Sentry 组织限制第三方集成，管理员可能需要先批准该连接。

## 连接 Sentry

1. 打开 **Connectors** 并选择 **Sentry**。
2. 阅读主按钮上方的访问权限说明行。该行说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他登录方法，请选择 **Change**。
3. 完成 Sentry 的浏览器登录，并在提示时选择组织。

Paperclip 会自动向 Sentry 注册其客户端，因此无需在开发者控制台中进行任何配置。这是受支持的路径，也是本页面所记录的路径。

## 选择访问权限

访问范围（Reach）是指授权 Sentry 账户的：登录期间选择的组织，以及该账户在该组织内可见的项目。Sentry 自身的团队成员资格和角色决定其余权限。

> **注意：** 连接器的定义中提到了组织、项目和环境范围，但这些仅是描述性的——Paperclip 不会在 Sentry 授予的权限之上强制执行项目或环境过滤器。如果智能体应仅查看一个项目，请使用团队成员资格仅限于该项目的账户进行授权。

大多数 Sentry 工作属于读取操作。问题变更——解决、忽略、分配、删除——会改变您的值班工程师看到的内容，因此请将它们保持在 **Ask first** 或 **Off** 状态。如果智能体解决了一个误诊的问题，就会隐藏一个正在发生的问题。请参阅 [Set action permissions](action-permissions.md)。

> **警告：** 堆栈跟踪和错误上下文通常包含用户数据、请求负载，有时还包含意外记录的密钥。读取 Sentry 的智能体可能会将这些内容暴露到任务记录中。对于此连接器，建议优先选择 **Just agents I pick** 而非 **Any agent**。

## 尝试使用

```txt
Find the most frequent unresolved Sentry issue from the last 24 hours and summarize the error and where it happens. Do not resolve or assign anything.
```

与 Sentry 仪表板进行对比。读取已知问题可以确认凭据、组织以及智能体的权限，而不会改变您团队的分诊状态。

> **注意：** 这是示例任务，而非记录在案的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 授权被拒绝或保持待定状态 | Sentry 组织限制第三方集成 | 请 Sentry 管理员批准 |
| 连接正常但找不到任何问题 | 登录期间选择了错误的组织 | 重新连接并选择正确的组织 |
| 某些项目不可见 | 授权账户的团队成员资格不包含这些项目 | 在 Sentry 中将该账户添加到这些团队 |
| 智能体无法解决某个问题 | 该操作设置为 **Off**，或账户角色不允许此操作 | 检查 **Permissions** 选项卡，然后检查 Sentry 角色 |
| 某个问题被意外解决 | 某项变更操作被设置为 **Allowed** | 在 Sentry 中重新打开该问题并收紧操作设置 |
| **Needs attention** | 授权已过期或被撤销 | 选择 **Reconnect** |

限制：每个连接仅限一个组织。Paperclip 内部没有项目或环境过滤器。Sentry 的数据保留期限决定了智能体可以回溯查看的时间范围。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/sentry/)

- [PagerDuty](pagerduty.md) — 事件响应，常与错误跟踪配合使用。
- [Set action permissions](action-permissions.md)
- [How connector access works](access-model.md)
- [Sentry MCP server](https://mcp.sentry.dev/)
