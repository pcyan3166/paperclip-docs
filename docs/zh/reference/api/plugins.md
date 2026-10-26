---
paperclip_version: v2026.529.0
seo_title: 插件 API
seo_description: 插件管理器背后的 REST 接口：安装、启用、禁用、升级、配置、作业、webhook 和本地文件夹开发流程。
---

# 插件 API

插件 API 是插件管理器 UI 和插件运行时桥接层背后的 REST 接口。它涵盖安装、生命周期管理（启用/禁用/升级）、配置、作业、webhook、本地文件夹挂载、仪表板数据，以及 worker 端插件向宿主发出的桥接调用。

面向运维人员的操作流程见[管理 → 插件](../../administration/plugins.md)；开发插件的 SDK 见[参考 → 插件 → 插件 SDK](../plugins/sdk.md)。

> 所有路由都挂载在 `/api` 下。大多数操作需要实例管理员权限；公司范围内的路由会在路径中明确体现。

---

## 注册表与发现

| 端点 | 用途 |
|---|---|
| `GET /api/plugins` | 列出此实例中已安装的插件。 |
| `GET /api/plugins/examples` | 列出在当前检出目录中发现的内置插件（参考示例和仓库内置的第一方插件）。详见下文。 |
| `GET /api/plugins/ui-contributions` | 汇总已启用插件提供的 UI 插槽和启动器。Paperclip UI 会在页面加载时读取此数据。 |
| `GET /api/plugins/tools` | 列出已启用插件声明的工具。 |
| `POST /api/plugins/tools/execute` | 按键执行插件声明的工具。 |

### 发现内置插件

`GET /api/plugins/examples` 返回当前 Paperclip 检出目录中随应用提供的插件。服务器不会返回固定列表，而是扫描磁盘上的内置软件包及其插件清单，因此结果会反映当前检出目录实际包含的内容，包括参考示例插件和仓库内置的第一方插件。首次扫描后会缓存结果。

每个条目的结构如下：

```json
{
  "packageName": "@paperclipai/plugin-llm-wiki",
  "pluginKey": "llm-wiki",
  "displayName": "LLM Wiki",
  "description": "…",
  "localPath": "packages/plugins/…",
  "tag": "first-party",
  "experimental": true
}
```

- 参考示例插件的 `tag` 为 `example`；仓库内置的实际插件则为 `first-party`。
- 对于尚未达到生产就绪状态的第一方插件，`experimental` 为 `true`；插件管理器会为它们显示 **Experimental** 徽标。
- 从当前检出目录安装插件时，安装流程会使用 `localPath`。

## 安装和生命周期管理

| 端点 | 用途 |
|---|---|
| `POST /api/plugins/install` | 安装插件软件包。 |
| `GET /api/plugins/:pluginId` | 读取已安装的插件记录。 |
| `DELETE /api/plugins/:pluginId` | 卸载插件。 |
| `POST /api/plugins/:pluginId/enable` | 启用插件并启动其 worker。 |
| `POST /api/plugins/:pluginId/disable` | 禁用插件、停止其 worker，并隐藏它提供的 UI 内容。 |
| `POST /api/plugins/:pluginId/upgrade` | 根据插件来源运行升级流程。 |
| `GET /api/plugins/:pluginId/health` | 读取插件 worker 报告的当前健康诊断信息。 |
| `GET /api/plugins/:pluginId/logs` | 读取插件最近的 worker 日志。 |

### 安装插件

`POST /api/plugins/install` 使用一个路由处理两种安装方式。Paperclip 根据请求正文判断安装类型。两者都需要实例管理员权限，因为安装流程会在主机上获取并检查软件包内容。

| 字段 | 类型 | 用途 |
|---|---|---|
| `packageName` | 字符串 | 必填。注册表安装时为 npm 软件包名；本地安装时为文件系统路径。 |
| `version` | 字符串 | 可选。npm 安装的目标版本，默认使用最新版本。 |
| `isLocalPath` | 布尔值 | 可选。`packageName` 是文件系统路径而非 npm 软件包名时设为 `true`。 |

安装成功后会返回插件记录，结构与 `GET /api/plugins/:pluginId` 的响应相同。

npm 安装时，如果 `packageName` 包含 `< > : " | ? *` 中的任何字符，请求会被拒绝并返回 `400` 和 `packageName contains invalid characters`。本地路径安装跳过此检查，因为有些平台上的路径需要这些字符；但本地路径会接受下文所述的更严格检查。

### 本地安装路径的检查方式

设置 `isLocalPath: true` 时，Paperclip 会先规范化 `packageName`，之后才会由其他组件处理：先将字符串解析为绝对路径，再解析为真实路径（折叠 `..` 遍历段并跟随每一级符号链接），并要求其指向一个已存在且可读取的目录。加载器只会收到这个规范路径作为 `localPath` 安装选项，因此同一文件夹的不同路径写法不会产生不同判断结果。

