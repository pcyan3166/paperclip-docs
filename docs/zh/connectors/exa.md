---
paperclip_version: v2026.1023.0
seo_title: Exa 网络搜索连接器
seo_description: 通过 Exa 的免费、无密钥服务器为智能体提供公共网络搜索和页面阅读功能。无需账户或密钥，了解连接覆盖范围并进行快速测试。
---

# Exa

智能体可以通过 Exa 搜索公共网络并阅读其找到的页面。这是一种快速为智能体提供来自公司外部最新信息的方式，且完全不需要账户。

Exa 出现在每个实例的目录中；无需启用实验性设置。

## 连接前准备

无需任何准备。此连接使用 Exa 的公共搜索服务器，因此无需登录、无需 API 密钥，也不产生费用。

## 连接 Exa

1. 打开 **Connectors** 并选择 **Exa**。
2. 阅读主按钮上方的访问权限说明行。该行显示连接所属对象以及哪些智能体可以使用它。选择 **Change** 以缩小智能体范围。
3. 选择 **Connect**。

## 选择访问权限

公共服务器提供两个只读操作：`web_search_exa` 用于搜索，`web_fetch_exa` 用于阅读页面。Exa 仅限账户使用的工具，例如其研究代理（research agent）和高级搜索选项，不包含在此连接中。

由于它仅读取公共网络，因此这是适合共享的低风险连接器之一。但仍需注意以下两点：

- **Exa 看到的内容。** 您的智能体的搜索词将发送给 Exa。请勿在查询中包含私密细节。
- **智能体如何处理结果。** 搜索结果属于外部内容。智能体应将其视为需要核实的信息，而非需要遵循的指令，并应在报告的任何内容中保留来源链接。有关在其他连接器中将后续操作保持在 **Ask first** 状态，请参阅 [设置操作权限](action-permissions.md)。

## 试用

```txt
Search the web with Exa for the official documentation of a tool we use, and give me the top three results with their links.
```

打开智能体返回的链接。可核实的结果能同时确认连接状态和智能体的权限。

> **注意：** 此为示例任务，非记录的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 大量使用后搜索开始失败 | 公共服务器的免费额度或速率限制 | 降低使用频率，或为大量工作使用其他搜索连接器 |
| 智能体无法找到 Exa 的研究或高级工具 | 这些工具需要 Exa 账户，且不在公共服务器上 | 仅使用搜索和获取操作 |
| 无法阅读某个页面 | 该站点阻止自动化阅读，或需要登录 | 尝试其他来源 |
| 智能体报告某项主张但未提供链接 | 它未保留引用 | 要求它在每个事实旁附上来源 URL |

限制：仅限公共网络，且仅限公共服务器暴露的两个操作。此版本中没有基于账户的方法。

## 相关指南

- [并行搜索](parallel-search.md) — 另一个无密钥网络搜索连接器。
- [Tavily](tavily.md) — 使用您自己的 Tavily 账户进行网络搜索。
- [Telem.AI](telem.md) — 使用一个密钥跨多个提供商进行搜索。
- [设置操作权限](action-permissions.md)
- [Exa MCP documentation](https://exa.ai/docs/get-started/exa-mcp)

## 来源

- [Exa definition](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/exa.json) — 方法标签、服务器及指南。
- [Permission review](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/doc/connections/tool-method-permission-reviews.json) — 两个公共操作及排除项。
