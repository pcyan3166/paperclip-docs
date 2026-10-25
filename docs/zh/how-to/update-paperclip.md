---
paperclip_version: v2026.1012.0
seo_title: 将 Paperclip 更新至最新版本
seo_description: 在不丢失数据的情况下将现有安装迁移到新版本——先检查什么、如何运行更新，以及如何确认更新已生效。
---

# 将 Paperclip 更新至最新版本

Paperclip 发布了新版本——也许你在[发布页面](https://github.com/paperclipai/paperclip/releases)上看到了它，也许某个智能体标记了一个上游已修复的 Bug，也许你只是想要最新的 UI。本指南将指导你更新现有安装。

如果你是通过 `paperclipai install` 安装 Paperclip 的，简而言之：

```bash
paperclipai update
```

该命令会检查 npm，备份数据库，替换为新版本，如果你有后台服务则重启它，并保留上一个版本以便随时回滚。本页面的其余部分将解释其含义，并涵盖你可能使用的其他 Paperclip 安装方式。

如果你是首次安装，请遵循[安装指南](../guides/getting-started/installation.md)。

---

## Paperclip 版本机制

Paperclip 使用**日历版本控制**：`YYYY.MDD.P`。各部分分别为年份、月-日，以及同日补丁槽。因此 `v2026.525.0` 是 2026 年 5 月 25 日发布的第一个稳定版。Canary 构建带有 `-canary.N` 后缀，并通过 `canary` npm dist-tag 发布；稳定版通过 `latest` 发布。

每个稳定版在父仓库的 `releases/vYYYY.MDD.P.md` 以及 [GitHub 发布页面](https://github.com/paperclipai/paperclip/releases) 上都有发布说明。更新前请浏览这些说明——它们会指出破坏性变更、迁移和新环境变量。

Paperclip 还会主动提示你。`paperclipai run` 和 `paperclipai doctor` 会每天对 npm 进行一次缓存检查，当你的通道中存在更新版本时，会打印一行通知。设置 `PAPERCLIP_UPDATE_CHECK=0`，或在配置中将 `updates.checkEnabled` 设为 `false`，即可关闭此功能。

---

## 提交前检查

`--check` 会在 npm 上查找你的通道，将其与你正在运行的版本进行比较，并告知你发现的情况——而不进行任何更改。

```bash
paperclipai update --check
paperclipai update --check --json
```

当有可用更新时，它以退出码 `10` 退出；当你已是最新版本时，以 `0` 退出，这使其易于脚本化：

```bash
paperclipai update --check || paperclipai update --yes
```

`--dry-run` 更进一步，描述它*将要*采取的确切操作——包括是否会进行备份——而不实际执行任何操作。

```bash
paperclipai update --dry-run
```

---

## 升级到 v2026.1012.0 或更高版本：先停止所有服务

> **警告：** 此版本需要完全停止，而非热重启。工作区恢复和智能体文件收集过去使用 `<hash>.lock/` 文件夹来锁定每个目标目录。现在它们使用 `<instance root>/locks/directory-merge/` 下的操作系统支持的 SQLite 文件锁，如果进程崩溃，操作系统会释放这些锁。旧进程不参与新锁机制，因此旧版本和新版本同时写入同一实例根目录可能会损坏彼此的工作。

除非你设置了 `PAPERCLIP_HOME` 或 `PAPERCLIP_INSTANCE_ID`，否则你的实例根目录是 `~/.paperclip/instances/default`。如果你是从更早版本升级，请执行以下操作，而不是简单的 `paperclipai update`：

1. **在服务器仍运行时进行备份。** `paperclipai update` 需要可访问的数据库来创建自己的快照，而你即将停止它，所以现在先创建一个：

   ```bash
   paperclipai db:backup
   ```

2. **排空并停止所有可能写入实例根目录的旧进程。** 让活动的智能体运行完成，然后停止服务器（`paperclipai service stop`，或在 `paperclipai run` 上按 Ctrl+C）。不要忘记其他主机或共享同一实例根目录的其他容器中的工作进程。
3. **将遗留的锁定文件夹移至一旁。** 在停止所有服务后，将 `<instance root>/locks/directory-merge/` 中的任何 `<hash>.lock/` 目录移动到你自己的临时目录中。如果可能需要未完成运行的证据，请保留它们。切勿删除或移动 `.lock.sqlite` 文件——它们是永久性的，锁定机制依赖于它们保持原位。
4. **更新并启动新版本。**

   ```bash
   paperclipai update --no-backup    # you took the backup in step 1
   paperclipai service start         # or: paperclipai run
   ```

   也在新版本上启动其他所有工作进程。
5. **确认运行已保存文件。** 让一个智能体运行完成，并检查其文件更改是否已落入工作区，而不仅仅是模型回复了。仅成功的回复并不能证明文件已被收集。

跨此版本回滚也需要完全停止：在启动旧版本之前，排空并停止所有新进程。`.lock.sqlite` 文件可以保留在磁盘上——旧版本会简单地忽略它们。

> **提示：** 此升级还会运行一批数据库迁移。它们都是附加性的，只有一个 `0298` 会触及现有行：它会为你已有的任何 Honcho、Mem0、Zep、Supermemory 和 Cognee 连接添加默认使用指令。之后你可以编辑或关闭这些指令——参见[保存的连接指令](../../connectors/connection-instructions.md)。

---

## 应用更新

```bash
paperclipai update
```

`upgrade` 是一个别名，如果你的手指习惯使用这个词。

以下是执行序列，以免出现意外：

1. **它确定你安装 Paperclip 的方式。** 托管安装、全局 npm、`npx` 或源码检出——每种方式都有不同的处理方式，下文将详细介绍。
2. **它进行数据库备份。** 这在任何替换之前发生，使用与 [`paperclipai db:backup`](../../reference/cli/setup-commands.md#paperclipai-dbbackup) 相同的机制。如果实例从未完成引导且没有数据，则跳过备份并给出提示。
3. **它下载并验证新版本**，然后原子地将活动负载切换为它。你的上一个版本保留在磁盘上。
4. **它重启你的后台服务**（如果该实例有活动服务），使用[热重启](../../reference/cli/service.md#restart-without-losing-work)将活动的智能体运行移交给新进程，而不是丢弃它们。
5. **它修剪不再需要保留的旧负载。**

如果重启的服务在新版本下无法恢复健康状态，更新会自动回滚到上一个负载并重启它，然后报告故障。你不应该最终处于损坏的构建状态。

| 标志 | 用途 |
|---|---|
| `--check` | 检查是否有可用更新，但不应用。 |
| `--dry-run` | 打印操作而不更改任何内容。 |
| `--latest` | 切换到最新稳定通道。 |
| `--canary` | 切换到 Canary 通道。 |
| `--version <version>` | 安装确切的已发布版本。 |
| `--rollback` | 切换回保留的上一个托管负载。 |
| `--no-backup` | 跳过更新前的数据库备份。 |
| `-y`, `--yes` | 确认显式降级而无需提示。 |
| `--json` | 打印机器可读输出。 |

`--latest`、`--canary` 和 `--version` 是互斥的——最多选择一个。

---

## 查看你所在的通道

在切换之前，了解你所在的位置很有帮助。`paperclipai channels` 显示四个发布通道，并标记此安装遵循哪一个。

```bash
paperclipai channels
paperclipai channels --json    # machine-readable output
```

通道按从最稳定到最不稳定的顺序打印，每个通道都带有其当前发布的版本和一行安装提示：

| 通道 | npm dist-tag | 说明 |
|---|---|---|
| `stable` | `latest` | 手动发布，经过 3 天以上的 Beta 测试——几乎适用于所有人的推荐发布。 |
| `beta` | `beta` | 在审批门后手动提升——发布候选版：几天后成为稳定版的内容。 |
| `nightly` | `nightly` | 每晚一次，经过冒烟测试——昨天的合并，作为整体进行测试。 |
| `canary` | `canary` | 每次合并到 master——最前沿。 |

底部会告诉你此安装所在的通道。源码检出一个占位符版本，该版本不对应已发布的通道，因此你会看到相关说明。相同的名称也适用于 Docker 镜像：`ghcr.io/paperclipai/paperclip:{latest,beta,nightly,canary}`。

要移动到新的构建，请使用带有通道标志的更新命令。

---

## 切换通道

如果没有通道标志，`update` 会保持你当前的状态：稳定版保持稳定，Canary 保持 Canary，固定版本保持固定。传递标志以移动。

```bash
paperclipai update --canary                # try the canary channel
paperclipai update --latest                # come back to stable
paperclipai update --version 2026.609.0    # pin to one exact version
```

Canary 构建能更早获得新功能，但可能更不稳定，并且随时可以切换回 `--latest`。

移动到旧版本是降级，CLI 会在继续前要求你确认。在脚本中传递 `--yes` 以预先确认。请记住，代码回退并不意味着数据库架构回退——参见下方的警告。

---

## 回滚

如果更新出现问题，你不需要重新下载任何内容。托管安装会在磁盘上保留你的上一个负载，`--rollback` 会直接切换回它。

```bash
paperclipai update --rollback
paperclipai update --rollback --dry-run    # see which version you'd land on
```

回滚过程中会重启活动服务，因此你可以在几秒钟内回到旧版本并恢复服务。

> **警告：** 回滚逆转的是*代码*，而不是你的*数据*。更新期间运行的数据库迁移不会被撤销。如果新版本迁移了你的数据库，而你需要完全回退，请恢复更新前的备份——这正是 `update` 默认执行备份的原因。

`--rollback` 仅适用于托管安装。其他安装类型没有保留的负载可供切换回。

> **警告：** 从 v2026.1012.0 或更高版本回滚到更早版本？不要让 `--rollback` 在运行中的工作上进行热重启。首先排空并停止所有写入实例根目录的服务器和工作进程，然后回滚并启动旧版本——参见[升级到 v2026.1012.0 或更高版本](#upgrading-to-v202610120-or-later-stop-everything-first)。让 `.lock.sqlite` 文件保持原位。

---

## 关于更新前备份

每个托管更新都以数据库快照开始，使用你实例配置的备份目录和保留设置。你不需要记住手动执行。

有两点值得注意：

- **如果数据库不可达，更新将停止。** Paperclip 不会在没有安全网的情况下静默更新。错误信息会提示你使用 `paperclipai service start` 启动服务并重试，或使用 `--no-backup` 故意跳过备份。
- **`--no-backup` 是一个真实选项，而非陷阱。** 在一次性开发实例上，或在你刚刚自己创建快照后，跳过它是合理的。对于你关心的任何环境，请让它运行。

要在更新之外创建快照，请使用 [`paperclipai db:backup`](../../reference/cli/setup-commands.md#paperclipai-dbbackup)。

---

## 如果你未使用托管安装

`paperclipai update` 会识别你安装 Paperclip 的方式并执行正确的操作——有时这意味着它会告诉你它不会触碰你的设置。

### 全局 npm 安装

`update` 会针对你请求的版本为你运行 `npm install -g`，并在显式降级前进行提示。`--dry-run` 会打印确切的 npm 命令而不运行它。

```bash
paperclipai update
paperclipai update --version 2026.609.0
```

### `npx`

`npx` 运行本质上是临时的——没有已安装的内容可供更新。`update` 会说明这一点，并指向持久化路径：

```bash
npx paperclipai install     # move to a managed install
paperclipai update          # from then on, this works
```

如果你更愿意继续使用 `npx`，只需添加 `@latest` 以强制重新解析，而不是使用缓存副本：

```bash
npx paperclipai@latest run
```

### 源码检出

Paperclip 不会修改你正在工作的 git 仓库。`update` 会检测检出并将其交还给你：

```bash
git pull
pnpm install
pnpm db:migrate
pnpm dev
```

在拉取之前停止开发进程，并先解决脏树问题。要固定到标记的发布版本而不是跟踪 `master`：

```bash
git fetch --tags
git checkout v2026.525.0
pnpm install
pnpm db:migrate
pnpm dev
```

### 从 git 引用构建的托管安装

如果你使用 `paperclipai install --ref <branch-or-tag>` 安装，`update` 会在 GitHub 上重新解析该引用，如果提交已移动则重新构建。它会先警告你，因为从源码构建会执行该仓库的构建脚本，并在执行前需要确认（或 `--yes`）。如果你固定到确切的提交 SHA，则没有可移动的目标，`update` 会告诉你安装已固定。

---

## 验证更新是否成功

1. **检查运行版本。** 在 UI 中，将鼠标悬停在左侧边栏底部的小 **`v`** 徽章上（位于文档链接和设置/主题图标旁边）——工具提示会显示完整的服务器版本。从终端，`paperclipai --version` 报告 CLI 版本，`paperclipai service status` 报告运行中的服务器实际提供的版本。
2. **运行 `paperclipai doctor`。** 它检查托管安装和后台服务，包括运行中的服务器版本是否与你刚安装的版本匹配。
3. **打开仪表板。** 确认 UI 加载，你的公司和智能体存在，且没有渲染为错误状态。
4. **触发一次心跳。** 为现有智能体分配一个小任务，或等待下一次计划的心跳。观察运行日志中的成功回合。这确认适配器在新构建下仍能启动。

---

## 故障排除

**`doctor` 报告服务器与托管安装之间的版本不匹配** — 服务仍在运行旧构建。重启它并要求新版本：

```bash
paperclipai service restart --expected-version <version>
```

**更新停止并提示“无法进行更新前备份”** — 数据库未运行或不可达。启动它（`paperclipai service start`，或在终端中 `paperclipai run`），然后重试。仅在你故意跳过快照时使用 `--no-backup`。

**“实例 … 的另一次重启仍在运行”** — 之前的重启未完成并留下了锁。消息中包含锁文件路径；确认没有其他重启后，删除它并重试。

**`npx paperclipai` 在 `@latest` 后仍报告旧版本** — npx 按名称缓存，如果注册表查找被限流或离线，则回退到缓存。清除它并重试：

```bash
npx clear-npx-cache         # or: rm -rf ~/.npm/_npx
npx paperclipai@latest --version
```

**源码检出上数据库迁移失败** — 不要针对半迁移的数据库向前滚动。恢复你的最后一个快照，提交包含迁移错误的 issue，并停留在上一个标记直到问题解决。参见[备份和恢复公司](./back-up-and-restore-a-company.md)。

**服务器启动但 UI 空白或显示旧资源** — 硬刷新浏览器（Cmd+Shift+R）以绕过缓存的 UI 包。如果你位于反向代理后面，也请刷新其缓存。

**更新后智能体停止运行** — 检查运行日志中的适配器错误。新版本偶尔会收紧环境变量验证或要求更新的适配器二进制文件（Claude Code、Codex 等）。在主机上更新这些二进制文件并重新测试。

> **注意：** 使用非官方的、社区维护的桌面应用？它会自我更新——参见[社区桌面应用](./community-desktop-app.md#updating)。

---

## 相关

- [安装 CLI](../../reference/cli/installation.md) — 托管安装存储、通道和 `paperclipai uninstall`。
- [服务](../../reference/cli/service.md) — 在后台运行 Paperclip，以及热重启。
- [安装](../guides/getting-started/installation.md) — 每种路径的全新安装。
- [备份和恢复公司](./back-up-and-restore-a-company.md) — 在更新生产安装前创建快照。
- [部署到 VPS 或 Fly.io](./deploy-to-vps-or-fly.md) — 影响你重启方式的生产部署模式。
