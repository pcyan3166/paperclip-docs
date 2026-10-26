---
paperclip_version: v2026.609.0
seo_title: CLI 技能命令
seo_description: 浏览应用内置目录、将技能安装到公司库，并为代理关联所需技能。
---

# 技能命令

使用这些命令为代理提供能力：浏览应用内置技能目录、将技能安装到公司技能库、关联代理实际需要的技能，并使已安装技能与其固定来源保持同步。技能可以教会代理可重复执行的流程（例如创建拉取请求、运行浏览器、遵循评审清单），无需每次都重写提示词。

命令分为两组。日常使用主要选择复数形式的 `paperclipai skills` 命令组——它可以识别目录引用，按 `id` / `key` / `slug` 解析技能，并输出易读表格。单数形式的 `paperclipai skill` 命令组是同一公司技能 API 端点的轻量请求体封装；只有脚本需要直接控制请求体时才使用。

---

## 三种操作

`paperclipai skills` 涵盖三种不同操作。理解这些区别是掌握整体机制的关键：

| 操作 | 命令 | 变更内容 |
|---|---|---|
| **安装到公司** | `skills install`、`import`、`create`、`scan-projects` | 在公司技能库中添加或更新技能。公司所有成员均可使用，但**不会**自动将技能关联到任何代理。 |
| **关联到代理** | `skills agent sync`、`skills agent clear` | 修改代理期望使用的公司技能集合。`sync` 必须指定 `--mode`（`add`、`remove` 或 `replace`），以增补、移除或替换集合；`clear` 会清空集合。 |
| **适配器运行时同步** | `skills agent list` 会报告同步状态；`sync` / `clear` 会触发同步 | 服务器端适配器会将期望集合与磁盘文件协调同步，并返回 `AgentSkillSnapshot`。 |

> **注意：** `skills agent sync` 现在必须指定 `--mode`（`add`、`remove` 或 `replace`），用于选择新增、移除或替换指定技能。`skills agent clear` 会直接清空集合（内部通过空列表执行 `replace`）。内置 Paperclip 技能仍以只读条目的形式保留在公司库中，但服务器不会再强行将它们关联到代理。

> **提示：** 在任意命令后添加 `--json` 可输出原始 API 结果以供脚本处理。每条公司范围命令都接受 `--company-id <company-id>`（并遵循已选配置文件/上下文）。详情请参阅[通用选项](./common-options.md)。

---

## 目录：先浏览再安装

Paperclip 应用内置精选技能目录。浏览、搜索或查看目录不需要公司上下文，也不会向公司写入任何内容。

```sh
paperclipai skills browse --kind bundled --category github
paperclipai skills search "pull request" --kind bundled
paperclipai skills inspect github-pr-workflow
```

| 命令 | 用途 |
|---|---|
| `browse` | 列出目录技能；可按 `--kind`、`--category` 或 `--query` 筛选。 |
| `search <query>` | 与 `browse` 类似，但将搜索文本作为位置参数传入。 |
| `inspect <catalogRef>` | 完整显示一项目录技能，包括其文件清单（路径、类型、大小和 sha256）。 |

浏览/搜索筛选标志：

| 标志 | 用途 |
|---|---|
| `--kind <kind>` | 按目录类别筛选：`bundled` 或 `optional`。 |
| `--category <slug>` | 按目录类别 slug 筛选。 |
| `--query <text>` | 全文搜索（仅用于 `browse`；`search` 使用其位置参数）。 |

`<catalogRef>` 是目录技能的 `id`、`key` 或唯一 `slug`。安装前请先检查技能，以确认哪些文件会加入公司技能库。

---

## 将目录技能安装到公司库

`skills install` 会将一项目录技能实体化为公司管理的技能。这属于**公司级安装**，不会将技能关联到代理。

```sh
paperclipai skills install github-pr-workflow --company-id <company-id>
paperclipai skills install github-pr-workflow --as review-prs --company-id <company-id>
```

| 标志 | 用途 |
|---|---|
| `--as <slug>` | 覆盖公司技能 slug。 |
| `--force` | 在服务器允许时，替换同 key 的目录管理技能。此标志不能绕过硬性校验或审计硬停止项。 |

