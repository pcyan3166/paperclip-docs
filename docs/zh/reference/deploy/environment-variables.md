---
paperclip_version: v2026.1023.0
seo_title: 环境变量参考
seo_description: Paperclip 用于服务器配置的全部环境变量，以及运行时注入智能体进程的变量。部署接线时请查阅本列表。
---

# 环境变量

本页列出 Paperclip 读取的服务器配置环境变量，以及它在运行时注入智能体进程的环境变量。

配置部署、排查启动问题，或检查适配器进程环境可见内容时，请使用本页。

---

## 服务器配置

| 变量 | 默认值 | 含义 |
|---|---|---|
| `PORT` | `3100` | 服务器端口。 |
| `HOST` | `127.0.0.1` | 服务器绑定的主机地址。 |
| `DATABASE_URL` | 内嵌 PostgreSQL | PostgreSQL 连接字符串。 |
| `DATABASE_MIGRATION_URL` | 回退到 `DATABASE_URL` | 可选的 PostgreSQL URL，仅用于运行迁移。当运行时用户没有 DDL 权限、需要单独角色修改 schema 时很有用。 |
| `PAPERCLIP_HOME` | `~/.paperclip` | 所有 Paperclip 数据的基础目录。 |
| `PAPERCLIP_INSTANCE_ID` | `default` | 多个本地实例的实例标识符。 |
| `PAPERCLIP_DEPLOYMENT_MODE` | `local_trusted` | 运行模式覆盖值。 |
| `SERVE_UI` | `true`（来自 `config.json` 中的 `server.serveUi`） | 设置后会覆盖文件配置中的开关，用于控制服务器是否提供内置 UI。`SERVE_UI=true` 表示启用；`SERVE_UI=false` 表示禁用。 |
| `PAPERCLIP_BIND` | 根据 `HOST` 推断 | 服务器套接字的绑定模式。必须是 `BIND_MODES` 中的值之一（见 `packages/shared`）；会覆盖 `config.json` 中的 `server.bind`。 |
| `PAPERCLIP_BIND_HOST` | 自动推断 | `PAPERCLIP_BIND` 为自定义模式时使用的主机；会覆盖 `server.customBindHost`。 |
| `PAPERCLIP_TAILNET_BIND_HOST` | 通过 `tailscale ip -4` 自动检测 | 绑定模式为 `tailnet` 时服务器绑定的 Tailnet IPv4 地址。明确设置此项可跳过对 `tailscale` CLI 的探测。 |
| `PAPERCLIP_WORKSPACE_GIT_SCAN_CONCURRENCY` | `2` | 同时运行的高开销工作区完整 Git 树扫描数量。限制在 1–16 之间。 |
| `PAPERCLIP_WORKSPACE_GIT_SCAN_QUEUE_CAPACITY` | `32` | 新扫描被拒绝前可在队列中等待的扫描数量。限制在 0–1024 之间。 |
| `PAPERCLIP_WORKSPACE_GIT_SCAN_TIMEOUT_MS` | `8000` | 快速工作区 Git 扫描（变更文件浏览器和干净状态检查）的超时时间，单位为毫秒；超过后扫描会被放弃。限制在 100–120000 之间。工作区快照改用 `PAPERCLIP_WORKSPACE_GIT_SNAPSHOT_TIMEOUT_MS`。 |
| `PAPERCLIP_WORKSPACE_GIT_SCAN_CACHE_TTL_MS` | `10000` | 重新扫描前复用已完成扫描结果的时长，单位为毫秒。限制在 0–60000 之间。 |
| `PAPERCLIP_WORKSPACE_GIT_SNAPSHOT_TIMEOUT_MS` | `1800000`（30 分钟） | 工作区快照允许花费的最长时间，单位为毫秒；超时后会放弃。沙盒运行前，快照会列出工作区中已更改、未跟踪和已忽略的文件；截止时间也包括在慢速磁盘上等待的时间。对于很大的目录树或较慢的存储可以调高。小于 `1000` 的值会回退为默认值，大于 `86400000`（24 小时）的值会被限制为该上限。 |
| `PAPERCLIP_WORKSPACE_MANIFEST_MIN_FREE_BYTES` | `268435456`（256 MiB） | 工作区快照始终为主机预留的磁盘空间，单位为字节。快照会将文件列表写入临时磁盘清单；如果会占用这部分预留空间，就会报错停止。小于 `67108864`（64 MiB）的值会回退为默认值。 |
| `PAPERCLIP_RUNNER_API_COMPANY_CAPTURE_MAX_BYTES` | `21474836480`（20 GiB） | 单个公司中 Paperclip Runner 智能体可保存的大型 API 响应总字节数。智能体 API 调用的响应超过单次回复可容纳的大小时，响应会保存为公司资源，供智能体分页读取；此上限限制这些响应累计占用的空间。删除资源后会释放空间。小于 `1073741824`（1 GiB）的值会回退为默认值。 |
| `PAPERCLIP_RUNNER_API_TOOLS_ENABLED` | 开启（未设置） | 是否为 Paperclip Runner 智能体提供广泛的 Paperclip API 工具（`search_api`、`call_api` 和 `hire_agent`）。默认开启；设置为 `true` 以外的任何值，可为所有公司关闭这些工具。 |
| `PAPERCLIP_RUNNER_API_TOOLS_COMPANY_IDS` | 未设置（所有公司） | 以逗号分隔的公司 ID。设置后，只有这些公司的 Runner 智能体会获得 API 工具。此项只能收窄范围；如果 `PAPERCLIP_RUNNER_API_TOOLS_ENABLED` 已关闭工具，则此项不起作用。 |
| `PAPERCLIP_WORKSPACE_REAPER_COOLDOWN_DAYS` | `7` | 终态工作区清理器在任务树进入终态后，等待多少天再归档其工作区。在此窗口期内仍可重新打开工作。`0` 会禁用冷却时间并立即清理；负值或非数字值会回退为默认值。 |
| `PAPERCLIP_DB_BACKUP_IDLE_CHECKPOINT_ENABLED` | 关闭 | 在启用自动数据库备份的实例上设为 `1`，即可允许实例空闲时休眠：服务器会在确认实例空闲前执行并验证最新备份；备份失败时拒绝休眠。休眠期间须保持数据库和备份卷处于挂载状态。 |
| `PAPERCLIP_ISSUE_PRIVACY_MODE` | `enforce` | 私有任务和项目的强制执行方式。`enforce` 会对无访问权限的人隐藏内容；`shadow` 只记录本应拒绝的读取；`off` 则跳过检查。只有在诊断逐步上线过程时才使用后两者，绝不要在必须保密私有任务时使用。 |

