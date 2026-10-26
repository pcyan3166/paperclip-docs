---
seo_title: Paperclip 终端安装
seo_description: 使用设置命令下载 Paperclip、创建配置目录、初始化数据库并启动服务器。还介绍部署模式和相关选项。
---

# 终端安装

> **说明：** 本指南介绍更深入的设置。如果你是第一次安装 Paperclip，请先从[安装指南](../getting-started/installation.md)开始，其中逐步说明了一条命令完成安装的方式。

Paperclip 以 Node.js 服务器的形式运行。设置命令会一步完成完整配置。

---

## 前置条件

- Node.js 24 或更高版本（从 [nodejs.org](https://nodejs.org) 下载 LTS 版本）
- pnpm（包管理器）

```bash
node --version   # Should print v24.11.0 or higher
```

---

## 第 1 步：安装 pnpm

```bash
npm install -g corepack
corepack enable
corepack prepare pnpm@latest --activate
```

---

## 第 2 步：运行设置命令

```bash
npx paperclipai onboard --yes
```

这条命令会完成所有操作：下载 Paperclip，在 `~/.paperclip/` 下创建配置目录，初始化内嵌数据库，并在 `http://localhost:3100` 启动服务器。`--yes` 会接受所有默认值。

```
✓ Created config at ~/.paperclip/instances/default/config.json
✓ Initialised database
✓ Server running at http://localhost:3100
→ Opening Paperclip in your browser...
```

省略 `--yes` 可进入交互式设置，自行选择部署模式、数据库和存储配置。

---

## 第 3 步：打开 Paperclip

Paperclip 会自动在浏览器中打开。如果没有，请访问 [http://localhost:3100](http://localhost:3100)。

电脑重启后如需再次启动 Paperclip：

```bash
npx paperclipai run
```

---

## 部署模式

默认模式 `local_trusted` 在本机运行且不要求身份验证，适合单人本地使用。若要供团队访问或公开托管，请选择以下模式之一：

| 模式 | 身份验证 | 适用场景 |
|---|---|---|
| `local_trusted` | 无 | 个人、本机使用 |
| `authenticated` + `private` | 必须登录 | 私有网络（Tailscale、VPN、局域网） |
| `authenticated` + `public` | 必须登录 | 面向互联网的云端部署 |

修改部署模式：

```bash
npx paperclipai configure --section server
```

---

## 高级配置

有关持久后台运行、Docker 部署、外部数据库和云托管选项，请参阅[部署概览](../../reference/deploy/overview.md)。

---

## 已完成设置

Paperclip 已启动。你会进入初始设置流程或起始页面，接下来可以[创建你的第一家公司](../getting-started/your-first-company.md)。
