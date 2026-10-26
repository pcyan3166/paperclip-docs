---
paperclip_version: v2026.1012.0
seo_title: 公司技能策略 API
seo_description: 限制智能体对技能的操作，包括创建、导入、安装、编辑和移除，而不是向所有智能体开放整个技能库。
---

# 公司技能策略

智能体通过技能获得新能力，但如果没有防护措施，智能体可以随意创建、导入、安装、编辑或移除技能。**公司技能策略**提供防护规则。这是一份按公司设置的文档，用于指定哪些智能体可以执行哪些技能操作、针对哪些技能以及从哪些来源执行。

每家公司有一份策略文档，并带有修订编号以支持安全的并发编辑。你可以读取策略、整体替换、重置为开放默认值；也可以在更改之前让服务器解释某项决策。

下方所有路由都限定在公司范围内，每次请求都会检查公司访问权限。如果你刚开始使用 API，请先阅读 [API 概览](./overview.md)，了解基础 URL、身份验证和错误代码约定；本页不再重复说明。

---

## 开放默认策略

新建公司时，最初没有任何已存储的策略。此时读取策略会返回**开放默认值**：

```json
{
  "schemaVersion": 1,
  "revision": 0,
  "defaultEffect": "allow",
  "rules": [],
  "materialized": false
}
```

需要特别关注 `materialized`。值为 `false` 时表示不存在策略记录：尚未写入策略，因此允许所有技能操作。保存策略后，此值会变成 `true`，`revision` 从 `1` 开始递增。

此区别也会体现在评估结果中。未实体化的策略会直接得到“允许”，原因是 `no_policy_default`，不会逐条检查规则。

---

## 策略文档的结构

策略由三部分构成：schema 版本、默认效果和有序规则集合。

| 字段 | 说明 |
|---|---|
| `schemaVersion` | 必须是 `1`。 |
| `defaultEffect` | `allow` 或 `deny`。当没有规则匹配时采用此值。 |
| `rules` | 最多 1,000 项的规则对象数组。规则 `id` 必须唯一。 |

文档采用严格校验，未知字段会被拒绝。

### 规则

| 字段 | 必填 | 说明 |
|---|---|---|
| `id` | 是 | 长度为 1–128 个字符，且匹配 `^[a-zA-Z0-9][a-zA-Z0-9._:-]*$`。 |
| `priority` | 是 | `-1000000` 到 `1000000` 之间的整数。数值较小的规则优先检查。 |
| `effect` | 是 | `allow` 或 `deny`。 |
| `subject` | 是 | 规则适用于谁，见下文。 |
| `actions` | 是 | 至少包含一个技能操作，且不能重复。 |
| `resources` | 否 | 将规则限制到指定技能或来源。省略时匹配所有资源。 |

### 主体

`subject` 对象是一个由 `type` 标记的联合类型：

| `type` | 结构 | 匹配对象 |
|---|---|---|
| `all_agents` | `{ "type": "all_agents" }` | 任何智能体主体。 |
| `agents` | `{ "type": "agents", "agentIds": [...] }` | 列出的智能体。必须包含 1–500 个唯一 UUID。 |
| `roles` | `{ "type": "roles", "roles": [...] }` | 角色匹配的任何主体；比较前会修剪空白并忽略大小写。必须包含 1–500 个唯一字符串。 |

`all_agents` 和 `agents` 只会匹配智能体主体。`roles` 会根据主体的角色字符串匹配，因此角色为 `board` 的看板主体也可以由 `roles` 规则指定。

### 操作

| 操作 | 覆盖范围 |
|---|---|
| `skills.create` | 创建新技能。 |
| `skills.import` | 导入技能。 |
| `skills.install` | 安装技能。 |
| `skills.edit` | 编辑已有技能。 |
| `skills.update` | 从来源更新技能。 |
| `skills.test` | 运行技能测试。 |
| `skills.reset` | 重置技能。 |
| `skills.remove` | 移除技能。 |

