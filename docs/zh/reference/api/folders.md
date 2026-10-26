---
paperclip_version: v2026.720.0
seo_title: 文件夹 API
seo_description: 为例程和技能提供限定公司范围的嵌套目录，避免平铺列表变得难以使用。可创建、重命名、移动文件夹并读取其内容。
---

# 文件夹

当公司拥有多个例程或技能时，平铺列表就不再实用。**文件夹**为它们提供了组织位置：在公司范围内形成嵌套目录，并可重命名、更改颜色、重新排序以及拖动项目。

文件夹分为两种类型，二者完全独立：`routine` 文件夹用于存放例程，`skill` 文件夹用于存放技能。每个请求都要指明操作哪个目录树，服务器会拒绝混用。

有些文件夹由 Paperclip 自行创建和维护，例如随产品提供的 `Bundled` 分类文件夹、每个看板用户专属的 `My Skills` 个人文件夹，以及每个项目对应的文件夹。这些文件夹标记为系统管理，并且对你只读。其他文件夹则可按你的需要自由整理。

文件夹限定在公司范围内；与控制平面 API 的其他部分一样，每次请求都会检查公司访问权限。如果你刚开始使用 API，请先阅读 [API 概览](./overview.md)，了解基础 URL、身份验证和错误代码约定；本页不再重复说明。

---

## 文件夹的结构

每个文件夹都包含以下字段：

| 字段 | 含义 |
|---|---|
| `id` | 文件夹 UUID。 |
| `companyId` | 文件夹所属公司。 |
| `kind` | `routine` 或 `skill`。文件夹类型创建后不能更改。 |
| `parentId` | 父文件夹 UUID；根文件夹为 `null`。 |
| `name` | 显示名称，长度为 1–120 个字符。 |
| `slug` | URL 安全标识符，在同级文件夹中唯一。 |
| `systemKey` | 系统管理文件夹会设置此字段，例如 `bundled`、`my`、`my:{userId}`、`project:{projectId}`、`bundled:{slug}`；你创建的文件夹中为 `null`。 |
| `path` | 从根目录到此文件夹的 slug 链，以 `/` 连接，例如 `my/ada-lovelace`。由服务器计算，不存储。 |
| `depth` | 根目录为 `1`，其子目录为 `2`，依此类推。 |
| `color` | 可选的颜色标签，长度为 1–80 个字符，或为 `null`。 |
| `position` | 同级文件夹之间的整数排序位置。 |
| `createdAt` / `updatedAt` | 时间戳。 |

### Slug

Slug 必须匹配 `^[a-z0-9]+(?:-[a-z0-9]+)*$`：由小写字母、数字和单个连字符组成，长度为 1–120 个字符。如果不提供，服务器会从文件夹名称生成 slug（移除变音符号、全部转为小写，并将连续的其他字符折叠成一个连字符）。

Slug 只需在**同级文件夹中**唯一，因此不同父文件夹下可以各有一个名为 `research` 的文件夹。

### 深度和嵌套

文件夹最多可嵌套 **4 层**。创建或移动文件夹时，如果会导致任意目录超过此限制，服务器会返回 `422 Unprocessable Entity` 和 `{ "error": "Folder depth cannot exceed 4" }`。移动时会计算整个子树的深度，而不只是被拖动的文件夹。

### 系统管理文件夹

包含 `systemKey` 的文件夹，以及 `bundled` 目录树中的所有文件夹，都由 Paperclip 管理。尝试重命名、移动或删除这些文件夹会返回 `403 Forbidden`：

```json
{ "error": "System-managed folders cannot be changed" }
```

任何会向 bundled 目录树写入内容的操作也会被拒绝：

```json
{ "error": "Bundled folders are read-only" }
```

`skill` 目录树中有三个根 slug 被保留：`bundled`、`my` 和 `projects`。你不能创建、重命名或移动根技能文件夹并使用这些 slug；服务器会以 `403 Forbidden` 返回 `{ "error": "Reserved skill folders are system-managed" }`。`my` 和 `projects` 容器也不允许手动创建子文件夹，因为 Paperclip 会自动填充它们。

---

## 列出文件夹

```
GET /api/companies/{companyId}/folders?kind={kind}
```

返回公司的完整文件夹目录树，展平为数组，并依次按 `position`、`name`、`id` 排序。

