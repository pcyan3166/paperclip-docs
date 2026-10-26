---
paperclip_version: v2026.609.0
seo_title: 团队目录 API
seo_description: 团队目录的接口，提供可直接使用的起始团队，将智能体、项目、例程和技能打包，可一步安装到公司中。
---

# 团队目录 API

团队目录 API 是目录中**团队**的控制平面接口。团队是 Paperclip 随应用提供的即用型起始小组。每个目录团队包含一小组智能体、项目、任务及其所需技能，因此可以一步将整个工作单元加入公司。通过这些端点，你可以浏览目录、查看单个团队及其文件、查看公司已安装的团队、预览导入计划并安装团队。

有关面向用户的概念和 CLI 工作流，请参见[团队命令](../cli/teams.md)和[团队目录指南](../../guides/org/team-catalog.md)。本页介绍这些工具调用的原始 HTTP 端点。

> 本页所有路由都挂载在 `/api` 下。浏览路由只要求调用方已通过身份验证，不需要公司上下文；`installed`、`preview` 和 `install` 路由限定在公司范围内，并会检查公司访问权限。

目录团队分为两类。**内置团队**随应用提供，适合日常作为起点。**可选团队**是额外提供的团队，其中一部分会从外部来源引入技能。使用 `kind` 过滤器可以在两者之间切换。

---

## 浏览目录

```
GET /api/teams/catalog
```

返回调用方可见的目录团队。浏览操作只读取目录，不会写入任何内容，也不需要公司上下文。

### 查询参数

| 参数 | 值 | 说明 |
|---|---|---|
| `kind` | `bundled`、`optional` | 按目录类型过滤。 |
| `category` | 字符串 | 按目录类别 slug 过滤。 |
| `q` | 字符串 | 在目录中进行自由文本搜索。 |

响应中的每个目录团队都包含 `id`、`key`、`kind`、`category`、`slug`、`name`、`description`、`path`、`entrypoint`、`schema`、`defaultInstall`、`recommendedForCompanyTypes`、`tags`、`counts` 对象（其中包含 `agents`、`projects`、`tasks`、`routines`、`localSkills`、`catalogSkills`、`externalSkillSources`）、`rootAgentSlugs`、`agentSlugs`、`projectSlugs`、`requiredSkills`、`envInputs`、`sourceRefs`、`files`、`trustLevel`、`compatibility` 和 `contentHash`。从软件包获取的可选团队还会包含 `packageName` 和 `packageVersion`。

---

## 查看单个目录团队

```
GET /api/teams/catalog/{catalogId}
```

返回单个目录团队的完整记录，结构与列表条目相同，包括其智能体、项目、任务、所需技能和文件清单。安装前查看团队，是了解它将被添加到公司中的确切内容的最直接方式。

路径段 `{catalogId}` 是目录引用，可以是目录团队 ID、key 或唯一 slug。也可以通过 `ref` 查询参数显式传入引用；如果存在，该参数优先于路径段。

### 查询参数

| 参数 | 说明 |
|---|---|
| `ref` | 目录引用（ID、key 或 slug）。存在时覆盖 `{catalogId}` 路径段。 |

---

## 读取目录团队文件

```
GET /api/teams/catalog/{catalogId}/files
```

返回目录团队软件包中的一个文件，例如 `TEAM.md`。可以在不安装团队的情况下读取其清单或引用的文件。

### 查询参数

| 参数 | 说明 |
|---|---|
| `ref` | 目录引用（ID、key 或 slug）。存在时覆盖 `{catalogId}` 路径段。 |
| `path` | 团队软件包中待读取文件的相对路径，默认为 `TEAM.md`。 |

响应包含 `catalogTeamId`、`path`、`kind`、`content`、`language` 和 `markdown`。

---

## 列出已安装的目录团队

```
GET /api/companies/{companyId}/teams/catalog/installed
```

返回指定公司中已经安装的目录团队。这是以公司为视角的列表：它会告知客户端安装了哪些团队，并允许客户端标记出与当前目录版本不一致的团队。开始列出团队前会检查公司访问权限。

---

## 预览安装

```
POST /api/companies/{companyId}/teams/catalog/{catalogId}/preview
```

服务器会执行导入流程并返回计划，但不会向公司写入任何内容。预览会报告即将创建的内容、团队需要哪些技能及其准备方式，以及所有警告或错误。开始预览前会检查公司访问权限。

所有正文参数都是可选的：

