---
paperclip_version: v2026.626.0
seo_title: CLI 插件命令
seo_description: 管理服务器端扩展，为运行时增加连接器、工作区集成、自动化任务、仪表板和 UI 界面。
---

# 插件命令

插件在服务器端扩展 Paperclip 运行时，可添加代理和看板可以使用的连接器、工作区集成、自动化任务、仪表板和 UI 界面。这些命令可用于搭建插件脚手架、将插件安装到实例、监控运行状况、启用或禁用插件，以及从终端调用其运行时接口。编写基于 alpha SDK 的插件，或管理已安装的插件时，都可以使用这些命令。

与所有插件端点一样，实际工作都由 Paperclip 运行时执行。CLI 会注册插件、切换其生命周期状态，并代理对其工具、任务、配置和桥接通道的调用；它不会在你的机器上执行插件代码。唯一例外是**本地路径安装**，它会运行本机磁盘上的可信代码，因此这类安装会打印明确警告。

> **注意：** 除 `plugin init` 外，每条命令都会调用 API，并接受通用客户端标志。`plugin init` 完全在本机运行，不会访问 API。标志包括 `--api-base`、`--api-key`、`--context`、`--profile`、`--data-dir` 和 `--json`，详情请参阅[通用选项](./common-options.md)。

---

## 生命周期一览

插件在一组有限状态之间切换。CLI 会在人类可读输出中用颜色标记状态，在 `--json` 输出中则原样返回状态值。

| 状态 | 含义 |
|---|---|
| `ready` | 已安装、已启用且正在运行。 |
| `installed` | 已注册，但尚未完全启动。 |
| `upgrade_pending` | 有较新版本可用或已暂存。 |
| `disabled` | 已安装但关闭，当前未运行。 |
| `error` | 插件加载或运行失败。请检查 `lastError` 和日志。 |

常见的插件开发循环是 `init` → 开发 → `install`（本地路径）→ 迭代；常见的运维循环是 `list` → `inspect` → `enable` / `disable` / `uninstall`。

---

## 搭建插件脚手架——`plugin init`

从入门模板创建新的本地插件项目。命令完全在本机运行：写入文件并打印后续操作提示，不会调用 API。

```sh
paperclipai plugin init @acme/plugin-linear
paperclipai plugin init @acme/plugin-linear --template connector --category connector \
  --display-name "Linear Connector" --author "Acme"
```

项目会创建在以软件包名称命名的目录中（scope 前缀会被去掉，因此 `@acme/plugin-linear` 会生成 `plugin-linear/`）。可使用 `--output` 指定其父目录。

| 标志 | 用途 |
|---|---|
| `--output <dir>` | 创建插件目录的父目录。默认为当前目录。 |
| `--template <template>` | 入门模板：`default`、`connector`、`workspace` 或 `environment`。默认 `default`。 |
| `--category <category>` | 清单类别：`connector`、`workspace`、`automation`、`ui` 或 `environment`。 |
| `--display-name <name>` | 清单中的显示名称。 |
| `--description <description>` | 清单中的描述。 |
| `--author <author>` | 清单中的作者。 |
| `--sdk-path <path>` | 本地 `@paperclipai/plugin-sdk` 软件包路径，用于基于尚未发布的构建版本开发 SDK。 |

成功后，命令会打印脚手架路径和后续需要运行的准确命令：

```sh
cd plugin-linear
pnpm install
pnpm dev
paperclipai plugin install plugin-linear
```

开发期间请保持 `pnpm dev` 运行；Paperclip 会监视重建后的 `dist` 输出并重新加载插件工作进程。

---

## 安装插件——`plugin install`

从本地文件系统路径或 npm 软件包安装插件。CLI 会自动识别本地路径（绝对路径、以 `./`、`../` 或 `~` 开头的路径，以及已存在的相对目录），并将其解析为绝对路径，这样服务器无论从哪个目录启动都能找到插件。

```sh
paperclipai plugin install ./my-plugin                 # local path (auto-detected)
paperclipai plugin install @acme/plugin-linear         # npm package
paperclipai plugin install @acme/plugin-linear --version 1.2.0
paperclipai plugin install /abs/path/to/plugin --local # force local-path treatment
```

| 标志 | 用途 |
|---|---|
| `-l, --local` | 即使 `<package>` 看起来不像本地路径，也将其作为本地文件系统路径处理。 |
| `--version <version>` | 固定到指定 npm 版本。仅适用于 npm 软件包；与本地路径搭配会被拒绝。 |

