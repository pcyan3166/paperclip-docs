---
paperclip_version: v2026.1023.0
seo_title: Paperclip 连接器
seo_description: Paperclip 可连接的所有服务，按用途分组，并提供访问模型、操作权限及各自的设置指南。
---

# 连接器

**连接器（connector）**是保存下来的外部服务连接，例如 Gmail 邮箱、GitHub 组织、Notion 工作区或 PostHog 项目。你只需设置一次，决定连接供谁使用、用于什么，Paperclip 就会在每次运行时沿用这些设置。

> **注意：**本指南介绍 Paperclip **v2026.1023.0** 中的连接器模型。可用方法取决于已安装的版本、实例配置和提供商审批情况。目录中列出某项服务，并不代表该服务一定支持提供商登录。

> **警告：****Google 验证尚未完成。**Paperclip 尚未完成 Google 应用验证。授权期间可能会看到未验证应用警告。如果 Google 提供 **Advanced**（高级）选项，让你继续前往 Paperclip，你可以先检查请求的访问范围，再决定是否继续。并非所有账户都会显示此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：**Google 验证完成前，九个 Google Workspace 连接器以及已保存的 Google 账户会暂时从 **Connectors** 页面隐藏。只有列表会改变：现有 Google 连接仍按原有访问范围和权限运行；智能体需要 Google 服务时，仍可在任务中通过连接卡向你请求连接。

## 从这里开始

- [连接首个连接器](connectors/first-connector.md)——从头到尾演示一次设置，并以智能体实际执行的一项只读任务结束。
- [连接器访问方式](connectors/access-model.md)——了解身份、授权、智能体选择，以及“有效策略”的含义。

## 可以连接什么

**Connectors** 列表包含三类连接。它们看起来相似——每一种都是 Paperclip 为你保存的凭据——但设置方式不同，使用者也不同。

**应用集成**让智能体能够处理你已在使用的应用中的信息和操作：读取 Notion 页面、打开 Linear 问题、查看 Sentry 错误、搜索 Drive 文件夹。设置完成后会显示可单独启用或关闭的操作列表，由智能体发起工作。

**模型提供商**提供 Paperclip 运行模型所需的凭据。设置完成后会保存密钥或订阅；这里没有操作列表，因为它不是智能体可以调用的工具。每当智能体进行推理时，Paperclip 都会代它使用该凭据。如果你曾疑惑为什么 Anthropic 连接没有可配置的权限，原因就在这里。

**聊天渠道——实验性**让用户可以通过消息应用与智能体交谈。管理员必须启用默认关闭的 **Chat connectors**。设置时需要将提供商应用或机器人连接到 Paperclip；符合条件的消息可以启动或继续智能体任务。

同一服务可能属于不止一种类型。Slack 同时提供应用集成和实验性聊天渠道，二者分别配置、使用不同凭据。Google Chat 属于应用集成：智能体可以读取和发送消息，但用户不能通过它与智能体交谈。GitHub 有两个条目：Developer tools 下的 **GitHub** 提供仓库工具；Chat channels 下的 **GitHub Code Review Bot** 是实验性的审查机器人。审查机器人有单独的实验设置 **GitHub review bots**，与 **Chat connectors** 相互独立。

## 连接方式

目录中的每一行都会列出该连接器支持的方式。以下名称在各处含义相同：

| 方式 | 含义 |
| --- | --- |
| Connect with Paperclip | OAuth 客户端由 Paperclip 持有；你在提供商自己的授权页面批准访问。仅适用于已加入 Paperclip Cloud 的实例。 |
| Sign in with the provider | 连接时由 Paperclip 向提供商的授权服务器注册自身，然后你在那里批准访问。无需预先创建任何内容。 |
| Your own OAuth app | 连接前，你先向提供商注册 OAuth 客户端，并添加 Paperclip 的回调 URI。提供商不支持自动注册，或你希望客户端归自己所有时，可使用此方式。 |
| API key | 在提供商控制台生成密钥或令牌，然后粘贴到 Paperclip。 |
| Provider app registration | 在提供商开发者控制台创建应用或机器人，授予文档要求的权限和事件，并连接其凭据。提供商专属的聊天渠道指南会说明托管收件箱等例外情况。 |
| No credential | 无需登录。你只需指定目标，例如商店域名、生成的网址或共享文件，设置即告完成。 |