| 字段 | 类型 | 说明 |
|---|---|---|
| `targetManagerAgentId` | 字符串或 `null` | 已有智能体 ID，目录团队中的根智能体将向其汇报。 |
| `targetManagerSlug` | 字符串或 `null` | 可移植的经理 slug，目录团队中的根智能体将向其汇报。 |
| `include` | 对象 | 可移植性包含配置块，用于选择要导入的实体类型。 |
| `agents` | `"all"` 或 slug 数组 | 将导入范围限制为所选智能体 slug。 |
| `collisionStrategy` | `rename`、`skip`、`replace` | 指定如何处理已存在的实体。 |
| `nameOverrides` | 对象（`slug` → `name`） | 覆盖导入实体的名称。 |
| `selectedFiles` | 字符串数组 | 将导入范围限制为所选可移植文件。 |
| `sourcePolicy` | 对象 | 限制外部技能来源。接受 `allowExternalSources`、`allowUnpinnedOptionalSources` 和 `allowLocalPathSources` 布尔值。 |

`{catalogId}` 路径段是目录引用；如果存在 `ref` 查询参数，它将覆盖路径段，与浏览路由的处理方式相同。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/companies/company-1/teams/catalog/core-exec-team/preview" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "targetManagerSlug": "cto",
    "collisionStrategy": "rename"
  }'
```

<!-- tab: JavaScript -->

```js
const res = await fetch(
  "http://localhost:3100/api/companies/company-1/teams/catalog/core-exec-team/preview",
  {
    method: "POST",
    headers: {
      Authorization: `Bearer ${token}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      targetManagerSlug: "cto",
      collisionStrategy: "rename",
    }),
  },
);
const plan = await res.json();
```

<!-- tab: Python -->

```py
import requests

res = requests.post(
    "http://localhost:3100/api/companies/company-1/teams/catalog/core-exec-team/preview",
    headers={"Authorization": f"Bearer {token}"},
    json={
        "targetManagerSlug": "cto",
        "collisionStrategy": "rename",
    },
)
plan = res.json()
```

<!-- /tabs -->

---

## 安装团队

```
POST /api/companies/{companyId}/teams/catalog/{catalogId}/install
```

执行 `preview` 显示的操作：将团队导入公司、创建其中的智能体、项目和任务，并准备所需技能。成功时路由返回 `201`。由于安装团队会创建智能体，调用方必须有权在公司中创建智能体：看板调用方需要 `agents:create` 权限（本地隐式看板用户和实例管理员看板用户无需检查）；智能体调用方则需要 `agents:create` 授权或等效的智能体权限。

安装正文接受 `preview` 支持的全部字段，另有：

| 字段 | 类型 | 说明 |
|---|---|---|
| `adapterOverrides` | 对象（`slug` → 覆盖配置） | 为导入的智能体指定适配器。每个覆盖项是一个对象，包含 `adapterType`（字符串）和可选的 `adapterConfig` 对象。 |
| `secretValues` | 对象（`key` → `value`） | 为团队声明的密钥环境输入提供值。 |

与预览一样，`{catalogId}` 是目录引用，若存在 `ref` 查询参数，则以后者为准。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/companies/company-1/teams/catalog/product-engineering/install" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "targetManagerSlug": "cto",
    "adapterOverrides": {
      "senior-coder": { "adapterType": "claude_local" }
    }
  }'
```

<!-- tab: JavaScript -->

```js
const res = await fetch(
  "http://localhost:3100/api/companies/company-1/teams/catalog/product-engineering/install",
  {
    method: "POST",
    headers: {
      Authorization: `Bearer ${token}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      targetManagerSlug: "cto",
      adapterOverrides: {
        "senior-coder": { adapterType: "claude_local" },
      },
    }),
  },
);
const result = await res.json();
```

<!-- tab: Python -->

```py
import requests

res = requests.post(
    "http://localhost:3100/api/companies/company-1/teams/catalog/product-engineering/install",
    headers={"Authorization": f"Bearer {token}"},
    json={
        "targetManagerSlug": "cto",
        "adapterOverrides": {
            "senior-coder": {"adapterType": "claude_local"},
        },
    },
)
result = res.json()
```

<!-- /tabs -->

---

## 实用注意事项

- 目录团队中的智能体有意不预先固定适配器。使用 `adapterOverrides` 将指定智能体固定到 `claude_local` 或 `codex_local` 等运行时；未指定覆盖的智能体会在导入时解析运行时。
- 浏览、查看团队和读取文件只要求调用方已通过身份验证。`installed`、`preview` 和 `install` 路由会先检查公司访问权限。
- 预览不会写入内容。安装前先运行预览，确认将创建的内容以及团队技能的准备方式。
- 安装路由会强制检查 `agents:create` 权限。没有此权限时，请求会在创建任何内容前被拒绝。
