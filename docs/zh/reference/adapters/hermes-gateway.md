---
paperclip_version: v2026.1023.0
seo_title: Hermes Gateway 适配器
seo_description: 通过 HTTP 或 HTTPS 连接到已在其他位置运行的 Hermes API 服务器（例如另一台主机或云虚拟机），而不是在本地运行 Hermes。
---

# Hermes Gateway

`hermes_gateway` 通过 HTTP 或 HTTPS 将 Paperclip 连接到**远程 Hermes API 服务器**。如果 Hermes Agent 已在某处运行（例如另一台主机、云虚拟机或独立进程），并且公开了 API 服务器，请使用此适配器。Paperclip 会直接调用 Hermes REST/SSE API 来创建执行、传输事件，并在超时时停止执行。

从 v2026.626.0 开始，此适配器是内置的一等适配器。你可以通过标准 Paperclip 智能体配置表单进行设置，方式与配置 `claude_local` 或 `codex_local` 相同。

> 有关较旧的外部 Hermes 智能体邀请流程（v2026.626.0 之前），请参阅[接入自有智能体](../../how-to/bring-your-own-agent.md)。

---

## 适用场景

- Hermes Agent 在另一台主机或独立进程中运行，并公开了 API 服务器。
- 希望将 Paperclip 连接到托管在云端或由团队共享的 Hermes 实例。
- 需要从一个 Paperclip 部署中调用多台 Hermes 服务器。
- 正在构建多机部署，希望 Hermes 和 Paperclip 分开运行。

## 不适用场景

- 希望 Paperclip 在本地启动 Hermes 子进程。请改用 [`hermes_local`](./hermes.md)。
- 目标主机上未启用 Hermes API 服务器。

---

## 配置字段

