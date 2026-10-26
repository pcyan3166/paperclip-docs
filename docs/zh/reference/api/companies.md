---
paperclip_version: v2026.831.1
seo_title: 公司 API
seo_description: 公司是每个智能体、项目、任务和成本事件所属的租户边界。了解公司管理端点及其范围限制方式。
---

# 公司

公司是 Paperclip 顶层的租户边界。每个智能体、项目、任务、审批、成本事件和资源都且只属于一个公司；API 会在每个限定公司范围的路由上强制执行此边界。

看板调用方可以列出和管理公司。智能体调用方可以读取所属公司；CEO 智能体还可以更新所属公司的品牌信息。删除公司具有破坏性，归档则不会。

---

## 公司对象结构

公司 API 会返回精简的公司对象，包含最常用的字段：

| 字段 | 含义 |
|---|---|
| `id` | 稳定的公司 ID。 |
| `name` | 显示名称。 |
| `description` | 可选的纯文本说明。 |
| `status` | `active`、`paused` 或 `archived`。 |
| `issuePrefix` | 根据公司名称自动生成的任务前缀。 |
| `issueCounter` | 公司的下一个任务编号。 |
| `budgetMonthlyCents` | 公司预算上限，单位为美分。 |
| `spentMonthlyCents` | 本月已支出的美分数。 |
| `requireBoardApprovalForNewAgents` | 招聘是否需要看板审批。 |
| `logoAssetId` | 已存储的 logo 资源 ID；没有时为 `null`。 |
| `logoUrl` | 已存储 logo 的派生内容 URL；没有时为 `null`。 |
| `createdAt` | 创建时间戳。 |
| `updatedAt` | 上次更新时间戳。 |

如果设置了反馈共享同意字段，响应也会包含这些字段。`logoUrl` 根据 `logoAssetId` 派生，不应直接设置。

---

## 列出公司

```
GET /api/companies
```

返回当前看板调用方有权访问的公司。一般情况下，这是你的公司列表，而不是实例中的所有公司。

> **注意：** 这是看板路由。智能体调用方不应使用它。

## 获取公司

```
GET /api/companies/{companyId}
```

如果调用方有权访问该公司，则返回一个公司对象。

---

## 列出公司的资源

```
GET /api/companies/{companyId}/artifacts
```

返回公司范围内智能体在工作中生成内容的统一视图。它会将三个来源（智能体撰写的任务文档、直接附加到任务的附件，以及 `artifact` 工作成果）合并为适合卡片展示的列表，因此调用方无需拼接各任务端点的结果。生成视图前会先检查公司访问权限。

### 查询参数

| 参数 | 值 | 说明 |
|---|---|---|
| `kind` | `image`、`video`、`text`、`document`、`file`、`all` | 按媒体类型过滤，默认为 `all`。 |
| `projectId` | UUID | 仅返回某个项目中任务产生的资源。 |
| `q` | 字符串（最多 160 个字符） | 在资源标题/摘要以及来源任务的标识符/标题中进行全文搜索。 |
| `groupBy` | `none`、`task`、`parent_task` | 分组方式。`none`（默认值）返回平铺列表；`task` 按来源任务分组；`parent_task` 将子任务的资源归入根任务。 |
| `groupIssueId` | UUID | 使用分组模式时，展开一个分组（资源栈），只显示该组中的资源。 |
| `starred` | `true`、`false` | 只返回当前用户加过星标的文档。默认为 `false`。 |
| `limit` | 1–100 的整数 | 每页大小，默认为 `30`。 |
| `cursor` | 字符串 | 上一次响应中的 `nextCursor` 所返回的不透明分页游标。 |

### 仅返回你加星标的文档

如果只需要个人收藏，而非所有内容，请设置 `starred=true`。这会只返回你亲自加过星标的文档，并按最近加星标的时间从新到旧排列。注意：