> **注意：** `DATABASE_URL` 是在内嵌数据库和外部 PostgreSQL 之间切换的主要开关。

---

## 部署与身份验证

从默认本地安装迁移后，以下变量最为重要。

| 变量 | 含义 |
|---|---|
| `PAPERCLIP_PUBLIC_URL` | 邀请、重定向和身份验证来源配置使用的规范公网 URL。 |
| `PAPERCLIP_MCP_AUTHORIZATION_ORIGIN` | 公网 MCP 服务器浏览器页面可选的第二个 HTTPS 来源地址，供对外公布的授权端点、同意页面和设备验证页面使用。MCP 客户端的浏览器无法访问 `PAPERCLIP_PUBLIC_URL` 主机，但可以访问同一实例的另一个可信入口时，可使用此项。MCP 资源、签发者和 token 端点仍位于 `PAPERCLIP_PUBLIC_URL` 上。只有回环主机才接受纯 HTTP。 |
| `PAPERCLIP_AUTH_PUBLIC_BASE_URL` | 希望 Better Auth 使用固定公网来源地址时，明确指定身份验证基础 URL。 |
| `BETTER_AUTH_URL` | Better Auth 基础 URL 的备用输入。 |
| `BETTER_AUTH_SECRET` | Better Auth 会话和 token 的签名密钥。未设置时回退到 `PAPERCLIP_AGENT_JWT_SECRET`；两者都未配置时服务器会拒绝启动。本地开发的 `.env.example` 提供了 `paperclip-dev-secret`。 |
| `BETTER_AUTH_BASE_URL` | 某些部署使用的 Better Auth 基础 URL 备用输入。 |
| `BETTER_AUTH_TRUSTED_ORIGINS` | 以逗号分隔的可信身份验证来源地址允许列表。 |
| `PAPERCLIP_AGENT_JWT_SECRET` | 签发智能体 API JWT 的密钥。本地适配器身份验证需要此项。 |
| `PAPERCLIP_AGENT_JWT_TTL_SECONDS` | 智能体 JWT 的有效期，单位为秒。 |
| `PAPERCLIP_AGENT_JWT_ISSUER` | 智能体 JWT 的签发者。 |
| `PAPERCLIP_AGENT_JWT_AUDIENCE` | 智能体 JWT 的受众。 |
| `PAPERCLIP_TOOL_ACTION_SIGNING_SECRET` | 工具操作审批的签名密钥。它没有 `BETTER_AUTH_SECRET` 那样的回退值；未设置时，服务器无法签发已签名的工具操作审批，并会抛出 `ToolActionSigningSecretMissingError`。请在实例自己的环境中设置；工作树会从 `.paperclip/.env` 继承此项。本地开发的 `.env.example` 提供了 `paperclip-dev-tool-action-signing-secret-change-me`。 |

相关部署变量：

| 变量 | 含义 |
|---|---|
| `PAPERCLIP_DEPLOYMENT_EXPOSURE` | 暴露策略覆盖值；在需要身份验证的模式下通常为 `private` 或 `public`。 |
| `PAPERCLIP_AUTH_BASE_URL_MODE` | 基础 URL 处理模式，例如 `auto` 或 `explicit`。 |
| `PAPERCLIP_ALLOWED_HOSTNAMES` | 用于已验证身份/私有主机验证的逗号分隔主机名允许列表。 |
| `TRUST_PROXY` | 服务器位于反向代理或负载均衡器后方时，对 `X-Forwarded-For` 请求头的信任程度。默认未设置（完全不信任）。详见下文。 |

> **提示：** 如果 `paperclipai doctor` 报告主机名、重定向或身份验证来源地址问题，请先检查这一组变量。

### 信任反向代理（`TRUST_PROXY`）

Paperclip 运行在负载均衡器或反向代理（nginx、Caddy 或云负载均衡器）后方时，真实客户端 IP 会通过 `X-Forwarded-For` 请求头传入，而不是来自套接字连接。`TRUST_PROXY` 用来告知服务器应该信任此请求头中的多少层转发地址，以便 `req.ip` 和速率限制使用真实客户端地址，而不是代理地址。

