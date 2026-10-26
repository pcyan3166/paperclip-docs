---
paperclip_version: v2026.1023.0
seo_title: 并行搜索连接器
seo_description: 通过 Parallel Search 为智能体提供无需密钥的公共网络搜索和页面阅读功能。无需账户，了解两个操作的作用，以及首先运行的快速测试。
---

# 并行搜索

智能体可以通过 Parallel Search 搜索公共网络并获取源页面的文本。它不需要账户，因此可以快速为智能体提供公司外部的最新信息。

> **注意：** Parallel Search 出现在每个实例的目录中；无需启用实验性设置。

## 连接前准备

无需任何准备。此连接使用 Parallel 的公共搜索服务器，因此无需登录，也无需 API 密钥。

## 连接 Parallel Search

1. 打开 **连接器** 并选择 **Parallel Search**。
2. 阅读主按钮上方的访问权限行。它说明了该连接属于谁以及哪些智能体可以使用它。选择 **更改** 以缩小智能体范围。
3. 选择 **连接**。

## 选择访问权限

公共服务器提供两个只读操作：`web_search` 和 `web_fetch`。这就是整个连接的全部内容。Parallel 独立的 Task 产品是另一个服务器，不包含在内。

因为它仅读取公共网络，所以这是风险较低的连接器之一，适合共享。请牢记以下两点：

- **Parallel 能看到什么。** 您智能体的搜索词会发送给 Parallel。请勿在查询中包含私密细节。
- **智能体如何处理结果。** 搜索结果属于外部内容。要求智能体在报告时保留源链接和摘录，并将结果视为需要核查的信息，而非需要遵循的指令。参见 [设置操作权限](action-permissions.md)。

## 试用

```txt
Search the web with Parallel Search for the latest pricing page of a tool we use, and summarise it with the source link.
```

打开链接并进行比较。一个可核查的结果可以确认连接状态以及智能体的权限。

> **注意：** 此为示例任务，并非记录在案的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 大量使用后搜索开始失败 | Parallel 可能会限制对公共服务器的重度使用 | 降低使用频率，或使用其他搜索连接器 |
| 智能体寻找 Parallel 的任务或研究工具 | 这些属于 Parallel Task，是一个独立产品 | 仅使用搜索和获取操作 |
| 无法读取页面 | 该网站阻止自动化读取，或需要登录 | 尝试其他来源 |
| 智能体报告某项主张但未提供链接 | 它未保留来源 | 要求它在每个事实旁附上 URL |

限制：仅限公共网络，且仅支持公共服务器暴露的两个操作。

## 相关指南

- [Exa](exa.md) — 另一个无需密钥的网络搜索连接器。
- [Tavily](tavily.md) — 使用您自己的 Tavily 账户进行网络搜索。
- [Telem.AI](telem.md) — 使用一个密钥跨多个提供商进行搜索。
- [设置操作权限](action-permissions.md)
- [Parallel Search MCP 文档](https://docs.parallel.ai/integrations/mcp/search-mcp)

## 来源

- [Parallel Search 定义](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/parallel-search.json) — 方法标签、服务器及指南。
- [权限审查](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/doc/connections/tool-method-permission-reviews.json) — 两个公共操作。
