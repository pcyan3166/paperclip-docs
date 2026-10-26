---
paperclip_version: v2026.722.0
seo_title: CLI 公司命令
seo_description: 在终端中检查和管理顶层容器；每个代理、项目、目标、issue 和预算都属于一个公司。
---

# 公司命令

公司是 Paperclip 的顶层容器：每个代理、项目、目标、issue 和预算都位于某个公司内。需要检查公司、修改设置和品牌信息、将整个公司作为可移植软件包在实例间迁移、获取反馈轨迹，或在服务器策略允许时删除公司，可使用 `paperclipai company`。

看板操作员可以查看并管理实例中的所有公司。代理角色身份则严格限定在一个公司内；因此，公司范围的读取和删除安全检查都会针对该公司执行，除非你显式提供 ID。

> **提示：** 编写脚本时，可在任意命令后添加 `--json`。人类可读输出是摘要，JSON 输出则包含服务器的完整记录。

---

## 通用选项

此页所有命令都接受标准客户端选项：`--data-dir <path>`、`--api-base <url>`、`--api-key <token>`、`--context <path>`、`--profile <name>` 和 `--json`。API 基础 URL 和凭据的解析方式请参阅[通用选项](./common-options.md)。

有两个命令——`feedback:list` 和 `feedback:export`——通过必填标志 `-C, --company-id <id>` 指定公司，而非使用位置参数。`import` 命令也接受 `-C, --company-id <id>` 来指定现有目标公司。其他地方的公司 ID 都是位置参数 `<companyId>`。

---

## 读取命令

以下命令只读取数据，可安全地在任何位置运行。

```sh
paperclipai company list
paperclipai company get <company-id>
paperclipai company current
paperclipai company stats
```

| 命令 | 返回内容 |
|---|---|
| `company list` | 你有权查看的所有公司。人类可读输出显示 `id`、`name`、`status`、月预算和支出（单位为分），以及新代理是否需要看板审批。 |
| `company get <companyId>` | 单个公司的完整记录。 |
| `company current` | 当前作用域公司完整记录。根据 `-C, --company-id`、上下文文件、`PAPERCLIP_COMPANY_ID` 或代理凭据解析公司，不需要位置参数 ID。 |
| `company stats` | `/api/companies/stats` 提供的实例级公司统计信息。 |

使用代理凭据时，`company list` 只返回该代理所属公司；使用看板凭据时，则返回实例中的所有公司。

`company current` 可便捷地读取当前作用域内的公司：它和其他命令一样解析当前公司 ID，并返回与 `company get <companyId>` 相同的完整记录。代理凭据恰好绑定到一个公司，因此使用代理凭据时它始终返回该公司。

---

## 创建和更新

`create`、`update` 和 `branding:update` 会将原始 JSON 请求体转发给 API。CLI 不会校验请求体结构，只会将 `--payload-json` 传给服务器，由服务器按对应 schema 处理（`CreateCompany`、`UpdateCompany`、`UpdateCompanyBranding`）。请构造符合 API 要求的对象，并将其作为单个 JSON 字符串传入。

```sh
paperclipai company create \
  --payload-json '{"name":"Acme","issuePrefix":"ACME"}'

paperclipai company update <company-id> \
  --payload-json '{"budgetMonthlyCents":500000}'

paperclipai company branding:update <company-id> \
  --payload-json '{"primaryColor":"#1f6feb","logoUrl":"https://cdn.example.com/acme.png"}'
```

| 命令 | 方法/路由 | 说明 |
|---|---|---|
| `company create` | `POST /api/companies` | 必须提供 `--payload-json`。 |
| `company update <companyId>` | `PATCH /api/companies/{id}` | 必须提供 `--payload-json`；只发送你要更改的字段。 |
| `company branding:update <companyId>` | `PATCH /api/companies/{id}/branding` | 必须提供 `--payload-json`；只能包含品牌字段。 |

> **注意：** 请求体会原样转发，因此格式错误或未知字段会触发服务器端校验错误，而非 CLI 错误。写入被拒绝时，可使用 `--json` 查看响应。

---

## 归档

```sh
paperclipai company archive <company-id>
```

