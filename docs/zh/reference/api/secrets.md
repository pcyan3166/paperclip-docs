---
paperclip_version: v2026.824.0
seo_title: 密钥 API
seo_description: 将敏感值从代理配置中移出，同时允许代理在运行时使用。此 API 提供仅看板可用、按公司隔离的密钥定义和解析端点。
---

# 密钥

Paperclip 使用密钥功能将敏感值从代理配置中移出，同时仍允许代理在运行时使用。此 API 以公司为范围；密钥创建、读取、更新和删除主要由看板操作员管理，代理仅能通过明确授权的运行绑定接口访问。

可在以下场景使用此 API：

- 列出公司中存储的密钥。
- 创建新的密钥值。
- 轮换密钥，同时不改变代理引用它的方式。
- 将新值写入存储在外部保险库中的密钥。
- 更新显示名称或描述等密钥元数据。
- 完全删除密钥。
- 查看此部署中可用的提供方。

## 密钥提供方

```http
GET /api/companies/{companyId}/secret-providers
```

返回此部署中可用的提供方。

内置提供方为：

- `local_encrypted`——在本地以静态加密方式存储。

列表中也可能出现其他提供方，但如果它们未在当前部署中配置，创建或解析操作就会失败。

提供方描述符包含以下字段：

- `id`
- `label`
- `requiresExternalRef`
- `supportsManagedValues`
- `supportsExternalReferences`
- `supportsExternalValueWrites`
- `configured`

如果 `requiresExternalRef` 为 `true`，除密钥值外，该提供方还要求提供外部引用字符串。

三个 `supports…` 能力标志用于判断提供方允许的操作：

- `supportsManagedValues`：Paperclip 可自行创建并轮换值。默认 `local_encrypted` 对此返回 `true`，对外部引用返回 `false`。
- `supportsExternalReferences`：可关联已存储在其他位置的密钥，Paperclip 只保存指针。
- `supportsExternalValueWrites`：除了更换指针，还可从 Paperclip 将新值写入关联的外部密钥。`aws_secrets_manager` 对此返回 `true`。当前版本的 `gcp_secret_manager` 和 `vault` 只是占位实现：`supportsManagedValues: false`，且不会声明支持外部值写入。

