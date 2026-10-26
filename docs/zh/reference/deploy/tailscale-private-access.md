---
seo_title: Tailscale 私有访问
seo_description: 通过私有网络而非仅限 localhost 访问 Paperclip 实例，是部署需要身份验证的私有实例的实用方式。
---

# Tailscale 私有访问

如果你希望 Paperclip 实例可从私有网络访问，而不仅限于 `localhost`，请阅读本页。

这是部署 `authenticated` + `private` 模式的实用方法。

如果需要浏览器信任的 HTTPS 地址，或需要为 Connections 提供公网回调，请遵循[设置 HTTPS](./https.md)。本页介绍通过 HTTP 直接进行私有网络访问。

---

## 以私有模式启动

使用带身份验证的私有开发模式启动 Paperclip：

```sh
pnpm dev --tailscale-auth
```

等效参数：

```sh
pnpm dev --authenticated-private
```

此命令会配置：

- `PAPERCLIP_DEPLOYMENT_MODE=authenticated`
- `PAPERCLIP_DEPLOYMENT_EXPOSURE=private`
- `PAPERCLIP_AUTH_BASE_URL_MODE=auto`
- `HOST=0.0.0.0`

> **注意：** 绑定地址很重要。如果主机仍绑定在 `localhost`，私有网络中的其他设备将无法访问应用。

---

## 查找可访问地址

在运行 Paperclip 的机器上执行：

```sh
tailscale ip -4
```

你也可以使用 MagicDNS 主机名，例如 `my-macbook.tailnet.ts.net`。

---

## 打开实例

使用私有网络主机名或 IP 地址，并附上 Paperclip 端口：

```txt
http://<tailscale-host-or-ip>:3100
```

例如：

```txt
http://my-macbook.tailnet.ts.net:3100
```

---

## 允许自定义主机名

如果使用 Paperclip 尚未见过的私有主机名，请将其加入允许列表：

```sh
pnpm paperclipai allowed-hostname my-macbook.tailnet.ts.net
```

如果应用重定向错误，或拒绝私有网络中有效的主机名，请使用此命令。

---

## 验证连通性

在另一台已连接 Tailscale 的设备上执行：

```sh
curl http://<tailscale-host-or-ip>:3100/api/health
```

预期响应：

```json
{"status":"ok"}
```

---

## 故障排查

- 如果登录或重定向错误提到主机名，请使用 `paperclipai allowed-hostname` 添加该主机名。
- 如果应用只能通过 `localhost` 使用，请确认启动时指定了 `--tailscale-auth` 或 `--authenticated-private`。
- 如果本地访问正常但远程访问失败，请确认两台设备都连接到同一个 Tailscale 网络，并检查端口 `3100` 是否可达。

> **提示：** 排查私有访问问题时，先在第二台设备上执行 `curl /api/health`，再检查浏览器重定向。这个测试可以直接确认网络是否连通。
