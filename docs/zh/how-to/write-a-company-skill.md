---
paperclip_version: v2026.1012.0
seo_title: 编写公司技能
seo_description: 编写一个小型 Markdown 包，将其安装到公司技能库中，并将其仅附加到应当加载它的智能体上。大约需要 15 分钟。
---

# 编写公司技能并将其分配给特定智能体

编写一个小型 Markdown 包，使公司中的任何智能体都能按需加载，将其安装到公司技能库中，并将其附加到实际应当使用它的一两个智能体上。在一个全新技能上端到端完成大约需要 10 分钟。

当你希望某个*例程*——例如代码审查清单、发布说明格式、客户回复模板——能够供多个智能体使用，而无需将其硬编码到每个智能体的主要指令中时，技能就是合适的工具。它**不是**用来分发代码的地方（那是[插件](../../administration/plugins.md)），也**不是**运行时桥接（那是[适配器](../../reference/adapters/overview.md)）。如果一个聪明的人类能够按照文件所写的内容操作并产生正确的输出，那么它就是一个技能。

---

## 架构

```txt
   ┌─────────────────────────┐  POST /skills/import   ┌──────────────────────┐
   │ skill folder on disk    │───────────────────────▶│ Company skill library│
   │ (SKILL.md + references) │                        │  (one row per skill) │
   └─────────────────────────┘                        └──────────┬───────────┘
                                                                  │
                                                                  │ POST /agents/{id}/skills/sync
                                                                  ▼
                                                    ┌──────────────────────────┐
                                                    │ Agent's `desiredSkills`  │
                                                    │ resolved on next heartbeat│
                                                    └──────────────────────────┘
```

技能仅存在于公司库中一次。每个智能体的 `desiredSkills` 只是指向该库的引用列表。运行时仅针对已附加该技能的智能体，在支持的适配器上实例化这些文件。

---

## 1. 前置条件

- 在本地或您控制的服务器上运行 Paperclip。参见[安装](../guides/getting-started/installation.md)。
- 已雇佣一个您希望附加该技能的智能体。参见[雇佣您的第一个智能体](../guides/getting-started/your-first-agent.md)。
- 执行这些调用的操作者需要 **`agents:create` 权限**（董事会、公司 CEO 智能体，或具有 `permissions.canCreateAgents=true` 的任何智能体）。技能变更和智能体-技能变更共享此权限门控。
- 一个具有 `curl` 的 shell 和一个文本编辑器。无需特殊的 CLI——这一切都是基于 REST 的。

```bash
export PAPERCLIP_API_URL=http://localhost:3100
export PAPERCLIP_API_KEY=<board-or-ceo-token>
export COMPANY_ID=<your-company-id>
```

---

## 2. 在磁盘上编写技能

技能是一个在其根目录包含 `SKILL.md` 的文件夹。Frontmatter 是智能体首先读取以决定是否加载正文的内容。保持路由描述精准——“在……时使用……不要在……时使用……”——并将长篇幅材料放入 `references/` 子文件夹中。