`configured` 表示此提供方在当前部署是否具备所需配置。提供方即使声明支持某能力，也可能尚未配置；此时尝试写入仍会失败。外部写入行为见[轮换密钥](#轮换密钥)。

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl "http://localhost:3100/api/companies/company-1/secret-providers" \
  -H "Authorization: Bearer <board-token>"
```

<!-- tab: JavaScript -->
```javascript
const res = await fetch("/api/companies/company-1/secret-providers", {
  headers: {
    Authorization: `Bearer ${boardToken}`,
  },
});

const providers = await res.json();
```

<!-- tab: Python -->
```python
import requests

response = requests.get(
    "http://localhost:3100/api/companies/company-1/secret-providers",
    headers={
        "Authorization": f"Bearer {board_token}",
    },
)

providers = response.json()
```
<!-- /tabs -->

## 保存了什么

Paperclip 在两层中存储密钥：

- `company_secrets` 保存密钥记录、元数据和最新版本指针。
- `company_secret_versions` 保存各版本的密钥材料。

对于每个密钥，API 会公开以下元数据：

- `id`
- `companyId`
- `name`
- `provider`
- `managedMode`
- `externalRef`
- `latestVersion`
- `description`
- `createdByAgentId`
- `createdByUserId`
- `createdAt`
- `updatedAt`

API 不会返回明文密钥值。

轮换前应理解 `managedMode`，它决定值的实际存放位置：

- `paperclip_managed`：值由 Paperclip 管理；由 Paperclip 创建、保存，轮换时也由 Paperclip 写入新值。
- `external_reference`：值保存在外部保险库，`externalRef` 是指向它的指针。Paperclip 保存此指针及其版本历史，不保存凭据本身。

`external_reference` 密钥过去只能关联已有密钥。若提供方声明支持 `supportsExternalValueWrites`，现在也可以将新值直接写入外部保险库，见[轮换密钥](#轮换密钥)。默认 `local_encrypted` 提供方用本地 master key 对版本材料进行 AES-GCM 加密；版本行还保存原始值的 SHA-256 哈希。

## 列出密钥

```http
GET /api/companies/{companyId}/secrets
```

按创建时间从新到旧返回公司的密钥。

需要查看以下信息时可使用此路由：

- 存在哪些密钥。
- 每个密钥使用哪个提供方。
- 当前最新版本是什么。
- 密钥是否有描述或外部引用。

密钥值绝不会返回。此路由仅供看板调用。

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl "http://localhost:3100/api/companies/company-1/secrets" \
  -H "Authorization: Bearer <board-token>"
```

<!-- tab: JavaScript -->
```javascript
const res = await fetch("/api/companies/company-1/secrets", {
  headers: {
    Authorization: `Bearer ${boardToken}`,
  },
});

const secrets = await res.json();
```

<!-- tab: Python -->
```python
import requests

response = requests.get(
    "http://localhost:3100/api/companies/company-1/secrets",
    headers={
        "Authorization": f"Bearer {board_token}",
    },
)

secrets = response.json()
```
<!-- /tabs -->

## 密钥目录

```http
GET /api/companies/{companyId}/secrets/catalog
```

返回精简密钥列表，每项仅含 `id`、`name`、`key` 和 `status`。与仅看板可调用且返回完整元数据的[密钥列表](#列出密钥)不同，目录用于选择要引用的密钥，因此看板或有公司访问权的代理均可调用。密钥值绝不返回。

适合在代理配置中填充密钥选择器，只显示可绑定项目，而不暴露提供方细节或版本历史。

```bash
curl "http://localhost:3100/api/companies/company-1/secrets/catalog" \
  -H "Authorization: Bearer <token>"
```

## 创建密钥

```http
POST /api/companies/{companyId}/secrets
Content-Type: application/json
```

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `name` | 是 | 公司内唯一。 |
| `value` | 是 | 要存储的明文密钥值。 |
| `provider` | 否 | 默认 `local_encrypted`。 |
| `description` | 否 | 面向操作员的易读备注。 |
| `externalRef` | 否 | 部分外部提供方必填。 |

省略 `provider` 时，Paperclip 使用有效的部署默认提供方；若默认值无效，则回退到 `local_encrypted`。

创建后会立即将该值保存为新版本：

- 创建版本 `1`。
- 将 `latestVersion` 设为 `1`。
- API 返回密钥元数据，不返回明文值。

若同一公司中已存在同名密钥，API 会返回冲突。

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -X POST "http://localhost:3100/api/companies/company-1/secrets" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "anthropic-api-key",
    "value": "sk-ant-...",
    "description": "Primary Claude key for worker agents"
  }'
```

<!-- tab: JavaScript -->
```javascript
const res = await fetch("/api/companies/company-1/secrets", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${boardToken}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "anthropic-api-key",
    value: "sk-ant-...",
    description: "Primary Claude key for worker agents",
  }),
});

const created = await res.json();
```

<!-- tab: Python -->
```python
import requests

response = requests.post(
    "http://localhost:3100/api/companies/company-1/secrets",
    headers={
        "Authorization": f"Bearer {board_token}",
        "Content-Type": "application/json",
    },
    json={
        "name": "anthropic-api-key",
        "value": "sk-ant-...",
        "description": "Primary Claude key for worker agents",
    },
)

created = response.json()
```
<!-- /tabs -->

## 更新密钥

```http
PATCH /api/secrets/{secretId}
Content-Type: application/json
```

请求体：

| 字段 | 必填 | 说明 |
|---|---|---|
| `name` | 否 | 重命名密钥。在公司内仍须唯一。 |
| `description` | 否 | 更新面向操作员的备注。 |
| `externalRef` | 否 | 更新提供方引用，但不会创建新的密钥版本。 |

此端点不会更改密钥值。这里没有 `value` 字段，而且这项规则没有改变：任何值写入（包括写入外部保险库）都必须通过[轮换密钥](#轮换密钥)。

当你要整理元数据，或在不更改密钥在 Paperclip 中版本记录的情况下，将外部托管的密钥指向新的提供方引用时，可以使用此端点。



<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -X PATCH "http://localhost:3100/api/secrets/secret-uuid" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "anthropic-api-key-prod",
    "description": "Production Claude key"
  }'
```


<!-- tab: JavaScript -->
```javascript
const res = await fetch("/api/secrets/secret-uuid", {
  method: "PATCH",
  headers: {
    Authorization: `Bearer ${boardToken}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "anthropic-api-key-prod",
    description: "Production Claude key",
  }),
});

const updated = await res.json();
```


<!-- tab: Python -->
```python
import requests

response = requests.patch(
    "http://localhost:3100/api/secrets/secret-uuid",
    headers={
        "Authorization": f"Bearer {board_token}",
        "Content-Type": "application/json",
    },
    json={
        "name": "anthropic-api-key-prod",
        "description": "Production Claude key",
    },
)

