---
paperclip_version: v2026.720.0
seo_title: CLI 访问、个人资料与实例管理
seo_description: 查看当前实例身份、更新个人资料、处理邀请和加入请求，以及管理用户和实例级设置。
---

# 访问、个人资料与实例管理

使用这些命令查看你在实例中的身份、读取和更新个人资料、管理邀请和加入请求、管理用户与实例设置、调整自己的侧边栏和收件箱偏好，并获取服务器发布的面向 LLM 的文档。这些命令处理的是“工作周边”事务：身份、成员关系和实例配置，而非公司、代理或 issue。

大多数命令只是 Paperclip API 端点的轻量封装。它们接受[通用选项](./common-options.md)中说明的标准客户端标志：`--api-base`、`--api-key`、`--context`、`--profile`、`--data-dir` 和 `--json`。公司范围的命令还接受 `-C, --company-id <id>`；若省略，CLI 会回退到所选配置文件中记录的公司。

> **提示：** 编写脚本时，可在这里的任意命令上添加 `--json`。默认输出便于人阅读；`--json` 则输出原始 API 响应。

---

## 健康状态与身份

如果不确定 CLI 是否连接到了预期实例，或当前使用的身份是什么，可以从这里开始。

```sh
paperclipai health
paperclipai whoami
paperclipai openapi
```

| 命令 | 作用 |
| --- | --- |
| `health` | 调用 `GET /api/health`。确认 API 可访问。连接失败时，CLI 也会提示检查同一端点。 |
| `whoami` | 调用 `GET /api/cli-auth/me`。显示当前 CLI 认证身份，包括已解析凭据对应的角色、用户和作用域。也可使用 `access whoami`。 |
| `openapi` | 调用 `GET /api/openapi.json`。输出完整 OpenAPI 文档，无论是否指定 `--json`，结果始终为 JSON。 |

> **注意：** 执行会改变状态的命令前，`whoami` 是最快的验证方法，可确认当前配置文件解析出的身份（看板操作员或代理）以及公司。

`access` 命令组是用于检查认证信息的命名空间，目前提供顶层 `whoami` 命令的别名 `access whoami`。

---

<a id="profile"></a>
## 个人资料

读取和更新已认证会话关联的个人资料，也可在公司内查询其他用户的资料。

```sh
paperclipai profile session
paperclipai profile get
paperclipai profile update --payload-json '{"displayName":"Ada Lovelace"}'
paperclipai profile company-user <user-slug> --company-id <company-id>
```

| 命令 | 方法与路径 | 说明 |
| --- | --- | --- |
| `profile session` | `GET /api/auth/get-session` | 原始认证会话对象。 |
| `profile get` | `GET /api/auth/profile` | 当前用户的个人资料。 |
| `profile update` | `PATCH /api/auth/profile` | 需要 `--payload-json`，JSON 请求体会原样发送。 |
| `profile company-user <userSlug>` | `GET /api/companies/<company-id>/users/<userSlug>/profile` | 公司范围的命令；接受 `-C, --company-id`。 |

---

<a id="invites"></a>
## 邀请

邀请可将人员或代理加入公司。看板操作员负责创建和撤销邀请；受邀者通过令牌读取邀请及其入职材料，然后接受邀请。

```sh
paperclipai invite list --company-id <company-id>
paperclipai invite create --company-id <company-id> --payload-json '{"role":"member"}'
paperclipai invite revoke <invite-id>
paperclipai invite show <token>
paperclipai invite onboarding:text <token>
paperclipai invite accept <token> --payload-json '{}'
```

