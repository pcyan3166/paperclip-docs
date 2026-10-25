---
paperclip_version: v2026.513.0
seo_title: 在本地开发插件
seo_description: 从笔记本电脑上的文件夹构建插件，直接从磁盘将其安装到运行中的实例中，并在每次保存时通过工作区重新加载进行迭代。
---

# 在本地开发插件

从笔记本电脑上的文件夹构建 Paperclip 插件，直接从磁盘将其安装到运行中的 Paperclip 实例中，并在每次保存后通过工作区重新加载进行迭代。在新鲜检出的代码库中端到端完成大约需要 15 分钟——其中大部分时间用于获取第一个 `pnpm install`。

其心智模型与普通安装无异：Paperclip 是控制平面，而插件是一个自包含的包，它自带工作区、清单以及（可选的）UI。本地开发循环唯一增加的是，Paperclip 从您机器上的路径读取该包，而不是从 npm 注册表读取，并且它会监视该包的 `dist/` 输出，因此您无需在每次代码更改后重新安装。

> **面向所有人，而不仅仅是开发者。** 如果您想*使用*他人已发布的插件，请参阅 [管理 → 插件](../../administration/plugins.md)。本页适用于您想要编写或分支（fork）一个插件的时刻。

---

## “本地路径安装”的实际含义

当您运行 `paperclipai plugin install <path>` 时，CLI 与 `paperclipai plugin install @scope/name` 安装相比，在两个方面有所不同：

1. 它会自动检测参数是本地文件系统路径（任何以 `/`、`./`、`../`、`~` 开头，或相对于当前目录的现有文件夹），并在将其发送到服务器之前将其解析为绝对路径。
2. 它将请求中的 `isLocalPath: true` 设置为 `POST /api/plugins/install`，这告诉 Paperclip 从磁盘读取包，而不是从 npm 获取。

如果您希望更明确，可以使用 `--local` 强制进行本地路径解释。CLI 拒绝在本地路径安装中使用 `--version`——版本固定仅适用于 npm 包。

在服务器端，Paperclip 在从路径加载任何内容之前会规范化该路径：该值会被解析为绝对路径，然后解析为其真实路径——折叠 `..` 段，跟随符号链接——并且它必须是服务器可读的目录。普通的 `paperclipai plugin install ~/dev/my-plugin` 不受此影响，并且对文件夹在文件系统中的*位置*没有限制。不过有两点值得注意。符号链接的插件文件夹会安装为其真实目标，因此这是 Paperclip 记录并监视的路径。如果路径不是可读目录，它会立即以 `Invalid localPath: …` 失败，而不是在后续产生令人困惑的清单错误。

本地路径安装完全不工作的唯一情况是，由 Paperclip 控制平面管理而非由您自行运行的实例——这些实例仅接受随应用程序捆绑的插件。请针对您自行运行的实例进行开发，这也是本指南其余部分所假设的情况。

本地路径插件作为受信任的本地代码运行，与运行中的 Paperclip 实例的其余部分处于相同的信任边界内。工作区代码没有沙箱隔离，除了“Paperclip 可以读取您提供的路径”之外没有签名检查。不要本地安装您未编写或未审计的插件。

---

## 先决条件

- Node.js 24（24.11.0 或更高版本）和 `pnpm`。
- 一个可以从源代码运行的本地 Paperclip 检出。本地插件安装需要运行中的 Paperclip 服务器能够读取您指向的路径——这几乎总是与您的编辑器位于同一台机器上。
- 您 `PATH` 上的 Paperclip CLI。如果您还没有，请参阅 [安装](../guides/getting-started/installation.md)。

---

## 1. 启动 Paperclip

```bash
pnpm paperclipai run
```

Paperclip 默认监听 `http://127.0.0.1:3100`。CLI 与该服务器通信，因此请将其保留在单独的终端中运行。

---

## 2. 生成插件脚手架

```bash
paperclipai plugin init @acme/hello-plugin --output ~/dev/paperclip-plugins
```

`plugin init` 会在 `--output` 下创建一个以包名命名的文件夹（去掉 npm 作用域）。因此，使用 `--output ~/dev/paperclip-plugins` 的 `@acme/hello-plugin` 会落在 `~/dev/paperclip-plugins/hello-plugin/`。在其中，您将获得一个 `src/manifest.ts`、一个 `src/worker.ts`、一个入门 `src/ui/index.tsx`、一个 esbuild 监视配置、一个 Vitest 配置，以及来自您 Paperclip 检出的 `@paperclipai/plugin-sdk` 快照，以便包可以在不发布到 npm 的情况下构建。

