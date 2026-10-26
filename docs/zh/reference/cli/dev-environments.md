---
paperclip_version: v2026.1023.0
seo_title: Worktree 与环境实验室命令
seo_description: 为每个 Git worktree 运行完全隔离的 Paperclip 实例，或构建确定性的本地夹具用于开发和测试。
---

# Worktree 与环境实验室

需要为每个 Git worktree 运行完全隔离的 Paperclip 实例，或创建确定性的本地夹具用于测试时，请使用这些命令。`worktree` 命令组会创建仅供该 worktree 使用的配置、在独立端口运行的嵌入式 Postgres，以及从其他实例复制并播种的数据。这样你可以在主实例旁运行第二个 Paperclip，两者不会共用数据库、端口或公司状态。`env-lab` 命令组则会启动临时基础设施夹具（目前是 SSH 服务器），供适配器和主机配置代码实际连接。这些工具面向代码贡献者和测试实例，不用于日常公司运营。

> **注意：**“worktree 本地实例”是绑定到某个 Git worktree 的独立 Paperclip 数据目录和数据库。配置位于 `<worktree>/.paperclip/config.json`，数据保存在 worktree home 目录下（默认 `~/.paperclip-worktrees`；可用 `--home` 或 `PAPERCLIP_WORKTREES_DIR` 覆盖）。除非通过 `--instance` 指定，否则实例 ID 根据 worktree 名称生成。

---

## 命令一览

请注意命名方式：部分命令是顶层带冒号的命名空间形式（例如 `worktree:make`），其他命令则是 `worktree` 的子命令（例如 `worktree init`）。这是有意设计的，请按下方所示准确输入。

| 命令 | 作用 |
|---|---|
| `paperclipai worktree:make <name>` | 将 `~/paperclip-<name>` 创建为 Git worktree，然后在其中初始化隔离实例。 |
| `paperclipai worktree init` | 为当前所在的 worktree 创建仓库本地配置/环境文件和隔离实例。 |
| `paperclipai worktree env` | 打印指向当前 worktree 本地实例的 shell 环境变量导出语句。 |
| `paperclipai worktree:list` | 列出 Git worktree，并标记疑似 Paperclip worktree 的项目。 |
| `paperclipai worktree:merge-history [source]` | 预览或导入其他 worktree 的 issue/评论历史到当前实例。 |
| `paperclipai worktree reseed` | 从其他实例或 worktree 重新播种现有 worktree 本地实例。 |
| `paperclipai worktree repair` | 创建或修复关联的 worktree 本地实例，不影响主检出目录。 |
| `paperclipai worktree:cleanup <name>` | 安全移除 worktree、其分支及隔离实例数据。 |
| `paperclipai env-lab up` / `status` / `down` / `doctor` | 管理本地 SSH 环境实验室夹具。 |

---

## `worktree:make`

快速创建方式。将 `~/paperclip-<name>` 创建为新的 Git worktree（如果名称未包含 `paperclip-` 前缀，会自动添加），然后在其中初始化隔离的 Paperclip 实例，并从源实例播种数据库。

```sh
paperclipai worktree:make feature-x
# creates the worktree at ~/paperclip-feature-x
```

| 标志 | 用途 |
|---|---|
| `--start-point <ref>` | 新分支基于的远端引用（环境变量：`PAPERCLIP_WORKTREE_START_POINT`）。 |
| `--instance <id>` | 显式指定隔离实例 ID，而不是从名称生成。 |
| `--home <path>` | worktree 实例的 home 根目录（环境变量：`PAPERCLIP_WORKTREES_DIR`；默认 `~/.paperclip-worktrees`）。 |
| `--from-config <path>` | 用于播种新实例的源 `config.json`。 |
| `--from-data-dir <path>` | 推导源配置时使用的 `PAPERCLIP_HOME`。 |
| `--from-instance <id>` | 推导源配置时使用的源实例 ID（默认 `default`）。 |
| `--server-port <port>` | 首选服务器端口；若已被占用，CLI 会选择下一个空闲端口。 |
| `--db-port <port>` | 首选嵌入式 Postgres 端口；若已被占用，CLI 会选择下一个空闲端口。 |
| `--seed-mode <mode>` | 播种配置：`minimal` 或 `full`（默认 `minimal`）。 |
| `--preserve-live-work` | 不隔离复制过来的代理定时器或已分配的开放 issue。 |
| `--no-seed` | 完全跳过从源实例播种数据库。 |
| `--empty` | 创建空实例。此时不会复制任何数据，也会关闭首次启动时自动延迟播种。实例会生成独立的新代理 JWT 和工具操作签名密钥，而不会沿用源实例。以后成功执行 `worktree reseed` 会清除此选项。 |
| `--force` | 替换已有仓库本地配置和隔离实例数据。 |

