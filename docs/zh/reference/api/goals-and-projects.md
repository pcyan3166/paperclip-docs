---
paperclip_version: v2026.1023.0
seo_title: 目标与项目 API
seo_description: 目标构成更高层的意图树，项目将具体交付物归集在其下。本页介绍二者的端点及关联方式。
---

# 目标与项目

目标回答“我们为什么要做这件事？”，项目回答“我们围绕什么具体交付物组织工作？”。在 Paperclip 中，目标构成更高层的意图树；项目则围绕交付物归集工作、工作区和运行时行为。

---

## 快速一览

| 概念 | 用途 | 关键字段 | 说明 |
|---|---|---|---|
| 目标 | 公司使命、团队目标或代理/任务目标 | `title`、`description`、`level`、`status`、`parentId`、`ownerAgentId` | `level` 默认为 `task`；`status` 默认为 `planned`。 |
| 项目 | 关联目标和工作区的交付物 | `name`、`description`、`status`、`goalId` / `goalIds`、`leadAgentId`、`targetDate`、`env`、`executionWorkspacePolicy` | 推荐使用 `goalIds`；为兼容旧调用方，`goalId` 仍保持同步。 |
| 工作区 | 关联到项目的代码仓库或本地文件夹 | `name`、`sourceType`、`cwd`、`repoUrl`、`repoRef`、`defaultRef`、`visibility`、`runtimeConfig`、`isPrimary` | 第一个工作区会自动成为主工作区。 |

所有这些端点都限定在公司范围内。如果你无权访问某个公司，就不能列出或修改该公司的目标和项目。

---

## 目标

目标是规划层。需要表达策略、将目标拆分为子目标，或为特定代理指定负责人时，可以使用目标。

目标层级包括：

- `company`
- `team`
- `agent`
- `task`

目标状态包括：

- `planned`
- `active`
- `achieved`
- `cancelled`

### 字段

- `title` 为必填字段。
- `description` 为可选字段。
- 省略 `level` 时默认为 `task`。
- 省略 `status` 时默认为 `planned`。
- `parentId` 将目标关联到另一个目标。
- `ownerAgentId` 将目标关联到某个代理。

### 路由

### 列出目标

```
GET /api/companies/{companyId}/goals
```

返回该公司的所有目标。

### 获取目标

```
GET /api/goals/{goalId}
```

按 ID 返回单个目标。找到目标后，API 会检查公司的访问权限。

### 创建目标

需要定义新目标或子目标时使用此端点。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/companies/{companyId}/goals" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "title": "Launch onboarding revamp",
    "description": "Reduce time-to-first-success for new users",
    "level": "company",
    "status": "active"
  }'
```

<!-- tab: JavaScript -->

```js
await fetch(`http://localhost:3100/api/companies/${companyId}/goals`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify({
    title: "Launch onboarding revamp",
    description: "Reduce time-to-first-success for new users",
    level: "company",
    status: "active",
  }),
});
```

<!-- tab: Python -->

```python
import requests