首次生成时重要的标志：

| 标志 | 作用 |
|---|---|
| `--output <dir>` | 创建插件文件夹的父目录。默认是当前目录。 |
| `--template <default\|connector\|workspace\|environment>` | 入门形状。`default` 是一个良好的首个插件；`connector` 添加出站集成脚手架；`workspace` 添加项目范围界面；`environment` 生成环境驱动插件。 |
| `--category <connector\|workspace\|automation\|ui\|environment>` | 清单类别——显示在插件管理器页面上的芯片。 |
| `--display-name <name>` | 清单显示名称。 |
| `--description <description>` | 清单描述。 |
| `--author <author>` | 清单作者。 |
| `--sdk-path <path>` | 指向特定 `packages/plugins/sdk` 检出的绝对路径。如果您有多个 Paperclip 克隆，这很有用。 |

当 `plugin init` 完成时，它会逐字打印接下来的四个命令。您可以直接从终端复制它们——它们是接下来两个步骤中的相同命令。

---

## 3. 安装依赖并运行监视构建

```bash
cd ~/dev/paperclip-plugins/hello-plugin
pnpm install
pnpm dev
```

`pnpm dev` 对插件源运行 `esbuild --watch`，并将构建输出输出到 `dist/`：`dist/manifest.js`、`dist/worker.js` 和 `dist/ui/`。请保持其运行。每次您在 `src/` 下保存 `.ts` 或 `.tsx` 文件时，esbuild 都会重写相应的输出文件。

如果您正在迭代 UI 并希望浏览器中进行热模块替换，请在第二个终端中运行 `pnpm dev:ui`。它在 `http://127.0.0.1:4177` 上提供 `dist/ui/`。这是可选的——Paperclip 可以直接从 `dist/ui/` 读取构建后的 UI，而无需开发服务器。

---

## 4. 从绝对路径安装

在第三个终端中（第一个终端运行 Paperclip，第二个终端运行 `pnpm dev`），将 CLI 指向插件文件夹：

```bash
paperclipai plugin install ~/dev/paperclip-plugins/hello-plugin
```

您会看到类似以下内容：

```
Installing plugin from local path: /Users/you/dev/paperclip-plugins/hello-plugin
✓ Installed acme.hello-plugin v0.1.0 (ready)
Local plugin installs run trusted local code from your machine.
Keep `pnpm dev` running in /Users/you/dev/paperclip-plugins/hello-plugin; Paperclip watches rebuilt dist output and reloads the plugin worker.
```

底部的提示是需要内化的部分。Paperclip 将继续监视包的 `dist/` 目录；您只需保持监视构建向其提供数据即可。

相对路径也可以工作——从插件文件夹内部使用 `paperclipai plugin install .` 是相同的安装。如果您希望更明确，`--local` 无论参数形状如何都强制进行本地路径解释。

---

## 5. 确认和仪表板检查

```bash
paperclipai plugin list
paperclipai plugin inspect acme.hello-plugin
```

`list` 为每个已安装的插件打印一行，包含 `key=`、`status=`、`version=` 和数据库 `id=`。`inspect` 添加完整的最后错误（如果有记录）。两者都接受 `--json` 用于脚本化。

在浏览器中打开 **设置 → 插件** 处的插件管理器，您将在已安装列表中看到相同的插件，并在权限侧边栏中显示清单声明的任何功能。详情页面的状态选项卡是工作区进程、最近的作业运行和滚动日志出现的地方——在迭代时将其保留在一个标签页中。

---

## 重新加载循环

这是使本地开发变得愉快的部分。Paperclip 监视包 `paperclipPlugin` 字段中声明的运行时入口点——默认情况下是 `dist/manifest.js`、`dist/worker.js` 和 `dist/ui/`。当文件更改时，监视器会短暂去抖动并采取行动：

- **工作区代码。** 在 `src/` 下保存 `.ts` 文件 → esbuild 重写 `dist/worker.js` → Paperclip 重启插件工作区。下一次工作区调用使用新代码。工作区代码没有进程内热重载——它是干净的重启。
- **清单。** 保存 `src/manifest.ts` → `dist/manifest.js` 重写 → 工作区重启，主机重新读取清单。
- **插件 UI。** 保存 `.tsx` 文件 → esbuild 重写 `dist/ui/` → Paperclip 在下次挂载时重新加载 UI 包。对于 UI 工作期间的 HMR，运行 `pnpm dev:ui` 并在开发时将清单中的 `devUiUrl` 指向 `http://127.0.0.1:4177`。
- **没有监视构建，就没有重新加载。** 监视器仅在 `dist/*` 更改时触发。如果 `pnpm dev` 已停止，源编辑永远不会到达 Paperclip——在期望更改之前，请重启 `pnpm dev`（或运行一次 `pnpm build`）。

