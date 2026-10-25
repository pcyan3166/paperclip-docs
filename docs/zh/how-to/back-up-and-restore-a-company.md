---
paperclip_version: v2026.1005.0
seo_title: 备份与恢复公司
seo_description: 导出可移植的公司包，并保留数据库和智能体文件备份以实现完整恢复。在使用前预览并验证每次恢复。
---

# 备份与恢复公司

您可以保存可移植的公司包并通过导出和导入进行恢复——这些文件既可以直接交给同事，也可以纳入版本控制。本指南将引导您走通 API 路径：预览 → 导出 → 导入 → 验证，并包含 CEO 作用域路由所强制的安全护栏。为了实现完整的实例恢复，还需保留数据库和文件系统备份，包括下文所述的智能体文件。

同一对操作也可将公司从一个实例迁移到另一个实例：在旧主机上导出，在新主机上导入。由于主机间 Cloud Sync 已退役，这现在是受支持的迁移路径。

CLI 快捷方式（`paperclipai company export | import`）适用于一次性工作——参见 [导出与导入](../guides/power/export-import.md)。下方的 HTTP 路由是您将其集成到例程、CI 作业或任何需要持久备份的智能体中的方式。

---

## 包的内容

一个包（bundle）是一组 Markdown 文件树，外加一个 `.paperclip.yaml` 伴随文件和一个内容寻址的 Blob 存储：

```text
my-company/
├── README.md
├── COMPANY.md
├── agents/
│   └── ceo/AGENTS.md
├── projects/
│   └── main/PROJECT.md
├── skills/
│   └── review/SKILL.md
├── tasks/
│   └── 2026-04-27-onboarding/
│       ├── TASK.md
│       └── documents/brief.md
├── images/
├── blobs/
└── .paperclip.yaml
```

`include` 标志决定了哪些切片会被包含在内：

| 标志 | 默认值 | 覆盖范围 |
|---|---|---|
| `company` | `true` | `COMPANY.md` + 品牌、Logo、预算、招聘政策 |
| `agents` | `true` | `agents/<slug>/AGENTS.md` + 适配器类型、运行时配置、权限、预算、环境变量声明 |
| `projects` | `false` | `projects/<slug>/PROJECT.md` + 工作区配置 |
| `skills` | `false` | `skills/<key>/SKILL.md`（引用或内置） |
| `issues` | `false` | `tasks/<slug>/TASK.md` 和例程，包含评论、标签、阻塞项、文档、工作产物、监控器和附件（请谨慎使用——包会变得很大） |

除了包含标志外，您还可以使用 `agents`、`skills`、`projects`、`issues` 和 `projectIssues` 数组按 ID 进行范围限定。使用 `projectIssues` 可以拉取特定项目内的所有议题，而无需逐一命名。

附件字节永远不会内联在 Markdown 文件中。它们存储在 `blobs/` 中，以其内容的 SHA-256 命名，每个任务的清单条目指向其所需的 Blob。这使得包在导入时具有自验证能力。

**绝不在包中。** 密钥值、API 密钥、机器路径、数据库 ID。任何与环境相关的特定内容。包仅声明智能体所需的环境变量；值保留在源机器上。

**故意遗留。** 审批、成本事件和活动日志条目。这些描述的是源实例上发生了什么，而非公司的配置方式，因此它们不会随包迁移。下一节将展示如何在导出前查明您的公司是否包含这些内容。

---

## 在实例备份中保留智能体文件

如果您希望智能体的笔记、记忆和其他个人文件在实例恢复后依然存在，请在备份数据库的同时备份持久化的实例文件系统。托管的智能体文件是磁盘上的当前字节，而不是新的数据库修订行。仅靠数据库转储无法恢复这些字节。

每个托管文件夹位于 `<paperclipInstanceRoot>/companies/<companyId>/agents/<agentId>/instructions/`。请将此目录作为实例存储备份的一部分保留，并在让智能体运行之前将其与匹配的数据库一起恢复。如果智能体使用外部指令文件夹，也请将配置的该文件夹包含在您自己的文件系统备份中。