**智能体在任务中途创建的技能按 `skills.create` 检查。** 智能体在任务中保存新技能时（参见[智能体在任务期间创建的技能](../../guides/org/skills.md#skills-your-agents-create-during-a-task)），请求会像 Skill Studio 创建操作一样进行评估：操作为 `skills.create`，来源类型为 `generated`。如果要阻止智能体以此方式创建技能，请对相应智能体拒绝 `skills.create`。单独设置 `sourceTypes: ["generated"]` 不能只筛选这种创建方式，因为在 UI 中从头创建技能也使用同一来源类型，这样会同时阻止两者。

**智能体在任务中途更新的技能按 `skills.edit` 检查。** 原生 Runner 的 **Update skill** 工具会通过与 Skill Studio 相同的文件路由保存，因此检查方式与 Studio 编辑该技能完全相同。每次尝试（包括重试已成功的请求）都会检查权限；撤销 `skills.edit` 会立即生效。参见[允许智能体改进现有技能](../../guides/org/skills.md#letting-an-agent-improve-an-existing-skill)。

**GitHub 技能来源的类型是 `git`。** 在 **Skills → Sources** 中扫描、预览和添加仓库时会检查 `skills.import`；刷新来源时检查 `skills.update`；更改所选技能或断开来源时检查 `skills.edit`。定位符为仓库 URL，因此 `sourceLocators` 规则可以针对单个仓库允许或拒绝操作。参见[技能参考 → GitHub 技能来源](../skills.md#github-skill-sources)。

### 资源选择器

`resources` 用于收紧规则范围。必须至少提供以下键中的一个，未知键会被拒绝：

| 键 | 说明 |
|---|---|
| `skillIds` | 1–500 个唯一技能 UUID。 |
| `skillKeys` | 1–500 个唯一字符串，每个长度为 1–512 个字符。 |
| `sourceTypes` | 一个或多个 `workspace`、`catalog`、`git`、`external_package`、`generated`、`unknown`。值必须唯一。 |
| `sourceLocators` | 1–500 个唯一定位符字符串，每项最长 2,048 个字符。 |

所列的每个选择器都必须匹配，规则才会生效。规则如果指定了 `skillKeys`，而评估请求没有 `skillKey`，就永远不会匹配。

**来源定位符会进行规范化。** 规则中保存的值与评估时提供的值会使用相同规则转换为规范形式：仓库形式的 `https` URL 会将主机名转为小写（`www.github.com` 变成 `github.com`），将所有者和仓库名转为小写，并去除 `.git` 后缀。这很重要：如果双方规范化方式不同，拒绝规则可能永远无法生效。

定位符也会接受内嵌凭据检查。URL 中包含用户名或密码，或者查询参数或片段中出现类似 `token`、`secret`、`password`、`api_key` 或 `authorization` 的参数时，会被拒绝，并显示 *"Source locators must not contain credentials or secret query or fragment parameters"*。

---

## 谁可以读取和更改策略

读取策略以及对自己执行策略评估，只要求有权访问公司。

更改策略（使用 `PUT`、`DELETE`，或代表其他智能体进行评估）需要具有**技能策略管理权限**，符合以下任一条件即可：

- 调用方是来自隐式本地会话的看板用户，或是实例管理员；
- 调用方是在该公司持有 `users:manage_permissions` 权限的看板用户；
- 调用方是在该公司持有 `users:manage_permissions` 权限的智能体。

其他调用方会收到 `403 Forbidden`、错误码 `skill_policy_admin_required` 和补救提示 *"Ask a company administrator to manage the skill policy."*。

本页所有路由还受两项约束：

- 未经身份验证的调用方会收到 `401`，错误码为 `skill_authentication_required`。
- 智能体密钥限定的公司与目标公司不同时，会收到 `403`，错误码为 `skill_company_boundary_denied`。

---

## 获取策略

```
GET /api/companies/{companyId}/skill-policy
```

返回公司的有效策略：已存储的文档（`materialized: true`，并包含当前 `revision`），或上方介绍的开放默认值。

---

## 替换策略

```
PUT /api/companies/{companyId}/skill-policy
```

策略采用整体替换，而不是局部修改。请发送完整文档，以及你认为当前有效的修订版本：

| 字段 | 必填 | 说明 |
|---|---|---|
| `schemaVersion` | 是 | `1`。 |
| `defaultEffect` | 是 | `allow` 或 `deny`。 |
| `rules` | 是 | 完整规则集；没有规则时发送 `[]`。 |
| `expectedRevision` | 是 | 非负整数。策略尚不存在时使用 `0`。 |

成功后，服务器会将修订号加一，并返回设置为 `materialized: true` 的已保存文档。

如果 `expectedRevision` 与存储值不匹配，写入会被拒绝，返回 `409 Conflict`、错误码 `skill_policy_revision_conflict`；详情中包含 `expectedRevision` 和 `currentRevision`，因此你可以重新读取、重新应用更改后重试。

文档验证失败时会返回 `422 Unprocessable Entity`、正文 `{ "error": "Invalid skill policy document" }`、错误码 `skill_policy_validation_failed`，以及说明问题的 `issues` 数组。

成功替换后会记录 `company.skill_policy_replaced` 活动，实体类型为 `company_skill_policy`，并包含旧修订号、新修订号、默认效果和规则数量。

### 示例

```bash
curl -sS -X PUT \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/companies/{companyId}/skill-policy" \
  -d '{
    "schemaVersion": 1,
    "defaultEffect": "allow",
    "expectedRevision": 0,
    "rules": [
      {
        "id": "no-external-installs",
        "priority": 10,
        "effect": "deny",
        "subject": { "type": "all_agents" },
        "actions": ["skills.install", "skills.import"],
        "resources": { "sourceTypes": ["external_package", "git"] }
      }
    ]
  }'
```

---

## 重置策略

```
DELETE /api/companies/{companyId}/skill-policy
```

删除已存储策略并恢复开放默认值。响应是开放默认文档（`defaultEffect: "allow"`、无规则、`materialized: false`）。

重置已有策略的公司时，会记录 `company.skill_policy_reset` 活动，其中包含旧修订号和新的修订号 `0`。从未拥有策略的公司执行重置属于空操作，但仍会返回默认值。

---

## 评估决策

```
POST /api/companies/{companyId}/skill-policy/evaluate
```

向策略引擎询问它会如何决定，但不实际执行操作。如果你希望解释拒绝原因，或在依赖规则集之前测试规则，请使用此端点。

请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `action` | 是 | 八种技能操作之一。 |
| `resource` | 否 | `skillId`、`skillKey`、`sourceType`、`sourceLocator`，可任选其子集。默认为 `{}`。 |
| `principal` | 否 | `{ "agentId": "<uuid>" }`。指定以此智能体身份评估，而非以你自己的身份评估。 |

省略 `principal` 时，会以**你自己的身份**运行评估：智能体调用方以自身智能体（及其角色）进行评估；看板调用方则以角色为 `board` 的看板主体进行评估。

提供 `principal` 时表示你正在询问其他用户的情况，因此此路由需要与写入操作相同的技能策略管理权限。指定的智能体必须存在（否则返回 `404 Agent not found`），并且必须属于当前公司（否则返回 `403`，错误码 `skill_company_boundary_denied`）。

### 决策过程

1. 如果策略尚未实体化，则允许操作，原因是 `no_policy_default`。
2. 否则，规则按 `priority` 升序排列；优先级相同时按 `id` 的字典序排列。首条操作、主体和资源选择器都匹配的规则胜出。决定遵循其 `effect`，原因是 `explicit_rule`，`matchedRuleId` 为该规则 ID。
3. 如果没有匹配规则且 `defaultEffect` 为 `deny`，会执行一项兼容性检查：如果主体在公司中持有 `skills:create` 或 `skills:suggest-changes` 权限授权，则允许该操作，原因是 `legacy_compatibility`。历史上这些授权允许整个技能修改范围；此处仅在默认拒绝时保留该范围。显式规则始终优先。
4. 其他情况下，决定遵循 `defaultEffect`，原因是 `policy_default`。

### 决策对象

```json
{
  "allowed": false,
  "action": "skills.install",
  "reason": "explicit_rule",
  "policyRevision": 4,
  "matchedRuleId": "no-external-installs",
  "remediation": "Contact a company administrator to change the skill policy."
}
```

允许操作时，`remediation` 为 `null`。

| `reason` | 含义 |
|---|---|
| `no_policy_default` | 未存储策略，因此允许该操作。 |
| `explicit_rule` | 有规则匹配；`matchedRuleId` 指明该规则。 |
| `policy_default` | 没有规则匹配；由 `defaultEffect` 决定结果。 |
| `legacy_compatibility` | 默认拒绝，但旧版 `skills:create` / `skills:suggest-changes` 授权允许该操作。 |
| `platform_invariant` | 技能路由在策略之前强制执行的平台级规则。 |

---

## 后续阅读

- [智能体](./agents.md)：了解在 `subject.agentIds` 和 `principal.agentId` 中指定的智能体。
- [活动记录](./activity.md)：查看 `company.skill_policy_replaced` 和 `company.skill_policy_reset` 记录。
- [API 概览](./overview.md)：查看基础 URL、身份验证、公司范围和通用错误代码表。