> **警告：** 本地路径安装会在你的机器上运行可信代码。只安装你控制的路径。本地安装完成后，CLI 会提醒你继续运行 `pnpm dev`，以便 Paperclip 在重建时重新加载工作进程。

成功后，命令会显示已安装插件的 key、版本和状态。如果运行时在加载时记录了 `lastError`，CLI 会将其作为警告显示，即使安装请求返回了记录也不例外。依赖插件前请务必确认其状态为 `ready`。

---

## 检查安装目标——`plugin target`

安装插件前，可以运行 `plugin target` 确认插件命令会连接哪个 Paperclip 实例。命令会解析配置的 API URL，并探测该实例的 `GET /api/health`，以便确认连接到了正确运行环境，例如正在开发的分支运行环境，而不是过期的控制平面主机。

```sh
paperclipai plugin target
paperclipai plugin target --json
```

输出字段如下：

| 字段 | 说明 |
|---|---|
| `apiBase` | CLI 将使用的解析后 API URL。 |
| `reachable` | 实例是否响应健康检查请求。 |
| `health.status` | `GET /api/health` 返回的状态字符串。 |
| `health.version` | 实例运行的服务器版本。 |
| `health.deploymentMode` | 部署模式，例如 `local` 或 `cloud`。 |
| `health.deploymentExposure` | 部署的暴露方式，例如 `private` 或 `public`。 |

此命令接受 `--json` 和所有通用客户端标志（`--api-base`、`--api-key`、`--context`、`--profile`、`--data-dir`）。它仅用于诊断，不会修改实例中的任何内容。

---

## 列出已安装插件——`plugin list`

列出在实例上注册的所有插件，包括其 key、状态、版本和 ID。可用 `--status` 筛选。

```sh
paperclipai plugin list
paperclipai plugin list --status error
paperclipai plugin list --json
```

| 标志 | 用途 |
|---|---|
| `--status <status>` | 按状态筛选：`ready`、`error`、`disabled`、`installed` 或 `upgrade_pending`。 |

插件处于 `error` 状态时，人类可读输出会附加截断的错误信息；需要完整错误消息时，请运行 `plugin inspect`。

---

## 检查插件——`plugin inspect`

通过插件 key 或数据库 ID 查看某个插件的完整记录。如果插件带有 `lastError`，命令会在摘要下方显示完整错误文本。

```sh
paperclipai plugin inspect @acme/plugin-linear
paperclipai plugin inspect <plugin-id> --json
```

找不到插件时命令会以非零状态退出，因此可以在脚本中安全地用它检查插件是否存在。

---

## 启用和禁用——`plugin enable` / `plugin disable`

不卸载插件即可切换其运行状态。`enable` 会重新启动处于 `disabled` 或 `error` 状态的插件；`disable` 会停止运行中的插件，但保留其配置和状态。两条命令都接受插件 key 或 ID，并报告变更后的状态。

```sh
paperclipai plugin disable @acme/plugin-linear
paperclipai plugin enable @acme/plugin-linear
```

集成出现异常而你又不想丢失配置时，可先用 `disable` 暂停；排除原因后再用 `enable` 恢复。

---

## 卸载插件——`plugin uninstall`

按插件 key 或 ID 移除插件。默认操作会注销插件；添加 `--force` 可彻底清除其所有状态和配置。

```sh
paperclipai plugin uninstall @acme/plugin-linear
paperclipai plugin uninstall @acme/plugin-linear --force
```

| 标志 | 用途 |
|---|---|
| `--force` | 清除全部插件状态和配置（彻底删除），而不只是注销。 |

> **提示：** 如果以后可能重新安装同一插件并希望恢复其配置，请先普通卸载。只有确实要清空所有数据时才使用 `--force`。

---

## 发现示例——`plugin examples`

列出应用内置的示例插件。每项都会打印显示名称、插件 key、描述以及可直接运行的本地安装命令。这是端到端了解可用插件的最快方式。

```sh
paperclipai plugin examples
paperclipai plugin examples --json
```

复制命令输出中的 `paperclipai plugin install <local-path>` 行，即可直接从内置文件安装插件。

---

## 运行时接口

插件处于 `ready` 状态后，可以使用以下命令调用已注册的工具、任务、配置、仪表板及数据/操作端点。大多数命令接受 `<pluginId>`（插件 ID 或 key）；接受 POST 请求的命令通过 `--payload-json <json>` 接收请求体，默认值为 `{}`。

### 工具与扩展贡献