updated = response.json()
```
<!-- /tabs -->


## 轮换密钥

```http
POST /api/secrets/{secretId}/rotate
Content-Type: application/json
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `value` | `paperclip_managed` 必填 | 新的明文密钥值。`external_reference` 可不传；仅在要将新值写入外部保险库时传入。 |
| `externalRef` | 否 | 省略时保留密钥当前外部引用。 |
| `providerVersionRef` | 否 | 重新指定 `external_reference` 密钥时，固定到提供方侧某个具体版本。不能与 `value` 同时传入，对 `paperclip_managed` 密钥无效。 |
| `providerConfigId` | 否 | 将写入固定到指定提供方保险库。省略时沿用密钥当前使用的保险库；传 `null` 时回退到部署默认值。 |

轮换会创建不可变的新版本，并推进 `latestVersion`。

重要行为如下：

- 密钥 ID 保持不变。
- 使用 `version: "latest"` 的现有引用会自动使用新值。
- 固定到数字版本的引用会继续使用该版本。
- 旧版本会作为历史记录保留。

凭据变化但密钥身份保持不变时，应调用此端点。

<a id="rotating-an-external-reference-secret"></a>

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -X POST "http://localhost:3100/api/secrets/secret-uuid/rotate" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "value": "sk-ant-new-value..."
  }'
```


<!-- tab: JavaScript -->
```javascript
const res = await fetch("/api/secrets/secret-uuid/rotate", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${boardToken}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    value: "sk-ant-new-value...",
  }),
});

const rotated = await res.json();
```


<!-- tab: Python -->
```python
import requests

response = requests.post(
    "http://localhost:3100/api/secrets/secret-uuid/rotate",
    headers={
        "Authorization": f"Bearer {board_token}",
        "Content-Type": "application/json",
    },
    json={
        "value": "sk-ant-new-value...",
    },
)

rotated = response.json()
```
<!-- /tabs -->


### 轮换外部引用密钥

前面的示例适用于 `paperclip_managed`，其轮换总是“保存此新值”。对 `external_reference` 密钥，轮换有两种不同操作，请求体决定执行哪一种。

**更改引用指针：**传 `externalRef`（可选附带 `providerVersionRef`），不传 `value`。Paperclip 会记录指向保险库中另一密钥的新元数据版本；不会写入保险库。这是外部引用密钥一直以来的行为。

```bash
curl -X POST "http://localhost:3100/api/secrets/secret-uuid/rotate" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "externalRef": "prod/anthropic/api-key"
  }'
