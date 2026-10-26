---
paperclip_version: v2026.916.0
seo_title: CLI 设置命令
seo_description: 启动 Paperclip 实例、修复损坏的实例或更改服务器运行设置——操作安装和启动的命令。
---

# 设置命令

启动 Paperclip 实例、修复实例或更改服务器运行设置时，请使用这些命令。它们会操作**安装和启动**相关内容，包括配置文件、本地数据库、秘密、存储、主机绑定和引导凭据。它们**不会**创建公司、建立 issue 或唤醒代理；这些操作由控制平面命令负责，详见[`company`](./company.md)、[`issue`](./issue.md)、[`agent`](./agent.md)等参考页。

可以简单地这样区分：设置命令回答“这个实例是否已配置并运行？”，控制平面命令回答“公司在做什么？”。`run` 同时有两种含义，是唯一横跨这两个领域的命令；本页会明确区分它们。

---

## 何时使用这些命令

以下情况下可以使用设置命令：

- 创建初始配置和本地数据目录（[`onboard`](#paperclipai-onboard)）。
- 在更改配置前后验证并修复安装（[`doctor`](#paperclipai-doctor)）。
- 打印远程部署需要的环境变量块（[`env`](#paperclipai-env)）。
- 更改部署模式、数据库、存储、秘密或日志设置（[`configure`](#paperclipai-configure)）。
- 手动创建一次数据库快照（[`db:backup`](#paperclipai-dbbackup)）。
- 在已认证模式下信任私有主机名（[`allowed-hostname`](#paperclipai-allowed-hostname)）。
- 一步完成本地服务器引导和启动（[`run`](#paperclipai-run-local-bootstrap)）。
- 让实例在登录会话之间继续作为后台服务运行（[`service`](./service.md)）。
- 安装、更新、回滚或移除 CLI 本身（[安装 CLI](./installation.md)和[更新 Paperclip](../../how-to/update-paperclip.md)）。
- 在无头环境中生成首个看板凭据（[`auth bootstrap-ceo`](#paperclipai-auth-bootstrap-ceo)）。
- 为调试触发代理的一次心跳（[`heartbeat run`](#paperclipai-heartbeat-run)）。
- 发生事故时暂停公司的所有例行任务（[`routines disable-all`](#paperclipai-routines-disable-all)）。

> **注意：** 示例使用已安装的可执行文件 `paperclipai`。在 monorepo 中可换为 `pnpm paperclipai`；参阅[安装](./installation.md)。

---

<a id="paperclipai-run-local-bootstrap"></a>
## `paperclipai run`（本地引导）

> **警告：** 不带子命令的 `paperclipai run` 是设置命令，会引导并启动本地服务器。`run list`、`run get`、`run events`、`run log`、`run cancel` 等 `run` **子命令**则是另一项功能——它们通过 API 检查和控制心跳运行，详见[运行](./run.md)。不要混淆二者。

裸 `run` 是从“什么都没有”到“可接受请求的健康本地实例”最快的途径。它负责协调以下步骤：

1. 解析实例 ID 和配置路径；如果 `~/.paperclip` 或实例根目录不存在，就创建它们。
2. 如果配置不存在且当前处于 TTY 环境，则交互式调用[`onboard`](#paperclipai-onboard)。非交互式 shell 中会报错并提示先运行 `onboard`。
3. 默认启用修复并运行[`doctor`](#paperclipai-doctor)；任意检查失败时，会停止且不启动服务器。
4. 启动 Paperclip 服务器。
5. 如果实例处于 `authenticated` 模式且使用嵌入式数据库，则会在启动后生成 CEO 引导邀请（参阅[`auth bootstrap-ceo`](#paperclipai-auth-bootstrap-ceo)）。

```sh
paperclipai run
paperclipai run --instance dev
paperclipai run --data-dir ./tmp/paperclip-dev
paperclipai run --no-repair
```

| 标志 | 用途 |
|---|---|
| `-c, --config <path>` | 配置文件路径。 |
| `-d, --data-dir <path>` | 将所有本地状态与 `~/.paperclip` 隔离。 |
| `-i, --instance <id>` | 本地实例 ID（默认 `default`）。 |
| `--bind <mode>` | 仅首次运行时，将引导可达性预设设为 `loopback`、`lan` 或 `tailnet`。 |
| `--repair` | 在 doctor 检查期间尝试自动修复。默认启用。 |
| `--no-repair` | 在 doctor 检查期间禁用自动修复。 |
| `--force` | 即使同一实例已在服务管理器下运行，也继续启动。 |

> **提示：** `--bind` 只在首次运行 `run`、将预设传给引导程序时生效。配置文件创建后，`run` 会继续使用现有配置；之后可通过 [`configure --section server`](#paperclipai-configure) 修改绑定方式。

> **警告：** `run` 在执行其他操作前，会检查该实例是否已由服务管理器运行。如果正在运行，它会拒绝再次启动——两个服务器共用一个端口和数据库会造成问题——并提示你运行 `paperclipai service status --instance <id>`。确实需要以前台进程运行时，可用 `--force` 绕过检查。参阅[服务](./service.md)。

---

## `paperclipai onboard`

交互式首次设置命令。可用于创建全新的本地安装，或通过引导提示重新生成配置。它会写入配置（默认为 `~/.paperclip/instances/<id>/config.json`），将 `PAPERCLIP_AGENT_JWT_SECRET` 和 `PAPERCLIP_TOOL_ACTION_SIGNING_SECRET` 写入相邻的 `.env` 文件，并创建本地秘密密钥文件。

```sh
paperclipai onboard
paperclipai onboard --yes
paperclipai onboard --run
paperclipai onboard --bind lan
```

第一个提示会提供两种设置方式：

| 路径 | 配置内容 |
|---|---|
| **快速入门** | 可直接运行的本地默认配置：嵌入式 PostgreSQL、文件日志、本地磁盘存储、本地加密秘密和环回绑定。适用时会遵循环境变量覆盖，例如 `DATABASE_URL`、`PAPERCLIP_PUBLIC_URL`、`PAPERCLIP_DEPLOYMENT_MODE`。 |
| **高级设置** | 逐步设置数据库、LLM 提供商、日志、服务器/认证、存储和秘密。向导会测试数据库连接；如果你提供 LLM API 密钥，也会验证密钥。 |

| 标志 | 用途 |
|---|---|
| `-c, --config <path>` | 配置文件路径。 |
| `-d, --data-dir <path>` | 将所有本地状态与 `~/.paperclip` 隔离。 |
| `--bind <mode>` | 快速入门可达性预设：`loopback`、`lan` 或 `tailnet`。 |
| `-y, --yes` | 非交互式接受快速入门默认值并立即启动。如果没有 `--bind`，会强制使用可信本机环回默认值，并忽略冲突的可达性环境变量。 |
| `--install-service` | 引导后安装并启动后台服务。 |
| `--no-install-service` | 不安装或建议安装后台服务。 |
| `--run` | 保存配置后立即启动服务器。 |

向导接近结束时，会询问是否将 Paperclip 安装为后台服务，以便在登录时启动，并在关闭终端后继续运行。`--install-service` 会直接安装而不询问；`--no-install-service` 会跳过安装和建议。安装了服务后，引导程序不会再启动前台服务器，因为服务已经在运行 Paperclip。平台不支持服务管理时，`--install-service` 会发出警告并继续。服务安装和管理方式详见[服务](./service.md)。

服务启动后，引导程序会移交给服务，而不是停留在终端：它会等待服务报告实际绑定的端口（配置端口被占用时会自动改用备用端口），打印仪表板 URL，并在交互式终端中打开浏览器。无头和非交互式运行只打印 URL，不会启动浏览器。在交互式终端中不希望引导程序打开浏览器时，可设置 `PAPERCLIP_NO_BROWSER=1`。如果服务未能及时启动，引导程序会明确报告并指引你查看 `paperclipai service logs`，不会谎称成功。

> **注意：** 如果有效配置已经存在，`onboard` 会保持其不变，确保代理 JWT 秘密、工具操作签名秘密和秘密密钥文件存在，并打印后续命令。修改现有安装设置时，请使用[`configure`](#paperclipai-configure)，而不是重新运行引导。

如果实例使用嵌入式数据库且处于 `authenticated` 模式，CEO 引导邀请会延迟到服务器运行后再创建——`onboard` 会提醒你先运行 `paperclipai run`，然后运行[`auth bootstrap-ceo`](#paperclipai-auth-bootstrap-ceo)。

---

## `paperclipai doctor`

执行诊断检查，并可选择修复问题。启动服务器前、修改配置后或实例运行异常时都可以运行。`run` 会自动调用 `doctor`；如果只想查看报告，可单独运行。

```sh
paperclipai doctor
paperclipai doctor --repair
paperclipai doctor --repair --yes
```

命令会加载配置的 `.env` 文件，然后按以下顺序检查；只有配置本身无效时才会提前停止：

- 配置有效性（必须先通过）
- 部署模式/认证模式兼容性
- 代理 JWT 秘密
- 秘密适配器
- 存储
- 数据库连接
- LLM 提供商
- 日志目录
- 监听端口
- Node.js 运行时
- 托管安装：存储区、shim、`PATH` 和程序包保留情况
- 后台服务：定义、运行时、健康状态、版本和残留情况

每行会报告 `✓ pass`、`! warn` 或 `✗ fail`，并附上修复提示。汇总会统计通过、警告和失败的项目；只要存在失败，命令就会明确提示“修复后重新运行”。

最后三组检查关注的是*Paperclip 的安装与服务管理方式*，而非实例配置：

| 检查组 | 检查内容 |
|---|---|
| Node.js 运行时 | Node.js 版本低于 24（24.11.0）时失败。 |
| 托管安装 | 仅在采用[托管安装](./installation.md#托管安装)时运行。验证清单和活动程序包一致、`~/.local/bin/paperclipai` 是有效的 Paperclip shim、其目录已加入 `PATH`（否则警告），以及没有遗留程序包。 |
| 后台服务 | 仅在为此实例安装了[服务](./service.md)时运行。验证磁盘上的服务定义与当前安装生成的内容一致、服务处于活动状态、健康端点有响应，且运行版本与托管安装版本匹配。Linux 上如果启用了登录时启动但关闭了 systemd lingering，会发出警告。 |

按设计，这三项均为可选检查。通过 `npx`、npm 全局安装或源码检出运行 Paperclip，且未安装后台服务时，检查会正常通过，并报告“未安装”，不会报错。

> **注意：** `doctor`（与 `run` 相同）还会快速检查是否存在更新的已发布版本。此检查有缓存，每天最多一次；发现更新时会打印一行提示。设置 `PAPERCLIP_UPDATE_CHECK=0` 或在配置中设置 `updates.checkEnabled` 为 `false` 可关闭此功能。

| 标志 | 用途 |
|---|---|
| `-c, --config <path>` | 配置文件路径。 |
| `-d, --data-dir <path>` | 将所有本地状态与 `~/.paperclip` 隔离。 |
| `--repair`（别名 `--fix`） | 尝试修复可以修复的问题。 |
| `-y, --yes` | 跳过每项修复操作的确认提示。 |

> **警告：** 如果检查支持自动修复，`--repair` 可能会创建或更新本地文件（JWT `.env`、秘密密钥、日志目录）。在共享或类生产实例上运行前，请先检查输出；仅在信任这些修复操作时才同时使用 `--yes`。

---

## `paperclipai env`

打印部署需要的环境变量及其值来源，并提供可直接粘贴的 `export` 块。可用于检查合并配置、默认值和环境变量覆盖后，实例实际解析出的值；也可据此设置远程/容器部署环境。

```sh
paperclipai env
```

命令会读取配置（文件缺失或无法解析时发出警告，但不会失败），然后打印 **Required** 和 **Optional** 两部分，并将每个变量标记为 `set`、`default` 或 `missing`。必需变量为 `PAPERCLIP_AGENT_JWT_SECRET` 和 `DATABASE_URL`。可选项包括 `PORT`、`PAPERCLIP_PUBLIC_URL`、`BETTER_AUTH_TRUSTED_ORIGINS`、代理 JWT 和心跳调度器参数，以及完整的秘密/存储提供商变量。缺失值会在导出块中显示为 `<set-this-value>`，供你补全。

| 标志 | 用途 |
|---|---|
| `-c, --config <path>` | 配置文件路径。 |
| `-d, --data-dir <path>` | 将所有本地状态与 `~/.paperclip` 隔离。 |

> **提示：** 可将导出块传给 shell 或部署清单。标记为 `missing` 的变量必须先提供，否则服务器无法正常启动。

---

## `paperclipai configure`

修改现有安装的一个或多个配置部分，无需重新创建实例。每次保存都会重写配置并更新元数据。可以交互式从菜单选择多个部分，也可用 `--section` 直接指定一项。

```sh
paperclipai configure
paperclipai configure --section server
paperclipai configure --section database
paperclipai configure --section storage
paperclipai configure --section secrets
paperclipai configure --section logging
paperclipai configure --section llm
```

| 配置部分 | 可更改内容 |
|---|---|
| `server` | 部署模式、暴露方式、主机绑定、端口、UI 服务方式和认证基础 URL。 |
| `database` | 嵌入式或外部 PostgreSQL、连接字符串、备份设置。 |
| `storage` | 存储提供商（`local_disk` 或 `s3`）及其选项。 |
| `secrets` | 秘密提供商、严格模式和本地密钥文件（需要时会创建）。 |
| `logging` | 日志模式和目录。 |
| `llm` | LLM 提供商和 API 密钥。 |

| 标志 | 用途 |
|---|---|
| `-c, --config <path>` | 配置文件路径。 |
| `-d, --data-dir <path>` | 将所有本地状态与 `~/.paperclip` 隔离。 |
| `-s, --section <section>` | 配置一个部分后退出；可选 `llm`、`database`、`logging`、`server`、`storage` 或 `secrets`。 |

> **注意：** `configure` 要求已有配置；找不到时会报错并提示先运行 `onboard`。指定 `--section` 时只配置该部分后退出；省略时则循环操作，方便连续编辑多个部分。

---

## `paperclipai db:backup`

使用当前配置为实例数据库创建一次性快照。这是手动备份，与服务器定时执行的备份相互独立。连接字符串按以下顺序解析：`DATABASE_URL`、已配置的 `postgres` 连接字符串、嵌入式 PostgreSQL 默认值。

```sh
paperclipai db:backup
paperclipai db:backup --dir /backups/paperclip
paperclipai db:backup --retention-days 14
paperclipai db:backup --json
```

| 标志 | 用途 |
|---|---|
| `-c, --config <path>` | 配置文件路径。 |
| `-d, --data-dir <path>` | 将所有本地状态与 `~/.paperclip` 隔离。 |
| `--dir <path>` | 备份输出目录；优先于已配置的备份目录。 |
| `--retention-days <days>` | 每日清理时保留文件的天数（正整数）。默认为配置值或 30。 |
| `--filename-prefix <prefix>` | 备份文件名前缀（默认 `paperclip`）。 |
| `--json` | 以 JSON 输出备份元数据（文件、大小、清理数、目录、保留期、来源）。 |

> **提示：** 使用嵌入式数据库时，即使服务器未启动也可备份——`db:backup` 会直接连接嵌入式端口。请查看打印出的 `Backup dir` 确认备份位置。

---

## `paperclipai allowed-hostname`

将主机名添加到实例允许列表，使其在 authenticated/private 模式下被接受。Paperclip 在私有网络中拒绝应被信任的主机时（例如 Tailscale 机器名），可以使用此命令。

```sh
paperclipai allowed-hostname dotta-macbook-pro
paperclipai allowed-hostname my-host.ts.net
```

| 参数/标志 | 用途 |
|---|---|
| `<host>` | 要允许的主机名（会规范化并转换为小写）。 |
| `-c, --config <path>` | 配置文件路径。 |
| `-d, --data-dir <path>` | 将所有本地状态与 `~/.paperclip` 隔离。 |

此命令会就地更新本地配置。新条目只有在**重启服务器**后才会生效。允许的主机名只在 `authenticated` + `private` 模式下执行校验；其他模式中，命令会说明此设置不起作用。

---

## `paperclipai auth bootstrap-ceo`

在无头环境中生成**首个**看板凭据：向数据库插入一次性引导邀请并打印邀请 URL。`authenticated` 实例没有现存登录身份时，可用它创建初始实例管理员。`local_trusted` 实例不需要此操作——环回连接默认拥有看板权限，命令会说明原因并退出。

```sh
paperclipai auth bootstrap-ceo
paperclipai auth bootstrap-ceo --expires-hours 24
paperclipai auth bootstrap-ceo --force
paperclipai auth bootstrap-ceo --base-url https://paperclip.example.com
```

| 标志 | 用途 |
|---|---|
| `-c, --config <path>` | 配置文件路径。 |
| `-d, --data-dir <path>` | 将所有本地状态与 `~/.paperclip` 隔离。 |
| `--force` | 即使已有实例管理员，也创建新的邀请。 |
| `--expires-hours <hours>` | 邀请有效期（小时），限制在 1–720 之间，默认 72。 |
| `--base-url <url>` | 用于生成邀请链接的公共基础 URL。 |

如果已有管理员，除非传入 `--force`，否则命令会拒绝创建。每次运行还会先撤销所有仍有效、尚未接受的 CEO 引导邀请，再生成新邀请。使用嵌入式数据库时，必须先启动服务器（或启动嵌入式集群）才能成功写入；连接失败时命令会提示启动服务器后重试。

> **注意：** 生成首个凭据后，无需再使用 `bootstrap-ceo`。看板令牌可以非交互式生成更多看板令牌和代理密钥；详见[认证](./authentication.md)。

---

## `paperclipai heartbeat run`

为单个代理准确触发**一次**心跳，并将实时日志流式输出到终端。这是调试和观察工具，不是常规工作方式——它会向服务器的唤醒端点发送请求，再由服务器端运行时运行适配器。CLI 只负责触发和跟踪；模型由服务器端执行。

```sh
paperclipai heartbeat run --agent-id <agent-id>
paperclipai heartbeat run --agent-id <agent-id> --source on_demand --trigger manual
paperclipai heartbeat run --agent-id <agent-id> --timeout-ms 120000 --debug
```

| 标志 | 用途 |
|---|---|
| `-a, --agent-id <agentId>` | **必填。** 要调用的代理。 |
| `-c, --config <path>` | 配置文件路径。 |
| `-d, --data-dir <path>` | 将所有本地状态与 `~/.paperclip` 隔离。 |
| `--context <path>` | CLI 上下文文件路径。 |
| `--profile <name>` | CLI 上下文配置文件名称。 |
| `--api-base <url>` | Paperclip 服务器 API 基础 URL。 |
| `--api-key <token>` | 用于代理认证请求的 Bearer 令牌。 |
| `--source <source>` | 调用来源：`timer`、`assignment`、`on_demand` 或 `automation`（默认 `on_demand`）。 |
| `--trigger <trigger>` | 触发详情：`manual`、`ping`、`callback` 或 `system`（默认 `manual`）。 |
| `--timeout-ms <ms>` | 等待的最长时间（毫秒）；默认 `0`，即不设置客户端超时。 |
| `--json` | 在适用时输出原始 JSON。 |
| `--debug` | 输出原始适配器 stdout/stderr JSON 块，而非格式化事件。 |

命令会轮询运行事件和日志，直到运行进入终态（`succeeded`、`failed`、`cancelled`、`timed_out`）；运行未成功时以非零状态退出，并打印错误及结果详情。检查其他位置触发的运行，请使用 [`run`](./run.md) 子命令。

> **提示：** 正常运营时，代理的日常唤醒建议使用[`agent wake`](./agent.md)和[`prompt`](./prompt.md)中的提示移交流程。只有需要观察单次运行全过程时，才使用 `heartbeat run`。

---

## `paperclipai routines disable-all`

在配置的本地实例中暂停某公司所有未归档的例行任务。这是事故/维护开关：调查问题期间可暂停计划自动化，而不删除任何内容。它直接操作本地数据库（需要时启动嵌入式集群并应用待处理迁移），因此即使服务器处于关闭状态也能运行。

```sh
paperclipai routines disable-all --company-id <company-id>
paperclipai routines disable-all --company-id <company-id> --json
```

| 标志 | 用途 |
|---|---|
| `-c, --config <path>` | 配置文件路径。 |
| `-d, --data-dir <path>` | 将所有本地状态与 `~/.paperclip` 隔离。 |
| `-C, --company-id <id>` | 要暂停例行任务的公司；未提供时回退到 `PAPERCLIP_COMPANY_ID`。 |
| `--json` | 以 JSON 输出结果计数。 |

命令会报告暂停了多少例行任务、多少原本已暂停，以及多少已归档（保持不变）。所有状态不是 `paused` / `archived` 的例行任务都会被设为 `paused`。要通过 API 创建、编辑或恢复单个例行任务，请使用 [`routine`](./routine.md) 命令。

> **警告：** 此操作会一次性暂停公司的**所有**例行任务。恢复操作需要通过 [`routine`](./routine.md) 逐个执行，目前没有统一的“全部启用”命令；请谨慎使用。

---

## 本地路径

本地实例的状态保存在 `~/.paperclip/instances/<instance-id>` 下（默认实例 ID 为 `default`）：

| 数据 | 路径 |
|---|---|
| 配置 | `~/.paperclip/instances/default/config.json` |
| 数据库（嵌入式） | `~/.paperclip/instances/default/db` |
| 日志 | `~/.paperclip/instances/default/logs` |
| 存储（本地磁盘） | `~/.paperclip/instances/default/data/storage` |
| 秘密密钥 | `~/.paperclip/instances/default/secrets/master.key` |

使用 `--data-dir`（或 `--instance`）可运行隔离实例，适合创建干净测试环境和 worktree，且不会影响主安装。

---

## 另请参阅

- [安装](./installation.md)——安装 CLI 并选择部署方式。
- [服务](./service.md)——使用 `paperclipai service` 在后台运行实例。
- [更新 Paperclip](../../how-to/update-paperclip.md)——检查、应用并回滚新版本。
- [认证](./authentication.md)——引导后的看板令牌、代理密钥和登录流程。
- [运行](./run.md)——检查和控制心跳运行（`run` *子命令*）。
- [例行任务](./routine.md)——创建、编辑和恢复计划自动化。
- [代理](./agent.md)——唤醒代理并使用 `agent local-cli` 本地运行。
- [通用选项](./common-options.md)——共用标志及上下文/配置文件解析方式。