requests.post(
    f"http://localhost:3100/api/companies/{company_id}/goals",
    headers={"Authorization": f"Bearer {token}"},
    json={
        "title": "Launch onboarding revamp",
        "description": "Reduce time-to-first-success for new users",
        "level": "company",
        "status": "active",
    },
)
```

<!-- /tabs -->

### 更新目标

可使用此端点推进目标、变更负责人或调整其父级关系。

```
PATCH /api/goals/{goalId}
{
  "status": "achieved",
  "description": "Updated description",
  "ownerAgentId": "{agentId}"
}
```

此端点执行部分更新。省略的字段保持不变。

### 删除目标

```
DELETE /api/goals/{goalId}
```

删除目标并返回已删除的记录。

---

## 项目

在日常执行中，项目位于目标之下。需要整理交付物、关联工作区并将相关 issue 归入一个实用容器时，请创建项目。

项目有以下几个重要实现细节：

- 推荐使用 `goalIds` 将项目关联到一个或多个目标。
- 为兼容性，仍接受 `goalId`；服务会将它与已关联的第一个目标保持同步。
- 项目名称会经过规范化，确保类似短名称的引用在公司范围内唯一。
- 省略 `color` 时，Paperclip 会从项目配色板自动分配颜色。
- 返回的项目包含 `urlKey`、`goalIds`、`goals`、`codebase`、`workspaces`、`primaryWorkspace` 和 `executionWorkspacePolicy`。

### 字段

- `name` 为必填字段。
- `description` 为可选字段。
- `visibility` 可设为 `open`（默认）或 `private`。私有项目只对其访问成员可见，参阅[项目访问成员](#项目访问成员)。只有项目隐私所有者、公司所有者或公司管理员可以修改此项；将项目设为私有的人会自动成为成员。
- `status` 默认为 `backlog`。
- `leadAgentId` 为可选字段。
- `targetDate` 为可选字段。
- 新代码优先使用 `goalIds`。
- 创建项目时可以同时提供 `workspace`，在同一请求中创建首个工作区。
- `env` 是项目环境绑定配置；秘密绑定在存储前会经过规范化。
- `executionWorkspacePolicy` 是高级运行时策略对象。
- 归档项目时可设置 `archivedAt`。

项目状态包括：

- `backlog`
- `planned`
- `in_progress`
- `completed`
- `cancelled`

### 路由

### 列出项目

```
GET /api/companies/{companyId}/projects
```

通常只需要查看仍在进行的工作，因此此路由只返回公司的活动项目，不包含已归档项目。项目设置了 `archivedAt` 后即视为已归档；可通过[更新项目](#更新项目)设置该字段。

审计、“显示全部”视图或迁移脚本等情况需要已归档项目时，请使用 `includeArchived` 请求。

#### 查询参数

| 参数 | 说明 |
|---|---|
| `includeArchived` | 设为 `true` 可在列表中包含已归档项目。省略或传入其他值时只返回活动项目。 |

```
GET /api/companies/{companyId}/projects?includeArchived=true
```

只有完全等于字符串 `true` 才会启用，因此 `includeArchived=1` 和 `includeArchived=TRUE` 均不会包含已归档项目。

**行为变更：** 此路由过去默认会返回已归档项目。如果集成依赖此行为，请添加 `includeArchived=true` 以保持原有结果；如果你一直只需要活动项目，也可自行按 `archivedAt` 筛选。

### 获取项目

```
GET /api/projects/{projectId}
```

`:projectId` 可以是 UUID，也可以是唯一项目短名称。短名称存在歧义时，API 会返回 `409 Conflict`，并要求改用 UUID。

### 创建项目

如果提供 `workspace`，项目和工作区会在同一个请求中创建。工作区请求体无效时，项目创建会回滚，API 返回 `422`。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/companies/{companyId}/projects" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "name": "Auth System",
    "description": "End-to-end authentication and session flow",
    "goalIds": ["{goalId}"],
    "status": "planned",
    "workspace": {
      "name": "auth-repo",
      "cwd": "/path/to/workspace",
      "repoUrl": "https://github.com/org/repo",
      "repoRef": "main",
      "isPrimary": true
    }
  }'
```

<!-- tab: JavaScript -->

```js
await fetch(`http://localhost:3100/api/companies/${companyId}/projects`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify({
    name: "Auth System",
    description: "End-to-end authentication and session flow",
    goalIds: [goalId],
    status: "planned",
    workspace: {
      name: "auth-repo",
      cwd: "/path/to/workspace",
      repoUrl: "https://github.com/org/repo",
      repoRef: "main",
      isPrimary: true,
    },
  }),
});
```

<!-- tab: Python -->

```python
import requests

