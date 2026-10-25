---
seo_title: CLI 认证与看板认领
seo_description: 命令行中的两种身份验证流程，均设计为令牌材料永不离开您的浏览器或 CLI。认领看板，通过设备代码登录，轮换令牌。
---

# CLI 认证与看板认领

Paperclip 为命令行和浏览器中的用户提供了两个独立的身份验证流程：

- **看板认领 (Board claim)** — 这是一次性的引导步骤，将经过浏览器身份验证的用户提升为全新启动的已认证实例的所有者。它将受信任的“本地看板”占位符迁移为真实的、具名的人类账户。
- **CLI 认证** — 这是日常使用的流程，它将本地的 `paperclipai` CLI 进程与已登录的看板用户配对，以便 CLI 可以代表您调用 `/api` 端点，而无需您在 shell 中输入凭据。

这两个流程都使用短生命周期的、由服务器生成的挑战（challenge）和浏览器批准页面。两者均设计为秘密材料（看板 API 令牌、所有权迁移）永不离开请求它的浏览器或 CLI 进程。

本指南将介绍这两个流程，展示批准页面，并解释如何使用生成的令牌配合 CLI 使用。

---

## 何时需要每个流程

- 您刚刚在服务器上以 **authenticated**（已认证）部署模式安装了 Paperclip，且回环信任（loopback trust）仍处于活动状态 → 运行一次 **看板认领**。
- 您希望从笔记本电脑针对该实例运行 `paperclipai` 命令 → 运行 **CLI 认证**（`paperclipai auth login`）。
- 您正在使用 **trusted**（受信任）部署模式（仅限回环的本地开发）→ 不需要这两个流程；CLI 和浏览器已经信任回环连接。

---

## 从 CLI 认领看板

仅当以下所有条件均为真时，看板认领才存在：

- 部署模式为 `authenticated`。
- 唯一的实例管理员是内置的 `local-board` 占位符（即尚未有人类认领该实例）。
- 认领挑战尚未过期（服务器保留活动挑战 24 小时，之后会自动刷新）。

当服务器启动且这些条件匹配时，它会将 **看板认领** URL 打印到服务器日志中。它看起来像这样：

```
http://localhost:3000/board-claim/<token>?code=<code>
```

`token` 是 48 个十六进制字符；`code` 是 24 个十六进制字符。它们共同标识恰好一个待处理的认领。

在浏览器中打开该 URL。如果您尚未登录，页面将要求您先登录或创建账户，并在身份验证后重定向回同一 URL。

一旦登录，页面将显示一个标题为 **认领看板所有权** 的面板，其中包含简短说明（“这将提升您的用户为实例管理员，并将公司所有权访问权限从本地受信任模式迁移。”）以及一个 **认领所有权** 按钮。

点击该按钮，服务器将在单个事务中执行以下三件事：

1. 为您的用户插入 `instance_admin` 角色（如果您还不是该角色）。
2. 删除 `local-board` 占位符管理员。
3. 遍历数据库中的每个公司，并为每个公司为您创建 `active`、`owner` 角色的成员资格，或重新激活您现有的成员资格并将角色设置为 `owner`。

在请求期间，按钮会切换为 **认领中…**。当请求成功返回时，页面切换为 **看板所有权已认领**，并提供一个打开看板的链接。内存中的挑战被标记为已认领，且无法再次使用。

如果挑战已过期，或 URL 格式错误，页面将显示 **认领挑战不可用**。重启服务器（或等待下一次刷新）以生成新的挑战。

---

## 设备代码流程：`paperclipai auth login`

对于日常 CLI 使用，该流程是一种设备代码风格的配对。CLI 创建一个挑战，打开您的浏览器，您批准它，然后 CLI 使用存储的令牌完成操作。

### 开始登录

在您希望使用 CLI 的机器上：

```sh
paperclipai auth login
```

CLI 执行以下操作：

1. 向 `/api/cli-auth/challenges` 发送 POST 请求，包含其自身的描述（`command`、`clientName`、`requestedAccess` 以及可选的 `requestedCompanyId`）。
2. 获取包含 `token`、`boardApiToken`、`approvalUrl`、`pollPath`、`expiresAt` 以及轮询间隔的挑战。
3. 将批准 URL 打印到您的终端，并尝试在默认浏览器中打开它。
4. 开始以 `pollPath` 轮询服务器，直到挑战被批准、取消或过期。

常用标志：

```sh
# Request instance-admin approval (only instance admins can approve this)
paperclipai auth login --instance-admin

# Scope to a specific company
paperclipai auth login --company-id <company-id>

# Target a non-default server
paperclipai auth login --api-base https://paperclip.example.com
```

### 在浏览器中批准挑战

您的浏览器将到达一个标题为 **批准 Paperclip CLI 访问** 的页面：

