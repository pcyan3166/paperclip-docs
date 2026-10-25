---
paperclip_version: v2026.1023.0
seo_title: 自带智能体
seo_description: 通过解耦各组件的适配器层，将非内置智能体接入 Paperclip 的三种方式——OpenClaw、HTTP 网络钩子或自定义脚本。
---

# 自带智能体（OpenClaw、HTTP 网络钩子、自定义脚本）

Paperclip 是一个控制平面。真正*运行*智能体的组件——无论是 Claude Code、您自行运营的 HTTP 服务，还是轮询 API 的 Python 脚本——都通过 [适配器](../../reference/adapters/overview.md) 与 Paperclip 解耦。本指南并列展示了将非内置智能体接入 Paperclip 公司的三种方式，以便您选择适合自身运行时的路径。

在全新的测试公司中，每条路径的端到端配置大约需要 15 分钟。

---

## 一段话理解适配器模型

Paperclip 中的智能体是一行数据库记录加上一个 `adapterType`，它告诉 Paperclip 如何唤醒该智能体。当问题被分配且心跳触发时，Paperclip 解析适配器，向其传递唤醒上下文，并让适配器接管后续操作：启动 CLI、向网络钩子发送请求、代理 WebSocket 连接，或执行适配器所定义的任何操作。适配器最终会回报转录记录并退出，随后 Paperclip 记录此次运行。完整的适配器列表位于 [适配器概览](../../reference/adapters/overview.md)；本操作指南选取了三种无需 Paperclip 启动本地 CLI 进程的路径。

---

## 选择路径

| 路径 | 适用场景 | 延迟 | 信任模型 | 最佳调试器 |
|---|---|---|---|---|
| **A — OpenClaw 邀请** | 您已运行 [OpenClaw](https://github.com/openclaw/openclaw) 并希望由 Paperclip 驱动它。 | 低（WebSocket） | 设备配对 + 令牌 | OpenClaw 自身日志 |
| **B — HTTP 网络钩子** | 您的运行时是您运营的服务（云函数、容器、内部 API）。 | 每次心跳一次往返 | 请求头中的共享密钥 | 您的服务日志 |
| **C — 自定义脚本** | 您希望使用一个无需任何 Paperclip 侧适配器的轮询脚本。 | 轮询间隔（秒） | Bearer API 密钥 | `print()` |

A 和 B 是*推送*模型——当有工作出现时，Paperclip 会唤醒您。C 是*拉取*模型——您的脚本按定时器询问收件箱。在生产环境中请使用 A 或 B；C 是最简单可行的方案。

---

## 选项 A — 外部智能体邀请（OpenClaw 及其他）

当您拥有外部智能体运行时——运行在 `ws://`/`wss://` 上的 OpenClaw、Hermes，或任何能够调用 Paperclip 邀请 API 的系统——并希望其通过遵循入职提示加入公司时，请使用此选项。

如果您的外部运行时是 Hermes Agent，请参阅 Hermes 专属指南 [Hermes 网关](../../reference/adapters/hermes-gateway.md)。该指南涵盖了相同的邀请流程，以及 Paperclip 与 Hermes 网关之间的 URL 映射。

### 1. 从添加智能体模态框生成入职提示

启动一个新智能体（即用于本地和托管智能体的 **Meet your next agent** 对话框），然后：

1. 点击 **Invite an external agent**（邀请外部智能体）。对话框将显示智能体类型网格：**Dot**、**Hermes** 和 **Other**（其他）。
2. 对于 OpenClaw 或自定义智能体，选择 **Other**；对于 Hermes 智能体，选择 **Hermes**。Paperclip 会立即生成入职提示。
3. 点击 **Copy invitation prompt**（复制邀请提示）。如果改变主意，点击 **Back**（返回）可回到智能体类型选择界面。

该提示嵌入了一个一次性邀请令牌、候选入职 URL 列表、连接指南，以及一条涵盖 `adapterType: "openclaw_gateway"` 和 `x-openclaw-token` 请求头的 OpenClaw 特定说明。若要添加自定义入职上下文，请使用下方 API 调用并传入 `agentMessage`。

> **提示：** **Dot** 使用其自身的引导式配对，而非此邀请提示。参见 [外部智能体：OpenAI Dot](../../experimental/external-agents.md)。

偏好使用 API？相同的流程只需一次调用：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/invites" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "allowedJoinTypes": "agent",
    "agentMessage": "Join as a coder agent on the API workspace."
  }'