- 只会返回文档。工作成果和附件会被排除，因为只有文档支持星标。因此，`starred=true` 与非 `all`、非 `document` 的 `kind` 值组合时不会返回任何结果。
- 常规文档过滤规则会放宽。即使文档并非由智能体创建或编辑，或者属于未过滤列表会隐藏的系统任务文档，已加星标的文档仍会包含在结果中。
- 星标按用户区分，因此只有已登录的看板调用方可以使用。智能体调用方请求 `starred=true` 时会得到空列表，不会看到他人的星标。
- `cursor` 分页仍然有效，并根据你加星标的时间分页。

如需添加或取消星标，请使用[资源成员关系](./resource-memberships.md)页面上的文档路由。

### 响应

响应使用信封对象。在未分组的情况下，返回平铺的 `artifacts` 列表；当 `groupBy` 为 `task` 或 `parent_task` 时，改为返回 `groups`（如果提供 `groupIssueId`，则返回展开组的 `artifacts` 和 `selectedGroup`）。没有更多页面时，`nextCursor` 为 `null`。

```json
{
  "artifacts": [
    {
      "id": "work_product:...",
      "source": "work_product",
      "mediaKind": "video",
      "title": "Launch teaser render",
      "previewText": null,
      "contentType": "video/mp4",
      "contentPath": "/api/attachments/.../content",
      "openPath": "/api/attachments/.../content",
      "downloadPath": "/api/attachments/.../content?download=1",
      "issue": { "id": "...", "identifier": "PAP-123", "title": "Produce launch teaser" },
      "project": { "id": "...", "name": "Launch" },
      "createdByAgent": { "id": "...", "name": "Video Agent" },
      "updatedAt": "2026-06-01T12:00:00.000Z",
      "href": "/PAP/issues/PAP-123#work-product-..."
    }
  ],
  "nextCursor": null
}
```

每个资源包含以下字段：

| 字段 | 含义 |
|---|---|
| `id` | 稳定的资源 ID，以来源前缀开头（`document:`、`attachment:` 或 `work_product:`）。 |
| `source` | `document`、`attachment` 或 `work_product`。 |
| `mediaKind` | `image`、`video`、`text`、`document`、`file` 或 `empty`。 |
| `title` | 显示标题。 |
| `previewText` | 简短纯文本预览；没有时为 `null`。 |
| `contentType` | 底层内容的 MIME 类型；没有时为 `null`。 |
| `contentPath` | 原始内容路径；没有时为 `null`。 |
| `openPath` | 用于内联打开的路径；没有时为 `null`。 |
| `downloadPath` | 下载路径；没有时为 `null`。 |
| `issue` | 来源任务的 `{ id, identifier, title }`。 |
| `project` | 任务所属项目的 `{ id, name }`；没有项目时为 `null`。 |
| `createdByAgent` | 产出该资源的 `{ id, name }`；没有时为 `null`。 |
| `updatedAt` | 上次更新时间戳。 |
| `href` | 指向来源任务中该资源的深层链接。 |

分组对象（在 `groups` / `selectedGroup` 中返回）包含 `id`、`groupBy`、`issue`、`title`、`count`、`mediaKinds`、`previewArtifacts`、`updatedAt` 和 `href`。

---

## 创建公司

```
POST /api/companies
{
  "name": "Horizon Labs",
  "description": "An autonomous research and marketing company",
  "budgetMonthlyCents": 50000
}
```

创建新公司，并自动分配任务前缀。如果 `budgetMonthlyCents` 大于 `0`，Paperclip 还会以 UTC 自然月模式创建相应的月度预算策略。

此路由供看板调用方使用。在本地可信模式中，隐式看板会话即可满足此要求。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/companies" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Horizon Labs",
    "description": "An autonomous research and marketing company",
    "budgetMonthlyCents": 50000
  }'
```

<!-- tab: JavaScript -->

```js
const res = await fetch("http://localhost:3100/api/companies", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "Horizon Labs",
    description: "An autonomous research and marketing company",
    budgetMonthlyCents: 50000,
  }),
});
const company = await res.json();
```

<!-- tab: Python -->

```py
import requests

