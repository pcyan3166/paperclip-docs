---
seo_title: 部署模式
seo_description: 两种运行模式和一种暴露方式选择。运行模式决定看板运维人员如何进行身份验证；暴露方式决定实例保持私有还是可从公网访问。
---

# 部署模式

Paperclip 提供两种运行模式和一种暴露方式选择。运行模式决定看板运维人员如何进行身份验证；暴露方式决定实例是保持私有，还是可从公网访问。

当你要选择实例的运行方式、迁移现有安装，或检查 `paperclipai doctor` 的预期设置时，请阅读本页。

---

## 模式概要

| 模式 | 暴露方式 | 身份验证 | 常见用途 |
|---|---|---|---|
| `local_trusted` | 仅本地 | 无需登录 | 个人安装、开发、演示 |
| `authenticated` + `private` | 私有网络 | 需要登录 | Tailscale、VPN、局域网 |
| `authenticated` + `public` | 面向互联网 | 需要登录 | 托管生产环境 |

> **注意：** `local_trusted` 是默认值。如果你完全不更改设置，Paperclip 就会像可信的本地运维工具一样运行。

---

## `local_trusted`

这是最简单的运行模式。

- 看板运维人员默认受到信任。
- 实例面向单一本地用户。
- 无需经历登录流程。
- 主机通常只绑定到 `localhost`。

以下情况适合选择此模式：

- 你为自己安装 Paperclip；
- 你希望尽快启动可用实例；
- 你不会向其他用户或设备开放应用。

以下情况不要选择此模式：

- 其他人需要登录；
- 实例可通过网络访问；
- 你需要持久的部署环境。

在引导过程中设置：

```sh
pnpm paperclipai onboard
```

---

## `authenticated`

此模式要求人类用户登录后才能访问。

如果看板并非只有你一个人使用，或者实例已暴露在单台可信本地机器之外，就应选择此模式。

`authenticated` 有两种暴露策略：

### `authenticated` + `private`

用于通过 Tailscale、VPN 或局域网访问。

- 使用 Better Auth 处理登录。
- 服务器绑定所有网络接口。
- 可以根据当前网络环境推断基础 URL。
- 可能需要将私有主机名加入允许列表。

如果实例应当可由其他可信设备访问，但不能公开到互联网，请选择此模式。

引导流程示例：

```sh
pnpm paperclipai onboard
```

然后选择“已验证身份 + 私有网络”选项。

如果需要允许自定义主机名，请明确添加：

```sh
pnpm paperclipai allowed-hostname my-machine.tailnet.ts.net
```

私有网络的完整流程见 [Tailscale 私有访问](./tailscale-private-access.md)。

### `authenticated` + `public`

用于面向互联网的部署。

- 需要登录。
- 必须明确指定公网基础 URL。
- `doctor` 会运行更严格的检查。
- 应将此部署视为正式托管服务。

如果要将 Paperclip 发布到公共互联网，请选择此模式。

引导流程示例：

```sh
pnpm paperclipai onboard
```

然后选择“已验证身份 + 公网”选项。

---

## 迁移运行模式

如果最初使用 `local_trusted`，之后再迁移到需要身份验证的部署，Paperclip 会在启动时生成一次性的看板认领 URL。

认领链接格式如下：

```txt
/board-claim/<token>?code=<code>
```

已登录用户访问此链接即可认领看板所有权。该流程会：

- 将当前用户提升为实例管理员；
- 将自动创建的本地看板管理员降级；
- 保留认领用户在活动公司成员关系中的身份。

> **警告：** 请将认领 URL 视为敏感信息。它仅用于一次性所有权转移，不应随意分享。

---

## 更改模式

完成设置后，更改模式的规范方式是修改配置：

```sh
pnpm paperclipai configure --section server
```

也可以通过环境变量覆盖单次运行的模式：

```sh
PAPERCLIP_DEPLOYMENT_MODE=authenticated pnpm paperclipai run
```

> **提示：** 如果 `paperclipai doctor` 报告主机或身份验证设置有问题，请先检查部署模式。很多表面上的配置错误，实际上都是模式不匹配。