```sh
paperclipai plugin ui-contributions
paperclipai plugin tools
paperclipai plugin tool:execute --payload-json '{"tool":"...","input":{}}'
```

`ui-contributions` 和 `tools` 是实例范围的读取命令；`tool:execute` 会使用指定请求体运行已注册的插件工具。

### 健康状态、日志与升级

```sh
paperclipai plugin health <plugin-id>
paperclipai plugin logs <plugin-id>
paperclipai plugin upgrade <plugin-id> --payload-json '{}'
```

插件显示 `error` 时，请先查看 `health` 和 `logs`。`upgrade` 会升级处于 `upgrade_pending` 状态的插件。

### 配置

```sh
paperclipai plugin config <plugin-id>
paperclipai plugin config:set <plugin-id> --payload-json '{"configJson":{"apiKey":"..."}}'
paperclipai plugin config:test <plugin-id> --payload-json '{"configJson":{"apiKey":"..."}}'
```

`config` 读取当前配置，`config:set` 写入配置，`config:test` 则校验候选配置（例如确认凭据能否正常连接），供提交之前检查。

### 任务

```sh
paperclipai plugin jobs <plugin-id>
paperclipai plugin job:runs <plugin-id> <job-id>
paperclipai plugin job:trigger <plugin-id> <job-id> --payload-json '{}'
```

`jobs` 列出插件的自动化任务，`job:runs` 列出单项任务的运行历史，`job:trigger` 则会立即触发任务，不必等到计划时间。

### Webhook、仪表板、数据与操作

```sh
paperclipai plugin webhook <plugin-id> <endpoint-key> --payload-json '{}'
paperclipai plugin dashboard <plugin-id>
paperclipai plugin data <plugin-id> <key> --payload-json '{}'
paperclipai plugin action <plugin-id> <key> --payload-json '{}'
```

`webhook` 会向指定名称的 webhook 端点发送请求体；`dashboard` 读取插件的仪表板数据；`data` / `action` 调用插件暴露的、由 URL key 标识的读取和修改端点。

---

## 桥接通道

桥接（bridge）是插件的实时双向通道。可通过它向插件发送数据和操作，并将事件流式接收回来。

```sh
paperclipai plugin bridge:data <plugin-id> --payload-json '{}'
paperclipai plugin bridge:action <plugin-id> --payload-json '{}'
paperclipai plugin bridge:stream <plugin-id> <channel>
paperclipai plugin bridge:stream <plugin-id> <channel> --duration-ms 10000
```

`bridge:data` 和 `bridge:action` 会将请求体 POST 到桥接端点。`bridge:stream` 会在指定通道打开流式连接，并在收到数据时将原始响应直接写入 stdout。

| 标志 | 用途 |
|---|---|
| `--duration-ms <ms>` | 在指定毫秒数后停止流式读取。省略时会持续读取，直到你中断命令。 |

> **提示：** `bridge:stream` 会一直运行，直到你取消它或达到 `--duration-ms` 指定的时长，非常适合开发时跟踪插件事件。编写脚本时请设置时长，确保命令会终止。

---

## 本地文件夹绑定

部分插件会为特定公司绑定到操作员机器上的文件夹。这些命令限定在公司范围内，必须提供 `-C, --company-id <id>`。

```sh
paperclipai plugin local-folders <plugin-id> -C <company-id>
paperclipai plugin local-folder:status <plugin-id> <folder-key> -C <company-id>
paperclipai plugin local-folder:validate <plugin-id> <folder-key> -C <company-id> --payload-json '{}'
paperclipai plugin local-folder:set <plugin-id> <folder-key> -C <company-id> --payload-json '{"path":"/abs/path"}'
```

| 命令 | 用途 |
|---|---|
| `local-folders` | 列出公司中该插件的本地文件夹绑定。 |
| `local-folder:status` | 报告某个文件夹绑定的当前状态。 |
| `local-folder:validate` | 提交之前检查候选绑定。 |
| `local-folder:set` | 创建或更新绑定。此命令必须提供 `--payload-json`。 |

---

## 另请参阅

- [通用选项](./common-options.md)——所有调用 API 的插件命令都接受的客户端标志。
- [输出与脚本](./output-and-scripting.md)——使用 `--json` 编写 CLI 脚本。
- [适配器命令](./adapter.md)——配置代理在运行时使用的适配器。
- [技能命令](./skills.md)——扩展代理能力的另一种方式。
- [安装](./installation.md)——安装 CLI 及仓库内的开发别名。