```

**直接写入新值：**传 `value`，保持现有引用不变，形状与上面标签页示例相同。提供方支持时，Paperclip 会将值写入 `externalRef` 当前指向的外部密钥，无需离开 Paperclip 再手动登录保险库粘贴凭据。

此操作仅在提供方实现外部值写入时可用。提供方描述符中的 `supportsExternalValueWrites` 会告知看板是否显示此选项；目前只有 AWS Secrets Manager 支持。

AWS 写入过程如下：

- Paperclip 会先读取当前 `AWSCURRENT` 版本，以确定将替换的版本。
- 随后写入新 AWS 版本，并将其设为 `AWSCURRENT`。这是有意为之：AWS 密钥的**所有**使用者都会在之后读取到新值，而不只是 Paperclip。
- 保存材料会继续跟踪 `AWSCURRENT`，而不是固定到刚写入的版本；因此之后直接在 AWS 执行的轮换也会同步反映到 Paperclip。
- 如果保险库写入成功，但 Paperclip 未能记录新版本，它会将 `AWSCURRENT` 恢复到写入前的版本，避免控制平面不知道某个值已存在。

由于写入的是共享保险库，应将其视为真实轮换：读取该 AWS 密钥的任何其他使用方在下次读取时也会看到新值。

成功响应与普通轮换相同：返回更新后的密钥记录、推进后的 `latestVersion`、写入的 `lastRotatedAt`，以及提供方确认后的 `externalRef`。AWS 会将其规范化为被写入密钥的完整 ARN，因此返回值可能比绑定密钥时传入的引用更具体。若希望在看板操作，可参阅[连接 AWS Secrets Manager 保险库](../../how-to/connect-aws-secrets-vault.md#change-the-value-of-a-linked-aws-secret)。



### 外部值写入的验证错误

在任何内容到达提供方之前，下列情况会拒绝向外部引用密钥写值：

| 情况 | 错误消息 |
|---|---|
| 密钥没有可写入的 `externalRef`。 | `External reference secrets require externalRef` |
| 同时发送新 `value` 和不同的 `externalRef`。 | `Provide either a new value or a new external reference, not both` |
| 同时发送 `value` 和 `providerVersionRef`。 | `Value updates cannot pin providerVersionRef` |
| 提供方不支持写入外部密钥。 | `<provider label> does not support writing values to external reference secrets` |

最后一条中的标签来自提供方；例如 GCP Secret Manager 占位提供方会显示 `GCP Secret Manager does not support writing values to external reference secrets`。AWS Secrets Manager 实现了此写入，不会触发该错误。

还有一类拒绝来自 AWS 本身，而非上述通用验证：若引用指向保险库中 Paperclip 管理的命名空间，会返回 `AWS Paperclip-managed namespace secrets cannot be imported as external references`。这些密钥应由 Paperclip 自行轮换，请改为关联你自行管理的密钥。

第二、三条校验体现相同原则：一次请求只做一件事。更改指针和替换值是两种独立轮换；两者都需要时应分别调用。




## 删除密钥

```http
DELETE /api/secrets/{secretId}
```

删除密钥及其版本历史。

这是 API 层的硬删除：

- 密钥记录会被移除。
- 版本记录会随之级联删除。
- 仍指向此密钥的配置今后将无法解析。

确认没有任何配置需要解析此密钥后再删除。

## 在代理配置中使用密钥

代理适配器配置可以在 `env` 中引用密钥，避免内联保存明文。

支持的绑定格式如下：

```json
{
  "env": {
    "ANTHROPIC_API_KEY": {
      "type": "secret_ref",
      "secretId": "secret-uuid",
      "version": "latest"
    }
  }
}
```

也可以固定到数字版本：

```json
{
  "env": {
    "ANTHROPIC_API_KEY": {
      "type": "secret_ref",
      "secretId": "secret-uuid",
      "version": 2
    }
  }
}
```

运行时会发生以下情况：

- Paperclip 会验证密钥属于同一家公司。
- Paperclip 会解析请求的版本。
- Paperclip 会通过提供方解密或获取底层值。
- Paperclip 会将明文注入代理进程的环境变量。

版本规则如下：

- `version: "latest"` 会自动跟踪未来的轮换。
- 数字版本会固定到该特定历史值。
- 如果省略 `version`，Paperclip 会将其视为 `latest`。

出于向后兼容，某些配置仍接受敏感的内联值；但对于任何敏感值，都应优先使用密钥引用。

> **提示：** 预计会轮换的凭据请使用 `version: "latest"`。只有需要代理继续使用已知历史凭据时，才固定数字版本。

<a id="delivery-modes-env-vs-access"></a>

<!-- tabs: cURL, JavaScript, Python -->
<!-- tab: cURL -->
```bash
curl -X POST "http://localhost:3100/api/companies/company-1/agents" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Worker",
    "role": "engineer",
    "adapterType": "claude_local",
    "adapterConfig": {
      "env": {
        "ANTHROPIC_API_KEY": {
          "type": "secret_ref",
          "secretId": "secret-uuid",
          "version": "latest"
        }
      }
    }
  }'
```

<!-- tab: JavaScript -->
```javascript
await fetch("/api/companies/company-1/agents", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${boardToken}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "Worker",
    role: "engineer",
    adapterType: "claude_local",
    adapterConfig: {
      env: {
        ANTHROPIC_API_KEY: {
          type: "secret_ref",
          secretId: "secret-uuid",
          version: "latest",
        },
      },
    },
  }),
});
```

<!-- tab: Python -->
```python
import requests

