---
paperclip_version: v2026.618.0
seo_title: OpenClaw Gateway 适配器
seo_description: 当 OpenClaw 在另一台计算机、Docker 容器中或网络边界之后运行时，通过 Gateway WebSocket 协议连接 OpenClaw 实例。
---

# OpenClaw Gateway

`openclaw_gateway` 会通过 OpenClaw Gateway WebSocket 协议将 Paperclip 连接到 OpenClaw 实例。如果 OpenClaw 在另一台计算机、Docker 容器中或共享网关后面运行，并且你希望 Paperclip 智能体将工作委派给它，请使用此适配器。

> **信息：** `openclaw_gateway` 在运行时中功能完整，但目前在智能体配置中的适配器类型下拉菜单里显示为 **“即将推出”**。它通过 OpenClaw 邀请提示词流程接入，不能手动选择。本页介绍此适配器，方便你通过 API 或导入的公司配置来使用。直接从界面选择的功能计划稍后提供。

---

## 适用场景

- 可通过 `ws://` 或 `wss://` 访问 OpenClaw（本地 Docker、远程主机或 Tailscale）。
- 希望多个 Paperclip 智能体共用一个 OpenClaw 实例。
- 需要在 Paperclip 与网关之间使用设备身份验证进行配对。

## 不适用场景

- 远程智能体不支持 OpenClaw Gateway 协议。请使用 [HTTP](./http.md)。
- 希望在同一台计算机上运行无头本地 CLI 智能体。请使用 [Claude Code](./claude-code.md)、[Codex](./codex.md) 或 [Process](./process.md)。

---

## 传输

适配器始终使用 WebSocket 网关传输方式。URL 必须以 `ws://` 或 `wss://` 开头。连接流程如下：

1. 从网关接收 `connect.challenge`。
2. 发送带有协议、客户端、身份验证和设备载荷的 `req connect`。
3. 发送 `req agent`。
4. 通过 `req agent.wait` 等待完成。
5. 将 `event agent` 帧传输到 Paperclip 的日志和执行记录解析器。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `url` | 是 | 网关 WebSocket URL（`ws://` 或 `wss://`）。 |
| `authToken` / `token` | 否* | 网关身份验证令牌。 |
| `headers` | 否* | 身份验证的另一种方式：`headers["x-openclaw-token"]` 或旧版的 `headers["x-openclaw-auth"]`。 |
| `password` | 否* | 共享密码身份验证模式。 |
| `disableDeviceAuth` | 否 | 设为 `true` 时不发送签名设备载荷。默认为 `false`。 |
| `devicePrivateKeyPem` | 否 | 固定使用稳定的 Ed25519 签名密钥。未设置时，适配器会在每次运行时生成临时密钥对。 |
| `autoPairOnFirstConnect` | 否 | 默认为 `true`。首次连接遇到一次“需要配对”时，会通过共享身份验证调用 `device.pair.list` 和 `device.pair.approve`，然后重试一次。 |
| `sessionKeyStrategy` | 否 | `issue`、`fixed` 或 `run`。用于确定 `agent.sessionKey` 的生成方式。 |
| `sessionKey` | 否 | `sessionKeyStrategy` 为 `fixed` 时使用。 |
| `agentId` | 否 | 可选的目标 OpenClaw 智能体 ID。 |
| `role` | 否 | 发送给 OpenClaw 的网关身份角色。默认为 `operator`。 |
| `scopes` | 否 | 网关能力权限范围（数组，例如 `["operator.admin"]`）。在创建界面中输入逗号分隔的字符串，系统会将其拆分为数组。默认为 `["operator.admin"]`。 |
| `paperclipApiUrl` | 否 | 可选，用于覆盖网关/智能体回调 Paperclip 的 API 基础 URL。没有默认值。 |
| `workspaceRuntime` | 否 | 可选的工作区运行时服务定义（`{ services: [...] }` 对象）。在创建界面中以 `runtimeServicesJson` 输入；只有成功解析为包含 `services` 数组的对象时才会应用。 |
| `payloadTemplate` | 否 | 合并到智能体请求中的字段。`message` / `text` 会添加到唤醒文本前面。 |
| `timeoutSec` | 否 | 适配器级请求时间预算。 |
| `waitTimeoutMs` | 否 | 单次等待调用的 `agent.wait.timeoutMs`。 |

*必须提供 `authToken` / `token`、`headers` 或 `password` 其中之一。如果提供了令牌且未设置 `authorization` 请求头，适配器会生成 `Authorization: Bearer <token>`。

---

## 设备身份验证

默认情况下，适配器会在 `connect` 参数中发送签名的 `device` 载荷。建议配置如下：

- 不设置 `disableDeviceAuth`，或将其设为 `false`。
- 提供稳定的 `devicePrivateKeyPem`，使设备身份在多次运行间保持不变。
- 首次配对由 `autoPairOnFirstConnect` 自动处理。

如果无法自动完成配对（令牌不匹配或没有待处理请求），第一次执行会返回 `pairing required`。请在 OpenClaw 中批准待处理设备，然后重试任务。

如果要在本地 Docker 环境中快速测试，可以在主机上批准：

```sh
docker exec openclaw-docker-openclaw-gateway-1 sh -lc \
  'openclaw devices approve --latest --json \
    --url "ws://127.0.0.1:18789" \
    --token "$(node -p \"require(process.env.HOME+\\\"/.openclaw/openclaw.json\\\").gateway.auth.token\")"'
```

检查待处理设备和已配对设备：

```sh
docker exec openclaw-docker-openclaw-gateway-1 sh -lc \
  'TOK="$(node -e \"const c=require(\\\"/home/node/.openclaw/openclaw.json\\\");process.stdout.write(c.gateway?.auth?.token||\\\"\\\")\")"; \
   openclaw devices list --json --url "ws://127.0.0.1:18789" --token "$TOK"'
```