实际显示哪些方式取决于你的实例。除非实例已加入 Paperclip Cloud，且 Cloud 为该连接器提供配置，否则 **Connect with Paperclip** 不会显示。所有聊天渠道都受实例设置 **Chat connectors** 控制，默认关闭；**GitHub Code Review Bot** 例外，它由单独且默认关闭的 **GitHub review bots** 设置控制。AgentMail 不受此限制：所有实例都可以使用其电子邮件收件箱。[记忆提供商](#memory-experimental)受单独的 **Memory connectors** 设置控制，该设置也默认关闭。各连接器页面会列出自身的前提条件。

### 设置界面

连接目录中的应用只需经过一个界面。Paperclip 会为你选择推荐方式，只询问用于证明身份所需的信息：提供商登录、密钥或地址。目录卡片显示 **Add key** 时表示需要粘贴密钥；其他卡片均显示 **Connect**。

主按钮上方会用一行文字说明 Paperclip 将采用的访问范围，例如 *“为组织中的所有人建立连接，所有智能体均可使用。”* 如果连接是智能体在任务中提出的，这里会改为 *“仅供提出请求的智能体使用。”* 选择 **Change** 可以更换身份、缩小可用智能体范围，或选择其他方式或能力；这些设置之后也可以修改。连接成功后，可在 **Permissions** 标签页中设置单项操作，参见[设置操作权限](connectors/action-permissions.md)。

## 连接器目录

找到应用并打开对应的设置指南。

### Google Workspace

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [Gmail](connectors/gmail.md) | Google 电子邮件服务。智能体可以搜索和阅读邮件；使用草稿连接时，还可创建草稿供你在 Gmail 中审阅。发送、删除和更改标签的操作始终不会启用。 | Connect with Paperclip · Your own OAuth app |
| [Google Calendar](connectors/google-calendar.md) | Google 日历。智能体可以读取日历；使用写入连接时，可以创建和修改日程。 | Connect with Paperclip · Your own OAuth app |
| [Google Chat](connectors/google-chat.md) | Google Workspace 团队消息服务。智能体可以搜索和阅读对话；使用写入连接时，可以发送消息。这是供智能体调用的工具，不是用户与智能体交谈的渠道。 | Connect with Paperclip · Your own OAuth app |
| [Google Docs](connectors/google-docs.md) | Google 文档处理服务。智能体可以读取文档；使用写入连接时，可以编辑文档。 | Connect with Paperclip · Your own OAuth app |
| [Google Drive](connectors/google-drive.md) | Google 文件存储服务。智能体可以搜索和读取文件；使用写入连接时，可以创建和复制文件。 | Connect with Paperclip · Your own OAuth app |
| [Google People](connectors/google-people.md) | Google 账户中的联系人和目录。智能体可以查找人员及个人资料。此连接只读，不提供写入连接。 | Connect with Paperclip · Your own OAuth app |
| [Google Sheets](connectors/google-sheets.md) | Google 电子表格。智能体可以读取表格；使用写入连接时，可以更新表格。还可以通过第三种方式将单个电子表格共享给 Paperclip 机器人账户，而无需连接 Google 身份。 | Connect with Paperclip · Your own OAuth app · No credential |
| [Google Slides](connectors/google-slides.md) | Google 演示文稿。智能体可以读取演示文稿；使用写入连接时，可以编辑演示文稿。 | Connect with Paperclip · Your own OAuth app |
| [Google Workspace 搜索](connectors/google-workspace-search.md) | 在单个 Google 账户中跨 Gmail、Drive、Calendar 和 Chat 进行只读搜索。当智能体不知道信息位于哪个应用中时，可用它查找内容。 | Connect with Paperclip · Your own OAuth app |

> **注意：**Google 应用验证尚未完成。关于未验证应用警告、账户限制以及单独的 Developer Preview 前提条件，请参阅 [Google 设置要求](connectors/google-setup.md)。

### 生产力与协作

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [Arcade](connectors/arcade.md) | 工具网关：将你从多个服务器中选出的工具组合到一个 URL 后。智能体使用公开的工具；身份验证遵循你的网关设置。该设置尚未验证。 | 网关 URL · 浏览器登录或令牌/请求头 |
| [Asana](connectors/asana.md) | 用于团队项目、任务和目标的工作管理服务。智能体可访问授权 Asana 账户能够访问的项目和任务。 | Connect with Paperclip · Your own OAuth app |
| [Calendly](connectors/calendly.md) | 日程预约链接和预订。智能体可以查看活动类型和可用时段，并在你允许的情况下使用 Calendly 的预约操作。登录授权会同时授予读取和写入权限，因此应将预订操作设为 **Ask first**。 | Sign in with Calendly |
| [Composio](connectors/composio.md) | 通过 Composio Connect 或已配置的会话发现并执行应用操作。上游应用授权在 Composio 中完成；范围较广的执行操作需要单独审查。该设置尚未验证。 | Composio Connect 登录 · 已配置的会话 URL 和请求头 |
| [Executor](connectors/executor.md) | 为已配置的集成提供单一远程端点。智能体在 Executor 的上游策略范围内使用其公开操作。该设置尚未验证。 | 部署 URL · 浏览器登录或令牌/请求头 |
| [Fireflies](connectors/fireflies.md) — **线上例外；尚不稳定** | 访问连接账户可读取的会议转录、摘要和行动项。该设置尚未验证。 | Sign in with Fireflies · API key |
| [Jira](connectors/jira.md) | Atlassian 面向软件团队的问题跟踪工具。智能体可处理你所连接 Jira 站点中的问题。 | Sign in with Jira |
| [Linear](connectors/linear.md) | 面向产品和工程团队的问题跟踪工具。智能体可以创建、更新并读取授权 Linear 账户所能访问的问题。只读 API 密钥会建立只读连接。 | Sign in with Linear · API key |
| [Make](connectors/make.md) | 通过 Make 向 MCP 公开的自动化场景。智能体可以查找组织和团队并运行场景；请将场景运行操作设为 **Ask first**。 | Sign in with Make · MCP token |
| [Miro](connectors/miro.md) | 用于绘图、规划和研讨会的共享在线白板。智能体可使用 Miro 账户能够访问的白板。 | Sign in with Miro |
| [Notion](connectors/notion.md) | 用于笔记、文档和数据库的工作区。智能体可以读取并更新授权 Notion 账户可访问的内容。Paperclip 不会额外添加页面级筛选。 | Sign in with Notion |
| [Todoist](connectors/todoist.md) | 面向个人和小型团队的任务列表。智能体可处理 Todoist 账户中的任务和项目。 | Sign in with Todoist |
| [Zapier](connectors/zapier.md) | 将数千种应用连接起来的自动化服务。你在 Zapier 中选择操作并粘贴生成的网址，智能体就只能获得你添加到该网址中的操作。请将该网址视为机密——令牌包含在地址中。 | Generated MCP URL |

### MCP 聚合器

这类服务会替你连接多个应用，并通过一个 MCP 服务器将它们提供给 Paperclip。对于此处没有原生连接器的应用，可以使用聚合器。设置方法以及智能体如何请求你选择聚合器，请参阅[通过 MCP 聚合器连接应用](connectors/mcp-aggregators.md)。

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [Arcade](connectors/mcp-aggregators.md) | 使用你设置的 Arcade MCP 网关公开的工具。 | Connect MCP server |
| [Composio](connectors/mcp-aggregators.md) | 通过 Composio Connect 发现并使用已连接的应用。 | Composio Connect |
| [Executor](connectors/mcp-aggregators.md) | 使用 Executor 工作区公开的工具。 | Connect MCP server |
| [Zapier](connectors/zapier.md) | 使用你添加到 Zapier MCP 服务器中的操作。也列在“生产力与协作”类别中。 | Generated MCP URL |

### 通信

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [AgentMail](connectors/agentmail.md) | 专为软件智能体打造的电子邮件收件箱。为智能体提供独立收件箱，并将每段邮件对话转换为 Paperclip 任务。所有实例均可使用，无需启用实验设置。 | API key |
| [Resend](connectors/resend.md) | 面向开发者的事务性邮件发送服务。智能体可使用你登录的 Resend 团队中的发件域名和投递记录。授权范围是完整访问，包括发送和删除，因此请谨慎设置操作权限。 | Sign in with Resend |
| [Slack](connectors/slack.md) | 团队消息服务。作为应用集成时，智能体只能在授权账户的访问范围内读取和发送消息。单独的**实验性**聊天渠道允许用户通过 Slack 与某个智能体协作。 | 智能体工具设置：参阅兼容性说明 · 聊天使用提供商应用注册 |

### 开发者工具

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [Browser Use Cloud](connectors/browser-use-cloud.md) | 在 Browser Use Cloud 中运行浏览器任务、从任务打开实时浏览器会话，并重复使用浏览器配置文件。 | API key |
| [Cloudflare](connectors/cloudflare.md) | DNS、CDN 和边缘计算。智能体可使用 Cloudflare 登录账户能够访问的资源。 | Sign in with Cloudflare · API key |
| [Context7](connectors/context7.md) | 获取最新的公共库文档和代码示例。只读且无需密钥；匿名访问受速率限制，无法访问私有仓库。 | No credential |
| [GitHub](connectors/github.md) | 代码托管和审查。智能体使用仓库工具时受已连接凭据的权限约束。设置方法和 shell 访问边界请参阅指南。拉取请求审查机器人作为单独条目 **GitHub Code Review Bot** 列在[聊天渠道](#chat-channels)下。 | Connect with Paperclip · API key |
| [Netlify](connectors/netlify.md) | Web 前端托管和部署。智能体可访问 Netlify 账户能够访问的团队和站点。 | Sign in with Netlify |
| [PagerDuty](connectors/pagerduty.md) | 值班排班和事件响应。智能体可访问令牌能够读取的事件、服务和排班。连接时请选择 US 或 EU 服务区域。 | API key |
| [Postman](connectors/postman.md) | API 开发和测试工作区。智能体可处理 Postman 账户能够访问的集合和 API。请选择工具集：Minimal、Code 或 Full。Minimal 并非只读；操作权限需另行设置。 | Sign in with Postman · API key |
| [Railway](connectors/railway.md) | 在已授权工作区中处理基础设施，包括 API 访问获准后的服务、部署和范围受限的日志操作。实时资格验证尚待完成；容器命令需要单独设置 SSH。 | Sign in with Railway |
| [Sentry](connectors/sentry.md) | 错误和性能监控。智能体可调查你授权的 Sentry 组织中的错误、发布版本和生产问题。 | Sign in with Sentry |
| [Superagent](connectors/superagent.md) | 安全平台。智能体可以审查安全发现、启动红队报告，并为内容和软件包评分。部分操作会消耗额度或永久删除数据，请设置审批门槛。 | API key |

### 数据与分析

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [Airtable](connectors/airtable.md) | 结合电子表格和数据库特性的结构化团队数据工具。智能体可处理 Airtable 登录账户或个人访问令牌能够访问的 base。 | Sign in with Airtable · Personal access token |
| [Bitly](connectors/bitly.md) | 短链接和点击分析。智能体可以读取链接、二维码和点击数据。授权范围包括 Bitly 的全部工具，还包括永久删除链接和发布到 Sites；因此请将写入和删除操作保持为 Off。 | Sign in with Bitly · API key |
| [ClickHouse](connectors/clickhouse.md) | 面向超大型数据集分析查询的列式数据库。智能体可以查询你指定的 ClickHouse Cloud 服务。 | Sign in with ClickHouse |
| [Enterpret](connectors/enterpret.md) | 客户反馈分析。智能体通过 Enterpret 只读服务器查询反馈主题并提取带引用的原文。图查询默认设为 **Ask first**。 | API key (auth token) · Sign in with Enterpret |
| [Gauge](connectors/gauge.md) | AI 答案可见度和内容工作流。智能体可以读取一个 Gauge 组织中的品牌提及、SEO 和流量数据。内容工具可以直接发布到你的 CMS；请将发布操作设为 **Ask first**。 | Sign in with Gauge · API key |
| [GSC Wizard](connectors/gsc-wizard.md) | 从 Google Search Console、Bing Webmaster 以及你在 GSC Wizard 中连接的资源获取搜索表现和分析数据。部分工具会提交 URL 以供编入索引；请单独审查这些工具。 | Sign in with GSC Wizard · MCP key |
| [Mixpanel](connectors/mixpanel.md) | 用户和事件行为的产品分析。智能体可使用 Mixpanel 账户能够访问的事件和报告。 | Sign in with Mixpanel |
| [Neon](connectors/neon.md) | 支持分支的无服务器 Postgres。智能体可以管理项目和分支、执行 SQL、检查架构，也可以将连接限定到某个项目。建议使用开发项目。 | Sign in with Neon · API key |
| [PostHog](connectors/posthog.md) | 提供会话回放、功能标志和实验功能的产品分析服务。智能体可以分析产品使用情况、错误、标志和实验。 | Sign in with PostHog · API key |
| [Supabase](connectors/supabase.md) | 托管 Postgres，附带身份验证、存储和边缘函数。智能体可使用连接范围限定的 Supabase 项目。建议使用开发项目。 | Sign in with Supabase · API key |
| [Windsor.ai](connectors/windsor-ai.md) | 汇总你在 Windsor.ai 中关联的数据源，提供营销、分析、CRM 和数据仓库数据。登录始终包含提供商的删除权限，部分工具还会更改广告活动或预算。 | Sign in with Windsor.ai · API key |

Google Sheets 也是常见的表格数据存储位置；它与 Google 套件中的其他服务一起列在 [Google Workspace](#google-workspace) 下。

### 内容与设计

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [Box](connectors/box.md) | 企业文件存储和共享。智能体可处理 Box 集成能够访问的文件和文件夹。 | Your own OAuth app |
| [Cloudinary](connectors/cloudinary.md) | 支持即时转换的图像和视频托管服务。智能体可使用 Cloudinary 角色允许访问的资源。 | Sign in with Cloudinary |
| [Webflow](connectors/webflow.md) | 带 CMS 的可视化网站构建工具。智能体可使用 Webflow 角色允许访问的站点和集合。 | Sign in with Webflow |
| [Wix](connectors/wix.md) | 网站构建和托管服务。智能体可访问 Wix 账户能够访问的站点。 | Sign in with Wix |

### 商务与金融

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [Shopify](connectors/shopify.md) | 在线商店电子商务平台。智能体可以搜索商店商品和政策并管理购物车。你只需提供商店域名，无需登录，也不会获得商店管理后台的访问权限。 | No credential |
| [Stripe](connectors/stripe.md) | 支付、开票和账单服务。智能体可以读取所连接账户中的客户、发票和付款。 | Sign in with Stripe · API key |

### AI 工具

包括机器学习相关服务，以及智能体用于查询资料的网页搜索和页面读取服务。它们属于带有操作列表的普通应用集成，这也是它们与下方[模型提供商](#model-providers)的区别。

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [Exa](connectors/exa.md) | 通过 Exa 无密钥服务器进行公共网页搜索和页面读取。仅有两项只读操作；无需账户或密钥。 | No credential |
| [Firecrawl](connectors/firecrawl.md) | 搜索、抓取和解析公共网页。无密钥访问提供三项读取操作；登录账户后会增加消耗团队额度的抓取和研究任务。 | Try keyless access · Sign in with Firecrawl · API key |
| [Hugging Face](connectors/hugging-face.md) | 开放机器学习模型、数据集和演示的公共平台。智能体可以搜索模型、数据集和文档。公共 Hub 方式无需账户或密钥，是最容易尝试的连接器；登录后可访问私有仓库以及仓库和作业操作。刚开始使用时，请关闭其写入操作。 | No credential (public Hub) · Sign in with Hugging Face |
| [并行搜索](connectors/parallel-search.md) | 通过 Parallel 无密钥服务器进行公共网页搜索和页面提取。仅有两项只读操作；无需账户或密钥。 | No credential |
| [Tavily](connectors/tavily.md) | 使用 Tavily 账户额度进行网页搜索和页面提取。只读。登录要求 Paperclip 使用公共 HTTPS 地址。 | Sign in with Tavily |
| [Telem.AI](connectors/telem.md) | 通过一个 API 密钥搜索网页并读取多个搜索提供商的页面。可选配置路由、层级和提供商列表。用量计入你的 Telem 账户。 | API key |
| [You.com](connectors/youcom.md) | 托管式网络智能服务。免费配置提供精简的只读工具集；仍受提供商限制。此设置尚未验证。 | Sign in with You.com · API key · No credential (free profile) |

<span id="memory-experimental"></span>

<a id="memory-experimental"></a>
### 记忆——实验性

这类长期记忆服务允许智能体跨运行保存并回忆信息。管理员在实例的实验设置中启用 **Memory connectors** 前，所有这些服务的设置入口都会隐藏；该选项默认关闭。再次关闭后只会隐藏设置入口，现有连接仍会继续运行。

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [Mem0](connectors/mem0.md) | 面向 AI 应用的托管长期记忆。智能体可在 Mem0 项目中跨运行存储和回忆事实。 | API key |
| [Zep](connectors/zep.md) | 检索时间图记忆和已授权的业务上下文。需要启用了 Memory MCP 的 Zep 项目；项目 API 密钥在此处不可用。 | Sign in with Zep |
| [Supermemory](connectors/supermemory.md) | 搜索并保存共享记忆、文档和个人资料。你可以选择此连接可使用的空间。 | Sign in with Supermemory |
| [Cognee](connectors/cognee.md) | 使用 Cognee Cloud 租户，从文档和对话构建并检索共享图记忆。它会在 Paperclip 运行时主机上运行 Cognee 的 MCP 客户端，因此公共部署需要可信任的 MCP 运行时主机。 | API key |
| [Honcho](connectors/honcho.md) | 记住对话，并检索与相关人员有关的上下文。 | API key |

实验开关、提供商差异以及连接共享哪些内容，请参阅[记忆连接器](connectors/memory-connectors.md)。

<a id="model-providers"></a>
### 模型提供商

这里保存 Paperclip 运行模型所需的凭据。它们不提供任何操作，也不会出现在权限列表中。不同适配器使用哪些提供商，请参阅[智能体适配器](guides/org/agent-adapters.md)。

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [Anthropic](connectors/anthropic.md) | 运行 Claude 模型。订阅方式使用 Claude 套餐登录；API 密钥可以单独限定范围和轮换。 | Claude subscription · API key |
| [OpenAI](connectors/openai.md) | 运行 OpenAI 模型。 | OpenAI subscription · API key |
| [OpenRouter](connectors/openrouter.md) | 通过一个凭据和一张账单运行多个厂商的模型。 | API key |
| [Grok](connectors/xai.md) | 运行 xAI 的 Grok 模型。 | Grok subscription · API key |
| [自定义模型提供商](connectors/custom-model-providers.md) | 通过受支持的智能体 harness 使用兼容的自定义或本地端点。是否兼容取决于 API 路由。 | API key · Endpoint URL |

[检查 AI 用量](connectors/ai-usage.md)介绍提供商支持的用量快照和重新连接提示。这些凭据与[助手连接（MCP）](experimental/assistant-connections.md)不同；后者允许外部助手以个人身份登录并使用 Paperclip。

<span id="chat-channels"></span>

<a id="chat-channels-experimental"></a>
<a id="chat-channels"></a>
### 聊天渠道——实验性

这些渠道让用户可以与智能体交谈。管理员必须在实例实验设置中启用 **Chat connectors**；该选项默认关闭。GitHub Code Review Bot 是例外：它由单独且默认关闭的 **GitHub review bots** 设置控制。设置时需要在提供商处注册应用或机器人，并将其指向 Paperclip webhook；一个连接对应一个智能体。

| 连接器 | 可执行的操作 | 连接方式 |
| --- | --- | --- |
| [GitHub Code Review Bot](connectors/github-review-bot-setup.md) | 让智能体以自己的 App 身份审查 GitHub 拉取请求并回复提及。选择智能体后，GitHub 会为你创建并安装 App。需要单独启用 **GitHub review bots** 设置。 | Provider app registration |
| [Discord](connectors/discord.md) | 在服务器频道中提及智能体，Paperclip 会开启一个与单项任务关联的讨论串。 | Provider app registration |
| [iMessage Photon](connectors/imessage-photon.md) | 通过 Photon Cloud 项目从 Apple Messages 向智能体发送消息。共享 Pro 线路支持私信；专用线路还支持你启用的群组。 | Provider app registration |
| [Microsoft Teams](connectors/microsoft-teams.md) | 在聊天、团队频道或群聊中向智能体发消息。需要工作或学校 Microsoft 365 组织；个人 Teams 账户无法完成设置。 | Provider app registration |
| [Slack](connectors/slack.md) | 直接向智能体发消息，或在频道中提及它，从该讨论串启动任务。它与同一页面上的 Slack 智能体工具需分别设置。 | Provider app registration |
| [Telegram](connectors/telegram.md) | 通过使用 BotFather 创建的机器人向智能体发送消息。 | Provider app registration |

<a id="cant-find-your-app"></a>
## 找不到你的应用？

上方目录列出了此版本文档记录的连接器。如果你需要的服务不在其中，可采用以下现有方式。

**先检查你自己的实例。**已安装版本可能提供不同的目录；实例是否加入 Paperclip Cloud，以及聊天渠道是否启用，也会影响可用方式。你在 **Connectors** 中看到的内容才是准确信息，本页面不能代表你的实例。

**将其作为自定义服务器连接。**如果服务提供自己的远程 MCP 服务器，你可以直接连接，无需等待目录收录。它与目录连接器一样支持身份、授权、智能体选择和单项操作控制。兼容要求和提供 URL 的两种方式见[连接自定义 MCP 服务器](connectors/custom-mcp-servers.md)。

**通过 MCP 聚合器访问。**[Arcade、Composio、Executor 和 Zapier](connectors/mcp-aggregators.md)都可以代表你连接多个应用。你不一定需要自行寻找：智能体需要尚无原生连接器的应用时，会询问你选择上述哪个提供商，或者选择 **None for now**（暂时不选）。使用 Zapier 时，你可以在 Zapier 一侧挑选操作，因此智能体无法超出这些操作范围。

**改为给智能体工作区。**有些工作完全不需要连接器。只要智能体能通过 shell 和 CLI 完成，就可以在执行工作区中进行。实现方式见[将智能体连接到 GitHub 仓库](how-to/connect-agent-to-github.md)；适配器级连接方式见[向智能体添加 MCP 服务器](how-to/add-mcp-server-to-agent.md)。

**如果提供商限制访问。**有些提供商会先审核并批准 MCP 客户端，独立注册的客户端才能工作。Paperclip 中的操作不会改变这一要求，批准也必须由提供商作出。[Paperclip 识别但未列出的提供商](connectors/recognized-providers.md)列出了当前清单中的相关提供商及其待处理事项。

## 连接不代表授予什么

连接之前，有两点容易让人意外，值得先了解。

**连接器不等于权限。**连接 Notion 并不意味着所有智能体都能编辑你的 Notion。连接器保存凭据；另外一组设置决定凭据属于哪些人员、哪些智能体可以使用，以及这些智能体能调用哪些具体操作。[连接器访问方式](connectors/access-model.md)介绍基本原理；[设置操作权限](connectors/action-permissions.md)说明如何控制。

**大多数操作列表由提供商提供，而不是 Paperclip。**连接器连接到提供商托管的服务器时，工具列表由提供商发布。Paperclip 读取列表，将各项归类为读取、写入或破坏性操作，并允许你分别将其设为 **Allowed**、**Ask first** 或 **Off**。提供商更新服务器时，你看到的列表也会随之变化。因此，要了解连接器实际能做什么，应以你自己实例中的列表为准，而不是本页的列表。

## 通用指南

以下指南介绍连接器提供的控制功能。操作权限和审查请求适用于智能体工具连接；模型凭据和聊天渠道使用不同的控制方式。

| 指南 | 适用场景 |
| --- | --- |
| [与人员和智能体共享连接器](connectors/share-access.md) | 决定凭据属于谁，以及哪些智能体可以使用。 |
| [为人员和智能体使用不同账户](connectors/separate-accounts.md) | 某个智能体需要使用自己的账户，而不是借用某个人的账户。 |
| [设置操作权限](connectors/action-permissions.md) | 将单项操作设为 Allowed、Ask first 或 Off。 |
| [处理连接器审查请求](connectors/review-requests.md) | 智能体正在等待你批准 Ask first 操作。 |
| [验证连接器并修复故障](connectors/verify-and-troubleshoot.md) | 检查新连接，或排查已经停止工作的连接。 |
| [重新授权、撤销或断开连接](connectors/reauthorize-and-disconnect.md) | 轮换凭据、移除某个人的访问权限，或删除连接。 |
| [连接自定义 MCP 服务器](connectors/custom-mcp-servers.md) | 服务不在目录中，或你需要粘贴 URL 或配置。 |
| [通过 MCP 聚合器连接应用](connectors/mcp-aggregators.md) | 应用没有原生连接器，或智能体要求你选择外部提供商。 |
| [连接说明](connectors/connection-instructions.md) | 保存与连接器一同提供给智能体的服务专属说明。 |

## 相关内容

- [工具网关](reference/api/tool-gateway.md)——智能体工具调用与外部服务之间的中间层。
- [Paperclip 识别但未列出的提供商](connectors/recognized-providers.md)——当前被隐藏以及受提供商准入限制的服务清单。
- [为智能体的工具包添加 MCP 服务器](how-to/add-mcp-server-to-agent.md)——适配器级接入方式，与连接器不同。
- [将智能体连接到 GitHub 仓库](how-to/connect-agent-to-github.md)——在工作区中使用 git 和 `gh`，与 GitHub 连接器不是一回事。
- [Connections v3（连接器的底层基础）](experimental/connections-apps.md)——这些连接器所依赖的存储与授权基础。
