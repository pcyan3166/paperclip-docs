---
paperclip_version: v2026.517.0
seo_title: LLM Wiki 插件
seo_description: 智能体从原始资料（任务、评论、文档和文件）构建并维护的带引用内嵌 Wiki。
---

# LLM Wiki

如果你想要一个带引用、嵌入控制面板的 Wiki，并让智能体从原始资料（Paperclip 任务、评论、文档以及你放入文件夹的文件）构建和维护它，那么 `@paperclipai/plugin-llm-wiki` 插件就能实现这一点。它会将资料收集到本地文件夹，让智能体将其整理为带反向链接和来源信息的 Markdown 页面，并在 Paperclip 中提供 Wiki 页面，供你和智能体浏览结果。

本页面向希望启用和配置此插件的**操作员**。

> LLM Wiki 插件和插件运行时目前都处于 alpha 阶段。Paperclip 版本升级可能带来不兼容变化；依赖此插件时请固定版本。

---

## 快速安装

此插件以已发布的 npm 包形式提供，因此任何 Paperclip 安装方式（Docker、systemd、裸机部署等）都可以在配置了 Paperclip CLI 的主机上运行一条命令完成安装：

```sh
paperclipai plugin install @paperclipai/plugin-llm-wiki
```

也可以在控制面板中依次打开：**设置 → 插件 → 安装插件**，粘贴 `@paperclipai/plugin-llm-wiki` 并提交。

在 `paperclipai plugin examples` 中不会显示此插件——该命令只列出四个内置参考**示例**插件，而非所有第一方插件。安装 LLM Wiki 时请使用完整包名。

如果你使用的是 monorepo 检出版本，请参阅[本地开发插件](../../how-to/develop-a-plugin-locally.md)，并将安装器指向 `packages/plugins/plugin-llm-wiki`。

> **注意（v2026.517.0）：** 现在发布的插件包中包含所有引导资源（模板、智能体指令文件和迁移脚本），首次启用后不再出现 Wiki 根目录尚未展开的情况。用于创建 `wiki/index.md`、`wiki/log.md` 和标准目录结构的引导模板与开发环境使用的是同一份模板。从旧版本升级时，请在插件设置中重新执行**引导**操作，以生成任何新增的必需文件；现有文件会保留。

首次启用时，插件会申请一组范围较广的能力，包括 `local.folders`、`database.namespace.*` 系列、`agents.managed`、`skills.managed`、`routines.managed` 和 `ui.page.register`。这是因为插件会管理智能体、项目、例程、技能、REST 路由和数据库命名空间。批准之前，请查看插件详情页中的权限卡片。

---

## 安装后会获得什么

安装并启用插件后，Paperclip 界面中会出现以下三项：

- **Wiki 侧边栏入口**（插槽 ID 为 `wiki-sidebar`，顺序为 `35`），用于打开插件页面。
- **Wiki 页面**，挂载在插件所属公司命名空间中的 `wiki` 路由下（插槽 ID 为 `wiki-page`）。你可以在此浏览页面、收集资料、启动查询会话并检查操作。
- **Wiki 路由侧边栏**（插槽 ID 为 `wiki-route-sidebar`），用于在 Wiki 内导航空间和页面。

插件还会在后台注册：

- 一个托管智能体——**Wiki 维护者**（`agentKey: wiki-maintainer`，角色为 `knowledge-maintainer`），负责摄取、查询、检查和索引工作。
- 一个托管项目——**LLM Wiki**（`projectKey: llm-wiki`），用于收集插件的操作任务。
- 三个默认暂停的托管例程：`cursor-window-processing`（处理 Paperclip 任务历史窗口）、`nightly-wiki-lint`（检查页面孤立、来源信息过期和内容矛盾）以及 `index-refresh`（重建 `wiki/index.md`）。
- 一组为维护者智能体安装的托管技能：`wiki-maintainer`、`wiki-ingest`、`wiki-query`、`wiki-lint`、`paperclip-distill` 和 `index-refresh`。

这些例程默认暂停，因此在你决定启用之前不会运行。你可以在插件设置中启用它们，也可以通过标准的托管例程界面启用。

---

## Wiki 根文件夹

插件声明了一个本地文件夹挂载——**Wiki 根目录**（`folderKey: wiki-root`，`access: readWrite`）。Wiki 生成或读取的所有内容都存放在这里。

通过插件设置页面或 `bootstrap` / `bootstrap-space` API 路由引导文件夹时，插件会创建以下目录结构：

```text
<wiki-root>/
  AGENTS.md
  IDEA.md
  wiki/
    index.md
    log.md
    sources/
    projects/
    entities/
    concepts/
    synthesis/
  raw/
```

`AGENTS.md` 和 `IDEA.md` 是控制文件，分别存放维护者智能体的指令和公司说明的方向。这两个文件受到保护：智能体工具无法覆盖它们，只能由操作员编辑。`wiki/log.md` 是各例程持续追加内容的审计日志。`raw/` 用于保存尚未整理为正式页面的原始资料。

引导操作会保留现有文件而不会覆盖它们，因此开始编辑后仍可安全地再次运行。

---

## 空间

一个 Wiki 根目录可以托管多个**空间**，每个空间都是同一已配置文件夹下相互独立的 Wiki 内容集合。插件始终会创建一个默认空间（slug：`default`），你可以通过 `POST /spaces` 路由或设置界面创建更多空间。每个空间都有独立的 `wiki/`、`raw/`、页面索引和操作历史。

