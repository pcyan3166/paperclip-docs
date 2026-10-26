---
paperclip_version: v2026.831.1
seo_title: 导出和导入公司
seo_description: 将已配置的公司打包，以便备份、分享或迁移到其他实例。了解完整保真度包含哪些内容，以及包内文件的结构。
---

# 导出和导入

创建公司、设定目标、聘用智能体、配置适配器并设置项目后，这套配置就具有实际价值。通过导出和导入，你可以把配置保存为可移植的包，用来备份、分享、迁移到另一个 Paperclip 实例，或作为新公司的起点。

导出内容是可供人阅读的 Markdown 文件。任何拿到软件包的人都可以了解公司的结构，不必解析数据库转储。

> **要在不同实例间迁移公司？** 请使用导出和导入：在一个实例上导出公司，再在另一个实例导入软件包。这是完整保留数据的迁移方式，用来替代已退役的实例间 Cloud Sync。现在无需将两个实例互相连接，就能迁移工作。

---

## “完整保真度”是什么意思

软件包不只是公司结构的草图；它还包含实际工作状态。恢复或迁移后，你会看到熟悉的公司，而不是从头重建。

**软件包会包含：**

- 公司名称、描述、徽标和招聘策略
- 智能体信息：身份、角色、职位、汇报关系、指令、适配器和运行环境配置、权限及权限授予、单个智能体预算
- 项目、关联工作区和侧边栏顺序
- 技能（作为上游引用，或将完整内容一并打包）
- 任务及其评论、状态、优先级、标签、阻塞关系、文档、工作成果和监控器
- 例程及其计划触发器
- 附件和图片（按内容寻址保存，导入时重新附加）
- 每位智能体所需环境变量的名称

**永远不会打包：**机密值，包括 API 密钥、令牌和密码；特定机器的文件路径；内部数据库 ID。这些信息与环境绑定，在另一台机器上通常无效。软件包会声明智能体需要哪些环境变量，由你在目标实例上填写实际值。

**特意不包含：**审批记录、费用历史和活动日志条目。这些是源实例上发生过的事件记录，而非配置，因此仍保留在源实例中。如果公司有这些记录，导出页面会在下载前提示具体类型和数量。

---

## 软件包包含哪些内容

导出的公司软件包目录如下：

```
my-company/
├── README.md           ← A human-readable summary of the package
├── COMPANY.md          ← Company name, goal, and metadata
├── agents/
│   ├── ceo/AGENTS.md   ← Agent identity, role, and instructions
│   └── cto/AGENTS.md
├── projects/
│   └── main/PROJECT.md
├── skills/
│   └── review/SKILL.md
├── tasks/
│   └── onboarding/
│       ├── TASK.md
│       └── documents/brief.md
├── images/             ← Company logo and inline images
├── blobs/              ← Attachment bytes, named by content hash
└── .paperclip.yaml     ← Adapter types, environment variable declarations, budgets
```

除了 `blobs/` 中的内容，其余文件都是文本，因此可在 Git 中清晰比较，也能用任何编辑器阅读。

---

## 导出公司

现在可以通过 Paperclip 界面导入和导出。如果你更喜欢使用终端，也可以继续使用 CLI 命令，但它已不再是唯一方式。

### 在界面中操作

打开 **Company Settings（公司设置）**，找到 **Company Packages（公司软件包）** 部分。其中有 **Export（导出）** 和 **Import（导入）** 两个按钮。

导出页面提供少量开关，用来选择要包含的内容：**Agents（智能体）**、**Projects（项目）**、**Skills（技能）**、**Routines（例程）**、**Tasks（任务）** 和 **Attachments（附件）**。默认全部选中。附件会随引用它们的任务和例程一起打包；只有仍选中任务或例程时，附件开关才适用。

下载前，Paperclip 会显示软件包内容；你还可以取消选择单个文件，进一步缩小软件包。确认后，系统会下载一个以软件包名称命名的 `.zip` 文件。

导入时，Paperclip 会预览将要创建、重命名或跳过的内容，然后才应用导入。

