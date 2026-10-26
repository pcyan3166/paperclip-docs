---
paperclip_version: v2026.609.0
seo_title: CLI 认证与令牌
seo_description: CLI 身份参考：它如何识别自身、连接哪个实例，以及该身份获准执行哪些操作。
---

# 认证与令牌

本文介绍 Paperclip CLI 的身份机制：CLI 如何知道*自己是谁*、*连接哪个实例*以及*获准执行哪些操作*。连接新机器、在看板操作员和代理角色之间切换、为无头自动化生成凭据，或轮换和撤销令牌时，请使用这些命令。简而言之：先通过交互式 `connect` 获取可用配置文件，之后用 `context`、`auth` 和 `token` 管理一切。

CLI 身份由两种**角色身份**和两种**凭据**构成。请先阅读下一节，后续内容都以你了解二者区别为前提。

---

## 角色身份：看板操作员与代理

CLI 会在每个配置文件中记录当前代表的*角色身份*。角色身份决定你拥有的权限范围。

| 角色身份 | 权限 | 用途 |
|---|---|---|
| **board** | 在整个实例中拥有完整看板权限，可访问所有公司、代理，并执行所有管理读取和写入操作。 | 自行运营公司、招募代理、创建其他令牌、执行设置和治理命令。 |
| **agent** | 限定于**一个公司**和**一个代理**。凭据只能代表该代理执行操作。 | 让单个代理（由人或本地运行的 AI 驱动）以自身身份通过 API 回报工作。 |

角色身份不只是标签，它对应不同的凭据类型。board 角色身份使用**看板 API 令牌**；agent 角色身份使用**代理 API 密钥**。不能将代理密钥扩权为看板权限；需要看板范围时，应生成看板令牌。

| 凭据 | 范围 | 生命周期 | 获取方式 |
|---|---|---|---|
| **看板 API 令牌** | 整个实例（看板权限）。 | 可命名、可选过期时间、服务器端可撤销、经过审计。 | 使用 `token board create`，或通过 `auth login` / `connect` 后的浏览器审批流程获取。 |
| **代理 API 密钥** | 单个公司和单个代理。 | 可命名，创建时明文仅显示**一次**，可撤销。 | 使用 `token agent create`，或在 `connect` 中选择 agent 角色身份。 |

> **警告：** 两类凭据的完整值都只在**创建时**显示。令牌字符串只会打印一次，之后无法再次查看。请立即将其存入秘密管理器或环境变量；系统没有“显示令牌”命令。

关于角色身份的概念模型及其与运营模式的关系，请参阅[角色身份与令牌](../../administration/cli-auth.md)。

---

## CLI 如何查找实例

每条客户端命令都按以下确切顺序解析 API 基础 URL，并在遇到第一个已设置的值后停止：

1. 命令上的 `--api-base <url>`。
2. `PAPERCLIP_API_URL` 环境变量。
3. 所选上下文配置文件的 `apiBase`。
4. 本地 Paperclip 配置中的服务器端口（连接本地实例时）。
5. `http://localhost:3100`。

连接失败时，错误消息会包含尝试连接的 URL，并提示检查该地址上的 `GET /api/health`。如果命令连接到了错误服务器，错误信息可以帮助你判断是哪一步的设置生效。

所有客户端命令共用的标志（`--data-dir`、`--api-base`、`--api-key`、`--context`、`--profile`、`--json`）请参阅[通用选项](./common-options.md)。

---

## 上下文配置文件

配置文件保存在 `~/.paperclip/context.json`。每个配置文件都带有**角色身份信息**，记录 API 基础 URL、可选默认公司、角色身份，以及（对代理配置文件而言）代理 ID 和名称。特别要注意，配置文件存储的是**`apiKeyEnvVarName`**——即保存令牌的环境变量*名称*，而非令牌本身。应将秘密保存在环境或秘密管理器中，并让配置文件引用它们。

### `context show`

显示解析后的当前上下文和活动配置文件。

```sh
paperclipai context show
paperclipai context show --profile staging --json
```

输出包括解析后的 `contextPath`、`currentProfile` 名称、解析后的配置文件，以及存储中的所有配置文件。

