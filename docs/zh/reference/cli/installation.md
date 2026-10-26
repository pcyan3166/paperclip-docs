---
paperclip_version: v2026.831.1
seo_title: 安装 Paperclip CLI
seo_description: 将 paperclipai 单文件程序安装到本机、连接到实例，并从终端启动本地安装。
---

# 安装 CLI

Paperclip CLI 以单个可执行文件 `paperclipai` 的形式发布。本页说明如何在本机获取该程序、连接到实例，以及从全新状态启动本地实例。从“尚未安装”到“拥有可操作的健康服务器”最快只需两条命令：`paperclipai onboard` 写入配置，然后 `paperclipai run` 启动服务。

如果你确定会长期在这台机器上运行 Paperclip，可以直接使用[托管安装](#托管安装)——这是可更新、可回滚并可作为后台服务运行的 CLI 版本。

---

## 前置条件

CLI 是一个 Node 程序。开始前，`PATH` 中必须有 Node.js 24 或更高版本（具体而言，至少为 24.11.0）。托管安装无法在较旧版本上运行；使用 `npx` 安装或直接在 Node 22/23 上启动服务器时，会显示版本不受支持的警告并继续运行，但发出警告并不代表该版本受支持——依赖服务之前请先升级 Node。

```sh
node --version   # must report v24.11.0 or higher
```

常见的本地安装只需要这一项条件：`paperclipai run` 可以为你启动嵌入式 PostgreSQL，因此开始使用时不需要额外安装数据库服务器。如果要让 CLI 连接自己的 PostgreSQL，请准备好连接字符串，CLI 会改用该数据库。

---

## 获取 `paperclipai` 可执行文件

`paperclipai` 发布在 npm 上，因此最快的方式是不安装，直接通过 `npx` 运行：

```sh
npx paperclipai onboard --yes
```

这是启动程序的标准方式，始终运行最新发布版本。如果希望 `paperclipai` 命令在多次运行之间保留，有两种选择：优先使用下一节介绍的[托管安装](#托管安装)，或通过 npm 全局安装：

```sh
npm install -g paperclipai
# verify it is on your PATH
paperclipai --help
```

该包只提供一个可执行文件：`paperclipai`。**本文档中的所有示例都使用 `paperclipai <command>` 的形式。**如果尚未安装，请在命令前加上 `npx`；通过 `npx` 调用与直接运行已安装的可执行文件等效。

> **注意：** 如果你在 Paperclip monorepo 的克隆仓库中工作，请改用仓库内的开发别名 `pnpm paperclipai <command>`。该命令通过 `tsx` 直接从源码运行相同 CLI，适用于开发 CLI 本身。monorepo 同样将 `npx paperclipai onboard --yes` 作为打包安装的标准快速入门方式。

---

## 托管安装

`npx` 适合尝试 Paperclip，但它是临时运行方式——每次运行时都会解析当前缓存的版本，也没有更新或回滚机制。希望长期使用 Paperclip 时，请将其安装到**托管 CLI 存储区**：

```sh
npx paperclipai install
```

这条命令会下载最新发布版本、验证文件，并将 `paperclipai` 命令安装到 `~/.local/bin`。之后可以直接输入 `paperclipai`，并获得 `npx` 无法提供的三项能力：通过单条命令更新、立即回滚到上一个版本，以及将 Paperclip 作为[后台服务](./service.md)运行。

托管安装与 CLI 其他部分一样，需要 Node.js 24 或更高版本；与 `npx` 安装不同，它会在版本较低时直接拒绝运行。

### 安装文件位置

托管安装会将下载的每个版本保存为独立、自包含的文件，并通过符号链接指向当前启用版本。安装内容不会进入系统范围，也不需要 `sudo`——所有文件都保存在你的 home 目录下。

| 内容 | 路径 |
|---|---|
| 托管存储根目录 | `~/.paperclip/cli` |
| 下载的程序包 | `~/.paperclip/cli/installs` |
| 安装清单 | `~/.paperclip/cli/install.json` |
| 指向当前程序包的链接 | `~/.paperclip/cli/current` |
| `paperclipai` 命令 | `~/.local/bin/paperclipai` |

清单还会记住你之前使用的两个程序包，因此 `paperclipai update --rollback` 可以立即回滚。下次成功更新时，会清理更早的程序包。

如果 `~/.local/bin` 尚未加入 `PATH`，`install` 会询问是否将其加入 `~/.bashrc` 或 `~/.zshrc`，并在文件中添加清晰标记的代码块，以便日后识别和移除。拒绝提示（或使用其他 shell）时，命令会打印供你手动添加的语句：

```sh
export PATH="$HOME/.local/bin:$PATH"
```

### 选择安装版本

默认安装最新稳定版。使用标志可以选择其他版本：

```sh
paperclipai install                          # latest stable
paperclipai install --canary                 # the canary channel
paperclipai install --version 2026.609.0     # one exact published version
paperclipai install --ref main               # built from GitHub source
```

| 标志 | 用途 |
|---|---|
| `--canary` | 安装 npm canary 通道版本；功能较新，但稳定性较低。 |
| `--version <version>` | 安装指定的完整 npm 版本，必须为完整的 `x.y.z` 格式；不接受 `latest` 或不完整版本号。 |
| `--ref <ref>` | 安装 GitHub 分支、标签或提交 SHA，并在本机从源码构建。 |
| `--repo <owner/name>` | 与 `--ref` 搭配时，覆盖使用的 GitHub 仓库（默认 `paperclipai/paperclip`）。必须提供 `--ref`。 |
| `-y`、`--yes` | 对 Git 引用代码的执行和受支持的 shell `PATH` 更新自动表示同意，不再提示。适用于脚本和 CI。 |

`--canary` 与 `--version` 不能同时使用；`--ref` 也不能与二者中的任何一个组合——每次安装只能选择一个渠道。

> **警告：** `--ref` 会下载该提交对应的仓库，并在本机运行其中的依赖安装和构建脚本。CLI 会先发出警告并请求确认，然后才开始获取代码；非交互式 shell 中除非传入 `--yes`，否则会拒绝执行。只应安装你信任的源码。

无论选择哪种方式，CLI 都会先对新程序包执行烟雾检查：运行刚安装的程序，并确认其报告的版本号与预期一致，然后才将其设为活动版本。检查失败时不会切换当前版本。

随时可以验证安装结果：

```sh
paperclipai --version
```

不确定选择哪个通道时，可运行 `paperclipai channels` 查看全部四个发布通道——`stable`、`beta`、`nightly` 和 `canary`——以及各通道当前已发布版本和此安装正在跟随的通道。添加 `--json` 可获取机器可读输出。完整介绍请参阅[查看当前所在通道](../../how-to/update-paperclip.md#see-which-channel-youre-on)。

之后要切换通道或版本时，无需重新安装；请参阅[`paperclipai update`](../../how-to/update-paperclip.md)。

---

<a id="removing-the-managed-install"></a>
## 移除托管安装

`paperclipai uninstall` 会移除托管 CLI，但保留你的全部数据。

```sh
paperclipai uninstall
```

此命令不接受任何选项。它会依次停止并移除已安装的[后台服务](./service.md)、删除 `~/.local/bin` 中的 `paperclipai` 命令、移除之前添加到 `~/.bashrc` 和 `~/.zshrc` 的 `PATH` 代码块，最后删除 `~/.paperclip/cli` 托管存储区。

公司、配置、数据库、日志和秘密都存放在 `~/.paperclip` 下的其他位置，不会被修改；以后重新安装时，可以从原处继续使用。

> **注意：** 如果 `~/.local/bin/paperclipai` 不是 Paperclip 托管安装创建的链接（例如你手动替换过它），`uninstall` 会保留该文件并告知你，而不是删除其他命令。如果还有为*其他*实例安装的服务，该命令也会拒绝运行；请先使用 [`paperclipai service uninstall --instance <id>`](./service.md) 卸载它们。

---

## 首次运行：引导本地实例

两条命令即可从空机器启动服务器：

```sh
paperclipai onboard
paperclipai run
```

`onboard` 写入实例配置；`run` 验证配置并启动服务器。如果尚无配置且当前处于交互式终端，也可以只运行 `run` 完成所有步骤：服务器启动前，`run` 会自动先调用引导程序。

---

### `paperclipai onboard`

`onboard` 是交互式首次运行向导。它会写入配置文件和服务器所需的本地密钥材料，然后询问是否立即启动 Paperclip。设置新的本地安装或通过向导提示重新生成配置时使用。

```sh
paperclipai onboard
```

第一个提示会让你选择设置路径：

| 路径 | 配置内容 |
|---|---|
| **快速入门** | 可直接运行的本地默认配置：嵌入式 PostgreSQL、本地磁盘文件存储和本地加密秘密。这是推荐方式。 |
| **高级设置** | 逐步配置数据库、LLM 提供商、日志记录、服务器绑定、存储和秘密。需要明确控制其中任意设置时使用。 |

引导过程中，CLI 还会确保代理 JWT 秘密（`PAPERCLIP_AGENT_JWT_SECRET`）和本地秘密密钥文件存在；如果缺失则会创建。在高级设置中，如果你提供 PostgreSQL 连接字符串，向导会测试数据库连接；如果你提供 LLM API 密钥，向导也会当场验证该密钥，让你在服务器启动前发现问题。

完成时，引导程序会打印最终解析出的配置和下一步命令。

| 标志 | 用途 |
|---|---|
| `--yes`、`-y` | 非交互式接受快速入门默认值并立即启动。如果未使用 `--bind`，此选项会强制采用可信本机环回默认值，并忽略可达性环境变量。适用于脚本和 CI。 |
| `--run` | 保存配置后立即启动服务器，相当于在启动提示中回答“是”。 |
| `--bind <mode>` | 为快速入门服务器配置应用可达性预设；可选 `loopback`、`lan` 或 `tailnet`。 |
| `--install-service` | 引导完成后安装并启动后台服务，不询问。 |
| `--no-install-service` | 不安装或建议安装后台服务。 |
| `--config <path>`、`-c` | 将配置写入指定路径，而非默认实例配置路径。 |
| `--data-dir <path>`、`-d` | 将所有本地 Paperclip 状态与 `~/.paperclip` 隔离；适合创建干净的测试实例和 worktree。 |

```sh
# Non-interactive Quickstart that boots immediately
paperclipai onboard --yes

# Save the config and start, but keep interactive prompts
paperclipai onboard --run

# Quickstart bound to the LAN instead of loopback
paperclipai onboard --bind lan
```

在交互式终端中，引导程序还会询问是否要将 Paperclip 安装为后台服务，使其登录时自动启动，并在关闭终端后继续运行。选择是即可安装并启动服务，而不是以前台方式运行。`--install-service` 会跳过提问并直接安装；`--no-install-service` 会跳过安装建议。具体安装内容请参阅[服务](./service.md)。

> **注意：** 如果目标路径已经配置了 Paperclip，重新运行 `onboard` 会检测到现有安装并保持当前配置不变。如果代理 JWT 秘密或秘密密钥文件缺失，它仍会补齐，然后询问是否启动。修改现有安装的设置时，请使用 `paperclipai configure`，而不是重新引导。

---

### `paperclipai run`

`run` 是一次性启动健康本地实例的命令，会依次完成引导和启动：

1. 解析活动实例与配置路径，并加载实例环境文件。
2. 如果配置不存在且终端处于交互状态，先调用引导程序。
3. 默认启用修复并运行 `doctor` 检查；任意检查失败时，都会在服务器启动前停止。
4. 启动 Paperclip 服务器。

```sh
paperclipai run
```

| 标志 | 用途 |
|---|---|
| `--instance <id>`、`-i` | 选择本地实例 ID（默认 `default`）。可用于并行运行多个隔离实例。 |
| `--bind <mode>` | 首次运行（配置尚不存在）时，将可达性预设传给引导程序：`loopback`、`lan` 或 `tailnet`。 |
| `--repair` / `--no-repair` | 默认启用修复；`--no-repair` 会以只读模式运行 `doctor`，只报告问题而不更改文件。 |
| `--force` | 即使同一实例已由服务管理器运行，也继续启动。 |
| `--config <path>`、`-c` | 使用指定配置文件。 |
| `--data-dir <path>`、`-d` | 将本地状态与 `~/.paperclip` 隔离。 |

```sh
# Run a second, isolated instance
paperclipai run --instance dev

# Keep all state inside a worktree
paperclipai run --data-dir ./tmp/paperclip-dev

# Diagnose without letting doctor modify anything
paperclipai run --no-repair
```

> **提示：** 在没有 TTY 的非交互式环境中，`run` 不会替你启动引导。请先运行一次 `paperclipai onboard`，再重试 `paperclipai run`。

> **注意：** 如果同一实例已经作为后台服务运行，`run` 会报错并退出，避免在同一端口启动第二个副本。可使用 `paperclipai service status` 检查运行中的实例；确实要并行以前台进程启动时，可传入 `--force`。

> **警告：** 不带子命令的 `run` 是*本地引导*命令，会在你的机器上启动服务器。`run list`、`run live`、`run get`、`run events`、`run log` 等 `run` *子命令*则完全不同：它们通过 API 检查和控制已经运行实例中的心跳运行。请勿混淆两者。子命令详见[运行](./run.md)。

---

## 数据的存放位置

`onboard` 和 `run` 会将所有内容写入每个实例各自的 home 目录。默认实例位于 `~/.paperclip`，其中包含配置、嵌入式数据库、日志、存储和秘密密钥。传入 `--data-dir <path>` 可更改根目录，传入 `--instance <id>` 可清晰隔离多个实例。

设置期间，CLI 还会管理两项凭据材料，无需你手动连接：代理 JWT 秘密（保存在实例环境文件的 `PAPERCLIP_AGENT_JWT_SECRET` 中）和本地秘密主密钥文件。二者在首次引导时创建，此后会重复使用。

---

## 服务器启动后

启动服务器还不会自动为 CLI 提供可用于操作公司的认证身份。`run` 显示 Paperclip 已开始监听后，请将 CLI 连接到该实例并选择角色身份：

- 在全新且可交互的机器上运行连接向导。向导会解析 API 基础 URL、检查健康状态、以看板身份登录、生成令牌并保存配置文件。
- 在无头环境或 CI 中，生成看板令牌或代理密钥，并显式配置 CLI 使用它。

具体步骤请参阅[认证](./authentication.md)。关于 CLI 如何查找服务器（`--api-base` / `PAPERCLIP_API_URL` 的解析顺序）以及所有客户端命令共享的标志，请参阅[通用选项](common-options.md)。

---

## 后续步骤

- [认证](./authentication.md)——连接 CLI、选择角色身份并生成看板令牌或代理密钥。
- [服务](./service.md)——让 Paperclip 在后台持续运行，而不必一直打开终端。
- [更新 Paperclip](../../how-to/update-paperclip.md)——检查、应用并回滚新版本。
- [设置命令](./setup-commands.md)——使用 `doctor`、`configure`、`env` 和 `allowed-hostname` 修复和调整实例。
- [通用选项](common-options.md)——共用标志及 CLI 解析目标服务器的方式。
- [运行](./run.md)——在运行中的实例中检查心跳运行的 `run` 子命令。
- [概览](overview.md)——CLI 的用途及其在整体运营模式中的位置。