运行时的临时副本不是备份：Paperclip 会在会话停止时将其移除。没有新的文件修订历史可供回滚，如果两个写入者修改了同一路径，最后完成的同步将胜出。有关保存规则和限制，请参阅 [智能体文件跨任务持久化](../guides/org/agents.md#agent-files-persist-across-tasks)。

实现参考：[托管存储路径](https://github.com/paperclipai/paperclip/blob/467125fafb47a8520856504fecc48d6e32055db1/server/src/services/agent-instructions.ts) 和 [运行副本生命周期](https://github.com/paperclipai/paperclip/blob/467125fafb47a8520856504fecc48d6e32055db1/server/src/services/agent-directory-working-copies.ts)。

---

## 0. 检查包不会携带的内容

询问保真度路由哪些内容将被遗留：

```bash
curl "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/export/fidelity" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```

响应会统计包处理的每个类别中的记录数，并返回一个 `warnings` 数组——每个有行但无法导出的类别对应一个条目：

```json
{
  "schema": "paperclip-export-fidelity-v1",
  "companyId": "…",
  "counts": { "issueDocuments": 14, "issueAttachments": 6, "approvals": 3, "costEvents": 812, "activityLogEntries": 4210, "…": 0 },
  "warnings": [
    { "code": "approvals_not_exported",       "severity": "warning", "message": "3 approvals are not included in the export bundle." },
    { "code": "cost_history_not_exported",    "severity": "warning", "message": "812 cost events are not included in the export bundle." },
    { "code": "activity_history_not_exported","severity": "warning", "message": "4210 activity log entries are not included in the export bundle." }
  ],
  "generatedAt": "2026-08-01T02:00:00.000Z"
}
```

警告是信息性的——它们不会阻止导出。请将它们与包一起记录，以便未来的恢复知道具体缺少了什么。UI 会将相同的列表显示为 **未包含在此导出中** 面板。

> **谁可以调用。** 与导出路由相同的规则：路由公司的 CEO 智能体，或具有公司访问权限的董事会调用者。

---

## 1. 预览导出

在保留包之前始终进行预览。预览返回文件清单、清单和任何警告——而不持久化任何内容。

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/exports/preview" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "include": { "company": true, "agents": true, "projects": true, "issues": false }
  }'
```

响应（`CompanyPortabilityExportPreviewResult`）结构：

```json
{
  "rootPath": "my-company",
  "counts": { "files": 12, "agents": 3, "skills": 2, "projects": 1, "issues": 0 },
  "fileInventory": [
    { "path": "COMPANY.md",             "kind": "company" },
    { "path": "README.md",              "kind": "readme" },
    { "path": "agents/ceo/AGENTS.md",   "kind": "agent" },
    { "path": "projects/main/PROJECT.md", "kind": "project" }
  ],
  "manifest": { "schemaVersion": 6, "...": "..." },
  "files": { "COMPANY.md": "name: ...\n" },
  "warnings": []
}
```

路径是相对于 `rootPath` 的，而不是以其为前缀——`rootPath` 仅是包解包到的文件夹名称。

`fileInventory` 是您在保留任何内容之前浏览的清单。如果某个路径看起来不对——例如您打算排除的项目，或您已终止的智能体——请调整请求并重新预览。

> **谁可以调用。** 路由公司的 CEO 智能体，或具有公司访问权限的董事会调用者。来自不同公司的智能体 JWT 将被拒绝并返回 `403 Agent key cannot access another company`；路由公司内部非 CEO 智能体将被拒绝并返回 `403 Only CEO agents can manage company exports`。

---

## 2. 构建导出（带 `selectedFiles`）

一旦清单看起来正确，将相同的主体发布到构建路由：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/exports" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "include": { "company": true, "agents": true, "projects": true },
    "agents": ["ceo", "cto"],
    "expandReferencedSkills": true
  }'
```

为了进一步缩小范围——例如，从覆盖所有内容的备份中丢弃某个嘈杂项目的 `PROJECT.md`——请传递一个显式的 `selectedFiles` 路径数组，这些路径来自预览的 `fileInventory`：

```json
{
  "include": { "company": true, "agents": true, "projects": true },
  "selectedFiles": [
    "COMPANY.md",
    "agents/ceo/AGENTS.md",
    "agents/cto/AGENTS.md",
    "projects/main/PROJECT.md"
  ]
}
```

未列出的任何内容都将从生成的 `files` 对象中丢弃。清单仍然描述整个公司；`selectedFiles` 仅过滤文件负载。

响应是一个 `CompanyPortabilityExportResult`，其中包含一个以路径为键的 `files` 映射。按照您的备份目标所需的方式持久化它——将每个条目写入磁盘，将 JSON 发送到 S3，或提交到私有 Git 仓库。包是文本，因此差异比较和审计成本很低。

---

## 3. 恢复到新公司

要从头重建公司（真正的灾难恢复），请使用 **董事会** 导入路由。它们接受任何 `target.mode`，包括 `new_company`：

```bash
# Preview the restore plan
curl -X POST "$PAPERCLIP_API_URL/api/companies/import/preview" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d @- <<'JSON'
{
  "source": {
    "type": "inline",
    "rootPath": "my-company",
    "files": { "...": "..." },
    "expectedFileCount": 12
  },
  "target": { "mode": "new_company", "newCompanyName": "Horizon Labs (restored)" },
  "include": { "company": true, "agents": true, "projects": true },
  "collisionStrategy": "rename"
}
JSON
```

`expectedFileCount` 是可选的，但值得发送：它是您对放入主体的文件数量的断言，如果到达的文件更少，服务器将拒绝导入。参见下方的 [两个完整性保护](#the-two-integrity-guards)。

预览返回一个 `CompanyPortabilityPreviewResult`，包含智能体、项目和议题计划（`create` / `update` / `skip`）以及任何必需的环境输入。请仔细阅读——这是应用步骤将执行内容的契约。

当计划看起来正确时，应用它：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/import" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  --data-binary @import-request.json
```

响应包括新的 `company.id` 以及每个实体的 `created` / `updated` / `skipped` 操作。**导入的智能体始终以定时器心跳关闭状态启动**——设置预算，填写环境变量，然后在准备好时重新开启心跳。

如果您也希望导入的智能体和例程以暂停状态落地，请在请求主体中添加 `"pauseAutomations": true`。这与心跳默认值是独立的开关：心跳关闭阻止它们按定时器唤醒，`pauseAutomations` 阻止它们运行，直到您取消暂停每一个。UI 默认勾选此项；CLI 默认关闭。

> **为什么这仅限实例管理员。** 创建新公司是实例级操作：它分配租户、议题前缀和预算策略。此路由要求具有实例管理员权限的董事会访问权限。智能体无法执行此操作，即使是 CEO 也不行。下一节中的 CEO 安全路由仅处理导入到路由公司的情况。

### 恢复大型包

繁忙公司的完整保真包会变得很大，而单个内联 JSON 主体是传输它的脆弱方式。针对这种情况存在两种机制，两者都值得集成到您预期在压力下运行的任何恢复流程中。

**上传 zip 而不是内联 JSON。** `POST /api/companies/import`（及其 `/preview` 兄弟路由）也直接接受压缩包——作为 `multipart/form-data`，其中归档在 `package` 文件字段中，其余导入请求作为 JSON `meta` 字段，或者作为带有 `meta` 在查询字符串中的裸 `application/zip` 主体。服务器将其解压缩为与内联源携带的完全相同的 `{ rootPath, files }` 包，因此无论哪种方式，导入语义都是相同的。压缩上传的大小大约只有三分之一，并且能经受住会截断膨胀 JSON 的代理。

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/import?async=1" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -F "package=@my-company.zip" \
  -F 'meta={"target":{"mode":"new_company","newCompanyName":"Horizon Labs (restored)"},"collisionStrategy":"rename"}'
```

**作为后台作业运行。** 添加 `?async=1`，路由会立即回答 `202`，返回一个作业 ID 和一个 `statusUrl`，而不是保持连接打开：

```json
{ "job": { "id": "…", "status": "running" }, "statusUrl": "/api/companies/import/jobs/…", "retryAfterMs": 1000 }
```

轮询该 URL，直到 `job.status` 为 `succeeded` 或 `failed`；成功的董事会作业在 `job.importResult` 下携带完整的导入结果。作业仅对创建它的身份可读，并驻留在内存中——服务器重启、未知 ID 或他人的 ID 都会返回相同的 `404`。每个账户一次只运行一个导入：在作业仍在运行时重新提交 *相同* 的请求会返回 `409`，指向已在途中的作业以便您接管它，而 *不同* 的导入会返回 `409` 且不附带作业。

### 两个完整性保护

恢复失败时是封闭的，而不是半写入公司。

1. **截断。** 内联源可以声明 `expectedFileCount`——客户端认为它发送的文件数量。如果到达的文件更少，导入将被拒绝并返回 `import_payload_incomplete` 以及预期/接收计数，因此被截断的上传永远无法导入片段。比声明更多的文件不是截断症状，是被允许的；省略该字段可保持旧调用者正常工作。
2. **篡改或损坏。** 在写入任何一行之前，每个 `blobs/<sha256>` 条目都会被哈希并与其所归档的名称进行比较。不匹配会中止整个导入并指出有问题的文件。

---

## 4. 导入到同一公司

对于向 **同一** 公司的非破坏性合并——重新导入您自己的备份，应用来自版本化包的刷新——请使用 CEO 安全路由：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/imports/preview" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "source": { "type": "inline", "rootPath": "my-company", "files": { "...": "..." } },
    "target": { "mode": "existing_company", "companyId": "'"$COMPANY_ID"'" },
    "include": { "agents": true },
    "collisionStrategy": "rename"
  }'
```

这些路由在入口处强制两条规则：

1. `target.companyId` 必须等于路由公司。任何其他 ID 都会返回 `403 forbidden: Safe import route can only target the route company`。
2. `collisionStrategy: "replace"` 会被拒绝并返回 `403 forbidden: Safe import route does not allow replace collision strategy`。

*确实* 有效的冲突策略：

| 策略 | 名称冲突时发生什么 |
|---|---|
| `rename`（默认） | 追加后缀——`ceo` 变为 `ceo-2`。始终安全。 |
| `skip` | 保留现有实体不动；对冲突的导入不做任何处理。 |

将预览主体发送到应用路由以应用计划：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/imports/apply" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  --data-binary @import-request.json
```

---

## 5. 为什么 `replace` 在安全路由上被拒绝

`replace` 会用包的内容覆盖现有的智能体、项目和技能。如果使用不当，它会静默销毁自备份以来已被编辑的智能体的生产版本——包括适配器配置、指令等。

CEO 安全路由禁止它，因为 CEO 智能体可以无人值守地触发导入（例程、Webhook、评论后的虚假 `apply`）。非破坏性默认值可防止自主恢复覆盖正在进行的实时工作。

如果您确实需要 `replace` 语义——例如，您正在强制将生产环境回滚到已知良好的包——请通过 `POST /api/companies/import` 处的董事会路由并使用董事会令牌。该路径是显式的、经过审计的，并由人类会话门控。

---

## 6. 每日导出例程

安排每日导出，以便在需要时存在新鲜的包。创建一个例程，唤醒一个备份所有者智能体——一个专门用于运行导出并将包写入备份存储位置（对象存储、Git 仓库、cron 挂载卷）的小型 CEO 角色智能体。

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Nightly company export",
    "description": "Run a full export and ship it to backup storage.",
    "assigneeAgentId": "<backup-owner-agent-id>",
    "projectId": "<ops-project-id>",
    "concurrencyPolicy": "skip_if_active",
    "catchUpPolicy": "skip_missed"
  }'
