---
seo_title: 为 Paperclip 设置 HTTPS
seo_description: 通过 Tailscale 或反向代理为 Paperclip 配置可信的 HTTPS 地址，并让外部服务可以访问连接器 webhook。
---

# 设置 HTTPS

Paperclip 中的 Connections 等部分服务要求其回调地址使用 HTTPS。HTTPS 会加密流量，并为浏览器或服务提供可信证书，以确保 token、登录重定向和传入消息安全地到达正确的服务器。

**Paperclip Cloud 免费提供 HTTPS。** 你的实例已经拥有 HTTPS 地址和托管证书，无需安装或续期。

自托管 Paperclip 需要配置带可信证书的 HTTPS 地址。Tailscale 可以在无需购买域名的情况下提供此功能。如果服务器使用自己的域名，Caddy 等反向代理可以为你获取并续期证书。

## HTTPS 与公网访问并不相同

HTTPS 地址仍然可以是私有地址。连接到 Tailscale 网络的浏览器可以打开私有 HTTPS 地址，包括登录回调。Slack 这类从自己的服务器发送 webhook 的服务则需要**可从公网访问的 HTTPS 回调地址**；它无法访问 `localhost` 或私有 Tailscale 地址。

以下示例假设 Paperclip 已安装并正在端口 `3100` 上监听。分享实例前，请按照[部署模式](./deployment-modes.md)配置身份验证模式，包括身份验证密钥和看板所有权。HTTPS 会在 Tailscale 或反向代理处终止，然后代理通过同一台机器上的 HTTP 将请求转发到 Paperclip。Paperclip 本身仍可绑定到 `127.0.0.1`。

## 方案一：Tailscale

