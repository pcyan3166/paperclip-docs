---
paperclip_version: v2026.1023.0
seo_title: 将 Assistant 连接到 Paperclip
seo_description: 将 Codex、Claude 或其他 Assistant 连接到你的组织。查看授权、委派任务、获取结果，并在需要时撤销访问权限。
---

# Assistant 连接（MCP）

你可以使用日常使用的 Assistant 查看 Paperclip 组织、委派任务，并在另一段对话中回来查看结果。此连接以你的人类用户账户身份运行，只能访问你批准的一个组织。

此功能为实验性功能。在自托管实例和 Cloud 实例上，**Assistant connections (MCP)**（Assistant 连接，`enablePublicMcp`）默认关闭。

## 启用此功能

1. 以实例管理员身份打开 **Settings → Experimental**（设置 → 实验性功能）。
2. 开启 **Assistant connections (MCP)**（Assistant 连接，MCP）。
3. 返回 **Connectors → Assistant Connection (MCP)**（连接器 → Assistant 连接，MCP）。

你的实例必须启用已认证的用户账户，并配置 Assistant 能访问的规范 URL。公开连接必须使用 HTTPS；本地开发可使用回环 HTTP。自托管的私有端点必须能由你的客户端直接访问。运维设置请参阅 [Assistant MCP 参考](../reference/api/assistant-mcp.md#instance-requirements)。

## 邀请 Assistant

1. 选择你要连接的组织。
2. 打开 **Connectors → Assistant Connection (MCP) → Set up**（连接器 → Assistant 连接，MCP → 设置）；连接 Assistant 后，此处会显示 **Manage**（管理）。
3. 选择 **Copy invitation**（复制邀请），然后把消息粘贴给你的 Assistant。
4. 打开 Assistant 提供的批准链接。登录 Paperclip，检查客户端和组织，然后选择 **Connect organization**（连接组织）。
5. 要求 Assistant 调用 `paperclip_connection`，并确认你的账户、组织和权限。

邀请中包含指向公开设置说明的链接。邀请本身不包含凭据，也不会授予任何访问权限。有些 Assistant 需要你在其设置中添加连接器；仅在聊天中阅读邀请并不会安装连接器。可能需要新建对话或重启客户端才能加载工具。

### 手动设置

打开 **Set up manually**（手动设置），查看实例生成的说明。若要直接连接 Codex，请将下面的主机替换为你的实例主机名：

```sh
codex mcp add paperclip --url https://YOUR-PAPERCLIP-HOST/mcp/paperclip
codex mcp login paperclip --scopes paperclip:read,paperclip:write,paperclip:configure,offline_access
```

对于 Claude Code：

```sh
claude mcp add --transport http paperclip https://YOUR-PAPERCLIP-HOST/mcp/paperclip
```

在 Claude Code 中打开 `/mcp`，并对 Paperclip 服务器进行身份验证。OpenCode、浏览器 Assistant 和其他客户端各自在设置页面中有对应标签页。保留客户端已有设置；如果 `paperclip` 已指向其他实例，请另选一个名称，并始终使用该名称。

在你批准连接期间，请保持授权命令运行。如果授权过期或命令停止，请先发起新的请求，再使用其他批准链接。对于没有浏览器回调监听器的客户端，请使用[设备登录和 stdio 桥接](../reference/cli/mcp.md)。设备登录会将你带到实例上的 `/mcp-device`：检查页面上显示的代码是否与你的 Assistant 中显示的代码一致（或者在 **Enter the code shown by your assistant**（输入 Assistant 显示的代码）下方输入该代码），然后照常批准。

## 选择访问权限

授权页面会显示客户端和组织。除了名称，也要检查用于识别身份的域名；仅凭客户端名称不能证明其身份。

| 权限 | 在直接连接实例时允许执行的操作 |
| --- | --- |
| `paperclip:read` | 读取允许访问的工作内容和非机密配置。所有连接都必须拥有此权限。 |
| `paperclip:write` | 创建和编辑任务、添加评论、写入文档，以及管理任务交付物和附件。 |
| `paperclip:configure` | 编辑允许访问的智能体、项目、指令和技能设置。 |
| `offline_access` | 使用会轮换的刷新令牌保持访问权限，无需每个会话都重新登录。 |

**Write all of your Paperclip data**（写入你的所有 Paperclip 数据）复选框会同时批准所请求的写入权限和配置权限。若请求中包含这些权限且你的角色允许，此复选框默认选中。取消选择即可授予只读访问权限。你当前的组织权限仍适用于每次调用；若要扩大权限范围，必须重新进行授权。

## 查看、委派并跟进

先让 Assistant 确认已连接的组织，并列出智能体和未完成的任务。这些读取操作可以验证访问是否正常，而不会启动任何工作。

准备好后，让它为可用的智能体创建一项任务，并保存返回的任务链接。之后可以在另一段对话中让它查找该任务、读取进度，并获取已保存的报告或附件。反馈评论可以唤醒已分配的智能体，或将更多工作加入队列。

智能体执行工作时使用 Paperclip 中配置的凭据、环境、预算和审批流程。你的 Assistant 订阅不会提供执行能力。Assistant 可以列出待处理的审批，并链接到 Paperclip，由你在那里作出决定。

直接连接会开放工作、配置、文件传输和 API 发现工具。支持 Events 的客户端可以订阅任务状态、评论和文档更新。监控需要显式订阅；仅建立连接不会自动开始监控。请参阅[协议和工具参考](../reference/api/assistant-mcp.md)。

## 直接连接和托管分发

实例直接端点会开放 37 个工具。托管目录代理提供另一套规模较小、面向工作的工具目录；不要认为通过该代理也能使用实例直接连接中的配置、文件传输或 Events 功能。

即使 Assistant 商店中没有上架，也可以配置客户端直连。托管代理的部署、浏览器客户端兼容性，以及公开商店或目录中的上架，分别有各自的发布要求。本指南并不表示你的 Assistant 账户一定能使用某个上架条目。

## 撤销或禁用访问权限

返回 **Assistant Connection (MCP)**，即可查看并撤销所选组织中的连接。你也可以通过 `/assistant-connections` 管理整个账户的连接。

每个连接都会列在个人头像下方，名称由你的名字和 Assistant 客户端组成，例如 *Alex’s Codex connection*；其后会注明访问权限（例如 *Connected as you · Read and write*（以你的身份连接 · 读取和写入）；如果你批准了配置权限，还会附加 *Configure agents, projects and skills*（配置智能体、项目和技能））。智能体连接（例如配对的 [Dot](external-agents.md)）需要在智能体自身进行管理，不会显示在此处。这里只会列出你自己的有效连接。选择 **Revoke**（撤销）后，该连接会立即从列表中消失；Paperclip 仍会保留审计记录。

撤销会阻止今后的调用。你已经委派的工作会继续执行，正在进行的更改也可能已经完成。如果需要停止，请在 Paperclip 中管理相应任务或运行。

关闭实验性设置会立即阻止工具发现、新登录、工具调用和事件推送。连接管理仍可使用，已委派的工作会继续执行。重新开启后，尚未过期的连接即可恢复。

## 相关指南

- [Assistant MCP API 参考](../reference/api/assistant-mcp.md)
- [MCP CLI 命令](../reference/cli/mcp.md)
- [自定义 MCP 服务器](../connectors/custom-mcp-servers.md) — 让 Paperclip 智能体使用外部工具。
- [实验性功能概览](overview.md)
