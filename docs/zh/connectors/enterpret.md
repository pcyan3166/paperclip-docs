---
paperclip_version: v2026.1023.0
seo_title: Enterpret 连接器
seo_description: 让智能体向 Enterpret 询问客户反馈并获取带引用的引文。通过组织身份验证令牌或登录进行连接，并将图查询保持在“先询问”状态。
---

# Enterpret

您可以让智能体在 Enterpret 中询问有关客户反馈的问题——包括主题、情感以及带有指向原始记录引用的逐字引文。当任务需要客户本人的原话，而非对客户想法的猜测时，这非常有用。

Enterpret 出现在每个实例的目录中；无需启用实验性设置。

该连接使用 Enterpret 官方的只读 MCP 服务器。Enterpret 运行一个独立的 Agent MCP 服务，该服务可以执行更改；此连接器不会访问该服务。

## 连接前准备

选择以下两种接入方式之一：

- **组织身份验证令牌**（推荐）。在 Enterpret 的 **Settings → Enterpret MCP** 下生成。一个令牌属于一个 Enterpret 组织，连接由您选择的人员和智能体共享。
- **您自己的 Enterpret 账户**，用于浏览器登录。每个人单独连接，Enterpret 将他们的查询单独归因于他们。该账户需要访问您组织的反馈。

客户反馈通常包含姓名、账户和其他个人细节。请预先决定哪些智能体应查看这些内容。

## 连接 Enterpret

1. 打开 **Connectors** 并选择 **Enterpret**。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。选择 **Change** 以缩小智能体范围或切换方法。
3. 选择连接方式：
   - **使用身份验证令牌** — 将令牌粘贴到 **Your Enterpret key** 字段中。这是默认方法。
   - **使用 Enterpret 登录** — 完成浏览器登录。Paperclip 会自动注册其客户端，因此无需在开发者控制台中进行设置。此方法仅连接您自己的账户。
4. 完成设置并打开 **Permissions** 以检查操作列表。

Paperclip 将令牌作为机密存储，并在每次调用时将其发送给 Enterpret。智能体永远看不到它。

## 选择访问权限

几乎每个 Enterpret 操作都是读取操作，这些操作默认状态为 **Allowed**（允许）。有一个例外：

- **`run_graph_query` 默认为 Ask first**（先询问）。它运行由智能体编写的图查询。Enterpret 将其标记为只读，但 Paperclip 尚未确认这一点，因此将该操作视为写入操作，并在每次调用前询问人员。除非您愿意批量批准开放式查询，否则请保持此状态。

还有两件事值得了解：

- **新操作会被保留以供审查。** 如果 Enterpret 稍后添加操作，**Refresh actions**（刷新操作）会将它们展示给您，而不是直接启用它们。参见 [全新操作会发生什么](action-permissions.md#what-happens-to-a-brand-new-action)。
- **引文会出现在任务记录中。** 答案可能包含带有说话者归属的逐字客户文本。对于令牌连接，建议优先选择 **Just agents I pick**（仅我选择的智能体），并保持智能体列表简短。

## 试用

从一个您可以自己在 Enterpret 中核对的问题开始：

```txt
Using Enterpret, tell me which organization this connection belongs to, then list the top three feedback themes from the last 30 days with one cited quote each.
```

在 Enterpret 中打开被引用的记录并确认引文是否匹配。

> **注意：** 这是示例任务，而非记录的测试结果。

## 令牌过期与登出

身份验证令牌有有效期。在 Enterpret 的 **Settings → Enterpret MCP** 下检查有效期，并在其失效前替换令牌：创建新令牌，然后使用它重新连接。参见 [重新授权、撤销或断开连接](reauthorize-and-disconnect.md)。

对于浏览器登录，在 Enterpret 中撤销访问权限可能不会立即生效——Enterpret 可能会在长达 24 小时内继续将该令牌视为有效。若要立即停止 Paperclip 的访问，请在 Paperclip 中断开连接。

## 故障排除与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 令牌被拒绝 | 已过期、已撤销或复制不完整 | 在 **Settings → Enterpret MCP** 下生成新令牌并重新连接 |
| 答案来自错误的组织 | 令牌属于不同的 Enterpret 组织 | 使用来自正确组织的令牌重新连接 |
| 智能体正在等待 `run_graph_query` | 默认设置为 **Ask first**（先询问） | 批准或拒绝请求——参见 [回答连接器审查请求](review-requests.md) |
| 登录显示的权限范围比预期多 | Enterpret 报告的权限范围比 Paperclip 请求的更广，包括 `mcp:write` 和 `email` | Paperclip 仅请求 `mcp:read`；Enterpret 正在更正报告。这不会为此连接器授予写入操作 |
| 新操作未显示为允许 | Enterpret 稍后添加的操作会被保留以供审查 | 选择 **Refresh actions**（刷新操作）并审查新条目 |
| **Needs attention**（需要关注） | 令牌已过期或登录已被撤销 | 选择 **Reconnect**（重新连接） |

限制：每个令牌连接对应一个 Enterpret 组织。浏览器登录仅限个人使用；不能作为组织连接共享。

## 相关指南

- [PostHog](posthog.md) 和 [Mixpanel](mixpanel.md) — 产品分析连接器。
- [连接器访问权限如何工作](access-model.md)
- [设置操作权限](action-permissions.md)
- [验证连接器并修复损坏的连接器](verify-and-troubleshoot.md)
- [Enterpret MCP 服务器文档](https://enterpret.support.site/article/enterpret-mcp-server)

## 来源

- [Enterpret 定义](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/enterpret.json) — 方法、标签、凭据字段、权限范围和警告。
- [应用默认设置](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions.ts) 和 [工具连接服务](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/server/src/services/tool-access.ts) — `run_graph_query` 的“先询问”默认设置以及保留的新操作。
