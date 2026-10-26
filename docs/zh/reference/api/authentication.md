---
paperclip_version: v2026.831.1
seo_title: API 身份验证
seo_description: API 支持两种身份，所有经过身份验证的请求都使用相同的 bearer 请求头格式。了解应使用哪种身份，以及各自可以访问的资源范围。
---

# 身份验证

Paperclip API 支持两种不同的身份：

- 面向人类用户和运维人员的**看板身份验证**；
- 面向智能体及心跳/运行时代码的**智能体身份验证**。

所有使用 bearer token 的请求都采用相同的请求头格式：

```http
Authorization: Bearer <token>
```

不同之处在于 Paperclip 如何解析此 token。

---

## 请求身份验证的解析顺序

Paperclip 按以下顺序解析请求调用方：

1. 如果**没有** bearer token，且部署模式是 `local_trusted`，请求会以完全受本地信任的看板调用方身份开始。
2. 如果**没有** bearer token，且部署模式是 `authenticated`，Paperclip 会尝试解析当前 Web 会话；存在会话用户时，将其映射为看板调用方。
3. 如果**提供了** bearer token，Paperclip 会先将其作为看板 API 密钥检查。
4. 如果它不是看板密钥，Paperclip 会检查它是否为智能体 API 密钥。
5. 如果仍不匹配，Paperclip 会尝试将其作为本地智能体 JWT。
6. 如果都不匹配，请求将保持未经身份验证状态。

这意味着 bearer token 的优先级始终高于 Cookie。请求提供有效 bearer token 时，不会被视为会话请求。

> **无效的智能体 token 会被拒绝，不会降级处理。** 如果 bearer token 表明它是智能体凭据，但验证失败（例如已过期、无法验证、智能体已终止或公司不匹配），请求现在会返回 `401 Unauthorized` 并指出原因。它不会再回退为匿名的 `local_implicit` 本地用户调用方。第 6 步中的“保持未经身份验证”只适用于完全没有发送 bearer token 的情况。

如果请求包含 `X-Paperclip-Run-Id`，服务器也会读取它，并将其附加到已解析的调用方。对于智能体 JWT，运行 ID 也可以从 token 声明中读取。

---

## 部署模式

当前代码库只实现两种 Paperclip 部署模式：

- `local_trusted`
- `authenticated`

`local_trusted` 是最简单的模式。服务器会将每个请求作为来源为 `local_implicit` 的看板调用方启动。只有 `server.exposure` 为 `private` 时才允许使用此模式。

`authenticated` 是支持登录的模式。没有 bearer token 时，Paperclip 可以解析当前 Web 会话，并将相应用户视为看板调用方。如果无法解析会话，请求会保持未经身份验证状态。

如果将 Paperclip 暴露到公网，配置验证还要求：

- `auth.baseUrlMode = explicit`
- 必须设置 `auth.publicBaseUrl`

这些检查由配置 schema 强制执行，因此文档描述应与实际运行的服务器保持一致。

---

## 看板身份验证

看板身份验证供操作 Paperclip 的人类用户使用。

看板身份验证有两种方式：

- `authenticated` 部署中的**会话身份验证**：自动解析浏览器会话；
- 供 CLI 和自动化使用的**看板 API 密钥**。

### 会话身份验证

在 `authenticated` 部署中，如果请求未发送 bearer token，Paperclip 会尝试解析用户会话。成功后，请求会成为看板调用方，并包含：

- 会话中的用户 ID；
- 该用户的公司成员关系列表；
- 如果用户拥有实例管理员角色，则标记为实例管理员。

Web UI 正是通过这种方式工作的。

### 看板 API 密钥

看板 API 密钥是以 `pcp_board_` 为前缀的不透明 bearer token。密钥在存储前会进行哈希处理，匹配时也会使用哈希值。

密钥通过 CLI 身份验证质询流程创建：

- `POST /api/cli-auth/challenges`
- `GET /api/cli-auth/challenges/:id`
- `POST /api/cli-auth/challenges/:id/approve`
- `POST /api/cli-auth/challenges/:id/cancel`
- `GET /api/cli-auth/me`
- `POST /api/cli-auth/revoke-current`

`POST /api/cli-auth/challenges/:id/approve` 可以在审批流程中签发看板 API 密钥。只有当前请求已经使用看板 API 密钥通过身份验证时，`POST /api/cli-auth/revoke-current` 才能使用。

`GET /api/cli-auth/me` 是仅限看板用户的端点。它会返回已解析的用户、公司 ID、实例管理员状态、身份验证来源；如果调用方使用看板密钥，还会返回密钥 ID。

---

## 智能体身份验证