`company archive` 会向 `/api/companies/{id}/archive` 发送 POST 请求，不接受请求体。归档是将公司移出当前运营状态的可逆方式。只有确实需要永久删除记录，且服务器配置允许删除时，才使用 [`company delete`](#删除公司)。

---

## 导出与导入

Paperclip 公司可移植。`company export` 会将公司写入由 Markdown 文件、`.paperclip.yaml` 清单及二进制资源组成的目录；`company import` 则可从本地路径、`.zip` 文件或 GitHub 仓库读取该软件包，并导入到新公司或现有公司中。

导出包完整保留数据：除公司记录、代理、项目和技能外，还包含任务及其评论、标签、阻塞关系、文档、工作成果、监控器、例行任务及其计划触发器，以及按内容寻址的附件文件。审批、成本历史和活动日志仍留在源实例，因为它们描述源公司发生过什么，而非公司如何配置。

> **注意：** 这两条命令用于在实例之间迁移公司，取代了已停用的主机到主机 Cloud Sync，因此不再需要先将一个实例连接到另一个实例。操作指南参阅[导出与导入](../../guides/power/export-import.md)。

### 导出可移植软件包

```sh
paperclipai company export <company-id> \
  --out ./exports/acme \
  --include company,agents,projects,issues,skills
```

| 标志 | 用途 |
|---|---|
| `--out <path>` | 必填。写入软件包的输出目录。 |
| `--include <values>` | 以逗号分隔的内容集合：`company`、`agents`、`projects`、`issues`（别名 `tasks`）和 `skills`。默认 `company,agents`。 |
| `--skills <values>` | 要包含的技能 slug/key，逗号分隔。 |
| `--projects <values>` | 要包含的项目短名称/ID，逗号分隔。 |
| `--issues <values>` | 要包含的 issue 标识符/ID，逗号分隔。 |
| `--project-issues <values>` | 要导出其 issue 的项目短名称/ID，逗号分隔。 |
| `--expand-referenced-skills` | 将技能内容包含在软件包中，而不是仅导出上游引用。默认关闭。 |
| `--force` | 不经交互式确认就覆盖非空输出目录。默认关闭。 |

默认 `--include` 集合刻意设得较小（`company,agents`），以免快速导出时附带所有任务。需要完整克隆时请显式增加范围。如果输出目录已有文件，交互式终端会询问是否覆盖，非交互式终端则会直接拒绝；可传入 `--force` 强制覆盖，或在无头环境中选择空目录。夜间备份例行任务等自动化导出需要写入本来就有文件的目录时，适合使用 `--force`。

成功后，命令会打印解析后的输出路径、软件包根目录、已写入文件数、`.paperclip.yaml` 扩展路径和警告数。

### 导入软件包

```sh
paperclipai company import ./exports/acme \
  --target new \
  --new-company-name "Acme (imported)"

paperclipai company import https://github.com/acme/company-pack \
  --target existing --company-id <company-id> \
  --collision replace --yes
```

源参数可以是本地目录、本地 `.zip` 文件、GitHub URL 或 `owner/repo[/path]` 简写。通用 HTTP URL 会被拒绝；只支持 GitHub / GitHub Enterprise URL 和本地路径。

| 标志 | 用途 |
|---|---|
| `--include <values>` | 以逗号分隔的导入集合：`company,agents,projects,issues,tasks,skills`。省略时默认导入全部五类资源。 |
| `--target <mode>` | `new` 表示创建公司；`existing` 表示导入已有公司。省略时根据 `--company-id` / 上下文推断。 |
| `-C, --company-id <id>` | 现有目标公司的 ID。`--target existing` 时必填。 |
| `--new-company-name <name>` | `--target new` 时覆盖公司名称。 |
| `--agents <list>` | 要导入的代理 slug，逗号分隔；也可指定 `all`。默认为 `all`。 |
| `--collision <mode>` | 处理已有匹配项的方式：`rename`（默认）、`skip` 或 `replace`。 |
| `--ref <value>` | GitHub 源使用的 Git 引用（分支、标签或提交）。只适用于 GitHub 导入。 |
| `--paperclip-url <url>` | 此命令中 `--api-base` 的别名。 |
| `--yes` | 接受默认选择并跳过导入前确认提示。 |
| `--dry-run` | 仅运行预览，不应用任何更改。 |

导入始终先在服务器端执行预览，并在写入前显示计划，包括软件包内容、各资源组的创建/更新/跳过数量、实体示例及警告或错误。在交互式终端中，如果未指定 `--include` 或 `--yes`，还会显示选择菜单，让你挑选要导入的项目、任务、代理和技能。添加 `--dry-run` 后，命令会在预览完成后停止。

> **警告：** 应用导入需要明确授权。使用 `--json` 时必须传入 `--yes`（先运行 `--dry-run` 可通过 JSON 检查预览）。从非交互式终端运行时也必须提供 `--yes`。交互式终端中如未提供 `--yes`，CLI 会在应用导入前明确要求确认。

导入的代理如果使用 `process` 适配器但未明确指定适配器，CLI 当前会回退使用 `claude-local` 适配器，并在预览/结果信息块中告知你。

写入任何记录前，服务器都会核对软件包中每个附件与其内容哈希是否匹配。若不匹配，整个导入会中止并指出问题文件，因此已损坏或被篡改的软件包不会留下只导入一部分的公司。对于特别大的公司，CLI 的目录导入也是推荐方式；当上传文件太大、浏览器无法预览时，UI 也会提示使用此命令。

导入的代理无论软件包中如何设置，其定时心跳都会关闭，避免审查之前开始产生支出。

### 原始 API 转发命令

如果希望使用手工构造的请求体直接调用服务器可移植性路由，可使用四条转发命令将 `--payload-json` POST 到原始路由。这些命令不会读取或写入本地文件，是脚本调用 `export` / `import` 背后 API 端点的接口。

```sh
paperclipai company export:preview <company-id> --payload-json '{...}'
paperclipai company export:api <company-id> --payload-json '{...}'
paperclipai company import:preview <company-id> --payload-json '{...}'
paperclipai company import:apply <company-id> --payload-json '{...}'
```

| 命令 | 路由 |
|---|---|
| `company export:preview <companyId>` | `POST /api/companies/{id}/exports/preview` |
| `company export:api <companyId>` | `POST /api/companies/{id}/exports` |
| `company import:preview <companyId>` | `POST /api/companies/{id}/imports/preview` |
| `company import:apply <companyId>` | `POST /api/companies/{id}/imports/apply` |

日常工作建议使用高级 `export` / `import` 命令；它们会处理文件读写、源解析、预览和确认。只有需要精确控制请求体时，才使用 API 转发变体。

---

## 反馈轨迹

反馈轨迹记录针对代理工作提交的投票和评审。使用以下命令读取反馈或导出供分析，两者都要求提供 `-C, --company-id <id>`。

```sh
paperclipai company feedback:list --company-id <company-id> --status open --vote down
paperclipai company feedback:export --company-id <company-id> --from 2026-01-01 --format ndjson --out ./feedback.ndjson
```

| 标志 | 用途 |
|---|---|
| `--target-type <type>` | 按目标类型筛选。 |
| `--vote <vote>` | 按投票值筛选。 |
| `--status <status>` | 按轨迹状态筛选。 |
| `--project-id <id>` | 按项目筛选。 |
| `--issue-id <id>` | 按 issue 筛选。 |
| `--from <iso8601>` | 只包含在此时间戳或之后创建的轨迹。 |
| `--to <iso8601>` | 只包含在此时间戳或之前创建的轨迹。 |
| `--shared-only` | 只包含允许分享/导出的轨迹。 |
| `--include-payload` | 在响应中包含已存储的请求体快照。 |

`feedback:list` 会为每条轨迹打印紧凑行（`id`、issue、投票、状态、目标类型、目标标签），也可添加 `--json` 获取完整记录。`feedback:export` 还接受两个标志：

| 标志 | 用途 |
|---|---|
| `--out <path>` | 将导出内容写入文件，而不是 stdout。 |
| `--format <format>` | `json` 或 `ndjson`。默认为 `ndjson`。 |

`feedback:export` 默认包含请求体快照；`feedback:list` 则需要显式使用 `--include-payload`。指定 `--out` 时，命令会写入文件并报告数量；否则会将序列化的轨迹流式写入 stdout，方便传给其他工具。

---

## 删除公司

`company delete` 会永久删除公司及其记录，属于破坏性操作，特意设计为不容易意外触发。

```sh
paperclipai company delete PAP --yes --confirm PAP
paperclipai company delete 5cbe79ee-acb3-4597-896e-7662742593cd \
  --yes --confirm 5cbe79ee-acb3-4597-896e-7662742593cd
```

选择器参数可以是公司 ID（UUID），也可以是 issue 前缀/短名称（例如 `PAP`）。

| 标志 | 用途 |
|---|---|
| `--by <mode>` | 选择器解析方式：`auto`（默认）、`id` 或 `prefix`。 |
| `--yes` | 必填安全标志；不提供则删除失败。 |
| `--confirm <value>` | 必填。必须与目标公司的 ID 或 issue 前缀一致。 |

### 安全检查

按设计，删除必须通过以下四项检查：

1. **服务器策略。** 实例必须启用 `PAPERCLIP_ENABLE_COMPANY_DELETION`。若未启用，无论命令标志如何，API 都会拒绝删除。
2. **`--yes`。** 未提供时命令立即报错，不会调用 API。
3. **`--confirm <value>`。** 该值必须与解析出的公司 ID 或 issue 前缀一致（前缀不区分大小写）。不匹配时会报错并给出预期 ID 和前缀，便于纠正。
4. **无歧义解析。** 在 `auto` 模式下，如果选择器同时匹配一个公司的 ID 和另一公司的前缀，命令会以歧义错误拒绝执行；请改用 `--by id` 或 `--by prefix`。

解析顺序：如果选择器看起来像 UUID（或使用 `--by id`），CLI 首先按 ID 查找；接着查找上下文中的作用域公司，最后尝试看板范围的公司列表。代理凭据无法执行看板范围查找；删除限定在公司范围内，因此请使用当前公司的 ID 或前缀（通过 `--company-id` 或 `PAPERCLIP_COMPANY_ID`），不要指定其他公司。

> **警告：** 删除无法撤销。如果只是希望让公司暂时退出运营，请改用[`company archive`](#归档)。

成功后，命令会打印 `ok: true`，并显示已删除公司的 ID、名称和前缀。

---

## 另请参阅

- [通用选项](./common-options.md)——API 基础 URL 解析、凭据和所有命令共享的标志。
- [输出与脚本](./output-and-scripting.md)——使用 `--json` 和管道处理 CLI 输出。
- [项目命令](./project.md)——管理公司内的项目。
- [代理命令](./agent.md)——检查代理并设置本地代理 CLI 会话。
- [反馈命令](./feedback.md)——使用超出公司范围视图之外的反馈轨迹。
- [通过 CLI 创建首个公司](../../guides/getting-started/your-first-company.md)——端到端操作指南。
