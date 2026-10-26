---
paperclip_version: v2026.824.0
seo_title: Tailscale HTTPS 代理
seo_description: 为托管工作区中的开发服务器提供真正有效、证书可信的 HTTPS 预览 URL，而不只是无法验证或无法访问的链接。
---

# Tailscale HTTPS 代理

如果你希望 Paperclip 实例为其托管工作区中的开发服务器提供真正有效、证书可信的 `https://` 预览 URL，而不仅仅是回环地址，请阅读本页。

该功能需要按服务单独选择启用，并依赖一个运行在主机侧的小型辅助程序：**Tailscale HTTPS 代理**。本页面向负责安装和运行该代理的运维人员。

---

## 代理的用途

工作区运行时服务选择通过 HTTPS 暴露时，Paperclip 需要在本地 Tailscale 节点上发布 HTTPS 监听器，并将请求转发到服务的回环端口。此发布操作会修改主机的 Tailscale 状态，因此 Paperclip 不会直接从应用进程执行，而是委托给专用代理。该代理作为独立的最小权限主机服务运行，只负责管理由 Paperclip 拥有、仅限 tailnet 的 HTTPS 到回环映射，不做其他事情。

将代理分离运行是此设计的核心。应用无法直接修改 Tailscale；代理只接受范围狭窄且经过验证的请求；它可以操作的对象也被刻意限制在很小的范围内。

---

## 为什么代理独立运行

代理采用默认拒绝策略。它只管理由 Paperclip 创建的映射，并拒绝超出范围的操作：

- **绝不会接触 `:443`。** 实例的主路由 `:443 → 127.0.0.1:3100` 保持不变；代理会直接拒绝端口 `443`、其他特权端口和保留端口，以及运维人员明确保护的端口。
- **只管理 Paperclip 拥有的映射。** 已有的 Tailscale Serve 路由和你自行设置的服务都不在管理范围内；代理不会删除或重写这些配置。
- **失败时关闭。** 配置缺失或不安全时，代理会拒绝启动，而不会静默运行且实际上未保护任何内容。配置为 HTTPS 的预览绝不会在回退到纯 HTTP 后仍报告健康。

代理使用独立账户运行，因此即使应用遭到入侵或出现漏洞，攻击者也不能悄然修改 Tailscale。应用只能向代理提出请求，代理也只执行专门设计的有限操作。

---

## 专用账户和服务单元

代理以专用系统账户 `paperclip-tsbroker` 运行，该账户没有登录 shell，并由随软件包提供的 systemd 单元（`paperclip-tailscale-https-broker.service`）管理。应用通过 Unix 套接字与代理通信，而不是通过网络。

该单元以 `User=paperclip-tsbroker` 运行，并从环境文件加载设置：

```ini
EnvironmentFile=/etc/paperclip/tailscale-https-broker.env
```

该文件只应允许 root 和代理账户读取，因为文件中指定了代理信任的身份信息。

---

## 环境文件

所有设置都从 `/etc/paperclip/tailscale-https-broker.env` 读取。四个身份变量为必填项，其余变量有安全默认值。缺少必填值，或路径、端口不安全时，代理会拒绝启动。

| 变量 | 必填 | 默认值 | 用途 |
|---|---|---|---|
| `BROKER_NODE_IDENTITY` | 是 | — | 代理用于发布 HTTPS 映射的本地 Tailscale 节点身份。 |
| `BROKER_SERVICE_UID` | 是 | — | 代理服务自身运行时使用的 UID。 |
| `BROKER_SERVICE_GID` | 是 | — | 代理服务自身运行时使用的 GID。 |
| `BROKER_RUNTIME_UID` | 是 | — | 获准发出请求的 Paperclip 运行时账户 UID。 |
| `BROKER_PROTECTED_PORTS` | 否 | （空） | 代理绝不能修改的端口。列表格式错误时，代理会拒绝启动，而不是在未保护任何端口的情况下运行。 |
| `BROKER_SOCKET_PATH` | 否 | `/run/paperclip-tailscale-broker/broker.sock` | 代理监听应用请求的 Unix 套接字路径。 |
| `BROKER_REGISTRY_PATH` | 否 | `/var/lib/paperclip-tailscale-broker/registry.json` | 代理记录其拥有映射的位置，供后续协调使用。 |
| `BROKER_AUDIT_PATH` | 否 | `/var/log/paperclip-tailscale-broker/audit.log` | 仅追加的日志，记录每次允许/拒绝决定和修改结果。 |
| `BROKER_TAILSCALE_BIN` | 否 | `/usr/bin/tailscale` | `tailscale` 可执行文件路径。必须是绝对路径，否则代理会拒绝启动。 |

---

## 安全要点

如果只能记住代理的几个特性，请记住以下内容：

- **默认拒绝。** 代理只管理 Paperclip 拥有且仅限 tailnet 的映射；其他操作一律拒绝。
- **不会触碰 `:443`。** 实例主路由受到保护；与其他特权、保留和运维人员保护的端口一样，`443` 也会被拒绝。
- **失败时关闭。** 配置错误或缺失时，代理不会启动；回退到纯 HTTP 时绝不会报告服务已就绪。
- **最小权限。** 代理使用无登录 shell 的专用 `paperclip-tsbroker` 账户、范围受限的套接字接口，并通过 `BROKER_AUDIT_PATH` 记录审计日志。

---

## 相关页面

- [Tailscale 私有访问](./tailscale-private-access.md)：通过 tailnet 访问 Paperclip 实例本身。
- [环境变量](./environment-variables.md)：查看实例环境变量完整列表。