```

然后附加一个计划触发器：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/routines/<routine-id>/triggers" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "schedule",
    "label": "Daily 02:00 UTC",
    "enabled": true,
    "cronExpression": "0 2 * * *",
    "timezone": "UTC"
  }'
```

智能体的心跳处理器在每次运行时执行三件事：`POST /exports/preview` 进行清单，`POST /exports` 进行构建，然后将生成的 `files` 负载上传到您的备份目标。在智能体的指令中固定保留策略——例如“保留 7 天每日，4 周每周，12 个月每月”。专门的例程食谱将作为 HT6 推出；在此之前，[心跳与例程](../guides/projects-workflow/routines.md) 涵盖了端到端的例程模型。

---

## 7. 往返验证

在第一次恢复后，进行健全性检查：

1. **计数。** 将源公司的智能体和项目计数与恢复结果进行比较。董事会路由的应用响应包括每个实体的操作——`created` 应与预览承诺的一致。
2. **适配器配置。** 打开每个恢复的智能体，确认适配器类型和运行时配置看起来正确。环境变量值不会存在（这是设计如此）；在启用心跳之前填写它们。
3. **首次心跳。** 选择一个恢复的智能体，设置一个微小的预算，启用心跳，并分配给它一个琐碎的任务。如果它唤醒、检查并评论，则恢复是健康的。

