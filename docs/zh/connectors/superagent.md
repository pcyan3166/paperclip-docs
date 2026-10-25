---
paperclip_version: v2026.1023.0
seo_title: Superagent 连接器
seo_description: 允许智能体审查 Superagent 安全发现，启动红队报告，并对内容进行评分。使用专用 API 密钥，并对计费或删除类工具进行门控。
---

# Superagent

您可以让智能体与 Superagent（一个安全平台）协同工作：审查安全发现，启动红队报告，并在智能体信任内容和包之前对其进行评分。这适用于诸如“对本周的发现进行分诊”或“在添加此包之前检查它”之类的任务。

> **注意：** Superagent 出现在每个实例的目录中；无需启用实验性设置。

## 连接前准备

- 一个 Superagent 组织 API 密钥。在 Superagent 中，打开 **Settings → API keys** 并仅为 Paperclip 创建一个密钥。密钥以 `sk_live_` 开头。

使用单独的密钥在此处至关重要。Superagent 密钥没有作用域限制：一个密钥可以访问其组织内的所有内容，包括永久删除发现以及启动消耗您组织积分的工作。仅由 Paperclip 使用的密钥可以在不影响其他任何内容的情况下被撤销。

## 连接 Superagent

1. 打开 **Connectors** 并选择 **Superagent**。
2. 阅读主按钮上方的访问权限行。它说明了该连接属于谁以及哪些智能体可以使用它。选择 **Change** 以缩小智能体范围。
3. 将密钥粘贴到 **Your Superagent key** 字段中，然后选择 **Connect**。
4. 打开 **Permissions** 并在任何智能体无人值守运行之前审查操作列表。

API 密钥是唯一的方法。Paperclip 将密钥作为机密存储，并在每次调用时将其发送给 Superagent；智能体永远不会看到它。

## 选择访问权限

每个操作默认均为 **Allowed**，因此这一步骤比平时更重要。Superagent 暴露了一个庞大的工具列表，其中一些操作会产生费用或无法撤销：

- **计费工作。** 启动报告和分诊发现会消耗组织积分。
- **永久删除。** 删除发现后无法恢复。
- **对智能体保护的更改。** 某些操作会更改规则或撤销 Superagent 所保护的智能体的凭据。

Paperclip 对 Superagent 的操作采取保守的分类方式以提供帮助。只有列出或获取某项内容，或被 Superagent 标记为只读的操作才计为读取。任何删除、移除或撤销的操作都计为破坏性操作。其他所有操作都计为写入操作，即使名称听起来无害也是如此——`triage_finding` 和 `restore_agent_builtin_rule` 是写入操作，因为它们启动了计费工作或更改了策略。

这使得安全设置变得快速：将写入和破坏性操作设置为 **Ask first**（或 **Off**），并保留读取操作为 **Allowed**。参见 [Set action permissions](action-permissions.md)。

## 尝试使用

从一个您可以在 Superagent 中检查的读取操作开始：

```txt
Using Superagent, list our five most recent open security findings with their severity. Do not start any reports or triage.
```

将该列表与 Superagent 中的发现视图进行比较。

> **注意：** 这是示例任务，而非记录的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 密钥被拒绝 | 密钥已被撤销或复制不完整 | 在 **Settings → API keys** 下创建新密钥并重新连接 |
| 积分消耗速度比预期快 | 报告或分诊操作被设置为 **Allowed** | 在 **Permissions** 中将其设置为 **Ask first** |
| 智能体正在等待批准 | 该操作被设置为 **Ask first** | 参见 [Answer a connector review request](review-requests.md) |
| 在繁忙运行时调用失败 | Superagent 的速率限制 | 等待并重试，或分散工作负载 |
| **Needs attention** | 密钥已被撤销 | 创建新密钥并选择 **Reconnect** |

限制：每个连接对应一个 Superagent 组织。没有浏览器登录功能，且 Paperclip 无法缩小已发行密钥的可达范围——请使用操作权限来实现这一点。

若要完全断开连接，请在 Superagent 中撤销密钥，然后在 Paperclip 中移除该连接。

## 相关指南

- [How connector access works](access-model.md)
- [Set action permissions](action-permissions.md)
- [Reauthorize, revoke, or disconnect](reauthorize-and-disconnect.md)
- [Superagent MCP documentation](https://www.superagent.sh/docs/mcp)

## 来源

- [Superagent definition](https://github.com/paperclipai/paperclip/blob/abbd88007f059dbadbd3f266febf264577b6ee78/packages/shared/src/app-definitions/superagent.json) — 方法、凭据字段、密钥指南和警告。
- [Tool connection service](https://github.com/paperclipai/paperclip/blob/abbd88007f059dbadbd3f266febf264577b6ee78/server/src/services/tool-access.ts) — Superagent 风险分类。