res = requests.post(
    "http://localhost:3100/api/companies",
    headers={"Authorization": f"Bearer {token}"},
    json={
        "name": "Horizon Labs",
        "description": "An autonomous research and marketing company",
        "budgetMonthlyCents": 50000,
    },
)
company = res.json()
```

<!-- /tabs -->

---

## 更新公司

```
PATCH /api/companies/{companyId}
{
  "name": "Horizon Labs",
  "description": "Updated description",
  "budgetMonthlyCents": 75000,
  "status": "active",
  "logoAssetId": "11111111-1111-4111-8111-111111111111"
}
```

看板调用方可以在此更新完整公司记录。CEO 智能体也可以调用此路由，但只能更新品牌字段。如果只想更改品牌，使用下方专用的 `/branding` 路由更清晰。

品牌字段的重要注意事项：

- `logoAssetId` 必须指向属于同一公司的资源。
- 替换或清除 logo 时，之前的 logo 资源记录会被移除。
- logo 关联完成后，服务器会派生出 `logoUrl`。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X PATCH "http://localhost:3100/api/companies/company-1" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Horizon Labs",
    "description": "Updated description",
    "budgetMonthlyCents": 75000,
    "status": "active"
  }'
```

<!-- tab: JavaScript -->

```js
const res = await fetch("http://localhost:3100/api/companies/company-1", {
  method: "PATCH",
  headers: {
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "Horizon Labs",
    description: "Updated description",
    budgetMonthlyCents: 75000,
    status: "active",
  }),
});
const company = await res.json();
```

<!-- tab: Python -->

```py
import requests

res = requests.patch(
    "http://localhost:3100/api/companies/company-1",
    headers={"Authorization": f"Bearer {token}"},
    json={
        "name": "Horizon Labs",
        "description": "Updated description",
        "budgetMonthlyCents": 75000,
        "status": "active",
    },
)
company = res.json()
```

<!-- /tabs -->

---

## 更新品牌信息

```
PATCH /api/companies/{companyId}/branding
{
  "name": "Horizon Labs",
  "logoAssetId": "11111111-1111-4111-8111-111111111111"
}
```

如果你只需要更新品牌信息，请使用这个范围更窄的端点。它接受以下字段：

- `name`
- `description`
- `logoAssetId`

看板调用方可以使用此路由。CEO 智能体只能为所属公司调用。

如果传入的 `logoAssetId` 属于其他公司，API 会返回 `422`。

---

## 上传公司 logo

```
POST /api/companies/{companyId}/logo
Content-Type: multipart/form-data
```

此路由上传 logo 图片本身并返回资源记录，**不会**自动将 logo 关联到公司。上传成功后，将返回的 `assetId` 作为 `logoAssetId` 发送给 `PATCH /api/companies/{companyId}` 或 `PATCH /api/companies/{companyId}/branding`。

接受的图片类型：

- `image/png`
- `image/jpeg`
- `image/jpg`
- `image/webp`
- `image/gif`
- `image/svg+xml`

Logo 上传使用常规附件大小限制。SVG 文件会在存储前进行净化：脚本和外部链接会被剥除，空 SVG 或不安全 SVG 会被拒绝。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/companies/company-1/logo" \
  -H "Authorization: Bearer <token>" \
  -F "file=@./logo.svg"
```

<!-- tab: JavaScript -->

```js
const formData = new FormData();
formData.append("file", fileInput.files[0]);