有两点实际影响值得了解。符号链接到的插件文件夹会按其真实目标安装，因此 Paperclip 记录和监视的也是目标路径。尚不存在的路径会立即失败并显示明确消息，而不会等到后续才出现难以理解的清单错误。

路径规范化失败时，会返回 `400`，原因位于 `error` 字段中：

| `error` | 含义 |
|---|---|
| `Invalid localPath: path contains a null byte` | 提交的字符串中包含 `\0` 字节。 |
| `Invalid localPath: path does not exist: <absolute path>` | 解析后的路径不存在。通常是拼写错误或目录尚未创建。 |
| `Invalid localPath: path is not a directory: <canonical path>` | 路径解析结果是一个文件。请指向软件包目录，而不是目录中的文件。 |
| `Invalid localPath: path is not readable: <canonical path>` | 目录存在，但服务器进程无法读取。 |

### 云托管实例上的安装来源

如果实例由 Paperclip 控制平面管理，而不是由你自行运行，则插件安装范围会收紧为明确允许列表：只能安装应用本身随附的插件。其他请求一律会收到 `403`。

| `error` | 触发条件 |
|---|---|
| `npm installs are disabled on cloud-managed instances; only plugins bundled with the application may be installed` | 请求没有设置 `isLocalPath: true`，因此属于注册表安装。 |
| `cloud-managed instances may only install plugins from the bundled plugin catalog` | 请求指定了本地路径，但其规范路径不在内置插件目录根路径（应用软件包中的 `packages/plugins`）下。目录根路径本身不算；来源必须是目录中的软件包文件夹。 |

在这类实例上，实用的可安装插件列表可通过 [`GET /api/plugins/examples`](#发现内置插件) 查看。此端点扫描同一个目录根路径；每项的 `localPath` 都是允许列表接受的路径。自托管实例不受影响：服务器文件系统中任何可读取的目录都是有效本地安装来源，npm 安装也照常工作。上述路径规范化规则始终适用。

## 配置

| 端点 | 用途 |
|---|---|
| `GET /api/plugins/:pluginId/config` | 读取解析后的实例配置。 |
| `POST /api/plugins/:pluginId/config` | 写入新的实例配置。 |
| `POST /api/plugins/:pluginId/config/test` | 验证候选配置但不保存，并调用插件的 `validateConfig` RPC。 |

## 作业与 webhook

| 端点 | 用途 |
|---|---|
| `GET /api/plugins/:pluginId/jobs` | 列出插件声明的作业。 |
| `GET /api/plugins/:pluginId/jobs/:jobId/runs` | 列出某个作业最近的运行。 |
| `POST /api/plugins/:pluginId/jobs/:jobId/trigger` | 手动触发作业运行。 |
| `POST /api/plugins/:pluginId/webhooks/:endpointKey` | 将传入 webhook 交付给插件 worker 的 `handleWebhook`。 |

## 本地文件夹

插件可以声明本地文件夹挂载。UI 和 CLI 使用以下端点按公司检查和配置这些挂载。

| 端点 | 用途 |
|---|---|
| `GET /api/plugins/:pluginId/companies/:companyId/local-folders` | 列出插件/公司组合已配置的本地文件夹挂载。 |
| `GET /api/plugins/:pluginId/companies/:companyId/local-folders/:folderKey/status` | 读取单个挂载的状态（已配置/存在问题/列表摘要）。 |
| `POST /api/plugins/:pluginId/companies/:companyId/local-folders/:folderKey/validate` | 验证候选文件夹路径，但不保存。 |
| `PUT /api/plugins/:pluginId/companies/:companyId/local-folders/:folderKey` | 保存此挂载的文件夹配置。 |

## 仪表板

| 端点 | 用途 |
|---|---|
| `GET /api/plugins/:pluginId/dashboard` | 读取插件向 UI 提供的仪表板数据。 |

## Worker 桥接

以下端点支撑 worker 到主机的桥接调用：插件 worker 通过这些调用请求主机提供数据、执行操作或打开数据流。这些接口通常由运行时使用，但也是可用于测试和重放的有效 REST 端点。

| 端点 | 用途 |
|---|---|
| `POST /api/plugins/:pluginId/bridge/data` | 处理 worker 发起的 `data` 请求。 |
| `POST /api/plugins/:pluginId/bridge/action` | 处理 worker 发起的 `action` 请求。 |
| `GET /api/plugins/:pluginId/bridge/stream/:channel` | 从 worker 打开数据流通道。 |
| `POST /api/plugins/:pluginId/data/:key` | 面向 UI，按键读取插件已注册的数据动态流。 |
| `POST /api/plugins/:pluginId/actions/:key` | 面向 UI，调用插件已注册的操作。 |

> **稳定性说明：** 插件运行时仍处于 Alpha 阶段。不同 Paperclip 版本之间的端点结构可能变化；请确保插件和 Paperclip 版本同步固定。