如果公司有无法打包的数据（审批、费用事件或活动日志条目），导出页面会显示 **Not included in this export（本次导出不包含）** 面板，准确列出每类数据留下的数量。这不是错误提示，而是为了避免你几个月后恢复时感到意外。

### 在终端中操作

将公司导出到文件夹：

```sh
paperclipai company export <company-id> --out ./my-export
```

将 `<company-id>` 替换为公司的 ID（在 Paperclip 中查看公司时，可从 URL 中找到）。

默认情况下，命令会导出公司元数据和智能体。要包含更多内容：

```sh
# Export everything: company, agents, projects, skills, and tasks
paperclipai company export <company-id> --out ./full-export \
  --include company,agents,projects,tasks,skills
```

### 导出选项

| 选项 | 用途 |
|---|---|
| `--out <path>` | 导出文件的保存位置（必填） |
| `--include <values>` | 要包含的内容：`company`、`agents`、`projects`、`issues`、`tasks`、`skills`，以逗号分隔 |
| `--skills <values>` | 仅导出指定名称的技能 |
| `--projects <values>` | 仅导出指定项目 |
| `--issues <values>` | 仅导出指定任务 |
| `--project-issues <values>` | 导出指定项目内的所有任务 |
| `--expand-referenced-skills` | 将技能内容复制到软件包中，而非仅指向上游来源 |
| `--force` | 不经确认，覆盖已有文件的目录 |

> **建议：** 定期导出作为备份，尤其在大幅修改智能体配置或组织结构之前。若自动执行夜间导出并重复写入同一个目录，可以使用 `--force`。

---

## 导入公司

导入来源可以是本地路径、URL 或 GitHub：

```sh
# From a local folder
paperclipai company import ./my-export

# From a local .zip package
paperclipai company import ./my-export.zip

# From a GitHub repository URL
paperclipai company import https://github.com/org/repo

# From a GitHub subfolder, using owner/repo shorthand
paperclipai company import org/repo/companies/acme
```

对于 GitHub 来源，可以使用 `--ref` 固定到特定版本；参数可以是分支、标签或提交：

```sh
paperclipai company import https://github.com/org/repo --ref v1.2.0
```

`--ref` 仅适用于 GitHub 来源；若用于本地路径，命令会停止并提示错误。URL 必须指向 GitHub 或 GitHub Enterprise；普通 HTTP 链接如果指向其他主机，将被拒绝，不会尝试获取。

> **说明：** 在界面中导入本地软件包时，请选择由 Paperclip 导出的 `.zip` 文件。使用 Finder 或 Explorer 重新压缩的归档文件可能无法导入。CLI 则可以接受 `.zip` 文件或解压后的文件夹。

### 根据软件包创建新公司

如果导入时未指定已有公司，Paperclip 会创建一家新公司：

```sh
paperclipai company import ./my-export --target new --new-company-name "My Restored Company"
```

### 合并到已有公司

要将软件包中的智能体或项目添加到已在运行的公司中：

```sh
paperclipai company import ./shared-agents \
  --target existing \
  --company-id <your-company-id> \
  --include agents
```

### 应用前预览

使用 CLI 时，请始终先预览导入，尤其是合并到现有公司时：

```sh
paperclipai company import ./my-export --target new --dry-run
```

预览会准确显示将创建、重命名、跳过或替换的内容，但不会执行任何更改。请在继续前仔细检查。

### 处理名称冲突

导入到现有公司时，智能体或项目名称可能与现有名称冲突。Paperclip 提供三种处理方式：

| 策略 | 处理方式 |
|---|---|
| `rename`（默认） | 添加后缀以避免冲突，例如将 `ceo` 改为 `ceo-2` |
| `skip` | 不改动现有实体，并跳过可能冲突的内容 |
| `replace` | 覆盖现有实体；请谨慎使用 |

> **注意：** `replace` 冲突策略会覆盖已有智能体配置。使用前请先导出备份。

### 导入大型公司

任务和附件积累多年的活跃公司会生成较大的软件包，Paperclip 会负责处理。界面会将软件包作为压缩的 `.zip` 文件上传，而不是先在浏览器中解压；服务器则在后台运行导入，页面会显示进度。因此，即使网络较慢或代理中断连接，也不会导致导入丢失。

