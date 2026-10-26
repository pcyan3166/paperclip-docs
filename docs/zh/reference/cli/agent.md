---
paperclip_version: v2026.831.1
seo_title: CLI 代理命令
seo_description: 招募、检查、配置并引导负责公司工作的 AI 代理，包括适配器、预算和生命周期管理。
---

# 代理命令

`agent` 命令组用于创建、检查、配置并引导负责公司工作的 AI 代理。需要招募代理、为其分配预算和权限、调整指令与技能、唤醒它开始运行，或暂停和终止它时，可以使用这些命令。它们也是配置本地 Claude 或 Codex 会话、让会话*以某个代理身份*运行的直接方式——参见 [`agent local-cli`](#agent-local-cli)。

请牢记执行模型：代理实际的推理、编码和研究工作都在 Paperclip 运行时及其适配器中**由服务器端执行**。这些命令通过 API 触发和观察工作；CLI 本身不会运行模型。唯一例外是 `agent local-cli`：它会配置 shell，让人或 AI 在本地以该代理身份运行 Claude/Codex，再通过 API 回报结果。

所有命令都接受通用客户端标志：`--data-dir <path>`、`--api-base <url>`、`--api-key <token>`、`--context <path>`、`--profile <name>` 和 `--json`。公司范围命令还接受 `-C, --company-id <id>`。解析方式请参阅[通用选项](./common-options.md)。

---

## 身份与收件箱

这些命令从*当前*凭据的视角读取数据，也就是 API 密钥或配置文件所限定的代理。

```sh
paperclipai agent me
paperclipai agent inbox
paperclipai agent inbox-mine --user-id <board-user-id> --status todo,in_progress
```

| 命令 | 用途 |
|---|---|
| `agent me` | 显示当前代理身份（`GET /api/agents/me`）。这是确认密钥或配置文件解析到哪个代理的最快方法。 |
| `agent inbox` | 列出当前代理被分配的收件箱条目，即等待该代理处理的 issue 的简要视图。 |
| `agent inbox-mine --user-id <id>` | 列出当前代理收件箱中由指定看板用户处理或归档的条目。`--status <csv>` 可按 issue 状态缩小范围，例如 `todo,in_progress`。 |

> **注意：** `agent me` 和 `agent inbox` 以当前凭据代表的身份运行。使用代理 API 密钥时，它们描述该代理；使用看板凭据时，它们描述看板自身的身份，通常不是你想要的结果——请先显式传入代理上下文。

---

## 列出和读取代理

```sh
paperclipai agent list --company-id <company-id>
paperclipai agent get <agent-id>
```

`agent list` 必须提供 `-C, --company-id <id>`，并返回公司中所有代理的角色、状态、汇报对象、月预算与月支出（单位为分）。`agent get <agentId>` 返回一条完整代理记录。编写脚本时，可对任一命令使用 `--json`。

---

## 创建和招募代理

创建代理有两种方式，二者含义不同。

```sh
paperclipai agent create --company-id <company-id> \
  --payload-json '{"name":"Builder","adapterType":"codex_local"}'

paperclipai agent hire --company-id <company-id> \
  --payload-json '{"name":"CTO","role":"cto","reason":"Lead engineering"}'
```

| 命令 | 用途 |
|---|---|
| `agent create` | 根据 `CreateAgent` JSON 请求体直接创建代理（`POST /api/companies/{id}/agents`）。拥有看板权限且希望立即创建代理时使用。 |
| `agent hire` | 创建代理*招募请求*（`POST /api/companies/{id}/agent-hires`）。招募流程受治理、必须经过看板审批，而不能直接创建启用中的代理时使用。 |

两者都必须提供 `-C, --company-id <id>` 和 `--payload-json <json>`。请求体在发送前会经过校验；格式不正确时会在本地给出清晰错误，而不必等待服务器响应。

> **提示：** 手动编写请求体容易出错。`paperclip-create-agent` 技能可以检查适配器选项并为你起草有效配置。看板可通过[审批](./approval.md)命令审核并处理招募请求。

---

## 更新和删除代理

```sh
paperclipai agent update <agent-id> --payload-json '{"title":"Senior Builder"}'
paperclipai agent delete <agent-id> --yes
```

`agent update` 根据 `UpdateAgent` JSON 请求体修改代理。`agent delete` 会移除代理，且**必须提供 `--yes` 才会运行**——命令没有其他确认步骤，因此该标志就是防误删的安全措施。

---

## 生命周期控制

这些命令可切换代理的运行状态。每条命令都接受单个 `<agentId>` 参数，并向对应代理端点发送 POST 请求。

```sh
paperclipai agent pause <agent-id>
paperclipai agent resume <agent-id>
paperclipai agent approve <agent-id>
paperclipai agent terminate <agent-id>
```

| 命令 | 用途 |
|---|---|
| `agent pause` | 暂停代理，使其不再被唤醒处理新工作。 |
| `agent resume` | 恢复已暂停的代理。 |
| `agent approve` | 批准待审批代理，例如等待看板签字确认的招募请求所创建的代理。 |
| `agent terminate` | 终止代理。需要退役代理但保留其记录时使用，而不是删除记录。 |

---

## 唤醒代理

`agent wake` 用于请求立即唤醒一次心跳，让服务器现在运行代理适配器，而不是等待调度器。它接受代理 ID 或短名称/URL 键；按短名称传入时，必须提供 `-C, --company-id <id>`，让服务器能够解析该名称。

```sh
paperclipai agent wake <agent-id-or-shortname> \
  --company-id <company-id> \
  --reason "Pick up the new spec" \
  --payload '{"issueId":"<issue-id>"}'
```

| 标志 | 用途 |
|---|---|
| `--source <source>` | 调用来源：`timer`、`assignment`、`on_demand` 或 `automation`。默认值为 `on_demand`。 |
| `--trigger <trigger>` | 触发详情：`manual`、`ping`、`callback` 或 `system`。默认值为 `manual`。 |
| `--reason <text>` | 便于人阅读的唤醒原因。 |
| `--payload <json>` | 传给本次运行的 JSON **对象**。任何非 JSON 对象值都会被拒绝。 |
| `--idempotency-key <key>` | 去重键，防止重复发送的唤醒请求触发多次。 |
| `--force-fresh-session` | 请求新建适配器会话，而不是恢复已有会话。 |

命令解析代理后，会向 `/api/agents/{id}/wakeup` 发送 POST 请求。服务器运行适配器，CLI 输出唤醒响应。要跟踪和检查生成的运行，请使用 [`run`](./run.md) 子命令。

> **注意：** `agent wake`、`agent heartbeat:invoke` 和顶层 `heartbeat run` 最终都会进入相同的服务器端唤醒流程。三者中 `agent wake` 的控制选项最完整，支持来源、触发方式、请求体、幂等键和新会话设置。

---

## 心跳与 Claude 登录

```sh
paperclipai agent heartbeat:invoke <agent-id>
paperclipai agent claude-login <agent-id>
```

| 命令 | 用途 |
|---|---|
| `agent heartbeat:invoke` | 直接调用代理心跳（`POST /api/agents/{id}/heartbeat/invoke`）。这是无参数的简单唤醒；需要控制来源、请求体或幂等性时使用 `agent wake`。 |
| `agent claude-login` | 为使用 Claude 适配器的代理触发 Claude 登录流程（`POST /api/agents/{id}/claude-login`）。代理需要在服务器端重新建立 Claude 凭据时使用。 |

---

## 权限

```sh
paperclipai agent permissions:update <agent-id> \
  --payload-json '{"canCreateAgents":true,"canAssignTasks":true}'
```

`permissions:update` 根据 `UpdateAgentPermissions` JSON 请求体更新代理的作用域权限（`PATCH /api/agents/{id}/permissions`）。它控制代理可执行哪些受治理操作，例如创建其他代理或分配任务；这些权限独立于预算和指令。

---

## 配置与修订版本

代理配置带有版本记录。你可以读取当前已脱敏配置、浏览修订历史、查看单个修订版本并回滚到指定版本。

```sh
paperclipai agent configuration <agent-id>
paperclipai agent config-revisions <agent-id>
paperclipai agent config-revision:get <agent-id> <revision-id>
paperclipai agent config-revision:rollback <agent-id> <revision-id>
```

| 命令 | 用途 |
|---|---|
| `agent configuration` | 获取已脱敏的代理配置。秘密字段会被遮盖，因此可以安全地打印和分享。 |
| `agent config-revisions` | 列出代理配置修订历史。 |
| `agent config-revision:get` | 按 `<revision-id>` 获取一个修订版本，查看具体变更。 |
| `agent config-revision:rollback` | 将代理回滚到较早的修订版本（`POST .../config-revisions/{id}/rollback`）。配置更改有误时，可借此快速恢复。 |

---

## 运行时状态与任务会话

这些命令查看代理的*实时*执行状态，包括当前运行时会话和正在运行的任务会话。

```sh
paperclipai agent runtime-state <agent-id>
paperclipai agent runtime-state:reset-session <agent-id> --task-key <key>
paperclipai agent task-sessions <agent-id>
```

| 命令 | 用途 |
|---|---|
| `agent runtime-state` | 获取代理当前运行时状态。 |
| `agent runtime-state:reset-session` | 重置代理运行时会话。传入 `--task-key <key>` 可重置特定任务会话；省略该选项则会整体重置代理会话。代理因陈旧会话状态而卡住时可以使用。 |
| `agent task-sessions` | 列出代理的任务会话。 |

---

## 技能

代理的*期望*技能集合是服务器端的期望状态字段。这些命令可读取和替换该字段。它们**不会**在本机安装技能——本地安装由 `agent local-cli` 负责。

```sh
paperclipai agent skills <agent-id>
paperclipai agent skills:sync <agent-id> --desired-skills paperclip,github
```

| 命令 | 用途 |
|---|---|
| `agent skills` | 列出代理当前技能。 |
| `agent skills:sync` | 使用逗号分隔的 `--desired-skills` 列表替换代理期望使用的技能（`POST /api/agents/{id}/skills/sync`）。Paperclip 强制要求的技能仍由服务器统一执行。 |

> **注意：** 填充公司的技能*库*以及编写技能属于另一个功能领域，请参阅[技能参考](./skills.md)。`agent skills:sync` 只决定某个代理使用库中的哪些技能。

---

## 指令

代理指令可以是磁盘上的单个路径（例如读取 `AGENTS.md` 的适配器使用），也可以是保存在服务器端、由系统管理的一组文件。以下命令涵盖这两种情况。

```sh
paperclipai agent instructions-path:update <agent-id> \
  --payload-json '{"path":"/tmp/AGENTS.md","adapterConfigKey":"instructionsFilePath"}'

paperclipai agent instructions-bundle <agent-id>
paperclipai agent instructions-bundle:update <agent-id> --payload-json '{"mode":"managed"}'

paperclipai agent instructions-file:get <agent-id> --path AGENTS.md
paperclipai agent instructions-file:put <agent-id> --path AGENTS.md --content-file ./AGENTS.md
paperclipai agent instructions-file:delete <agent-id> --path AGENTS.md
```

| 命令 | 用途 |
|---|---|
| `agent instructions-path:update` | 将代理指向某个指令文件路径。进程适配器要求请求体包含 `adapterConfigKey`；相对路径还要求设置 `adapterConfig.cwd`。 |
| `agent instructions-bundle` | 获取代理由系统管理的指令包。 |
| `agent instructions-bundle:update` | 使用 `UpdateAgentInstructionsBundle` JSON 请求体更新指令包，例如将 `mode` 切换为 `managed`。 |
| `agent instructions-file:get` | 读取指令包内相对路径的文件。必须指定 `--path <path>`。 |
| `agent instructions-file:put` | 创建或更新指令包内相对路径的文件。可以用 `--content <text>` 内联提供内容，或通过 `--content-file <path>` 从磁盘读取；添加 `--clear-legacy-prompt-template` 可在同一次写入中清除旧提示模板。 |
| `agent instructions-file:delete` | 根据 `--path` 删除指令包内的文件。 |

> **提示：** 多行内容优先使用 `--content-file`，而非 `--content`。这样无需处理复杂 shell 引号，也能将指令文件纳入自己的代码库进行版本管理。

---

## `agent local-cli`

这是在本地将 Claude 或 Codex 作为 Paperclip 代理运行的实用桥梁。它接受代理 ID 或短名称/URL 键，要求提供 `-C, --company-id <id>`，并一次性完成以下操作：

1. 生成一个长期有效的代理 API 密钥（使用 `--key-name <name>` 指定标签；默认生成带时间戳的 `local-cli` 密钥）。
2. 将内置 Paperclip 技能安装到 `~/.codex/skills`、`~/.claude/skills` 和 `~/.kimi-code/skills`，安装方式为符号链接（如果已设置，则遵循 `CODEX_HOME`、`CLAUDE_HOME` 和 `KIMI_CODE_HOME`）。
3. 输出 shell `export` 语句，设置 `PAPERCLIP_API_URL`、`PAPERCLIP_COMPANY_ID`、`PAPERCLIP_AGENT_ID` 和 `PAPERCLIP_API_KEY`。

```sh
paperclipai agent local-cli <agent-id-or-shortname> --company-id <company-id>
paperclipai agent local-cli codexcoder --company-id <company-id> --key-name workstation
paperclipai agent local-cli claudecoder --company-id <company-id> --no-install-skills
```

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 必填。代理所属的公司（也用于解析短名称）。 |
| `--key-name <name>` | 新建 API 密钥的标签。默认为 `local-cli`；传入空值时会回退到带时间戳的名称。 |
| `--no-install-skills` | 跳过向 `~/.codex/skills`、`~/.claude/skills` 和 `~/.kimi-code/skills` 安装技能。仅需要密钥和导出语句时使用。 |

将输出的 export 语句导入 shell，然后启动 Codex 或 Claude；它们现在会以该代理身份通过 Paperclip API 执行操作：

```sh
eval "$(paperclipai agent local-cli codexcoder --company-id <company-id> --json | jq -r .exports)"
```

> **警告：** 新生成的 API 密钥明文只会在命令输出中显示**一次**。导出语句中包含该密钥。请将输出视为秘密信息，并在本地会话结束后使用 `token agent revoke` 撤销密钥。参阅[认证](./authentication.md)。

安装技能时，命令会输出每种工具的汇总（`linked` / `removed` / `skipped` / `failed` 计数及目标目录）。它会修复失效的符号链接并移除仅供维护者使用的技能，因此安全且幂等，可以重复运行 `local-cli`。安装要求检出目录中存在内置的 `skills` 目录；如果找不到该目录，命令会直接失败，不会静默地什么也不安装。

---

## 另请参阅

- [认证](./authentication.md)——看板令牌、代理密钥，以及 `connect` 如何生成与角色身份关联的配置文件。
- [通用选项](./common-options.md)——共享标志和 API 基础 URL 解析方式。
- [运行](./run.md)——检查和控制 `agent wake` 触发的心跳运行。
- [提示](./prompt.md)——通过 `agent prompt` 将任务和评论交给代理处理。
- [技能](./skills.md)——浏览技能目录、填充公司技能库并编写技能。
- [审批](./approval.md)——处理招募请求及其他需要看板审批的操作。
- [公司](./company.md)——公司范围代理命令所需的公司上下文。