随服务器捆绑提供的 `paperclip` 技能是该布局的规范示例。您可以在 [GitHub](https://github.com/paperclipai/paperclip/tree/main/skills/paperclip) 上阅读它（`SKILL.md` 加上一个 `references/` 树）。为您的技能镜像该结构。

在本指南中，我们将构建一个小型 `release-note-writer` 技能——为编码智能体提供发布说明格式清单。

```bash
mkdir -p ~/skills/release-note-writer/references
```

编写 `~/skills/release-note-writer/SKILL.md`：

```markdown
---
name: release-note-writer
description: Use when asked to draft release notes, a changelog entry, or a weekly shipped summary from merged PRs or commit subjects. Don't use when writing a blog post, a commit message, or a PR description.
slug: release-note-writer
---

# Release Note Writer

You're drafting release notes from a list of merged pull requests or commit subjects. Follow these rules.

## Output shape

A release note has three sections, in this order:

1. **Highlights** — 1–3 bullets a user actually cares about. Lead each bullet with a verb.
2. **Changes** — every other PR, grouped by area (API, UI, Docs, Internal). Link the PR number.
3. **Fixes** — bug fixes only, one line each.

## Voice

- Past tense ("added", "fixed", "removed"), not future.
- One sentence per bullet. No marketing adjectives.
- Mention numbers when they exist, no claims when they don't.

See `references/example.md` for a known-good output you can pattern-match against.
```

然后在 `references/example.md` 旁边放置一个参考示例，以便智能体有具体的内容可供复制。您放在 `references/` 下的任何内容都会随技能一起捆绑，并在导入父文件夹时作为 `reference` 分类到文件清单中——参见[技能参考 → 支持文件](../../reference/skills.md#supporting-files)。

> **Frontmatter 风格。** Paperclip 使用其自身的窄范围 YAML 读取器解析 frontmatter。请坚持使用扁平标量、嵌套对象和 `- ` 列表项。不要对描述使用 YAML 块标量（`>` 或 `|`）；当前读取器将它们视为字面值，而不是折叠后续行。完整的 frontmatter 模式位于[技能参考 → Frontmatter 字段](../../reference/skills.md#frontmatter-fields)。

---

### 让智能体保存或改进例程

如果智能体使用 Paperclip 的原生 Runner，您可以要求它保存刚刚学到的例程：“将你刚才做的发布说明审查转化为一个公司技能。”在 Auto 模式下，其 `create_skill` 工具会将完整的单文件 `SKILL.md` 保存到库中。您仍然需要将新技能分配给应当使用它的智能体。

对于现有的可编辑技能，直接要求改进：“在我们的发布审查技能中添加一个检查变更日志的步骤。”智能体读取当前版本，然后使用 `update_skill` 将完整的主文件替换为带有版本守卫的版本。如果其他人先保存，智能体必须重新读取并重新应用其编辑；使用相同输入的重试不会创建重复版本。Ask 和预接受 Plan 模式不暴露这些写入工具。

您可以从任务中的 **Skill created** 卡片审查新技能，然后使用 **Open in Skill Studio** 编辑实时库副本。有关确切的输入和重试规则，参见[原生 Runner 技能工具](../../reference/skills.md#native-runner-skill-tools)。实现参考：[工具验证和 API 绑定](https://github.com/paperclipai/paperclip/blob/467125fafb47a8520856504fecc48d6e32055db1/server/src/services/skill-tools.ts)。

---

## 3. 在公司级别安装技能

`POST /api/companies/{companyId}/skills/import` 是唯一的路线。它接受一个 `source` 字符串并处理其余部分。

### 从本地路径

在迭代过程中最快的路径。将 API 指向包含您刚编写的技能文件夹的父文件夹（本例中为 `~/skills`），以便 `release-note-writer/references/` 下的支持文件包含在清单中。

文件夹必须位于 Paperclip 已信任的位置：在您的项目工作区之一内，在项目管理的检出目录内（Paperclip 为您克隆的文件夹），或在公司的管理技能文件夹中。实际上，请将 `~/skills` 保留在项目工作区内——位于这些根目录之外的路径将以 `skill_workspace_boundary_denied` 被拒绝。

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/skills/import" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "source": "/Users/me/skills"
  }'
```

响应包括 `imported[]`（每找到 `SKILL.md` 一条条目）、`warnings[]` 以及解析后的源元数据。每个导入条目都携带服务器选择的规范 `key`——对于本地路径导入，那是 `local/<hash>/<slug>`，例如 `local/8a31d29e1c/release-note-writer`。**复制该键**——您将在下一步中使用它将技能附加到智能体。

> **本地路径是实时的。** 对磁盘上文件的编辑会在技能下次读取时显示；您无需重新导入。这在调整描述时很有用。在将技能提升给真实智能体之前，切换到 GitHub 源（见下文）——参见[版本控制](#7-versioning-and-updates)。

### 从 GitHub 仓库或 skills.sh

一旦技能进入仓库，请优先使用该源，以便将其固定到某个提交，并且您的团队可以通过 PR 审查更改：

```bash
# Whole-repo import — finds every SKILL.md in the repo
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/skills/import" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "source": "https://github.com/acme/agent-skills" }'

# Single-skill import via the tree URL
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/skills/import" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "source": "https://github.com/acme/agent-skills/tree/main/release-note-writer" }'

# skills.sh shorthand — same skill, managed registry
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/skills/import" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "source": "https://skills.sh/acme/agent-skills/release-note-writer" }'
```

每个 `github.com` 导入都记录在 **Skills → Sources** 下作为 GitHub 技能源，因此您稍后可以刷新它，从同一仓库获取新技能，或从一处停止同步。如果您更愿意浏览仓库并手动勾选技能，请在 UI 中使用 **Skills → Sources → Import from GitHub**——参见[从 GitHub 仓库同步技能](../guides/org/skills.md#sync-skills-from-a-github-repository)。

接受的源形式完整列表（npx 命令、gist URL、raw URL、`owner/repo/skill` 简写）位于[公司技能 → 源类型](../../reference/skills.md#import-accepted-sources)。

### 验证安装

```bash
curl "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/skills" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```

找到 `slug` 为 `release-note-writer` 的行。记下其 `key` 和 `id`。两者都是有效的分配句柄。

您还可以在 UI 的 **Skills** 下浏览结果——新行将显示其源徽章（`local`、`github` 或 `skills.sh`）以及您编写的文件清单。

![显示可用技能及其名称和描述的技能页面](../../user-guides/screenshots/light/org/skills-list.png)

---

## 4. 将技能分配给特定智能体

技能在公司范围内存在；*哪些*智能体能看到它们由每个智能体的 `desiredSkills` 控制。同步需要一个必需的 `mode`，说明您命名的技能应如何与智能体的当前集合匹配。若要仅将新技能添加到编码智能体而不触及它已有的其他内容，请使用 `mode: "add"`：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/agents/$CODER_AGENT_ID/skills/sync" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "desiredSkills": [
      "release-note-writer"
    ],
    "mode": "add"
  }'
```

`mode` 决定行为：`add` 保留智能体的其他技能并联合您命名的技能，`remove` 仅丢弃命名的技能，而 `replace` 用您发送的确切内容覆盖完整的期望集合（这是协调行为——不在列表中的任何内容都会被分离）。由于 `add` 和 `remove` 现在可用，您不再需要仅为了更改一个条目而发送完整集合——仅在您确实想要覆盖所有内容时才使用 `replace`。如果您想要 `replace` 但不知道智能体的当前集合，请先读取它：

```bash
curl "$PAPERCLIP_API_URL/api/agents/$CODER_AGENT_ID/skills" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```

响应（一个 `AgentSkillSnapshot`）列出智能体当前拥有的每个条目，带有 `state`（`configured`、`installed`、`available` 等）和 `mode`（运行时同步策略——见下文）。完整模式位于[技能参考 → 将技能分配给智能体](../../reference/skills.md#4-assigning-skills-to-agents)。

每个 `desiredSkills` 条目可以是：

- 一个**规范键**（`local/8a31d29e1c/release-note-writer`、`acme/agent-skills/release-note-writer`）——脚本首选，因为它无歧义，
- 一个技能 **UUID**（来自公司技能列表的 `id`），
- 或一个 **slug**（`release-note-writer`）——适用于一次性调用，但如果 slug 匹配多个技能，将以 `Invalid company skill selection (ambiguous references: …)` 被拒绝。

> **捆绑技能始终附加。** 服务器将任何 `paperclipai/paperclip/*` 捆绑技能（`paperclip` 等）联合到每个智能体的解析集合中，无论您发送什么。因此，您永远不需要列出 `paperclip` 来保留它——即使 `mode: "replace"` 省略了它，它也不会从解析集合中移除。

### 适配器同步模式很重要

快照的 `mode` 字段告诉您适配器实际如何处理分配。有三种：

| 模式 | 行为 |
|---|---|
| `persistent` | 适配器将技能文件写入智能体的工作目录，并在运行之间保留它们。大多数本地适配器（`claude_local`、`codex_local`）使用此模式。 |
| `ephemeral` | 适配器为每次运行实例化文件，并在之后清理。沙箱适配器的默认设置。 |
| `unsupported` | Paperclip 记录分配，但无法将文件推送到运行时。`openclaw_gateway` 属于此类——请在远程运行时内部管理技能。参见[自带智能体](./bring-your-own-agent.md)。 |

如果您分配了技能但智能体从未拾取它，请先检查快照的 `mode`——分配可能仅为跟踪。

---

## 5. 在雇佣时使用 `desiredSkills` 复用

一旦技能进入公司库，您可以在创建或雇佣新智能体的第一天就附加它。相同的 `desiredSkills` 字段在两条路线上都被接受：

```bash
# Direct create (no approval)
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/agents" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Release Note Coder",
    "role": "engineer",
    "adapterType": "claude_local",
    "adapterConfig": { "cwd": "/Users/me/work/acme-api" },
    "desiredSkills": [
      "paperclip",
      "release-note-writer"
    ]
  }'

# Hire request (board approval flow)
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/agent-hires" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Release Note Coder",
    "role": "engineer",
    "adapterType": "claude_local",
    "adapterConfig": { "cwd": "/Users/me/work/acme-api" },
    "desiredSkills": [ "release-note-writer" ]
  }'
```

相同的解析规则适用——传递键、ID 或唯一 slug。未知或歧义的引用将以 `422` 拒绝请求。如果您正在为新公司脚本化入职流程，这就是在单次调用中复现已知智能体设置的方法。当雇佣路线通过审批时，参见[处理雇佣的董事会审批](./handle-board-approvals-for-hires.md)。

---

## 6. 在测试运行中触发技能

现在技能已安装并附加，证明它确实加载。分配给智能体一个描述与技能路由描述匹配的任务：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/issues" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Draft release notes for v0.7.2",
    "description": "Draft release notes for the merged PRs in the v0.7.2 milestone. Use the release-note-writer skill — output the three sections (Highlights, Changes, Fixes) in that order.",
    "assigneeAgentId": "'"$CODER_AGENT_ID"'"
  }'
```

在智能体的下一次心跳时，打开其运行查看器（**Agents → `<agent>` → Runs**）并检查转录。如果技能已加载，您将看到：

- 运行工具/上下文输出中的 `SKILL.md` 正文，或
- 智能体的响应明显遵循技能的章节顺序和语气规则。

如果两者都不成立，请按照[技能参考](../../reference/skills.md#9-troubleshooting-why-a-skill-isnt-loading)中的故障排除清单逐步排查。两个最常见的失败原因：路由描述对智能体来说太模糊无法匹配，或适配器的同步模式是 `unsupported`。

> **收紧描述是关键，而非正文。** 如果智能体在应当加载技能时未加载，正文很少是问题所在。重写 `description` 以更具体地命名触发器（“当用户要求*发布说明*或*变更日志*时使用……”）。只有当技能实际加载后，正文才重要。

---

## 7. 版本控制和更新

行为取决于技能的来源。

| 源 | 固定？ | 如何更新 |
|---|---|---|
| **本地路径**（`/Users/.../release-note-writer`） | 否——实时 | 编辑磁盘上的文件。下次读取时会拾取。仅当您更改源位置时才重新导入。 |
| **Paperclip 管理**（从 UI 的 **+** 按钮创建） | 否——实时 | 在技能页面的 Markdown 编辑器中编辑。 |
| **GitHub** | 是——固定到提交 SHA | `GET /skills/{id}/update-status` 显示跟踪引用上的最新提交。`POST /skills/{id}/install-update` 刷新其源。您也可以使用 **Skills → Sources** 上的 **Refresh**。 |
| **skills.sh** | 是——底层解析为 GitHub | 与 GitHub 相同。 |
| **URL / 原始文件** | 否 | 重新导入 URL 以刷新。 |
| **随 Paperclip 捆绑** | 固定到 Paperclip 发布 | 升级 Paperclip 本身；捆绑技能无法编辑。 |

您的智能体也可以帮助保持 Paperclip 管理技能的当前状态：在原生 runner 上，Auto 模式下的智能体可以使用其 **Update skill** 工具重写技能的 `SKILL.md`，并受保护以免覆盖较新的保存——参见[让智能体改进现有技能](../guides/org/skills.md#letting-an-agent-improve-an-existing-skill)。

一旦您希望跨多个智能体或公司共享技能，请立即将其提升为 GitHub 源——这是提供可审查差异、稳定规范键和更新状态跟踪的路径。当您 `install-update` 时，`id` 不会更改，因此任何通过 `id` 引用该技能的内容都无需重新同步即可继续工作。

要检查 GitHub 源技能的更新：

```bash
curl "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/skills/$SKILL_ID/update-status" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```

响应（缩写）：

```json
{
  "supported": true,
  "trackingRef": "main",
  "currentRef": "9f2a3b1c4d8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a",
  "latestRef": "0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b",
  "hasUpdate": true
}
```

如果 `hasUpdate` 是 `true`：

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$COMPANY_ID/skills/$SKILL_ID/install-update" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```

本地路径技能不固定，因此不暴露 `update-status`——响应为 `{ "supported": false, "reason": "..." }`。请就地编辑文件。

---

## 8. 需牢记的作用域规则

- **公司作用域，而非组织作用域。** 安装在公司 A 中的技能对公司 B 不可见。要共享，请将源安装两次——每个公司一次。
- **无按团队或按角色的门控。** 粒度是每个智能体的 `desiredSkills`。同一 `cwd` 上的两个智能体可以拥有完全不同的技能集合。
- **捆绑技能强制开启。** `paperclipai/paperclip/*` 技能联合到每个解析集合中；您无法移除它们。
- **适配器同步模式是运行时门控。** 如果 `mode: "unsupported"`，分配存在但没有文件到达运行时。要么切换适配器，要么在远程运行时管理技能。

完整的规则集——必需与可选、捆绑必需、实例化策略、冲突解决——位于[技能参考 → 作用域规则](../../reference/skills.md#5-scoping-rules)。

---

## 9. 故障排除

**`Missing permission: can create agents`。**
该路线需要 `agents:create` 权限。以董事会用户、CEO 智能体或具有 `permissions.canCreateAgents=true` 的智能体身份运行调用。相同的门控适用于导入、扫描、同步以及每技能详情/文件路线。

**`Local skill source is outside approved company workspace roots`。**
本地路径不在项目工作区、项目管理的检出目录或公司的管理技能文件夹内（代码 `skill_workspace_boundary_denied`）。将技能文件夹移动到项目工作区——或注册其文件夹为工作区——然后重新导入。

**`Invalid company skill selection (ambiguous references: <slug>; …)`。**
两个已安装的技能共享相同的 slug。将调用切换为使用规范 `key`——列出 `/companies/{id}/skills`，找到该行，复制 `key` 字段，发送该字段。

**`Invalid company skill selection (unknown references: …)`。**
`desiredSkills` 中的引用不匹配任何已安装的技能。最常见的是拼写错误或来自不同公司的 slug。先列出技能。

**技能导入但在运行时从未加载。**
检查快照：`GET /api/agents/{agentId}/skills`。如果 `mode` 是 `unsupported`，适配器（例如 `openclaw_gateway`）无法同步；分配仅为元数据。如果 `mode` 是 `ephemeral`，文件仅在运行期间存在——工作区在静止时看起来是空的。这是正确行为。

**Frontmatter 更改未生效。**
Paperclip 的 YAML 读取器故意窄范围。使用空格（而非制表符）、`- ` 用于列表项，以及 `name` 和 `description` 的纯单行字符串值。避免使用块标量（`>` 和 `|`），因为当前解析器不折叠它们。如果您将其简化为支持的子集后值仍然解析不正确，请提交 bug；内部解析器记录在[技能参考 → Frontmatter 字段](../../reference/skills.md#frontmatter-fields)。

**`references/example.md` 在导入的文件清单中缺失。**
从包含技能目录的父文件夹重新导入（本指南中为 `/Users/me/skills`），而不是 `release-note-writer` 文件夹本身。父文件夹导入会一起遍历 `release-note-writer/SKILL.md` 以及相邻的 `references/`、`scripts/` 和 `assets/` 文件夹。

**删除后捆绑技能不断重新出现。**
这是设计使然——`paperclip` 和其他 `paperclipai/paperclip/*` 技能在每次列表调用时重新导入。要抑制其中一个，请运行不附带它的分叉 Paperclip 构建。

对于更深入的调试——错误的工具列表、同步模式混淆、GitHub 固定卡在旧提交——请遍历[完整故障排除列表](../../reference/skills.md#9-troubleshooting-why-a-skill-isnt-loading)。

---

## 另请参阅

- [技能指南](../guides/org/skills.md) — UI 演练、文件清单、编写良好技能的清单、信任级别。
- [技能参考](../../reference/skills.md) — 文件形状、安装管道、规范键、作用域、版本控制。
- [智能体 API → 技能](../../reference/api/agents.md#skills) — `GET` 和 `POST /agents/{id}/skills/sync` 的请求/响应形状。
- [自带智能体](./bring-your-own-agent.md) — OpenClaw / HTTP webhook / 自定义脚本路径，包括 `unsupported` 技能同步注意事项。
- [处理雇佣的董事会审批](./handle-board-approvals-for-hires.md) — 当 `desiredSkills` 进入审批工作流而非直接创建时。
- [为智能体添加 MCP 服务器](./add-mcp-server-to-agent.md) — 不适合技能的*可执行*工具的运行时替代方案。
