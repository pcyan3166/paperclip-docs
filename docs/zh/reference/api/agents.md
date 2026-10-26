---
paperclip_version: v2026.1023.0
seo_title: 代理 API
seo_description: 创建代理、检查和更新配置、管理生命周期、轮换密钥、同步技能、触发运行，以及注册托管和远程代理配置档。
---

# 代理

管理公司中的代理。此 API 可用于创建代理、查看配置、管理生命周期、轮换 API 密钥、同步技能、查看组织结构，或手动触发工作。

## 快速规则

- 此 API 按公司隔离，每个有效请求都关联到一个公司。
- 看板认证请求可管理所选公司中的代理，但仍须通过常规权限检查。
- 代理认证请求只能在自身所属公司内操作。
- `GET /api/agents/:id` 接受 UUID 或公司内唯一的短名称；短名称查询要求服务器知道公司上下文，通常通过 `?companyId=...` 或代理认证提供。
- 已终止代理不会出现在列表和组织图中，但知道 ID 时仍可直接读取。
- 调用者无权读取配置时，`GET /api/agents/:id` 可能返回同公司其他代理的删减视图。
- 所有序列化代理的响应都会把明文 `adapterConfig.env` 值替换为 `***REDACTED***`，包括 `GET /api/agents/me` 和创建、更新响应。`secret_ref` 与 `user_secret_ref` 绑定会原样返回，因为它们只是引用，不是秘密值。

## 常用字段

代理负载是普通 JSON 对象，常见字段如下：