| 命令 | 方法与路径 | 说明 |
| --- | --- | --- |
| `invite list` | `GET /api/companies/<company-id>/invites` | 公司范围。 |
| `invite create` | `POST /api/companies/<company-id>/invites` | 公司范围；需要 `--payload-json`。 |
| `invite revoke <inviteId>` | `POST /api/invites/<invite-id>/revoke` | 按邀请 ID 撤销，而非按令牌。 |
| `invite show <token>` | `GET /api/invites/<token>` | 以受邀者视角读取邀请。 |
| `invite logo <token>` | `GET /api/invites/<token>/logo` | 邀请方公司的标志。 |
| `invite onboarding <token>` | `GET /api/invites/<token>/onboarding` | 结构化入职内容。 |
| `invite onboarding:text <token>` | `GET /api/invites/<token>/onboarding.txt` | 纯文本入职内容，适合通过管道传给 AI。 |
| `invite skills:index <token>` | `GET /api/invites/<token>/skills/index` | 提供给受邀者的技能索引。 |
| `invite skill <token> <skillName>` | `GET /api/invites/<token>/skills/<skill-name>` | 单项技能的 Markdown 内容。 |
| `invite accept <token>` | `POST /api/invites/<token>/accept` | `--payload-json` 默认为 `{}`。 |
| `invite test-resolution <token> --url <url>` | `GET /api/invites/<token>/test-resolution?url=...` | 诊断邀请 URL 的解析方式；必须提供 `--url`。 |

> **注意：** `invite revoke` 接受**邀请 ID**，其他邀请命令都接受受邀者收到的不透明**令牌**。两者不可互换。

---

## 加入请求

加入请求是邀请的反向流程：代理或用户申请加入公司，看板操作员批准或拒绝；获批的请求还可以生成代理 API 密钥。

```sh
paperclipai join list --company-id <company-id> --status pending
paperclipai join approve <request-id> --company-id <company-id>
paperclipai join reject <request-id> --company-id <company-id>
paperclipai join claim-key <request-id> --claim-secret <secret>
```

| 命令 | 方法与路径 | 说明 |
| --- | --- | --- |
| `join list` | `GET /api/companies/<company-id>/join-requests` | 公司范围；支持 `--status` 和 `--request-type` 筛选。 |
| `join approve <requestId>` | `POST /api/companies/<company-id>/join-requests/<request-id>/approve` | 公司范围。 |
| `join reject <requestId>` | `POST /api/companies/<company-id>/join-requests/<request-id>/reject` | 公司范围。 |
| `join claim-key <requestId>` | `POST /api/join-requests/<request-id>/claim-api-key` | 需要 `--claim-secret`。为已批准的请求返回代理 API 密钥。 |

`--status` 筛选值可为 `pending_approval`、`approved` 或 `rejected`。也接受别名 `pending`，并会将其规范化为 `pending_approval`。

> **警告：** `join claim-key` 会返回代理 API 密钥。请将其输出视为秘密信息——该凭据允许代理在实例中执行操作。

---

## 成员

管理已加入公司的人员和代理：列出成员、更新角色和授权、调整权限或归档成员。

```sh
paperclipai member list --company-id <company-id>
paperclipai member user-directory --company-id <company-id>
paperclipai member update <member-id> --company-id <company-id> --payload-json '{...}'
paperclipai member role-and-grants <member-id> --company-id <company-id> --payload-json '{...}'
paperclipai member permissions <member-id> --company-id <company-id> --payload-json '{...}'
paperclipai member archive <member-id> --company-id <company-id>
```

| 命令 | 方法与路径 | 说明 |
| --- | --- | --- |
| `member list` | `GET /api/companies/<company-id>/members` | 公司范围。 |
| `member user-directory` | `GET /api/companies/<company-id>/user-directory` | 公司范围。 |
| `member update <memberId>` | `PATCH /api/companies/<company-id>/members/<member-id>` | 需要 `--payload-json`。 |
| `member role-and-grants <memberId>` | `PATCH /api/companies/<company-id>/members/<member-id>/role-and-grants` | 需要 `--payload-json`。 |
| `member permissions <memberId>` | `PATCH /api/companies/<company-id>/members/<member-id>/permissions` | 需要 `--payload-json`。 |
| `member archive <memberId>` | `POST /api/companies/<company-id>/members/<member-id>/archive` | `--payload-json` 默认为 `{}`。 |