安装后，命令会提醒你技能尚未关联到代理。要让代理使用此技能，请接着执行[关联到代理](#关联到代理期望技能)。

> **注意：** 目录命令只适用于应用内置目录。GitHub、skills.sh、本地路径和普通 URL 来源应通过 `skills import` 导入。

---

## 公司技能库

以下命令管理公司技能库中的技能，不受来源影响。所有命令都需要公司上下文。

```sh
paperclipai skills list --company-id <company-id>
paperclipai skills show <skill-ref> --company-id <company-id>
paperclipai skills file <skill-ref> --path SKILL.md --company-id <company-id>
paperclipai skills import <source> --company-id <company-id>
paperclipai skills create --name "Review PRs" --slug review-prs --body-file SKILL.md --company-id <company-id>
paperclipai skills scan-projects --project-id <project-id> --company-id <company-id>
```

| 命令 | 用途 |
|---|---|
| `list` | 列出公司技能库中的所有技能。 |
| `show <skillRef>` | 显示单项公司技能的完整详情。 |
| `file <skillRef>` | 输出单个技能文件。默认为 `SKILL.md`；其他文件可用 `--path <path>` 指定。人类可读模式会将原始内容写入 stdout，方便通过管道传递。 |
| `import <source>` | 从本地路径、GitHub、skills.sh 或 URL 来源导入技能。会打印导入的技能及任何警告。 |
| `create` | 根据标志和可选 Markdown 正文创建本地管理技能。 |
| `scan-projects` | 扫描项目工作区中的技能，并导入或更新发现的内容。 |

`<skillRef>` 可按公司技能库中的 `id`、规范 `key` 或唯一 `slug` 解析。slug 有歧义时会被拒绝，请改用 `id` 或 `key`。

`skills create` 选项：

| 标志 | 用途 |
|---|---|
| `--name <name>` | 必填。技能名称。 |
| `--slug <slug>` | 可选的显式 slug。 |
| `--description <text>` | 可选描述。 |
| `--body-file <path>` | Markdown 正文文件。使用 `-` 从 stdin 读取正文。 |

`skills scan-projects` 选项（每项都可以重复使用）：

| 标志 | 用途 |
|---|---|
| `--project-id <id>` | 要扫描的项目 ID；可重复传入多个。 |
| `--workspace-id <id>` | 要扫描的工作区 ID；可重复传入多个。 |

不指定 `--project-id` / `--workspace-id` 时，会扫描公司中的所有项目和工作区。摘要行会报告扫描的项目/工作区数量，以及发现、导入、更新、跳过、冲突和警告的数量。

---

## 维护流程：检查、更新、审计和重置

从目录安装的技能会固定到来源。随着目录更新，或有人在本地编辑已安装文件，就需要维护。以下四条命令组成维护流程。`check`、`update` 和 `audit` 中的技能引用是可选参数；省略时会处理整个技能库。

```sh
paperclipai skills check --company-id <company-id>
paperclipai skills update <skill-ref> --company-id <company-id>
paperclipai skills update --all --company-id <company-id>
paperclipai skills audit <skill-ref> --company-id <company-id>
paperclipai skills reset <skill-ref> --yes --company-id <company-id>
```

| 命令 | 用途 |
|---|---|
| `check [skillRef]` | 报告更新状态：`supported`、`hasUpdate`、`currentRef` / `latestRef`、`installedHash` / `originHash`、`updateHoldReason`（若有）以及 `auditVerdict`。 |
| `update [skillRef]` | 为单个技能安装其固定来源中的更新。 |
| `update --all` | 检查所有技能，只更新 `hasUpdate=true` 的技能；跳过支持但已是最新的技能以及不受支持的技能，并说明原因。 |
| `audit [skillRef]` | 重新扫描已安装文件并报告问题（严重性、代码、路径、消息），不会执行任何内容。 |
| `reset <skillRef>` | 从固定来源重新安装目录管理的技能，丢弃本地修改。 |

`update` 和 `reset` 接受 `--force`：

| 标志 | 用途 |
|---|---|
| `--all` | （仅用于 `update`）更新所有有可用更新的技能。 |
| `--force` | 忽略本地修改和软性审计阻止项。审计硬停止项仍会阻止操作。 |
| `--yes` | （仅用于 `reset`）跳过交互式提示并确认；非 TTY 环境中必填。 |

> **警告：** `update` 必须指定技能引用或 `--all`，二者不能同时指定；命令会拒绝这种组合。

---

## 移除技能

```sh
paperclipai skills remove <skill-ref> --yes --company-id <company-id>
```

`skills remove` 会从公司技能库删除技能。交互式终端会要求确认；非交互式运行时必须提供 `--yes`。

---

## 关联到代理：期望技能

技能进入公司库后，`skills agent sync` 可决定你指定的技能如何与代理当前的期望集合合并。必须提供 `--mode`，并从以下三种行为中选择：

- `add` 保留代理已有的所有技能，并合并指定技能（如果某项技能已存在，会原位更新）。
- `remove` 仅移除指定技能，集合中其余内容不变。
- `replace` 将完整期望集合替换为你传入的技能，未包含的技能都会解除关联。这是旧版本的整体替换行为。

之后适配器会协调同步，命令将返回 `AgentSkillSnapshot`。

```sh
paperclipai skills agent list <agent-ref> --company-id <company-id>
paperclipai skills agent sync <agent-ref> --skill review-prs --skill agent-browser --mode add --company-id <company-id>
paperclipai skills agent clear <agent-ref> --yes --company-id <company-id>
```

| 命令 | 用途 |
|---|---|
| `agent list <agentRef>` | 显示代理技能运行时快照：适配器类型、同步是否 `supported`、`mode`、期望数量以及逐技能运行时条目（状态、来源、是否托管）。 |
| `agent sync <agentRef>` | 增补、移除或替换代理期望使用的公司技能，并同步运行时状态。至少需要一个 `--skill` 和一个 `--mode`。 |
| `agent clear <agentRef>` | 清空期望集合（内部使用空列表执行 `replace`）。TTY 环境会提示确认；其他环境必须提供 `--yes`。 |

| 标志 | 用途 |
|---|---|
| `--skill <skillRef>` | （用于 `sync`）期望使用的公司技能 `id`、`key` 或 `slug`。可重复指定多个；至少需要一个。 |
| `--mode <mode>` | （用于 `sync`）**必填。** 合并模式：`add` 保留其他技能，`remove` 仅删除指定技能，`replace` 会破坏性地替换整个集合。 |
| `--yes` | （用于 `clear`）跳过交互式提示并确认。 |

`<agentRef>` 是代理 `id` 或短名称/URL key。使用 `--mode replace` 时，每次都必须列出完整期望集合；未列出的技能会解除关联。`--mode add` 和 `--mode remove` 只操作列出的技能。`clear` 会解除全部关联。

---

## 单数形式的 `skill` 命令组

`paperclipai skill` 是同一公司技能 API 的低级封装。它不会按 `key` / `slug` 解析引用——每个子命令都要传入字面值 `<skillId>`；创建/导入/扫描/文件更新操作则通过原始 JSON 请求体驱动。只有在脚本中已知技能 ID，且需要直接控制请求体时才使用它。

```sh
paperclipai skill list -C <company-id>
paperclipai skill get <skill-id> -C <company-id>
paperclipai skill file <skill-id> --path SKILL.md -C <company-id>
paperclipai skill file:update <skill-id> --payload-json '{"path":"SKILL.md","content":"..."}' -C <company-id>
paperclipai skill update-status <skill-id> -C <company-id>
paperclipai skill install-update <skill-id> -C <company-id>
paperclipai skill delete <skill-id> -C <company-id>
```

| 命令 | 用途 |
|---|---|
| `list` | 列出公司技能（原始 API 输出）。 |
| `get <skillId>` | 按 ID 获取单个技能详情。 |
| `file <skillId>` | 读取技能文件；`--path` 默认为 `SKILL.md`。 |
| `file:update <skillId>` | 更新技能文件。需要 `--payload-json`（`CompanySkillFileUpdate` 请求体）。 |
| `create` | 创建本地技能，需要 `--payload-json`。 |
| `import` | 从来源导入技能，需要 `--payload-json`。 |
| `scan-projects` | 扫描项目工作区中的技能，需要 `--payload-json`。 |
| `update-status <skillId>` | 读取某项技能的更新状态。 |
| `install-update <skillId>` | 为单项技能安装可用更新。 |
| `delete <skillId>` | 按 ID 删除技能。 |

每条 `skill` 子命令都接受 `-C, --company-id <id>`。日常工作建议使用复数形式的 `skills` 命令组，它提供引用解析、确认提示和易读输出，而单数命令组不具备这些便利功能。

---

## 另请参阅

- [代理命令](agent.md)——检查代理并设置本地代理 CLI 会话。
- [适配器命令](adapter.md)——将期望技能同步为磁盘文件的运行时。
- [公司命令](company.md)——公司上下文、导出和导入（公司软件包会携带技能）。
- [项目命令](project.md)——`scan-projects` 扫描的项目和工作区。
- [通用选项](./common-options.md)和[输出与脚本](./output-and-scripting.md)——共享标志及 `--json` 处理方式。