| 字段 | 含义 |
|---|---|
| `name` | 易读名称；服务器还会据此派生公司内唯一 URL key。 |
| `role` | 角色标签，如 `ceo`、`engineer` 或 `general`。 |
| `title` | 可选展示职称。 |
| `appearance` | 代理角色（头像）身份：`{ "schemaVersion": 1, "characterVersion": "cap-v1", "paletteId": "<palette>" }`。调色板 ID 见[代理头像](#代理头像)。上传自定义图片时还含 `customAvatarAssetId`，见[上传自定义头像](#上传自定义头像)。 |
| `avatarUrl` | 只读。由 `appearance` 派生的图片 URL；有自定义图片时指向 `/api/assets/{assetId}/content`，否则指向角色图片。 |
| `reportsTo` | 组织树中的上级代理，必须属于同一公司且不能形成循环。 |
| `adapterType` | 运行时类型，如 `process`、`http`、`paperclip_runner`、`claude_local`、`codex_local`、`gemini_local`、`opencode_local`、`pi_local`、`hermes_local`、`cursor` 或 `openclaw_gateway`；也可注册外部适配器。 |
| `adapterConfig` | 适配器专属配置；`env` 中可使用秘密引用。 |
| `runtimeConfig` | 运行时设置；创建代理时 `heartbeat.enabled` 默认 `false`。 |
| `budgetMonthlyCents` | 以美分计的月度预算。创建时大于 `0` 会自动创建匹配的预算策略。 |
| `status` | 生命周期状态。共享枚举包括 `active`、`paused`、`idle`、`running`、`error`、`pending_approval`、`terminated`。准备、暂停或恢复期间为 `paused`，终止或清理期间为 `terminated`；要了解精确步骤请读 `lifecycleState`。 |
| `lifecycleState` | 只读，表示从招聘到终止的阶段：`pending_approval`、`preparing`、`verifying`、`ready`、`pausing`、`paused`、`resuming`、`terminating`、`cleaning_up`、`terminated` 或 `rejected`。只有 `ready` 状态可接收工作；更早指派的工作会排队。见[生命周期步骤与重试](#生命周期步骤与重试)。 |
| `lifecycleError` | 只读；生命周期步骤失败时显示不含凭据的简短错误，未失败时为 `null`。 |
| `lifecycleVersion` | 只读；每次生命周期操作都会递增的计数器。 |
| `permissions` | 代理级权限：`canCreateAgents` 和 `canCreateSkills`，默认均为 `true`。 |

## 列出代理

`GET /api/companies/{companyId}/agents`



获取公司代理名册，默认排除已终止代理。调用者无权读取配置时，响应会删减而不会直接失败。










<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -s \
  "http://localhost:3100/api/companies/{companyId}/agents" \
  -H "Authorization: Bearer <token>"
```

<!-- tab: JavaScript -->
```js
const res = await fetch(`http://localhost:3100/api/companies/${companyId}/agents`, {
  headers: { Authorization: `Bearer ${token}` },
});

const agents = await res.json();
```

<!-- tab: Python -->
```python
import requests

res = requests.get(
    f"http://localhost:3100/api/companies/{companyId}/agents",
    headers={"Authorization": f"Bearer {token}"},
)
agents = res.json()
```
<!-- /tabs -->

## 获取代理

`GET /api/agents/{agentId}`


按 UUID 或公司短名称返回代理记录及派生元数据。

完整记录还包含：

- `chainOfCommand`。
- `access`。

`access` 对象说明代理是否可以指派任务，以及该能力来自何处。

注意：

- 如果传入短名称而非 UUID，服务器必须知道公司上下文；必要时使用 `?companyId={companyId}`。
- 如果代理请求读取同公司另一代理的信息但无权读取其配置，响应会返回删减视图，而不是拒绝请求。











<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -s \
  "http://localhost:3100/api/agents/{agentId}?companyId={companyId}" \
  -H "Authorization: Bearer <token>"
```

<!-- tab: JavaScript -->
```js
const res = await fetch(
  `http://localhost:3100/api/agents/${agentId}?companyId=${companyId}`,
  {
    headers: { Authorization: `Bearer ${token}` },
  },
);

const agent = await res.json();
```

<!-- tab: Python -->
```python
import requests

res = requests.get(
    f"http://localhost:3100/api/agents/{agentId}",
    params={"companyId": companyId},
    headers={"Authorization": f"Bearer {token}"},
)
agent = res.json()
```
<!-- /tabs -->

<a id="public-identity"></a>
## 公开身份

`GET /api/agents/{id}/identity`



读取代理的公开加密身份。使用代理 UUID，并具备读取该代理所需的授权。代理尚未配置身份时响应为 `null`，否则返回：`algorithm` 为 `Ed25519`；`keyId` 为 `sha256:` 加上公钥 SPKI DER 字节的 base64url SHA-256 指纹；`publicKeyPem` 为 SPKI PEM 格式公钥；`createdAt` 为身份创建时间。

| 字段 | 值 |
|---|---|
| `algorithm` | `Ed25519`。 |
| `keyId` | `sha256:` 加上公钥 SPKI DER 字节的 base64url SHA-256 指纹。 |
| `publicKeyPem` | SPKI PEM 格式的公钥。 |
| `createdAt` | 身份创建时间戳。 |

读取操作不会创建身份。新代理创建时就会获得身份；较早创建的代理会在下一次受支持的托管运行中获得身份。端点绝不返回私钥。签名和恢复方式见[代理加密身份](../../guides/org/agent-identity.md)。这些密钥不能替代代理 API 令牌。代理提交产品反馈请使用单独的[投诉与建议端点](../../administration/agent-commentary.md#submit-through-the-api)。

## 获取当前代理

`GET /api/agents/me`

返回已认证的代理自身信息。代理认证流程可用它读取自身记录、汇报关系或访问状态。此路由仅供代理调用。

## 精简收件箱（`inbox-lite`）

`GET /api/agents/me/inbox-lite`



返回代理在心跳开始时用于安排工作的精简指派列表。与完整 issue 列表相比，每项只提供足以安排工作的字段，无需再次查询：

- `id`、`identifier`、`title`、`status` 和 `priority`。
- `projectId`、`goalId` 和 `parentId`。
- `updatedAt`。
- `activeRun`：该 issue 当前正在执行的运行（如有）。
- `dependencyReady`：仍有阻塞项时为 `false`。
- `unresolvedBlockerCount` 和 `unresolvedBlockerIssueIds`。

心跳启动时应优先使用此路由。只有需要完整 issue 对象时，才回退到 `GET /api/companies/{companyId}/issues?assigneeAgentId=...`。此路由仅供代理调用。











<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -s \
  "http://localhost:3100/api/agents/me/inbox-lite" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```

<!-- tab: JavaScript -->
```js
const res = await fetch("http://localhost:3100/api/agents/me/inbox-lite", {
  headers: { Authorization: `Bearer ${token}` },
});
const inbox = await res.json();
```

<!-- tab: Python -->
```python
import os, requests

res = requests.get(
    "http://localhost:3100/api/agents/me/inbox-lite",
    headers={"Authorization": f"Bearer {os.environ['PAPERCLIP_API_KEY']}"},
)
inbox = res.json()
```
<!-- /tabs -->

<a id="fetch-granted-secrets"></a>
## 获取获准的密钥

代理有时需要读取启动时未提供的凭据，例如不应在整个运行期间都放在进程环境中的值，或代理开始工作后由操作员新增授权的值。以下两个仅代理可调用的路由允许运行中的代理按需列出并读取获准的密钥，无需重启。

两条路由都绑定到运行上下文：调用者必须以代理身份认证，令牌背后要有有效且已验证的心跳运行，并且须通过 `secrets:read` 授权检查。低信任令牌（技能测试运行令牌和低信任审核代理）会按设计被拒绝，避免扩大受限运行的访问范围。

密钥的完整生命周期（创建、轮换、交付模式）见[密钥参考](secrets.md)。操作员可在代理设置的 **Secret access** 编辑器中选择代理能访问的密钥；选择器说明见[密钥文件夹](../../administration/secret-folders.md)。

### 列出获准密钥

`GET /api/agents/me/secrets`



只返回此代理有权访问的别名，绝不返回密钥值。响应为 `{ "secrets": [...] }`，每项包含：

- `key`：传给下方取值路由的别名。
- `name` 和 `description`：面向操作员的信息。
- `delivery`：`env`、`api` 或 `both`，见[交付模式](secrets.md#delivery-modes-env-vs-access)。
- `projectionClass`。
- `latestVersion`、`versionSelector` 和 `resolvedVersion`。

响应会有意移除内部标识符，包括密钥 ID、绑定 ID 和配置路径。


<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -s \
  "http://localhost:3100/api/agents/me/secrets" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```

<!-- tab: JavaScript -->
```js
const res = await fetch("http://localhost:3100/api/agents/me/secrets", {
  headers: { Authorization: `Bearer ${token}` },
});
const { secrets } = await res.json();
```

<!-- tab: Python -->
```python
import os, requests

res = requests.get(
    "http://localhost:3100/api/agents/me/secrets",
    headers={"Authorization": f"Bearer {os.environ['PAPERCLIP_API_KEY']}"},
)
secrets = res.json()["secrets"]
```
<!-- /tabs -->

### 读取单个密钥值

`POST /api/agents/me/secrets/{key}/value`



返回单个已授权别名的明文值。路径参数 `{key}` 是列表路由返回的 key。响应为 `{ "key", "value", "version" }`，并带有 `Cache-Control: no-store`，防止中间层和客户端缓存密钥值。别名未授权给该代理时，请求会因权限不足而失败。每次成功读取都会同时写入安全审计记录和操作员活动日志，因此可以追溯到具体代理及运行。



<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -s -X POST \
  "http://localhost:3100/api/agents/me/secrets/stripe-api-key/value" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```
<!-- tab: JavaScript -->
```js
const res = await fetch(
  "http://localhost:3100/api/agents/me/secrets/stripe-api-key/value",
  {
    method: "POST",
    headers: { Authorization: `Bearer ${token}` },
  },
);
const { value, version } = await res.json();
```
<!-- tab: Python -->
```python
import os, requests

res = requests.post(
    "http://localhost:3100/api/agents/me/secrets/stripe-api-key/value",
    headers={"Authorization": f"Bearer {os.environ['PAPERCLIP_API_KEY']}"},
)
secret = res.json()
```
<!-- /tabs -->

以上两个密钥路由仅代理可调用。













## 创建代理

`POST /api/companies/{companyId}/agents`


用于招聘或配置新代理；当前实现中此路由仅看板可调用。

重要行为：

- 服务器会验证目标公司。
- 新代理会立即开始[设置步骤](#生命周期步骤与重试)：`lifecycleState` 从 `preparing` 开始（`status` 显示为 `paused`），直到准备和验证完成、代理进入 `ready`。此期间指派的工作会排队。
- 若公司要求新代理必须经看板审批，请改用下方招聘流程；该流程会创建 `pending_approval` 并记录审批负载。
- 创建时会对重复名称去重。若名称映射到公司中已有的短名称，服务器会追加数字后缀。
- 若未提供 `runtimeConfig.heartbeat.enabled`，其默认值为 `false`。
- `adapterConfig.env` 可以包含密钥引用，但这些密钥必须属于同一公司。
- 若 `budgetMonthlyCents > 0`，服务器会自动创建匹配的月度预算策略。
- 若省略 `appearance`，服务器会为新代理随机选择角色调色板并保存，因此后续会保持相同外观。
- 对于支持指令包的适配器，请通过 `instructionsBundle.files` 传入代理指令，例如 `{"files": {"AGENTS.md": "You are the CTO. You own technical direction."}}`，并可选传入 `entryFile`。若省略，服务器会生成简短的默认 `AGENTS.md`：`role: "ceo"` 使用一行角色说明，其他角色使用一行通用占位内容。请保持简短，因为 Paperclip 运行时和已安装技能已经提供操作流程。新代理不要使用 `adapterConfig.promptTemplate` 或 `bootstrapPromptTemplate`。
- 某些适配器会在创建时应用默认值。例如，`codex_local`、`gemini_local` 和 `cursor` 可以填入默认模型；未禁用设备认证时，`openclaw_gateway` 可以生成设备私钥。




创建请求示例：


### 示例

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -s -X POST \
  "http://localhost:3100/api/companies/{companyId}/agents" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Engineering Lead",
    "role": "ceo",
    "title": "CEO",
    "reportsTo": null,
    "capabilities": "Plans strategy, delegates work, reviews progress",
    "adapterType": "claude_local",
    "adapterConfig": {
      "model": "claude-sonnet-4-20250514",
      "cwd": "/Users/me/projects/company",
      "env": {
        "ANTHROPIC_API_KEY": {
          "type": "secret_ref",
          "secretId": "secret-id",
          "version": "latest"
        }
      }
    },
    "runtimeConfig": {
      "heartbeat": {
        "enabled": false
      }
    },
    "budgetMonthlyCents": 25000,
    "desiredSkills": ["strategy", "planning"],
    "metadata": {
      "team": "platform"
    }
  }'
```

<!-- tab: JavaScript -->
```js
const res = await fetch(`http://localhost:3100/api/companies/${companyId}/agents`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "Engineering Lead",
    role: "ceo",
    title: "CEO",
    reportsTo: null,
    capabilities: "Plans strategy, delegates work, reviews progress",
    adapterType: "claude_local",
    adapterConfig: {
      model: "claude-sonnet-4-20250514",
      cwd: "/Users/me/projects/company",
      env: {
        ANTHROPIC_API_KEY: {
          type: "secret_ref",
          secretId: "secret-id",
          version: "latest",
        },
      },
    },
    runtimeConfig: {
      heartbeat: { enabled: false },
    },
    budgetMonthlyCents: 25000,
    desiredSkills: ["strategy", "planning"],
    metadata: { team: "platform" },
  }),
});