`node_modules`、`.git`、`.paperclip-sdk` 和其他点文件夹被监视器忽略。添加新依赖项需要实际从代码中导入它，并让监视构建在工作区看到它之前重新构建。

Paperclip 服务器永远不会为您编译插件源。您包自己的构建脚本负责该步骤。

---

## 在迭代之间清理

一旦您开始循环，您会用到几个 CLI 命令：

```bash
paperclipai plugin disable acme.hello-plugin    # pause without uninstalling
paperclipai plugin enable acme.hello-plugin     # bring it back
paperclipai plugin uninstall acme.hello-plugin  # remove the install record
paperclipai plugin uninstall acme.hello-plugin --force  # also purge state and config
```

`disable` 是您想要暂时停止工作区时的正确操作——配置和插件存储的任何数据都会保留。`uninstall` 删除安装记录，但默认保留存储状态；`--force` 清除所有内容，以便下次安装从头开始。

要浏览捆绑的示例插件（当您想要借鉴清单形状或界面贡献时很有用），请运行：

```bash
paperclipai plugin examples
```

每一行都包含一个可运行的 `paperclipai plugin install <path>` 行。

---

## 何时切换到 npm 安装

本地路径安装和 npm 安装命中相同的端点，但它们的含义不同：

- **本地路径** 是您控制的检出中的受信任本地代码。适用于开发或针对您自己的克隆进行操作。没有签名检查，除了路径之外没有来源元数据。
- **npm 包** 是可部署的工件。`paperclipai plugin install @acme/plugin-hello`（如果您想固定版本，则加上 `--version`）从您配置的注册表安装，生成其他操作员可以重现的安装记录，并且是发布的形式。

当您完成迭代后，发布该包并从 npm 形式重新安装，以便安装反映您实际将部署的内容。

---

## 故障排除

**安装返回 `error` 状态。** 运行 `paperclipai plugin inspect <key>` 查看最后错误。三个常见的嫌疑对象：(1) 插件尚未构建——先运行 `pnpm dev` 或 `pnpm build`；(2) `package.json` 中的 `paperclipPlugin` 条目指向磁盘上不存在的文件；(3) 清单验证失败。Paperclip 服务器日志包含完整的验证错误。

**安装失败，提示 `Invalid localPath: …`。** 服务器无法将您的参数转换为可读目录。`path does not exist` 意味着解析后的路径错误——检查拼写。（CLI 始终发送绝对路径。如果您改为直接调用 `POST /api/plugins/install`，则相对路径将根据服务器进程自己的工作目录进行解析。）`path is not a directory` 意味着您指向的是文件而不是包文件夹。`path is not readable` 意味着文件夹存在，但 Paperclip 服务器进程没有权限读取它。

**编辑似乎没有重新加载。** 确认 `pnpm dev` 仍在运行并写入 `dist/`。如果您重命名了入口文件，请更新 `package.json` 中的 `paperclipPlugin.manifest`、`paperclipPlugin.worker` 或 `paperclipPlugin.ui`，以便监视器针对它们。

**工作区重启但 UI 过时。** 强制重新加载页面。对于 UI 迭代期间的 HMR，运行 `pnpm dev:ui` 并将清单中的 `devUiUrl` 设置为 `http://127.0.0.1:4177`。

**路径参数在 Windows 上失败。** 对包含空格的路径加引号，并在非 bash shell 中优先使用绝对路径而不是 `~` 前缀的路径。

**`--version is only supported for npm package installs`。** 您同时传递了本地路径和 `--version`。去掉 `--version`——版本固定仅适用于 npm 包。

对于在启动时崩溃或健康检查失败的插件工作区进行更深入诊断，插件详情页面的状态选项卡是最佳的第一站——工作区状态、最近运行和滚动日志通常比在 CLI 上摸索更快地指向原因。

---

## 另请参阅

- [管理 → 插件](../../administration/plugins.md) — 日常操作插件（安装、启用/禁用、升级、权限）。
- [插件 SDK](../../reference/plugins/sdk.md) — 工作区端的编写界面：`definePlugin`、`runWorker`、`PluginContext` 客户端、清单类型。
- [编写公司技能](./write-a-company-skill.md) — 当智能体只需要指令而不是服务器代码时。
- [向智能体工具包添加 MCP 服务器](./add-mcp-server-to-agent.md) — 另一种常见的“扩展 Paperclip”路径；为智能体工具选择 MCP，为服务器界面选择插件。
