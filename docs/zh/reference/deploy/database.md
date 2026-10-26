---
paperclip_version: v2026.1023.0
seo_title: 数据库配置
seo_description: Paperclip 通过 Drizzle ORM 使用 PostgreSQL。所有受支持模式使用同一套 schema，差异在于 PostgreSQL 的运行位置以及连接方式。
---

# 数据库

Paperclip 通过 Drizzle ORM 使用 PostgreSQL。所有受支持的数据库模式都使用同一套 schema；区别在于 PostgreSQL 运行在哪里，以及 Paperclip 如何连接它。

如果你正在选择内嵌数据库、本地 Docker 数据库或托管 PostgreSQL，请阅读本页。

---

## 数据库模式

| 模式 | 适用场景 | 配置成本 |
|---|---|---|
| 内嵌 PostgreSQL | 本地开发、一条命令即可安装 | 无需配置 |
| Docker 中的本地 PostgreSQL | 类似生产环境的本地数据库 | 低 |
| 托管 PostgreSQL | 生产和共享部署 | 较高 |

> **注意：** 应用逻辑在不同模式下不变。只有 `DATABASE_URL` 和存储位置会变化。

---

## 内嵌 PostgreSQL

未设置 `DATABASE_URL` 时，默认使用此模式。

```sh
pnpm dev
```

首次启动时，Paperclip 会：

1. 创建 `~/.paperclip/instances/default/db/`；
2. 确保 `paperclip` 数据库存在；
3. 对空数据库运行迁移；
4. 开始处理请求。

数据会保存在该目录中，重启后仍然存在。

如果你希望本地配置尽可能简单，请使用此模式。

删除数据目录即可重置本地数据库：

```sh
rm -rf ~/.paperclip/instances/default/db
```

Docker 快速入门默认也使用内嵌 PostgreSQL。容器设置见 [Docker](./docker.md)。

---

## Docker 中的本地 PostgreSQL

如果你希望在本地使用标准 PostgreSQL 服务器，而不是内嵌运行时，请选择此方式。

启动数据库：

```sh
docker compose up -d
```

这会在 `localhost:5432` 启动 PostgreSQL 17。

在环境中设置连接字符串：

```sh
cp .env.example .env
# DATABASE_URL=postgres://paperclip:paperclip@localhost:5432/paperclip
```

然后推送 schema：

```sh
DATABASE_URL=postgres://paperclip:paperclip@localhost:5432/paperclip \
  npx drizzle-kit push
```

如果你希望更接近常规 PostgreSQL 部署，同时仍在本地运行，请使用此模式。

---

## 托管 PostgreSQL

生产环境应使用托管 PostgreSQL 服务提供商，例如 [Supabase](https://supabase.com/)。

典型设置流程：

1. 在 [database.new](https://database.new) 创建项目；
2. 从数据库设置中复制连接字符串；
3. 在部署环境中设置 `DATABASE_URL`。

Supabase 提供两种常用连接方式：

- 端口 `5432` 上的直接连接，用于迁移和一次性脚本；
- 端口 `6543` 上的连接池，用于运行中的应用。

如果使用连接池，请在 `packages/db/src/client.ts` 中禁用预处理语句：

```ts
export function createDb(url: string) {
  const sql = postgres(url, { prepare: false });
  return drizzlePg(sql, { schema });
}
```

> **警告：** 修改 schema 时使用直接连接。将迁移指向连接池端点，是造成可避免连接错误的常见原因。

---

## 切换模式

| `DATABASE_URL` | 结果 |
|---|---|
| 未设置 | 内嵌 PostgreSQL |
| `postgres://...localhost...` | Docker 中的本地 PostgreSQL |
| `postgres://...supabase.com...` | 托管 PostgreSQL |

无论采用哪种模式，`packages/db/src/schema/` 下的 Drizzle schema 都保持不变。

## 数据库备份会保留哪些内容

普通实例数据库备份包括加密的[智能体身份](../../guides/org/agent-identity.md)以及本地存储的[智能体问题反馈与建议](../../administration/agent-commentary.md)。恢复身份还需要与之匹配的密钥加密密钥：请另外保留 `PAPERCLIP_SECRETS_MASTER_KEY` 或实例配置的主密钥文件。

数据库不包含持久化智能体文件或本地上传文件的实际内容。还需备份这些文件系统目录或配置的资源存储。参见[存储](storage.md#persistent-agent-files)。

可移植公司导出在导入时会创建新的身份。数据库恢复会保留原实例的身份，因此灾难恢复备份应用于恢复该实例，而不应将其用于创建独立的智能体副本。

> **提示：** 如果部署运行正常，但数据总是消失，请检查是否仍在使用内嵌 PostgreSQL。此模式有意设计为本地运行，数据只持久保存在实例目录中。
