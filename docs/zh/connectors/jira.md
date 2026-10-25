---
paperclip_version: v2026.1023.0
seo_title: Jira 连接器
seo_description: 让智能体通过 Atlassian 托管服务器读取和更新 Jira 问题。站点选择、管理员审批，以及连接为何可能找不到任何内容的原因。
---

# Jira

智能体可以通过 Atlassian 的托管服务器处理 Jira 问题——包括搜索、读取、评论和更新。

同一个 Atlassian 服务器也涵盖 Confluence，因此在此处建立的连接可能同时暴露 Confluence 工具和 Jira 工具。请阅读连接的操作列表，以查看实际获取了哪些工具。

## 连接前准备

- 一个处于活动状态的 Jira Cloud 站点，以及一个有权访问您希望智能体使用的项目的 Atlassian 账户。
- 您的 Atlassian 租户可能要求管理员在任何人连接之前批准该客户端。如果授权被拒绝或处于待定状态，通常就是原因所在。
- Jira Data Center 和 Server 部署不是此托管服务器所针对的对象。这是 Atlassian Cloud 远程 MCP 服务器。

## 连接 Jira

1. 打开 **连接器** 并选择 **Jira**。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他登录方法，请选择 **更改**。
3. 选择 **继续前往 Jira** 并在浏览器中完成 Atlassian 的授权。
4. 如果您的 Atlassian 账户拥有多个站点，请在 Atlassian 的流程中选择您想要的站点。这是需要做出正确决定的环节——该决定是在 Atlassian 端做出的，而非在 Paperclip 中。

Paperclip 会自动向 Atlassian 注册其客户端，因此无需在开发者控制台中进行任何配置。

## 选择访问权限

项目和问题的访问范围由 Atlassian 决定：连接可以看到授权账户在所选站点上能看到的内容，但受 Jira 项目权限和问题级别安全设置的限制。

> **注意：** 成功的授权并不意味着智能体可以看到您的问题。如果选择了错误的站点，或者账户缺乏对某个项目的浏览权限，搜索将返回空结果，而连接状态却显示正常。请先检查站点选择和项目权限，然后检查失败或空的工具响应。

问题创建、状态转换和评论属于写入操作，且对您的整个团队可见。在您信任该工作流之前，请将其保持在 **先询问** 状态。参见 [设置操作权限](action-permissions.md)。

归属权遵循授权账户，因此除非使用专用账户，否则智能体的评论将显示在该人名下。参见 [为人员和智能体使用独立账户](separate-accounts.md)。

## 试用

```txt
Find Jira issue PROJ-123 and tell me its status, assignee, and most recent comment. Do not change it.
```

将其与 Jira 中的问题进行对比。如果返回结果为空，请先检查站点。

> **注意：** 这是一个示例任务，而非记录在案的测试结果。请替换为您自己可以打开的问题键。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 授权被拒绝或保持待定状态 | Atlassian 租户要求管理员批准该客户端 | 请 Atlassian 管理员批准 |
| 连接正常但找不到任何问题 | 在授权期间选择了错误的站点 | 重新连接并选择正确的站点 |
| 某些项目不可见 | 通常是因为授权账户缺乏浏览权限；问题级别安全设置和项目角色配置也可能隐藏问题 | 以该账户身份在 Jira 中打开其中一个缺失的问题。如果您也看不到它，则是 Jira 权限问题；请在那里修复，无需重新连接 |
| 意外出现 Confluence 工具 | Atlassian 服务器涵盖这两个产品 | 在 **权限** 选项卡中关闭您不需要的工具 |
| 状态转换被拒绝 | Jira 工作流不允许该账户进行此转换 | 在 Jira 中检查工作流 |
| **需要关注** | Atlassian 授权已过期或被撤销 | 选择 **重新连接** |

限制：本指南涵盖 Atlassian Cloud。Paperclip 不添加项目选择器；访问范围遵循已授权账户和 Atlassian 的权限。在分配工作之前，请检查已连接账户实际可以访问哪些站点和项目。在一个站点上的成功读取并不能验证对另一个站点的访问权限。

## 相关指南

- [连接器概述](https://paperclip.ing/product/connectors/jira/)

- [Linear](linear.md)、[Asana](asana.md)、[Todoist](todoist.md) — 其他工作跟踪连接器。
- [为人员和智能体使用独立账户](separate-accounts.md)
- [设置操作权限](action-permissions.md)
- [Atlassian 远程 MCP 服务器文档](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/)