---

## 会话策略

会话路由方式与 HTTP OpenClaw 模式相同：

- `sessionKeyStrategy=issue` — 每项问题使用一个 OpenClaw 会话（适合作为默认值）。
- `sessionKeyStrategy=fixed` — 所有执行共用一个 `sessionKey`。
- `sessionKeyStrategy=run` — 每次执行使用新会话（心跳之间不保留记忆）。

解析后的会话密钥会在每个请求中作为 `agent.sessionKey` 发送。

---

## 载荷映射

智能体请求的构造方式如下：

必填字段：
- `message` — 唤醒文本；可在前面添加 `payloadTemplate.message` 或 `payloadTemplate.text`。
- `idempotencyKey` — Paperclip 的 `runId`。
- `sessionKey` — 根据策略解析的会话密钥。

可选字段：
- 会合并 `payloadTemplate` 中的其他所有字段。
- 如果已配置 `agentId`，且模板中未设置该字段，则会注入 `agentId`。

---

## 超时

- `timeoutSec` 控制适配器级请求时间预算。
- `waitTimeoutMs` 控制单次调用的 `agent.wait.timeoutMs`。

如果 `agent.wait` 返回 `timeout`，适配器会返回 `openclaw_gateway_wait_timeout`。

---

## 日志格式

结构化网关事件会按以下格式输出：

- `[openclaw-gateway] ...` — 生命周期及系统日志。
- `[openclaw-gateway:event] run=<id> stream=<stream> data=<json>` — `event agent` 帧。

界面和 CLI 解析器会读取这些行并渲染执行记录。

---

## 在 Docker 中运行 OpenClaw

Paperclip 提供了多种 smoke 测试脚手架，可以在 Docker 中启动 OpenClaw 并自动连接到 Paperclip。

### 自动加入 smoke 测试

```sh
pnpm smoke:openclaw-join
```

此测试会自动完成邀请创建、智能体加入、看板审批、一次性 API 密钥领取，以及向 Docker 化接收器（`docker/openclaw-smoke`）发送唤醒回调。

### 一键启动网关界面

```sh
pnpm smoke:openclaw-docker-ui
```

此命令会在 `/tmp/openclaw-docker` 中克隆或更新 `openclaw/openclaw`，构建 `openclaw:local`，在 `~/.openclaw-paperclip-smoke/` 下写入独立的 smoke 配置，通过 Compose 启动 `openclaw-gateway`，并输出 `http://127.0.0.1:18789/#token=...`。

常用变量：

| 变量 | 默认值 | 用途 |
|---|---|---|
| `OPENAI_API_KEY` | —（必需） | 从环境变量或 `~/.secrets` 加载。 |
| `OPENCLAW_GATEWAY_PORT` | `18789` | 网关主机端口。 |
| `OPENCLAW_GATEWAY_TOKEN` | 随机生成 | 网关身份验证令牌。 |
| `OPENCLAW_BUILD` | — | 设为 `0` 可跳过重建。 |
| `OPENCLAW_DISABLE_DEVICE_AUTH` | `1` | 禁用控制界面设备配对；设为 `0` 则要求配对。 |
| `OPENCLAW_MODEL_PRIMARY` | `openai/gpt-5.2` | 主模型。 |
| `OPENCLAW_RESET_STATE` | `1` | 每次运行时重置 smoke 智能体状态。 |
| `PAPERCLIP_HOST_FROM_CONTAINER` | `host.docker.internal` | 容器访问 Paperclip 的方式。 |

### 网络提示

在 OpenClaw Docker 容器内，`127.0.0.1` 指向容器本身，而不是主机。Docker 中使用的 URL 应改用 `host.docker.internal`。如果 Paperclip 拒绝此主机名，请将其加入允许列表：

```sh
pnpm paperclipai allowed-hostname host.docker.internal
```

对于远程网关，建议使用可访问的主机名（Docker 主机别名、Tailscale 主机名或公开域名）。

---

<a id="onboarding-checklist"></a>
## 接入检查清单

将 OpenClaw 添加到 Paperclip 公司时：

1. 以身份验证模式启动 Paperclip，并启动一个干净的 OpenClaw Docker 实例。
2. 在公司设置中使用 **Generate OpenClaw Invite Prompt**（生成 OpenClaw 邀请提示词），并粘贴到 OpenClaw 主聊天中。
3. 在 Paperclip 中批准产生的加入请求，并确认智能体已出现在公司中。
4. 预先检查智能体配置：
   - `adapterType` 为 `openclaw_gateway`（不是 `openclaw`）。
   - `url` 以 `ws://` 或 `wss://` 开头。
   - 令牌长度足够（至少 16 个字符）。
   - 已持久化 `devicePrivateKeyPem`，并且 `disableDeviceAuth` 为 `false`。
5. 为智能体分配一项 smoke 测试任务（例如“发表评论 `OK` 并标记为完成”）。确认评论已显示且问题状态变为 `done`。

---

## 示例

```json
{
  "adapterType": "openclaw_gateway",
  "adapterConfig": {
    "url": "ws://127.0.0.1:18789",
    "headers": {
      "x-openclaw-token": {
        "type": "secret_ref",
        "secretId": "openclaw-gateway-token",
        "version": "latest"
      }
    },
    "devicePrivateKeyPem": "<PEM private key stored as a secret reference in production>",
    "autoPairOnFirstConnect": true,
    "sessionKeyStrategy": "issue",
    "timeoutSec": 300,
    "waitTimeoutMs": 60000
  }
}
```

---

## 后续阅读

- [创建适配器](./creating-an-adapter.md)
- [外部适配器](./external-adapters.md)
