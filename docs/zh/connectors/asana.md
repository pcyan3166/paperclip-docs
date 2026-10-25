---
paperclip_version: v2026.1012.0
seo_title: Asana 连接器
seo_description: 允许智能体读取和更新 Asana 任务。通过 Paperclip 的 Asana 应用或您自己的应用登录，设置工作区和项目访问权限，进行读取测试，以及故障排除。
---

# Asana

智能体可以处理 Asana 任务——查找任务、读取详细信息，以及更新或创建任务。

## 连接前准备

- 一个 Asana 账户，该账户需拥有您希望智能体使用的工作区和项目的访问权限。
- 某些 Asana 组织会限制谁可以授权或创建应用。如果您不是管理员，请在开始之前而非之后进行检查。

有两种连接方式：

| 方法 | 所需条件 | 提供时机 |
| --- | --- | --- |
| **使用 Asana 登录** | 无需注册。您通过 Paperclip 自己的 Asana 应用进行登录 | 当您的实例已注册 Paperclip Cloud 且 Cloud 提供 Asana 配置文件时 |
| **使用您自己的 Asana OAuth 应用** | 一个由您自行注册的 Asana MCP 应用 | 始终可用，包括未注册的实例 |

## 使用 Paperclip 的 Asana 应用连接

1. 打开 **Connectors**（连接器）并选择 **Asana**。
2. 阅读主按钮上方的访问权限说明行。此方法以您本人或专用智能体账户的身份连接，绝不会以共享组织身份连接。若要更改此设置或缩小智能体范围，请选择 **Change**（更改）。
3. 选择 **Continue to sign in**（继续登录）。Paperclip 将显示 *“Sign in with Asana to choose your workspace and connect it to Paperclip.”*（使用 Asana 登录以选择您的工作区并将其连接到 Paperclip。）
4. 在 Asana 中，选择工作区并批准访问权限。此后，Paperclip 将保持该连接的登录状态。

## 使用您自己的 Asana OAuth 应用连接

回调 URI 来自 Paperclip，而客户端凭据来自 Asana，因此两个控制台需要交替操作。请先打开 Paperclip 的设置以读取 URI，然后注册应用，最后再返回。

### 1. 读取 Paperclip 的回调 URI

1. 打开 **Connectors**（连接器）并选择 **Asana**。
2. 阅读主按钮上方的访问权限说明行，如果您希望使用不同的身份或减少智能体数量，请选择 **Change**（更改）。
3. 选择 **Use your own Asana OAuth app**（使用您自己的 Asana OAuth 应用）。在未注册 Paperclip Cloud 的实例上，这是唯一的方法，且屏幕将直接打开 **Your OAuth app**（您的 OAuth 应用）页面。Paperclip 会显示确切的回调 URI——请复制它。它是您实例源地址上的 `/api/tools/oauth/callback` 路由。请保持此屏幕打开。

> **注意：** 对于本地开发，即使您在 `127.0.0.1` 处打开了 Paperclip，Paperclip 在回调中也会使用 `localhost`。公共部署应使用其规范的 HTTPS 地址。

### 2. 创建 Asana MCP 应用

在 Asana 的开发者控制台 [app.asana.com/0/my-apps](https://app.asana.com/0/my-apps) 中：

1. 选择 **Create new app**（创建新应用）并输入名称。
2. **将应用类型设置为 “MCP app”（MCP 应用）。** 这是容易出错的一步——标准 API 应用或个人访问令牌在此处无法使用，因为 Asana 将 MCP 令牌单独管理。
3. 选择 **Create app**（创建应用）。
4. 在侧边栏的 **OAuth** 部分，将步骤 1 中的回调 URI 添加为 **Redirect URL**（重定向 URL）。
5. 在 **Manage distribution**（管理分发）下，选择该应用可使用的特定工作区，或允许任何工作区，然后保存。
6. 复制 **client ID**（客户端 ID）和 **client secret**（客户端密钥）。

> **注意：** Asana MCP 应用没有可选的作用域。Asana 授予一个固定的 `default` 作用域——因此，如果您在此处寻找权限清单，是没有的。访问权限来自授权账户，而 Paperclip 的操作权限决定智能体可以使用哪些工具。

### 3. 在 Paperclip 中完成设置

返回设置屏幕，提供客户端 ID 和密钥，然后以您希望连接拥有其访问权限的账户身份在 Asana 中进行授权。Paperclip 会加密存储密钥。如果您使用相同的客户端 ID 返回未完成的设置，无需再次粘贴密钥；更改客户端 ID 则需要提供其匹配的密钥。

Asana 自己的参考文档是 [Integrating with Asana's MCP server](https://developers.asana.com/docs/integrating-with-asanas-mcp-server)（集成 Asana 的 MCP 服务器）。

## 选择访问权限

访问权限取决于授权 Asana 账户：即其所属的工作区和其可以打开的项目。该账户未加入的私有项目将保持不可见，这通常是将某些内容排除在访问范围之外的最简单方式。

Paperclip 中没有工作区或项目选择器。若要缩小访问权限，请使用加入较少项目的账户进行授权。

任务创建、更新和评论是您的团队将看到的写入操作。在工作流得到验证之前，请将其保持在 **Ask first**（先询问）状态。读取操作可以保持 **Allowed**（允许）状态。参见 [Set action permissions](action-permissions.md)（设置操作权限）。

归属关系遵循授权账户，因此使用专用账户可以使智能体活动具有可区分性。参见 [Use separate accounts for people and agents](separate-accounts.md)（为人员和智能体使用单独账户）。

## 试用

```txt
Find the Asana task called "Update onboarding checklist" and tell me its assignee, due date, and current status. Do not change it.
```

与 Asana 中的任务进行对比。查找一个您已经可见的任务可以确认凭据和智能体的权限，而不会影响您团队的看板。

> **注意：** 这是示例任务，而非记录的测试结果。请替换为您自己工作区中的任务名称。

## 故障排除与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 设置要求提供客户端 ID 和密钥 | 实例未注册 Paperclip Cloud，或您选择了自己的应用 | 注册 Asana MCP 应用，或注册实例以使用 **Sign in with Asana**（使用 Asana 登录） |
| 使用 API 应用授权失败 | Asana 应用不是 MCP 应用 | 创建一个新的应用，并将类型设置为 MCP 应用 |
| 授权因重定向或 URI 错误而失败 | Asana 应用上的回调 URI 与 Paperclip 显示的不匹配 | 精确复制 URI 并重试 |
| 无法创建应用 | Asana 组织限制应用创建 | 联系 Asana 管理员 |
| 某个项目的任务不可见 | 授权账户不是该项目的成员 | 在 Asana 中将其添加到该项目；无需重新连接 |
| 更新被拒绝 | Asana 自身在该项目上的字段规则或权限 | 在 Asana 中检查任务和项目设置 |
| **需要关注** | 授权已在 Asana 中撤销 | 选择 **Reconnect**（重新连接） |

限制：每个连接仅限一个 Asana 账户。Paperclip 内部没有工作区或项目限制。Asana 的速率限制适用。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/asana/)（连接器概述）

- [Jira](jira.md)、[Linear](linear.md)、[Todoist](todoist.md) — 其他工作跟踪连接器。
- [Use separate accounts for people and agents](separate-accounts.md)（为人员和智能体使用单独账户）
- [Set action permissions](action-permissions.md)（设置操作权限）
- [Asana MCP server documentation](https://developers.asana.com/docs/integrating-with-asanas-mcp-server)（Asana MCP 服务器文档）