requests.post(
    "http://localhost:3100/api/companies/company-1/agents",
    headers={
        "Authorization": f"Bearer {board_token}",
        "Content-Type": "application/json",
    },
    json={
        "name": "Worker",
        "role": "engineer",
        "adapterType": "claude_local",
        "adapterConfig": {
            "env": {
                "ANTHROPIC_API_KEY": {
                    "type": "secret_ref",
                    "secretId": "secret-uuid",
                    "version": "latest",
                }
            }
        },
    },
)
```
<!-- /tabs -->

## 交付模式：env 与 access

密钥可以通过多种方式交付给代理；如果关心明文在内存中停留多久，这种差异很重要：

- **环境注入（`env`）**：传统模式。Paperclip 在启动时解析密钥并将明文注入代理进程环境，整个运行期间值都存在。
- **运行绑定访问（`api`）**：通过运行中的代理按需调用的 API 暴露密钥，不预先注入环境；代理仅在需要时读取。
- **`both`**：同一密钥既作为环境变量注入，也可通过运行绑定访问 API 读取。

列出代理获准访问的密钥时，每项 `delivery` 都是 `env`、`api` 或 `both`，代理在获取值前可判断交付方式。仅访问模式授权会标记为 API-only。操作员在代理设置的 **Secret access** 编辑器中管理代理可访问的密钥，并选择作为环境变量还是运行绑定访问交付；目录和选择器界面详见[密钥文件夹](../../administration/secret-folders.md)。

## 运行绑定的代理密钥访问

运行中的代理可通过以下两个路由按需读取已授权密钥，而不必只依赖启动时注入的值。它们仅供代理调用且绑定到运行：调用者令牌必须关联有效且已验证的心跳运行，每个请求都要通过 `secrets:read` 授权检查。技能测试令牌和低信任审核代理无法使用这些 API，避免扩大受限运行的访问范围。

### 列出获准密钥

```http
GET /api/agents/me/secrets
```

只返回获准别名，不返回密钥值。响应 `{ "secrets": [...] }` 中每项包含 `key`、`name`、`description`、`delivery`、`projectionClass`、`latestVersion`、`versionSelector` 和 `resolvedVersion`。密钥 ID、绑定 ID 及配置路径等内部信息会从负载中移除。使用每项的 `key` 作为下方取值路由的别名。

### 读取单个密钥值

```http
POST /api/agents/me/secrets/{key}/value
```

返回单个已授权别名的明文值；`{key}` 是列表路由返回的 key。响应体为 `{ "key", "value", "version" }`，并带 `Cache-Control: no-store`，避免缓存或中间层留存。调用代理未获准该别名时返回禁止访问。每次成功读取都会写入安全审计记录和操作员活动日志，关联到具体代理和运行。

两个路由的完整请求和响应示例见[代理参考](agents.md#fetch-granted-secrets)，因为它们绑定到代理运行。

## 密钥提案

创建密钥及其绑定仅看板可执行，但执行工作的代理往往最先发现缺少凭据。密钥提案填补了这一缺口：运行中的代理可以**提议**需要的密钥，或提议将现有密钥接入某代理配置；之后由看板人员批准、拒绝或任其过期。代理不能自行创建密钥，只能提出请求。

> **代理提议，人来决策。** 代理可以提交提案并查看状态，但只有看板成员能批准。批准时才会真正创建密钥或应用绑定，仅有提案不会改变任何敏感配置。

提案有两种，由 `kind` 字段区分：

- `secret`——“请创建此公司密钥。”代理提供名称、值和理由；批准后，看板会根据提议的值创建正式公司密钥。
- `binding`——“请将一个密钥接入某代理的配置。”代理通过 `secretId` 指向已有密钥，或通过 `secretProposalId` 指向另一个待处理的密钥提案，并指定要绑定到的 `configPath`。批准后，看板会在目标代理的适配器配置中写入 `secret_ref`。

绑定可以依赖尚未批准的密钥提案，因此代理可先提议密钥，再创建通过 ID 引用该提案的绑定提案。看板可以一起批准，见[批准提案](#批准提案)。

### 谁可以提议

代理端路由要求**已验证且绑定到运行的代理令牌**：必须满足[运行绑定代理密钥访问](#运行绑定的代理密钥访问)的活动心跳条件，还必须是有效签名的代理 JWT。任务桥接令牌和技能测试令牌会被拒绝，运行还须通过 `secrets:propose` 授权操作检查。否则响应为：

```json
403 Forbidden
{ "error": "Secret proposals require a verified run-bound agent token" }
```

### 提议密钥或绑定

```http
POST /api/agents/me/secret-proposals
Content-Type: application/json
```

请求体由 `kind` 字段区分。

**提议 `secret`：**

| 字段 | 必填 | 说明 |
|---|---|---|
| `kind` | 是 | `"secret"`。 |
| `name` | 是 | 以斜杠分隔的路径，不能有空段，例如 `anthropic/api-key`。 |
| `value` | 是 | 明文值，最多 `65536` 字节；会立即加密，并登记为运行日志删减值。 |
| `justification` | 是 | 说明代理为何需要此值；看板可见。 |
| `key` | 否 | 环境变量风格的键；默认为规范化后的 `name` 最后一段。 |
| `description` | 否 | 面向操作员的备注；创建密钥时会一并保存。 |

**提议 `binding`：**

| 字段 | 必填 | 说明 |
|---|---|---|
| `kind` | 是 | `"binding"`。 |
| `configPath` | 是 | 绑定路径；必须为 `env.<KEY>` 或 `access.<ALIAS>`。 |
| `secretId` | 二选一 | 要绑定的现有活动公司密钥。 |
| `secretProposalId` | 二选一 | 批准后要绑定的待处理 `secret` 提案。`secretId` 与此字段必须且只能提供一个。 |
| `targetAgentId` | 否 | 绑定目标；默认为提议代理自身，只能是提议者或其下属。 |
| `justification` | 是 | 说明为何需要此绑定。 |

绑定目标策略由服务器固定为 `self_and_reports`：代理只能将密钥绑定给自己或汇报给自己的代理，不能指定汇报链上的更高层。超出范围时会被拒绝：

```json
403 Forbidden
{ "error": "Binding proposals may target only the proposing agent or its reports" }
```

成功返回 `201 Created` 及提案记录，字段见[提案字段](#提案字段)。提议值不会原样返回；代理视图也会删减值指纹和长度。

```bash
# Propose a secret
curl -X POST "http://localhost:3100/api/agents/me/secret-proposals" \
  -H "Authorization: Bearer <agent-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "secret",
    "name": "anthropic/api-key",
    "value": "sk-ant-...",
    "justification": "Needed to call the Anthropic API for this issue"
  }'
