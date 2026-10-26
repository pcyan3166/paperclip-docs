---
paperclip_version: v2026.609.0
seo_title: CLI 团队命令
seo_description: 从 Paperclip 随应用提供的目录中安装现成的入门团队，包括智能体、项目、任务及其所需技能。
---

# 团队命令

目录中的一个*团队*是现成的入门工作组，由少量智能体、项目、任务及所需技能组成，并随 Paperclip 应用一同发布。使用 `paperclipai teams` 可以浏览团队模板、检查安装内容、针对实际公司预览导入计划，然后安装团队。它可以看作技能目录的“团队版”：技能目录提供一项可复用流程，而这里可以一步将完整的工作单元添加到公司。

目录团队有两种类型。**内置**团队随应用发布，是日常使用的起点（例如核心高管团队、产品工程小组和产品设计团队）。**可选**团队则用于有额外需求时，其中一部分会从外部来源引入技能。浏览命令中的 `--kind` 筛选器可在两类团队之间切换。

> **提示：** 在任意命令后添加 `--json`，即可输出原始 API 结果以供脚本处理。浏览命令不需要公司上下文；但 `list`、`preview` 和 `install` 会针对某家公司执行，因此需要接受 `--company-id <id>`，并遵循当前选中的配置文件。详见[通用选项](./common-options.md)。

---

## 安装前浏览

浏览命令只读取目录，不会向公司写入任何内容。浏览、搜索或查看详情时不需要公司上下文。

```sh
paperclipai teams browse --kind bundled --category company-defaults
paperclipai teams search "engineering" --kind bundled
paperclipai teams inspect core-exec-team
```

| 命令 | 用途 |
|---|---|
| `browse` | 列出目录团队，可按 `--kind`、`--category` 或 `--query` 筛选。 |
| `search <query>` | 与 `browse` 相同，但将搜索文本作为位置参数传入。 |
| `inspect <catalogRef>` | 完整显示一个目录团队，包括智能体、项目、任务、必需技能和文件清单。 |

浏览和搜索筛选器：

| 标志 | 用途 |
|---|---|
| `--kind <kind>` | 按目录类型筛选：`bundled` 或 `optional`。 |
| `--category <slug>` | 按目录类别 slug 筛选。 |
| `--query <text>` | 全文搜索（仅 `browse` 支持；`search` 使用位置参数）。 |

`<catalogRef>` 可以是目录团队 ID、键或唯一 slug。安装前先检查团队，是准确了解将要导入哪些智能体和项目的最佳方式。

`inspect` 也可以打印团队包中的单个文件，而不是清单详情：

| 标志 | 用途 |
|---|---|
| `--file <path>` | 打印指定目录团队文件（例如 `TEAM.md`），而不是清单详情。在人类可读模式下，原始内容会写入标准输出，供管道处理。 |

---

## 查看安装状态

`teams list` 是带有公司视角的 `browse`：它会列出每个目录团队，标明该团队是否已安装到指定公司，以及已安装副本是否偏离当前目录版本。此命令需要公司上下文。

```sh
paperclipai teams list --company-id <company-id>
paperclipai teams list --kind optional --company-id <company-id>
```

| 标志 | 用途 |
|---|---|
| `--kind <kind>` | 按目录类型筛选：`bundled` 或 `optional`。 |
| `--category <slug>` | 按目录类别 slug 筛选。 |
| `--query <text>` | 全文搜索。 |

每行都会报告团队的安装状态（未安装、已安装、版本过期或安装后缺失），以及该团队包含的智能体和项目数量。如果已安装智能体记录的来源与目录团队当前内容不一致，该团队会被标记为过期，提示你重新安装较新版本。

---

## 预览安装

提交安装前，`teams preview` 会在服务器上试运行导入并显示计划，但不会写入内容。它会报告将要创建的对象、团队需要哪些技能以及这些技能将如何准备，并列出警告和错误。此命令需要公司上下文。

```sh
paperclipai teams preview core-exec-team --company-id <company-id>
paperclipai teams preview product-engineering \
  --target-manager-slug cto \
  --collision-strategy rename \
  --company-id <company-id>
```