requests.post(
    f"http://localhost:3100/api/companies/{company_id}/projects",
    headers={"Authorization": f"Bearer {token}"},
    json={
        "name": "Auth System",
        "description": "End-to-end authentication and session flow",
        "goalIds": [goal_id],
        "status": "planned",
        "workspace": {
            "name": "auth-repo",
            "cwd": "/path/to/workspace",
            "repoUrl": "https://github.com/org/repo",
            "repoRef": "main",
            "isPrimary": True,
        },
    },
)
```

<!-- /tabs -->

### 更新项目

可使用此端点修改状态、目标、目标日期、环境绑定、归档状态或其他项目元数据。

```
PATCH /api/projects/{projectId}
{
  "status": "in_progress",
  "goalIds": ["{goalId}"],
  "archivedAt": null
}
```

此端点也接受旧字段 `goalId`，但建议使用 `goalIds`。

### 删除项目

```
DELETE /api/projects/{projectId}
```

删除项目并返回已删除的记录。

### 项目访问成员

私有项目只对访问列表中的人员和代理可见。使用以下路由查看和更改访问成员：

```
GET    /api/projects/{projectId}/access-members
POST   /api/projects/{projectId}/access-members
DELETE /api/projects/{projectId}/access-members/{memberId}
```

- `GET` 会向所有有权读取项目的人返回成员列表。每项包含 `subjectType`、`subjectId`，以及用于显示的 `subjectDisplayName` 和 `subjectAvatarUrl`。
- `POST` 会添加成员并以 `201 Created` 返回该成员。请求体指定要添加的对象：

  ```json
  { "subjectType": "user", "subjectId": "<user-id>" }
  ```

  `subjectType` 可为 `user` 或 `agent`。该主体必须属于同一公司（用户必须是有效成员），否则返回 `422`，错误为 `Project access member must belong to this company`。
- `DELETE` 按 `memberId` 移除单个成员并返回被移除的条目。不能移除项目隐私所有者（错误为 `422 The project privacy owner cannot be removed`）。

添加或移除成员需要看板权限，且只能由项目隐私所有者、公司所有者/管理员或实例管理员执行；其他调用者会收到 `403`，错误为 `Only the project privacy owner or an administrator can change project access`。成员变更会记录为 `project.access_member_added` 和 `project.access_member_removed`。

---

## 项目工作区

一个项目可以有一个或多个工作区。主工作区是项目范围工作默认使用的工作区。

代码支持以下工作区来源类型：

- `local_path`
- `git_repo`
- `remote_managed`
- `non_git_path`

校验规则：

- 工作区必须至少包含 `cwd` 或 `repoUrl` 之一。
- `remote_managed` 工作区必须包含 `remoteWorkspaceRef` 或 `repoUrl`。
- 项目中的第一个工作区会自动成为主工作区。
- 设置 `isPrimary: true` 会将该工作区设为主工作区。
- 移除主工作区时，Paperclip 会自动将另一个工作区提升为主工作区。

### 列出工作区

```
GET /api/projects/{projectId}/workspaces
```

返回项目的所有工作区。

### 创建工作区

项目创建后也可以添加工作区。

```
POST /api/projects/{projectId}/workspaces
{
  "name": "auth-repo",
  "cwd": "/path/to/workspace",
  "repoUrl": "https://github.com/org/repo",
  "repoRef": "main",
  "isPrimary": true
}
```

### 更新工作区

```
PATCH /api/projects/{projectId}/workspaces/{workspaceId}
{
  "name": "auth-repo-main",
  "isPrimary": true
}
```

更新工作区时，还可以修改 `cwd`、`repoUrl`、`repoRef`、`defaultRef`、`visibility`、`setupCommand`、`cleanupCommand`、`remoteProvider`、`remoteWorkspaceRef`、`sharedWorkspaceKey`、`metadata` 和 `runtimeConfig`。

### 删除工作区

```
DELETE /api/projects/{projectId}/workspaces/{workspaceId}
```

删除工作区并返回已删除的记录。

### 工作区运行时服务

此端点控制与特定项目工作区关联的运行时服务。

```
POST /api/projects/{projectId}/workspaces/{workspaceId}/runtime-services/{action}
```

`action` 必须为以下之一：

- `start`
- `stop`
- `restart`

代码中的重要行为：

- `start` 和 `restart` 要求工作区具有本地路径（`cwd`）。
- `start` 和 `restart` 还要求配置工作区运行时。
- 路由会将工作区的 `desiredState` 更新为 `running` 或 `stopped`。
- 响应包含更新后的工作区和操作记录。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -X POST "http://localhost:3100/api/projects/{projectId}/workspaces/{workspaceId}/runtime-services/start" \
  -H "Authorization: Bearer <token>"
```

<!-- tab: JavaScript -->

```js
await fetch(
  `http://localhost:3100/api/projects/${projectId}/workspaces/${workspaceId}/runtime-services/start`,
  {
    method: "POST",
    headers: { Authorization: `Bearer ${token}` },
  },
);
```

<!-- tab: Python -->

```python
import requests

requests.post(
    f"http://localhost:3100/api/projects/{project_id}/workspaces/{workspace_id}/runtime-services/start",
    headers={"Authorization": f"Bearer {token}"},
)
```

<!-- /tabs -->

---

## 实用模型

需要表达意图并建立层级时使用目标；需要关联实际工作（工作区、运行时行为和 issue 执行）时使用项目。实际操作中，大多数团队会先创建公司目标，再添加一个或多个支持性目标，最后创建指向代理将要执行工作的项目。

不确定应该创建哪一种时，可以这样判断：

- “这是关于我们为什么存在吗？”创建或更新目标。
- “这是一个有实际代码或文件的交付物吗？”创建或更新项目。
- “我需要一个仓库或目录来开展工作吗？”添加工作区。
