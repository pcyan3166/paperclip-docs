---
paperclip_version: v2026.1023.0
seo_title: 实例管理 API
seo_description: 无法归入其他 API 页面的实例级端点：常规和实验性设置、按需数据库备份，以及 LLM 反射端点。
---

# 实例管理 API

本页汇总其他 API 页面不便涵盖的实例级 REST 端点：常规和实验性实例设置、按需数据库备份、LLM 反射端点、环境（沙盒/运行时驱动），以及执行工作区生命周期。

这些接口足够稳定，可供你自己的工具调用，但范围有意收窄。大多数运维人员通过 UI、CLI 或 `paperclipai doctor` 使用它们，而不是手动调用。

> 所有路由都挂载在 `/api` 下。大多数需要实例管理员或看板身份验证；各路由的例外情况会单独说明。

---

## 实例设置

| 端点 | 用途 |
|---|---|
| `GET /api/instance/settings` | 读取完整实例设置记录（常规与实验性配置块，以及实例默认环境）。任何经过身份验证的组织成员均可读取。 |
| `PATCH /api/instance/settings` | 更新设置记录。请求正文根据 `patchInstanceSettingsSchema` 验证。设置 `defaultEnvironmentId` 可指定实例的默认执行环境。 |
| `GET /api/instance/settings/general` | 读取常规实例设置（显示名称、默认值等）。 |
| `PATCH /api/instance/settings/general` | 更新常规设置。请求正文根据 `patchInstanceGeneralSettingsSchema` 验证。 |
| `GET /api/instance/settings/experimental` | 读取实验功能配置块。 |
| `PATCH /api/instance/settings/experimental` | 切换实验功能。请求正文根据 `patchInstanceExperimentalSettingsSchema` 验证。 |

两个 `GET` 路由只要求经过身份验证的组织访问权限；每个 `PATCH` 都需要实例管理员权限（本地可信模式中的 `local_implicit` 看板会话也通过检查）。在云托管实例中，平台会在常规设置中固定 `executionMode`，因此尝试更改它会返回 `403`，错误码为 `execution_mode_platform_managed`。托管运维人员隐藏的设置也会被固定，错误码为 `settings_operator_managed`；但如果写入的值与当前值相同，则始终会通过。

常规设置中已不再包含 `keyboardShortcuts` 字段。所有已登录用户始终启用键盘快捷键，因此没有实例级开关，也没有个人偏好可供读取或更改。常规设置的 patch 请求正文采用严格验证：发送 `keyboardShortcuts` 现在会失败并返回 `400`。曾短暂提供个人快捷键开关的用户偏好端点也已移除。

实验性设置的响应和 patch 请求正文包含布尔字段 `enableBuiltInAgents`。使用[内置智能体 API](./built-in-agents.md) 前，请先将它设为 `true`；关闭时，这些路由会返回 `404 Not Found`。

---

## 任务排空

任务排空是实例范围的暂停机制。启用期间，智能体不会接收新任务，因此你可以在重启或维护窗口前平稳关闭实例，而不必取消已经在运行的工作。

| 端点 | 用途 |
|---|---|
| `GET /api/instance/task-drain` | 读取当前排空状态（是否正在排空，以及开始和到期时间）。任何经过身份验证的组织成员均可读取。 |
| `POST /api/instance/task-drain` | 开始排空或刷新现有排空。请求正文根据 `startTaskDrainRequestSchema` 验证；可传入 `ttlMs`，让排空在指定毫秒数后自动到期；省略时则会持续生效，直到你手动停止。返回最终排空状态。 |
| `DELETE /api/instance/task-drain` | 停止排空。返回 `{ "wasActive": <boolean> }`，表示调用时是否处于排空状态。 |

读取状态只要求经过身份验证的组织访问权限；开始或停止排空需要实例管理员权限。重叠的启动/停止请求会串行处理，因此审计日志和实时排空状态始终一致。

### 空闲保持锁

为了节省计算资源，需要停止空闲实例的主机可以获取短期的独占*空闲保持锁*，而不是使用常规排空：

- **获取保持锁：** 使用 `POST /api/instance/task-drain`，请求正文为 `{ "purpose": "idle", "ttlMs": <5000–300000> }`。空闲保持锁必须在五分钟内自行过期。若已有任何其他排空处于活动状态，请求会以 `409 Conflict` 失败。响应包含 `ownerId`。
- **检查能否安全休眠：** 调用 `GET /api/instance/task-drain?idleSleepSafety=1&ownerId=<ownerId>`。除常规排空状态外，响应还会包含 `idleSleepSafety` 对象，其中 `backgroundWork` 可能是 `none`、`present` 或 `unknown`。只有 `none` 表示在同一保持锁持续生效的期间，已接受的请求、计划任务、清理工作、插件 worker 和已保存工作均已结束。读取此信息需要实例管理员权限。
- **释放保持锁：** 调用 `DELETE /api/instance/task-drain?ownerId=<ownerId>`。所有者不匹配时，释放请求会以 `409 Conflict` 失败。

如果设置了 `PAPERCLIP_DB_BACKUP_IDLE_CHECKPOINT_ENABLED=1`（见[环境变量](../deploy/environment-variables.md)），安全检查还会先创建并验证数据库备份，然后才报告 `none`。云托管实例中，只有 Paperclip Cloud 控制平面可以运行此备份检查点检查。

---

## 云端生命周期

这两个路由供 Paperclip Cloud 管理的实例使用，以便 Cloud 控制平面保持组织归档状态与实例一致。在自托管实例中，二者都会返回 `404` 和 `{ "error": "not_cloud_managed" }`。两个路由都要求实例管理员权限。