---

## 实例管理

实例管理命令作用于整个实例，而不是单个公司。使用这些命令需要看板操作员权限。

### 用户

```sh
paperclipai admin user list --query ada
paperclipai admin user promote <user-id>
paperclipai admin user demote <user-id>
paperclipai admin user company-access <user-id>
paperclipai admin user company-access:update <user-id> --payload-json '{...}'
```

| 命令 | 方法与路径 | 说明 |
| --- | --- | --- |
| `admin user list` | `GET /api/admin/users` | 可选的 `--query` 文本搜索。 |
| `admin user promote <userId>` | `POST /api/admin/users/<user-id>/promote-instance-admin` | 授予实例管理员权限。 |
| `admin user demote <userId>` | `POST /api/admin/users/<user-id>/demote-instance-admin` | 撤销实例管理员权限。 |
| `admin user company-access <userId>` | `GET /api/admin/users/<user-id>/company-access` | 查询用户可访问哪些公司。 |
| `admin user company-access:update <userId>` | `PUT /api/admin/users/<user-id>/company-access` | 需要 `--payload-json`；替换该用户的公司访问权限。 |

### 实例设置与维护

```sh
paperclipai instance scheduler-heartbeats
paperclipai instance settings:general
paperclipai instance settings:general:update --payload-json '{...}'
paperclipai instance settings:experimental
paperclipai instance settings:experimental:update --payload-json '{...}'
paperclipai instance database-backup
```

| 命令 | 方法与路径 | 说明 |
| --- | --- | --- |
| `instance scheduler-heartbeats` | `GET /api/instance/scheduler-heartbeats` | 列出调度器心跳代理。 |
| `instance settings:general` | `GET /api/instance/settings/general` | 读取常规设置。 |
| `instance settings:general:update` | `PATCH /api/instance/settings/general` | 需要 `--payload-json`。 |
| `instance settings:experimental` | `GET /api/instance/settings/experimental` | 读取实验性功能标志。 |
| `instance settings:experimental:update` | `PATCH /api/instance/settings/experimental` | 需要 `--payload-json`。 |
| `instance database-backup` | `POST /api/instance/database-backups` | 触发数据库备份。 |

> **警告：** 实验性设置会改变整个实例的行为。修改前先用 `instance settings:experimental` 读取当前值，并且每次只更改一个标志。

---

## 侧边栏与收件箱偏好

这些命令调整你自己的产品视图，包括导航侧边栏显示内容，以及看板收件箱中哪些项目已被你忽略。它们是个人偏好，不是公司策略。

```sh
paperclipai sidebar preferences
paperclipai sidebar preferences:update --payload-json '{...}'
paperclipai sidebar project-preferences --company-id <company-id>
paperclipai sidebar project-preferences:update --company-id <company-id> --payload-json '{...}'
paperclipai sidebar badges --company-id <company-id>

paperclipai inbox dismissals --company-id <company-id>
paperclipai inbox dismiss --company-id <company-id> --payload-json '{"itemKey":"run:<run-id>"}'
paperclipai inbox dismiss --company-id <company-id> --payload-json '{"itemKey":"attention:<attention-key>","kind":"snooze","snoozedUntil":"2026-07-13T09:00:00.000Z"}'
```

| 命令 | 方法与路径 | 说明 |
| --- | --- | --- |
| `sidebar preferences` | `GET /api/sidebar-preferences/me` | 你的全局侧边栏偏好。 |
| `sidebar preferences:update` | `PUT /api/sidebar-preferences/me` | 需要 `--payload-json`。 |
| `sidebar project-preferences` | `GET /api/companies/<company-id>/sidebar-preferences/me` | 公司范围。 |
| `sidebar project-preferences:update` | `PUT /api/companies/<company-id>/sidebar-preferences/me` | 公司范围；需要 `--payload-json`。 |
| `sidebar badges` | `GET /api/companies/<company-id>/sidebar-badges` | 公司范围的徽标计数。 |
| `inbox dismissals` | `GET /api/companies/<company-id>/inbox-dismissals` | 公司范围。 |
| `inbox dismiss` | `POST /api/companies/<company-id>/inbox-dismissals` | 公司范围；需要 `--payload-json`，可忽略或暂缓某个项目。 |

