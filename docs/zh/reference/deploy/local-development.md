---
seo_title: 本地开发设置
seo_description: 在本地运行 Paperclip 的最快方式，默认使用内嵌 PostgreSQL、本地磁盘存储和本地可信部署模式。
---

# 本地开发

这是在本地运行 Paperclip 的最快方式。默认使用内嵌 PostgreSQL、本地磁盘存储和本地可信部署模式。

如果你想快速启动应用、重置本地安装，或启动私有网络开发实例，请阅读本页。

---

## 前置条件

- Node.js 24 或更高版本（24.11.0 或更高版本）；
- pnpm 9 或更高版本。

> **注意：** 常规本地工作流不需要另行安装 PostgreSQL。

---

## 启动服务器

```sh
pnpm install
pnpm dev
```

这会启动：

- 位于 `http://localhost:3100` 的 API 服务器；
- 由 API 服务器通过开发中间件模式提供的 UI。

首次启动时，如果未设置 `DATABASE_URL`，系统会自动使用内嵌 PostgreSQL。其背后的存储模式说明见[数据库](./database.md)。

---

## 一条命令完成引导

如果你想一步完成设置和启动，请使用：

```sh
pnpm paperclipai run
```

首次安装时最适合使用此命令，因为它会：

1. 如果不存在配置，则自动运行引导设置；
2. 启用修复功能运行 `paperclipai doctor`；
3. 检查通过后启动服务器。

如果你希望像使用普通应用一样运行 Paperclip，而不是手动拼装服务，请使用此命令。

---

## 托管开发运行器

对于当前仓库和实例，开发运行器具有幂等性。如果相应的 Paperclip 开发运行器已经在运行，Paperclip 会报告现有进程，而不会启动重复实例。

使用以下命令检查或停止运行器：

```sh
pnpm dev:list
pnpm dev:stop
```

---

## 私有网络开发

如果你希望通过 Tailscale 或其他私有网络访问本地实例，请运行带身份验证的私有开发模式：

```sh
pnpm dev --tailscale-auth
```

别名：

```sh
pnpm dev --authenticated-private
```

此命令会将实例配置为通过身份验证的私有暴露模式，并将服务器绑定到 `0.0.0.0`。

如果还需要允许其他私有主机名：

```sh
pnpm paperclipai allowed-hostname dotta-macbook-pro
```

完整流程见 [Tailscale 私有访问](./tailscale-private-access.md)。

---

## 健康检查

启动后验证实例是否运行：

```sh
curl http://localhost:3100/api/health
```

预期响应：

```json
{"status":"ok"}
```

你也可以确认 API 正在提供公司数据：

```sh
curl http://localhost:3100/api/companies
```

---

## 重置本地数据

要清除内嵌数据库并重新开始：

```sh
rm -rf ~/.paperclip/instances/default/db
pnpm dev
```

此操作只会移除本地数据库，不会更改仓库检出目录或其他工作区数据。

---

## 工作树与隔离实例

如果你在多个 Git 工作树中工作，不要让两个 Paperclip 服务器指向同一个内嵌 PostgreSQL 数据目录。

请创建一个仓库本地的隔离实例：

```sh
pnpm paperclipai worktree init
```

该命令会创建仓库本地 `.paperclip` 配置，并在 `~/.paperclip-worktrees` 下创建隔离实例，避免新工作树与默认内嵌数据库共用数据。

> **提示：** 如果要在多个分支上并行测试文档或功能，这是最安全的方式。

---

## 默认数据路径

| 数据 | 路径 |
|---|---|
| 配置 | `~/.paperclip/instances/default/config.json` |
| 数据库 | `~/.paperclip/instances/default/db` |
| 存储 | `~/.paperclip/instances/default/data/storage` |
| 密钥 | `~/.paperclip/instances/default/secrets/master.key` |
| 日志 | `~/.paperclip/instances/default/logs` |

这些路径都位于 `PAPERCLIP_HOME` 和 `PAPERCLIP_INSTANCE_ID` 指定的位置之下。如需隔离本地实例，可覆盖它们：

```sh
PAPERCLIP_HOME=/custom/path PAPERCLIP_INSTANCE_ID=dev pnpm paperclipai run
```

> **提示：** 如果你在第二个检出目录或工作树中工作，请为它指定独立的实例 ID，以免意外共用同一内嵌数据库。