const agent = await res.json();
```

<!-- tab: Python -->
```python
import requests

res = requests.post(
    f"http://localhost:3100/api/companies/{companyId}/agents",
    headers={
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json",
    },
    json={
        "name": "Engineering Lead",
        "role": "ceo",
        "title": "CEO",
        "reportsTo": None,
        "capabilities": "Plans strategy, delegates work, reviews progress",
        "adapterType": "claude_local",
        "adapterConfig": {
            "model": "claude-sonnet-4-20250514",
            "cwd": "/Users/me/projects/company",
            "env": {
                "ANTHROPIC_API_KEY": {
                    "type": "secret_ref",
                    "secretId": "secret-id",
                    "version": "latest",
                }
            },
        },
        "runtimeConfig": {
            "heartbeat": {"enabled": False},
        },
        "budgetMonthlyCents": 25000,
        "desiredSkills": ["strategy", "planning"],
        "metadata": {"team": "platform"},
    },
)
agent = res.json()
```
<!-- /tabs -->

<a id="hire-agent"></a>
## 招聘代理

`POST /api/companies/{companyId}/agent-hires`


若公司可能要求看板审批，或要在招聘请求中关联来源 issue，请使用此招聘流程。

重要行为：

- 请求体接受与创建代理相同的核心代理字段。
- 可以包含 `sourceIssueId` 或 `sourceIssueIds`，将招聘请求关联到一个或多个 issue。
- 使用 `paperclip_runner` 适配器的代理可以发送 `"inheritRuntimeFrom": "caller"`，让新代理继承调用代理的 Runner 设置和默认环境。新代理也必须使用 `adapterType: "paperclip_runner"`；不能同时传入 `adapterConfig`、`runtimeConfig` 或 `defaultEnvironmentId`，因为服务器会从调用代理复制适配器配置和默认环境。看板用户不能使用此选项；请求会以 `403 Forbidden` 拒绝。
- 若公司要求新代理经看板审批，此路由会创建待审批记录，并保存所请求配置的快照。
- 此路由还会执行与直接创建相同的配置规范化和适配器验证。

<a id="update-agent"></a>
## 更新代理

`PATCH /api/agents/{agentId}`



用于常规代理修改，例如改名、更改角色、调整适配器配置、更改经理或运行时设置。

重要行为：

- 此路由不接受 `permissions`；请使用下方权限路由。
- 部分更新 `adapterConfig` 时，服务器通常会将其与现有配置合并。
- 设置 `replaceAdapterConfig: true` 时，更新会改为替换适配器配置。
- 更改包含指令包键的适配器配置时，服务器会尽可能保留指令包相关设置。
- `reportsTo` 必须指向同一公司的代理，且不能形成汇报循环。
- 传入符合[常用字段](#常用字段)所述格式的 `appearance`，即可更改代理外观。
- 若新短名称与公司中其他未终止代理冲突，改名会返回 `409 Conflict`。
- 已终止代理不能通过状态更新恢复。
- 待审批代理不能直接通过状态更新激活。


### 示例

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -s -X PATCH \
  "http://localhost:3100/api/agents/{agentId}" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Principal Engineer",
    "capabilities": "Designs systems, reviews architecture, and guides implementation",
    "runtimeConfig": {
      "heartbeat": {
        "enabled": true,
        "intervalSec": 300
      }
    }
  }'
```

<!-- tab: JavaScript -->
```js
const res = await fetch(`http://localhost:3100/api/agents/${agentId}`, {
  method: "PATCH",
  headers: {
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    title: "Principal Engineer",
    capabilities: "Designs systems, reviews architecture, and guides implementation",
    runtimeConfig: {
      heartbeat: {
        enabled: true,
        intervalSec: 300,
      },
    },
  }),
});

const agent = await res.json();
```

<!-- tab: Python -->
```python
import requests