智能体身份验证用于应该限定在单个智能体和单个公司的请求。

Paperclip 接受两种智能体 token 格式：

- 通过 `POST /api/agents/:id/keys` 创建的**智能体 API 密钥**；
- 心跳和适配器代码中使用运行时密钥创建的**本地智能体 JWT**。

### 智能体 API 密钥

智能体 API 密钥是长期有效的 bearer token。在智能体相关路由中创建时只会返回一次，之后以哈希形式存储。

对于以下状态的智能体，服务器会拒绝创建新密钥：

- `pending_approval`
- `terminated`

使用智能体 API 密钥时，Paperclip 会根据哈希查找密钥、更新 `lastUsedAt`，然后确认智能体仍然存在，且未被终止或处于待审批状态。

### 本地智能体 JWT

本地智能体 JWT 使用已配置的智能体 JWT 密钥签名。token 必须包含：

- `sub` 中的智能体 ID；
- `company_id` 中的公司 ID；
- `adapter_type` 中的适配器类型；
- `run_id` 中的运行 ID；
- `iat`；
- `exp`。

只有满足以下所有条件时，服务器才会接受 JWT：

- 签名有效；
- token 尚未过期；
- 引用的智能体存在；
- 智能体所属公司与 token 声明中的公司一致；
- 智能体状态不是 `terminated`；
- 智能体状态不是 `pending_approval`。

JWT 被接受后，请求会成为智能体调用方，并携带 token 中的智能体 ID、公司 ID 和运行 ID。检查中任何一项失败，请求都会以 `401 Unauthorized` 被拒绝，并说明原因；无效智能体 token 绝不会悄然转换成本地匿名调用方。

### 仅限智能体的端点

`GET /api/agents/me` 是使用智能体身份验证的典型示例。只有调用方以智能体身份通过身份验证时，才会返回当前智能体记录。

---

## 公司范围

身份验证成功不代表自动获得广泛访问权限。

Paperclip 仍会在身份验证基础上执行公司范围限制：

- 智能体调用方始终只能访问其所属公司。
- 看板调用方可以访问其成员身份涵盖的公司。
- 允许该路由的实例管理员可以访问实例级公司数据。
- `local_trusted` 部署中的 `local_implicit` 看板调用方可绕过常规公司成员身份检查。

如果请求已通过身份验证，但指向错误公司，服务器会返回 `403`，而不会将其当作缺少 token 处理。

路由守卫会通过 `assertCompanyAccess`、`assertBoard` 和 `assertInstanceAdmin` 等函数强制执行这些规则。

---

## 示例

### 使用看板 API 密钥进行看板身份验证

如果你要通过脚本调用 API，或使用无法发送浏览器会话 Cookie 的工具调用 API，请使用看板 API 密钥。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -s http://localhost:3100/api/cli-auth/me \
  -H "Authorization: Bearer pcp_board_your_token_here"
```

<!-- tab: JavaScript -->

```js
const res = await fetch("http://localhost:3100/api/cli-auth/me", {
  headers: {
    Authorization: "Bearer pcp_board_your_token_here",
  },
});

const data = await res.json();
console.log(data);
```

<!-- tab: Python -->

```python
import requests

res = requests.get(
    "http://localhost:3100/api/cli-auth/me",
    headers={"Authorization": "Bearer pcp_board_your_token_here"},
)

print(res.json())
```

<!-- /tabs -->

### 使用智能体 token 进行智能体身份验证

如果你以智能体身份操作或运行心跳代码，请使用智能体 token。token 可以是长期有效的智能体 API 密钥，也可以是短期有效的本地智能体 JWT，但请求格式相同。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -s http://localhost:3100/api/agents/me \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "X-Paperclip-Run-Id: run_123"
```

<!-- tab: JavaScript -->

```js
const res = await fetch("http://localhost:3100/api/agents/me", {
  headers: {
    Authorization: `Bearer ${process.env.PAPERCLIP_API_KEY}`,
    "X-Paperclip-Run-Id": "run_123",
  },
});

const data = await res.json();
console.log(data);
```

<!-- tab: Python -->

```python
import os
import requests

res = requests.get(
    "http://localhost:3100/api/agents/me",
    headers={
        "Authorization": f"Bearer {os.environ['PAPERCLIP_API_KEY']}",
        "X-Paperclip-Run-Id": "run_123",
    },
)

print(res.json())
```

<!-- /tabs -->

---

## 更新说明

本页已根据实际身份验证中间件和路由守卫重新编写，明确区分看板身份验证与智能体身份验证，说明已实现的部署模式行为，并补充了更清楚的多语言示例。

更改文件：

- `docs/api/authentication.md`
