---
paperclip_version: v2026.720.0
seo_title: 收件箱智能体策略 API
seo_description: 控制智能体可以如何管理你收件箱的个人开关，包括归档、清理和整理待处理事项队列，并可随时收回权限。
---

# 收件箱智能体策略

智能体可以操作你的收件箱，例如归档事项、清理内容和整理待处理事项队列。这些功能有时很实用，有时也可能不合适。**收件箱智能体策略**是你的个人开关：你可以针对每家公司决定是否允许智能体管理收件箱，以及具体允许哪些智能体。

每项策略属于某个公司中的一位用户。你可以通过 `me` 路由管理自己的策略；公司管理员则可以通过按用户路由读取或设置其他成员的策略。

下列所有路由都限定在公司范围内，并且每次请求都会检查公司访问权限。如果你刚开始使用 API，请先阅读 [API 概览](./overview.md)，了解基础 URL、身份验证和错误代码约定；本页会沿用这些规则，不再重复说明。

---

## 三种模式

| 模式 | 含义 |
|---|---|
| `open` | 任何智能体都可以管理你的收件箱。这是默认模式。 |
| `allowlist` | 只有列在 `allowedAgentIds` 中的智能体可以管理你的收件箱。 |
| `disabled` | 禁止所有智能体管理你的收件箱。 |

`open` 不只是初始值；根本没有策略记录时也会采用此模式。

---

## 策略对象

```json
{
  "companyId": "…",
  "userId": "…",
  "mode": "open",
  "allowedAgentIds": [],
  "materialized": false,
  "createdAt": null,
  "updatedAt": null
}
```

| 字段 | 说明 |
|---|---|
| `mode` | `open`、`allowlist` 或 `disabled`。 |
| `allowedAgentIds` | 智能体 UUID。除非 `mode` 为 `allowlist`，否则始终为 `[]`。 |
| `materialized` | 尚未保存策略、当前显示默认值时为 `false`；策略写入后为 `true`。 |
| `createdAt` / `updatedAt` | 创建和更新的时间戳；尚未写入策略时为 `null`。 |

---

## 你自己的策略

```
GET /api/companies/{companyId}/users/me/inbox-agent-policy
PUT /api/companies/{companyId}/users/me/inbox-agent-policy
```

这些路由只要求公司访问权限，不需要额外的权限键。你始终可以管理自己的收件箱策略。

但它们要求**看板用户上下文**：当前身份必须是携带用户 ID 的看板调用方。没有此上下文的调用方（例如智能体 API 密钥）会收到 `401` 和 `{ "error": "Board user context required" }`，因为系统无法确定“我”指的是谁。

### 示例

```bash
curl -sS -X PUT \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/companies/{companyId}/users/me/inbox-agent-policy" \
  -d '{
    "mode": "allowlist",
    "allowedAgentIds": ["6d2f1c8e-1f4b-4a51-9f0b-2c2a4c9c8f11"]
  }'
```

---

## 其他用户的策略

```
GET /api/companies/{companyId}/users/{userId}/inbox-agent-policy
PUT /api/companies/{companyId}/users/{userId}/inbox-agent-policy
```

管理员可在成员入职时查看或设置其策略，也可以在成员离开期间关闭其收件箱的智能体操作权限。

两个路由都要求具有**收件箱智能体策略管理权限**，即符合以下任一条件：

- 调用方是来自隐式本地会话的看板用户，或是实例管理员；
- 调用方是在该公司持有 `users:manage_permissions` 权限的看板用户；
- 调用方是在该公司持有 `users:manage_permissions` 权限的智能体。

其他调用方会收到 `403 Forbidden`、正文 `{ "error": "Inbox agent policy administration authority required" }` 和错误码 `inbox_agent_policy_admin_required`。

通过权限检查后，目标用户必须是公司的**活动成员**。如果用户没有成员关系，或成员状态不是 `active`，请求会返回 `404` 和 `{ "error": "Active company user membership not found" }`。

> **注意：** 这些路由检查的是 `users:manage_permissions`，而不是 `inbox:manage`。权限目录中确实存在 `inbox:manage`，但它管理的是智能体是否可以*操作*收件箱，而不是智能体能否编辑策略。参见下文[策略实际管控的对象](#who-the-policy-actually-governs)。

---

## 更新策略

两个 `PUT` 路由接受相同的正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `mode` | 是 | `open`、`allowlist` 或 `disabled`。 |
| `allowedAgentIds` | 否 | 最多包含 100 个智能体 UUID 的数组，默认为 `[]`。 |

规则：

- 不接受未知字段。
- 除非 `mode` 为 `allowlist`，否则 `allowedAgentIds` 必须为空。若在 `open` 或 `disabled` 模式下仍发送智能体 ID，验证会失败并返回 *"allowedAgentIds must be empty when mode is not \"allowlist\""*。
- 验证失败时会返回 `400 Bad Request` 和 `{ "error": "Validation error", "details": [...] }`。
- 保存前会去除重复的智能体 ID。
- 允许列表中的每个智能体都必须属于当前公司。否则会返回 `422 Unprocessable Entity`、正文 `{ "error": "Inbox agent policy contains agents outside the company" }`、错误码 `inbox_agent_policy_invalid_agents` 和列出无效智能体的 `invalidAgentIds` 数组。

从 `allowlist` 切换到其他模式会清空列表。将 `mode` 设为 `open` 或 `disabled` 时，`allowedAgentIds` 会保存为空数组；之后若切回 `allowlist`，你需要重新指定智能体。

无论通过 `me` 路由还是按用户路由成功写入策略，系统都会在实体类型为 `user_inbox_agent_policy` 的对象上记录 `inbox.agent_policy_updated` 活动条目，其中包含受影响的 `userId`、`previousMode`、新 `mode` 和最终的 `allowedAgentIds`。

---

<a id="who-the-policy-actually-governs"></a>
## 策略实际管控的对象

智能体尝试执行 `inbox:manage` 操作时会检查此策略；具体规则取决于被操作的是谁的收件箱。

**智能体所代表的用户。** 智能体操作其运行时所代表用户的收件箱时，由该用户的策略决定是否允许：

- `disabled` → 拒绝，原因是 `inbox_management_disabled`。
- 模式为 `allowlist` 且智能体不在列表中 → 拒绝，原因是 `inbox_agent_not_allowed`。
- 其他情况 → 允许，同时附带生效的 `inboxPolicyMode`（无已保存策略时为 `open`）。

**其他用户。** 访问另一位成员的收件箱属于看板管理员覆盖操作，目标用户的策略不会因此授予权限。智能体必须在公司中拥有明确的 `inbox:manage` 授权，且授权范围必须覆盖目标用户。没有授权时，拒绝原因为 `deny_missing_grant`；若有授权但其范围未覆盖该用户，则原因为 `deny_scope`。跨用户操作获准时，依据的是明确授权，响应会报告 `inboxPolicyMode: "grant_override"`；此路径不应用目标用户自己的策略。

两种情况下，执行操作的智能体都必须是公司的活动成员，目标用户也必须是活动成员。

---

## 后续阅读

- [待处理事项](./attention.md)：了解这些策略所保护的收件箱界面。
- [智能体](./agents.md)：了解你可以在 `allowedAgentIds` 中指定的智能体。
- [资源成员关系](./resource-memberships.md)：了解公司成员关系，以及 `users:manage_permissions` 和 `inbox:manage` 背后的权限授权。
- [API 概览](./overview.md)：查看基础 URL、身份验证、公司范围和通用错误代码表。
