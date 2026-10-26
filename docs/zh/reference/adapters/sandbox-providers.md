---
paperclip_version: v2026.1023.0
seo_title: 沙盒服务提供商
seo_description: 通过服务提供商插件将外部计算资源配置为智能体执行环境，让 Paperclip 在其他计算机而不是主机上运行执行任务。
---

# 沙盒服务提供商

沙盒服务提供商插件可将外部计算资源配置为智能体执行环境。它们位于上游仓库的 `packages/plugins/sandbox-providers/` 下，并以已发布的 npm 软件包形式提供，可通过 Plugin Manager 安装（请参阅[插件](../../administration/plugins.md)）。

沙盒服务提供商插件会注册一种 `kind` 为 `sandbox_provider` 的 `environmentDriver`。安装后，在 **Settings → Instance settings → Environments** 下配置沙盒环境时即可使用该服务提供商。

> **注意：** 本概览仅介绍选定配置字段，并非完整的架构参考。下列字段摘自上游仓库中各服务提供商在 `v2026.512.0` 版本时的 `README.md`；完整配置请查阅已安装服务提供商的文档。

---

## Cloudflare（`provider: "cloudflare"`）

软件包：`@paperclipai/plugin-cloudflare-sandbox`

在 **Settings → Instance settings → Environments** 中配置核心字段 `driver: "sandbox"` 和 `provider: "cloudflare"`。

必填字段：`bridgeBaseUrl`、`bridgeAuthToken`。

验证规则：

- `reuseLease: true` 要求同时设置 `keepAlive: true`。
- 非本地的 `bridgeBaseUrl` 必须使用 `https://`。
- `sessionStrategy` 为 `named` 时必须提供 `sessionId`。
- `timeoutMs` 和 `bridgeRequestTimeoutMs` 都必须介于 1 到 86,400,000 毫秒之间。
- `requestedCwd` 必须为绝对 POSIX 路径。默认值：`/workspace/paperclip`。

### 可靠性调整（v2026.517.0）

Cloudflare 桥接程序在 v2026.517.0 中得到了一系列可靠性加固：

- **增大默认容器规格。** 桥接工作器的容器 `instance_type` 从 `lite` 改为 `standard-2`（并设为 `max_instances: 10`），让长时间运行的智能体有更多余量，不会过早受到限流。
- **流式执行时发送 SSE 保活消息。** 执行流端点会在命令运行期间每 15 秒发送一条 `: keepalive\n\n` SSE 注释，使中间代理和 Cloudflare 边缘节点在静默期间也不会因空闲超时断开连接（例如 `npm install` 下载持续一分钟但没有输出）。
- **桥接控制流量不再使用流式输出。** 标记为桥接通道的命令（就绪探测、文件载荷读取、队列响应等由 Paperclip 在机器侧消费 stdout 的操作）现在使用非流式 `exec` 路径。`@cloudflare/sandbox` SDK 的流式模式在短 shell 进程写入后同一时刻退出时可能丢弃最后一段 stdout，导致出现含义不明的 `"invalid readiness JSON"` 错误。适配器会话仍使用流式输出，因此实时日志不变。
- **默认桥接请求超时提高到 5 分钟。** `DEFAULT_BRIDGE_REQUEST_TIMEOUT_MS` 从 30,000 毫秒提高到 300,000 毫秒，与沙盒默认的 `timeoutMs` 相同；较长的智能体命令不再会在内部超时前耗尽请求时间。
- **环境测试超时会根据沙盒目标调整。** Claude Code、Cursor Local 和 OpenCode 的 `testEnvironment()` 使用的 `helloProbeTimeoutSec` 现在会根据目标是否为沙盒调整：沙盒目标为 **90 秒**；其他目标为 Claude 和 Cursor **45 秒**、OpenCode **60 秒**。Cursor 的前置 `versionProbeTimeoutSec` 也采用同一规则（沙盒 60 秒，其他目标 45 秒）。额外时间可覆盖 Cloudflare `standard-2` 的冷启动，同时不会掩盖本地执行中的真正卡顿。（Grok Local 在此版本中使用自己的 `testEnvironment`，探测时间固定为 45 秒；Grok 的沙盒感知支持计划后续添加。）
- **修正 Pi 适配器安装命令。** `pi_local` 的 `SANDBOX_INSTALL_COMMAND` 现在会固定安装 `@earendil-works/pi-coding-agent@0.74.0`，不再使用以前无人维护的命名空间，因此 Cloudflare 沙盒中的 Pi 智能体首次运行时即可顺利安装。

Paperclip 侧无需配置——将桥接工作器镜像和主机升级到与此版本匹配的版本后，这些修复就会生效。

---

## Daytona（`provider: "daytona"`）

软件包：`@paperclipai/plugin-daytona`

在 **Settings → Instance settings → Environments** 中配置。请将 Daytona API 密钥设置在沙盒环境本身——Paperclip 会将粘贴的 API 密钥保存为公司密钥。如果环境未提供密钥，仍可选用主机级 `DAYTONA_API_KEY` 作为回退值。

