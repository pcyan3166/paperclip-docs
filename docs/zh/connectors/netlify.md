---
paperclip_version: v2026.1023.0
seo_title: Netlify 连接器
seo_description: 让智能体处理 Netlify 团队和站点。区分读取部署状态与启动部署、团队范围、读取测试以及故障排查。
---

# Netlify

智能体可以处理您的 Netlify 团队和站点——检查部署状态、检查配置以及调查构建失败。

## 连接前准备

- 一个具有访问权限的 Netlify 账户，该账户需能访问您希望智能体使用的团队和站点。
- 如果您属于多个 Netlify 团队，请明确您想要使用哪一个。权限范围（Reach）跟随账户，Paperclip 中没有团队选择器。

## 连接 Netlify

1. 打开 **Connectors** 并选择 **Netlify**。
2. 阅读主按钮上方的访问权限说明行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择其他身份、缩小智能体范围或使用其他登录方法，请选择 **Change**。
3. 选择 **Continue to Netlify** 并完成浏览器登录。

Paperclip 会自动向 Netlify 注册其客户端，因此无需在开发者控制台中进行任何设置。

## 选择访问权限

权限范围（Reach）是授权 Netlify 账户的：其所属的团队以及其中的站点，受 Netlify 自身角色权限的限制。

此处关键的区别在于观察部署与触发部署之间：

| 智能体执行的操作 | 后果 |
| --- | --- |
| 读取部署状态、日志和构建历史 | 不会改变任何内容。保持 **Allowed** 是安全的 |
| 读取站点和环境配置 | 可能会暴露环境变量名称，有时甚至包括值 |
| 启动部署 | 取决于目标——见下文 |
| 更改站点配置 | 可能会改变生产站点的构建和行为方式 |

并非每次部署都是生产部署。Netlify 区分生产部署、部署预览和分支部署，只有前者会替换访客看到的内容。请将此视为谨慎对待智能体可以触发*哪种*部署的理由，而不是假设每次部署都是安全的或每次部署都是上线的理由——在允许之前，请检查特定操作的目标。

> **警告：** 生产部署会将代码置于真实用户面前，而配置更改可能会破坏站点。请将部署和配置操作保持在 **Ask first** 或 **Off** 状态。读取构建日志以诊断故障是该连接器的常见且安全的用途；触发重建则是应由人员批准的部分。

Netlify 环境变量通常包含 API 密钥。能够读取站点配置的智能体可能会暴露这些密钥，因此对于具有配置访问权限的连接，建议优先使用 **Just agents I pick**。请参阅 [Set action permissions](action-permissions.md)。

## 试用

```txt
List the Netlify sites this account can see, and tell me the status and time of the most recent deploy for one of them. Do not start a deploy.
```

与 Netlify 仪表板进行对比。读取部署状态是自然的初步检查，因为它也是该连接器最常见的实际用途。

> **注意：** 这是示例任务，而非记录的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 缺少某个站点 | 授权账户不是该团队的成员 | 在 Netlify 中将其添加到团队；无需重新连接 |
| 操作被拒绝 | 账户的 Netlify 角色不允许该操作 | 在 Netlify 中调整角色，或关闭该功能 |
| 意外启动了部署 | 部署操作被设置为 **Allowed** | 将其设置为 **Ask first**。恢复工作由您在 Netlify 中执行——取消正在运行的构建，或发布之前的部署。Paperclip 不会回滚任何内容 |
| 构建日志被截断 | Netlify 自身的日志保留策略和大小限制 | 在 Netlify 仪表板中查看完整日志 |
| 缺少预期的功能 | Netlify 服务器未暴露该功能 | 使用 **Refresh actions**；否则该功能不可用 |
| **Needs attention** | 授权已过期或被撤销 | 选择 **Reconnect** |

限制：每个连接对应一个 Netlify 账户。Paperclip 内部没有团队或站点过滤器。Paperclip 无法回滚部署——请在 Netlify 中执行此操作。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/netlify/)

- [Cloudflare](cloudflare.md) — 另一个托管和基础设施连接器。
- [GitHub](github.md) — 部署工作流的代码仓库端。
- [Set action permissions](action-permissions.md)
- [Netlify agent setup guides](https://docs.netlify.com/build/build-with-ai/agent-setup-guides/agent-setup-overview/)