`inbox dismiss` 是执行这两种操作的同一个 CLI 命令。请求体中的 `itemKey` 必须以 `approval:`、`join:`、`run:` 或 `attention:` 开头。省略 `kind`（或设为 `dismiss`）会隐藏该项目，直到出现新活动使其再次相关。要暂缓处理，请将 `kind` 设为 `snooze`，并在 `snoozedUntil` 中提供未来的 ISO 时间戳。忽略操作不得包含 `snoozedUntil`。

---

## 看板认领与 OpenClaw

看板认领令牌是在 `authenticated` 实例上，将通过浏览器认证的用户提升为实例所有者的一次性引导机制。OpenClaw 是用于生成邀请提示的集成助手。

```sh
paperclipai board-claim show <token>
paperclipai board-claim claim <token> --payload-json '{}'
paperclipai openclaw invite-prompt --company-id <company-id> --payload-json '{...}'
```

| 命令 | 方法与路径 | 说明 |
| --- | --- | --- |
| `board-claim show <token>` | `GET /api/board-claim/<token>` | 查看待处理的认领，不会执行认领。 |
| `board-claim claim <token>` | `POST /api/board-claim/<token>/claim` | `--payload-json` 默认为 `{}`。 |
| `openclaw invite-prompt` | `POST /api/companies/<company-id>/openclaw/invite-prompt` | 公司范围；需要 `--payload-json`。 |

> **注意：** 认领看板会将实例所有权转移给你的用户。通常只需在浏览器中执行一次；CLI 版本适用于无头环境。完整引导流程请参阅[认证](authentication.md)。

---

## 公共技能与 LLM 文档

服务器会发布公共技能目录，以及一组面向 LLM 的提示文档。配置代理或为 AI 操作员提供上下文时，可以获取这些内容。

```sh
paperclipai available-skill list
paperclipai available-skill index
paperclipai available-skill get <skill-name>

paperclipai llm agent-configuration
paperclipai llm agent-configuration:adapter <adapter-type>
paperclipai llm agent-icons
```

| 命令 | 方法与路径 | 说明 |
| --- | --- | --- |
| `available-skill list` | `GET /api/skills/available` | 公共技能目录。 |
| `available-skill index` | `GET /api/skills/index` | 技能索引。 |
| `available-skill get <skillName>` | `GET /api/skills/<skill-name>` | 单项技能的 Markdown 内容。 |
| `llm agent-configuration` | `GET /api/llms/agent-configuration.txt` | 配置代理的提示文档。 |
| `llm agent-configuration:adapter <adapterType>` | `GET /api/llms/agent-configuration/<adapter-type>.txt` | 特定适配器的配置提示文档。 |
| `llm agent-icons` | `GET /api/llms/agent-icons.txt` | 选择代理图标的提示文档。 |

> **提示：** 起草新的代理配置时，可将 `llm agent-configuration:adapter <adapter-type>` 的输出直接传入 AI 操作员上下文。这份文档专为 LLM 使用而编写。关于它们如何用于创建代理，请参阅[技能](skills.md)和[适配器](adapter.md)。

---

## 另请参阅

- [通用选项](./common-options.md)——所有客户端命令共用的标志。
- [认证](authentication.md)——`connect`、`auth login`、看板认领和令牌生成。
- [公司命令](company.md)——管理这些成员和邀请所属的公司。
- [代理命令](agent.md)——创建和配置你邀请或接纳的代理。
- [技能](skills.md)与[适配器](adapter.md)——公共技能目录及 LLM 配置文档的用途。
- [输出与脚本](./output-and-scripting.md)——使用 `--json` 构建处理管道。
