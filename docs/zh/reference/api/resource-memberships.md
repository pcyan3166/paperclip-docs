---
paperclip_version: v2026.529.0
seo_title: 资源成员关系 API
seo_description: 已登录的看板用户如何通过加入或离开项目和智能体来整理个人侧边栏，以及离开资源为何不等于失去访问权限。
---

# 资源成员关系

资源成员关系用于让已登录的看板用户整理个人侧边栏。每位用户可以通过**加入**或**离开**项目和智能体，决定哪些内容要保留在视图中。离开项目或智能体不会影响其他任何人，只会整理你的个人导航。默认情况下，所有资源都视为已加入，直到你主动离开。

文档的处理方式略有不同。你不能加入或离开文档，而是可以为文档**加星标**，方便之后快速找到。星标也是个人设置：为文档加星标不会影响同事。

这些路由始终针对**当前用户**（路径中的 `me` 部分）。你不能通过此 API 查看或更改其他人的成员关系。

> 所有路由都限定在公司范围内，并要求看板（用户）身份验证。智能体 token 会被 `403` 拒绝。

---

## 列出你的成员关系

```
GET /api/companies/{companyId}/resource-memberships/me
```

返回调用用户在该公司内明确加入或离开的所有项目和智能体的成员状态，以及用户已加星标的文档。

响应按资源类型分组。每条记录将资源 ID 映射到 `joined` 或 `left`。你从未操作过的资源不会出现；未列出的资源应视为已加入。

```json
{
  "projectMemberships": {
    "1f3c…": "joined",
    "8ad2…": "left"
  },
  "agentMemberships": {
    "b91e…": "left"
  },
  "starredDocumentIds": ["7c4a…", "2e08…"],
  "documentStarredAt": {
    "7c4a…": "2026-05-26T13:40:02.000Z",
    "2e08…": "2026-05-24T09:12:44.000Z"
  },
  "updatedAt": "2026-05-26T13:41:23.000Z"
}
```

以下两个字段描述文档星标：

| 字段 | 类型 | 说明 |
|---|---|---|
| `starredDocumentIds` | ID 数组 | 你加过星标的文档，按最近加星标的时间从新到旧排列。字段始终存在；没有加星标时为空数组。 |
| `documentStarredAt` | 对象 | 将每个已加星标的文档 ID 映射到加星标时间的 ISO-8601 时间戳字符串。字段始终存在。 |

项目和智能体对应的字段 `starredProjectIds`、`starredAgentIds`、`projectStarredAt` 和 `agentStarredAt` 在响应中是可选的，不要依赖它们一定存在。上面两个文档字段则不会省略。

`updatedAt` 表示你所有成员关系（包括文档星标）中最近一次变更的时间；如果你从未更改过，则为 `null`。

### 示例

```bash
curl -sS \
  -H "Authorization: Bearer {token}" \
  "https://paperclip.example.com/api/companies/{companyId}/resource-memberships/me"
```

---

## 加入或离开项目

```
PUT /api/companies/{companyId}/resource-memberships/me/projects/{projectId}
```

设置当前用户是否在侧边栏中保留某个项目。

请求正文：

| 字段 | 类型 | 说明 |
|---|---|---|
| `state` | 枚举，必填 | `joined` 表示在侧边栏保留该项目；`left` 表示将其隐藏。 |

项目必须属于该公司，否则服务器会返回 `404 Not Found`。如果设置的状态与当前状态相同，则不会发生变化；调用仍会成功并返回当前状态。

响应：

```json
{
  "resourceType": "project",
  "resourceId": "{projectId}",
  "state": "left",
  "updatedAt": "2026-05-26T13:41:23.000Z"
}
```

状态实际改变时，服务器会在活动日志中记录 `resource_membership.joined` 或 `resource_membership.left` 条目，以便审计。

### 示例

```bash
curl -sS -X PUT \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/companies/{companyId}/resource-memberships/me/projects/{projectId}" \
  -d '{ "state": "left" }'
```

---

## 加入或离开智能体

```
PUT /api/companies/{companyId}/resource-memberships/me/agents/{agentId}
```

此路由与上面的项目路由相对应，用于设置当前用户是否在侧边栏中保留某个智能体。

请求正文：

| 字段 | 类型 | 说明 |
|---|---|---|
| `state` | 枚举，必填 | `joined` 表示在侧边栏保留该智能体；`left` 表示将其隐藏。 |

智能体必须属于该公司，否则服务器会返回 `404 Not Found`。与项目一样，重复设置相同状态不会改变数据，但会返回当前状态；状态确实改变时，会写入 `resource_membership.joined` 或 `resource_membership.left` 活动条目。

响应：

```json
{
  "resourceType": "agent",
  "resourceId": "{agentId}",
  "state": "joined",
  "updatedAt": "2026-05-26T13:41:23.000Z"
}
```

### 示例

```bash
curl -sS -X PUT \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/companies/{companyId}/resource-memberships/me/agents/{agentId}" \
  -d '{ "state": "joined" }'
```

---

## 为文档加星标或取消星标

```
PUT /api/companies/{companyId}/resource-memberships/me/documents/{documentId}
```

为文档加星标，以便快速再次找到；也可以取消星标。文档不能加入或离开，只能设置星标。

请求正文：

| 字段 | 类型 | 说明 |
|---|---|---|
| `starred` | 布尔值，必填 | `true` 为文档加星标；`false` 则移除星标。 |

请求正文格式严格：只接受 `starred` 字段，发送其他字段会被拒绝。文档必须属于该公司，否则服务器会返回 `404 Not Found`。再次为已加星标的文档加星标，或取消从未加过星标的文档的星标，都不会改变数据，但会成功并返回当前状态。

响应：

```json
{
  "resourceType": "document",
  "resourceId": "{documentId}",
  "state": "joined",
  "starredAt": "2026-05-26T13:41:23.000Z",
  "updatedAt": "2026-05-26T13:41:23.000Z"
}
```

文档的 `state` 始终为 `joined`，因为文档没有可离开的状态。`starredAt` 表示为文档加星标的时间；取消星标后会变为 `null`。星标确实发生改变时，服务器会在活动日志中记录 `resource_membership.starred` 或 `resource_membership.unstarred` 条目，以便审计。

如需浏览已加星标的文档，请在公司 artifact 列表中设置 `starred=true`，详见[公司](./companies.md)。

### 示例

```bash
curl -sS -X PUT \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/companies/{companyId}/resource-memberships/me/documents/{documentId}" \
  -d '{ "starred": true }'
```

---

## 注意事项

- **仅限自助操作。** 用户只能读取和更新自己的成员关系。使用智能体身份验证的请求，或指定其他用户的请求，都会被 `403 Forbidden` 拒绝。
- **需要拥有活动的公司访问权限。** 在本地单用户实例之外，调用用户必须是公司的活动成员。
- **默认状态为已加入。** 没有存储记录的资源视为 `joined`。离开后重新加入会恢复默认的可见状态。
- **文档只能加星标。** 文档路由接受 `starred` 而不是 `state`，且始终报告 `state: "joined"`。取消星标只会移除你的个人星标，不会更改文档本身。