默认值为**未设置**，也就是完全不信任。这样最安全，因为否则不可信客户端可以发送自己的 `X-Forwarded-For` 来伪造地址。只有服务器前方确实存在代理时，才应启用此选项。

接受的值：

| 值 | 含义 |
|---|---|
| 未设置、`""`、`false`、`0` | 完全不信任。默认值。 |
| `true` | 无条件信任请求头。除非服务器只能通过代理访问，否则**不安全**。 |
| 正整数（例如 `1`） | 信任指定数量的代理跳数。服务器前方只有一个负载均衡器时使用 `1`。 |
| 逗号分隔列表 | 按子网名称（`loopback`、`linklocal`、`uniquelocal`）或 CIDR（例如 `10.0.0.0/8`、`fd00::/8`）信任指定来源。 |

值格式错误（例如意外的符号、前导零或无法识别的项）会导致服务器**拒绝启动**并显示说明性错误；拼写错误会明确暴露，而不会静默关闭代理信任。设置后，请检查请求日志中的 `req.ip`，确认经代理访问时显示的是真实客户端 IP。

---

<a id="cloud-managed-instances"></a>
## 云托管实例

| 变量 | 默认值 | 含义 |
|---|---|---|
| `PAPERCLIP_MANAGED_CONFIG` | 未设置（自托管） | 仅由 Paperclip Cloud 运行环境设置。包含一份 JSON 文档，描述运行环境为此实例管理哪些实验功能开关、声明哪些插件键用于自动安装，以及可选地描述启动时为实例配置的沙盒环境。**未设置**表示自托管：不应用覆盖项，也不改变行为。**已设置但为空或格式错误时，服务器会拒绝启动。** |

你不应自行设置此变量。它让托管实例从运行该实例的集群获得功能配置，而不是从自己的数据库读取；本页记录它是为了方便你在运行中的容器环境里识别此项。

其值是一份 JSON 文档：

```json
{
  "v": 1,
  "mode": "cloud",
  "catalogVersion": "2026.720.0",
  "features": { "enableApps": false, "enableCases": true },
  "plugins": { "autoInstall": ["daytona", "kubernetes"] },
  "environments": [
    { "name": "Daytona", "provider": "daytona", "config": { "target": "us" } }
  ]
}
```

只允许六个顶层键：`v`、`mode`、`catalogVersion`、`features`、`plugins` 和 `environments`。任何其他键都会导致服务器拒绝启动。

其中五项为必填：`v`、`mode`、`catalogVersion`、`features` 和 `plugins.autoInstall`。不过，`features` 可以是 `{}`，`plugins.autoInstall` 可以是 `[]`。`mode` 始终为 `"cloud"`；`catalogVersion` 记录验证此文档所依据的功能目录版本。