请预先了解：**从 Paperclip 摄取的资料始终写入 `default` 空间。** 游标窗口、提炼和回填流程目前尚不支持按空间区分，因此非默认空间暂时只能通过手动方式或原始文件摄取资料。后续阶段会加入按空间配置的 Paperclip 摄取配置。

空间表通过插件的 `003_spaces.sql` 迁移创建，目前该迁移已通过安全限定命名空间的 SQL 校验并执行。插件运行时会拒绝任何越出插件数据库命名空间的迁移，因此错误迁移会在安装时明确失败，而不会污染宿主数据库架构。

---

## 收集资料并构建页面

插件提供两种将原始资料整理为 Wiki 页面的方式：

**手动收集。** 将文件放入 `<wiki-root>/raw/`，或调用 `POST /sources` 路由（`routeKey: capture-source`），将其注册为已收集的来源，并在插件数据库命名空间中保存元数据。随后，智能体（通常是运行 `wiki-ingest` 技能的 Wiki 维护者）会将原始资料整理为带引用的 Markdown 页面，放入 `wiki/concepts/`、`wiki/entities/`、`wiki/synthesis/` 或某个项目目录。

**摄取 Paperclip 事件。** 可选择启用公司范围内对 Paperclip 任务、评论和文档的摄取。此功能**默认关闭**。启用后，插件工作进程会监听宿主事件，将符合条件的内容收集到默认空间的 `raw/`，再由维护者智能体提炼为项目页面。默认情况下，每个来源的大小限制为：任务 12,000 个字符、游标窗口 60,000 个字符、例程运行 120,000 个字符。

页面以普通 Markdown 文件的形式存放在磁盘上。插件会在自己的数据库命名空间（`namespaceSlug: llm_wiki`）中为页面建立索引，以支持搜索、反向链接和修订历史；同时使用 Paperclip 本地文件夹 API 执行路径包含检查、符号链接检查和原子写入。两个文件为保留文件：智能体工具不能写入 `AGENTS.md` 和 `IDEA.md`。

---

## 智能体可调用的工具

当维护者智能体（或任何被授予 `pluginTools: [paperclipai.plugin-llm-wiki]` 的智能体）运行时，插件会提供一组专用于 Wiki 的工具：

| 工具 | 用途 |
|---|---|
| `wiki_search` | 在一个空间内对页面和来源元数据进行全文搜索。 |
| `wiki_list_pages` | 返回某个空间中已建立索引的页面列表。 |
| `wiki_read_page` | 读取 Markdown 页面。 |
| `wiki_write_page` | 原子写入页面；可选使用 `expectedHash` 防止基于过期内容写入。 |
| `wiki_propose_patch` | 返回结构化的拟议写入，但不更改文件，适用于审核流程。 |
| `wiki_list_sources` / `wiki_read_source` | 查看已收集的原始资料，然后再添加引用。 |
| `wiki_list_backlinks` | 查找链接到指定页面的其他页面。 |
| `wiki_update_index` | 原子替换 `wiki/index.md`。 |
| `wiki_append_log` | 向 `wiki/log.md` 追加维护记录。 |

每个工具都接受 `companyId`、`wikiId` 和可选的 `spaceSlug`。执行操作的智能体应传入任务中的 `spaceSlug`；省略时会回退到 `default` 空间。

---

## REST 路由

插件会在其命名空间下挂载以下路由（授权列表示哪些调用方可以使用）：

| 方法与路径 | 路由键 | 授权 |
|---|---|---|
| `GET /overview` | `overview` | board-or-agent |
| `POST /bootstrap` | `bootstrap` | board |
| `POST /sources` | `capture-source` | board-or-agent |
| `GET /spaces` | `spaces` | board-or-agent |
| `POST /spaces` | `create-space` | board |
| `PATCH /spaces/:spaceSlug` | `update-space` | board |
| `POST /spaces/:spaceSlug/bootstrap` | `bootstrap-space` | board |
| `POST /spaces/:spaceSlug/archive` | `archive-space` | board |
| `GET /operations` | `operations` | board-or-agent |
| `POST /query-sessions` | `start-query` | board |
| `POST /file-as-page` | `file-as-page` | board |

需要 `board` 授权的路由用于操作员操作；需要 `board-or-agent` 授权的路由可由维护者智能体在例程中安全调用。

---

## 提示与常见用法

- **先从一个空间开始。** 默认空间足以用于评估插件。只有在确实存在不应与其他内容混合的独立知识集合时，才添加空间。
- **配置好 Wiki 根目录之前，先让例程保持暂停。** 维护者智能体仍会创建操作任务，但文件夹引导完成前，其工具会拒绝写入。
- **由你自己编辑 `IDEA.md`。** 维护者会阅读此文件来理解公司的方向；应将其视为由操作员维护的文档，而非由智能体管理的文档。
- **审核流程使用 `wiki_propose_patch`。** 如果希望操作员参与修改，请让维护者提交拟议补丁，而不是直接写入，再通过对应的操作任务批准补丁。
- **留意 `wiki/log.md`。** 每个例程都会在其中追加简短记录，包括孤立页面数量、已处理的来源窗口和索引更新时间。这是查看维护者工作情况最快的方式。

---

## 相关内容

- [管理 → 插件](../../administration/plugins.md)——面向操作员的插件管理器。
- [操作指南 → 本地开发插件](../../how-to/develop-a-plugin-locally.md)——让 Paperclip 使用本地检出的插件代码。
- [参考 → 插件 SDK](./sdk.md)——插件开发接口；可用于扩展 Wiki 插件或编写自己的插件。