### `context list`

以行的形式列出所有配置文件，并标出当前使用的配置文件。

```sh
paperclipai context list
```

每行显示 `name`、`current`、`apiBase`、`companyId`、`persona`、`agentId`、`agentName` 和 `apiKeyEnvVarName`。

### `context use`

切换活动配置文件。

```sh
paperclipai context use staging
```

### `context set`

逐项创建或更新配置文件。未指定 `--profile` 时，会编辑当前配置文件（如果没有当前配置，则使用 `default`）。添加 `--use` 可同时将其设为活动配置文件。

```sh
# Configure a board profile that reads its token from PAPERCLIP_API_KEY
paperclipai context set \
  --profile prod \
  --api-base https://paperclip.example.com \
  --persona board \
  --api-key-env-var-name PAPERCLIP_API_KEY \
  --use

# Pin an agent profile to a company and agent
paperclipai context set \
  --profile acme-bot \
  --api-base https://paperclip.example.com \
  --company-id <company-id> \
  --persona agent \
  --agent-id <agent-id> \
  --agent-name "Acme Bot"
```

| 标志 | 用途 |
|---|---|
| `--profile <name>` | 要编辑的配置文件（默认：当前配置文件；若无则为 `default`）。 |
| `--api-base <url>` | 配置文件默认使用的 API 基础 URL。 |
| `--company-id <id>` | 默认公司 ID。 |
| `--persona <persona>` | 可选值为 `board` 或 `agent`；其他值会被拒绝。 |
| `--agent-id <id>` | 默认代理 ID（用于 agent 角色身份）。 |
| `--agent-name <name>` | 代理的显示名称。 |
| `--api-key-env-var-name <name>` | 保存令牌的环境变量名称。建议使用此方式，而不要直接填入秘密。 |
| `--use` | 保存后将此配置文件设为活动配置文件。 |

> **提示：** 无法运行 `connect` 向导的无头机器上，可通过 `context set` 以非交互方式创建配置文件。它写入的角色身份配置结构与 `connect` 生成的相同。

---

## `connect` 向导

交互式 `connect` 是最快的可用配置方式。它**仅支持 TTY**——没有交互式终端时运行会失败，并提示改用 `--api-base` / `--api-key` 或 `context set` / `token` 命令。

```sh
paperclipai connect
paperclipai connect --persona agent --token-name laptop-cli
```

向导会依次完成以下步骤：

1. **解析 API 基础 URL**——根据上述解析顺序预填地址，由你确认或修改。
2. **健康检查**——调用 `GET <api-base>/api/health`；实例不可访问时停止。
3. **看板登录**——执行看板审批流程（见下文），以便读取公司信息并生成令牌。
4. **选择角色身份**——看板操作员，或某个公司的代理。
5. **选择公司**（如果选择代理，还要选择该公司中的代理）。
6. **生成令牌**——看板配置文件生成看板 API 密钥；代理配置文件生成代理 API 密钥。
7. **保存角色身份配置文件**并将其设为活动配置文件。
8. **打印 shell 环境变量导出语句**，以便立即使用新凭据。

| 标志 | 用途 |
|---|---|
| `--persona <persona>` | 跳过选择提示；可选 `board` 或 `agent`。 |
| `--api-key-env-var-name <name>` | 写入配置文件的环境变量名称（默认 `PAPERCLIP_API_KEY`）。 |
| `--token-name <name>` | 新令牌标签（默认使用带时间戳的 `cli-board-…` / `cli-agent-…` 名称）。 |

打印的导出语句如下所示（以 agent 角色身份为例）：

```sh
export PAPERCLIP_API_URL='https://paperclip.example.com'
export PAPERCLIP_COMPANY_ID='<company-id>'
export PAPERCLIP_AGENT_ID='<agent-id>'
export PAPERCLIP_API_KEY='<token-shown-once>'
```

board 配置文件不会设置 `PAPERCLIP_AGENT_ID`；除非指定了默认公司，否则也不会设置 `PAPERCLIP_COMPANY_ID`。请将这些语句复制到 shell 或秘密管理器中——令牌只会在此处显示一次。

