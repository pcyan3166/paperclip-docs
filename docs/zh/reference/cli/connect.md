---
paperclip_version: v2026.609.0
seo_title: connect 命令
seo_description: 在新终端中快速连接到正在运行的 Paperclip 实例，通过引导流程完成 CLI 配置。
---

# connect 命令

在新终端中使用 Paperclip CLI 连接正在运行的实例时，`connect` 是最简单的入口。它会通过简短的交互询问服务器、身份类型、公司和智能体；完成后会生成令牌、保存可重复使用的配置文件，并输出所需的环境变量导出命令。你不必预先知道任何 ID，也不必手动编写上下文文件；向导会发现可用选项并让你选择。

首次设置 CLI、想要新增保存的配置文件（例如连接第二个实例、另一家公司或新智能体）时，或想通过交互选择而不是手动输入 `context set` 和 `auth login` 命令时，都可以运行 `paperclipai connect`。

> **注意：** `connect` 有意设计为交互式命令。标准输入和输出都必须连接到真实终端（TTY）。在脚本和 CI 中运行时，它会拒绝执行，并提示传入 `--api-base` / `--api-key`，或改用 `context` 和令牌命令。非交互式配置方式请参阅[通用选项](./common-options.md)。

---

## 完整执行流程

`connect` 是一个向导，而不是一组子命令。运行一次命令并回答几个提示即可：

```sh
paperclipai connect
```

其内部执行步骤如下：

1. **确认 API 基础地址。** 预填按常规解析顺序得到的 API 地址（解析顺序见[通用选项](./common-options.md)），供你确认或修改。占位地址为 `http://localhost:3100`。
2. **检查服务器健康状态。** 对所选地址调用 `GET /api/health`；如果实例无法访问，向导会提前停止，使你立即发现连接问题，而不是执行三条命令后才发现。
3. **以控制面板操作员身份登录。** 运行交互式控制面板登录以获取控制面板凭据，并用其查找你有权查看的公司。
4. **选择身份类型。** 选择此配置文件以**控制面板操作员**身份连接，还是作为**公司中的智能体**连接。
5. **为配置文件命名。** 向导会建议当前配置文件名称（或 `default`），你可以保留或更改。
6. **选择公司（和智能体）。** 控制面板配置文件可以选择固定一个默认公司；智能体配置文件则必须选择公司，然后再选择该公司中的一个智能体。
7. **生成令牌、保存配置并输出环境变量命令。** 向导会创建正确类型的令牌，将包含身份类型信息的配置文件写入上下文文件，将其设为当前配置文件，并打印可粘贴到 shell 中的环境变量命令，使新凭据立即可用。

---

## 两种身份类型

影响其余所有设置的关键选择是**身份类型**。可以传入 `--persona board` 或 `--persona agent` 跳过询问；否则向导会提示选择。

| 身份类型 | 创建内容 | 公司和智能体 |
|---|---|---|
| `board` | 作用于整个实例的控制面板 API 令牌。 | 公司为**可选项**——可以为配置文件固定一个默认公司，也可以选择 `(none)`。不绑定智能体。 |
| `agent` | 作用于一个公司和一个智能体的智能体 API 密钥。 | **必须**选择公司，然后选择该公司中的一个智能体。该公司至少要有一个智能体。 |

控制面板配置文件在整个实例范围内拥有控制面板权限；智能体配置文件则精确绑定到一个公司和一个智能体。这与[上下文配置文件](./common-options.md)中描述的身份类型划分相同。各类凭据如何生成及其功能，请参阅[身份验证](./authentication.md)。

```sh
# Connect straight as a board operator, skipping the persona prompt
paperclipai connect --persona board

# Connect as an agent (you'll still pick the company and agent interactively)
paperclipai connect --persona agent
```

---

## 为令牌和环境变量命名

可以通过两个标志控制生成凭据的标签和存储方式，无需接受默认值。

```sh
paperclipai connect --token-name "laptop-board" --api-key-env-var-name PAPERCLIP_API_KEY
```

| 标志 | 默认值 | 用途 |
|---|---|---|
| `--persona <persona>` | _在提示中选择_ | 要配置的身份类型：`board` 或 `agent`。传入此参数可跳过身份类型询问。 |
| `--api-key-env-var-name <name>` | `PAPERCLIP_API_KEY` | 配置文件调用时用于读取密钥的环境变量名称。配置文件只保存此**名称**，绝不保存令牌本身。 |
| `--token-name <name>` | 带时间戳的标签 | 为创建的令牌设置标签。默认控制面板令牌名称为 `cli-board-<timestamp>`，智能体密钥名称为 `cli-agent-<timestamp>`。 |

> **提示：** 配置文件只记录环境变量的**名称**，不保存密钥。连接后请将令牌保留在环境变量中（打印的导出命令正是为此准备的），这样上下文文件便可安全分享。详情请参阅[通用选项](./common-options.md)中的 `apiKeyEnvVarName` 机制。

---

## 返回内容

向导结束时会确认连接（例如 `Connected profile '<name>' as board.` 或 `... as <agent name>.`），并返回可供阅读或脚本使用的结果。传入 `--json` 可获取机器可读格式。

结果包含配置文件名称、身份类型、解析出的 `apiBase`、`companyId`（智能体配置文件还包括 `agentId` / `agentName`）、新建的 `key`（包括 `id`、`name`、`createdAt`、`token`，控制面板令牌还包括 `expiresAt`），以及可直接粘贴的 **`exports`** 代码块：

```sh
export PAPERCLIP_API_URL='http://localhost:3100'
export PAPERCLIP_COMPANY_ID='<company-id>'
export PAPERCLIP_AGENT_ID='<agent-id>'
export PAPERCLIP_API_KEY='<token>'
```

仅在适用时才会显示 `PAPERCLIP_COMPANY_ID` 和 `PAPERCLIP_AGENT_ID`。最后一行的变量名称采用你通过 `--api-key-env-var-name` 设置的值。将代码块粘贴到 shell 后，CLI 的其他命令会立即使用新配置文件和凭据。

> **警告：** 明文 `token` 只会显示一次，位于向导输出和 `exports` 代码块中。请立即记录它，保存到密钥管理器或导出到环境中。已保存的配置文件只引用环境变量名称，不保存令牌，因此之后无法从配置文件中恢复令牌。

---

## 通用选项

`connect` 接受标准客户端选项：`--data-dir <path>`、`--api-base <url>`、`--api-key <token>`、`--context <path>`、`--profile <name>`、`-C, --company-id <id>` 和 `--json`。其中，`--api-base`、`--context`、`--profile` 和 `--company-id` 会为向导提供初始值（预填的 API 地址、写入的上下文文件、建议的配置文件名称和预选公司）。API 地址、上下文和公司作用范围的解析方式见[通用选项](./common-options.md)；`--json` 的使用方式见[输出和脚本](./output-and-scripting.md)。

> **注意：** `connect` 会在流程中执行交互式控制面板登录，因此传入 `--api-key` 不能使它变成非交互式命令。如果需要用脚本配置，请通过其他方式生成令牌，再使用 `context set` 配置文件；没有 TTY 时，`connect` 也会提示你采用此路径。

---

## 另请参阅

- [身份验证](./authentication.md)——控制面板令牌和智能体密钥的生成方式及其功能。
- [通用选项](./common-options.md)——共享客户端标志、上下文配置文件和 API 地址解析。
- [输出和脚本](./output-and-scripting.md)——处理 `--json` 输出。
- [概览](./overview.md)——CLI 在整体操作模式中的作用。