res = requests.patch(
    f"http://localhost:3100/api/agents/{agentId}",
    headers={
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json",
    },
    json={
        "title": "Principal Engineer",
        "capabilities": "Designs systems, reviews architecture, and guides implementation",
        "runtimeConfig": {
            "heartbeat": {
                "enabled": True,
                "intervalSec": 300,
            },
        },
    },
)
agent = res.json()
```
<!-- /tabs -->

## 更新权限

`PATCH /api/agents/{agentId}/permissions`



这是修改代理级权限的唯一受支持方式。主 `PATCH /api/agents/:id` 路由会拒绝 `permissions` 请求字段。

当前权限字段：

- 必需的 `canCreateAgents`。
- 可选的 `canCreateSkills`。
- 必需的 `canAssignTasks`：此路由会将其作为代理的主体权限授权（`tasks:assign`）应用，而不是存储在代理记录中。

看板会话可以调用此路由。只有当代理会话的调用者是公司 CEO 代理时，才可以调用。

切换 `canAssignTasks` 时，服务器会确保代理有一条有效的 `companyMembership` 记录，并在 `principalPermissionGrants` 中写入或清除 `tasks:assign` 项。CEO 角色和旧版 `canCreateAgents` 标记仍具有隐式指派权限；`canAssignTasks` 只会在此基础上增加显式授权。

<a id="scoped-permissions-and-authorization"></a>
## 有范围的权限与授权

代理和看板用户共用一个授权服务。每次涉及代理或 issue 指派的变更都会通过 `authorizationService.decide`，评估公司成员资格、显式权限授权、作用域规则，以及目标所附的受保护代理/项目策略。

### 权限键

授权的 `permissionKey` 可以是：

| 键 | 授予的能力 |
|---|---|
| `agents:create` | 在公司内招聘或创建代理，对代理配置执行变更或环境探测。也满足 `agent_config:read` 和 `agent_config:update` 检查；但看板（人工）成员读取代理配置、技能或配置修订不再需要此键，只需公司成员身份。非人工代理读取同伴配置仍需此键或旧版 `canCreateAgents` 标记。 |
| `tasks:assign` | 将公司内任意 issue 指派给任意活动代理或人工成员。 |
| `tasks:assign_scope` | 仅在授权范围匹配时指派 issue，见下文；授权必须带结构化 scope。 |
| `tasks:manage_active_checkouts` | 强制释放或管理其他代理持有的检出。 |
| `environments:manage` | 管理执行工作区和环境。 |
| `users:invite` | 邀请人员加入公司。 |
| `users:manage_permissions` | 编辑其他成员的角色和授权。 |
| `joins:approve` | 批准待处理的加入请求。 |

新建的标准代理默认 `canCreateAgents: true`、`canCreateSkills: true`；低信任代理的 `canCreateAgents: false`。未显式设置 `canCreateAgents` 的已存代理按 `false` 处理。`ceo` 角色始终可以创建代理，无论该标志取何值。

新建标准代理或批准其招聘时，还会得到 14 项直接授权：`agents:configure`、`agents:suggest-changes`、`skills:create`、`skills:suggest-changes`、`tools:manage_connections`、`tools:manage_profiles`、`tools:view_audit`、`audit:view_agent_actions`、`tools:use`、`tools:manage_runtime`、scope 为 `{ "responsibleUserOnly": true }` 的 `inbox:manage`、`tasks:assign`、scope 为 `{ "subtreeRootAgentId": "<the new agent's id>" }` 的 `tasks:assign_scope`，以及 `tasks:manage_active_checkouts`。低信任代理和托管内置代理不会获得这些授权；现有代理不会补发。

用户加入时会自动套用角色默认授权：`owner` 和 `admin` 获得代理、技能、环境、邀请、任务指派、加入审批及四个工具权限键（`owner` 另有 `users:manage_permissions`）。`operator` 获得相同的代理、技能、环境、邀请和任务指派权限，另有 `pipelines:write`、工具权限（含 `tools:manage_profiles` 和 `tools:view_audit`）及 `audit:view_agent_actions`，但没有 `joins:approve` 或 `users:manage_permissions`。`viewer` 不获得这些权限。具体清单见[角色与权限](../../administration/roles-and-permissions.md#the-four-company-roles)。

### 授权范围

授权可以覆盖整个公司（没有 scope），也可以限制到子集。scope 是授权上的 JSON 对象，授权服务支持：

- `projectId` 或 `projectIds`：限制到指定项目；`allow` 数组中也可使用简写 `project:<id>`。
- `agentId`、`agentIds`、`assigneeAgentId`、`assigneeAgentIds`、`targetAgentId`、`targetAgentIds`：限制到指定受派代理；也接受 `agent:<id>` 简写。
- `managerAgentId`、`subtreeRootAgentId` 等：限制到经理的汇报子树；也接受 `subtree:<id>` 简写。

`tasks:assign_scope` 至少需要一个结构化约束。空 scope 的 `tasks:assign` 会视为公司级授权。

### 代理的 `access` 对象

`GET /api/agents/{agentId}` 会含 `access` 对象，概述指派权限：

| 字段 | 含义 |
|---|---|
| `canAssignTasks` | 代理当前能否在公司中指派工作。 |
| `taskAssignSource` | `ceo_role`、`agent_creator`、`explicit_grant`、`simple_default` 或 `none`。 |
| `membership` | 代理的 `companyMembership` 行（如有）。 |
| `grants` | 附加给代理的所有主体权限授权。 |

可直接使用此对象构建界面，无需自行重演授权规则。

### 受保护的代理、项目和 issue

代理、项目和 issue 可在 `permissions`、`executionWorkspacePolicy` 或 `executionPolicy` 中携带 `authorizationPolicy` 对象。授权服务识别以下三种形式：

- `agentVisibility.mode: "private"`：目标不会出现在简单的公司级指派默认范围内；指派需要显式授权。
- `assignmentPolicy.mode: "protected"`：指派需要显式的 `tasks:assign` 或匹配的 `tasks:assign_scope` 授权；简单成员默认权限不适用。
- `protectedAgent.requiresApproval: true`（或 `assignmentPolicy.protectedAgentRequiresApproval: true`）：必须先关联审批流程，否则无论是否有授权，指派都会被完全阻止。

如果服务无法识别策略对象（例如顶层键未知或模式字符串不受支持），它会将策略归类为“未知”，并通过 `deny_policy_restricted` 决策阻止指派。请将无法识别的策略数据视为硬性拦截。

### 决策原因

指派被拒绝时，错误负载会返回以下原因之一，可用于排查：

| 原因 | 含义 |
|---|---|
| `allow_local_board` | 本地隐式看板始终允许。 |
| `allow_instance_admin` | 实例管理员绕过公司级检查。 |
| `allow_explicit_grant` | 匹配的 `principalPermissionGrants` 行覆盖此操作。 |
| `allow_legacy_agent_creator` | 旧版 `canCreateAgents` 标志覆盖此操作。 |
| `allow_self` | 代理正在操作自己的指派或配置。 |
| `allow_company_agent` | 代理正在处理未指派代理的 issue。 |
| `allow_simple_company_member` | 简单模式默认值：任意有效非只读成员都可指派。 |
| `allow_manager_chain` | 因目标向该经理汇报而允许。 |
| `deny_unauthenticated` | 请求没有调用者，或没有代理 ID。 |
| `deny_company_boundary` | 跨公司访问，或目标代理不是公司内活动成员。 |
| `deny_missing_membership` | 主体没有活动公司成员资格。 |
| `deny_missing_grant` | 没有授权匹配所请求的权限键。 |
| `deny_policy_restricted` | 受保护代理、项目或 issue 阻止了操作。 |
| `deny_scope` | 存在授权，但它不涵盖请求的项目、受派人或子树。 |
| `deny_unsupported_action` | 此操作和调用者没有对应的权限映射。 |

### 管理成员角色和授权

`PATCH /api/companies/{companyId}/members/{memberId}/role-and-grants`



通过单个事务更改人工成员的角色并替换其授权集合。调用者需要公司级 `users:manage_permissions`。

| 字段 | 类型 | 说明 |
|---|---|---|
| `membershipRole` | 可选：`owner`、`admin`、`member`、`viewer` | 省略时保留现有角色；不能将最后一位活动 owner 降级。 |
| `status` | 可选：`active` 或 `inactive` | 默认沿用现有状态。 |
| `grants` | 可选数组 | 替换成员的完整授权集合；每项为 `{ permissionKey, scope? }`。 |

路由会先删除成员已有的 `principalPermissionGrants` 行，再插入新行，因此必须传入期望的完整集合，不支持部分更新。活动记录为 `company_member.updated`。

## 暂停、恢复、终止和删除

`POST /api/agents/{agentId}/pause`
`POST /api/agents/{agentId}/resume`
`POST /api/agents/{agentId}/terminate`
`DELETE /api/agents/{agentId}`



按以下方式使用：

- 需要暂时停止代理接收工作时，暂停代理。
- 需要让代理重新可以接收工作时，恢复代理。
- 不再希望调用代理、但要保留其记录时，终止代理。
- 需要硬删除并清理相关运行时状态、运行、会话、唤醒记录和密钥材料时，删除代理。

重要边界情况：

- 已终止的代理不能暂停。
- 已终止的代理不能恢复。
- 待审批代理不能恢复。
- 终止代理会撤销其全部 API 密钥。
- 终止仍处于 `pending_approval` 状态的代理会改为拒绝其招聘（其 `lifecycleState` 变为 `rejected`）。
- 暂停、恢复和终止会记录请求并立即返回。外部步骤在后台完成；请轮询 `GET /api/agents/{agentId}` 并查看 `lifecycleState`。
- 删除代理具有破坏性，清理内容远超代理记录本身。
- 只有终止流程已全部完成（或其招聘已被拒绝）后，才能删除代理。否则调用会返回 `409 Conflict`，并附带 `Complete termination before deleting the agent`。
- 生命周期请求与代理当前步骤不匹配时（例如代理仍在终止过程中却请求恢复），会返回 `409 Conflict`，错误消息类似 `Cannot resume an agent in state terminating`。

### 生命周期步骤与重试

`POST /api/agents/{agentId}/lifecycle/retry`


招聘、暂停、恢复和终止都会依序执行以下步骤，并通过 `lifecycleState` 展示：

| 操作 | 步骤 |
|---|---|
| 招聘（直接创建或审批后） | `preparing` → `verifying` → `ready`。 |
| 暂停 | `pausing` → `paused`。 |
| 恢复 | `resuming` → 回到暂停前保存的阶段（通常为 `ready`）。 |
| 终止 | `terminating` → `cleaning_up` → `terminated`。 |

`preparing` 阶段会让所需[插件](../plugins/sdk.md#required-agent-lifecycle-work)配置资源；`verifying` 会测试已保存的运行时配置。代理达到 `ready` 前收到的消息和任务会暂存，随后再派发。

步骤失败时，`lifecycleError` 为 `The lifecycle step failed. Retry the operation.` Paperclip 大约一分钟后会自动重试。修复配置后可立即手动重试：


此操作仅看板可调用，且需要更新代理的权限。返回更新后的代理，并记录 `agent.lifecycle_retried` 活动。

```bash
curl -s -X POST \
  "http://localhost:3100/api/agents/{agentId}/lifecycle/retry" \
  -H "Authorization: Bearer <token>"