> **注意：** 配置文件存储的是环境变量*名称*（`apiKeyEnvVarName`），而非令牌。导出语句会将实际令牌写入该环境变量。二者配合后，后续命令即可完成认证，同时秘密不会进入 `context.json`。

---

## 看板认证：`auth`

`auth` 命令管理某个 API 基础 URL 对应的看板用户凭据。凭据是存储在本地的看板 API 令牌，按规范化后的 `apiBase` 为键，因此同一台机器可以同时保存本地、预发布和生产环境的不同凭据。

### `auth login`

通过浏览器审批（设备码式）流程，为 CLI 验证看板用户权限。CLI 会创建验证请求，在浏览器中打开审批 URL，并持续轮询，直到你批准、取消或请求过期。批准后，CLI 会通过 `/api/cli-auth/me` 确认身份，并为该 `apiBase` 保存看板令牌。

```sh
paperclipai auth login
paperclipai auth login --instance-admin
paperclipai auth login --company-id <company-id>
paperclipai auth login --api-base https://paperclip.example.com
paperclipai auth login --no-browser
```

| 标志 | 用途 |
|---|---|
| `--instance-admin` | 申请实例管理员权限，而非普通看板权限。审批者本人必须是实例管理员。 |
| `--company-id <id>` | 将申请的访问范围限定到指定公司。 |
| `--no-browser` | 不尝试打开浏览器，只打印审批 URL，供你手动打开。适用于无头主机或 SSH 会话。 |

如果你在浏览器中取消，CLI 会以 `CLI auth challenge was cancelled.` 退出；等待时间过长时则显示 `CLI auth challenge expired before approval.`。两种情况下都可以重新运行命令。

### `auth whoami`

显示当前 API 基础 URL 对应的看板用户身份。调用 `/api/cli-auth/me`。

```sh
paperclipai auth whoami --json
```

结果会报告 `user`（ID、姓名、邮箱）、`userId`、`isInstanceAdmin`、你可以访问的 `companyIds`、凭据 `source` 和 `keyId`。

### `auth logout`

移除此 API 基础 URL 对应的已存看板凭据。命令会先尝试在服务器端撤销已存令牌，然后删除本地记录。如果没有已存凭据，则会报告 `revoked: false` 并正常退出。

```sh
paperclipai auth logout
```

> **注意：** 如果服务器端撤销失败（网络错误或令牌已失效），本地凭据仍会被删除，因此不会留下过期记录。要轮换凭据，可先执行 `logout`，再执行 `login`。

### `auth revoke-current`

在服务器端撤销当前正在使用的看板 API 令牌，但不改动本地存储。需要从服务器侧使当前令牌失效时可用。

```sh
paperclipai auth revoke-current
```

### `auth challenge`

低级接口，用于访问 CLI 认证请求的生命周期，便于编写自定义审批流程或自动化脚本。大多数用户不需要直接调用这些命令——`auth login` 会为你完成整个流程。

```sh
# Create a challenge from a JSON payload
paperclipai auth challenge create --payload-json '{"command":"ci-bot","requestedAccess":"board"}'

# Inspect, approve, or cancel a challenge (the secret is required)
paperclipai auth challenge get <challenge-id> --token <secret>
paperclipai auth challenge approve <challenge-id> --token-env CHALLENGE_TOKEN
paperclipai auth challenge cancel <challenge-id> --token <secret>
```

| 子命令 | 用途 |
|---|---|
| `create --payload-json <json>` | 根据 `CreateCliAuthChallenge` JSON 请求体创建验证请求（必填）。 |
| `get <id>` | 读取验证请求状态。 |
| `approve <id>` | 批准待处理的验证请求。 |
| `cancel <id>` | 取消待处理的验证请求。 |

对 `get`、`approve` 和 `cancel`，必须使用 `--token <secret>` 或 `--token-env <name>` 提供验证请求的秘密令牌（从指定环境变量读取）。如果两者都未提供，命令会报错。

---

## 具名令牌：`token`