```

```bash
# Propose a binding onto this agent's env
curl -X POST "http://localhost:3100/api/agents/me/secret-proposals" \
  -H "Authorization: Bearer <agent-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "binding",
    "secretId": "secret-uuid",
    "configPath": "env.ANTHROPIC_API_KEY",
    "justification": "Wire the shared key into my runtime"
  }'
```

提案设有频率和数量限制：每个代理最多只能有 `20` 个待处理提案，且每分钟最多创建 `20` 个。超出任一限制会返回 `422`：

```json
422 Unprocessable Entity
{ "error": "Agents may have at most 20 pending secret proposals" }
```

### 列出自己的提案

```http
GET /api/agents/me/secret-proposals
```

按时间倒序返回代理提交的提案，以及目标为该代理的 `binding` 提案，绝不包含值。响应为 `{ "proposals": [...], "nextOffset": 100 }`。使用 `limit`（默认 `100`，最大 `200`）和 `offset` 分页；也通过 `X-Page-Limit`、`X-Page-Offset`，以及还有后续页时的 `X-Next-Offset` 响应头报告分页。末页 `nextOffset` 为 `null`。非数字 `limit` / `offset` 返回 `422`（`limit must be a positive integer` / `offset must be a non-negative integer`）。

```json
{
  "proposals": [ /* proposal records */ ],
  "nextOffset": 100
}
```

### 撤回提案

```http
DELETE /api/agents/me/secret-proposals/{id}
```

提案创建者可在待处理时撤回。撤回后状态变为 `withdrawn`，存储密文会清除；若是 `secret` 提案，依赖它的待处理 `binding` 提案也会级联为 `rejected`。只有原提议者可撤回，且只有待处理提案可解决；否则分别返回 `403`（`Only the proposer can withdraw this proposal`）或 `409`（`Only pending proposals can be resolved`）。

```json
403 Forbidden
{ "error": "Only the proposer can withdraw this proposal" }
```

```json
409 Conflict
{ "error": "Only pending proposals can be resolved" }
```

### 审核提案

```http
GET /api/companies/{companyId}/secret-proposals
```

看板端列表也显示在 **Company settings → Secrets → Proposals** 下，待处理数量会显示为标签计数。路由按时间倒序返回公司提案的普通数组（不包装在对象中）。传 `?status=` 可按 `pending`、`approved`、`rejected`、`withdrawn` 或 `expired` 过滤。分页参数 `limit` / `offset` 及 `X-Page-*` 响应头与代理列表一致。

![公司密钥设置中的“提案”标签页：一条待处理提案显示路径形式的名称、提议代理、值的指纹和字节长度、过期时间、代理说明的理由，以及“批准”和“拒绝”按钮](../../../user-guides/screenshots/light/secrets/proposals-tab.png)

每行都包含便于审核的扩展信息：除提案字段外，还包括 `proposedBy`、绑定目标代理 `target`、来源 issue `originIssue`，以及任何被引用密钥的解析后名称 `secretName` 或前置密钥提案的名称 `secretProposalName`。还包括两个审核提示：

- `viewerCanApprove`——表示发起此请求的看板成员能否批准此提案。
- `approveBlockReason`——若不能批准，提供简短原因；可以批准时为 `null`。

批准 `secret` 提案需要公司管理员权限，因此非管理员看板成员会看到如下响应：

```json
{
  "viewerCanApprove": false,
  "approveBlockReason": "Company admin access required"
}
```

对于 `binding` 提案，能否批准取决于你是否有权更改**目标代理**的配置（`agent_config:update` 变更授权），因此阻止原因会反映该授权判断。任何已不再处于待处理状态的提案都会返回 `viewerCanApprove: false` 和 `approveBlockReason: "Proposal is no longer pending"`。

### 批准提案

```http
POST /api/companies/{companyId}/secret-proposals/{id}/approve
Content-Type: application/json
```

批准会实际应用提案。对于 `secret`，看板从提议值创建真正的公司密钥，并记录其 ID 为 `createdSecretId`；对于 `binding`，看板在目标代理适配器配置的指定 `configPath` 写入 `secret_ref`，并记录 `appliedBindingConfigPath`。两种情况下提案状态都会变为 `approved`、存储密文会清除，并写入 `resolvedByUserId` 和 `resolvedAt`。

请求体字段：

| 字段 | 必填 | 说明 |
|---|---|---|
| `cascade` | 否 | 当绑定依赖仍待处理的 `secret` 提案时，设为 `cascade: true` 可在同一事务中批准前置提案，并绑定其创建的密钥。 |
| `overrides.name` | 否 | 在批准时重命名新建的密钥。 |
| `overrides.description` | 否 | 覆盖新建密钥的说明。 |
| `overrides.providerConfigId` | 否 | 将新建密钥存入指定提供方保险库，而非 `local_encrypted`。 |

可批准者与 `viewerCanApprove` 指示相同：`secret` 提案要求公司管理员权限；`binding` 提案要求对目标代理具有变更授权。

若提案过期，返回 `409 Conflict` 和 `Expired proposals cannot be approved`。若绑定所依赖的提案仍待处理，必须传 `cascade: true`，否则返回 `Binding proposal requires pending secret proposal <id>; retry with cascade=true`。若绑定目标已不在提议者的汇报链内，会返回 `Binding proposal target is no longer allowed by its proposal-time and current chain-of-command policy`。这些情况都安全失败，不会部分应用。

```bash
curl -X POST "http://localhost:3100/api/companies/company-1/secret-proposals/proposal-uuid/approve" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{}'
```

### 拒绝提案

```http
POST /api/companies/{companyId}/secret-proposals/{id}/reject
Content-Type: application/json
```

拒绝会将状态设为 `rejected`、清除密文并记录理由。请求字段如下：

| 字段 | 必填 | 说明 |
|---|---|---|
| `reason` | 是 | 不能为空；会保存在提案中并显示给来源 issue。 |

拒绝 `secret` 提案时，所有依赖它的待处理 `binding` 提案也会级联拒绝，避免留下悬空绑定。

理由为空时返回 `422` 和 `{ "error": "Rejection reason is required" }`。

```bash
curl -X POST "http://localhost:3100/api/companies/company-1/secret-proposals/proposal-uuid/reject" \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{ "reason": "Use the shared vault key instead" }'
```

```json
422 Unprocessable Entity
{ "error": "Rejection reason is required" }
```

### 提案解决后的行为

提案通常在一项工作中创建。提出时，Paperclip 会解析当前运行对应的 issue（其执行运行或检出运行与当前运行匹配），并保存为 `originIssueId`。

看板批准或拒绝时，若提案有 `originIssueId`，Paperclip 会在该 issue 发布解决结果评论并唤醒受派代理，让代理获取决策。评论会写出提案名称和新状态；拒绝时会附上原因。通知是尽力而为：评论或唤醒失败会记录日志，但不影响解决结果。

```
Secret proposal resolution