Tailscale 会将你的设备连接到称为 *tailnet* 的私有网络。**Serve** 可在该网络内提供 HTTPS 访问；**Funnel** 则让服务可以通过公网访问。Serve 会自动为完整的 `machine-name.tailnet-name.ts.net` 主机名管理证书。参见 [Tailscale Serve](https://tailscale.com/docs/features/tailscale-serve)。

### 为实例配置私有 HTTPS 地址

1. 在 Paperclip 服务器以及用于访问它的设备上[安装 Tailscale](https://tailscale.com/download)，并登录同一个 tailnet。
2. 在 Tailscale 管理控制台中启用 MagicDNS 和 HTTPS 证书。请按照 [HTTPS 设置说明](https://tailscale.com/docs/how-to/set-up-https-certificates)操作，也可以使用下方命令显示的设置链接。
3. 在 Paperclip 服务器上执行：

```sh
tailscale serve --bg --https=443 http://127.0.0.1:3100
tailscale serve status
```

使用 Tailscale 输出的 HTTPS 主机名。将该地址填入 Paperclip 现有服务的环境配置中，并将示例主机名替换为你自己的值：

```dotenv
PAPERCLIP_DEPLOYMENT_MODE=authenticated
PAPERCLIP_DEPLOYMENT_EXPOSURE=private
PAPERCLIP_BIND=loopback
PAPERCLIP_AUTH_BASE_URL_MODE=explicit
PAPERCLIP_PUBLIC_URL=https://paperclip.your-tailnet.ts.net
PAPERCLIP_ALLOWED_HOSTNAMES=paperclip.your-tailnet.ts.net
```

重启 Paperclip，然后在连接到 Tailscale 的设备上打开 HTTPS 地址。如果你正在从本地安装迁移，请完成[部署模式](./deployment-modes.md)中说明的登录和看板认领设置。已验证身份模式需要稳定的 `BETTER_AUTH_SECRET` 或现有的 `PAPERCLIP_AGENT_JWT_SECRET`。如果两者都没有，请生成强密钥并设置 `BETTER_AUTH_SECRET`；重启时必须保留此值。更改 URL 时也请保留现有密钥。有关这些设置，参见[环境变量](./environment-variables.md)。

> **注意：** 即使使用 `http://` 地址，Tailscale 也会加密其网络流量。Serve 的作用是为浏览器提供 HTTPS URL 和证书。`PAPERCLIP_PUBLIC_URL` 设置的是 Paperclip 的规范地址；它不会将私有地址变成公网地址。

### 让 Connections 接收公网 webhook

让看板继续通过端口 `443` 保持私有，并使用单独的 Funnel 端口处理回调。添加路由前，先检查 `tailscale serve status` 和 `tailscale funnel status`。Funnel 会将所选端口上的**所有路由公开到互联网**，因此请选择未使用的端口。支持的 Funnel 端口为 `443`、`8443` 和 `10000`。按照 CLI 提示为 tailnet 启用 Funnel。参见 [Tailscale Funnel](https://tailscale.com/docs/features/tailscale-funnel)。

配置 Slack 连接时，请从设置过程中显示的 webhook URL 复制路径。将下方占位内容替换为完整路径，其中包括公网端点 ID 和 `/slack` 后缀：

```sh
CHAT_WEBHOOK_PATH='/api/chat-webhooks/REPLACE_WITH_PUBLIC_ID/slack'
tailscale funnel --bg --https=8443 \
  --set-path="$CHAT_WEBHOOK_PATH" \
  "http://127.0.0.1:3100$CHAT_WEBHOOK_PATH"
tailscale funnel status
```

此命令会发布该回调路径。代理目标地址中也重复包含此路径，以确保 Paperclip 收到正确的 URL。参见 [Funnel CLI 参考](https://tailscale.com/docs/reference/tailscale-cli/funnel)。

将单独的回调来源地址添加到 Paperclip 环境配置中，然后重启：

```dotenv
PAPERCLIP_CHAT_WEBHOOK_PUBLIC_URL=https://paperclip.your-tailnet.ts.net:8443
```

此值必须仅包含 HTTPS **来源地址**，不得包含回调路径、查询参数或片段。重新打开连接设置，并使用重新生成的 webhook URL 或应用清单。连接器需要的其他回调路径也要分别添加路由。请确保服务器和 Tailscale 持续运行，以便服务提供商可以发送事件。

不要通过 Funnel 发布整个 `local_trusted` 实例，因为该模式没有看板登录。如果要公开看板，请按下文配置经过身份验证的公网部署。

## 方案二：自有域名和反向代理

对于其他自托管方案，请使用类似 `paperclip.example.com` 的域名，并配置浏览器和外部服务都信任的证书。由你在浏览器中手动接受的自签名证书不足以支持服务提供商 webhook。

Caddy 的简要设置步骤：

1. 将域名的 DNS 记录指向服务器。
2. 安装 [Caddy](https://caddyserver.com/docs/install)，允许传入连接通过端口 `80` 和 `443` 到达 Caddy，并保持 Paperclip 的端口 `3100` 为私有端口。
3. 在 Caddyfile 中加入以下内容，并替换为你的域名：

```caddyfile
paperclip.example.com {
    reverse_proxy 127.0.0.1:3100
}
```

4. 使用安装环境对应的服务管理器启动或重新加载 Caddy。Caddy 会自动获取和续期公开可信的证书，并自动将 HTTP 重定向到 HTTPS。参见 [Caddy HTTPS 反向代理指南](https://caddyserver.com/docs/quick-starts/reverse-proxy)。

配置 Paperclip 并重启：

```dotenv
PAPERCLIP_DEPLOYMENT_MODE=authenticated
PAPERCLIP_DEPLOYMENT_EXPOSURE=public
PAPERCLIP_BIND=loopback
PAPERCLIP_AUTH_BASE_URL_MODE=explicit
PAPERCLIP_PUBLIC_URL=https://paperclip.example.com
```

通过[部署模式](./deployment-modes.md)完成身份验证设置。如果看板和聊天 webhook 共用此公网来源地址，请不要设置 `PAPERCLIP_CHAT_WEBHOOK_PUBLIC_URL`。现有的 `PAPERCLIP_AUTH_PUBLIC_BASE_URL` 或 `BETTER_AUTH_URL` 覆盖值也应与新地址一致。

NGINX、其他反向代理或托管服务提供商的负载均衡器也可以承担相同角色。请配置可信证书、自动续期，并将请求代理到 Paperclip。

## 检查结果

打开 HTTPS 地址，确认没有证书警告，然后登录并执行：

```sh
curl https://YOUR_PAPERCLIP_HOST/api/health
```

对于 webhook 连接，请在连接设置中运行服务提供商验证或测试事件。自己的设备能够打开页面，只能证明浏览器访问正常；服务提供商成功回调才证明公网可达。如果验证失败，请先检查回调主机名、端口、代理路径和证书，再重新输入凭据。

完整配置参考见[环境变量](./environment-variables.md)。[Tailscale HTTPS 代理](./tailscale-https-broker.md)是用于托管工作区预览的独立功能；为 Paperclip 实例本身提供 HTTPS 时不需要它。
