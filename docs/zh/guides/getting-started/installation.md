---
paperclip_version: v2026.831.1
seo_title: 安装 Paperclip：本地安装或自行托管
seo_description: 两种安装方式：通过 pnpm 和 Node.js 24 在本机运行，或将服务部署到自己的域名下。完成后都将进入同一设置流程。
---

# 安装 Paperclip

Paperclip 有两种安装方式，请根据你的使用场景选择：

- **终端**——用一条命令在本机安装并运行 Paperclip。这是个人电脑上的标准安装方式。
- **服务器 / VPS**——将 Paperclip 部署到云服务器（AWS、GCP、DigitalOcean、Hetzner 等），并通过自定义域名和 HTTPS 访问。适合团队，以及需要从任何地方访问实例的用户。

两种方式最终都会启动 Paperclip，并进入相同的设置流程，你可以在其中创建首家公司、首位智能体和第一项工作。

---

<!-- tabs: 终端, 服务器 / VPS -->

<!-- tab: 终端 -->

> **说明：** 不熟悉终端也没关系。这条路径只需一条命令，下面会逐步介绍所需设置。

---

## 第 1 步：安装 Node.js 24 或更高版本

如果尚未安装 Node.js，请从 [nodejs.org](https://nodejs.org) 下载安装程序并运行。请选择 **LTS** 版本。

检查是否已经安装，以及版本是否符合要求：

```bash
node --version
# Should print v24.11.0 or higher
```

---

## 第 2 步：安装 pnpm

```bash
npm install -g corepack
corepack enable
corepack prepare pnpm@latest --activate
```

---

## 第 3 步：运行设置命令

> **注意：** 请勿使用 `sudo` 或以 root/管理员身份运行此命令。默认设置会启动内嵌 PostgreSQL，而 PostgreSQL 不允许以管理员身份运行。本机上请使用普通用户；服务器上则先切换到专用的 `paperclip` 用户（`sudo -iu paperclip`），再运行命令。

```bash
npx paperclipai onboard --yes
```

这条命令会完成所有准备工作：下载 Paperclip，在 `~/.paperclip/` 下创建配置目录，初始化内嵌数据库，并在 `http://localhost:3100` 启动服务器。`--yes` 会接受所有默认值；省略此参数即可自定义部署模式、数据库和存储。

```
✓ Created config at ~/.paperclip/instances/default/config.json
✓ Initialised database
✓ Server running at http://localhost:3100
→ Opening Paperclip in your browser...
```

> **macOS（Apple 芯片）故障排查：** 如果设置流程在启动内嵌 PostgreSQL 时失败，通常是缺少库的符号链接。捆绑的 Postgres（`@embedded-postgres/darwin-arm64`）附带带版本号的压缩库，例如 `libzstd.1.5.7.dylib` 和 `liblz4.1.10.0.dylib`，但不包含 Postgres 所需的较短兼容符号链接（`libzstd.1.dylib`、`liblz4.1.dylib`）。在 `node_modules` 中找到 embedded-postgres 包的 `.../lib` 目录，为这些库创建缺少的链接，然后重新运行 `npx paperclipai run`：
>
> ```bash
> cd "$(dirname "$(find ~ -path '*@embedded-postgres/darwin-arm64*/lib/libzstd.*.dylib' 2>/dev/null | head -1)")"
> ln -sf libzstd.*.dylib libzstd.1.dylib
> ln -sf liblz4.*.dylib liblz4.1.dylib
> ```
>
> 不同版本中的版本号可能不同，请根据实际存在的 `.dylib` 文件创建链接。

---

## 第 4 步：打开 Paperclip

Paperclip 会自动在浏览器中打开。如果没有，请访问 [http://localhost:3100](http://localhost:3100)。

你会进入 Paperclip 的初始设置页面。目前还没有创建公司；下一篇指南会介绍这一步。

> **说明：** 重启电脑后如需再次运行 Paperclip，请在终端中运行 `npx paperclipai run`。

### 让 Paperclip 持续运行

上述命令会持续占用终端；关闭终端窗口后，Paperclip 也会停止。在刚开始体验时这样做没问题，但如果你每天都要使用 Paperclip，可以将它设为后台服务：

```bash
npx paperclipai install
paperclipai service install
```

第一条命令会正式安装 Paperclip（以后可进行更新或回滚），并询问是否将它加入 shell 的 `PATH`。请选择是，然后打开新的终端窗口，以便使用 `paperclipai` 命令。第二条命令会将 Paperclip 注册为后台服务，使它在你登录时启动，并在崩溃后自动重启。这些操作不需要管理员权限；运行 `paperclipai service status` 可查看服务状态。

完整命令列表见 [Service（服务）](../../reference/cli/service.md)；服务器部署方式见[高级部署文档](../../reference/deploy/overview.md)。

---

## 第 5 步：获取 API 密钥

智能体开始工作前需要 API 密钥。API 密钥是一种私密令牌，类似密码，用于授权智能体调用 Anthropic（Claude）或 OpenAI 等 AI 服务提供商。没有密钥，智能体就无法生成回复或执行操作。

> **费用提示：** AI 服务提供商会按用量收费。智能体每次工作都会发起有小额成本的 API 调用；费用取决于所用模型和工作量。Paperclip 支持设置预算以控制支出，请在启动智能体前考虑这一点。

选择 AI 服务提供商，并按步骤获取密钥：

<!-- tabs: Anthropic（Claude）, OpenAI -->

<!-- tab: Anthropic（Claude） -->

Anthropic 提供 Claude，它为 `claude_local` 适配器提供模型支持，也是 Paperclip 智能体最常用的选择。

1. 前往 [console.anthropic.com](https://console.anthropic.com) 创建账户或登录。
2. 在左侧边栏点击 **API Keys**。
3. 点击 **Create Key**。
4. 为密钥取一个容易识别的名称（例如“Paperclip”）。
5. 复制密钥，它以 `sk-ant-` 开头。

> **注意：** 请立即复制密钥。Anthropic 只显示一次；如果丢失，需要重新创建。

请妥善保存。下一篇指南会介绍如何在设置首位智能体时，将密钥添加为环境变量或机密信息。

<!-- tab: OpenAI -->

OpenAI 提供驱动 `codex_local` 适配器的模型。

1. 前往 [platform.openai.com](https://platform.openai.com) 创建账户或登录。
2. 点击右上角的个人资料图标，然后点击 **API keys**。
3. 点击 **Create new secret key**。
4. 输入名称（例如“Paperclip”），然后点击 **Create secret key**。
5. 复制密钥，它以 `sk-` 开头。

> **注意：** 请立即复制密钥。OpenAI 只显示一次；如果丢失，需要重新创建。

<!-- /tabs -->

现在还不需要把密钥填入 Paperclip。下一篇指南会介绍如何配置首位智能体并连接密钥。

<!-- tab: 服务器 / VPS -->

> **说明：** 这条路径适合将 Paperclip 部署到可从互联网访问、绑定域名并要求登录的服务器。你需要 Linux VPS 的 SSH 访问权限、一个已注册域名，以及基本的命令行使用经验。如果只想在自己的电脑上使用 Paperclip，请选择 **终端** 标签。

任意配有 1 个 vCPU 和 2 GB 内存的 Linux VPS 都可以开始使用。本指南以 **Ubuntu 22.04 / 24.04 LTS** 为例；AWS EC2、Google Cloud Compute Engine、DigitalOcean Droplets、Hetzner Cloud、Linode 等服务上的命令基本相同。

---

## 第 1 步：准备服务器

选择服务提供商并创建 VPS：

- **DigitalOcean**——创建运行 Ubuntu 24.04 的 Droplet；入门使用 2 GB 内存的 Basic 套餐即可。
- **AWS EC2**——启动运行 Ubuntu 24.04 的 `t3.small` 实例（ARM 可选 `t4g.small`）。
- **Google Cloud**——创建使用 Ubuntu 24.04 LTS 镜像的 `e2-small` Compute Engine 虚拟机。
- **Hetzner / Linode / Vultr**——选择每月约 5 欧元、运行 Ubuntu 24.04 的实例即可。

在服务提供商的防火墙或安全组中，开放以下入站端口：

| 端口 | 协议 | 用途 |
|---|---|---|
| 22 | TCP | SSH |
| 80 | TCP | HTTP（Let's Encrypt 验证及重定向） |
| 443 | TCP | HTTPS |

**不要**将 3100 端口开放给互联网。Paperclip 本身会绑定到 `127.0.0.1`；外部请求由 Nginx 接收。

以具有 sudo 权限的用户通过 SSH 登录：

```bash
ssh ubuntu@your.server.ip
```

---

## 第 2 步：将域名指向服务器

安装任何软件前，先为计划使用的主机名创建 DNS `A` 记录：

| 类型 | 名称 | 值 |
|---|---|---|
| A | `paperclip.example.com` | `<your server's public IPv4>`（服务器的公网 IPv4 地址） |

等待一两分钟，然后确认域名解析正确：

```bash
dig +short paperclip.example.com
# Should print your server's IP
```

第 8 步申请 HTTPS 证书时，如果 DNS 尚未指向服务器，申请会失败。因此请先确认这一设置正确。

---

## 第 3 步：安装 Node.js 24+ 和 pnpm

Paperclip 需要 **Node.js 24 或更高版本**（24.11.0 或更高）以及 **pnpm**。通过 NodeSource 安装 Node.js：

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs git ca-certificates
```

通过 Node.js 自带的 Corepack 启用 pnpm：

```bash
sudo npm install -g corepack
sudo corepack enable
corepack prepare pnpm@latest --activate
```

验证安装：

```bash
node --version     # v24.11.0 or higher
pnpm --version     # 9.x or higher
```

---

## 第 4 步：创建专用服务用户

以非 root 用户运行 Paperclip，能在出现问题时限制影响范围：

```bash
sudo useradd --system --create-home --shell /bin/bash paperclip
sudo -iu paperclip
```

除非命令前标有 `sudo`，后续安装命令均以 `paperclip` 用户身份执行。

---

## 第 5 步：以公网部署模式安装 Paperclip

在 `paperclip` 用户的主目录中，设置环境变量以声明这是可从互联网访问的实例，然后运行设置流程：

```bash
export PAPERCLIP_DEPLOYMENT_MODE=authenticated
export PAPERCLIP_DEPLOYMENT_EXPOSURE=public
export PAPERCLIP_AUTH_PUBLIC_BASE_URL=https://paperclip.example.com
export PAPERCLIP_ALLOWED_HOSTNAMES=paperclip.example.com

npx paperclipai onboard --yes
```

> **注意：** 变量名称必须准确。CLI 读取的是 `PAPERCLIP_AUTH_PUBLIC_BASE_URL`，不是 `PAPERCLIP_PUBLIC_BASE_URL` 或 `PAPERCLIP_API_URL`。如果设置了 `deploymentMode=authenticated` 和 `exposure=public` 却未设置此变量，`paperclipai doctor` 会报告配置错误 `auth.publicBaseUrl is required`，服务器也无法启动。

这些变量的作用：

- **`PAPERCLIP_AUTH_PUBLIC_BASE_URL`**——用户访问实例时使用的外部 URL。它会成为 Better Auth 的规范基础 URL，并自动将 `auth.baseUrlMode=explicit`。
- **`PAPERCLIP_ALLOWED_HOSTNAMES`**——Paperclip 接受请求的主机名列表，多个名称以逗号分隔。基础 URL 中的主机名会自动加入；如有其他别名，请一并填写（例如 `paperclip.example.com,www.paperclip.example.com`）。不在列表中的主机会被拒绝。
- 服务器默认绑定到 `127.0.0.1:3100`，这正适合在 Nginx 后运行，因此不需要覆盖 `PAPERCLIP_BIND`。如果要在局域网或 Tailnet 中开放访问，可使用 `PAPERCLIP_BIND=lan|tailnet|custom`；选择 `custom` 时还需设置 `PAPERCLIP_BIND_HOST`。

`--yes` 会接受以下快速入门默认值：**authenticated/public** 部署、**内嵌 PostgreSQL**（端口 54329，数据位于 `~/.paperclip/instances/default/db`）、**本地磁盘**存储，以及在 `~/.paperclip/instances/default/secrets/master.key` 生成新的 32 字节密钥主密钥。

> **注意：** 请将 `secrets/master.key` 备份到安全位置。它用于加密 Paperclip 保存的所有 API 密钥和机密信息；一旦丢失，你将无法再访问这些数据。

要自定义上述选项，请省略 `--yes` 并按提示操作；也可以稍后运行 `paperclipai configure --section <name>`。有效的部分名称为：`llm`、`database`、`logging`、`server`、`storage`、`secrets`。（身份验证 URL 设置属于 `server` 部分，而不是单独的 `auth` 部分；错误消息中建议使用 `--section database` 是误导。）

设置流程会在 `~/.paperclip/instances/default/config.json` 创建配置并初始化数据库。如果提示启动服务器，请在完成后按 `Ctrl+C` 停止；下一步会改用 systemd 管理服务器。第 9 步将使用 `paperclipai auth bootstrap-ceo` 创建首位用户的邀请链接。

---

## 第 6 步：使用 systemd 运行 Paperclip

以 `paperclip` 用户身份写入环境文件，让 systemd 读取相同配置：

```bash
cat > ~/paperclip.env <<'EOF'
PAPERCLIP_DEPLOYMENT_MODE=authenticated
PAPERCLIP_DEPLOYMENT_EXPOSURE=public
PAPERCLIP_AUTH_PUBLIC_BASE_URL=https://paperclip.example.com
PAPERCLIP_ALLOWED_HOSTNAMES=paperclip.example.com
EOF
chmod 600 ~/paperclip.env
```

> **说明：** `PAPERCLIP_AGENT_JWT_SECRET` 已在设置过程中写入 `~/.paperclip/instances/default/.env`，并会自动加载；不要在此重复配置。

然后退出 `paperclip` 用户会话（运行 `exit`），切换回 sudo 用户并创建服务单元：

```bash
sudo tee /etc/systemd/system/paperclip.service > /dev/null <<'EOF'
[Unit]
Description=Paperclip control plane
After=network.target

[Service]
Type=simple
User=paperclip
Group=paperclip
WorkingDirectory=/home/paperclip
EnvironmentFile=/home/paperclip/paperclip.env
ExecStart=/usr/bin/npx paperclipai run
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now paperclip
sudo systemctl status paperclip
```

如果出现异常，可查看日志：

```bash
sudo journalctl -u paperclip -f
```

你应当能看到 Paperclip 正在 `http://127.0.0.1:3100` 上监听。

---

## 第 7 步：在 Paperclip 前端配置 Nginx

安装 Nginx：

```bash
sudo apt-get install -y nginx
```

为你的域名创建站点配置：

```bash
sudo tee /etc/nginx/sites-available/paperclip > /dev/null <<'EOF'
server {
    listen 80;
    server_name paperclip.example.com;

    client_max_body_size 50m;

    location / {
        proxy_pass         http://127.0.0.1:3100;
        proxy_http_version 1.1;
        proxy_set_header   Host              $host;
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;

        # WebSocket / long-lived streaming endpoints
        proxy_set_header   Upgrade           $http_upgrade;
        proxy_set_header   Connection        "upgrade";
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }
}
EOF

sudo ln -sf /etc/nginx/sites-available/paperclip /etc/nginx/sites-enabled/paperclip
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
```

请将 `paperclip.example.com` 替换为你自己的主机名。

---

## 第 8 步：通过 Let's Encrypt 配置 HTTPS

安装 Certbot 并申请证书。Certbot 会自动修改 Nginx 配置，以处理 TLS 和 HTTP 到 HTTPS 的重定向：

```bash
sudo apt-get install -y certbot python3-certbot-nginx
sudo certbot --nginx -d paperclip.example.com
```

接受服务条款，填写用于接收证书到期提醒的电子邮箱，并在询问是否强制使用 HTTPS 时选择 **redirect（重定向）**。

Certbot 会设置 systemd 定时器，在证书到期前自动续订。确认定时器已启用：

```bash
systemctl list-timers | grep certbot
```

在浏览器中访问 `https://paperclip.example.com`，应该能看到通过 HTTPS 提供的 Paperclip 登录页面。

---

## 第 9 步：创建 CEO 账户

由于你设置了 `PAPERCLIP_DEPLOYMENT_MODE=authenticated`，访问实例需要登录。首位用户通过 CLI 生成的一次性邀请链接创建。

以 `paperclip` 用户身份生成邀请：

```bash
sudo -iu paperclip
npx paperclipai auth bootstrap-ceo
```

命令会输出 **Invite URL（邀请链接）**，格式如下：

```
Invite URL: https://paperclip.example.com/invite/<token>
```

> **说明：** `bootstrap-ceo` 仅在启用了身份验证的模式下工作，并且需要连接数据库。如果使用内嵌 PostgreSQL，请确保运行此命令时 `paperclip` systemd 服务已启动，否则数据库文件锁可能被其他进程占用。

在浏览器中打开邀请链接，通过 Better Auth 使用电子邮箱和密码创建账户，即可作为 CEO/所有者进入实例。

如果链接丢失，可使用 `--force` 重新生成令牌：

```bash
npx paperclipai auth bootstrap-ceo --force
```

其他可选参数包括：`--expires-hours N` 用于修改链接有效期，`--base-url <URL>` 用于覆盖邀请链接中的 URL，`--db-url <URL>` 用于连接外部数据库。

---

## 第 10 步：获取 API 密钥

智能体要开始工作仍需 Anthropic 或 OpenAI 密钥。按照“终端”标签页中的**第 5 步：获取 API 密钥**操作；服务器部署的步骤相同。登录后将密钥填入 Paperclip 的 Secrets（机密信息）界面。密钥会使用第 5 步的主密钥加密，并由适配器配置引用。

---

## 故障排查

遇到问题时，可以使用以下诊断命令：

- `paperclipai doctor`——验证配置和环境。在运行服务前使用可及早发现配置模式错误；传入 `--repair` 可自动修复能够处理的问题。
- `paperclipai env`——打印 Paperclip 实际读取的环境变量，以确认导出的变量已生效。
- `paperclipai allowed-hostname <host>`——安装后将主机名加入 `server.allowedHostnames`，例如添加第二个域名。
- `paperclipai configure --section server`——重新配置服务器和身份验证设置（绑定地址、访问范围、公开基础 URL、允许的主机名），不必重建全部配置。
- `sudo journalctl -u paperclip -f`——持续查看服务器日志。

常见错误：

- **`Embedded PostgreSQL failed`，并显示 `Execution of PostgreSQL by a user with administrative permissions is not permitted`**——Paperclip 是以 root 或提升权限的用户启动的。停止服务，切换到普通非 root 用户并重新运行 `npx paperclipai run`。在 VPS 上，请确保在设置或启动 Paperclip 前运行过 `sudo -iu paperclip`。如果意外以错误账户创建了文件，请删除由 root 所有的实例，或先修正文件所有权再试。
- **`auth.publicBaseUrl is required when deploymentMode=authenticated and exposure=public`**——运行 `onboard` 前没有导出 `PAPERCLIP_AUTH_PUBLIC_BASE_URL`。重新导出后运行 `paperclipai configure --section server`，或重新运行 `paperclipai onboard --yes`。
- **请求因主机名不匹配而被拒绝**——访问时所用主机名不在 `server.allowedHostnames` 中。使用 `paperclipai allowed-hostname <host>` 添加，或编辑 `~/paperclip.env` 中的 `PAPERCLIP_ALLOWED_HOSTNAMES` 并重启服务。
- **邀请链接返回 404**——邀请已经使用，或打印出的基础 URL 与浏览器访问的 URL 不一致。运行 `paperclipai auth bootstrap-ceo --force --base-url https://paperclip.example.com` 重新生成。

---

## 常见变体

- **托管 PostgreSQL**——在运行设置流程前，于 `~/paperclip.env` 中设置 `DATABASE_URL=postgres://...`。应用使用连接池连接（Supabase 的端口为 6543），迁移使用直连地址。详见[数据库部署](../../reference/deploy/database.md)。
- **对象存储**——设置 `PAPERCLIP_STORAGE_MODE=s3` 和相应的 S3 环境变量。详见[存储部署](../../reference/deploy/storage.md)。
- **使用 Tailscale 提供私有团队服务器**，而非公开域名——跳过 Nginx 和 Certbot，设置 `PAPERCLIP_DEPLOYMENT_EXPOSURE=private` 和 `PAPERCLIP_BIND=tailnet`。详见 [Tailscale 私有访问](../../reference/deploy/tailscale-private-access.md)。在启用身份验证的私有实例上，也可以不通过 CLI 邀请来创建首位用户：第一个从浏览器登录的人可以直接在界面中认领实例管理员身份，先认领者优先，其他人之后无法认领。详细说明见[认领首位实例管理员](../../reference/api/instance-admin.md#claim-first-instance-admin)。
- **使用 Docker，而非直接安装在服务器上**——仓库提供了可用于生产环境的镜像和 Compose 文件。详见 [Docker 部署](../../reference/deploy/docker.md)。

---

<!-- /tabs -->

---

## 已完成安装

Paperclip 已经运行。下一篇指南会带你创建首家公司；如果已经准备好目标，也可以在创建后设置，并为智能体做好准备。

[创建你的第一家公司 →](./your-first-company.md)

---

> **说明：** macOS 还有一个**非官方、由社区维护的桌面应用**，可以将 Paperclip 包装成普通 Mac 应用。它并非由 Paperclip 团队构建或支持。详情见[社区桌面应用](../../how-to/community-desktop-app.md)。