可选的 `apiUrl` 和 `target` 设置会直接映射到 Daytona SDK 或客户端配置。此驱动程序同时支持基于 `snapshot` 和基于 `image` 创建沙盒；同时设置两者会被判定为含义不明确并拒绝。可复用租约映射到 Daytona 的停止/启动语义；不可复用租约会在释放时删除。

当前发布的 Daytona SDK 依赖项为 `@daytonaio/sdk`。

### 可复用租约（选择启用）

设置 `reuseLease: true`，即可在多次运行之间保留热沙盒。租约释放时，驱动程序不会删除沙盒，而是将其停止；之后的运行会调用 Daytona 启动操作，恢复同一个沙盒。由于适配器安装和工作区引导内容会跨运行保留，因此无需像临时沙盒那样每次支付创建和引导的费用。将 `reuseLease` 保持为默认值 `false` 时，每次运行都会创建新沙盒，并在释放时删除，与过去的行为相同。

只有运行与创建此租约的运行匹配时，复用才被认为是安全的。驱动程序会在工作区中写入哨兵文件（`.paperclip-runtime/reusable-sandbox-lease.json`），并根据以下内容生成指纹，只有匹配时才恢复：`companyId`、`environmentId`、`agentId`、`executionWorkspaceId`、`adapterType` 以及决定资源规格的配置（`image`、`snapshot`、`target`、`cpu`、`memory`、`disk`、`gpu`）。其中任何一项发生变化（例如提高所需 CPU 或更换镜像），旧租约就不再匹配，会过期并配置新沙盒，而不会恢复规格不符的旧沙盒。复用还要求同时提供 `agentId` 和 `executionWorkspaceId`；缺少其中任一项的运行仍会使用临时沙盒。

### 安全配额的自动归档（默认行为）

Daytona 会将**已停止**的沙盒继续计入组织的存储配额；只有**已归档**的沙盒移入冷存储后才不再计入。为避免泄漏或空闲沙盒占满配额（进而阻止创建新沙盒），驱动程序会在创建时应用可通过各环境覆盖的安全配额默认值：

| 字段 | 默认值 | 说明 |
|---|---|---|
| `autoStopInterval` | `15`（分钟） | 停止空闲的运行中沙盒，以释放 CPU/RAM 并开始计算归档时间。`0` 表示禁用自动停止。 |
| `autoArchiveInterval` | `60`（分钟） | 归档已停止的沙盒，使其不再占用磁盘配额。`0` 表示使用 Daytona 的最大间隔。 |
| `autoDeleteInterval` | `10080`（分钟，7 天） | 清理无人恢复的沙盒。`-1` 表示禁用自动删除；`0` 表示停止后立即删除。 |

只有未设置字段时才使用这些默认值——显式设置的值（包括 `0` 或 `-1`）会原样传递给 Daytona。自动归档可逆：恢复已归档的可复用沙盒时会将其还原，因此这些默认值不会妨碍热沙盒复用。如果确实需要长期保持热沙盒，请增大或禁用相应间隔。

Paperclip 要为之后检查或修复而保留沙盒、而不是删除它时，会先关闭 Daytona 的自动删除，再重新读取沙盒以确认设置已生效。确认后，Paperclip 会停止沙盒（如果它尚未停止）。

### 资源覆盖需要镜像

可选的 `cpu`（核心数，默认 `4`）、`memory`（GiB；可选 `1`、`2`、`4`、`8`，默认 `4`）、`disk`（GiB，默认 `10`）和 `gpu`（单元数）允许申请更大规格的沙盒——但 Daytona 仅在基于镜像创建沙盒时才支持资源覆盖。基于快照/默认配置创建时会拒绝覆盖。驱动程序会提前执行检查：如果设置了其中任何字段却没有 `image`，验证和租约获取都会以清晰的 Paperclip 错误失败（“Daytona 资源设置要求通过镜像创建沙盒；基于快照/默认配置创建时不能覆盖 CPU、内存、磁盘或 GPU”），而不是让 Daytona 返回含义不明的错误。要运行指定规格的沙盒，请配置 `image`；否则不要设置资源字段，Daytona 会使用默认值。默认镜像引用为 `daytonaio/sandbox:0.8.0`。

### 运行器保温生命周期（选择启用）

除了 `reuseLease` 外，另外两个字段控制 `paperclip_runner` 进程在单次运行的各轮之间如何运行：

| 字段 | 默认值 | 说明 |
|---|---|---|
| `runnerLifecycleMode` | `"inherit"` | 可选 `inherit`、`per_turn` 或 `warm`。`inherit` 使用智能体自身的设置。`warm` 会在多轮之间保留 `runnerd` 和沙盒，而不是每轮都拆除。`per_turn` 会在每轮后回收它们。 |
| `runnerIdleTimeoutMs` | `300000`（5 分钟） | 热运行器空闲多久后进行检查点保存并挂起。验证范围为 1000 到 86400000 毫秒。 |

`warm` 会增加少量空闲费用，但可降低每轮延迟，因为运行器和沙盒无需在各轮之间重新启动。`runnerIdleTimeoutMs` 用于在无人使用时挂起热运行器，避免其一直运行。

### 存活检测超时

