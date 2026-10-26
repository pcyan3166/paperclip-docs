---
paperclip_version: v2026.1023.0
seo_title: Connections v3 基础架构
seo_description: Paperclip Connectors 底层重建后的连接存储与授权层：稳定身份、面向主体的授权，以及 mcp_remote 重命名。
---

# Connections v3（Connectors 的底层基础）

**Connections v3** 是用于存储、授权并分发外部服务连接的底层。 [连接器](../connectors.md) 就构建在这一层之上。

> **注意：** 本页现在介绍的不是实验性开关。最初撰写本页时，连接功能通过实验性设置中的 **Apps** 开关启用，目录也只是 Wave 1 的小规模版本。如今这两点都已改变：连接功能始终开启，开关已移除，目录则是 [连接器](../connectors.md) 中的完整列表。本页保留的是底层模型以及面向运维人员的一处重命名——这些内容仍值得查阅。

## 本页首次发布以来的变化

| 当时 | 现在 |
| --- | --- |
| 必须在 **Settings → Experimental** 中开启 **Apps**。 | 始终启用。`enableApps` 是已弃用的兼容性键：*“Apps 始终启用；存储的值和托管值会被忽略。”* Experimental 设置页不再显示此开关。 |
| 导航项名称为 **Apps**。 | 侧边栏项目名称为 **Connectors**。路由仍为 `/apps`。 |
| Wave 1 目录被描述为底层管道，而非应用商店。 | 完整目录，包含设置流程、按操作授权和审核队列。请参阅 [连接器](../connectors.md)。 |
| “设置流程尚不完整。” | 设置流程现已成为产品功能界面。每个服务提供商页面都会说明 Paperclip 对其支持的具体路径。 |

此领域仍有三个真正有效且默认关闭的实验性标记。第一个是 **Chat connectors**（聊天连接器），说明为 *“将智能体连接到 Slack、Discord、Microsoft Teams 和 Telegram 对话。”* 此标记控制[聊天渠道](../connectors.md#chat-channels-experimental)。关闭后会隐藏聊天设置和已连接任务的控件；现有聊天连接仍会继续运行。它不控制应用集成，因此即使隐藏了聊天功能，Slack 和 GitHub 智能体工具仍然不受影响。

第二个是 **GitHub review bots**（GitHub 审查机器人），说明为 *“让智能体使用各自的 App 身份连接到 GitHub，以便审查拉取请求并回复问题。”* 它独立控制 GitHub Code Review Bot，不受 **Chat connectors** 影响。关闭后会隐藏设置和管理界面；现有机器人仍会继续运行，GitHub 工具连接也仍可使用。AgentMail 过去由 **Chat connectors** 控制；现在它是默认连接，每个实例都可使用。

第三个是 **Memory connectors**（记忆连接器），说明为 *“连接 Mem0、Zep、Supermemory、Cognee 和 Honcho，以提供长期记忆和上下文。”* 此标记控制这些[记忆服务提供商](../connectors.md#memory-experimental)的设置界面显示或隐藏。关闭后只会隐藏设置界面；现有连接仍会继续运行。

MCP 聚合器（Zapier、Arcade、Composio 和 Executor）曾短暂拥有自己的开关。现在它们始终可用，而 `enableMcpAggregators` 是已弃用的兼容性键：*“MCP 聚合器始终启用；存储的值和托管值会被忽略。”* 请参阅[通过 MCP 聚合器连接应用](../connectors/mcp-aggregators.md)。

## 底层模型

过去，连接外部服务意味着一组松散的配置和密钥，既没有明确的所有者，也没有稳定的名称。Connections v3 以明确的模型取代了这种做法；如今 Connectors 仍使用此模型：

- **稳定的连接身份。** 每个连接都有一个作用域限于公司的 `uid`（例如 `google-sheets/finance-sheet-1a2b3c4d`），即使名称更改也保持不变，因此 Paperclip 的其他部分可以可靠地引用它。重命名连接不会破坏指向它的策略。
- **明确的所有权、身份验证方式和传输方式。** 每个连接都会记录其所有者、身份验证方式（`oauth`、`api_key` 或 `none`）以及与服务通信的方式。
- **面向主体的授权。** `connection_grants` 表将*连接本身*与*获准使用连接的主体*分开。授权范围可以限定为组织、单个用户或单个智能体，因此一个连接可同时用于共享默认授权和针对特定主体的授权。[连接器访问方式](../connectors/access-model.md)中的身份选择就建立在此机制上。
- **多密钥凭据。** 一个连接可以持有多个凭据引用，以满足现实应用需要多个密钥或权限范围的情况。

生成的 **AppDefinition** 目录为浏览和连接提供统一的类型化契约：每个条目都会声明其方法、传输方式、所有权模式、必填字段和风险级别。授权管理路由对其不认识的权限范围会**默认拒绝**——遇到未知范围时会拒绝，而不是悄悄放行。

## 运维说明：`remote_http` 重命名

Connections v3 将旧版 `remote_http` 传输方式重命名为 `mcp_remote`。远程 MCP 服务器连接使用此传输方式。

升级后无需执行任何操作。这项变更通过两个增量迁移发布，并在启动时自动运行：

- `0182_connections_v3_schema_core` — 为连接补齐 `uid`，添加所有权和身份验证字段，创建 `connection_grants` 表，为每个现有连接写入默认组织授权，并将任何值为 `remote_http` 的 `transport` 改写为 `mcp_remote`。
- `0183_connection_user_authorization_state` — 添加面向主体的授权状态，用于实现按用户授权。

唯一需要跟进的事项是：如果你的工具、脚本或笔记中写有 `remote_http` 传输方式，请将其更新为 `mcp_remote`。目前使用的传输方式包括 `mcp_remote`、`rest_api`、`local_stdio`、`chat_sdk` 和 `runtime_auth`。

## 后续阅读

- [连接器](../connectors.md) — 服务目录、通用指南和各服务提供商页面。
- [连接器访问方式](../connectors/access-model.md) — 授权、智能体访问权限和按操作授权。
- [Tool Gateway](../reference/api/tool-gateway.md) — 调用时强制执行有效策略的组件。
- [实验性功能概览](overview.md) — 了解实验性标记的工作方式，以及“无兼容性保证”的含义。
- [Cloud Sync](cloud-sync.md) — **已退役**，上游及其开关已一并移除。要在实例之间迁移公司，请改用[公司 Import/Export](../how-to/back-up-and-restore-a-company.md)。
