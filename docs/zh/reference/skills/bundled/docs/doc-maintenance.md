---
paperclip_version: v2026.618.0
seo_title: 文档维护技能
seo_description: 以尽量少的改动保持文档准确：发现文档与近期行为变化的偏差，更新受影响页面，并记录已发布内容。
---

# 文档维护

> 使项目文档与近期代码和功能变更保持一致：发现偏差、更新受影响页面，并补充与版本发布相关的说明；不要改写未发生变化的章节。

以尽量少的改动保持文档准确：找出文档与近期行为变化不一致之处，更新受影响页面，并补充与版本发布相关的说明，不要改写未受影响的章节。

此技能是**内置**目录技能，属于内置基础套件。如何安装、审计、更新、分配和重置目录技能，请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

## 适用场景

- 拉取请求或一系列近期合并改动了用户可见的行为，例如 CLI 标志、API 结构、默认值、配置键、端点、环境变量或支持的版本。
- 用户报告的问题最终追溯到文档过时。
- 即将发布新版本，需要根据已合并的提交检查文档。
- 新功能已经发布，但只有工程师的拉取请求说明介绍了使用方法。

### 不适用场景

- 变更仅影响内部实现（私有辅助函数重命名、重构），对用户没有可见影响。
- 你只想“改善文档”，却没有对应的行为变更。这属于另一个有明确范围的项目，应先制定计划。

## 目录元数据

| 字段 | 值 |
|---|---|
| 目录 ID | `paperclipai:bundled:docs:doc-maintenance` |
| 规范键 | `paperclipai/bundled/docs/doc-maintenance` |
| 目录路径 | `catalog/bundled/docs/doc-maintenance` |
| 类型 | `bundled` |
| 类别 | `docs` |
| Slug | `doc-maintenance` |
| 入口文件 | `SKILL.md` |
| 信任级别 | `markdown_only` |
| 兼容性 | `compatible` |
| 默认安装 | `false` |
| 建议角色 | `engineer`、`product`、`devrel` |
| 依赖项 | — |
| 标签 | `docs`、`documentation`、`release-notes` |
| 文件数 | 1 |
| 内容哈希 | `sha256:2e02299210fd17c1fe1867b4ee8c144a11b6fe1fe481f83b8268cfbaaf10f9aa` |
| 软件包 | `@paperclipai/skills-catalog@0.3.1` |

## 文件清单

| 路径 | 类型 | 字节数 |
|---|---|---:|
| `SKILL.md` | `skill` | 4,478 |

## 完整技能定义

下方代码块是此技能完整且权威的 `SKILL.md`，也是智能体运行时实际加载的文件。可以使用代码块右上角的控件进行复制，或将其下载为 `SKILL.md`。

````markdown skill-source
---
name: doc-maintenance
description: Keep project docs aligned with recent code and feature changes — detect drift, update affected pages, and add release-relevant notes without rewriting unchanged sections.
key: paperclipai/bundled/docs/doc-maintenance
recommendedForRoles:
  - engineer
  - product
  - devrel
tags:
  - docs
  - documentation
  - release-notes
---

# Doc Maintenance

Keep the documentation honest with minimum churn. The goal is alignment between docs and behavior, not stylistic rewrites or cosmetic re-organization. Reviewers should be able to read a diff and see "this updates docs to match recent behavior changes".

## When to use

- A PR or recent set of merges changed user-visible behavior: CLI flags, API shapes, default values, configuration keys, endpoints, environment variables, supported versions.
- A user-reported bug traced back to outdated documentation.
- A release is being cut and the docs need a pass against the merged commits.
- A new feature shipped but only the engineer's PR description describes how to use it.

## When not to use

- The change is internal-only (private helper rename, refactor) with no user-visible impact.
- You want to "improve the docs" without a behavior anchor. That is a separate scoped project, not maintenance — make a plan first.

## The pass

1. **Establish the baseline.** Get the commit range you are documenting against (since last release tag, since last merged-doc commit, or since a specific PR).
2. **Enumerate user-visible changes.** Read commits and PR descriptions. List, for each change, what a user can now do differently.
3. **Map changes to docs.** For each change, find every page that mentions the affected concept. Common targets: README, CLI reference, API reference, configuration reference, migration guide, FAQ, examples.
4. **Update precisely.** Edit only the lines that need to change. Do not rewrap paragraphs you did not modify — it pollutes the diff.
5. **Add new entries where needed.** New CLI flag → CLI reference entry. New env var → configuration reference entry. New endpoint → API reference entry. Don't only add it to the changelog.
6. **Update examples and snippets.** Code blocks in docs are wrong faster than prose. Re-run any example that touches new behavior.
7. **Write the release note.** One sentence per user-visible change. Group by Added / Changed / Fixed / Deprecated / Removed. Link to the relevant PRs and docs section.
8. **Cross-check.** Search the docs for the old behavior wording and remove or update stragglers.

## Style baseline

- Voice: second person ("you can pass `--json` to ..."). Avoid "we" except in narrative pages.
- Tense: present, not future. The behavior exists once shipped.
- Headings: imperative ("Configure the cache") or noun-phrase ("Cache configuration"), match the surrounding page.
- Code blocks: include the language tag so syntax highlighting works.
- Cross-links: link the first mention of a concept on each page; do not link every occurrence.
- Avoid promising future behavior. If something is unreleased, mark it `experimental` or omit it.