`environments` 是唯一的**可选**部分。它是在其他部分之后才加入的；旧版本生成的文档仍必须能在新版本中启动，因为集群镜像轮换不能强制要求同步重新下发配置。因此，省略此项表示“此实例未声明托管环境”。这与缺少 `features` 或 `plugins.autoInstall` 不同；缺少后两者会静默移除运行环境原本打算强制执行的控制。存在 `environments` 时，也会像其他所有内容一样严格验证。详细说明见下文[它配置的沙盒环境](#它配置的沙盒环境)。

### 验证失败时停止启动

只有**未设置**该变量时才表示自托管。其他情况都会被视为运行环境配置错误，服务器会在启动时给出明确错误并停止；截断或拼写错误的文档绝不会静默移除预期强制执行的控制。以下情况都会导致服务器拒绝启动：

- 已设置但为空（自托管模式应彻底取消设置）；
- 不是有效 JSON，或 JSON 顶层不是对象；
- 包含未知顶层键、`v` 不是 `1`，或 `mode` 不是 `"cloud"`；
- 缺少 `features` 或 `plugins.autoInstall`；
- 使用此构建版本不包含的功能键，或该功能键在功能目录中的级别不是 `managed`；
- 功能值不是布尔值，或插件键为空/重复；
- `environments` 部分格式错误，例如包含多条记录、记录中出现未知键、`name` 或 `provider` 为空、`provider` 未列入 `plugins.autoInstall`、`config` 自行设置 `provider`，或 `config` 键看起来像凭据。

### 可控制的内容和限制

`features` 只能设置实例实验性设置中的布尔型功能开关键，也就是 **Settings → Experimental** 页面中的开关。由服务器管理的其他字段（例如启用截止点、回溯小时数）不是布尔值，因此无法通过此方式修改。

此外，构建版本的功能目录会为每个开关标记一个级别：`preference`（租户自己的偏好，运行环境不会更改）、`managed`（运行环境可以设置）或 `floor`（在代码中固定）。只接受 `managed` 键。如果运行版本中某个键的级别不同，说明文档的 `catalogVersion` 与应用版本不匹配；服务器会拒绝启动，而不会用含义不一致的控制配置实例。

Paperclip 会在每个发行版本中通过 `feature-catalog.json` 资源发布此级别映射；该资源由 `scripts/generate-feature-catalog.ts --version <catalogVersion>` 生成。云运行环境会导入此资源，并在写入功能设置前进行验证。

### 不写入数据库

这些覆盖项不会持久化。它们在读取设置时应用，因此每次读取都会重新生效；数据库恢复或手动编辑设置记录都无法重新启用运行环境禁用的功能。

在 UI 中，受覆盖控制的每个键都会显示带锁徽标的 **Managed by Paperclip Cloud**，相应开关也会被禁用。自托管响应完全不包含托管键，因此所有开关仍可编辑。用户界面说明见[实验功能](../../experimental/overview.md#if-a-toggle-is-locked)。

### 它配置的沙盒环境

如果你在托管实例的 **Settings → Instance settings → Environments** 中看到一个已有沙盒环境，而你从未创建过它，那么它来自 `environments` 部分。运行环境会声明该沙盒，并在启动时由服务器代为设置，因此智能体无需先手动配置服务提供商即可运行。

上面的功能覆盖只在读取设置时生效，而此处不同：每个条目都会创建一个真实环境记录，并显示在 UI 中；服务器会在每次启动时重新应用。重复执行是安全的，因为这一步是幂等的，重启只会刷新该记录，不会创建重复记录。

每个条目如下所示：

```json
{
  "name": "Daytona",
  "description": "Managed sandboxes for agent runs",
  "provider": "daytona",
  "config": { "target": "us" }
}
```

- **`name`** 是环境记录的显示名称。
- **`description`** 是可选字段，与文档内容保持同步，因此之后省略它会清空说明，而不是永久保留旧值。
- **`provider`** 是沙盒插件的驱动键，与插件在 `plugins.autoInstall` 中使用的键相同，例如 `daytona` 或 `kubernetes`。此键也必须出现在自动安装列表中：托管实例只能通过运行环境安装插件，因此若服务提供商插件未配置，环境就无法提供运行租约。启动时会检查这种不一致，而不会等到首次运行才发现。
- **`config`** 会原样存入环境记录，并在获取租约时由服务提供商插件验证。各服务提供商接受的字段见[沙盒服务提供商](../adapters/sandbox-providers.md)。

### 每个实例最多一个条目，且不得包含凭据

此部分受两项规则限制。

**最多一条记录。** 数据库通过名为 `environments_managed_sandbox_idx` 的部分唯一索引，强制每个实例最多有一条由 Paperclip 管理的沙盒记录；列表中的每个条目都会配置这同一条记录，因此无法满足更长列表，解析时会直接拒绝。出于相同原因，**非空** `environments` 部分与强制设置 `PAPERCLIP_EXECUTION_MODE` 互斥：两者都会声明同一条记录，同时配置会导致服务器停止，而不会根据启动顺序选择一个。空的 `"environments": []` 表示未声明环境，因此可以与强制执行模式同时使用。

**绝不包含密钥。** 托管配置文档不存放凭据，因此 `config` 中任何看起来像凭据的键（无论处于多少层嵌套，只要匹配 `api_key`、`apiKey`、`token`、`secret`、`password` 或 `credential`）都会被视为放错位置的凭据，并导致服务器拒绝启动。服务提供商凭据应作为普通进程环境变量提供给托管实例：每个随附的沙盒服务提供商都有自己的文档化环境变量；`config` 未提供密钥时，例如会回退使用 `DAYTONA_API_KEY`。

### 服务提供商尚未就绪时

配置过程会等待随附插件启动检查完成，然后确认此条目对应的服务提供商插件已安装、状态为 `ready`，**并且**正在运行有效 worker。插件记录即使显示 `ready`，如果激活失败而没有 worker，也无法提供运行租约，因此此检查不可省略。

服务提供商尚未启动时，Paperclip 会跳过该条目，不会创建活动环境记录。如果上一次启动已经配置过该环境，则会将记录归档，以免调度器继续选择一个无法获取租约的环境。服务器会记录原因并继续启动；受到影响的只有这个环境，不会让整个运行环境陷入崩溃重启循环。

常见情况下可以自动恢复。若插件记录为 `ready`，只是 worker 因重启退避期间崩溃而停止，Paperclip 会监听 worker 管理器，等它重新生成 worker 后立即重做环境设置，因此无需重启服务器即可恢复环境。若插件不存在或状态不是 `ready`，则需要等到下一次健康启动。

这与文档其他部分有相同的有意区分：**格式错误**的配置会阻止启动；**设置步骤失败**则只记录日志，服务器仍会以功能受限的状态启动。

### 移除条目不会删除环境

移除 `environments` 部分或其中的条目，只会停止后续刷新操作，不会删除或归档环境记录。这与 `plugins.autoInstall` 的行为一致（插件方面的说明见[它不会撤销的操作](../../administration/plugins.md#what-it-wont-undo)）：运行可能仍然持有该环境的租约，因此撤销环境必须由人员明确操作，不会因编辑文档而附带发生。上文提到的归档只适用于已声明但无法启动的服务提供商，不适用于已被移除的条目。

---

## 密钥

| 变量 | 含义 |
|---|---|
| `PAPERCLIP_SECRETS_MASTER_KEY` | 32 字节加密密钥，可使用 base64、十六进制或原始字符串格式。 |
| `PAPERCLIP_SECRETS_MASTER_KEY_FILE` | 本地密钥文件路径。 |
| `PAPERCLIP_SECRETS_STRICT_MODE` | 要求服务端环境绑定使用密钥引用。不适用于 `paperclipai configure --section llm` 或 `config.llm.apiKey`。 |

这些值的详细说明见[密钥](./secrets.md)。

---

## 存储

| 变量 | 含义 |
|---|---|
| `PAPERCLIP_STORAGE_PROVIDER` | 存储后端，通常为 `local_disk` 或 `s3`。 |
| `PAPERCLIP_STORAGE_LOCAL_DIR` | 本地磁盘存储的基础目录。 |
| `PAPERCLIP_STORAGE_S3_BUCKET` | S3 存储桶名称。 |
| `PAPERCLIP_STORAGE_S3_REGION` | S3 区域。 |
| `PAPERCLIP_STORAGE_S3_ENDPOINT` | MinIO、R2 等服务使用的自定义 S3 兼容端点。 |
| `PAPERCLIP_STORAGE_S3_PREFIX` | 可选的对象键前缀。 |
| `PAPERCLIP_STORAGE_S3_FORCE_PATH_STYLE` | 服务提供商需要时启用 path-style S3 请求。 |
| `PAPERCLIP_IMPORT_ZIP_MAX_BYTES` | 公司导入 `.zip` 上传的最大大小，单位为字节。默认为 `1073741824`（1 GB）。增大此值可接受更大的数据包；减小则可收紧限制。数值限制在 1 字节至 64 GiB 之间；非数字、转换后为零的小数或超出范围的值都会回退到 1 GB 默认值。 |

---

## 调度器

| 变量 | 默认值 | 含义 |
|---|---|---|
| `HEARTBEAT_SCHEDULER_ENABLED` | `true` | 启用或禁用基于计时器的调度。 |
| `HEARTBEAT_SCHEDULER_INTERVAL_MS` | `30000` | 调度器轮询间隔，单位为毫秒。 |

---

<a id="telemetry-feedback-export"></a>
## 遥测与反馈导出

这些变量控制服务器将运维人员提交的反馈转发到哪里，也控制已弃用的遥测通道（该通道使用同一导出流程）。它们由 `server/src/config.ts` 读取；只有在要将反馈事件从实例发送到单独收集器时才会使用。

| 变量 | 默认值 | 含义 |
|---|---|---|
| `PAPERCLIP_FEEDBACK_EXPORT_BACKEND_URL` | 未设置 | 外部反馈收集器 URL。设置后，服务器会将 `paperclipai feedback` 提交内容转发到此端点。 |
| `PAPERCLIP_FEEDBACK_EXPORT_BACKEND_TOKEN` | 未设置 | 用于验证转发请求的 bearer token。 |
| `PAPERCLIP_TELEMETRY_BACKEND_URL` | 未设置 | `PAPERCLIP_FEEDBACK_EXPORT_BACKEND_URL` 的旧别名。为兼容旧版本仍受支持；新部署请设置 feedback 变量。 |
| `PAPERCLIP_TELEMETRY_BACKEND_TOKEN` | 未设置 | `PAPERCLIP_FEEDBACK_EXPORT_BACKEND_TOKEN` 的旧别名。 |

如果两个变量都未设置，反馈会保存在本地，不会离开实例。

### 关闭遥测

Paperclip 会收集匿名使用遥测信息（例如使用了哪些适配器，以及类似的粗略信号），帮助项目了解 Paperclip 的使用方式。这些数据不包含个人信息。设置以下任一变量即可彻底关闭遥测。两者都只检查精确值 `1`，其中任一项单独设置就足以关闭遥测。

| 变量 | 含义 |
|---|---|
| `PAPERCLIP_TELEMETRY_DISABLED` | 设为 `1` 可关闭 Paperclip 的匿名遥测。 |
| `DO_NOT_TRACK` | 各工具通用的 [Console Do Not Track](https://consoledonottrack.com/) 约定。设为 `1` 可关闭遥测。 |

---

## 可观测性（OpenTelemetry）

Paperclip 可以通过 OpenTelemetry（OTLP）发送分布式追踪数据，让你在 Jaeger、Tempo 或 Honeycomb 等追踪后端中观察请求在服务器中的流转。此功能**默认关闭，只有主动配置后才启用**；除非指定收集器，否则不会加载任何内容。

启用时，请设置 `OTEL_EXPORTER_OTLP_ENDPOINT`，并安装服务器需要的 OpenTelemetry 软件包（`@opentelemetry/sdk-node`、`@opentelemetry/auto-instrumentations-node`、对应协议的导出器、`@opentelemetry/resources` 和 `@opentelemetry/semantic-conventions`）。如果设置了端点但缺少软件包，服务器会记录一行提示，并继续运行，只是不发送追踪数据。

| 变量 | 默认值 | 含义 |
|---|---|---|
| `OTEL_EXPORTER_OTLP_ENDPOINT` | 未设置 | OTLP 收集器端点。设置它即可启用追踪。 |
| `OTEL_EXPORTER_OTLP_PROTOCOL` | `grpc` | 导出器协议：`grpc`、`http/protobuf` 或 `http/json`。未知值会记录警告并回退到 `grpc`。 |
| `OTEL_SERVICE_NAME` | `paperclip` | Span 中报告的服务名称。 |
| `OTEL_SERVICE_VERSION` | `unknown` | Span 中报告的服务版本。 |

端点错误或收集器不可达都不会导致服务器停止；SDK 会记录失败，追踪功能会保持关闭。服务器关停时会在退出前刷新缓冲中的 Span（设有较短超时）。

---

## 错误监控（Sentry）

Paperclip 可以将服务端错误报告到 [Sentry](https://sentry.io)。此功能**默认关闭，只有主动配置后才启用**；出现任何问题都会安全降级：没有 DSN、没有软件包、没有 Sentry，都不会影响服务运行。

启用时，请设置 `SENTRY_DSN` 并安装可选的对等依赖 `@sentry/node`。服务器配置 DSN 后，它也会通过已验证身份的会话响应传递到浏览器，因此 Web UI 也会报告自己的错误（受 React 错误边界保护）。

| 变量 | 默认值 | 含义 |
|---|---|---|
| `SENTRY_DSN` | 未设置 | Sentry DSN。设置此项并安装 `@sentry/node` 后，会启用服务端错误采集并将 DSN 传递到浏览器，以报告客户端错误。未设置时完全不会加载 Sentry。 |

---

## 运维人员设置控制

如果你为其他人托管 Paperclip，以下两个变量可用于控制用户能看到哪些设置页面，以及相应默认值。二者都在启动时读取、在读取设置时应用，且**永不持久化**；清除变量即可恢复默认行为。如果都未设置，UI 和 API 的表现与这些控制存在之前完全相同。完整说明见[实例设置](../../administration/settings.md)。

| 变量 | 默认值 | 含义 |
|---|---|---|
| `PAPERCLIP_HIDDEN_SETTINGS` | 未设置 | 以逗号分隔的设置页面键列表，用于从 UI 隐藏设置项：实例页面、公司页面（成员、邀请、密钥、导出、导入）、单个分区和选项卡，以及单个实验性开关。其中部分键也会让修改 API 返回 `403`；其他键只隐藏 UI。使用 `instance.experimental.*` 可隐藏所有实验开关；添加 `!instance.experimental.<key>` 条目可保留指定开关的可见性。未知键会产生警告并被忽略，因此同一列表可以部署到版本混杂的实例集群。 |
| `PAPERCLIP_SETTING_DEFAULTS` | 未设置 | JSON 对象，用于替换选定实例设置的 schema 默认值，例如 `{"feedbackDataSharingPreference":"allowed"}`。用户明确选择的值始终优先；只有仍采用 schema 默认值的字段才会解析为此处指定值。JSON 格式错误或已知字段的值无效时，服务器会拒绝启动；未知字段名会产生警告并被忽略。 |

---

<a id="agent-runtime"></a>
## 智能体运行时

服务器启动运行时会向智能体进程注入以下变量：

| 变量 | 始终设置？ | 含义 |
|---|---|---|
| `PAPERCLIP_AGENT_ID` | 是 | 智能体 ID。 |
| `AGENT_HOME` | 受支持的托管运行 | 智能体持久文件的可写副本。它与任务工作区、`HOME` 和 `CODEX_HOME` 分开；参见[智能体文件](../../guides/org/agents.md#agent-files-persist-across-tasks)。 |
| `PAPERCLIP_AGENT_KEY_ID` | 受支持的托管运行 | 持久化 Ed25519 公钥指纹。由 Paperclip 分配，已配置的值不能覆盖它。 |
| `PAPERCLIP_AGENT_PUBLIC_KEY` | 受支持的托管运行 | PEM 格式的多行公钥。 |
| `PAPERCLIP_AGENT_PRIVATE_KEY` | 受支持的托管运行 | PEM 格式的多行私钥。托管远程运行也会收到该密钥，因此必须信任持有该身份的主机。独立托管的 HTTP/网关智能体不会收到它。 |
| `PAPERCLIP_COMPANY_ID` | 是 | 公司 ID。 |
| `PAPERCLIP_API_URL` | 是 | Paperclip API 基础 URL。 |
| `PAPERCLIP_API_KEY` | 本地适配器 | 用于 API 身份验证的短期 JWT。使用 `Authorization: Bearer $PAPERCLIP_API_KEY` 发送。对于非本地适配器，运维人员需在适配器配置中设置此值。 |
| `PAPERCLIP_RUN_ID` | 是 | 当前心跳运行 ID。对任何会更改任务的请求，都应通过 `X-Paperclip-Run-Id` 请求头传回此值，以便服务器端审计日志关联到此运行。 |
| `PAPERCLIP_TASK_ID` | 由唤醒触发时 | 触发唤醒的任务。定时或非请求触发的唤醒中为空。 |
| `PAPERCLIP_WAKE_REASON` | 由唤醒触发时 | 此次运行的触发原因。参见下表枚举。 |
| `PAPERCLIP_WAKE_COMMENT_ID` | 评论唤醒 | 触发唤醒的特定评论（由 `issue_commented` 设置）。 |
| `PAPERCLIP_WAKE_PAYLOAD_JSON` | 已弃用 | 不再设置。唤醒上下文（任务摘要和新评论）现在会放在运行提示中传递，因为复制到环境变量中可能会超过操作系统的进程启动限制。如果你在适配器的 `env` 配置中设置此键，Paperclip 也会拒绝它。 |
| `PAPERCLIP_APPROVAL_ID` | 审批唤醒 | 已解决的审批 ID。 |
| `PAPERCLIP_APPROVAL_STATUS` | 审批唤醒 | 审批决定。 |
| `PAPERCLIP_LINKED_ISSUE_IDS` | 可选 | 以逗号分隔的关联任务 ID。 |

智能体运行时需要向 Paperclip 重新验证身份，或需要了解运行的触发上下文时，请使用这些值。

有关身份配置、签名和恢复，请参见[智能体加密身份](../../guides/org/agent-identity.md)。请备份数据库及其配套的密钥加密密钥，以保留身份。

### `PAPERCLIP_WAKE_REASON` 的取值

| 值 | 触发时机 |
|---|---|
| `issue_assigned` | 有任务刚刚分配给此智能体。 |
| `issue_commented` | 智能体负责的任务上发布了新评论。触发评论 ID 位于 `PAPERCLIP_WAKE_COMMENT_ID` 中。 |
| `issue_comment_mentioned` | 已弃用。@提及现在只作为上下文，不再唤醒被提及的智能体，因此新运行不会携带此原因；仍使用此值的唤醒请求会被忽略。旧运行中仍可能看到此值。 |
| `issue_blockers_resolved` | 此任务 `blockedBy` 列出的所有任务都已进入 `done` 状态。 |
| `issue_children_completed` | 此任务的所有直接子任务都已进入终态（`done` 或 `cancelled`）。 |
| `approval_resolved` | 智能体提出的审批已获批准或被拒绝。`PAPERCLIP_APPROVAL_ID` 和 `PAPERCLIP_APPROVAL_STATUS` 会被填充。 |
| `scheduled` | 心跳调度器或例程 cron 触发了定时运行。 |
| `assignment` | 一般由分配触发的运行，没有更具体的原因。 |

Paperclip 创建执行工作区时，还可能注入以下工作区专属变量：

- `PAPERCLIP_WORKSPACE_CWD`
- `PAPERCLIP_WORKSPACE_PATH`
- `PAPERCLIP_WORKSPACE_REPO_ROOT`
- `PAPERCLIP_WORKSPACE_BRANCH`
- `PAPERCLIP_PROJECT_ID`
- `PAPERCLIP_ISSUE_ID`

这些变量主要供适配器开发者和智能体侧工具使用，以便直接访问已解析的执行工作区。

> **审计记录：** 智能体运行发出的每个变更 API 请求都应包含 `X-Paperclip-Run-Id: $PAPERCLIP_RUN_ID` 请求头。服务器用它将任务更新、评论、检出和子任务归因到相应心跳运行。只读请求不需要此请求头。

### `PAPERCLIP_*` 环境绑定如何传入运行

你可以在智能体、项目、例程或适配器配置中定义自己的环境变量；这些变量都会进入运行环境，包括有意使用 `PAPERCLIP_` 前缀的自定义名称（例如密钥 `PAPERCLIP_CLOUD_PROD_PROVIDER_RAILWAY_TOKEN`）。服务器为每次运行构建环境时会采用以下简单规则：

- **绝不接受你配置的 `PAPERCLIP_API_KEY`。** 智能体、项目、例程或适配器配置都不能提供此键。运行框架会为每次运行签发短期运行 token，而它是 `PAPERCLIP_API_KEY` 的唯一来源。
- **运行框架分配的变量始终优先。** Paperclip 为运行设置的变量（例如 `PAPERCLIP_RUN_ID`、`PAPERCLIP_AGENT_ID` 以及上文列出的唤醒/工作区变量）优先于你定义的同名绑定，因此你无法意外覆盖它们。
- **其他所有 `PAPERCLIP_*` 绑定都会传入运行环境。** 它们与没有此前缀的变量一样处理。仅仅使用 `PAPERCLIP_` 前缀不再会导致变量被丢弃。

> **升级说明（v2026.722.0）：** 早期版本会在解析运行环境前删除*所有*带 `PAPERCLIP_` 前缀的绑定，因此以此前缀命名的自定义密钥会在不提示的情况下丢失；现在这些变量会按预期传入。此次更新还使你在适配器或配置环境中设置的静态 `PAPERCLIP_API_KEY` 无法再覆盖运行 token。如果你依赖过这种覆盖，请将其删除：运行 API 密钥现在只来自运行框架签发的 token。

---

## LLM 服务提供商密钥

| 变量 | 含义 |
|---|---|
| `ANTHROPIC_API_KEY` | 供 `claude_local` 使用的 Anthropic API 密钥。 |
| `OPENAI_API_KEY` | 供 `codex_local` 使用的 OpenAI API 密钥。 |
| `GEMINI_API_KEY` | 供 `gemini_local` 使用的 Gemini API 密钥。 |
| `CODEX_API_KEY` | `codex_local` 通过 ACP 运行时，可使用此项代替 `OPENAI_API_KEY`。 |
| `GOOGLE_API_KEY` | `gemini_local` 的备用 Google API 密钥来源。 |

> **提示：** 如果适配器测试失败，先检查进程环境中是否存在所需服务提供商的密钥。

---

## 适配器服务提供商覆盖

可以通过以下由服务器读取的变量，将本地 CLI 适配器指向自定义或远程 OpenAI 兼容网关。每个变量接受 JSON 值；Paperclip 会在运行前将其写入适配器自己的运行时配置，因此你可以将适配器路由到自有服务提供商，而无需手动修改智能体所在机器。每个变量的完整 JSON 结构和行为见相应适配器参考页。

| 变量 | 适配器 | 含义 |
|---|---|---|
| `PAPERCLIP_CODEX_PROVIDERS` | `codex_local` | 自定义服务提供商 JSON（以及可选的 `model_provider`），写入 Codex 管理的 `config.toml`。参见 [Codex](../adapters/codex.md)。 |
| `PAPERCLIP_PI_PROVIDERS` | `pi_local` | 自定义服务提供商 JSON，写入 Pi 管理的 `models.json`。参见 [Pi](../adapters/pi.md)。 |
| `PAPERCLIP_OPENCODE_PROVIDERS` | `opencode_local` | 合并到 OpenCode 的 `provider` 配置中的 JSON。参见 [OpenCode](../adapters/opencode.md)。 |
| `PAPERCLIP_OPENCODE_SMALL_MODEL` | `opencode_local` | 设置 OpenCode 的 `small_model`（辅助模型）。参见 [OpenCode](../adapters/opencode.md)。 |

值支持 `{env:VAR}` 占位符；占位符会在服务端展开，因此密钥不会进入已存储的 JSON。

<a id="adding-models-to-the-model-picker"></a>
### 在模型选择器中添加模型

如果适配器下拉列表中缺少你实际可以运行的模型，例如服务器无法发现的自有网关后方模型，请在 Paperclip 主机上设置 `PAPERCLIP_ADAPTER_MODELS`。其值是一个 JSON 对象，将适配器类型映射到要提供的模型列表：

```json
{ "codex_local": [{ "id": "my-gateway-model", "label": "My gateway model" }] }
```

每项都必须包含非空 `id`；`label` 可选，默认与 `id` 相同。适配器在此变量中定义模型列表后，下拉列表（及其刷新控件）只会显示该列表，而不会使用适配器内置或自动发现的模型。JSON 格式错误时会记录日志并忽略，因此选择器会回退到常规列表。

---

## Paperclip Cloud 连接器（Google OAuth 代理）

以下变量将实例连接到 Paperclip Cloud 代理；该代理为智能体处理 Gmail 和 Google Workspace 登录。大多数情况下你不需要设置这些变量：自托管实例从 Apps 设置中注册时会自动生成并保存实例密钥；Cloud 托管实例则会自动获取这些值。只有自行管理实例身份时才应设置它们，并且应将两个私钥保存在部署环境的密钥管理器中，而不是普通配置中。

实例 ID、两个私钥和环境必须作为一组配置。设置其中任意一项身份值后，四项都必须存在且有效，否则服务器会拒绝配置。

| 变量 | 含义 |
|---|---|
| `PAPERCLIP_CLOUD_CONNECTOR_BASE_URL` | 代理基础 URL，默认是 `https://my.paperclip.app`。必须使用 HTTPS；只有回环主机接受纯 `http://`。包含用户信息、查询字符串或片段的 URL 会被拒绝。 |
| `PAPERCLIP_CLOUD_CONNECTOR_ENVIRONMENT` | 代理环境。只能是 `development`、`staging` 或 `production`；其他值会被拒绝。此值必须与代理环境一致：`https://my.paperclip.app` 属于 `production`，`https://my-staging.paperclip.app` 属于 `staging`。 |
| `PAPERCLIP_CLOUD_CONNECTOR_INSTANCE_ID` | 向 Paperclip Cloud 注册实例时签发的实例标识符。 |
| `PAPERCLIP_CLOUD_CONNECTOR_SIGN_PRIVATE_KEY` | 用于签署连接器请求的 Ed25519 私钥。请保存在密钥管理器中。 |
| `PAPERCLIP_CLOUD_CONNECTOR_SEAL_PRIVATE_KEY` | 用于封装连接器负载的 X25519 私钥。请保存在密钥管理器中。 |

### 已弃用：`PAPERCLIP_ID_CONNECTOR_*`

| 变量 | 状态 |
|---|---|
| `PAPERCLIP_ID_CONNECTOR_BASE_URL` | 已弃用。 |
| `PAPERCLIP_ID_CONNECTOR_ENVIRONMENT` | 已弃用。 |
| `PAPERCLIP_ID_CONNECTOR_INSTANCE_ID` | 已弃用。 |
| `PAPERCLIP_ID_CONNECTOR_SIGN_PRIVATE_KEY` | 已弃用。 |
| `PAPERCLIP_ID_CONNECTOR_SEAL_PRIVATE_KEY` | 已弃用。 |

这些变量属于旧版 Paperclip ID 代理，使用的是不同且不兼容的协议。它们不是上方 `PAPERCLIP_CLOUD_CONNECTOR_*` 变量的别名。如果实例仍设置这些变量，却没有注册 Paperclip Cloud，Google 连接器会报告 `CONNECTOR_MIGRATION_REQUIRED`。要修复此问题，请在 Paperclip Cloud 注册实例、移除旧值，然后重新连接通过旧代理关联的每个 Google 账户。

---

## HTTP 适配器私有端点允许列表

默认情况下，HTTP 适配器可以调用公网 HTTP(S) 来源地址，但不能调用私有地址：套接字层守卫会拒绝私有网络和链路本地元数据目标，并固定 DNS 解析结果以防 DNS 重绑定。使用此变量可将指定的可信私有来源地址加入允许列表，以供 HTTP 适配器调用。

| 变量 | 默认值 | 含义 |
|---|---|---|
| `PAPERCLIP_HTTP_ADAPTER_PRIVATE_ENDPOINT_ALLOWLIST` | 未设置（只允许公网来源地址） | 以逗号分隔的精确来源地址列表，允许通过私有网络访问。每项都是完整来源地址，包括协议、主机和端口，例如 `http://hooks.internal.example:8080`；匹配时不区分大小写。条目必须使用 `http` 或 `https`，除 `/` 外不得包含路径，也不得包含用户信息、查询字符串或片段；列表中的其他格式都会被忽略。 |
