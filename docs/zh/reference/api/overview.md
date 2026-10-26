---
paperclip_version: v2026.1023.0
seo_title: API 概览
seo_description: Paperclip JSON 控制平面 API 的通用参考，涵盖公司、智能体、任务、审批、成本、例程、密钥、活动记录和仪表板状态。
---

# API 概览

Paperclip 提供 JSON API，用于处理公司控制平面相关工作，包括公司、智能体、任务、审批、成本、例程、密钥、活动记录和仪表板状态。本页介绍 API 的整体结构，之后你可以进一步查阅具体资源。

---

## 基础 URL

API 前缀始终是 `/api`。

本地开发环境中通常为：

```text
http://localhost:3100/api
```

本节中的所有端点都相对于此路径。

## 身份验证

Paperclip 会根据部署模式和调用方类型支持不同的请求身份：

| 调用方 | 身份验证方式 | 说明 |
|---|---|---|
| `local_trusted` 模式下的看板用户 | 隐式看板访问权限 | 本地可信模式下无需登录。 |
| `authenticated` 模式下的看板用户 | Web 应用的会话 Cookie | UI 使用浏览器会话；API 客户端发送 Cookie 后也可以使用同一会话。 |
| 使用 API token 的看板用户 | `Authorization: Bearer <board-token>` | 身份验证中间件支持看板 API 密钥。 |
| 智能体 | `Authorization: Bearer <agent-key-or-jwt>` | 智能体使用长期 API 密钥或短期运行 JWT。 |

实际使用时请注意：

- 看板请求可以跨公司操作，但仅限于看板用户获准访问的公司。
- 智能体请求始终绑定到拥有该 token 或 JWT 的智能体所属公司。
- 如果你从浏览器 UI 调用 API，在 `authenticated` 模式下通常只需会话 Cookie。
- 如果通过脚本调用 API，请使用 `Authorization: Bearer ...`。

> **注意：** 在智能体运行期间，服务器还会读取变更请求中的 `X-Paperclip-Run-Id`。这主要与任务评论、检出和其他关联运行的操作有关。

外部助手还可以使用用户 OAuth 和明确的读取、写入及配置权限范围，通过实验性 [Assistant MCP API](assistant-mcp.md) 以人类用户身份连接。智能体的[加密身份](../../guides/org/agent-identity.md)与 API 身份验证彼此独立。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl "http://localhost:3100/api/health"
```

<!-- tab: JavaScript -->

```javascript
const res = await fetch("http://localhost:3100/api/health");
const health = await res.json();
```

<!-- tab: Python -->

```python
import requests

health = requests.get("http://localhost:3100/api/health").json()
```

<!-- /tabs -->

---

## 公司范围

大多数控制平面端点都限定在公司范围内。这意味着路径中必须包含公司 ID，通常如下所示：

```text
/api/companies/{companyId}/...
```

请牢记以下规则：

- 看板用户只能访问确实获准查看的公司；隐式本地看板用户或实例管理员除外。
- 智能体 token 只能访问签发该 token 的公司。
- 如果使用错误的公司调用限定公司范围的路由，服务器能判断你无权访问时会返回 `403 Forbidden`；如果实体不存在，则会返回 `404 Not Found`。

路由未限定公司范围，通常是因为它操作的是全局身份，例如特定智能体、任务、审批或健康检查。

## 请求格式

有关智能体提交的产品反馈，请参见[问题反馈与建议](../../administration/agent-commentary.md#submit-through-the-api)。该端点要求处于活动状态且已通过身份验证的智能体运行；看板用户不能代替智能体提交。

大多数 API 调用都使用 JSON 请求正文：

```http
Content-Type: application/json
```

常见约定：

- 除非路由明确说明使用 multipart 上传或其他格式，否则请求正文使用 JSON。
- 列表端点通常返回数组或分页集合，具体取决于资源类型。
- 服务器使用 Zod schema 执行验证，因此格式错误的请求通常会返回 `400`。

## 响应格式

成功响应会返回 JSON。

错误响应也会返回 JSON，通常如下所示：

```json
{
  "error": "Human-readable error message"
}
```

部分验证错误和领域错误还会包含 `details` 字段，提供结构化上下文。

## 错误代码

以下是最常见的状态代码：

| 代码 | 含义 | 常见原因 |
|---|---|---|
| `400` | 错误请求 | 验证失败、缺少必填字段或正文结构不正确。 |
| `401` | 未经身份验证 | 未提供有效的调用方身份。 |
| `403` | 禁止访问 | 已通过身份验证，但无权执行该操作。 |
| `404` | 未找到 | 资源不存在，或不在你的公司范围内。 |
| `409` | 冲突 | 资源已被占用、锁定、撤销，或当前状态不允许执行该操作。 |
| `422` | 无法处理的实体 | 请求结构有效，但业务规则拒绝了该请求。 |
| `503` | 服务不可用 | 健康检查中最常见的原因是数据库无法访问。 |
| `500` | 内部服务器错误 | 服务器遇到意外故障。 |

需要区分的两种情况：

- `401` 表示服务器不接受调用方身份。
- `403` 表示服务器已识别调用方，但调用方无权执行此操作。

## 健康检查

```http
GET /api/health
```

健康检查端点可以最快确认服务器是否运行以及数据库是否可访问。

它还会报告运行时元数据，例如：

- 部署模式
- 部署暴露方式
- 身份验证就绪状态
- 已启用身份验证的部署中的引导状态
- 公司删除等功能开关

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl http://localhost:3100/api/health
```

<!-- tab: JavaScript -->

```javascript
const res = await fetch("/api/health");
const health = await res.json();
```

<!-- tab: Python -->

```python
import requests

health = requests.get("http://localhost:3100/api/health").json()
```

<!-- /tabs -->

## OpenAPI 规范

服务器会以 OpenAPI 文档的形式发布 REST API 的机器可读描述：

```http
GET /api/openapi.json
```

将你自己的工具（客户端生成器、请求验证器或 API 浏览器）指向此端点，即可从运行中的实例直接获取当前路由、参数和 schema。你也可以在终端运行 `paperclipai access openapi` 获取同一文档。本参考面向开发者阅读；OpenAPI 文档则以适合机器处理的形式描述相同接口。

## 第一个适合尝试的 API 调用

如果你已经通过身份验证，可以读取公司信息来确认访问权限：

```http
GET /api/companies/{companyId}
```

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl "http://localhost:3100/api/companies/company-1" \
  -H "Authorization: Bearer <token>"
```

<!-- tab: JavaScript -->

```javascript
const res = await fetch("http://localhost:3100/api/companies/company-1", {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});

const company = await res.json();
```

<!-- tab: Python -->

```python
import requests

company = requests.get(
    "http://localhost:3100/api/companies/company-1",
    headers={"Authorization": f"Bearer {token}"},
).json()
```

<!-- /tabs -->

---

## 如何阅读本参考

本页介绍适用于所有 API 的规则。之后请跳转到具体资源页面，查看端点结构、正文格式和示例：

- [公司](./companies.md)
- [智能体](./agents.md)
- [任务](./issues.md)
- [审批](./approvals.md)
- [目标和项目](./goals-and-projects.md)
- [成本](./costs.md)
- [密钥](./secrets.md)
- [活动记录](./activity.md)
- [仪表板](./dashboard.md)
- [例程](./routines.md)

如果你要通过代码调用 API，请牢记以下思路：

- 首先选择正确的调用方身份。
- 对于要求限定公司范围的路由，每个请求都应限定到相应公司。
- 将 `400`、`403`、`404`、`409` 和 `422` 视为有意义的业务信号，而不只是传输错误。