`livenessTimeoutMs`（默认 `30000`）是沙盒存活读取操作的单次调用超时。如果沙盒连接在没有提示的情况下停止响应，读取会迅速失败并显示错误，而不是一直卡到外层操作超时。设为 `0` 或负数可禁用超时限制。创建/启动/停止/执行操作仍使用 `timeoutMs` 预算，不受此设置影响——启动和恢复调用根据 `timeoutMs` 而非此字段计算截止时间。

### 实时日志流断开时

Daytona 会通过日志套接字将命令输出流式传输到 Paperclip；偶尔套接字会关闭，或在命令仍在运行时停止输出。驱动程序不会因为流断开就认为命令结束，因此不会丢失输出，也不会过早将运行标记为完成：

- **等待命令真正退出。** 只有 Daytona 记录了命令退出，驱动程序才会认为命令完成。连接正常关闭后，驱动程序会重新连接流一次，然后每秒至多一次地检查命令状态和日志快照，直到确认退出。
- **套接字静默时更快切换到轮询。** 如果套接字连续 15 秒没有输出（即使始终未能完成连接），驱动程序也会停止等待并直接开始轮询。
- **不会重复运行。** 命令绝不会再次发送，已经显示的输出不会在执行日志中重复，最后的快照会补上套接字关闭后写入的内容。
- **超时情况会如实报告。** 读取日志超时时会保留部分输出，并说明命令是否仍未确认退出，不会谎报成功。

无需对此进行任何配置。

### 智能体交互式登录

Daytona 是内置服务提供商中唯一能在沙盒内的真实伪终端中运行交互式编程智能体登录流程的服务提供商（声明 `supportsLoginPty`）。因此，你可以在 Daytona 沙盒内部为智能体登录其上游 CLI——Claude（`claude setup-token`）、Codex（`codex login --device-auth`）或 Grok（`grok login --device-auth`）。登录流程运行在终端中，其浏览器设备代码和提示会实时传回给你；Paperclip 会通过终端发送你粘贴的代码和 Enter 按键，而不是将它们放入命令行。凭据保存在沙盒内 `/tmp/paperclip-adapter-login/` 下按登录会话划分的主目录中。无需额外配置即可启用；所有已配置的 Daytona 镜像或快照均支持此功能。

---

## exe.dev（`provider: "exe-dev"`）

软件包：`@paperclipai/plugin-exe-dev`

在 **Settings → Instance settings → Environments** 中配置。请将 exe.dev API 令牌放在沙盒环境本身——Paperclip 会将粘贴的 API 密钥和 SSH 私钥保存为公司密钥。如果环境未提供令牌，仍可选用主机级 `EXE_API_KEY` 作为回退值。

该服务提供商通过 exe.dev HTTPS API 创建虚拟机，并通过 SSH 直接在创建的虚拟机上运行命令。你需要：

- 可以执行 `new`、`ls` 和 `rm` 生命周期命令的 exe.dev API 令牌；如果使用源虚拟机（见下文），还需要 `cp`。如果令牌没有允许服务提供商所需的命令，exe.dev 会返回 403。建议将 `whoami` 和 `help` 用于手动调试。
- 从 Paperclip 主机通过 SSH 访问生成的 `*.exe.xyz` 虚拟机。
- exe.dev 可以识别的 SSH 私钥。可以通过 `sshPrivateKey` 将私钥粘贴到环境配置中，或将 `sshIdentityFile` 指向主机上的绝对路径。

### 每次运行都从准备好的虚拟机开始

默认情况下，每次运行都会通过 `exe.dev new` 创建一台全新的虚拟机，因此较大的仓库每次运行都要承担完整的环境准备费用——工具链、智能体 CLI、软件包缓存、测试浏览器等。如果希望只配置一次，可以按需准备一台虚拟机，并通过 **Source VM**（源虚拟机，位于高级 **VM creation** 设置下，对应 `sourceVm`）让环境使用它。每次运行都会通过 `exe.dev cp` 复制该虚拟机，包括磁盘和配置，然后在副本上开始工作。留空此字段即可继续创建全新虚拟机。

启用前请注意：

- **令牌必须允许 `cp`。** 将它加入 API 令牌允许的命令，否则每次复制都会返回 403。
- **只能设置规格字段。** 复制操作只接受虚拟机名称以及 `cpu`、`memory` 和 `disk`。必须清除 `image`、`command`、`comment`、`env`、`integrations`、`tags`、`setupScript` 和 `prompt`；只要这些字段中任意一个与 `sourceVm` 同时设置，环境就无法保存。
- **不会运行设置脚本。** 请准备好源虚拟机，确保其中已安装 Node 24.11 或更高版本，并且能够使用环境配置的 SSH 密钥。
- **不要将密钥留在源虚拟机上。** 每个副本都会继承源虚拟机磁盘内容。
- **注意 30 秒限制。** 复制操作通过 exe.dev 的 `/exec` API 执行，因此大型磁盘必须在此请求超时前完成复制。

---

## E2B（`provider: "e2b"`）

软件包：`@paperclipai/plugin-e2b-sandbox`（自 `v2026.427.0` 起随产品发布）。