> **警告：** 默认情况下，播种过程会**隔离**复制过来的实时工作，防止重复数据库在隔离实例中悄悄自动运行代理：禁用定时心跳、将运行中的代理重置为闲置、取消进行中 issue 的代理分配并将其移至 `blocked`（附带说明评论），以及暂停计划任务。只有明确希望 worktree 实例接管并运行这些复制工作时，才使用 `--preserve-live-work`。

`minimal` 和 `full` 播种模式的区别在于复制源数据库的范围。`minimal` 会排除较大的历史记录表；`full` 会复制所有内容。`reseed` 默认使用 `full`；`make`、`init` 和 `repair` 默认使用 `minimal`。

---

## `worktree init`

如果你已经手动创建或检出了 Git worktree，只想为其配置隔离的 Paperclip 实例，请使用此命令。它会写入仓库本地 `.paperclip/config.json` 和 `.env`、分配空闲服务器及数据库端口并播种数据库，但不会创建 worktree 本身。

```sh
paperclipai worktree init --name feature-x
```

| 标志 | 用途 |
|---|---|
| `--name <name>` | 用于生成实例 ID 的显示名称。 |
| `--instance <id>` | 显式指定隔离实例 ID。 |
| `--home <path>` | worktree 实例的 home 根目录（环境变量：`PAPERCLIP_WORKTREES_DIR`；默认 `~/.paperclip-worktrees`）。 |
| `--from-config <path>` | 用作播种来源的 `config.json`。 |
| `--from-data-dir <path>` | 推导源配置时使用的 `PAPERCLIP_HOME`。 |
| `--from-instance <id>` | 推导源配置时使用的源实例 ID（默认 `default`）。 |
| `--server-port <port>` | 首选服务器端口。 |
| `--db-port <port>` | 首选嵌入式 Postgres 端口。 |
| `--seed-mode <mode>` | 播种配置：`minimal` 或 `full`（默认 `minimal`）。 |
| `--preserve-live-work` | 不隔离复制过来的代理定时器或已分配的开放 issue。 |
| `--no-seed` | 跳过从源实例播种数据库。 |
| `--empty` | 创建空实例，不立即复制或延迟复制数据库，效果与 `worktree:make` 相同。 |
| `--force` | 替换已有仓库本地配置和隔离实例数据。 |

如果配置或实例数据已经存在而你没有传入 `--force`，命令会拒绝操作，以免覆盖现有状态。

---

## `worktree env`

打印指向当前 worktree 本地实例的 shell 环境变量导出语句。导入这些变量后，此 shell 中后续所有 `paperclipai` 命令都会使用该 worktree 的配置、home 和实例，而不是默认的 `~/.paperclip`。

```sh
eval "$(paperclipai worktree env)"
```

命令始终输出 `PAPERCLIP_CONFIG`；如果 worktree 的 `.env` 中存在，还会输出 `PAPERCLIP_HOME`、`PAPERCLIP_INSTANCE_ID` 和 `PAPERCLIP_CONTEXT`，之后再输出该 `.env` 中的其他变量。

| 标志 | 用途 |
|---|---|
| `-c, --config <path>` | 要从中读取并导出变量的配置文件路径。 |
| `--json` | 输出 JSON，而不是 shell `export` 语句。 |

> **提示：** 自动化应使用 `--json`。普通输出适用于在交互式 shell 中运行 `eval "$(...)"`。

---

## `worktree:list`

列出当前仓库中可见的所有 Git worktree，并标记看起来属于 Paperclip worktree 的目录（即包含 `.paperclip/config.json`）。在重新播种或清理之前，可用此命令查看已经创建的 worktree。

```sh
paperclipai worktree:list
paperclipai worktree:list --json
```

| 标志 | 用途 |
|---|---|
| `--json` | 输出 JSON，而非文本。 |

---

## `worktree:merge-history`

