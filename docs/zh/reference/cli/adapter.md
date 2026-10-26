---
paperclip_version: v2026.916.0
seo_title: CLI 适配器命令
seo_description: 管理服务器端运行时；这些运行时通过统一接口封装 Claude、Codex 或其他服务提供商，将唤醒请求转换为实际运行。
---

# 适配器命令

适配器是实际执行智能体工作的服务器端运行时。它们封装 Claude、Codex 或其他服务提供商，并将唤醒请求转化为真正的运行。CLI 本身不会运行适配器；它只检查和管理 Paperclip 运行时已注册的适配器。需要查看有哪些适配器类型、在基于某适配器创建智能体前读取其配置架构、列出公司可用模型，或安装、更新和移除外部适配器软件包时，可使用这些命令。

> **注意：** Paperclip 中的大多数智能体工作由运行时及其适配器在服务器端执行。CLI 负责触发和观察这些工作——详见[工作在哪儿运行](../../guides/welcome/key-concepts.md)。`adapter` 命令让控制面板操作员能够检查这层执行环境。

---

## 适配器类型

每个适配器都通过其**类型**标识，这是一个稳定字符串，例如 `codex_local` 或 `claude_local`。几乎所有子命令都将类型作为第一个参数。`list` 是一个例外，它会列出所有已注册类型，供你确认可用值；`install` 是另一个例外，它会从软件包注册新的外部适配器。

创建智能体时需要设置其 `adapterType`。因此，先读取适配器配置架构（见下文）即可确切了解该智能体的 `payload-json` 可接受哪些字段。创建智能体的流程见[智能体命令](./agent.md)。

---

## 检查适配器

这些只读命令用于回答“已安装什么，以及如何配置？”。它们不限定在某家公司中，而是描述整个实例的适配器注册表。

```sh
paperclipai adapter list
paperclipai adapter get <adapter-type>
paperclipai adapter config-schema <adapter-type>
paperclipai adapter ui-parser <adapter-type>
```

| 命令 | 返回内容 |
|---|---|
| `adapter list` | 所有已注册的内置和外部适配器。从这里开始查找有效类型。 |
| `adapter get <type>` | 单个适配器的完整记录，包括当前设置、安装来源和状态。 |
| `adapter config-schema <type>` | 该适配器配置的 JSON 架构。编写智能体或适配器的 `payload-json` 前先读取，以了解允许的字段。 |
| `adapter ui-parser <type>` | 适配器的 UI 解析器 JavaScript，即界面用于呈现适配器运行输出的客户端代码。排查运行结果的显示问题时很有用。 |

```sh
# Discover what's installed, then read one adapter's schema
paperclipai adapter list --json
paperclipai adapter config-schema codex_local --json
```

> **提示：** 将 `config-schema` 与 `--json` 搭配使用，并将输出传给你的工具链。此架构就是传给 `adapter update`、`adapter override` 和 `agent create` 的 `--payload-json` 格式约定。

---

## 列出公司的模型

`adapter models` 是唯一按公司限定范围的检查命令，因为可用模型取决于公司的服务提供商凭据和配置。必须提供公司上下文，可以从所选配置文件读取，也可以明确传入 `--company-id`。

```sh
paperclipai adapter models <adapter-type> --company-id <company-id>
paperclipai adapter models codex_local --company-id <company-id> --refresh
paperclipai adapter models codex_local --company-id <company-id> --environment-id <environment-id>
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 要查询其模型列表的公司。除非上下文配置文件中已设置公司，否则必填。 |
| `--refresh` | 重新获取服务提供商的模型列表，而不是返回缓存。轮换密钥后，或未显示新发布的模型时使用。默认关闭。 |
| `--environment-id <id>` | 对支持环境感知的适配器，将查询限定到指定环境。 |

另有一个相关的公司范围只读命令：

```sh
paperclipai adapter detect-model <adapter-type> --company-id <company-id>
```

`detect-model` 会让适配器解析其实际会使用的模型。它接受 `-C, --company-id <id>`，并且需要公司上下文。

---

## 安装和移除外部适配器

外部适配器以软件包形式发布。`install` 会向运行时注册适配器；`delete` 会移除此注册。无法删除内置适配器；如需暂停内置适配器，请改用下文的 `override`。

```sh
paperclipai adapter install --payload-json '{"packageName":"@scope/adapter","version":"1.2.3"}'
paperclipai adapter delete <adapter-type>
```

| 命令 | 说明 |
|---|---|
| `adapter install` | 必须通过 `--payload-json` 描述要注册的软件包（例如 `packageName` 和 `version`）。 |
| `adapter delete <type>` | 移除**外部**适配器注册。 |

> **警告：** `adapter delete` 会从实例中移除适配器。任何 `adapterType` 指向该类型的智能体都将失去可用运行时。删除前请确认没有其他内容依赖该类型——检查每家公司的 `agent list`。

---

## 配置和控制适配器

这些命令会更改适配器状态。`update` 和 `override` 通过 `--payload-json` 发送 JSON 补丁（必填）。`reload` 和 `reinstall` 的 `--payload-json` 可选，默认值为 `{}`。

```sh
paperclipai adapter update <adapter-type> --payload-json '{"disabled":true}'
paperclipai adapter override <adapter-type> --payload-json '{"paused":true}'
paperclipai adapter reload <adapter-type>
paperclipai adapter reinstall <adapter-type>
```

| 命令 | 用途 | 负载 |
|---|---|---|
| `adapter update <type>` | 更新适配器设置。请先根据 `config-schema` 验证补丁。 | 必须传入 `--payload-json` |
| `adapter override <type>` | 暂停或恢复**内置**适配器。这是在不删除内置适配器的情况下禁用它的方式。 | 必须传入 `--payload-json` |
| `adapter reload <type>` | 原位重新加载适配器，重新读取其配置而不重新安装。 | `--payload-json` 可选，默认值为 `{}` |
| `adapter reinstall <type>` | 从其来源重新安装适配器。适用于 `reload` 不够、需要全新安装的情况。 | `--payload-json` 可选，默认值为 `{}` |

如果要依赖某个适配器，建议先使用公司范围的 `test-environment` 验证其环境配置：

```sh
paperclipai adapter test-environment <adapter-type> --company-id <company-id> --payload-json '{}'
```

`test-environment` 需要公司上下文（`-C, --company-id <id>`），并接受可选的 `--payload-json`（默认值为 `{}`），用于提供待验证的环境配置。

---

## 通用选项

每个 `adapter` 子命令都接受标准客户端选项：`--data-dir <path>`、`--api-base <url>`、`--api-key <token>`、`--context <path>`、`--profile <name>` 和 `--json`。按公司限定范围的子命令（`models`、`detect-model` 和 `test-environment`）还接受 `-C, --company-id <id>`，并且必须能解析出公司。API 地址和上下文的解析方式见[通用选项](./common-options.md)；`--json` 的使用方式见[输出和脚本](./output-and-scripting.md)。

> **注意：** 适配器管理属于控制面板操作员职责。智能体范围的凭据只绑定到一个公司和一个智能体，不适用于安装或覆盖整个实例的适配器。

---

## 另请参阅

- [智能体命令](./agent.md)——使用在此处读取的架构设置智能体的 `adapterType` 和 `payload-json`。
- [工作在哪儿运行](../../guides/welcome/key-concepts.md)——为什么适配器在服务器端执行，而 CLI 只负责触发和观察。
- [通用选项](./common-options.md)——共享客户端标志和 API 地址解析。
- [输出和脚本](./output-and-scripting.md)——处理 `--json` 输出。