在 **Settings → Instance settings → Environments** 中配置。插件清单声明了一个 `configSchema`，包含 `template`、`apiKey`（Paperclip 密钥引用；缺省时回退到 `E2B_API_KEY`）和 `timeoutMs`。

---

## CreateOS（`provider: "createos"`）

软件包：`@paperclipai/plugin-createos`

如果你希望智能体在 CreateOS 沙盒中运行，命令输出实时回传，并可选择暂停/恢复以便在多次运行间保留热沙盒，请使用 CreateOS。与其他第一方服务提供商一样，它随 Paperclip 一同提供，因此请从 [Plugins](../../administration/plugins.md) 页面安装，而不是从 npm 安装。

在 **Settings → Instance settings → Environments** 中使用核心字段 `driver: "sandbox"` 和 `provider: "createos"` 配置。请将 CreateOS API 密钥设置在沙盒环境本身——Paperclip 会将粘贴的 API 密钥保存为公司密钥。`CREATEOS_API_KEY` 仍可作为主机级可选回退值，但仅适用于官方端点 `https://api.sb.createos.sh`；如果使用自定义 `apiUrl`，必须在环境中明确提供密钥。

驱动程序的 `configSchema` 提供以下字段：

| 字段 | 默认值 | 用途 |
|---|---|---|
| `apiUrl` | `https://api.sb.createos.sh` | CreateOS API 端点。必须为 HTTPS 源（origin，可选以 `/v1` 结尾）；仅回环地址可使用普通 HTTP，并且不允许凭据、查询字符串和片段。必填。 |
| `apiKey` |（无）| 此环境专用的 CreateOS API 密钥——粘贴的密钥或已有 Paperclip 密钥引用。省略后回退到 `CREATEOS_API_KEY`，但仅适用于官方 `https://api.sb.createos.sh` 端点。 |
| `shape` |（无）| 来自 CreateOS shape 目录的沙盒 CPU/内存规格，例如 `s-1vcpu-1gb` 或 `s-2vcpu-4gb`。必填。 |
| `rootfs` |（服务提供商默认值）| 可选的根文件系统或就绪模板 ID/名称。镜像必须提供 Bash 和所选智能体运行时依赖。 |
| `region` |（服务提供商默认值）| 可选区域；必须与 API 端点的区域相匹配。 |
| `timeoutMs` | `300000` | 操作和命令默认超时时间，单位为毫秒。验证范围为 1 到 86400000 的整数。此值不是沙盒生命周期。 |
| `reuseLease` | `false` | 运行后暂停沙盒，并在后续运行中恢复，而不是释放时删除。 |

驱动程序声明了 `supportsReusableLeases` 和 `incrementalSessionOutput`，因此可复用租约（通过 `reuseLease` 设置）会映射到 CreateOS 暂停/恢复功能，命令输出也会在生成时实时流回。

---

## Modal（`provider: "modal"`）

软件包：`@paperclipai/plugin-modal`

