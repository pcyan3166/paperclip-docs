---
paperclip_version: v2026.831.1
seo_title: 启用多用户登录
seo_description: 从仅限回环的受信任模式迁移，以便团队成员能够真正登录。需要更改的内容，以及该切换对现有单操作员设置的影响。
---

# 启用多用户登录

默认情况下，Paperclip 以**本地受信任**模式运行：无需登录，仅限回环地址，只有一个隐式受信任的操作员。这对于个人安装是完美的，但一旦需要第二个人接入，这种模式就变得非常糟糕。为了让团队成员能够登录，您需要将实例切换到**已认证**模式，在浏览器中一次性认领所有权，然后从应用中邀请人员。

**每个步骤发生的位置：** 一次性模式切换属于主机/配置更改——故意没有提供应用内开关来将实例切换为已认证模式，因为这属于部署决策。在此之后的所有内容——认领所有权、邀请团队成员、管理角色和权限——都在浏览器和 Paperclip UI 中进行。

**开始之前：** 您需要对运行该实例的机器拥有 shell 访问权限（模式切换和首次所有权认领故意不支持远程触发）。

---

## 1. 理解变更内容

`authenticated` 模式要求每个人类用户进行登录，由 Better Auth（电子邮件 + 密码）处理。它包含一个暴露选择：

- **`authenticated` + `private`** — 适用于 Tailscale、VPN 或局域网。服务器绑定到所有接口；私有主机名可能需要加入允许列表。
- **`authenticated` + `public`** — 适用于面向互联网的部署。公共基础 URL 必须明确指定，且 `doctor` 会执行更严格的检查。

完整对比，包括何时选择每种模式，请参阅 [部署模式](../../reference/deploy/deployment-modes.md)。

---

## 2. 切换模式（在主机上）

如果您正在设置新实例，引导向导会直接提供已认证选项：

```sh
pnpm paperclipai onboard
```

对于已经在本地受信任模式下运行的实例，请通过配置进行更改：

```sh
pnpm paperclipai configure --section server
```

您也可以使用环境变量为单次运行覆盖模式，这在测试时非常有用：

```sh
PAPERCLIP_DEPLOYMENT_MODE=authenticated pnpm paperclipai run
```

如果您选择私有网络路径，请将人们访问该实例时使用的主机名加入允许列表：

```sh
pnpm paperclipai allowed-hostname my-machine.tailnet.ts.net
```

完整的私有网络工作流程请参阅 [Tailscale 私有访问](../../reference/deploy/tailscale-private-access.md)。

---

## 3. 在浏览器中认领所有权

当实例以已认证模式重启时，回环“本地看板”占位符仍持有所有权，服务器会在日志中打印一个**一次性看板认领 URL**：

```
http://localhost:3000/board-claim/<token>?code=<code>
```

在浏览器中打开该 URL。如果提示，请登录或创建您的账户（之后页面会将您重定向回来），然后在 **Claim Board ownership** 面板上点击 **Claim ownership**。Paperclip 会在一个事务中将您提升为实例管理员，退役占位符，并使您成为每个现有公司的 `owner`。页面会显示 **Board ownership claimed** 确认信息，并提供一个进入看板的链接。

> **警告：** 请将认领 URL 视为敏感信息——这是一次性的所有权转移，不应分享。如果在使用前过期，请重启服务器以生成新的 URL。

该页面的完整操作指南请参阅 [CLI 认证与看板认领](../../administration/cli-auth.md)。

---

## 4. 配对您的 CLI（可选）

如果您计划针对该实例运行 `paperclipai` 命令，请将 CLI 与您已登录的用户配对，以避免粘贴令牌。这是一个浏览器批准的设备代码流程：

```sh
paperclipai auth login     # opens an approval page in your browser
paperclipai auth whoami    # confirms the resolved identity
```

详细信息请参阅 [CLI 认证与看板认领 → 设备代码流程](../../administration/cli-auth.md#device-code-flow-paperclipai-auth-login)。

---

## 5. 邀请您的团队成员（在应用中）

在实例已认证且所有权属于您的情况下，添加人员是常规的应用内邀请流程：**Settings → Members → Invites**，创建链接，分享它，然后从 **Settings → Members** 批准他们的加入请求。这有单独的指南：

**→ [添加人类团队成员](./add-a-human-teammate.md)**

如果您在一个实例上托管多个公司，您也可以从 **Settings → Instance: Access** 集中授予人员访问权限——请参阅 [设置](../../administration/settings.md#instance-access)。

---

## 6. 部署健全性检查

运行 doctor 以确认模式、主机和认证设置是否一致：

```sh
pnpm paperclipai doctor
```

在已认证模式下，许多“为什么无法启动”的问题实际上是模式与主机不匹配（例如，公共部署没有明确的基础 URL）。如果 `doctor` 对主机或认证提出警告，请首先检查部署模式。

---

## 相关

- [部署模式](../../reference/deploy/deployment-modes.md) — 模式和暴露的权威参考。
- [CLI 认证与看板认领](../../administration/cli-auth.md) — 看板认领页面和 CLI 登录的详细信息。
- [添加人类团队成员](./add-a-human-teammate.md) — 启用登录后的人员入职。
- [角色与权限](../../administration/roles-and-permissions.md) — 这些团队成员进入后可以执行的操作。
