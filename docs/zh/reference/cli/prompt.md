---
paperclip_version: v2026.609.0
seo_title: 从终端移交提示
seo_description: 无需打开界面，直接从 shell 向智能体移交任务并观察它开始处理。
---

# 提示移交

提示移交是从终端向智能体交办任务的方式。希望直接告诉智能体“去完成这项工作”，而不打开界面时，就使用此功能。它是 CLI 中将工作直接交给智能体、让它立即开始处理的最直接命令。

使用前请牢记：**提示移交不是聊天。** 它不会建立对话、流式返回答复，也不会保持会话打开。Paperclip 的沟通模型是**任务和评论**，而不是消息。提示移交会创建一条分配给目标智能体、状态为 `todo` 的任务，然后**唤醒**该智能体，使服务器端运行时接手工作。CLI 会返回新建的任务或评论，然后退出。实际工作由服务器在下一次心跳期间针对该任务执行；你应通过任务、运行和活动记录观察进度，而不是查看提示命令的输出。

共有三种命令，各适用于一种情况：

| 命令 | 身份类型 | 使用的身份验证 |
|---|---|---|
| `agent-prompt <agent> <agentApiKey> <prompt...>` | agent | 将智能体 API 密钥作为位置参数传入 |
| `agent prompt <prompt...>` | agent | 智能体配置文件/上下文（或 `--api-key-env`） |
| `board prompt --agent <agent> <prompt...>` | board | 控制面板凭据，可指定公司的任意智能体 |

三种方式最终执行相同操作：为智能体创建或更新任务，然后唤醒它（除非你选择不唤醒）。

---

## 移交实际会做什么

两种模式会经过相同逻辑。是否传入 `--issue` 决定使用哪条路径。

**不传 `--issue`（默认）——创建任务。** CLI 会创建一条新任务，包含：

- `title`——如果传入 `--title`，则使用该值；否则取提示中第一条非空行，并截断至 100 个字符。
- `description`——完整的提示文本。
- `status`——`todo`。
- `priority`——`medium`。
- `assigneeAgentId`——目标智能体。

随后，CLI 会将任务发送到 `POST /api/companies/<company-id>/issues`。除非设置了 `--no-wake`，否则接着会针对新任务唤醒智能体。

**传入 `--issue <id>`——评论已有工作。** CLI 不会创建任务，而是将提示作为评论发送到 `POST /api/issues/<issue-id>/comments`（评论的 `resume` 标志会记录是否应唤醒智能体）。除非设置了 `--no-wake`，否则接着会针对该任务唤醒智能体。

> **注意：** 默认路径每次都会创建新的 `todo` 任务，因此运行三次 `agent prompt "..."` 会得到三个独立任务，而不是一个包含三条消息的讨论串。如果你想“继续同一项工作”，通常应传入 `--issue`。

唤醒操作会调用 `POST /api/agents/<agent-id>/wakeup`，并将 `source: "on_demand"` 传入请求。它不会直接在当前命令中运行模型，而是告知 Paperclip 运行时“此智能体有工作要做”，由服务器安排运行。任务/评论创建完毕并确认唤醒后，提示命令就会返回。

---

## `agent-prompt`——使用明确指定的智能体密钥移交

手头已有某个智能体的 API 密钥，并希望通过一条命令将工作交给该智能体时，使用此命令。智能体引用和密钥都是位置参数，因此这种形式很适合复制到脚本或其他工具的配置中。

```sh
paperclipai agent-prompt <agent-id> <agent-api-key> "Draft the Q3 launch checklist and link the source issues"
```

智能体引用可以是智能体 ID、短名称（`urlKey`）或名称。CLI 会验证密钥确实属于指定智能体：它会使用该密钥调用 `/api/agents/me`，并检查指定智能体是否与密钥身份相符。如果密钥对应另一个智能体，命令会明确告知密钥实际所属的智能体并失败，不会悄悄将任务交给其他人。

| 参数 / 标志 | 用途 |
|---|---|
| `<agent>` | 目标智能体：ID、短名称或名称。必须与 API 密钥自身身份相符。 |
| `<agentApiKey>` | 作为位置参数传入的智能体 API 密钥。 |
| `<prompt...>` | 提示文本。密钥之后的所有参数都会合并为一条提示。 |
| `--issue <issueId>` | 在现有任务上发表评论，而不是创建新任务。 |
| `--title <title>` | 新任务的标题（设置 `--issue` 时忽略）。 |
| `--no-wake` | 创建/更新工作，但不唤醒智能体。 |

---

## `agent prompt`——以已配置的智能体身份移交

在已有智能体身份的会话中使用此命令，例如已保存的智能体配置文件，或环境变量中的智能体 API 密钥。这里不需要位置参数形式的密钥；凭据来自你的上下文。

```sh
# Use the active agent profile/context
paperclipai agent prompt "Investigate the failing nightly build and report findings"

# Read the key from a named environment variable
paperclipai agent prompt --api-key-env PAPERCLIP_API_KEY "Investigate the failing nightly build"

# Target a specific saved profile
paperclipai agent prompt --profile my-agent "Investigate the failing nightly build"
```

如果传入 `--agent`，CLI 会确认它与已验证身份相符；如果省略，则使用配置文件中的智能体（如果没有，再使用凭据解析出的身份）。无论哪种方式，智能体身份都只能向**自身**提交提示——这是智能体身份的设计目的，系统也会强制执行。要为*其他*智能体提交工作，请使用 `board prompt`。