`auth login` 用于交互式获取并在本地保存看板凭据；`token` 则用于生成**具名、可管理**的凭据，可交给自动化、CI 或特定代理使用。看板令牌和代理密钥分别通过不同子命令管理。

### 看板令牌

看板 API 密钥拥有完整看板权限。密钥可命名、可设置过期时间、可撤销，并在服务器端接受审计。创建看板令牌需要已有看板凭据（可通过向导或 `auth login` 获取）。

```sh
# Long-lived board token for an external admin tool
paperclipai token board create --name external-admin

# Short-lived token that expires in 7 days
paperclipai token board create --name short-lived --ttl-days 7

# Explicit expiry timestamp, with audit company context
paperclipai token board create --name release-bot --company-id <company-id> --expires-at 2026-12-31T00:00:00Z

# Non-expiring token (be deliberate about this)
paperclipai token board create --name forever --never-expires

paperclipai token board list
paperclipai token board revoke <key-id>
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 记录为该密钥审计上下文的公司 ID。 |
| `--name <name>` | 密钥标签（默认 `cli-board`）。 |
| `--expires-at <iso8601>` | 明确指定过期时间。 |
| `--ttl-days <days>` | 从当前时间起，在指定天数后过期。 |
| `--never-expires` | 创建永不过期的密钥。 |

`--never-expires` 的优先级高于其他过期标志；否则由 `--expires-at` 和 `--ttl-days` 设置有效期。如果都未指定，则使用服务器默认值。`token board list` 会显示当前看板用户每个密钥的 `id`、`name`、`createdAt`、`lastUsedAt`、`expiresAt` 和 `revokedAt`。

### 代理密钥

代理 API 密钥限定在单个公司和单个代理。代理可通过 ID、短名称或公司范围内唯一的名称解析。必须提供 `--company-id` 和 `--agent`。

```sh
paperclipai token agent create --company-id <company-id> --agent <agent-id-or-name> --name external-worker
paperclipai token agent list   --company-id <company-id> --agent <agent-id-or-name>
paperclipai token agent revoke --company-id <company-id> --agent <agent-id-or-name> <key-id>
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 公司 ID（必填）。 |
| `--agent <agent>` | 代理 ID、短名称或公司范围内唯一的名称（必填）。 |
| `--name <name>` | 密钥标签，仅用于 `create`（默认 `cli-agent`）。 |

创建后，响应中包含密钥的 `token`，且仅显示一次。`token agent list` 会列出每个密钥的 `id`、`name`、`createdAt` 和 `revokedAt`。

> **提示：** 如果需要完整的本地设置流程，以便将 Claude 或 Codex 作为代理运行——包括长期有效的代理密钥、安装到 `~/.codex/skills` 和 `~/.claude/skills` 的技能，以及可直接粘贴的环境变量导出语句——请使用 `agent local-cli <agent>`，而不是手动生成裸密钥。参阅[代理](./agent.md)。

---

## 综合示例

在新机器上通常按以下顺序操作：

```sh
# 1. One interactive setup creates a profile and prints exports
paperclipai connect

# 2. Verify who you are
paperclipai auth whoami

# 3. Mint a named token for CI (board scope) or an agent
paperclipai token board create --name ci --ttl-days 30
paperclipai token agent create --company-id <company-id> --agent <agent-id> --name worker

# 4. Switch between saved profiles as needed
paperclipai context use prod
```

无法运行 `connect` 的无头服务器上，可将步骤 1 替换为 `auth login`（在新建且已完成身份验证的实例上，也可使用 `auth bootstrap-ceo` 或看板认领），再运行 `context set`，之后步骤相同。

---

## 另请参阅

- [通用选项](./common-options.md)——每条客户端命令共用的标志，包括 API 基础 URL 的解析方式。
- [角色身份与令牌](../../administration/cli-auth.md)——board 与 agent 身份的概念模型。
- [代理](./agent.md)——`agent local-cli` 和代理管理。
- [设置命令](./setup-commands.md)——引导和配置实例。
- [安装与连接](../../guides/getting-started/installation.md)——首次运行的引导流程。
- [无头引导](./installation.md)——无浏览器时生成首个凭据。