- Proposal: secret proposal `anthropic/api-key`
- Status: **approved**
```

### 状态与过期

提案的 `status` 始终为以下值之一：

- `pending`——等待决定。只有此状态可被批准、拒绝或撤回。
- `approved`——密钥已创建，或绑定已应用。
- `rejected`——被看板拒绝，或因前置提案被拒绝而级联拒绝。
- `withdrawn`——由提出提案的代理撤回。
- `expired`——待处理提案已过期。提案在提出 `14` 天后过期，后台扫描任务会将其状态改为 `expired`。

提案一旦离开 `pending`，加密值会被清除并写入 `ciphertextScrubbedAt`。批准的密钥会作为真正的公司密钥保留，提案记录不会继续保存敏感值。

### 提案字段

每个提案记录结构相同，不适用的字段为 `null`。代理视图中的 `secret` 提案示例：

```json
{
  "id": "proposal-uuid",
  "companyId": "company-1",
  "kind": "secret",
  "status": "pending",
  "proposedName": "anthropic/api-key",
  "proposedKey": "API_KEY",
  "proposedDescription": "Key the billing agent needs",
  "justification": "Needed to call the Anthropic API for this issue",
  "secretId": null,
  "secretProposalId": null,
  "targetType": null,
  "targetId": null,
  "configPath": null,
  "projectionClass": "unclassified",
  "proposedByAgentId": "agent-uuid",
  "originIssueId": "issue-uuid",
  "originRunId": "run-uuid",
  "resolvedByUserId": null,
  "resolvedAt": null,
  "resolutionReason": null,
  "createdSecretId": null,
  "appliedBindingConfigPath": null,
  "ciphertextScrubbedAt": null,
  "expiresAt": "2026-09-01T12:00:00.000Z",
  "createdAt": "2026-08-18T12:00:00.000Z",
  "updatedAt": "2026-08-18T12:00:00.000Z",
  "secretName": null,
  "secretProposalName": null,
  "proposedBy": { "id": "agent-uuid", "name": "Billing", "icon": "…" },
  "target": null,
  "originIssue": { "id": "issue-uuid", "key": "PC-42", "title": "Wire up billing" }
}
```

| 字段 | 说明 |
|---|---|
| `id` | 提案 ID。 |
| `companyId` | 所属公司。 |
| `kind` | `secret` 或 `binding`。 |
| `status` | 上述状态之一。 |
| `proposedName` / `proposedKey` / `proposedDescription` | 所请求密钥的名称、key 和备注；仅用于 `secret` 提案。 |
| `justification` | 代理说明的理由。 |
| `secretId` | 绑定现有密钥时使用。 |
| `secretProposalId` | 绑定仍待处理的密钥提案时使用。 |
| `targetType` / `targetId` / `configPath` | 绑定目标：`agent`、代理 ID，以及 `env.<KEY>` / `access.<ALIAS>` 路径；仅用于 `binding` 提案。 |
| `originIssueId` / `originRunId` | 提案来源 issue 和运行。 |
| `resolvedByUserId` / `resolvedAt` / `resolutionReason` | 解决提案的用户、时间和原因（拒绝时）。 |
| `createdSecretId` | 批准 `secret` 提案时创建的公司密钥。 |
| `appliedBindingConfigPath` | 批准 `binding` 提案时写入的路径。 |
| `ciphertextScrubbedAt` | 密文被清除的时间（任何非待处理提案）。 |
| `expiresAt` / `createdAt` / `updatedAt` | 生命周期时间戳。 |
| `secretName` / `secretProposalName` | 被引用密钥或前置提案的解析后显示名称。 |
| `proposedBy` / `target` / `originIssue` | 提议代理、目标代理和来源 issue 的扩展摘要。 |

看板视图还增加 `viewerCanApprove` 与 `approveBlockReason`；与代理视图不同，它会保留 `valueFingerprintSha256` 和 `valueLength`，供审核者核对提议值是否合理，但仍不显示明文。原始密文和汇报关系快照对两种受众都不会返回。

<a id="secret-ref-form-fields"></a>
## `secret-ref` 表单字段

部分配置由服务器发布的 JSON Schema 驱动，而非手写，界面会根据 schema 渲染表单。此类 schema 中字符串字段若声明 `"format": "secret-ref"`，界面会把普通文本输入框替换为密钥绑定选择器：

```json
{
  "type": "object",
  "properties": {
    "githubToken": {
      "type": "string",
      "format": "secret-ref",
      "title": "GitHub token"
    }
  }
}
```

界面会显示：

- 当前公司的活动密钥下拉列表。
- “粘贴原始值”回退选项，供尚未保存密钥的情况使用。
- 选择器会将 UUID 格式的值识别为已绑定的密钥引用；其他值均视为原始值，Paperclip 会在保存时将其存为新密钥。

例行任务 `env` 值、代理适配器 `env` 和其他通过 `format: "secret-ref"` 启用此功能的配置界面，都使用同一选择器。编写插件或适配器 schema 时，只要为敏感字段标记此格式，就会自动获得绑定选择器。

例行任务侧的示例见[例行任务环境映射](../../how-to/create-a-daily-routine.md#4-optional-give-the-routine-an-env-map)。

## 实用说明

- 密钥名称在公司内必须唯一。
- 创建时版本为 `1`，轮换会递增版本号。
- `update` 只更改元数据，不更改已保存的值。
- `rotate` 创建新值并更新最新版本指针。
- 对 `external_reference` 密钥，`rotate` 可更改引用；若提供方声明支持 `supportsExternalValueWrites`，也可将新值写入外部保险库。一个请求不能同时执行两项操作。
- 普通本地部署默认提供方为 `local_encrypted`。
- `GET /api/companies/{companyId}/secret-providers` 会列出外部提供方，但当前部署中只有已配置的提供方可实际使用。
- API 总体按公司隔离；密钥管理端点由看板操作员使用，代理访问例外见[运行绑定的代理密钥访问](#运行绑定的代理密钥访问)和[密钥提案](#密钥提案)。

---