| 标志 | 用途 |
|---|---|
| `--target-manager-agent-id <id>` | 指定现有智能体 ID，作为目录团队根智能体的汇报对象。 |
| `--target-manager-slug <slug>` | 指定可移植的管理者 slug，作为目录团队根智能体的汇报对象。 |
| `--agent <slug>` | 只预览指定 slug 的智能体。可重复传入多个。 |
| `--collision-strategy <strategy>` | 处理现有同名对象的方式：`rename`、`skip` 或 `replace`。 |
| `--name-override <slug=name>` | 覆盖导入实体的名称。可重复传入多个。 |
| `--selected-file <path>` | 将预览范围限制为指定的可移植文件。可重复传入多个。 |
| `--allow-external-sources` | 允许使用目录团队声明的 GitHub、URL 或 skills.sh 技能来源。 |
| `--allow-unpinned-optional-sources` | 允许可选团队使用未固定到某个提交的外部技能来源。 |
| `--allow-local-path-sources` | 仅用于开发：允许使用目录团队声明的本地路径技能来源。 |

两个管理者选项可将团队接入现有组织架构：团队根智能体不再独立工作，而是向已有智能体汇报。如果知道准确的智能体 ID，请使用 `--target-manager-agent-id`；如果需要可移植 slug，请使用 `--target-manager-slug`。

---

## 安装团队

`teams install` 会执行 `preview` 所显示的操作：将团队导入公司，创建相应的智能体、项目和任务，并准备所需技能。此命令需要公司上下文，接受 `preview` 的所有选项，并额外支持少数仅用于安装的标志。

```sh
paperclipai teams install core-exec-team --company-id <company-id>

paperclipai teams install product-engineering \
  --target-manager-slug cto \
  --adapter-override senior-coder=claude_local \
  --company-id <company-id>
```

`install` 支持 `preview` 的所有标志（`--target-manager-agent-id`、`--target-manager-slug`、`--agent`、`--collision-strategy`、`--name-override`、`--selected-file` 和三个 `--allow-*` 来源标志），并额外支持：

| 标志 | 用途 |
|---|---|
| `--secret-value <key=value>` | 为团队声明的密钥环境变量输入提供值。可重复传入多个。 |
| `--adapter-override <slug=type>` | 覆盖导入智能体 slug 所用的适配器类型，例如 `senior-coder=claude_local`。可重复传入多个。 |
| `--request-approval-on-forbidden` | 当因缺少 `agents:create` 权限而被拒绝安装时，创建控制面板审批请求，而不是直接以原始 403 错误退出。 |
| `--approval-issue-id <id>` | 关联到后备审批请求的任务 ID。若已设置 `PAPERCLIP_TASK_ID`，则默认使用其值。 |

目录团队刻意不为每个智能体指定固定适配器，因此安装时可选择运行时。使用 `--adapter-override <slug=type>` 可将特定智能体固定到 `claude_local` 或 `codex_local` 等适配器；未指定覆盖的智能体会在导入时解析适配器。

### 无权创建智能体时

安装团队需要创建智能体，因此需要 `agents:create` 权限。如果服务器因此拒绝安装，通常会返回 `403`。传入 `--request-approval-on-forbidden`（或在已设置 `PAPERCLIP_TASK_ID` 的 Paperclip 任务中运行）后，CLI 会改为提交一项控制面板审批请求，说明你尝试安装的内容，供控制面板操作员批准。你传入的任何密钥值都会从保存的请求中移除，并在返回结果中遮蔽。

---

## 另请参阅

- [团队目录指南](../../guides/org/team-catalog.md)——目录团队的概念，以及如何在应用中使用它们。
- [技能命令](skills.md)——最接近单一能力目录的技能目录。
- [公司命令](company.md)——公司上下文、导出和导入（目录安装使用相同的可移植性引擎）。
- [智能体命令](agent.md)——检查安装所创建的智能体。
- [审批命令](approval.md)——处理权限不足时的后备控制面板审批请求。
- [通用选项](./common-options.md)和[输出和脚本](./output-and-scripting.md)——共享标志和 `--json` 处理方式。
