---
paperclip_version: v2026.1023.0
seo_title: Firecrawl 网页抓取连接器
seo_description: 让智能体使用 Firecrawl 搜索、抓取和解析公共网页。对比无密钥、登录和 API 密钥方法，涉及消耗额度的任务、测试及修复方案。
---

# Firecrawl

智能体可以通过 Firecrawl 搜索网络、将页面抓取为干净的文本以及解析文档。连接账户后，它们还可以运行更大规模的抓取和研究任务。

Firecrawl 出现在每个实例的目录中；无需启用实验性设置。

## 连接前准备

首先选择一种方法，因为它们在覆盖范围和成本上差异很大：

| 方法 | 所需条件 | 智能体获得的功能 |
| --- | --- | --- |
| **尝试无密钥访问** | 无 | 仅限搜索、抓取和解析，具有有限的免费额度 |
| **使用 Firecrawl 登录** | Firecrawl 账户和团队 | 完整的账户工具集，包括消耗额度的任务 |
| **使用 Firecrawl API 密钥** | 对应 Firecrawl 团队的 API 密钥 | 完整的账户工具集，适用于无人值守或更高限额的使用场景 |

对于浏览器登录，Paperclip 必须通过 HTTPS 访问，或者在本地运行时通过 `http://` 回环地址访问。

## 连接 Firecrawl

1. 打开 **连接器** 并选择 **Firecrawl**。
2. 阅读主按钮上方的访问说明行。它说明了连接所属者以及哪些智能体可以使用它。选择 **更改** 以选择其他方法、身份或智能体集合。
3. 完成您选择的方法：
   - **尝试无密钥访问** — 连接即可。无需输入其他内容。
   - **使用 Firecrawl 登录** — 登录并选择应适用其使用情况的 Firecrawl 团队。Paperclip 会自动注册其客户端，并请求 `firecrawl:global` 和 `offline_access` 权限范围。
   - **使用 Firecrawl API 密钥** — 在 Firecrawl 中为预期团队创建密钥，然后将其粘贴到 **您的 Firecrawl 密钥** 字段中。

Paperclip 将密钥作为机密存储，并作为 Bearer 标头发送给 Firecrawl；智能体永远看不到它。Paperclip 无法赋予密钥超出 Firecrawl 授予的权限。

## 选择访问权限

无密钥方法暴露三个读取操作：`firecrawl_search`、`firecrawl_scrape` 和 `firecrawl_parse`。这是最简单的入门方式。

登录和 API 密钥连接暴露更多功能。抓取和研究任务在 Firecrawl 端运行，并消耗您团队的额度，广泛的抓取可能会迅速消耗大量额度。新连接默认对所有智能体将所有操作设置为 **允许**，因此在共享账户连接之前：

- 打开 **权限** 选项卡，将抓取和研究任务设置为 **先询问**。
- 将连接保持在您愿意为其付费的团队上。

参见 [设置操作权限](action-permissions.md)。

## 试用

```txt
Use Firecrawl to scrape https://paperclip.ing and give me the page title and a two-sentence summary. Do not start a crawl.
```

单次抓取成本低廉，且易于与浏览器中的页面进行核对。如果您使用 **权限** 选项卡上的操作测试器，请打开 **更多选项**，仅填写 URL，并保持嵌套的可选设置不变。

> **注意：** 此为示例任务，非记录的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 修复方法 |
| --- | --- | --- |
| 无密钥调用停止工作 | 无密钥额度已用尽 | 等待，或使用 Firecrawl 账户或密钥连接 |
| 仅列出三个操作 | 您使用 **尝试无密钥访问** 进行了连接 | 使用登录或 API 密钥重新连接以获取完整功能集 |
| 额度消耗速度超出预期 | 智能体启动了抓取或研究任务 | 将这些操作设置为 **先询问**，并在 Firecrawl 中检查任务历史 |
| 密钥被拒绝 | 密钥已被撤销，或属于另一个团队 | 为正确的团队创建新密钥并重新连接 |
| **需要关注** | 授权或密钥已被撤销 | 选择 **重新连接** |

限制：每个连接对应一个 Firecrawl 团队。可用的操作取决于 Firecrawl 托管服务器为您选择的方法所暴露的内容。

## 相关指南

- [Exa](exa.md)、[Parallel Search](parallel-search.md) 和 [Tavily](tavily.md) — 网络搜索连接器。
- [Browser Use Cloud](browser-use-cloud.md) — 运行完整的浏览器任务而非抓取。
- [设置操作权限](action-permissions.md)
- [Firecrawl MCP 文档](https://docs.firecrawl.dev/mcp-server)

## 来源

- [Firecrawl 定义](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions/firecrawl.json) — 方法、权限范围、凭据字段和警告。
- [权限审查](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/doc/connections/tool-method-permission-reviews.json) — 无密钥操作和消耗额度的任务。
- [默认操作策略](https://github.com/paperclipai/paperclip/blob/1b4c803816e82b0265c8422e1a804327a7ed8c54/packages/shared/src/app-definitions.ts) — 新连接对所有智能体开放，操作状态为允许。