导入过程有两道保护。如果上传内容在传输中缺失，导入会拒绝执行并要求重试，而不会只导入一部分。写入任何记录之前，Paperclip 都会用内容哈希校验软件包中的每个附件，避免损坏或被篡改的软件包造成不完整导入。

大型软件包也不必依赖一次性、不稳定的上传。软件包足够大时，Paperclip 会把 `.zip` 切分成多个部分，逐个上传，并显示类似 `Uploading part X of Y — N MB of M MB uploaded.` 的进度。无论从界面导入还是运行 `paperclipai company import`，这个过程都会自动进行，无需设置额外参数。如果上传因刷新页面、断开网络、某个分片失败或服务器重启而中断，只需重试或刷新同一导入页面。Paperclip 会继续同一传输，并只重新发送缺失部分，无需从头开始。传输完成后，该软件包即视为已导入；再次导入同一包时会提示 `This exact package was already imported by a completed transfer. Re-export the package to import it again.`。如果再次导入看似没有效果，原因就在这里；请重新导出公司以导入最新副本。

服务器默认接受最大 **1 GB** 的导入软件包。若导出文件更大，管理员可以通过 `PAPERCLIP_IMPORT_ZIP_MAX_BYTES` 环境变量提高上限（单位为字节，最高 64 GiB），详见[环境变量](../../reference/deploy/environment-variables.md)。

---

## 常见用途

**备份公司配置**

定期完整导出，并保存到安全的位置，例如云盘或私有 Git 仓库。发生问题时，可以从软件包恢复。包含计划夜间导出的完整备份与恢复方法，见[备份和恢复公司](../../how-to/back-up-and-restore-a-company.md)。

**将公司迁移到另一个实例**

在即将离开的实例上导出，再在目标实例导入；可以从笔记本迁到服务器、从自托管实例迁到 Paperclip Cloud，也可以反向迁移。软件包除了配置，还包含任务、例程和附件，因此公司能大致从上次停下的位置继续工作。实例间 Cloud Sync 已退役，现在应使用这一受支持的方式。

**从模板创建新公司**

将配置良好的公司导出为模板，以后需要创建类似公司时，用新名称导入。智能体配置、技能和项目结构都会一并迁移。

**分享智能体团队**

如果你创建了一支配置良好的团队（例如由 CEO、CTO 和工程师组成的标准工程团队），可以只导出智能体并分享软件包。其他人可以将其导入自己的公司。

```sh
# Share: export agents only
paperclipai company export <company-id> --out ./engineering-team --include agents

# Receive: import into a new company
paperclipai company import org/shared-templates/engineering-team \
  --target new \
  --new-company-name "My Engineering Team"
```

**从 GitHub 导入**

社区发布的公司模板位于公开 GitHub 仓库中。可直接导入：

```sh
paperclipai company import org/company-templates/research-team \
  --target new \
  --dry-run
```

检查预演输出后，再移除 `--dry-run` 重新运行以应用更改。

---

## 导入后

导入的智能体默认不会启用定时心跳。这是有意为之：这样你可以先检查配置，并在智能体开始产生费用前设置自己的预算和心跳计划。

界面还提供了额外保护：导入页面默认勾选 **Start imported agents and routines paused（以暂停状态启动导入的智能体和例程）**。导入完成后，会显示 **Activate imported agents and routines（启用已导入的智能体和例程）** 列表。勾选准备好唤醒的对象即可从此处启用；在你确认之前，它们不会运行。

导入后请：

1. 打开每位导入的智能体，确认适配器配置正确。
2. 根据你的使用量设置单个智能体预算。
3. 添加软件包已声明但未包含实际值的 API 密钥或环境变量。
4. 准备好开始工作时，启用心跳，并启动你暂停的智能体和例程。

---

## 已完成

导出和导入让你创建的配置可以长期保存、分享和备份。最后一篇指南是终端设置，适合希望更深入控制 Paperclip 运行方式的开发者。

[终端设置 →](./terminal-setup.md)