### 查询参数

| 参数 | 必填 | 说明 |
|---|---|---|
| `kind` | 是 | `routine` 或 `skill`。其他值（包括省略参数）会返回 `400 Bad Request` 和 `{ "error": "Folder kind query parameter is required" }`。 |

### 响应

```json
{
  "kind": "skill",
  "folders": [ ... ],
  "allCount": 42,
  "unfiledCount": 7
}
```

| 字段 | 含义 |
|---|---|
| `kind` | 你请求的类型。 |
| `folders` | 此类型的所有文件夹，每个对象都包含上文的全部字段及 `itemCount`（直接位于该文件夹中的项目数）。 |
| `allCount` | 公司中该类型项目的总数，包括已归档和未归档项目。 |
| `unfiledCount` | 没有文件夹的该类型项目数，也就是 UI 中“未归档”分组显示的数量。 |

计数只统计直接位于当前文件夹的项目，不累计子目录中的项目。父文件夹的 `itemCount` 不包含其子文件夹中的项目。

---

## 创建文件夹

```
POST /api/companies/{companyId}/folders
```

在任一目录树中创建文件夹。成功时返回 `201 Created` 和新文件夹。

请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `kind` | 是 | `routine` 或 `skill`。 |
| `name` | 是 | 修剪首尾空白后，长度为 1–120 个字符。 |
| `parentId` | 否 | 父文件夹 UUID；创建根文件夹时为 `null`。 |
| `slug` | 否 | 覆盖根据 `name` 生成的 slug，且必须符合 slug 格式。 |
| `color` | 否 | 长度为 1–80 个字符，或为 `null`。 |
| `position` | 否 | 大于或等于 `0` 的整数。默认设置为同级文件夹中最高 `position` 加一，因此新文件夹会排在末尾。 |

可能发生的错误：

- `parentId` 不存在或属于另一个目录树时，返回 `404 Not Found` 和 `{ "error": "Parent folder not found" }`。
- 同级文件夹已使用该 slug 时，返回 `409 Conflict` 和 `{ "error": "Folder slug already exists under this parent" }`。
- 超过深度限制时，返回 `422 Unprocessable Entity`。
- 根 slug 被保留或父文件夹属于 bundled 目录时，返回 `403 Forbidden`（见上文）。

### 示例

```bash
curl -sS -X POST \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  "https://paperclip.example.com/api/companies/{companyId}/folders" \
  -d '{
    "kind": "skill",
    "name": "Research",
    "color": "#4f46e5"
  }'
```

---

## 确保个人技能文件夹存在

```
POST /api/companies/{companyId}/folders/ensure-my
```

每位已登录的看板用户都可以在 `My Skills` 中拥有个人文件夹，用于存放属于个人而非公司的技能。此路由是幂等的：无论文件夹原本已存在，还是刚刚创建，调用后都会返回你的文件夹。响应状态为 `200 OK`。

请求正文可选，允许空正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `slug` | 否 | 个人文件夹的首选 slug，必须符合 slug 格式；允许为 `null`。如果与同级文件夹冲突，服务器会追加稳定后缀，而不会报错。默认根据你的显示名称生成。 |

文件夹使用 `systemKey: "my:{userId}"` 标识，因此你更改名称后它仍属于你。如果 `My Skills` 容器尚不存在，系统会自动创建。

这是本页唯一要求调用方为真实人类用户的路由。智能体 token 和其他非看板调用方会收到 `403 Forbidden`：

```json
{ "error": "A signed-in board user is required to create a personal skill folder" }
```

---

## 更新文件夹

```
PATCH /api/companies/{companyId}/folders/{folderId}
```

可原地重命名、更改颜色、重新设置 slug 或调整文件夹顺序。成功时返回更新后的文件夹；文件夹不存在时返回 `404 Not Found` 和 `{ "error": "Folder not found" }`。

请求正文至少发送一个字段：

| 字段 | 说明 |
|---|---|
| `name` | 长度为 1–120 个字符，首尾空白会被修剪。如果更改名称但未发送 `slug`，则会根据新名称重新生成 slug。 |
| `slug` | 必须符合 slug 格式。 |
| `color` | 长度为 1–80 个字符，或传入 `null` 清除颜色。 |
| `position` | 大于或等于 `0` 的整数。 |