| 字段 | 必填 | 默认值 | 说明 |
|---|---:|---|---|
| `apiBaseUrl` | 是 | — | Hermes API 服务器基础 URL。可以是仪表板根地址或聊天 URL（例如 `http://127.0.0.1:9119/chat`）；适配器会将其映射到 `/api`。非回环主机建议使用 HTTPS。 |
| `apiKey` | 是 | — | Hermes 的 `API_SERVER_KEY`，**不是**你的 `PAPERCLIP_API_KEY`。该值会以 Paperclip 密钥引用形式保存，绝不会以明文显示。 |
| `dangerouslyAllowInsecureRemoteHttp` | 否 | `false` | 仅供开发使用的不安全逃生开关。始终允许回环 HTTP（`127.0.0.1`、`localhost`）；非回环 HTTP 主机才需要开启此选项。 |
| `sessionKeyStrategy` | 否 | `"issue"` | 控制 `X-Hermes-Session-Key` 请求头。可选值：`issue`（按问题划分）、`agent`（按智能体划分）、`run`（按执行划分）、`none`（禁用）。请参阅下文的[会话密钥策略](#会话密钥策略)。 |
| `timeoutSec` | 否 | `600` | 执行超时秒数。 |
| `eventReconnectMs` | 否 | `2000` | SSE 事件流发生非终止性断开后，等待多少毫秒再重新连接。 |
| `paperclipApiUrl` | 否 | — | 远程 Hermes 主机可访问的可选 Paperclip API URL。这不是凭据；该地址用于在需要时告知 Hermes 如何回调 Paperclip。 |
| `headers` | 否 | — | 可选的额外非机密请求头 JSON 对象。API 密钥和会话密钥等安全关键请求头由适配器生成，无法在此覆盖。 |
| `instructions` | 否 | — | 发送给 Hermes 的可选固定指令，与唤醒输入分开，因此无需在每个任务提示词中重复加入。 |

---

## 工作原理

Paperclip 为 `hermes_gateway` 智能体触发执行时，适配器会执行以下操作：

1. 通过 `POST /v1/runs` **创建执行**。请求包含与 Paperclip 执行 ID 相同的 `Idempotency-Key`，便于将 Hermes 执行与 Paperclip 记录关联起来。
2. 从 `GET /v1/runs/{run_id}/events` **传输事件**（SSE）。如果事件流意外断开，适配器会等待 `eventReconnectMs` 后重新连接。当 SSE 事件流不可用时，还会轮询 `GET /v1/runs/{run_id}` 作为回退方式。
3. 如果执行完成前已达到超时时间，则通过 `POST /v1/runs/{run_id}/stop` **停止执行**。

---

## 安全

**API 密钥存储。** `apiKey` 字段对应 Hermes 的 `API_SERVER_KEY`。Paperclip 会将其作为密钥引用保存，绝不写入明文配置。如果字段显示已保存密钥，除非你有意轮换密钥，否则不要再次粘贴密钥。

**HTTP 与 HTTPS。** 回环地址（`127.0.0.1`、`localhost`）可以使用普通 HTTP。其他主机应使用 HTTPS 或专用覆盖网络（例如 Tailscale 或 VPN）。如果开发期间必须通过普通 HTTP 连接非回环主机，请开启 `dangerouslyAllowInsecureRemoteHttp`，但上线前必须关闭。

**额外请求头。** `headers` 字段仅用于非机密辅助请求头。身份验证和会话请求头由适配器自行生成；通过 `headers` 传入任何安全关键请求头都会被忽略。

---

## 会话密钥策略

`sessionKeyStrategy` 字段用于控制 `X-Hermes-Session-Key` 请求头。Hermes 使用该请求头将多个执行归入同一会话，并在执行间保留记忆连续性。

| 策略 | `X-Hermes-Session-Key` 范围 | 适用场景 |
|---|---|---|
| `issue`（默认） | 每项 Paperclip 问题一个会话 | 避免不同任务之间的记忆相互泄露。推荐大多数团队使用。 |
| `agent` | 此智能体的所有执行共用一个会话 | 适用于需要持续记忆的长期运行智能体。 |
| `run` | 每次执行都使用新会话 | 完全隔离执行，不共享记忆。 |
| `none` | 不发送请求头 | 由 Hermes 自行决定会话范围。 |

默认的 `issue` 策略最安全。它会让每项 Paperclip 问题都使用独立的 Hermes 会话，避免一项任务的上下文泄漏到另一项任务中。

---

## 示例

```json
{
  "adapterType": "hermes_gateway",
  "adapterConfig": {
    "apiBaseUrl": "https://hermes.internal.example.com",
    "apiKey": {
      "type": "secret_ref",
      "secretId": "hermes-api-key",
      "version": "latest"
    },
    "sessionKeyStrategy": "issue",
    "timeoutSec": 600,
    "eventReconnectMs": 2000,
    "paperclipApiUrl": "https://paperclip.internal.example.com",
    "instructions": "You are an engineering assistant. Work carefully and post status updates as you go."
  }
}
```

如果 Hermes 和 Paperclip 在同一台计算机上运行，可使用以下本地开发配置：

```json
{
  "adapterType": "hermes_gateway",
  "adapterConfig": {
    "apiBaseUrl": "http://127.0.0.1:9119",
    "apiKey": {
      "type": "secret_ref",
      "secretId": "hermes-api-key",
      "version": "latest"
    },
    "sessionKeyStrategy": "issue",
    "timeoutSec": 300
  }
}
```

---

## 故障排查

**执行无法启动，且适配器报告连接错误**

检查 Hermes API 服务器是否正在运行，以及 Paperclip 后端是否可以访问 `apiBaseUrl`——不能只检查浏览器是否可访问。如果 Paperclip 在 Docker 中运行，而 Hermes 在主机上，`127.0.0.1` 会指向容器，而不是主机。请改用 `host.docker.internal` 或局域网地址。

**连接到回环地址时被拒绝**

如果 `apiBaseUrl` 指向 `localhost` 或其他回环地址，而连接被拒绝，执行错误会说明地址是在 Paperclip 服务器上解析的——不是在智能体沙盒中，也不是在你的浏览器所在计算机上。请确认 Hermes API 服务器正在 Paperclip 服务器上运行，或将 `apiBaseUrl` 设置为服务器可访问的地址。Hermes Gateway 只连接到已经运行的网关。如果希望 Paperclip 为你启动 Hermes CLI，请改用 [Hermes](./hermes.md) 适配器（`hermes_local`）。

**API 密钥字段显示已保存密钥**

这是正常的。Paperclip 会隐藏已保存的适配器密钥。除非你要轮换密钥，否则不要再次粘贴。

**连接非回环主机时 HTTP 被拒绝**

默认会阻止连接非回环主机的普通 HTTP。请将 `apiBaseUrl` 改为 HTTPS URL，或者仅在开发环境中开启 `dangerouslyAllowInsecureRemoteHttp`。

**事件流持续重新连接**

频繁重新连接通常表示 SSE 事件流不断断开——常见原因包括代理超时、负载均衡器的保活时间限制较短或网络不稳定。请尝试增大 `eventReconnectMs` 以减少反复连接，并检查 Paperclip 与 Hermes 之间的代理是否过早终止长连接。

**Hermes 中的执行已完成，但 Paperclip 没有显示结果**

SSE 事件流丢失时，适配器会轮询 `GET /v1/runs/{run_id}` 作为回退。如果事件流和轮询都失败，请确认 `apiBaseUrl` 可以正确解析且 Hermes API 服务器运行正常。检查 Paperclip 执行日志中适配器报告的最后一条错误。

**不同任务之间的记忆相互泄漏**

请将 `sessionKeyStrategy` 设为 `issue`（默认值）。如果已经是 `issue`，请确认 Hermes 实例确实按密钥隔离会话。如果需要更强的隔离保证，可将策略设为 `run`，使每次执行都使用独立会话。

---

## 后续阅读

- [Hermes](./hermes.md) — 在同一台计算机上将 Hermes 作为本地子进程运行。
- [接入自有智能体](../../how-to/bring-your-own-agent.md) — 外部智能体邀请流程，以及将第三方智能体连接到 Paperclip 的整体说明。
- [智能体适配器](../../guides/org/agent-adapters.md) — Paperclip 适配器的通用工作方式。
