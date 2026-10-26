---
paperclip_version: v2026.609.0
seo_title: CLI 令牌命令
seo_description: 创建、列出和撤销 API 密钥，使智能体、CI 任务和控制面板操作员能从浏览器以外访问 Paperclip 实例。
---

# 令牌命令

当你需要创建、列出或撤销 API 密钥，以便智能体和控制面板操作员从浏览器之外连接 Paperclip 实例时，可使用这些命令，例如 CI 任务、脚本、无界面智能体运行器或自己的终端。`token` 命令组分为两部分：`token agent` 用于管理单个智能体在实例上操作时使用的密钥；`token board` 用于管理控制面板操作员身份在公司和控制平面上操作时使用的密钥。

此处的每个命令都接受[通用客户端选项](./common-options.md)（`--data-dir`、`--api-base`、`--api-key`、`--context`、`--profile`、`--json`）。`agent` 子命令按公司限定范围，必须提供 `-C, --company-id <id>`；`board` 命令中的 `--company-id` 是可选项，只会为创建的密钥设置审计上下文。

> **警告：** 创建令牌时，密钥值 `token` 只会返回一次。Paperclip 不会以可供读取的形式保存它——`token agent list` 和 `token board list` 只返回元数据（ID、名称、时间戳），不会返回令牌。创建密钥时请立即保存其值，并将其视为凭据：持有该值的任何人都能以相应智能体或控制面板用户的身份执行操作。

---

## 各命令的用途

| 命令 | 用途 |
|---|---|
| `token agent create` | 为指定智能体创建新的 API 密钥。 |
| `token agent list` | 列出某个智能体已有的 API 密钥（仅元数据）。 |
| `token agent revoke` | 根据 ID 撤销某个智能体的 API 密钥。 |
| `token board create` | 为你的控制面板操作员身份创建带名称的 API 密钥。 |
| `token board list` | 列出当前控制面板用户的 API 密钥。 |
| `token board revoke` | 根据 ID 撤销控制面板 API 密钥。 |

---

## 智能体 API 密钥

智能体 API 密钥是智能体在实例上执行操作时出示的凭据，与加入请求获批后生成的密钥类型相同。如果你要自行配置智能体运行器，并希望直接创建、审计或废弃其密钥，请使用 `token agent`。

每条 `token agent` 命令都限定在某家公司中，并通过 `--agent` 指定智能体。此参数可接受智能体 ID、短名称或明确无歧义的名称。CLI 会先在该公司内将其解析为具体智能体，再执行其他操作；拼写错误或指定了错误公司的智能体时，会立即报错 `Agent not found`。

### 创建智能体密钥

`token agent create` 会为指定智能体创建新密钥，并打印智能体、所属公司和新建的 `key`，包括仅显示一次的 `token` 值。

```sh
paperclipai token agent create --company-id <company-id> --agent <agent> --name ci-runner
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 必填。智能体所属公司。 |
| `--agent <agent>` | 必填。智能体 ID、短名称或明确无歧义的名称。 |
| `--name <name>` | API 密钥标签。默认为 `cli-agent`。 |

> **提示：** 请立即从输出中保存 `token`，之后不会再显示。为每个密钥设置易于辨认的 `--name`（例如运行它的主机或流水线），以便之后运行 `token agent list` 时知道各密钥的用途。

### 列出智能体密钥

`token agent list` 返回为某个智能体定义的密钥。人类可读输出会显示各密钥的 `id`、`name`、`createdAt` 和 `revokedAt`，但不会显示令牌值。

```sh
paperclipai token agent list --company-id <company-id> --agent <agent>
paperclipai token agent list --company-id <company-id> --agent <agent> --json
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 必填。智能体所属公司。 |
| `--agent <agent>` | 必填。智能体 ID、短名称或明确无歧义的名称。 |

### 撤销智能体密钥

`token agent revoke` 将密钥 ID 作为位置参数，并撤销对应密钥。密钥撤销后将无法再用于身份验证。

```sh
paperclipai token agent revoke <keyId> --company-id <company-id> --agent <agent>
```

| 参数 / 标志 | 用途 |
|---|---|
| `<keyId>` | 必填。要撤销的智能体 API 密钥 ID。 |
| `-C, --company-id <id>` | 必填。智能体所属公司。 |
| `--agent <agent>` | 必填。智能体 ID、短名称或明确无歧义的名称。 |

---

## 控制面板 API 密钥

控制面板 API 密钥是控制面板操作员身份用于通过脚本操作实例的凭据，相当于以无界面方式登录控制面板。使用 `token board` 可以创建长期密钥用于自动化、查看当前控制面板用户有哪些密钥，并在不再需要时撤销密钥。

与智能体命令不同，`token board` 不要求指定公司。可选的 `-C, --company-id <id>` 只会为所创建的密钥记录审计上下文中的公司。

### 创建控制面板密钥

`token board create` 会为当前控制面板用户创建带名称的密钥，并打印新建的 `key`，包括仅显示一次的 `token`。默认情况下，控制面板密钥沿用服务器的过期策略；你可以使用下列过期标志中的**一个**来控制有效期。

```sh
# A key that expires 90 days from now
paperclipai token board create --name ci-deploy --ttl-days 90

# A key with an explicit expiry timestamp
paperclipai token board create --name nightly --expires-at 2026-12-31T00:00:00Z

# A non-expiring key
paperclipai token board create --name long-lived --never-expires
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 可选。为审计上下文记录的公司 ID。 |
| `--name <name>` | API 密钥标签。默认为 `cli-board`。 |
| `--expires-at <iso8601>` | 过期时间戳。日期无效时会被拒绝。 |
| `--ttl-days <days>` | 从现在起多少天后过期。必须是正数。 |
| `--never-expires` | 创建永不过期的密钥。 |

> **注意：** 这些过期标志互为替代项，优先级依次为 `--never-expires`、`--expires-at`、`--ttl-days`。如果一个都不传，密钥会采用服务器默认过期策略。

### 列出控制面板密钥

`token board list` 返回当前控制面板用户的 API 密钥。人类可读输出只显示各密钥的 `id`、`name`、`createdAt`、`lastUsedAt`、`expiresAt` 和 `revokedAt` 等元数据。

```sh
paperclipai token board list
paperclipai token board list --json
```

### 撤销控制面板密钥

`token board revoke` 将密钥 ID 作为位置参数，并撤销对应密钥。

```sh
paperclipai token board revoke <keyId>
```

| 参数 | 用途 |
|---|---|
| `<keyId>` | 必填。要撤销的控制面板 API 密钥 ID。 |

---

## 另请参阅

- [通用选项](./common-options.md)——所有令牌命令都接受的共享客户端标志。
- [身份验证](./authentication.md)——`connect`、`auth login`、控制面板认领及更广泛的凭据初始化流程。
- [访问、配置文件和实例管理](./access.md)——加入请求（获批后会创建智能体 API 密钥）以及用于确认密钥对应身份的 `whoami`。
- [智能体命令](./agent.md)——创建和配置要为其创建密钥的智能体。
- [输出和脚本](./output-and-scripting.md)——使用 `--json` 在脚本中捕获刚生成的令牌。