| 标志 | 用途 |
|---|---|
| `--agent <agent>` | 智能体 ID、短名称或名称。默认使用配置文件/身份中的智能体；必须与已验证身份相符。 |
| `--api-key-env <name>` | 从指定环境变量读取智能体 API 密钥。变量未设置时会报错。 |
| `--issue <issueId>` | 在现有任务上发表评论，而不是创建新任务。 |
| `--title <title>` | 新任务的标题（设置 `--issue` 时忽略）。 |
| `--no-wake` | 创建/更新工作，但不唤醒智能体。 |

> **注意：** `agent prompt` 需要智能体配置文件。如果当前活动配置文件属于控制面板身份，此命令会拒绝执行，并提示改用智能体配置文件或 `board prompt`。配置文件中记录的身份类型才是判断依据。

---

## `board prompt`——以控制面板身份向任意智能体移交

拥有控制面板权限并希望将工作交给某公司的任意智能体时，使用此操作员命令。它不需要目标智能体的密钥，只需要你的控制面板凭据、公司和目标智能体。

```sh
paperclipai board prompt --company-id <company-id> --agent <agent-name-or-id> \
  "Take over the migration rollback and post a status update when staging is green"
```

`--agent` 为必填参数，`--company-id` 用于指定公司（简写为 `-C`）。CLI 会在公司内解析智能体、创建任务或评论，然后唤醒智能体；这与智能体命令使用相同的“创建/评论后唤醒”流程，但工作记录的操作者是控制面板身份。

| 标志 | 用途 |
|---|---|
| `--agent <agent>` | 必填。目标智能体：ID、短名称或名称。 |
| `-C, --company-id <id>` | 智能体所属公司。 |
| `--issue <issueId>` | 在现有任务上发表评论，而不是创建新任务。 |
| `--title <title>` | 新任务的标题（设置 `--issue` 时忽略）。 |
| `--no-wake` | 创建/更新工作，但不唤醒智能体。 |

> **注意：** `board prompt` 需要控制面板配置文件。使用智能体配置文件时，命令会拒绝执行并提示你改用控制面板身份。

---

## `--issue`：评论已有任务而非创建任务

`--issue <id>` 是这里最实用的标志，因为它会将操作从“创建”改为“继续”。传入后，提示文本会作为该任务的**评论**，而不是新建 `todo` 任务。此模式下会忽略 `--title`，因为没有创建任务，也就无需设置标题。

```sh
# Nudge an agent already working a task, and wake it to resume
paperclipai board prompt --company-id <company-id> --agent <agent-id> --issue PAP-142 \
  "The staging DB is back up — retry the migration step and confirm row counts"
```

任务引用可以是 UUID 或 `PAP-142` 这样的可读标识符。已有工作时都应使用 `--issue`：纠正正在进行的任务、回答智能体提出的问题，或在阻塞解除后将任务交还。只有确实要创建一项新的独立工作时，才使用默认路径（不传 `--issue`）。

---

## `--no-wake`：暂存工作但不触发运行

默认情况下，每次移交都会唤醒智能体。`--no-wake` 会跳过最后一步：任务仍会创建（或评论仍会发布），任务仍会分配给智能体，但不会发送唤醒信号，因此此命令不会安排运行。

```sh
# Queue several tasks, then let the normal scheduler pick them up
paperclipai board prompt --company-id <company-id> --agent <agent-id> --no-wake \
  "Audit the auth module for unused permission checks"
```

批量暂存工作、不希望每条命令都立即启动运行时，可使用 `--no-wake`；向智能体提交后续工作并希望它在下个周期自行处理时也可使用；编写脚本并想单独控制唤醒时同样适用。请谨慎使用：没有唤醒信号，智能体就不会立即处理任务，除非其他事件唤醒它（调度器、提及或明确运行 `agent wake`）。如果希望它立即行动，就不要关闭唤醒。

---

## 阅读结果

每个命令都会输出一个简短结果对象，说明执行了什么。`mode` 字段最便于确认操作是否符合预期：

| 字段 | 含义 |
|---|---|
| `mode` | 新建任务时为 `issue`，向现有工作追加内容时为 `comment`。 |
| `actor` | 根据运行命令的身份，取值为 `agent` 或 `board`。 |
| `agent` | 解析出的目标智能体（`id`、`name`、`urlKey`）。 |
| `issue` / `comment` | 新建的任务或评论。 |
| `wakeup` | 唤醒响应；使用 `--no-wake` 时为 `null`。 |

添加 `--json` 可获取原始对象以供脚本处理。`mode: "issue"` 时若 `wakeup` 为 `null`，表示任务已提交但尚未开始运行——这正是 `--no-wake` 的效果。

---

## 另请参阅

- [智能体命令](./agent.md)——检查智能体、配置 `agent local-cli` 并直接唤醒智能体。
- [任务命令](./issue.md)——列出、获取、更新任务并发表评论；提示移交会创建这些任务。
- [运行检查](./run.md)——查看唤醒信号安排的服务器端运行。
- [身份验证](./authentication.md)——生成这些命令所需的智能体密钥和控制面板令牌。
- [通用选项](./common-options.md)——`--api-base`、`--api-key`、`--context`、`--profile`、`--json` 等共享选项。
- [输出和脚本](./output-and-scripting.md)——在管道中处理 `--json` 结果。