```

## API 密钥

`GET /api/agents/{agentId}/keys`
`POST /api/agents/{agentId}/keys`
`DELETE /api/agents/{agentId}/keys/{keyId}`



若代理或工作进程需要归属于特定代理的长期认证，可使用密钥。

重要边界情况：

- 密钥令牌仅在创建时显示一次。
- 不能为待审批的代理创建密钥。
- 不能为已终止的代理创建密钥。
- 撤销密钥会将其标记为已撤销，不会硬删除。


### 示例

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -s -X POST \
  "http://localhost:3100/api/agents/{agentId}/keys" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"name":"deploy-bot"}'
```

<!-- tab: JavaScript -->
```js
const res = await fetch(`http://localhost:3100/api/agents/${agentId}/keys`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ name: "deploy-bot" }),
});

const key = await res.json();
```

<!-- tab: Python -->
```python
import requests

res = requests.post(
    f"http://localhost:3100/api/agents/{agentId}/keys",
    headers={
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json",
    },
    json={"name": "deploy-bot"},
)
key = res.json()
```
<!-- /tabs -->

## 唤醒代理

`POST /api/agents/{agentId}/wakeup`



需要手动触发工作并附加 issue ID、原因、评论 ID 或负载对象时，可使用此端点将带上下文的唤醒请求加入队列，或与现有请求合并。

请求体支持：

- `source`：`timer`、`assignment`、`on_demand` 或 `automation`。
- `triggerDetail`：`manual`、`ping`、`callback` 或 `system`。
- `reason`。
- `payload`。
- `idempotencyKey`。
- `forceFreshSession`。

重要行为：

- 此路由限定在公司范围内。
- 使用代理认证时，只能唤醒同一代理。
- 端点返回 `202 Accepted`。
- 即使服务器跳过唤醒，也会返回 `202` 和跳过负载。
- 如果 issue 已有活动执行运行，唤醒可能会延后或合并，而不是加入队列。
- 代理处于暂停、终止或待审批状态时，调用会以冲突响应失败。
- 心跳策略可以禁用定时唤醒。
- 策略也可以禁用非定时唤醒。


### 示例

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -s -X POST \
  "http://localhost:3100/api/agents/{agentId}/wakeup" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "source": "assignment",
    "triggerDetail": "manual",
    "reason": "Issue assigned",
    "payload": {
      "issueId": "issue-123",
      "commentId": "comment-456"
    },
    "idempotencyKey": "issue-123-assignment",
    "forceFreshSession": false
  }'
```

<!-- tab: JavaScript -->
```js
const res = await fetch(`http://localhost:3100/api/agents/${agentId}/wakeup`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    source: "assignment",
    triggerDetail: "manual",
    reason: "Issue assigned",
    payload: {
      issueId: "issue-123",
      commentId: "comment-456",
    },
    idempotencyKey: "issue-123-assignment",
    forceFreshSession: false,
  }),
});

const result = await res.json();
```

<!-- tab: Python -->
```python
import requests

res = requests.post(
    f"http://localhost:3100/api/agents/{agentId}/wakeup",
    headers={
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json",
    },
    json={
        "source": "assignment",
        "triggerDetail": "manual",
        "reason": "Issue assigned",
        "payload": {
            "issueId": "issue-123",
            "commentId": "comment-456",
        },
        "idempotencyKey": "issue-123-assignment",
        "forceFreshSession": False,
    },
)
result = res.json()
```
<!-- /tabs -->

### 常见的跳过响应

确切跳过原因取决于无法将唤醒加入队列的原因。常见情况包括：

- `wakeup_skipped`
- `issue_execution_deferred`
- `heartbeat.disabled`
- `heartbeat.wakeOnDemand.disabled`
- `budget.blocked`

## 调用心跳

`POST /api/agents/{agentId}/heartbeat/invoke`



如果不需要包含丰富唤醒负载，可使用此端点触发简单的按需心跳。其行为与唤醒类似，但响应更简单：

- `202 Accepted`。
- 返回运行对象，或 `{ "status": "skipped" }`。
- 代理认证只能调用自身。

## 组织图

`GET /api/companies/{companyId}/org`
`GET /api/companies/{companyId}/org.svg`
`GET /api/companies/{companyId}/org.png`



需要自行创建可视化时使用 JSON 路由；需要服务器直接渲染组织图时使用 SVG 或 PNG 路由。

注意：

- 这些路由不会返回已终止的代理。
- SVG 和 PNG 版本可以选择传入 `style`。

## 主要代理

`GET /api/companies/{companyId}/primary-agent/me`
`PUT /api/companies/{companyId}/primary-agent/me`


每个人可以在每家公司选择一个**主要代理**，作为新任务和 Chat 的个人默认代理。此偏好仅当前登录用户可见。应用使用方式见[你的主要代理](../../guides/org/agents.md#your-primary-agent)。


两个路由都要求登录的看板用户对公司有有效访问权（包括 viewer，因为这是个人偏好）。代理凭据不能调用。

`GET` 返回 `companyId`、`userId`、`primaryAgentId` 和 `initialized`。

- 如果尚未设置主要代理，或当前主要代理已不可读取，`primaryAgentId` 为 `null`。
- 首次在公司创建代理时会自动将 `initialized` 设为 `true`，也可通过 `PUT` 显式初始化。即使之后清除主要代理，`initialized` 仍保持 `true`，避免 Paperclip 静默替你选择新的代理。

`PUT` 只接受字段 `{ "primaryAgentId": "<agent-uuid>" }`，并返回更新后的偏好。设置主要代理也会让你加入该代理。代理必须属于当前公司（否则 `404 Agent not found`）、对你可读（否则 `403`），且状态不能是 `terminated` 或 `pending_approval`（否则返回 `422` 和 `Choose an active, approved agent as your primary.`）。没有删除路由，需设置其他代理来更换。活动日志记为 `primary_agent.updated`，但不会透露你选择了哪个代理。









```json
{
  "companyId": "...",
  "userId": "...",
  "primaryAgentId": "...",
  "initialized": true
}
```

```json
{ "primaryAgentId": "<agent-uuid>" }
```

```bash
curl -s -X PUT \
  "http://localhost:3100/api/companies/{companyId}/primary-agent/me" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{ "primaryAgentId": "{agentId}" }'