| 端点 | 用途 |
|---|---|
| `GET /api/instance/lifecycle` | 报告由 Cloud 固定的主公司的状态：`primaryCompanyId`、`primaryCompanyStatus`（公司不存在时为 `"missing"`）和 `otherUnarchivedCompanyCount`。 |
| `POST /api/instance/lifecycle/unarchive-primary` | 将已归档的主公司恢复为 `active`，例如从 Paperclip Cloud 恢复某个组织时会执行此操作。返回 `{ "status": ..., "changed": true }`；如果公司原本就未归档，则返回 `changed: false`。公司已删除时返回 `404` 和 `primary_company_not_found`。 |

---

## 数据库备份

```
POST /api/instance/database-backups
```

触发手动数据库备份。返回结构化结果，包含备份目录、保留策略和耗时。计划备份走相同代码路径，但没有独立的 HTTP 路由。

仅限实例管理员调用。

---

<a id="claim-first-instance-admin"></a>
## 认领首位实例管理员

```
POST /api/bootstrap/claim
```

在要求登录的新建私有实例中，此端点允许第一位登录者将自己提升为 `instance_admin`，无需手动向数据库写入初始数据。第一个提出认领请求的人会成功；之后的所有人都无法认领。

**可用条件。** 只有实例同时设置 `deploymentMode` 为 `authenticated`、`deploymentExposure` 为 `private` 时，此路由才存在。其他配置下会返回 `404` 和 `Browser first-admin claim is not available`。

**云托管实例。** 如果实例由 Paperclip 控制平面管理，你不会看到认领页面，因为实例启动后就可以直接使用。此类环境中的身份由控制平面管理；它登录的用户有意不会获得 `instance_admin` 角色。因此 `GET /api/health` 完全跳过首位管理员检查，并始终报告 `bootstrapStatus: "ready"`。自托管实例不受影响：如果服务器由你自行运行，认领流程会按本页所述方式工作。

**身份验证。** 调用方必须使用已登录的浏览器会话（看板调用方的会话来源必须是浏览器）。其他调用方（智能体、CLI token、未经身份验证的请求）会收到 `401` 和 `Sign in from a browser session before claiming first admin`。

**行为。** 如果尚无 `instance_admin`，系统会将已登录用户提升为 `instance_admin`。认领操作具有原子性：系统会锁定数据表，确保恰好只有一位用户可以成功。如果实例已经被认领，路由会返回 `409` 和 `Someone else has already claimed this instance`。

请求不需要正文。成功时响应如下：

```json
{ "claimed": true, "userId": "<the promoted user's id>" }
```

---

## LLM 智能体配置反射

这些端点供智能体查询服务器可用的适配器及图标。

| 端点 | 用途 |
|---|---|
| `GET /api/llms/agent-configuration.txt` | 以纯文本导出可用智能体适配器及其声明的结构。 |
| `GET /api/llms/agent-configuration/:adapterType.txt` | 相同内容，但只包含指定适配器类型。 |
| `GET /api/llms/agent-icons.txt` | 列出支持的智能体图标名称（`AGENT_ICON_NAMES`）。 |

只有看板调用方和权限包含 `canCreateAgents` 的智能体可以访问；其他调用方会收到 `403`。

---

## 环境

环境是由环境驱动插件声明、通过插件管理的执行后端（自定义沙盒、运行器等）。

| 端点 | 用途 |
|---|---|
| `GET /api/companies/:companyId/environments` | 列出公司已配置的环境。 |
| `GET /api/companies/:companyId/environments/capabilities` | 列出可用环境驱动报告的功能。 |
| `POST /api/companies/:companyId/environments` | 创建环境。请求正文根据 `createEnvironmentSchema` 验证。 |
| `POST /api/companies/:companyId/environments/probe-config` | 测试候选环境配置但不保存。请求正文根据 `probeEnvironmentConfigSchema` 验证。 |
| `GET /api/environments/:id` | 读取单个环境。 |
| `PATCH /api/environments/:id` | 更新环境。请求正文根据 `updateEnvironmentSchema` 验证。 |
| `DELETE /api/environments/:id` | 删除环境。 |
| `POST /api/environments/:id/probe` | 探测现有环境（检查驱动健康状态）。 |
| `GET /api/environments/:id/leases` | 列出环境中的活动租约。 |
| `GET /api/environment-leases/:leaseId` | 读取单个租约。 |

---

## 执行工作区

执行工作区是 Paperclip 为任务运行创建的实际工作目录。

| 端点 | 用途 |
|---|---|
| `GET /api/companies/:companyId/execution-workspaces` | 列出公司的执行工作区。 |
| `GET /api/execution-workspaces/:id` | 读取单个工作区。 |
| `GET /api/execution-workspaces/:id/close-readiness` | 检查工作区是否可以安全关闭（没有待处理操作或未提交变更）。 |
| `GET /api/execution-workspaces/:id/workspace-operations` | 列出工作区上记录的操作。 |
| `PATCH /api/execution-workspaces/:id` | 更新工作区。请求正文根据 `updateExecutionWorkspaceSchema` 验证。 |
| `POST /api/execution-workspaces/:id/runtime-services/:action` | 对工作区中的运行时服务执行控制操作。 |
| `POST /api/execution-workspaces/:id/runtime-commands/:action` | 对工作区中的运行时命令执行控制操作。两个路由共用 `workspaceRuntimeControlTargetSchema` 请求正文和同一个处理器。 |

---

## 相关页面

- [参考 → 适配器 API](./adapters.md)：适配器注册表端点。
- [参考 → 插件 API](./plugins.md)：插件安装和生命周期管理。
- [参考 → 部署 → 环境变量](../deploy/environment-variables.md)：这些管理接口会使用的环境变量。