空正文会被拒绝，返回 `{ "message": "At least one folder field is required" }`。此路由不会更改 `parentId`；如需更改父文件夹，请使用下方的移动路由。

---

## 移动文件夹

```
POST /api/companies/{companyId}/folders/{folderId}/move
```

更改文件夹的父级，并将其放到新同级文件夹中的指定位置。该文件夹的所有子项会随之移动。成功时返回更新后的文件夹，否则返回 `404 Not Found`。

请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `position` | 是 | 大于或等于 `0` 的整数，表示文件夹在新同级文件夹中的位置。 |
| `parentId` | 否 | 新父文件夹的 UUID；传入 `null` 可将其移动到根目录。完全省略此字段则保留当前父级，只调整位置。 |

保护规则：

- `parentId` 指向文件夹自身时，返回 `422 Unprocessable Entity` 和 `{ "error": "A folder cannot be its own parent" }`。
- `parentId` 指向该文件夹的子孙目录时，返回 `422` 和 `{ "error": "A folder cannot be moved into its own subtree" }`。
- 会根据子树最深分支检查 4 层限制，因此较深的子树可能无法放到深层父文件夹下。
- 目标位置发生 slug 冲突时，返回 `409 Conflict`。
- 不能移动系统管理文件夹和 bundled 文件夹。

---

## 将项目移入文件夹

```
POST /api/companies/{companyId}/folders/items/move
```

将单个例程或技能归档到文件夹，也可以将它移出文件夹恢复为未归档状态。看板中的拖放操作正是使用此端点。

请求正文：

| 字段 | 必填 | 说明 |
|---|---|---|
| `kind` | 是 | `routine` 或 `skill`，表示要移动的项目类型。 |
| `itemId` | 是 | 例程或技能的 UUID。 |
| `folderId` | 否 | 目标文件夹 UUID。传入 `null` 或省略字段可取消归档。 |

响应：

```json
{ "kind": "skill", "itemId": "...", "folderId": "..." }
```

可能发生的错误：

- `folderId` 不存在时，返回 `404 Not Found` 和 `{ "error": "Folder not found" }`。
- 文件夹的 `kind` 与项目类型不匹配时，返回 `422 Unprocessable Entity` 和 `{ "error": "Folder kind must match item kind" }`。
- 项目不存在时，返回 `404` 和 `{ "error": "Routine not found" }` 或 `{ "error": "Skill not found" }`。
- 目标文件夹属于 bundled 目录时，返回 `403 Forbidden` 和 `{ "error": "Bundled folders are read-only" }`；当前位于 bundled 目录中的技能也不能移出，返回 `{ "error": "Bundled skills cannot be moved" }`。

---

## 删除文件夹

```
DELETE /api/companies/{companyId}/folders/{folderId}
```

删除没有子文件夹的文件夹。成功时返回 `{ "deleted": { ...folder } }`，其中包含删除前的文件夹对象；文件夹不存在时返回 `404 Not Found`。

删除仍含子文件夹的文件夹会返回 `409 Conflict`：

```json
{ "error": "Move or delete nested folders first" }
```

删除文件夹不会删除其中的项目；已归档在其中的例程和技能只会恢复为未归档状态。不能删除系统管理文件夹和 bundled 文件夹。

---

## 活动日志

本页中的每次写入都会在公司活动日志中记录一条记录，使文件夹调整可供审计：

| 操作 | 记录时机 |
|---|---|
| `folder.created` | 创建文件夹时。 |
| `folder.personal_ensured` | 通过 `ensure-my` 确保个人技能文件夹存在时。 |
| `folder.updated` | 重命名、更改颜色、重新设置 slug 或调整位置时。 |
| `folder.moved` | 通过移动路由更改父级或位置时。 |
| `folder.item_moved` | 将例程或技能归档到文件夹，或移出文件夹时。 |
| `folder.deleted` | 删除文件夹时。 |

---

## 后续阅读

- [例程](./routines.md)：了解由 `kind: "routine"` 文件夹整理的定时工作。
- [活动记录](./activity.md)：查看上方 `folder.*` 活动记录出现的位置。
- [API 概览](./overview.md)：查看基础 URL、身份验证、公司范围和通用错误代码表。