面板显示 CLI 请求的内容：

- **命令** — 发起登录的 CLI 命令（例如 `paperclipai auth login`）。
- **客户端** — CLI 发送的 `clientName`，默认为 `paperclipai cli`。
- **请求的访问权限** — 为 `Board` 或 `Instance admin`。
- **请求的公司** — 仅当 CLI 请求限定于某个公司时显示。

底部有两个按钮：

- **批准 CLI 访问** — 仅当已登录的用户被允许批准此级别的访问时才启用。实例管理员挑战需要实例管理员批准者。
- **取消** — 拒绝挑战。CLI 在下一次轮询中看到 `cancelled` 状态并以错误退出。

如果您未登录，页面首先显示 **需要登录**，并提供一个在身份验证后返回此处的按钮。如果您访问 URL 时挑战已被批准、取消或过期，页面将反映该状态，而不是提供按钮。

### 在 CLI 上完成

一旦点击 **批准**，CLI 的下一次轮询将看到状态 `approved`，使用看板 API 令牌调用 `/api/cli-auth/me` 以确认身份，并将凭据写入本地凭据存储。它打印成功摘要：

```json
{
  "ok": true,
  "apiBase": "https://paperclip.example.com",
  "userId": "usr_...",
  "approvalUrl": "https://paperclip.example.com/cli-auth/..."
}
```

如果您取消或让挑战过期，CLI 将以 `CLI auth challenge was cancelled.` 或 `CLI auth challenge expired before approval.` 报错 — 重新运行 `paperclipai auth login` 以重试。

---

## 认领后：使用和轮换令牌

### 令牌存储位置

CLI 将看板 API 令牌（及其解析出的用户 ID）存储在平台相应的配置目录中，以 `apiBase` 为键。您无需将令牌复制到任何地方 — 每个接受 `--api-base` 的 CLI 命令都会自动查找匹配的存储凭据。

### 使用令牌

登录后，CLI 在每个请求中发送 `Authorization: Bearer <stored-token>`。您可以通过运行以下命令进行验证：

```sh
paperclipai auth whoami
```

该命令调用 `/api/cli-auth/me` 并打印：

```json
{
  "user": { "id": "...", "name": "...", "email": "..." },
  "userId": "...",
  "isInstanceAdmin": true,
  "companyIds": ["..."],
  "source": "board-cli",
  "keyId": "..."
}
```

所有其他 CLI 命令 — 在 [设置命令](../../reference/cli/setup-commands.md) 和控制平面命令中记录的命令 — 除非您传递显式的 `--token` 或覆盖 `PAPERCLIP_API_KEY`，否则复用此凭据。

### 轮换和撤销

当您想要轮换凭据时，登出并重新登录：

```sh
paperclipai auth logout
paperclipai auth login
```

`paperclipai auth logout`：

1. 使用存储的令牌向 `/api/cli-auth/revoke-current` 发送 POST 请求以在服务器端撤销它。
2. 删除此 `apiBase` 的本地凭据条目。

如果服务器端撤销失败（网络错误、令牌已无效），本地凭据仍会被移除 — 您不会卡在过期的条目中。新的 `paperclipai auth login` 生成一个新的挑战和新的令牌，与旧令牌没有任何关系。

### 多个服务器和用户

凭据存储以规范化后的 `apiBase` 为键，因此一台机器可以同时持有本地开发实例、预发布服务器和生产服务器的独立凭据。在它们之间切换只需传递 `--api-base`（或在配置中设置）。为相同的 `apiBase` 以不同用户登录将替换该服务器的现有凭据。

---

## 故障排除

- **“无效的 CLI 认证 URL。”** — 您使用缺少 `id` 或 `token` 查询参数的批准页面打开了链接。重新运行 `paperclipai auth login` 以获取新链接。
- **“CLI 认证挑战不可用”** — 令牌有效，但服务器不再拥有匹配的挑战（已过期或服务器已重启）。重新开始登录。
- **“此挑战需要实例管理员访问权限。”** — 您正在批准 `--instance-admin` 挑战，但已登录的用户不是实例管理员。使用管理员账户登录并重试。
- CLI 上显示 **“CLI 认证挑战在批准前已过期。”** — 您让浏览器标签页停留时间过长。只需重新运行 `paperclipai auth login`。
- **看板认领页面显示“认领挑战不可用。”** — 一次性所有权挑战已被消耗或已过期。如果实例尚未有真实管理员，重启服务器 — 它将生成新的挑战并在日志中重新打印 URL。

---

## 下一步去哪里

- [设置命令](../../reference/cli/setup-commands.md) — 从 CLI 安装、引导和配置 Paperclip 实例。
- [认证 (API)](../../reference/api/authentication.md) — 看板用户、智能体和运行的底层认证模型。