预览或导入其他 worktree 的 issue 和评论历史到当前实例。代理开始在各自隔离数据库中创建 issue 后，worktree 之间的数据就会分叉；此命令可将这些历史重新汇总。默认仅预览计划，只有提供 `--apply` 才会写入。

```sh
# Preview what would be imported from another worktree
paperclipai worktree:merge-history --from paperclip-feature-x --company <company-id>

# Apply the import
paperclipai worktree:merge-history --from paperclip-feature-x --company <company-id> --apply --yes
```

| 标志 | 用途 |
|---|---|
| `[source]`（位置参数） | 可选源 worktree 路径、目录名或分支名（向后兼容的 `--from` 别名）。 |
| `--from <worktree>` | 源 worktree 路径、目录名、分支名或 `current`。 |
| `--to <worktree>` | 目标 worktree（默认 `current`）。 |
| `--company <id-or-prefix>` | 所选源/目标实例中共用的公司 ID 或 issue 前缀。 |
| `--scope <items>` | 以逗号分隔的导入范围：`issues`、`comments`（默认 `issues,comments`）。 |
| `--apply` | 预览计划后执行导入。 |
| `--dry` | 只预览，不导入任何内容。 |
| `--yes` | 执行导入时跳过交互式确认。 |

> **注意：** 此命令会在实例之间合并 Paperclip **数据**（issue 和评论），不会改动 Git 历史。要合并源代码分支，请直接使用 Git。

---

## `worktree reseed`

从另一个实例或 worktree 为现有 worktree 本地实例重新播种。可用于从主实例获取最新状态以更新过时的 worktree 数据库，或改为使用不同的数据源。此操作会破坏目标数据库中的现有数据，因此会先确认，并默认使用 `full` 播种配置。

```sh
paperclipai worktree reseed --from current --to paperclip-feature-x --yes
```

| 标志 | 用途 |
|---|---|
| `--from <worktree>` | 源 worktree 路径、目录名、分支名或 `current`。 |
| `--to <worktree>` | 目标 worktree（默认 `current`）。 |
| `--from-config <path>` | 用作播种来源的 `config.json`（不能与 `--from` 同时使用）。 |
| `--from-data-dir <path>` | 推导源配置时使用的 `PAPERCLIP_HOME`。 |
| `--from-instance <id>` | 推导源配置时使用的源实例 ID。 |
| `--seed-mode <mode>` | 播种配置：`minimal` 或 `full`（默认 `full`）。 |
| `--preserve-live-work` | 不隔离复制过来的代理定时器或已分配的开放 issue。 |
| `--yes` | 跳过破坏性操作确认提示。 |
| `--allow-live-target` | 覆盖要求先停止目标 worktree 数据库的安全检查。 |

> **警告：** 重新播种会覆盖目标实例数据库。操作前请停止目标实例服务器；默认安全检查会阻止目标仍在运行时操作，除非传入 `--allow-live-target`，但只有在确认没有进程写入数据库时才应使用。`--from` 与 `--from-config` / `--from-data-dir` / `--from-instance` 参数组必须二选一，不能同时使用。

---

## `worktree repair`

创建或修复关联的 worktree 本地实例，不会改动主检出目录。当 worktree 的 `.paperclip` 配置缺失、损坏或从未初始化时使用，例如发生崩溃、部分克隆或手动编辑配置之后。如果传入的 `--branch` 选择器尚未注册为 worktree，命令会在 `<repo>/.paperclip/worktrees/` 下为你创建一个。

```sh
# Repair the worktree you are standing in
paperclipai worktree repair

# Bootstrap a worktree for a branch and link an instance to it
paperclipai worktree repair --branch feature-x
```

| 标志 | 用途 |
|---|---|
| `--branch <name>` | 要修复的现有分支/worktree 选择器，或要在 `.paperclip/worktrees` 下创建的分支名称。 |
| `--home <path>` | worktree 实例的 home 根目录（环境变量：`PAPERCLIP_WORKTREES_DIR`；默认 `~/.paperclip-worktrees`）。 |
| `--from-config <path>` | 用作播种来源的 `config.json`。 |
| `--from-data-dir <path>` | 推导源配置时使用的 `PAPERCLIP_HOME`。 |
| `--from-instance <id>` | 推导源配置时使用的源实例 ID（默认 `default`）。 |
| `--seed-mode <mode>` | 播种配置：`minimal` 或 `full`（默认 `minimal`）。 |
| `--preserve-live-work` | 不隔离复制过来的代理定时器或已分配的开放 issue。 |
| `--no-seed` | 仅修复元数据；引导缺失的 worktree 配置时跳过重新播种。 |
| `--allow-live-target` | 覆盖要求先停止目标 worktree 数据库的安全检查。 |