```

## 上传自定义头像

`PUT /api/companies/{companyId}/agents/{agentId}/avatar`


使用自己的图片替代代理的插画角色，也可恢复为角色头像。

- `imageBase64` 是图片文件本身的 Base64 编码，不是 URL，也不是 `data:` URL。传入 `null` 可删除自定义图片并恢复 Paperclip 角色。
- 只能使用静态 PNG、JPEG 或 WebP 图片，大小最多为 512 KiB、分辨率最多为 1600 万像素。SVG 和动画图片会被拒绝。Paperclip 会将图片重新编码为不超过 512×512 的 PNG，并移除所有元数据。
- **可调用者：**有权更新代理配置的看板用户（viewer 无此权限），或使用自身密钥的代理。代理只能更改自己的头像；任务范围凭据完全不能更改头像。
- 已终止或待审批的代理不能更改头像（返回 `409`）。

响应为 `{ agentId, appearance, avatarUrl }`。变更会记录为 `agent.avatar_updated` 活动，并创建一条配置修订。








```json
{ "imageBase64": "<raw image bytes, base64-encoded>" }
```

```bash
curl -s -X PUT \
  "http://localhost:3100/api/companies/{companyId}/agents/{agentId}/avatar" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d "{ \"imageBase64\": \"$(base64 < avatar.png | tr -d '\n')\" }"
```

## 代理头像

`GET /api/agent-avatars/{version}/{palette}/{pose}.png`



此路由渲染 `appearance` 对应的角色图片。通常无需手动构造 URL：代理响应已含 `avatarUrl`，实时运行列表也会为每次运行提供 `agentAppearance` 和 `avatarUrl`。需要不同尺寸或姿势时再直接使用此端点。它只提供预设图片，不会查询代理或公司，因此 URL 本身包含所有信息。

| 部分 | 可选值 |
|---|---|
| `version` | `cap-v1` |
| `palette` | `bubblegum-sky`、`pink-lemonade`、`orchid-peach`、`coral-mint`、`lime-lagoon`、`arctic-blue`、`solar-flare`、`violet-ember`、`deep-tide`、`coral-current`、`golden-hour`、`tangerine-cobalt`、`electric-grove`、`flamingo-jade`、`cherry-pop`、`turquoise-cherry`、`ultraviolet-tide`；灰化版本使用 `muted-dream`。 |
| `pose` | `rest`、`idle`、`listening`、`thinking`、`working`、`success`、`confused`、`sleepy`、`loading`。 |
| `size` 查询参数 | 像素尺寸 `16`、`20`、`24`、`32`、`40`、`48`、`64`、`96`、`128`、`256`、`512`，默认 `512`。 |
| `scale` 查询参数 | `1` 或 `2`（高像素密度屏幕），默认 `1`。 |
| `background` 查询参数 | `transparent` 或 `paperclip-dark`，默认 `transparent`。 |

注意：

- 图片缓存一年并带有 `ETag`，重复请求可能返回 `304 Not Modified`。
- 使用其他查询参数或列表外的值会返回 `400 Bad Request`。
- 并发请求过多会返回 `429 Too Many Requests`；渲染失败会返回 `503 Service Unavailable`。两种响应都会带有 `Retry-After`，请等待后重试。





```bash
curl -s -o ada.png \
  "http://localhost:3100/api/agent-avatars/cap-v1/arctic-blue/working.png?size=128&scale=2"
```

## 适配器辅助路由

`GET /api/companies/{companyId}/adapters/{type}/models`
`GET /api/companies/{companyId}/adapters/{type}/detect-model`
`POST /api/companies/{companyId}/adapters/{type}/test-environment`



用于选择模型、自动检测推荐模型，或在创建/更新代理前验证适配器配置。

重要说明：

- 适配器类型必须是服务器已知的类型。
- 测试环境路由需要调用者有权在公司内创建代理；检查方式与创建和招聘路由相同。
- 服务器会在运行测试前解析密钥。
- 测试已保存代理时，在测试环境请求体中传入 `agentId`。服务器会补齐已保存配置中被删减的环境值，无需重新输入。若要测试切换到另一适配器，则必须重新输入这些值；服务器不会把隐藏值带到无关的适配器。
- 省略 `environmentId` 时，如果传入了 `agentId`，测试会在代理已保存的默认环境中运行；否则使用实例默认环境。传入 `environmentId: null` 可明确指定使用实例默认环境。

<a id="skills"></a>
## 技能

`GET /api/agents/{agentId}/skills`
`POST /api/agents/{agentId}/skills/sync`



用于查看并同步附加到代理的技能，适用于支持技能发现与同步的适配器。

### 同步请求

发送目标技能集合及必填 `mode`，指定相对代理当前目标技能集的应用方式：

| 字段 | 必填 | 说明 |
|---|---|---|
| `desiredSkills` | 是 | 引用数组。每项是公司技能 UUID、规范 `key` 或唯一 slug，可以混用。 |
| `mode` | 是 | `"add"`、`"remove"` 或 `"replace"`。决定如何与当前集合合并。 |

`mode` 按技能 key 决定合并方式：

- `add`（并集）：保留代理的其他技能，并添加 `desiredSkills` 中列出的技能；若指定技能已存在，则原位更新。
- `remove`：只移除指定技能，其余技能保持不变。
- `replace`：用 `desiredSkills` 完全覆盖目标集合。这是破坏性操作：列表中的技能都会添加，列表外的技能都会删除。

如果缺少 `mode`，或其值不是上述三种之一，路由会返回 `422`，并附带消息：`Skill sync requires mode: "add", "remove", or "replace". Use "replace" only to overwrite the complete desired skill set.`

招聘/创建时，`POST /api/companies/{companyId}/agents` 和 `POST /api/companies/{companyId}/agent-hires` 也接受 `desiredSkills`，以便代理上线时已分配技能。这两个招聘/创建路由**不接受** `mode`；它只适用于 `/skills/sync`。

重要说明：

- 部分适配器尚未实现技能同步。
- 即使适配器不支持技能同步，路由仍会返回带警告的有效快照。
- 同步需要对目标代理拥有更新权限。
- 技能必须已安装到公司层级。完整公司技能生命周期（文件格式、导入、作用域和版本管理）见[技能参考](../skills.md)。

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -s -X POST \
  "http://localhost:3100/api/agents/{agentId}/skills/sync" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "desiredSkills": ["paperclip", "improve-skill"], "mode": "add" }'
```