第一方沙盒服务提供商，可配置镜像、App、身份验证、超时和网络控制，并使用 [Modal](https://modal.com/) 配置沙盒。必填字段为 `appName`、`image`、`tokenId` 和 `tokenSecret`；`tokenId` 和 `tokenSecret` 必须同时设置。`sandboxTimeoutMs` 默认值为 `3_600_000`（1 小时），必须为不超过 `86_400_000`（24 小时）的正数且为 `1000` 的倍数。Modal 没有原生暂停操作，因此 `reuseLease: true` 会让沙盒持续计费，直到 `sandboxTimeoutMs` 或 `idleTimeoutMs` 到期。完整字段说明和运维验证流程请参阅专门的 [Modal](./modal.md) 页面。

---

## Novita AI（`provider: "novita"`）

软件包：`@paperclipai/plugin-novita-sandbox`

为 Paperclip 智能体配置 Novita Agent Sandbox 实例。像其他插件一样，可在 [Plugins](../../administration/plugins.md) 页面按软件包名称安装：

```text
@paperclipai/plugin-novita-sandbox
```

主机会在安装时将 `npm install` 运行于受管理插件目录中，因此会自动为你安装该服务提供商自己的依赖项（例如 `novita-sandbox`）。

在 **Settings → Instance settings → Environments** 中配置 Novita。请将 Novita API 密钥设置在沙盒环境本身——Paperclip 会将粘贴的 API 密钥保存为公司密钥。如果环境未提供密钥，仍可选用主机级 `NOVITA_API_KEY` 作为回退值。

驱动程序的 `configSchema` 提供以下字段：

| 字段 | 默认值 | 用途 |
|---|---|---|
| `apiKey` |（无）| 此环境专用的 Novita API 密钥——粘贴的密钥或已有 Paperclip 密钥引用。省略后回退到 `NOVITA_API_KEY`（必须至少有一个来源提供 API 密钥）。 |
| `domain` |（SDK 默认值）| 可选的 Novita API 域名覆盖值。 |
| `template` |（SDK 默认值）| Novita 沙盒模板 ID 或名称。留空即可使用 SDK 默认基础模板。 |
| `requestedCwd` | `/home/user/paperclip-workspace` | 在沙盒租约中创建的工作区目录。必须为绝对路径。 |
| `timeoutMs` | `300000` | 沙盒生命周期和单条命令的默认超时时间，单位为毫秒。验证要求 `>= 10000`。 |
| `requestTimeoutMs` | `30000` | Novita SDK 调用的 HTTP/RPC 请求超时时间，单位为毫秒。验证要求 `>= 1000`。 |
| `secure` | `true` | 如果 Novita SDK 支持安全连接，则使用安全连接。 |
| `autoPause` | `false` | 所选模板支持时启用 Novita 沙盒自动暂停。 |
| `reuseLease` | `false` | 在多次 Paperclip 执行之间暂停并稍后恢复沙盒，而不是释放时终止。 |

---

## Kubernetes（`driver: "kubernetes"`）

软件包：`@paperclipai/plugin-kubernetes`（Alpha 阶段，目前为 `v0.1.0`）。

这是可自行托管的沙盒服务提供商。它不会将智能体执行交给受管理的云服务，而是在你自己的 Kubernetes 集群中将每次执行作为工作负载运行——每家公司一个租户命名空间，每次运行使用一个经过加固的 Pod，并以默认拒绝所有流量的网络策略为基础，再由你显式开放访问。如果你需要让智能体在你控制的基础设施上执行（自有计算资源、自有网络策略和自有隔离保证），请使用它。

像其他插件一样，可在 [Plugins](../../administration/plugins.md) 页面安装：

```text
@paperclipai/plugin-kubernetes
```

本地开发时，也可以通过工作区路径安装：

```bash
paperclipai plugin install --local /path/to/paperclip/packages/plugins/sandbox-providers/kubernetes
```

### 适用场景

- 希望智能体沙盒作为 Kubernetes Pod 运行在你管理的集群中，并由 Kubernetes 本身强制执行租户隔离和网络策略。
- 要求智能体在永不离开你环境的基础设施中运行，例如隔离网络、受监管环境或组织政策要求自行托管的环境。
- 希望每次执行都使用微虚拟机级别的隔离（通过 Kata Containers 和 Firecracker 实现）。

### 后端

插件有两种后端模式，由 `backend` 字段选择：

| 后端 | 默认值 | 稳定性 | 多命令执行 | 需要 |
|---|---|---|---|---|
| `sandbox-cr` | 是 | Alpha | 支持 | `kubernetes-sigs/agent-sandbox` 控制器 |
| `job` | 否 | 稳定 | 不支持 | 无，Kubernetes 1.27 及以上版本即可 |

`sandbox-cr`（默认模式）会创建一个 `Sandbox` 自定义资源。其控制器会配置一个长期运行的 Pod，Paperclip 可在其中逐条执行命令——适配器安装依赖的正是这种多命令模式。释放租约时会删除 `Sandbox` CR，控制器随即拆除 Pod。

`job` 是稳定的回退选项。它会创建一个 `batch/v1` Job，其容器入口点运行一次后退出，因此不支持多命令执行——Paperclip 的适配器安装模式不能在 job 模式下工作。只有在无法安装 agent-sandbox 控制器，或必须严格使用稳定版 Kubernetes API 时，才选择此模式。

### 前置条件

使用默认的 `sandbox-cr` 后端时：

1. 需要运行 Kubernetes 1.27 或更高版本的集群。
2. 需要在集群中安装 [`kubernetes-sigs/agent-sandbox`](https://github.com/kubernetes-sigs/agent-sandbox) 控制器。此控制器处于 Alpha 阶段，会安装 `sandboxes.agents.x-k8s.io/v1alpha1` CRD 及其控制器：

   ```bash
   kubectl apply -f https://github.com/kubernetes-sigs/agent-sandbox/releases/latest/download/install.yaml
   ```

3. Paperclip-server 必须能够访问集群——可以在集群内运行（`inCluster: true`），也可以通过 `kubeconfig` 从集群外部访问。

使用 `job` 后端时，只需 Kubernetes 1.27 及以上版本的集群，以及 Paperclip-server 的集群访问权限；无需额外控制器或 CRD。

> `sandbox-cr` 后端基于 agent-sandbox `v1alpha1` 构建。随着该 CRD 演进，请预期可能会有破坏性变更；稳定回退方案是 `job` 后端。

### 配置

在 **Settings → Instance settings → Environments** 下新建一个 `driver: kubernetes` 沙盒环境。以下身份验证字段必须且只能设置一个：

- `inCluster: true` — Paperclip-server 与 Kubernetes 集群位于同一集群时，使用 Pod 内的 ServiceAccount 凭据。
- `kubeconfig: <YAML>` — 内嵌 kubeconfig，并以公司密钥形式保存。
- `kubeconfigSecretRef: <secret-uuid>` — 引用现有 Paperclip 密钥。

常用可选字段：

| 字段 | 默认值 | 用途 |
|---|---|---|
| `backend` | `"sandbox-cr"` | `sandbox-cr`（Alpha，需要 agent-sandbox 控制器）或 `job`（稳定，一次性入口点）。 |
| `adapterType` | `"claude_local"` | 支持的适配器类型之一（`claude_local`、`codex_local`、`gemini_local`、`cursor_local`、`opencode_local`、`pi_local`）。此项决定运行时镜像、环境变量键和出站流量允许列表。 |
| `namespacePrefix` | `"paperclip-"` | 每家公司租户命名空间的前缀。 |
| `companySlug` | 根据 companyId 生成 | 覆盖自动生成的公司 slug。 |
| `imageRegistry` |（无）| 覆盖智能体运行时镜像的默认镜像仓库。 |
| `imageAllowList` | `[]` | 允许的 `target.imageOverride` 值的 glob 模式。为空时表示禁止覆盖镜像。 |
| `imagePullSecrets` | `[]` | 租户命名空间中预先创建的 Docker 镜像拉取密钥名称。 |
| `egressAllowFqdns` | `[]` | 在适配器默认值之外额外允许访问的 FQDN（例如 `api.anthropic.com`）。 |
| `egressAllowCidrs` | `[]` | 额外允许出站访问的 CIDR。 |
| `egressMode` | `"standard"` | `standard`（NetworkPolicy + CIDR）或 `cilium`（CiliumNetworkPolicy + FQDN 允许列表）。 |
| `runtimeClassName` |（无）| 例如 Firecracker 微虚拟机使用 `kata-fc`。集群必须已安装此 RuntimeClass。 |
| `serviceAccountAnnotations` | `{}` | 应用于每个租户 ServiceAccount 的注解（例如 IRSA `eks.amazonaws.com/role-arn`）。 |
| `jobTtlSecondsAfterFinished` | `900` | Job 完成后等待多少秒再进行垃圾回收。 |
| `podActivityDeadlineSec` | `3600` | 单次运行墙钟时间的硬性上限。 |

所选 `adapterType` 决定运行时镜像和默认出站规则。例如，`claude_local` 使用 `ghcr.io/paperclipai/agent-runtime-claude:v1`，并预先允许连接 `api.anthropic.com`；`codex_local` 使用 `ghcr.io/paperclipai/agent-runtime-codex:v1`，并允许连接 `api.openai.com`。完整 JSON Schema 位于上游仓库的 `src/manifest.ts`。

### 集群中会创建哪些内容

首次派发执行时，服务提供商会按需配置每家公司独立的资源。每个租户公司都有自己的命名空间和隔离配置：

```
Namespace          paperclip-{companySlug}   (Pod Security Standards: restricted)
ServiceAccount     paperclip-tenant-sa
Role               paperclip-tenant-role     (only get pods/log)
RoleBinding        paperclip-tenant-rb
ResourceQuota      paperclip-quota
LimitRange         paperclip-limits
NetworkPolicy      paperclip-deny-all        (deny ingress + egress baseline)
NetworkPolicy      paperclip-egress-allow    (DNS + paperclip-server callback + your CIDRs)
                   OR CiliumNetworkPolicy paperclip-egress-fqdn when egressMode=cilium
```

每次运行还会创建以 `pc-{ulid}` 命名的独立短期资源，释放租约时会级联删除（`sandbox-cr` 模式下包括 `Sandbox` CR、Pod 和 `pc-{ulid}-env` 密钥；`job` 模式下包括 `batch/v1` Job、Pod 和密钥）。

### 按任务授予出站访问权限

在上文[配置](#配置)表中设置的 `egressAllowFqdns` 和 `egressAllowCidrs` 会应用于环境中的每次运行，容易让你为了满足要求最高的任务而默默扩大允许列表。按任务授权可以避免这种取舍。环境允许列表可以保持严格，再仅在某项任务运行期间，开放该任务需要的额外目标。

在任务本身的 `executionWorkspaceSettings` 中配置授权（这是 [Issues API](../api/issues.md) 页面中介绍的执行字段之一）：

```json
{
  "executionWorkspaceSettings": {
    "networkEgress": {
      "allowFqdns": ["registry.npmjs.org", "files.pythonhosted.org"],
      "allowCidrs": ["203.0.113.0/24"]
    }
  }
}
```

两个列表都是可选的。Paperclip 会删除每个条目前后的空白字符、移除空项和重复项，并将 FQDN 转为小写；CIDR 则会保持原样。如果两个列表最后都为空，就不会创建额外策略，运行只使用命名空间的常规出站允许列表。

需要授予额外访问权限时，服务提供商会为该运行创建一条与其他资源并列的策略，并按运行工作负载将其命名为 `pc-{ulid}-egress`。（如果工作负载名称导致总长度超过 Kubernetes 的 253 字符上限，中间部分会缩短。）以下两项机制确保按任务授权安全：

- 策略只选择带有此运行 `paperclip.io/run-id` 标签的 Pod，因此额外目标不会对共用租户命名空间的其他智能体 Pod 开放。
- 策略会设置指向该运行 `Sandbox` CR 或 `Job` 的所有者引用，因此运行资源清理时策略也会一并清除。

每次运行的策略只包含获准目标。DNS 和回调 paperclip-server 的访问仍由命名空间级策略提供，因此任务授权只会增加允许范围，不会取代基线。沙盒在为某次运行获取租约时只会读取一次授权——之后编辑任务不会修改已经生效的策略。如果策略创建失败，Paperclip 会释放刚获取的工作负载并让租约获取失败，因此执行绝不会在授权未实际生效的情况下启动。

#### 不同出站模式的实际强制效果

这里的 `egressMode` 选择很重要，因为两种模式并不等价，依赖某一种模式前应了解差异。

使用 `egressMode: "cilium"` 时，授权会转换为 `CiliumNetworkPolicy`。`allowFqdns` 会按域名强制执行：每个条目都转换为精确名称的 `toFQDNs` 匹配规则，并允许 TCP 443。`allowCidrs` 会转换为没有端口限制的 `toCIDRSet` 条目。

使用 `egressMode: "standard"`（默认）时，授权会转换为普通的 `networking.k8s.io/v1` NetworkPolicy——而 Kubernetes NetworkPolicy 无法表达主机名，只能匹配 IP 地址块。因此：

- `allowCidrs` 的行为符合预期：每个 CIDR 对应一条 `ipBlock` 规则，不限制端口。
- **不会强制执行** `allowFqdns`。如果授予了 FQDN 且 `allowCidrs` 为空，策略会回退到允许所有公网 IPv4 地址上的 TCP 80 和 443，以便至少能访问指定主机，同时排除 `0.0.0.0/8`、`10.0.0.0/8`、`100.64.0.0/10`、`127.0.0.0/8`、`169.254.0.0/16`、`172.16.0.0/12`、`192.168.0.0/16` 和 `224.0.0.0/4`。集群内部、回环地址和链路本地元数据端点仍会被阻止——但其他所有公网主机的 80/443 端口也都可访问，不限于你列出的主机。
- 如果标准模式下同时授予 **FQDN 和 CIDR**，则不会启用上述回退。只会允许你列出的 CIDR；指定的 FQDN 只有在解析到这些 CIDR 中的地址时才能访问。

简而言之：在 `standard` 模式下，应将 `allowFqdns` 视为一种声明访问意图的设置，它会允许该次运行访问公网 HTTP/HTTPS；真正受到强制执行的是 `allowCidrs`。如果要求授权严格表示“只允许此主机名，除此之外均不允许”，请使用 Cilium 并设置 `egressMode: "cilium"`。

#### 请求被阻止时

有两项机制可让运行内部诊断被拒绝的请求，智能体无需猜测原因。

首先，每个沙盒都会通过环境变量获得实际生效的授权：

| 变量 | 值 |
|---|---|
| `PAPERCLIP_NETWORK_EGRESS_POLICY` | `kubernetes-default-deny` |
| `PAPERCLIP_NETWORK_EGRESS_GRANT_PATH` | `executionWorkspaceSettings.networkEgress` |
| `PAPERCLIP_NETWORK_EGRESS_ALLOW_FQDNS` | 获准的 FQDN，以逗号分隔（无 FQDN 时为空） |
| `PAPERCLIP_NETWORK_EGRESS_ALLOW_CIDRS` | 获准的 CIDR，以逗号分隔（无 CIDR 时为空） |

其次，如果命令 stderr 看起来像网络请求被阻止（例如 `could not resolve host`、`network is unreachable`、`connection timed out`、`failed to connect` 或 `temporary failure in name resolution`），Paperclip 会附加说明，指出网络策略拒绝或无法路由该请求，列出当前任务已获准的目标（或说明当前没有授权），并指出应在 `executionWorkspaceSettings.networkEgress` 中申请访问权限。这通常足以让智能体区分端点暂时故障和目标未获授权。

### 工作区文件同步

每次智能体运行都会双向传输文件：Paperclip 会在智能体启动前将工作区和运行的资源文件推送到沙盒，之后再将更改的文件和输出拉回主机。在默认的 `sandbox-cr` 后端上，服务提供商现在会直接通过 Pod exec 通道原生完成文件传输，无需启用其他功能。

服务提供商实现了两个文件同步生命周期钩子：`onEnvironmentSyncIn`（主机 → Pod）和 `onEnvironmentSyncOut`（Pod → 主机）。定义这些钩子后，插件工作器会声明 `environmentSyncIn` 和 `environmentSyncOut` 动作，Paperclip 随后会通过**一次** Pod exec 传输 tar 归档来执行每次同步，而不是使用通用 base64 回退方式（每个数据块都要新建一次 exec，也就是新建一次到 API 服务器的 WebSocket 连接）。入站时，主机会在磁盘上构建 tar 包，并将原始字节流写入 Pod 的 stdin；出站时，Pod 会将 tar 包直接写入 stdout，主机则将数据流写入文件。由于不会将整个载荷缓存在内存中，回退方式的 100 MB 内存上限不再适用——但 Paperclip 主机必须为归档文件提供临时磁盘空间；归档会暂存在系统临时目录下名为 `paperclip-k8s-sync-*` 的目录中。

有两项限制需要了解：沙盒不可信，遇到超限时两者都会失败关闭，而不会让恶意 Pod 耗尽主机资源：

- Pod 向主机磁盘传输的数据一旦超过 8 GiB，出站传输就会立即中止。
- 同步 exec 的 Pod stderr 上限为 1 MiB。

这两项都不会作为环境配置字段公开，**Configure** 表中也没有新增相应字段；影响同步的唯一设置是现有 `podActivityDeadlineSec`，每次同步 exec 都会继承该值（转换为毫秒）作为超时。如果你缩短了此截止时间，请记住大型传输现在会与运行本身共用此时间预算。

同步需要租约中有工作区远程目录（`remoteCwd`，默认 `/workspace`，除非你设置了 `remotePath`），该目录也是路径限制根目录：沙盒中的每个路径都必须解析到此目录内。Paperclip 会先在主机上检查每条路径，然后在 Pod 中通过 `realpath` 和 `/proc/self/fd` 固定路径再次检查，避免传输过程中替换符号链接，将写入重定向到根目录之外。出站归档由沙盒生成，因此主机会检查每个 tar 成员（包括所有符号链接和硬链接目标）；如果任何内容会落在提取目录之外，就会在提取前拒绝整个归档。

`job` 后端完全没有 exec 路径，因此无法执行原生同步。此类租约会标记为 `nativeFileSyncUnsupported`，Paperclip 的按租约能力检查会让其继续使用字节内容完全相同的 base64 回退方式——选择 `job` 不会失去文件传输功能，只是速度较慢。

#### 传输失败时

传输失败会明确报告。Pod 内脚本只要以非零状态退出，整个传输就会失败并附上捕获的 stderr，因此不会悄悄出现部分成功：

- **单文件以原子方式写入。** 每个文件会暂存到工作区根目录下权限为 `0700` 的目录中，先设置所需模式，再用 `mv -f` 移动到目标位置。传输中断不会留下截断文件，密钥文件也绝不会短暂地对所有用户可读。
- **目录映射在目标位置直接解压**，与 base64 回退方式使用 tar 时相同，不是原子操作。
- **始终清理临时文件。** 暂存目录使用保留前缀 `.paperclip-upload` 命名，并通过 shell `trap` 在任何退出时（包括失败时）移除。
- **路径越界时，在任何字节移动前就会拒绝，**源文件在校验和复制之间被替换时也一样。

镜像还必须满足一项前置条件：Pod 内脚本需要路径规范化工具 `realpath` 或 `readlink -f`。如果镜像两者都没有，同步会失败关闭，不会只依靠主机检查继续执行。脚本还会用到 `sh`、`tar`、`head`、`mkdir`、`chmod`、`mv`、`cp`、`dirname` 和 `dd`，并读取 `/proc/self/fd`。脚本使用纯 POSIX `sh`，且刻意避免 GNU 专有参数，因此基于 BusyBox 的镜像也可用——但没有 shell 或 `tar` 的镜像无法作为同步目标。第一方 `ghcr.io/paperclipai/agent-runtime-*` 镜像基于 `ubuntu:22.04` 构建，已包含上述所有程序，无需修改。

### 安全基线

每个智能体 Pod 都以非 root 身份运行（`runAsUser: 1000`、`runAsNonRoot: true`），移除所有 Linux 能力并设置 `allowPrivilegeEscalation: false`，使用 `readOnlyRootFilesystem: true`，并为所需可写路径显式挂载 `emptyDir`，同时应用 `seccompProfile: RuntimeDefault`。每个租户命名空间都会强制执行 `pod-security.kubernetes.io/enforce: restricted`，并默认使用拒绝所有流量的 NetworkPolicy；因此实际可用的出站访问仅限于适配器默认值、你通过 `egressAllowFqdns` / `egressAllowCidrs` 开放的内容，以及单项任务通过[按任务授予出站访问权限](#按任务授予出站访问权限)在其执行期间添加的内容。

如需更强隔离，可以安装使用 Firecracker 虚拟机管理程序的 [Kata Containers](https://github.com/kata-containers/kata-containers)，并设置 `runtimeClassName: kata-fc`。此时每个智能体 Pod 都会在 Firecracker 微虚拟机内运行。集群节点必须支持嵌套虚拟化。

---

## Fake Sandbox（`provider: "fake-plugin"`）

软件包：`@paperclipai/plugin-fake-sandbox`。

这是一个第一方确定性沙盒服务提供商：它会在隔离的本地临时目录中运行命令，同时覆盖测试沙盒服务提供商插件的完整生命周期。此服务提供商适用于开发、集成测试，以及在无需外部沙盒服务的情况下复现插件运行时问题。

该插件仅在单体仓库内部使用（`package.json` 中包含 `"private": true`），不会发布到 npm——你需要将其构建为本地工作区插件并安装。其 `configSchema` 提供 `image`（确定性的虚构标签，默认为 `fake:latest`）、`timeoutMs`（默认 `300000`）和 `reuseLease`。需要在测试中获得可预测的沙盒行为，或调试服务提供商插件契约本身时，请选择此服务提供商。

---

## 相关内容

- [插件](../../administration/plugins.md) — 从 Plugin Manager 安装和管理插件。
- [创建适配器](./creating-an-adapter.md) — 如果内置适配器都不适用，可自行开发。