`repair` 绝不会修改主检出目录，只会修复或关联 worktree 实例，因此可以放心在要恢复的 worktree 内运行。

---

## `worktree:cleanup`

安全移除 worktree、其分支及隔离实例数据。与 `worktree:make` 一样，如果名称未包含 `paperclip-` 前缀，会自动补上。

```sh
paperclipai worktree:cleanup feature-x
```

| 标志 | 用途 |
|---|---|
| `<name>`（位置参数） | worktree 名称；需要时会自动补上 `paperclip-` 前缀。 |
| `--instance <id>` | 显式指定实例 ID（实例 ID 与 worktree 名称不同时使用）。 |
| `--home <path>` | worktree 实例的 home 根目录（环境变量：`PAPERCLIP_WORKTREES_DIR`；默认 `~/.paperclip-worktrees`）。 |
| `--force` | 跳过安全检查（未提交更改或尚未合并到其他位置的提交）。 |

> **警告：** 如果 worktree 有未提交更改，或包含其他位置没有的提交，清理命令会拒绝删除，避免意外丢失工作。`--force` 会跳过这些检查并无条件删除 worktree、分支和实例数据；只有确认这些工作可以丢弃时才使用。

---

## `env-lab`

用于开发和测试的确定性本地环境夹具。目前只有一个 SSH 服务器夹具，适配器和主机配置代码在开发及测试期间可以连接它。每个子命令都接受 `-i, --instance <id>`，用于指定 Paperclip 实例（默认使用当前/默认实例），并接受 `--json` 输出机器可读结果。夹具状态记录在实例 `env-lab/ssh-fixture/` 目录下的 `state.json` 中。

```sh
paperclipai env-lab up        # start the SSH fixture
paperclipai env-lab status    # is it running, and on what host/port?
paperclipai env-lab doctor    # prerequisites + current status
paperclipai env-lab down      # stop it
```

| 子命令 | 作用 |
|---|---|
| `up` | 启动默认 SSH 环境夹具，并打印主机、端口、用户、工作区和日志路径。 |
| `status` | 显示当前夹具状态；没有运行时会报告未启动。 |
| `down` | 停止夹具；如果没有运行中的夹具，会报告这一情况。 |
| `doctor` | 检查 SSH 夹具的前置依赖是否已安装，然后显示当前状态；如果夹具正在运行，还会显示客户端私钥和 known-hosts 文件路径。 |

| 标志 | 用途 |
|---|---|
| `-i, --instance <id>` | Paperclip 实例 ID（默认：当前/默认实例）。 |
| `--json` | 输出机器可读的夹具详情。 |

> **提示：** 在新机器上先运行 `env-lab doctor`。如果依赖不齐全，它会准确指出缺少什么，避免 `up` 运行失败。完成后务必运行 `env-lab down`，避免遗留 SSH 服务器继续占用本地端口。

---

## 常见流程

**为功能分支启动隔离实例：**

```sh
paperclipai worktree:make feature-x --seed-mode minimal
cd ~/paperclip-feature-x
eval "$(paperclipai worktree env)"
paperclipai run
```

**从主实例更新过时的 worktree，之后再将 issue 合并回来：**

```sh
paperclipai worktree reseed --from current --to paperclip-feature-x --yes
# ...work happens in the worktree instance...
paperclipai worktree:merge-history --from paperclip-feature-x --company <company-id> --apply --yes
```

**完成后清理：**

```sh
paperclipai worktree:cleanup feature-x
```

---

## 另请参阅

- [设置命令](./setup-commands.md)——`run`、入职引导和实例级配置；每个 worktree 实例都会继承这些设置。
- [通用选项](./common-options.md)——共享的 `--data-dir`、`--api-base`、`--profile` 和 `--json` 标志。
- [输出与脚本](./output-and-scripting.md)——处理 worktree 和环境实验室命令的 `--json` 输出。
- [适配器参考](./adapter.md)——SSH 环境实验室夹具用于测试适配器。
- [认证](./authentication.md)——每个 worktree 实例如何解析自身凭据和上下文。