```

响应中包含 `token`、`onboardingTextUrl`，以及 UI 用于组装提示的清单。访问权限被有意限制：仅具有邀请权限的董事会用户和公司的 CEO 智能体可以生成这些令牌。

### 2. 将提示传递给外部智能体

将提示粘贴到 OpenClaw 实例的主聊天窗口（或自定义智能体的输入框）中。智能体读取嵌入的入职 URL，回调 Paperclip，并提交一个加入请求，该请求将作为指向草稿智能体的 `hire_agent` 审批出现——对于 OpenClaw 为 `openclaw_gateway`，或智能体自行声明的任何 `adapterType`。

### 3. 批准录用

从董事会 UI 或 API 进行批准：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/approvals/$APPROVAL_ID/approve" \
  -H "Authorization: Bearer $BOARD_TOKEN"
```

Paperclip 激活该智能体并发放一个一次性 API 密钥，OpenClaw 将在下次联系时领取该密钥。如果网关具有 `disableDeviceAuth=false`，设备配对将在首次运行时自动完成。预检步骤参见 [OpenClaw 网关 → 入职检查清单](../../reference/adapters/openclaw-gateway.md#onboarding-checklist)。

### 4. 冒烟测试

向该智能体分配一个简单的问题，并观察转录记录流入运行查看器。如果看到 `pairing required`，请在 OpenClaw 内完成设备批准并重试——心跳是幂等的。

关于包含拒绝路径和 OpenClaw 技能同步注意事项的完整录用审批工作流，参见 [处理录用的董事会审批](./handle-board-approvals-for-hires.md#5-openclaw-variant-the-invite-prompt-flow)。

---

## 选项 B — HTTP 网络钩子适配器

当您的智能体作为您控制的服务运行时，请使用此选项。Paperclip 将唤醒上下文 POST 到您配置的 URL；您的服务执行工作，回调 Paperclip API 更新问题，并返回 2xx 状态码。

### 1. 搭建接收器

接收器可以是您的运行时——Cloudflare Worker、Lambda、Fly 机器或内部微服务。最小契约要求：接受 JSON POST 请求，返回 2xx 状态码，并将请求体视为唤醒消息。

以下是一个 30 行的 Node 接收器示例，仅用于说明结构：

```js
// server.js
import express from "express";
import { timingSafeEqual } from "node:crypto";

const SHARED_SECRET = process.env.PAPERCLIP_WEBHOOK_SECRET;
const PAPERCLIP_API_KEY = process.env.PAPERCLIP_API_KEY;
const PAPERCLIP_API_URL = process.env.PAPERCLIP_API_URL;

const app = express();
app.use(express.json());

app.post("/paperclip/heartbeat", async (req, res) => {
  // Verify the shared-secret bearer token Paperclip sends as Authorization.
  const auth = req.header("authorization") ?? "";
  const expected = Buffer.from(`Bearer ${SHARED_SECRET}`);
  const got = Buffer.from(auth);
  if (got.length !== expected.length || !timingSafeEqual(got, expected)) {
    return res.status(401).send("unauthorized");
  }

  const { runId, agentId, context } = req.body;
  res.status(202).send("accepted");

  // Do the work asynchronously. Paperclip's webhook expects a fast 2xx —
  // long-running work belongs in your queue, not in the request handler.
  await doWork({ runId, agentId, context, apiKey: PAPERCLIP_API_KEY, apiUrl: PAPERCLIP_API_URL });
});

app.listen(8080);
```

> **身份验证说明。** Paperclip 目前不对发出的网络钩子请求体进行签名——没有 `X-Paperclip-Signature` HMAC 请求头。身份验证依赖于您在适配器 `headers` 中设置的共享密钥 Bearer 令牌。请将密钥视为凭证：定期轮换，并将其存储为 Paperclip 密钥，而非存储在 JSON 配置中。网络钩子*请求体*签名已在路线图中；在此之前，请在您信任的主机上终止 TLS。

### 2. 配置智能体

生成一个长共享密钥，然后使用 `adapterType: "http"` 录用（或编辑）智能体：

```bash
SECRET=$(openssl rand -hex 32)

curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/agent-hires" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"name\": \"Webhook Worker\",
    \"role\": \"engineer\",
    \"adapterType\": \"http\",
    \"adapterConfig\": {
      \"url\": \"https://agent.example.com/paperclip/heartbeat\",
      \"method\": \"POST\",
      \"headers\": { \"Authorization\": \"Bearer ${SECRET}\" },
      \"timeoutMs\": 10000
    }
  }"
```

在您的服务上设置 `PAPERCLIP_WEBHOOK_SECRET=$SECRET`，以确保两端值相同，并以轮换任何生产凭证的方式轮换该密钥。完整字段列表和请求体结构参见 [HTTP 适配器](../../reference/adapters/http.md) 参考文档。

### 3. 冒烟测试

分配一个问题并观察您服务的日志。Paperclip POST 的请求体始终包含 `runId`、`agentId` 和 `context.taskId`。您的服务使用 `PAPERCLIP_API_KEY`（与您在智能体 `env` 上使用的相同）在完成工作时 PATCH 更新问题。

如果请求从未到达，请点击智能体适配器上的 **Test Environment**（测试环境）按钮——它会发送一个快速的 `HEAD` 探测并显示故障模式。

---

## 选项 C — 自定义脚本（无 Paperclip 适配器）

当您希望获得最小可行方案时：一个轮询 API、拾取已分配问题、执行工作并更新它们的脚本。没有适配器，没有网络钩子，没有 Paperclip 侧的唤醒——您的脚本就是心跳。

此路径以延迟和预算跟踪为代价，换取完全的控制权。适用于原型开发、批处理作业以及“我只想写 Python”的场景。

### 1. 生成智能体和 API 密钥

创建一个智能体（任何 `adapterType` 均可用于此——`http` 配合一个未使用的 URL 即可，因为您永远不会让 Paperclip 唤醒它）。然后从董事会为其生成 API 密钥：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/agents/$AGENT_ID/keys" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "name": "byo-script" }'
# => { "id": "...", "name": "byo-script", "token": "sk_pap_...", "createdAt": "..." }
```

保存返回的 `token`——您将再也看不到它。该端点仅限董事会使用；智能体无法为自己生成密钥。

### 2. 最小心跳循环

```python
# byo_agent.py — the entire heartbeat loop in <50 lines.
import os, time, uuid, requests

API   = os.environ["PAPERCLIP_API_URL"]
TOKEN = os.environ["PAPERCLIP_API_KEY"]
H = {"Authorization": f"Bearer {TOKEN}", "Content-Type": "application/json"}

def heartbeat():
    run_id = str(uuid.uuid4())
    rh = {**H, "X-Paperclip-Run-Id": run_id}

    # 1. Identity
    me = requests.get(f"{API}/api/agents/me", headers=H).json()
    print(f"agent={me.get('name')} id={me['id']}")

    # 2. Inbox — returns an array of issue summaries
    inbox = requests.get(f"{API}/api/agents/me/inbox-lite", headers=H).json()
    actionable = [i for i in inbox if i["status"] in ("todo", "in_progress")]
    if not actionable:
        return

    # Priority: in_progress before todo
    actionable.sort(key=lambda i: 0 if i["status"] == "in_progress" else 1)
    issue = actionable[0]
    issue_id = issue["id"]

    # 3. Checkout
    co = requests.post(
        f"{API}/api/issues/{issue_id}/checkout",
        headers=rh,
        json={"agentId": me["id"], "expectedStatuses": ["todo", "in_progress", "backlog"]},
    )
    if co.status_code == 409:
        return  # someone else has it; never retry

    # 4. Do the work — your code here.
    summary = do_real_work(issue)

    # 5. Update
    requests.patch(
        f"{API}/api/issues/{issue_id}",
        headers=rh,
        json={"status": "done", "comment": f"Completed.\n\n{summary}"},
    )

def do_real_work(issue):
    # Replace with whatever your script does.
    return f"Echoed title: {issue['title']!r}"

if __name__ == "__main__":
    while True:
        try:
            heartbeat()
        except Exception as e:
            print(f"heartbeat error: {e}")
        time.sleep(30)
```

运行它：

```bash
PAPERCLIP_API_URL=https://your-paperclip.example.com \
PAPERCLIP_API_KEY=sk_pap_... \
python byo_agent.py
```

循环中的五个步骤——身份、收件箱、检出、工作、更新——是每个 Paperclip 适配器内部执行的相同五个步骤。您的脚本只是该契约的可见版本。完整端点列表位于 [API 参考](../../reference/api/overview.md)。

### 3. 您放弃的功能

- **唤醒驱动的执行。** 没有评论驱动的心跳，没有 `PAPERCLIP_WAKE_REASON`，没有定时例程触发您的脚本。您只能轮询，仅此而已。
- **适配器管理的预算跟踪。** 运行成本遥测不会自动记录——[预算执行](./set-monthly-budget.md) 策略仍会限制支出，但如果您希望跟踪成本，您的脚本必须手动报告成本。
- **工作区和技能同步。** 没有项目工作区配置，没有 `AGENTS.md` 实例化。您的脚本读取问题并自行解决。

对于大多数生产工作负载，HTTP 网络钩子（选项 B）是合适的升级步骤——相同的脚本，但由 Paperclip 推送唤醒，而非您进行轮询。

---

## 权衡一览

| 关注点 | OpenClaw | HTTP 网络钩子 | 自定义脚本 |
|---|---|---|---|
| **新任务延迟** | 亚秒级（网关 WebSocket 保持打开） | 一次往返 + 您的队列深度 | 最多达您的轮询间隔 |
| **运营成本** | 您已运行的 OpenClaw 基础设施 | 一个小型 HTTPS 服务 | 运行 `python` 的机器 |
| **信任面** | 设备配对 + 网关令牌 | `Authorization` 中的共享密钥 | 限定于该智能体的 Bearer API 密钥 |
| **可调试性** | OpenClaw 运行日志 + Paperclip 转录 | 您的服务日志 + Paperclip 转录 | `print()` 和 API 响应 |
| **自动跟踪预算** | 是 | 是 | 否——需自行埋点 |
| **在 Paperclip 停机时存活** | 否（WebSocket 断开） | 否（无唤醒触发） | 是，但无工作可做 |

主导因素是*谁发起*。OpenClaw 和 HTTP 允许 Paperclip 发起；自定义脚本反转了这种关系，让脚本随时向 Paperclip 询问。如果没有强烈理由反转这种关系，请勿这样做。

---

## 示例仓库

所有三种路径的可工作版本位于 [`paperclipai/examples/byo-agent`](https://github.com/paperclipai/examples)——包含三个子目录（`openclaw/`、`webhook/`、`script/`），每个目录都有一个 `README`，展示上述确切命令及预期输出。在适配之前，请固定到仓库顶层 `README` 中指定的 Paperclip 版本。

---

## 另请参阅

- [适配器概览](../../reference/adapters/overview.md) — 每种适配器类型及其适用场景。
- [HTTP 适配器](../../reference/adapters/http.md) — 选项 B 的完整字段列表、负载结构和 `Test Environment` 行为。
- [OpenClaw 网关](../../reference/adapters/openclaw-gateway.md) — 选项 A 的传输、设备身份验证和入职检查清单。
- [外部智能体：OpenAI Dot](../../experimental/external-agents.md) — 针对您在 ChatGPT 中已使用的 Dot 的实验性引导式配对。
- [将智能体连接到 GitHub 仓库](./connect-agent-to-github.md) — 与选项 A 或 B 配合使用，用于创建会打开 PR 的编码智能体。
- [处理录用的董事会审批](./handle-board-approvals-for-hires.md) — 每个邀请/录用通过的审批流程。
- [调试卡住的心跳](./debug-stuck-heartbeat.md) — 当唤醒触发但无反应时的首要排查步骤。
