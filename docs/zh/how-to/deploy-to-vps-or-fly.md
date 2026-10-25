---
seo_title: 将 Paperclip 部署到 VPS 或 Fly.io
seo_description: 一份获取具备持久化数据的工作 URL 的实用指南。Fly.io 是快速路径，单台 VPS 是稳妥的备选方案，两者均使用相同的 Docker 镜像。
---

# 将 Paperclip 部署到 VPS 或 Fly.io

这是一份获取具备持久化数据的工作 URL 的实用指南。Fly.io 是快速路径；单台 VPS 是稳妥的备选方案。两者均使用相同的 Docker 镜像。

---

## 架构

```txt
        ┌─────────────────────────────┐
        │  paperclip container        │
        │  ─ web app (port 3100)      │
        │  ─ agent runners (adapters) │
        └──────┬───────────────┬──────┘
               │               │
        DATABASE_URL    PAPERCLIP_HOME volume
               │               │
        ┌──────▼─────┐   ┌─────▼─────┐
        │ Postgres   │   │ Persistent │
        │ (hosted)   │   │ disk       │
        └────────────┘   └────────────┘
```

容器内包含 API、UI 以及运行时生成的任何适配器进程（`claude`、`codex`）。Postgres 位于容器外部。上传文件、密钥和实例配置存储在挂载于 `PAPERCLIP_HOME` 的卷中。

---

## 前提条件

- 一个您控制的域名（`paperclip.example.com`）。
- 提供商凭证：Fly.io 账户 + `flyctl`，**或者**一台安装了 Docker 的小型 Linux VPS。
- 托管 Postgres（Supabase、Neon、Fly Postgres）。嵌入式 Postgres 仅适用于本地——请参阅 [数据库](../../reference/deploy/database.md)。
- 您计划运行的 LLM 适配器的提供商密钥（`ANTHROPIC_API_KEY`、`OPENAI_API_KEY`）。

---

## Fly.io（主要路径）

```sh
flyctl launch --image paperclip-local --no-deploy
flyctl volumes create paperclip_data --size 5
```

编辑 `fly.toml`，使卷挂载到 `/paperclip`，并将内部端口设置为 `3100`。设置运行时配置：

```sh
flyctl secrets set \
  HOST=0.0.0.0 \
  PAPERCLIP_HOME=/paperclip \
  PAPERCLIP_DEPLOYMENT_MODE=authenticated \
  PAPERCLIP_DEPLOYMENT_EXPOSURE=public \
  PAPERCLIP_PUBLIC_URL=https://paperclip.example.com \
  DATABASE_URL=postgres://...:5432/paperclip \
  PAPERCLIP_AGENT_JWT_SECRET=$(openssl rand -hex 32) \
  ANTHROPIC_API_KEY=sk-...
flyctl deploy
flyctl certs add paperclip.example.com
```

首次请求会针对空数据库触发模式迁移。通过 `curl https://paperclip.example.com/api/health` → `{"status":"ok"}` 进行确认。

---

## VPS（次要路径）

在安装了 Docker 的 5–10 美元/月的 VPS 上：

```sh
docker run -d --name paperclip --restart=always \
  -p 80:3100 \
  -e HOST=0.0.0.0 \
  -e PAPERCLIP_HOME=/paperclip \
  -e PAPERCLIP_DEPLOYMENT_MODE=authenticated \
  -e PAPERCLIP_DEPLOYMENT_EXPOSURE=public \
  -e PAPERCLIP_PUBLIC_URL=https://paperclip.example.com \
  -e DATABASE_URL=postgres://... \
  -e PAPERCLIP_AGENT_JWT_SECRET=... \
  -v /opt/paperclip:/paperclip \
  paperclip-local
```

使用 Caddy 或 Nginx 为其提供 TLS 支持。有关完整的环境变量列表，请参阅 [Docker 参考](../../reference/deploy/docker.md)。

---

## 数据库迁移

迁移会在容器启动时针对 `DATABASE_URL` 自动运行。如果您的提供商同时提供直接连接（`:5432`）和池化连接（`:6543`）（如 Supabase），请将应用指向池化 URL 并禁用预编译语句；针对直接 URL 运行一次性迁移。详情：[数据库](../../reference/deploy/database.md)。

---

## 智能体运行器放置

默认：**共置**在同一容器中。本地适配器（`claude_local`、`codex_local`）在心跳触发时作为子进程运行。在超出容器的 CPU/内存限制之前，这是可行的。

一旦心跳开始与 API 争用资源，请将运行器移至**独立**机器：配置适配器指向远程运行器池，或运行第二个具有相同 `DATABASE_URL` 和 `PAPERCLIP_API_URL` 的容器，并仅接受智能体流量。当拥有少量繁忙智能体时，这样做是值得的。

---

## 可观测性

- **日志。** Fly：`flyctl logs`。VPS：`docker logs -f paperclip`。健康检查：`GET /api/health`。
- **指标。** 仪表板在 `/<prefix>/agents/<key>/runs` 处展示每个智能体的运行历史、成本和预算。
- **警报。** 对 `/api/health` 的非 200 响应设置钩子以监控存活状态，并对 `paperclipai doctor` 的退出码设置钩子（从 Fly 机器 SSH 或 cron 中运行）以监控配置漂移。

---

## 备份

Postgres 备份是您的提供商的责任——请开启备份。对于公司数据和上传文件，请安排定期的高管安全导出——有关完整指南，请参阅 [备份和恢复公司](./back-up-and-restore-a-company.md)。

---

## 成本（2026 年 4 月）

小规模、低流量部署，每月费用：

| 组件 | Fly.io | VPS |
|---|---|---|
| 计算 | $5–10（shared-cpu-1x, 1GB） | $5–10（Hetzner CX22, DO basic） |
| Postgres | $0（Neon/Supabase 免费层）– $25（Fly Postgres 开发版） | 相同 |
| 卷 / 磁盘 | ~$1（5GB） | 包含在内 |
| 带宽 | 低使用量通常免费 | 包含在内 |
| **总计** | **~$5–35/月** | **~$5–35/月** |

请与您的提供商核实当前费率——这些是截至 2026-04-27 的列表价格，而非承诺价格。

---

## 另请参阅

- [备份和恢复公司](./back-up-and-restore-a-company.md) — 夜间导出例程和灾难恢复流程。
- [导出与导入](../guides/power/export-import.md) — 包格式和 CLI 快捷方式。
- [调试卡住的心跳](./debug-stuck-heartbeat.md) — 当智能体行为异常时，首先检查的地方。
- [部署概述](../../reference/deploy/overview.md) — 模式、存储和密钥的完整参考。
