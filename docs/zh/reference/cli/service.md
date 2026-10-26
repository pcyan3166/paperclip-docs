---
seo_title: CLI 服务命令
seo_description: 将 Paperclip 从关闭终端就会退出的进程改为受管理的后台服务，使其在注销和重启后仍持续运行。
---

# 服务命令

许多人一开始会在终端窗口中运行 `paperclipai run`，但关闭窗口后 Paperclip 就会停止。`service` 命令会将此工作交给操作系统管理：登录时自动启动，崩溃后自行重启，保留可持续查看的日志，即使完全退出终端也能继续运行。希望 Paperclip *始终可用*，而不是每次都记得手动启动时，就使用这些命令。

在 macOS 上，服务由 launchd 代理管理；在 Linux 上，则由 systemd **用户**服务管理。两种情况下都以你自己的用户账户运行，无需使用 `sudo`，也不会执行系统范围的安装。

> **注意：** 每个 `service` 子命令都接受 `-i, --instance <id>`，用于指定本地实例（默认值为 `default`）；也接受 `--json`，用于输出机器可读结果而非文本。如果平台不支持服务管理（例如 Windows，或没有可用 systemd 用户管理器的 Linux 容器或 WSL1），命令会说明原因并正常退出。此时应改用[`paperclipai run`](./setup-commands.md#paperclipai-run-local-bootstrap)。

---

## 安装内容

服务定义是保存在你个人主目录下的一个小文件，它会指示操作系统运行 `paperclipai run --instance <id>`。定义根据当前安装生成，因此始终指向你实际使用的 CLI。

|  | macOS | Linux |
|---|---|---|
| 服务管理器 | launchd | systemd（用户级） |
| 服务名称 | `ing.paperclip.paperclipai` | `paperclipai.service` |
| 定义文件 | `~/Library/LaunchAgents/ing.paperclip.paperclipai.plist` | `~/.config/systemd/user/paperclipai.service` |
| 日志 | `~/.paperclip/instances/<id>/logs/service.log` 和 `service.err.log` | systemd 日志，可通过 `journalctl --user` 查看 |

非默认实例会有独立服务，互不冲突：实例 `dev` 在 macOS 上对应 `ing.paperclip.paperclipai.dev`，在 Linux 上对应 `paperclipai-dev.service`。

启动服务时会设置 `PAPERCLIP_SERVICE_MANAGED=1`、`PAPERCLIP_INSTANCE_ID` 和 `PAPERCLIP_HOME`，以便服务器知道自己由服务管理器启动，以及应使用哪个实例和主目录。

---

## 安装服务

`service install` 会写入定义、向服务管理器注册，并启动 Paperclip。

```sh
paperclipai service install
paperclipai service install --no-start-now
paperclipai service install --no-start-on-login
paperclipai service install --enable-linger
```

| 标志 | 用途 |
|---|---|
| `--no-start-now` | 安装但不立即启动。适用于先注册服务、之后再手动启动的情况。 |
| `--no-start-on-login` | 安装但不设置登录时自动启动。服务会保留，但在手动启动前处于停用状态。 |
| `--enable-linger` | 允许 systemd 在没有活动登录会话时启动用户服务。仅限 Linux。 |
| `-i, --instance <id>` | 本地实例 ID（默认值：`default`）。 |
| `--json` | 输出机器可读 JSON。 |

在 Linux 上，用户注销时用户级服务通常会停止；如果你只通过 SSH 访问无头服务器，这并非预期行为。“Linger”是 systemd 的一项设置，可使服务在注销后继续运行。在交互式终端中，CLI 会询问是否启用；传入 `--enable-linger` 即可预先确认启用。此操作会为你的用户运行 `loginctl enable-linger`，并可能请求系统授权。

命令会报告定义是否发生变化、平台、服务名称、定义文件路径以及是否启用了 linger。

> **提示：** 可以安全地重复运行 `service install`，这也是修复服务定义偏差的常用方式，例如安装位置改变后重新生成定义。只有内容确实发生变化时，命令才会重写定义。

首次运行设置期间也可以完成上述操作：[`paperclipai onboard`](./setup-commands.md#paperclipai-onboard) 会询问是否安装服务；`paperclipai onboard --install-service` 则会跳过询问并直接安装。

---

## 启动、停止和检查

```sh
paperclipai service status
paperclipai service start
paperclipai service stop
```

| 命令 | 用途 |
|---|---|
| `status` | 显示服务管理器状态和健康状态。 |
| `start` | 启动后台服务。 |
| `stop` | 停止后台服务。 |

出现异常时，应先运行 `status`，因为它会同时回答两个问题。它从服务管理器获取平台、服务名称、服务是否已 `installed`、`active` 和 `enabled`，以及进程 ID。然后探测实例自己的健康端点，报告服务器是否实际响应及其运行版本。

两类状态可能不一致，而这种不一致本身很有用：服务处于 `active` 但健康检查失败，表示服务器曾启动但随后失败；实例健康但服务不活动，通常表示某个前台 `paperclipai run` 正在占用端口。

`start` 和 `stop` 都会输出操作后的状态，以便你看到命令的效果。

---

<a id="restart-without-losing-work"></a>
## 重启而不中断工作

直接停止再启动会杀掉运行中的智能体。`service restart` 执行的是**热重启**：它会通知正在运行的服务器即将重启，使正在执行的智能体运行能够移交到新进程，而不是直接中断。

```sh
paperclipai service restart
paperclipai service restart --wait
paperclipai service restart --expected-version 2026.609.0
```

| 标志 | 用途 |
|---|---|
| `--wait` | 等待活动运行完成，而不是将它们移交给新进程。 |
| `--expected-version <version>` | 要求重启后的服务器报告指定版本。 |

具体流程如下：CLI 会为实例获取重启锁，避免两个重启操作重叠；记录当前服务器的进程 ID 和版本；请求服务管理器重启；然后持续轮询健康端点，最多等待一分钟，直到服务器重新就绪。如果传入 `--expected-version`，命令会继续等待，直到服务器报告的版本与之匹配。因此，如果重启后悄悄回到了旧版本，命令会报告失败而不是成功。最后会输出新状态、健康检查结果和服务器对工作移交情况的报告。

默认情况下，活动运行会被新进程*接管*。如果希望先等它们完成，再重启，请传入 `--wait`。

> **注意：** 如果重启过程被中断，下次运行可能会报告另一个重启仍在进行。此消息会包含锁文件 `hot-restart.lock` 的路径（位于实例目录中）。确认没有其他重启正在进行后，可以删除锁文件并重试。

检测到活动服务时，`paperclipai update` 会替你执行相同的热重启，因此更新后通常无需手动重启。详见[更新 Paperclip](../../how-to/update-paperclip.md)。

---

## 查看日志

```sh
paperclipai service logs
paperclipai service logs --follow
paperclipai service logs --lines 500
```

| 标志 | 用途 |
|---|---|
| `-f`、`--follow` | 持续跟踪新的日志输出。 |
| `-n`、`--lines <count>` | 显示最近多少行（默认值为 `100`）。必须为正整数。 |

日志会直接输出到终端。在 Linux 上会读取该服务的 systemd 日志；在 macOS 上会跟踪实例 `logs` 目录下的服务标准输出和标准错误日志文件。

> **提示：** 在一个窗口运行 `paperclipai service logs --follow`，同时在另一个窗口触发工作，这是实时观察启动问题最快的方法。

---

## 移除服务

```sh
paperclipai service uninstall
```

`uninstall` 会停止服务、禁用登录时自动启动并删除服务定义文件。随后它还会重新检查服务管理器；如果服务竟然仍处于已加载状态，就会明确报错，避免将部分移除误报为成功。

此操作只移除服务，不会更改 Paperclip 安装、配置和数据；`paperclipai run` 仍可照常使用。若要移除 CLI 本身，请参阅[`paperclipai uninstall`](./installation.md#removing-the-managed-install)；作为自身清理流程的一部分，它也会停止并移除服务。

---

## doctor 如何检查服务

[`paperclipai doctor`](./setup-commands.md#paperclipai-doctor) 在常规检查中会检查服务，因此通常无需主动排查就能发现问题。它会验证磁盘上的定义是否与当前安装生成的内容一致、服务是否活动、健康端点是否返回正常，以及运行版本是否与托管安装一致。在 Linux 上，如果启用了登录时启动但没有启用 linger（这种组合会导致 Paperclip 在注销时悄悄停止），它也会发出警告。

未安装服务时，doctor 会通过检查并说明这一点：前台运行 Paperclip 完全是有效的选择。

---

## 另请参阅

- [安装 CLI](./installation.md)——服务运行所使用的托管安装，以及如何移除它。
- [设置命令](./setup-commands.md)——`run`、`onboard` 和 `doctor`，包括与服务交互的标志。
- [更新 Paperclip](../../how-to/update-paperclip.md)——更新作为服务运行的实例。
- [通用选项](./common-options.md)——CLI 中共享的标志。
