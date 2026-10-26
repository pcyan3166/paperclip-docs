---
seo_title: 部署概览
seo_description: 从无需摩擦的本地使用，到需要登录的私有或公开部署，共有三种受支持的配置。先从这里开始，选择合适方案。
---

# 部署概览

Paperclip 支持三种部署配置，从无需繁琐设置的本地使用，到要求登录的私有和公开部署。

当你需要选择运行模式、了解存储和密钥的默认设置，或从本地开发迁移到更持久的部署时，请阅读本节。

---

## 部署模式

| 模式 | 身份验证 | 适用场景 |
|---|---|---|
| `local_trusted` | 无需登录 | 单人操作的本地机器 |
| `authenticated` + `private` | 需要登录 | 私有网络（Tailscale、VPN、局域网） |
| `authenticated` + `public` | 需要登录 | 面向互联网的云部署 |

## 快速对比

### 本地可信模式

- 只绑定回环地址（`localhost`）。
- 不需要人工登录流程。
- 本地启动最快。
- 最适合个人开发和试验。

### 已验证身份 + 私有网络

- 通过 Better Auth 登录。
- 绑定所有网络接口，以便通过网络访问。
- 自动基础 URL 模式降低私有访问配置的复杂度。
- 最适合通过 Tailscale 或局域网供团队使用。

### 已验证身份 + 公网

- 需要登录。
- 必须明确指定公网 URL。
- 安全检查更严格。
- 最适合云托管和面向互联网的部署。

---

## 选择模式

- 只是试用 Paperclip：先使用 `local_trusted`。
- 要在私有网络中与团队共享：使用 `authenticated` + `private`。
- 要部署到云端：使用 `authenticated` + `public`。

从引导流程开始：

```sh
pnpm paperclipai onboard
```

也可以稍后更改模式：

```sh
pnpm paperclipai configure --section server
```

---

## 从这里开始

如果你希望用最短步骤启动可用实例，请先阅读[本地开发](./local-development.md)。

如果你正在决定如何向外部开放实例，请接着阅读[部署模式](./deployment-modes.md)。

如果你已确定部署方式，只需要了解运维细节，请跳转到：

- [数据库](./database.md)
- [存储](./storage.md)
- [密钥](./secrets.md)
- [环境变量](./environment-variables.md)
- [Docker](./docker.md)
- [设置 HTTPS](./https.md)
- [Tailscale 私有访问](./tailscale-private-access.md)

> **提示：** 在文档导航中将部署页面放在一起。这些页面作为一个完整流程阅读最容易理解。
