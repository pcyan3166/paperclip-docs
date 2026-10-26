---
paperclip_version: v2026.831.1
seo_title: Modal 沙盒服务提供商
seo_description: 将 Modal 沙盒配置为智能体执行环境，以便使用受管理云计算资源和一等容器镜像运行智能体。
---

# Modal

`modal` 会将 [Modal](https://modal.com/) 沙盒配置为 Paperclip 智能体执行环境。如果你需要受管理的云沙盒、一等容器镜像、可配置的网络策略，以及每个沙盒最长 24 小时的生命周期限制，又不希望自行运行基础设施，请选择此服务提供商。

Modal 是沙盒服务提供商插件，因此它通过沙盒环境上的 `provider: "modal"` 设置启用。安装后，任何支持沙盒的适配器（例如 `claude_local` 或 `cursor_local`）都可以使用 Modal 环境。

---

## 适用场景

- 希望 Paperclip 智能体按需在 Modal 沙盒中运行。
- 希望每次运行都使用特定容器镜像、工作目录和网络策略。
- 拥有 Modal 账户凭据（`tokenId` / `tokenSecret`），并可以将其作为密钥提供给 Paperclip。

## 不适用场景

- 希望在 Paperclip 主机上本地执行。请改用本地适配器。
- 已统一使用其他沙盒服务提供商（Cloudflare、Daytona、E2B、exe.dev）。请参阅[沙盒服务提供商](./sandbox-providers.md)。
- Paperclip 运行的位置无法访问 Modal API。

---

## 安装

在 [Plugins](../../administration/plugins.md) 页面中安装：

```text
@paperclipai/plugin-modal
```

主机插件安装器会在安装期间获取 Modal 的 JS SDK。无需单独配置工作区——安装完成后，在设置环境时即可选择 Modal 作为沙盒服务提供商。

### 运行时支持说明

Modal 官方 JS SDK 支持 **Node 22 或更高版本**。Paperclip 现在要求 `node >= 24.11.0`，高于其最低要求，因此插件默认会在受支持的 Node 版本上运行，过去的尽力支持限制也已不再适用。

---

## 配置

请在 **Settings → Instance settings → Environments**（设置 → 实例设置 → 环境）中配置 Modal，不要在插件的实例设置页面中配置。新增一个沙盒环境，选择 `modal` 作为服务提供商，并填写以下字段。

| 字段 | 必填 | 说明 |
|---|---:|---|
| `appName` | 是 | Modal App 名称。插件会调用 `modal.apps.fromName(appName, { createIfMissing: true })`；如果 App 尚不存在，则会在首次获取资源时创建。 |
| `image` | 是 | 传递给 `modal.images.fromRegistry()` 的容器镜像，例如 `python:3.13` 或 `node:20`。 |
| `tokenId` | 是 | Modal 令牌 ID。请以密钥引用方式提供。 |
| `tokenSecret` | 是 | 与 `tokenId` 配套的 Modal 令牌密钥。两者必须同时提供。 |
| `environment` | 否 | 可选的 Modal 环境名称。未设置时回退到 SDK 配置文件中的默认值。 |
| `workdir` | 否 | 沙盒内的远程工作目录。默认为 `/workspace/paperclip`。 |
| `sandboxTimeoutMs` | 否 | 沙盒最长生命周期，单位为毫秒。必须为 `1000` 到 `86_400_000`（24 小时）之间的正数，且为 `1000` 的倍数。默认值为 `3_600_000`（1 小时）。 |
| `idleTimeoutMs` | 否 | 可选的空闲超时时间，单位为毫秒。如果在此时长内没有活动的 exec，Modal 就会终止沙盒。必须为正数且为 1000 的倍数。 |
| `execTimeoutMs` | 否 | 调用方未提供超时值时，每次 exec 的默认超时时间，单位为毫秒。必须为正数且为 1000 的倍数。默认值为 `300_000`（5 分钟）。 |
| `blockNetwork` | 否 | 阻止所有出站网络访问。 |
| `cidrAllowlist` | 否 | 沙盒可以访问的 CIDR 列表。不能与 `blockNetwork` 同时使用。 |
| `reuseLease` | 否 | 设为 `true` 时，释放后会分离沙盒，之后可按 ID 重新连接。默认为 `false`。 |

关于 `tokenId` / `tokenSecret` 的说明：插件工作器在一个不会继承主机环境变量的子进程中运行，因此在 Paperclip 服务器上设置的 `MODAL_TOKEN_ID` / `MODAL_TOKEN_SECRET` **不会**被插件读取。请始终通过环境配置提供令牌；Paperclip 会将粘贴的值保存为公司密钥。

---

## 复用语义

Modal 不提供单独的沙盒暂停/恢复原语——没有与 E2B 的 `pause()` 等效的功能。插件会将 `reuseLease` 映射到 Modal 实际提供的行为：

- **`reuseLease: false`（默认值）。** 释放时终止沙盒。下一次运行会新建沙盒。
- **`reuseLease: true`。** 释放时插件会分离沙盒。沙盒会继续在 Modal 上运行，直到 `sandboxTimeoutMs` 或 `idleTimeoutMs` 到期；之后再次获取资源时会按 ID 重新连接。如果沙盒已经过期，Paperclip 会检测到并创建新沙盒。

由于这里并没有真正的暂停操作，**`reuseLease: true` 会继续产生费用**，直到沙盒生命周期或空闲超时将其终止。如果开启复用，请将 `idleTimeoutMs` 设为与你预计保持资源租约的时间相符的值，以便控制费用。

---

## 探测超时

Modal 冷启动无需你在主机侧进行任何配置。`environmentProbe` RPC 的默认超时专门提高到 120 秒，让首次获取资源时的 Modal 沙盒（拉取镜像并启动容器）有足够时间完成初始化，避免探测过早失败。对全新 Modal 环境运行 **Probe**（探测）操作时需要等待一段时间，这是预期行为，并非卡死。

---

## 运维验证

将真实任务指向新环境之前，请完成以下检查：

1. 在 Modal 账户中创建 Modal 凭据（`modal token new`），或使用服务账户。
2. 在 Paperclip Plugins 页面安装插件。
3. 在 **Settings → Instance settings → Environments** 中添加 Modal 沙盒环境，至少填写 `appName`、`image`、`tokenId` 和 `tokenSecret`。
4. 运行环境 **Probe** 操作。成功结果可确认身份验证、App 创建、镜像拉取和 exec 往返调用正常。
5. 至少使用一个绑定到该环境的远程管理适配器（例如 `claude_local`）运行一次 Paperclip 任务。适配器应会配置沙盒、在其中运行命令并完成清理。

如果探测因身份验证失败，请再次确认 `tokenId` 和 `tokenSecret` 都已设置——只提供其中一个的配置会被插件拒绝。

---

## 示例

```json
{
  "driver": "sandbox",
  "provider": "modal",
  "config": {
    "appName": "paperclip-agents",
    "image": "node:20",
    "tokenId": { "type": "secret_ref", "secretId": "modal-token-id", "version": "latest" },
    "tokenSecret": { "type": "secret_ref", "secretId": "modal-token-secret", "version": "latest" },
    "workdir": "/workspace/paperclip",
    "sandboxTimeoutMs": 3600000,
    "idleTimeoutMs": 600000,
    "execTimeoutMs": 300000,
    "blockNetwork": false,
    "reuseLease": true
  }
}
```

---

## 相关内容

- [沙盒服务提供商](./sandbox-providers.md) — 所有沙盒服务提供商插件的概览。
- [插件](../../administration/plugins.md) — 从 Plugin Manager 安装和管理插件。
- [创建适配器](./creating-an-adapter.md) — 如果内置适配器都不适用，可自行开发。
