---
paperclip_version: v2026.1023.0
seo_title: Make 自动化连接器
seo_description: 让智能体运行您向 MCP 公开的 Make 场景。通过浏览器登录或具有作用域的 MCP 令牌进行连接，选择区域，并管控更改数据的操作。
---

# Make

智能体可以处理您的 Make 自动化——查找您的组织和团队，并运行您已向 MCP 公开的场景。如果您的团队已经在 Make 中连接了各个应用，这将使智能体能够利用这些现有工作，而无需重新构建。

> **注意：** Make 在每个实例的目录中均可见；无需启用实验性设置。

## 连接前准备

- 一个 Make 账户。
- 对希望智能体使用的场景的访问权限，且这些场景已在 Make 中向 MCP 公开。

如果您计划使用令牌而非浏览器登录，还需要：

- 一个具有 `mcp:use` 作用域的 Make API 令牌，以及所选场景所需的任何其他额外作用域。
- 您公司的区域主机，例如 `eu2.make.com`。Make 会在其 MCP 客户端设置说明中显示该主机。

## 连接 Make

1. 打开 **连接器** 并选择 **Make**。
2. 阅读主按钮上方的访问权限说明行。该行说明了连接所属的身份以及哪些智能体可以使用它。选择 **更改** 以选择不同身份、缩小智能体范围或选择令牌方法。
3. 使用以下方法之一进行连接：
   - **使用 Make 登录** — 选择 **继续登录** 并完成浏览器登录。Paperclip 会自动注册其客户端，因此无需在开发者控制台中进行任何设置。
   - **使用 MCP 令牌** — 输入您的 **Make 区域主机**（例如 `eu2.make.com`），并将令牌粘贴到 **您的 Make 密钥** 中。

使用令牌时，Paperclip 会将服务器地址构建为 `https://<zone>/mcp`。区域必须是一个 `make.com` 主机。令牌作为机密存储，并作为 Bearer 标头发送。切勿将其放入 URL 中。

## 选择访问权限

可达范围取决于 Make 允许已连接身份执行的操作：哪些场景向 MCP 公开，以及在使用令牌时，该令牌携带哪些作用域。仅授予令牌您的场景实际所需的作用域。

场景可以执行其构建时设计的所有功能——发送消息、更新记录、在系统间转移资金。当智能体运行某个场景时，这些效果是真实的。Paperclip 会列出 Make 公开的每个操作，并默认将其状态设为 **允许**，因此请在 **权限** 选项卡中审查该列表：

- 保持读取操作（如组织和团队查找）为 **允许**。
- 将任何运行场景或更改数据的操作设置为 **先询问**，或在需要之前设置为 **关闭**。

参见 [设置操作权限](action-permissions.md)。

## 试用

从读取操作开始：

```txt
Use Make to list the organization and teams this connection can see. Do not run any scenarios.
```

将结果与您在 Make 中看到的内容进行比较。这可以在不触发自动化的情况下确认凭据的有效性。

> **注意：** 此为示例任务，而非记录的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 区域主机被拒绝 | 它不是 `make.com` 主机，或者包含 `https://` 或路径 | 仅输入主机，例如 `eu2.make.com` |
| 令牌被拒绝 | 缺少 `mcp:use`，已被撤销，或属于另一个区域 | 在正确的公司中创建一个具有 `mcp:use` 的令牌并重新连接 |
| 场景未出现 | 该场景未向 MCP 公开，或令牌缺少其所需的作用域 | 在 Make 中公开该场景，或向令牌添加该作用域 |
| 场景意外运行 | 其操作被设置为 **允许** | 在 **权限** 选项卡中将其设置为 **先询问** |
| **需要关注** | 授权或令牌已被撤销 | 选择 **重新连接** |

限制：每个连接仅支持一个 Make 身份。可用场景由 Make 决定，而非由 Paperclip 决定。

## 相关指南

- [Zapier](zapier.md) — 另一种自动化服务，您可以在 Zapier 端选择操作。
- [连接器访问权限的工作原理](access-model.md)
- [设置操作权限](action-permissions.md)
- [Make MCP 服务器文档](https://developers.make.com/mcp-server)

## 来源

- [Make 定义](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/make.json) — 方法、标签、区域验证和令牌位置。
- [目录可见性](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions.ts) — Make 不再从商店中隐藏；默认操作策略。