<!-- tab: JavaScript -->
```js
await fetch(`http://localhost:3100/api/agents/${agentId}/skills/sync`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ desiredSkills: ["paperclip", "improve-skill"], mode: "add" }),
});
```

<!-- tab: Python -->
```python
import os, requests
requests.post(
    f"http://localhost:3100/api/agents/{agent_id}/skills/sync",
    headers={"Authorization": f"Bearer {os.environ['PAPERCLIP_API_KEY']}",
             "Content-Type": "application/json"},
    json={"desiredSkills": ["paperclip", "improve-skill"], "mode": "add"},
)
```
<!-- /tabs -->

## 配置

`GET /api/agents/{agentId}/configuration`
`GET /api/agents/{agentId}/config-revisions`
`GET /api/agents/{agentId}/config-revisions/{revisionId}`
`POST /api/agents/{agentId}/config-revisions/{revisionId}/rollback`
`GET /api/companies/{companyId}/agent-configurations`


这些路由用于读取和管理代理配置历史，可用于：

- 检查生效中的代理配置。
- 审计以往的配置变更。
- 回滚到以前的修订。
- 集中查看公司的所有代理配置。

重要说明：

- 公司看板（人工）成员读取配置和配置修订时，只需要公司成员资格，无需 `agents:create` 权限。
- 非人工代理读取这些数据时，仍需要 `agents:create`（已获授，或其自身记录中带有旧版 `canCreateAgents` 标记）；执行变更和环境探测始终需要 `agents:create`。
- 返回配置修订前会删减含密钥的字段，因此任何调用者都无法通过读取获得秘密值。若目标修订包含已删减的秘密值，回滚可能失败。

## 指令包

`PATCH /api/agents/{agentId}/instructions-path`
`GET /api/agents/{agentId}/instructions-bundle`
`PATCH /api/agents/{agentId}/instructions-bundle`
`GET /api/agents/{agentId}/instructions-bundle/file`
`PUT /api/agents/{agentId}/instructions-bundle/file`
`DELETE /api/agents/{agentId}/instructions-bundle/file`


以下路由管理代理的指令文件：


当代理提示以文件存储，而非仅内联配置时使用。对于**托管**指令包，这些文件构成代理持久目录，包含入口文件（通常为 `AGENTS.md`）及你或代理添加的笔记、子目录和二进制文件。Paperclip 只保留当前文件，新保存不会产生修订历史。指令包响应会标记 `persistence: "agent_files"`。运行如何读取和保存目录见[代理文件会跨任务保留](../../guides/org/agents.md#agent-files-persist-across-tasks)。

### 读取文件

`GET /api/agents/{agentId}/instructions-bundle/file?path=AGENTS.md`



查询参数 `path` 必填。响应包含文件 `content` 和 `contentHash`，保存或删除时需要携带该哈希。托管指令包可加 `download=true` 将原始字节作为附件流式下载，适合二进制文件或大文件。

```bash
curl -s -o notes.png \
  "http://localhost:3100/api/agents/{agentId}/instructions-bundle/file?path=notes/diagram.png&download=true" \
  -H "Authorization: Bearer <token>"
```

### 保存和删除文件

`PUT /api/agents/{agentId}/instructions-bundle/file`



| 字段 | 说明 |
|---|---|
| `path` | 指令包内的文件路径，如 `AGENTS.md` 或 `notes/today.md`。 |
| `content` | 完整文本，最多 1 MiB。 |
| `baseHash` | 最近读取到的 `contentHash`。创建新文件时传 `null`；托管指令包必填。 |
| `clearLegacyPromptTemplate` | 可选。设为 `true` 同时清除代理旧的内联 prompt 模板；需要管理指令路径的权限。 |

保存会防止丢失更新：若文件自读取后已更改，请求返回 `409 Conflict`，不写入任何内容。重新读取、重新应用修改并使用新哈希重试。托管指令包不传 `baseHash` 会返回 `422`。删除托管包中的文件时，将同一哈希作为查询参数传入：`DELETE /api/agents/{agentId}/instructions-bundle/file?path=notes/old.md&baseHash=<contentHash>`；入口文件不能删除。


```bash
curl -s -X PUT http://localhost:3100/api/agents/{agentId}/instructions-bundle/file \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "path": "AGENTS.md",
    "content": "You are the CTO. You own technical direction and the engineering team.",
    "baseHash": "<contentHash from your last read>"
  }'