今天能干净往返的包，六个月后也能干净往返。从未被任何人恢复过的包，只是名义上的备份。

---

## 8. 迁移到另一个实例

将公司移动到不同的主机——从笔记本到服务器，从自托管到 Paperclip Cloud，或反之——是相同的两个步骤，只是第二步指向别处：

1. 在源实例上导出（第 1 和第 2 节），使用 `include` 覆盖您想要保留的所有内容：`company`、`agents`、`projects`、`skills` 和 `issues`。
2. 在目标实例上使用董事会令牌和 `target.mode: "new_company"` 导入（第 3 节），如果包很大，使用 zip 上传和 `?async=1`。

然后完成恢复清单：环境变量值、预算、适配器，最后心跳。没有任何东西将两个实例相互连接——包是唯一迁移的东西，这使得通过 Git 仓库、对象存储或 U 盘移动它是安全的。

这取代了已退役的主机间 Cloud Sync。如果您的脚本指向 `cloud connect` 或 `cloud push`，请将其移植到 `POST /api/companies/{id}/export` 加上 `POST /api/companies/import`，或到 `paperclipai company export` 和 `paperclipai company import`。

---

## 另请参阅

- [导出与导入](../guides/power/export-import.md) — CLI 演练和包格式。
- [心跳与例程](../guides/projects-workflow/routines.md) — 每日食谱的计划、并发和追赶策略。
- [例程 API](../../reference/api/routines.md) — 创建和管理例程的每个端点。
- [公司 API](../../reference/api/companies.md#export-and-import) — 包括董事会级路径的完整路由表。
- [部署到 VPS 或 Fly.io](./deploy-to-vps-or-fly.md) — 与 Postgres 备份和主机外包存储配合使用。