## Drift detection

A doc page is drifting if any of these are true:

- It documents a flag, key, or endpoint that no longer exists.
- An example does not run as written.
- A default value in the docs does not match the code.
- A supported-versions list excludes a version the project actually supports, or includes one it dropped.
- A "Coming soon" section references a feature that shipped or was cancelled.

When you find drift, fix it in the same pass and note it in the release note's `Fixed` group.

## Release-note rules

- One sentence per item. If two sentences are needed, the item is likely two items.
- User impact first, internal cause second. `Faster cold start (avoid full bundle download on first run)` beats `Refactor bootstrap loader`.
- Link the PR for engineering readers and the docs page for users.
- Mark breaking changes explicitly: `**Breaking:**` prefix. Include migration steps inline or via link.

## Anti-patterns

- Massive doc PRs that bundle stylistic rewrites with real updates. Reviewers cannot tell which lines reflect actual behavior changes.
- "Updated docs" commit messages with no detail. Make the commit say what changed and why.
- Adding to the changelog without updating the reference docs the changelog points to.
- Marking a feature as available before its code lands. Documentation must follow behavior, not promise it.
````

## 中文译文

# 文档维护

以尽量少的改动确保文档准确。目标是让文档与实际行为一致，而不是改写风格或做表面重组。审核者应能从差异中看出：“这些改动是为了让文档符合近期的行为变更。”

## 适用场景

- 拉取请求或近期的一组合并变更了用户可见行为，例如 CLI 标志、API 结构、默认值、配置键、端点、环境变量或支持版本。
- 用户报告的问题最终被查明是文档过时所致。
- 即将发布版本，需要根据已合并的提交检查文档。
- 新功能已发布，但只有工程师的拉取请求说明写了使用方法。

## 不适用场景

- 变更仅涉及内部实现（例如私有辅助函数改名或重构），不会影响用户。
- 想在没有行为变更依据的情况下“改进文档”。这属于单独的范围明确的项目，不是维护工作；应先制定计划。

## 维护步骤

1. **确定基线。** 明确本次文档对应的提交范围（例如自上个版本标签、自上次文档提交，或某个指定拉取请求以来）。
2. **列出用户可见的变更。** 阅读提交和拉取请求说明，逐项列明用户现在可以做哪些不同的事。
3. **将变更映射到文档。** 对每项变更，找出所有提及相关概念的页面。常见目标包括 README、CLI/API/配置参考、迁移指南、常见问题和示例。
4. **精确更新。** 只编辑需要变更的行。不要重新换行未修改的段落，以免让差异变得杂乱。
5. **按需添加条目。** 新 CLI 标志应加入 CLI 参考；新环境变量应加入配置参考；新端点应加入 API 参考。不要只写入变更日志。
6. **更新示例和代码片段。** 文档中的代码块比说明文字更容易过时。重新运行涉及新行为的示例。
7. **撰写版本说明。** 每项用户可见变更用一句话说明，并归入 Added / Changed / Fixed / Deprecated / Removed。链接到相关拉取请求和文档章节。
8. **交叉检查。** 搜索文档中描述旧行为的文字，删除或更新遗漏之处。

## 基本写作风格

- 人称：使用第二人称（例如“你可以向……传入 `--json`”）。叙述性页面除外，避免使用“我们”。
- 时态：使用现在时，而不是将来时。功能发布后，行为已经存在。
- 标题：使用祈使句（“配置缓存”）或名词短语（“缓存配置”），并与所在页面风格一致。
- 代码块：标注语言，以启用语法高亮。
- 交叉链接：每页首次提到某个概念时添加链接，不要链接每一次出现。
- 不要承诺未来行为。尚未发布的内容应标为 `experimental`，或从文档中省略。

## 检查文档漂移

出现以下任一情况，即说明页面可能已与产品行为脱节：

- 文档提及的标志、键或端点已不存在。
- 示例按文中写法无法运行。
- 文档中的默认值与代码不一致。
- 支持版本列表遗漏了项目实际支持的版本，或包含已停止支持的版本。
- “即将推出”章节提到的功能已经发布或已取消。

发现漂移时，应在同一轮维护中修复，并在版本说明的 `Fixed` 类别中记录。

## 版本说明规则

- 每个条目只写一句话。若需要两句，该条目可能实际包含两项变更。
- 先说明用户影响，再说明内部原因。`更快的冷启动（首次运行时避免下载完整软件包）` 优于 `重构启动加载器`。
- 面向工程读者的版本说明链接到拉取请求，面向用户的说明链接到相关文档页面。
- 明确标注破坏性变更：使用 `**Breaking:**` 前缀，并在条目中或链接中提供迁移步骤。

## 反面模式

- 大型文档拉取请求把风格重写和真实更新混在一起，审核者无法分辨哪些行反映实际行为变化。
- 提交说明只写“更新文档”，没有说明改了什么以及原因。
- 只更新变更日志，却不更新该日志指向的参考文档。
- 在代码尚未合并前就把功能写成可用。文档必须跟随实际行为，不能提前承诺。

## 另请参阅

- [内置技能](../../bundled.md)——按类别整理的全部内置目录技能。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、分配和故障排查。
- [应用内置目录](../../../skills.md#3-app-shipped-catalog)——内置和可选目录技能的版本管理与更新方式。
- [技能指南](../../../../guides/org/skills.md)——界面操作指南和内置目录概览。