```

### 存储限制

托管指令包接受普通文件和目录（包括二进制文件），拒绝符号链接和特殊文件。入口指令必须是有效 UTF-8 文件且不超过 1 MiB；其他持久化文件遵循下列较大单文件上限。

托管目录最多 100,000 个文件和目录，每个文件最多 256 MiB，总计最多 2 GiB。API 保存若超限会返回 `422`，且不更改现存文件。若运行导致超限，运行期间的目录变更不会保存；保存回执会报告 `AGENT_FILES_LIMIT_EXCEEDED`，运行详情会显示存储警告，但代理仍继续运行。

运行同步只保存有变化或被删除的文件。若不同运行或浏览器保存修改了同一路径，以最后完成的同步为准；未变化文件保持不动。当前字节存储在 `<paperclipInstanceRoot>/companies/<companyId>/agents/<agentId>/instructions/`，因此备份时除数据库外还应备份实例文件系统。会话停止时会清理临时工作副本，且不会产生新修订。



实现参考：[文件存储](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/server/src/services/agent-file-store.ts)、[托管目录路径](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/server/src/services/agent-instructions.ts)和[指令入口验证](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/server/src/services/agent-instruction-files.ts)。

### 旧修订和冲突路由

`GET /api/agents/{agentId}/instructions-bundle/history`
`GET /api/agents/{agentId}/instructions-bundle/revision/{revisionId}`
`GET /api/agents/{agentId}/instructions-bundle/diff?from={revisionId}&to={revisionId}`
`POST /api/agents/{agentId}/instructions-bundle/restore`
`GET /api/agents/{agentId}/instructions-bundle/candidates`
`POST /api/agents/{agentId}/instructions-bundle/candidates/{runId}/resolve`



这些路由用于兼容在代理文件功能推出前已保存指令修订的实例。新保存操作不再增加修订，因此 `history` 只会列出升级前记录的条目。`restore` 接受 `path`、`revisionId` 和 `baseRevisionId`。`candidates` 路由用于列出并解决旧版纯指令会话保留的编辑；`resolve` 接受 `baseRevisionId` 和 `content`。

注意：

- 目标代理或其祖先经理可以管理指令路径。
- 代理只能在其负责用户当前获准的范围内读取和保存自己的文件。读取或保存其他代理的文件需要有权配置该代理。
- [低信任](../../administration/trust-and-low-trust-review.md)代理在普通工作中不能保存自己的指令。例外情况是：用户直接在自己的 Agent Chat 中请求编辑；见[从自己的聊天编辑指令](../../administration/trust-and-low-trust-review.md#instruction-edits-from-your-own-chat)。
- 相对指令路径需要 `adapterConfig.cwd`。
- 外部指令包保留现有行为；`download=true` 和上述 `baseHash` 规则仅适用于受管理的指令包。





## Claude 登录

`POST /api/agents/{agentId}/claude-login`



此路由仅看板可调用，且只支持 `claude_local` 代理。

## Claude 订阅登录（setup-token）

公司所有者无需粘贴 `ANTHROPIC_API_KEY`，可使用 Claude 订阅的 **setup-token** 流程登录。此流程限定公司和环境，不携带代理 ID，因此可以在招聘任何代理之前执行；服务器从已认证调用者推断所有者。

| 路由 | 用途 |
|---|---|
| `GET /api/companies/{companyId}/claude-oauth-token-status` | 查询当前认证所有者是否保存了 Claude OAuth 令牌。只返回密钥 ID 和最新版本，不返回令牌。不存在或属于他人的值都会返回相同固定 `404`。 |
| `POST /api/companies/{companyId}/setup-token-login-sessions` | 启动登录会话。请求体 `{ "environmentId": "<uuid>", "adapterType": "claude_local", "overwrite"?: { … } }`，返回 `201`。 |
| `GET /api/companies/{companyId}/setup-token-login-sessions/active` | 不用会话 ID 重新找到调用者自己的活动会话，适用于浏览器刷新且本地状态丢失的情况。返回面板模式和一次性提示，并带 `Cache-Control: no-store, private`。非所有者、其他公司或无活动会话均返回相同固定 `404`。 |
| `GET /api/companies/{companyId}/setup-token-login-sessions/{sessionId}` | 读取会话状态。 |
| `GET /api/companies/{companyId}/setup-token-login-sessions/{sessionId}/prompt` | 读取一次性登录提示（需在浏览器打开的授权 URL）。 |
| `POST /api/companies/{companyId}/setup-token-login-sessions/{sessionId}/code` | 提交浏览器返回的代码；请求体 `{ "browserCode": "…" }`。 |
| `POST /api/companies/{companyId}/setup-token-login-sessions/{sessionId}/completion` | 读取完成声明。只返回非秘密 `storedSessionId`，不返回令牌。 |
| `POST /api/companies/{companyId}/setup-token-login-sessions/{sessionId}/cancel` | 取消会话。幂等：缺失、终止或属于其他人的会话 ID 都返回 `200`，因此不会借此确认会话是否存在。 |

注意：

- 任何响应都不会携带令牌。完成接口只返回不透明的 `storedSessionId` 声明；存储的值保存在公司密钥库中。
- 提示、代码和完成请求必须通过机密传输发送。非机密连接会被保护逻辑以 `403` 拒绝；这些步骤传送登录材料，不能以明文传输。

<a id="managed-and-remote-agent-profiles"></a>
## 托管代理和远程代理配置档

并非所有代理都运行在可编辑适配器配置中。有时操作员会先验证固定后端（例如 Claude Managed Agent 或 AWS Bedrock AgentCore 远程 Runner），并将其保存为公司级**配置档**。之后运行时输入会从配置档行复制不可变资源身份，而非信任代理自己的适配器配置。凭据不会保存在配置档中：托管配置档按 ID 关联公司密钥；远程配置档不含凭据。

两种配置档 API 都是看板专用、按公司隔离；两个 `POST` 路由都会按 `profileKey` upsert，返回 `201`。

### 托管代理配置档

`GET /api/companies/{companyId}/managed-agent-profiles`
`POST /api/companies/{companyId}/managed-agent-profiles`



请求体字段：

| 字段 | 说明 |
|---|---|
| `profileKey` | 公司内唯一的配置档键。 |
| `displayName` | 易读标签。 |
| `anthropicAgentId` | 托管代理不可变的公开 ID。 |
| `agentVersion` | 已验证的代理版本。 |
| `environmentId` | 托管环境 ID。 |
| `defaultModel` | 模型 ID；默认 `claude-sonnet-5`。 |
| `defaultMaxListCostUsd` | 每次列表操作的美元费用上限；默认 `1` 且必须为正数。服务器内部将其存为 `defaultMaxListCostCents`。 |
| `apiKeySecretId` | 保存 API key 的公司密钥 ID；凭据留在密钥库中。 |
| `enabled` | 配置档是否启用；默认 `false`。 |
| `retentionAcknowledged` | 操作员是否确认数据保留规则；默认 `false`。 |
| `qualification` | 自由格式的资质证据对象。 |

保存后的配置档还公开 `service`（始终为 `anthropic_managed_agents`）、`betaVersion`、`qualifiedAt`、`qualifiedRevision` 和常规时间戳。只有资质已获证明后才能验证并启用。Upsert 记录为 `managed_agent_profile.upserted`。

### 远程代理配置档

`GET /api/companies/{companyId}/remote-agent-profiles`
`POST /api/companies/{companyId}/remote-agent-profiles`



列表路由可选用 `?service=` 过滤。请求体字段如下：

| 字段 | 说明 |
|---|---|
| `profileKey` | 公司内唯一的配置档键。 |
| `displayName` | 易读标签。 |
| `service` | 远程 Runner 服务；目前仅接受 `aws_bedrock_agentcore_harness`。 |
| `configuration` | 服务专属配置对象。 |
| `enabled` | 配置档是否启用；默认 `false`。 |
| `retentionAcknowledged` | 操作员是否确认数据保留规则；默认 `false`。 |
| `qualification` | 自由格式的资质证据对象。 |

AWS AgentCore 使用 Runner 环境的工作负载身份认证，因此配置档不存凭据。传入 `credentialSecretId` 会以 `422 Unprocessable Entity` 拒绝；不支持的 `service` 值也一样。保存后的配置档还提供 `qualifiedAt`、`qualifiedRevision` 和时间戳。Upsert 记录为 `remote_agent_profile.upserted`。

## 常见边界情况

- 短名称查询未提供公司上下文时返回 `422 Unprocessable Entity`。
- 短名称冲突返回 `409 Conflict`。
- 通过主更新路由更新 `permissions` 返回 `422`。
- 为已终止或待审批代理创建密钥返回 `409`。
- 已终止代理不能暂停或恢复。
- 心跳运行路由收到无效 UUID 格式的运行 ID 时返回 `400 Bad Request`（`Invalid heartbeat run ID`）。
- 即使唤醒请求已接受，也可能跳过执行，尤其是策略或执行状态阻止时。

## 说明

本页已按当前代理路由、认证规则、生命周期行为和文档站 Markdown 规范重写。

- 原始页面：`docs/api/agents.md`

---