const res = await fetch("http://localhost:3100/api/companies/company-1/logo", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${token}`,
  },
  body: formData,
});
const result = await res.json();
```

<!-- tab: Python -->

```py
import requests

with open("logo.svg", "rb") as f:
    res = requests.post(
        "http://localhost:3100/api/companies/company-1/logo",
        headers={"Authorization": f"Bearer {token}"},
        files={"file": ("logo.svg", f, "image/svg+xml")},
    )

result = res.json()
```

<!-- /tabs -->

上传响应包含 `assetId` 和 `contentPath`。`contentPath` 指向 `/api/assets/{assetId}/content`。

---

## 公司统计

```
GET /api/companies/stats
```

返回以公司 ID 为键的简要汇总：

```json
{
  "company-1": { "agentCount": 5, "issueCount": 42 }
}
```

此路由仅限看板调用方使用。

---

## 反馈追踪记录

```
GET /api/companies/{companyId}/feedback-traces
```

仅限看板调用方使用，用于查看公司范围内的反馈活动。查询过滤条件包括：

- `targetType`
- `vote`
- `status`
- `issueId`
- `projectId`
- `from`
- `to`
- `sharedOnly`
- `includePayload`

需要检查反馈循环时使用此端点；查看普通公司活动时不应使用它。

---

<a id="export-and-import"></a>
## 导出和导入

Paperclip 提供通用和公司范围内的数据迁移路由：

| 路由 | 用途 | 调用权限 |
|---|---|---|
| `POST /api/companies/{companyId}/export` | 导出公司数据包。 | 任何有权访问该公司的调用方。 |
| `POST /api/companies/{companyId}/exports/preview` | 预览公司导出。 | 该公司的看板用户或 CEO。 |
| `POST /api/companies/{companyId}/exports` | 通过限制更严格的数据迁移流程导出公司数据包。 | 该公司的看板用户或 CEO。 |
| `POST /api/companies/import/preview` | 预览导入到新公司或现有公司的操作。 | 仅限看板用户。 |
| `POST /api/companies/{companyId}/imports/preview` | 预览导入到同一公司的操作。 | 该公司的看板用户或 CEO。 |
| `POST /api/companies/{companyId}/imports/apply` | 执行导入到同一公司的操作。 | 该公司的看板用户或 CEO。 |

公司范围的 `imports/*` 路由有意设计得更安全：

- 只能将数据导入路由指定的公司；
- 不接受 `replace` 冲突策略。

如果要检查可能创建新公司或导入到其他公司的数据，请使用看板级预览路由。

### 分块导入传输

大型导出数据不必一次性上传。数据包较大时，客户端会先将其切分成多个分块，再通过**传输任务**逐块上传。服务器会持续记录此传输任务，因此上传中断后可从已有分块处恢复，而无需重新开始。UI 和 `paperclipai company import` 都会自动使用此流程；面向用户的操作说明见[导出与导入指南](../../guides/power/export-import.md)。

| 路由 | 用途 | 调用权限 |
|---|---|---|
| `POST /api/companies/import/transfers` | 启动或恢复分块导入传输，并获取其 `transferId`。 | 仅限看板用户。 |
| `PUT /api/companies/import/transfers/{transferId}/parts/{partIndex}` | 上传数据包的一个分块。 | 仅限看板用户。 |
| `GET /api/companies/import/transfers/{transferId}` | 查看传输进度以及仍缺少的分块。 | 仅限看板用户。 |
| `POST /api/companies/import/transfers/{transferId}/preview` | 执行导入前预览重组后的数据包。 | 仅限看板用户。 |
| `POST /api/companies/import/transfers/{transferId}/apply` | 重组分块、验证整份文件的哈希并执行导入。 | 仅限看板用户。 |

声明传输具有幂等性：清单中包含根据内容生成的键，因此重复声明同一个数据包会返回相同的运行，而不是启动新运行。传输完成后，该数据包就已处理；再次声明会直接结束。如果要再次导入，需重新导出公司数据。

---

## 归档或删除公司

```
POST /api/companies/{companyId}/archive
DELETE /api/companies/{companyId}
```

归档会将公司的状态改为 `archived`，并使其不再出现在默认列表中。该操作还会级联暂停公司内所有活动智能体（暂停原因为 `company_archived`），并取消它们排队中或正在执行的唤醒请求，因此归档后公司不会继续在后台工作。删除会移除公司及其相关数据。删除具有破坏性，应视为不可逆操作。

---

## 实用注意事项

- 读取或修改公司数据前始终会检查公司访问权限。
- `logoUrl` 根据已存储的 logo 资源派生，应视为只读字段。
- 如果只更新品牌信息，优先使用 `PATCH /api/companies/{companyId}/branding`。
- 上传 logo 和更新公司信息是两个独立步骤，操作时请牢记这一点。
