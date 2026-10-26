---
paperclip_version: v2026.1012.0
seo_title: connections 命令
seo_description: 在正在运行的心跳任务中使用运行时提供的工具搜索连接目录，或请求访问某项服务。
---

# connections 命令

`connections` 命令允许智能体在正在运行的心跳任务**内部**查询可连接的服务并请求访问权限。智能体可以借此发现“是否有此服务的连接器？”，然后发起连接意向，而不必离开运行时提供的终端环境。

```sh
paperclipai connections search "<query>"
paperclipai connections request <service>
```

> **注意：** 这些命令只能在活动心跳任务中使用。它们会调用运行时连接工具；工具端点和令牌会注入该运行的环境（`PAPERCLIP_RUNTIME_TOOLS_CONNECTIONS_SEARCH_URL`、`PAPERCLIP_RUNTIME_TOOLS_CONNECTION_REQUEST_URL` 和 `PAPERCLIP_RUNTIME_TOOLS_TOKEN`）。在其他环境运行时，命令会报错并说明需要活动运行提供的连接环境。

---

## 搜索连接

`connections search` 会在连接目录中查找服务或能力。查询内容可选——省略即可浏览。你可以传入服务名称，也可以用日常语言描述需求；完整句子也可以，多余的词不会妨碍匹配，轻微拼写错误或拆开的名称（例如 “Agent Mail”）通常也能找到对应服务。

```sh
paperclipai connections search "google calendar"
paperclipai connections search github
paperclipai connections search "I need somewhere to receive customer emails for this task"
paperclipai connections search
```

| 参数 / 标志 | 用途 |
|---|---|
| `[query]` | 要搜索的服务名称、能力或自然语言说明，最多 4,000 个字符。可选；默认执行空查询。 |
| `--retry-provider-choice` | 重新考虑之前选择的服务提供商（或之前选择的“None”）。仅在用户明确要求重新考虑时使用。 |
| `--json` | 以格式化（缩进）JSON 输出结果。 |

结果会以 JSON 格式写入标准输出。不带 `--json` 时输出压缩的单行 JSON；带上后输出经过美化的 JSON。

### 服务只能通过外部提供商访问时

优先返回内置应用。如果目录中有所搜索服务的原生应用，除非用户明确指定了某个提供商，否则会返回该应用。如果没有，搜索结果可能会提供通过**外部提供商**（例如 Arcade 这样的 MCP 聚合器）访问同一应用的方式。这类结果的服务 slug 格式为 `via:<provider>:<app>`；结果中的 `instruction` 会要求智能体询问用户使用哪家提供商，并将“暂不选择（None for now）”作为选项。

Paperclip 会记住用户的回答。如果用户选择“暂不选择”，同一任务后续运行时不会反复询问。传入 `--retry-provider-choice` 可重新打开选择流程。设置连接之前仍须取得用户的新答复；此标志表示“用户让我重新考虑”，不能用来绕过用户的决定。

---

## 请求连接

`connections request` 会为指定服务发起连接请求。服务通过 slug 标识，通常是 `connections search` 返回的值。

```sh
paperclipai connections request google-calendar
paperclipai connections request github --json
```

| 参数 / 标志 | 用途 |
|---|---|
| `<service>` | **必填。** 要请求的可连接服务 slug。 |
| `--selection-interaction-id <id>` | 已回答的提供商选择问题 ID。与用户选定的 `via:<provider>:<app>` 服务一并传入。 |
| `--target-service <slug>` | 用户明确指定外部提供商时，要通过该提供商访问的应用。请使用搜索结果给出的应用 slug。 |
| `--json` | 以格式化（缩进）JSON 输出结果。 |

与 `search` 一样，响应以 JSON 格式写入标准输出；带上 `--json` 时会进行美化排版。

服务器会再次检查两个提供商标志，并验证任务、请求智能体、用户答复以及当前是否仍允许该路由。这两个标志都不会授予提供商连接本身未具备的访问权限。

对于 `agentmail`，请求会在任务上为负责的用户显示 API 密钥卡片，而不是设置链接。该卡片会为发起请求的智能体创建收件箱。请求完成之前，不要告知他人该智能体的电子邮件地址。

> **提示：** 先搜索，再请求。将 `search` 结果中的服务 slug 直接传给 `request`，确保请求的是目录中实际存在的服务。

---

## 另请参阅

- [运行命令](./run.md)——提供这些命令所需环境的心跳运行。
- [适配器命令](./adapter.md)——在服务器端执行智能体工作的运行时。
- [输出和脚本](./output-and-scripting.md)——如何处理这些命令输出的 JSON。
