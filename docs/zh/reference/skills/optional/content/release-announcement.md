---
paperclip_version: v2026.618.0
seo_title: 版本公告技能
seo_description: 根据发布渠道撰写版本说明，可用于变更日志、博客、应用内通知或社交媒体；突出用户影响并说明升级方式。
---

# 版本公告

> 撰写版本公告（变更日志、博客文章、应用内通知或社交媒体帖子），突出用户影响，说明目标受众，并提供升级/迁移步骤，不要添加空话。

根据发布渠道撰写相应公告，可以是变更日志、博客文章、应用内通知或社交媒体帖子。开篇要说明用户影响和目标受众，并提供升级或迁移步骤，不要添加空话。读者应能在 30 秒内判断此次发布是否与自己相关，以及需要采取什么行动。

此技能是**可选**目录技能，可按需安装。如何安装、审计、更新、分配和重置目录技能，请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

## 目录元数据

| 字段 | 值 |
|---|---|
| Catalog id | `paperclipai:optional:content:release-announcement` |
| Canonical key | `paperclipai/optional/content/release-announcement` |
| Catalog path | `catalog/optional/content/release-announcement` |
| Kind | `optional` |
| Category | `content` |
| Slug | `release-announcement` |
| Entrypoint | `SKILL.md` |
| Trust level | `markdown_only` |
| Compatibility | `compatible` |
| Default install | `false` |
| Recommended roles | `devrel`、`product`、`writer` |
| Requires | — |
| Tags | `release`、`changelog`、`announcement`、`communication` |
| Files | 1 |
| Content hash | `sha256:f22a9ed696e6614c6db2757a149f48b3295e81f78c27d065d9cb164cf4f8a9bd` |
| Package | `@paperclipai/skills-catalog@0.3.1` |

## 文件清单

| 路径 | 类型 | 字节数 |
|---|---|---:|
| `SKILL.md` | `skill` | 4,416 |

## 完整技能定义

以下代码块是此技能完整且权威的 `SKILL.md`，也是智能体在运行时加载的确切文件。可使用代码块右上角的控件复制或下载为 `SKILL.md`。

````markdown skill-source
---
name: release-announcement
description: Write a release announcement — changelog, blog post, in-app note, or social post — that leads with user impact, names the audience, and includes upgrade/migration steps without filler.
key: paperclipai/optional/content/release-announcement
recommendedForRoles:
  - devrel
  - product
  - writer
tags:
  - release
  - changelog
  - announcement
  - communication
---

# Release Announcement

Write the channel-appropriate announcement for a release without churn. Different surfaces need different shapes: a changelog entry is not a blog post is not a social card. The bar is: a reader of the chosen surface can decide in under 30 seconds whether this release affects them, and if so what to do.

## When to use

- A version, feature, or fix is shipping and needs writeup for at least one surface.
- A previously private feature is going GA.
- A breaking change needs broadcast before users hit it.

## When not to use

- An internal-only change with no user impact. Update internal docs; do not announce.
- The release is incomplete (still in active development). Wait until it ships, even if marketing wants the post.

## Determine the audience and channel first

| Audience | Best channel | Tone |
|---|---|---|
| Existing power users | Changelog, in-app note | Terse, factual, links |
| Engineering teams adopting your API | Release notes, dev blog | Examples, migration steps, version pins |
| Prospective customers | Landing page, marketing blog | Story arc, problem → solution, social proof |
| Broad audience | Social post, email newsletter | One-sentence pitch, link to depth |
| Internal team | Slack/Discord post | What changed, who to ping if it breaks |

Pick the audience for *this* writeup. One release often needs several writeups; do not blend them.

## Universal structure

Whatever the channel, lead with:

1. **What changed.** One sentence in the user's vocabulary.
2. **Who it affects.** Which user role / use case.
3. **What to do.** Migrate now / opt-in / no action needed.

Everything else is depth that supports those three.

## Channel templates

### Changelog entry (terse)

```md
## v1.42.0 — 2026-05-26

### Added
- <feature> — <one-line user benefit>. ([#1234](link))

### Changed
- <change> — <one-line impact>. ([#1235](link))

### Fixed
- <bug> — <one-line user-visible symptom>. ([#1236](link))

### Deprecated
- <thing>. Replaced by <thing>. Removal planned for v<x>.

### Breaking
- <change>. **Migration:** <one-line> or <link to guide>.
```

### Release notes (for adopters)

Same as changelog, plus:

- Migration guide section with before/after code.
- Compatibility table (versions, runtimes, OS).
- Known issues and workarounds.
- Acknowledgements (contributors, reporters of fixed bugs).

### Dev blog post (300–800 words)

- **Hook (1 paragraph):** the problem the release solves, in a real-world scenario.
- **What's new (3–5 bullets with sub-paragraphs):** features, with one code or screenshot example each.
- **Upgrade (1 paragraph):** how to upgrade, what to check.
- **What's next:** one sentence about the next direction. Avoid promises.

### In-app note

- 1 sentence.
- 1 link.
- Dismiss after seen.

### Social post

- 1 sentence pitch.
- 1 link.
- 1 image or short clip.
- No threadbait. If it needs a thread, write a blog post instead.

## Writing rules

- Lead with the user, not the team. `You can now export to CSV` beats `We've added CSV export`.
- Numbers beat adjectives. `60% faster cold start` beats `much faster`. Cite the methodology.
- Show, don't just tell. One code snippet, one screenshot — more is noise.
- Date the post. Undated release content rots fastest.
- Link the migration path explicitly. Do not bury it.
- Mark breaking changes with `**Breaking:**` prefix. Repeat in the email/social channel.

## Avoid

- "We are excited to announce" filler.
- Lists of changes that mix user-visible and internal items.
- Marketing claims without a way to verify.
- Promised dates for unshipped work.
- Pre-announcing something the team has not yet committed to ship.

## Post-publish checklist

- Changelog is in source control alongside the release.
- Blog post date matches actual ship date.
- All links work (release tag, PRs, docs sections).
- Breaking changes are also in the upgrade guide, not only the post.
- Internal team is notified before the public post goes live, not after.
````

## 适用场景

- 版本、功能或修复即将发布，至少需要面向一个渠道撰写说明。
- 先前仅内部使用的功能即将正式发布。
- 不兼容变更需要在用户遇到之前广泛告知。

### 不适用场景

- 变更仅影响内部且不影响用户——应更新内部文档，而不要发布公告。
- 版本仍在积极开发中、尚未完成——即使市场团队希望尽快发布文章，也应等到真正发布后再撰写公告。

## 先确定受众和渠道

| 受众 | 推荐渠道 | 语气 |
|---|---|---|
| 现有高级用户 | 变更日志、应用内通知 | 简洁、客观、提供链接 |
| 正在采用你们 API 的工程团队 | 版本说明、开发者博客 | 示例、迁移步骤、固定版本 |
| 潜在客户 | 产品落地页、营销博客 | 叙事结构：问题 → 解决方案 → 社会证明 |
| 广泛受众 | 社交媒体帖子、电子邮件简报 | 一句话介绍，附深度内容链接 |
| 内部团队 | Slack/Discord 帖子 | 变更内容，以及出问题时应联系谁 |

先为**当前这篇**说明选择目标受众。同一版本往往需要多篇不同说明，不要把不同受众的内容混在一起。

## 通用结构

无论发布渠道是什么，开头都要说明：

1. **变更内容。** 用用户熟悉的语言写一句话。
2. **影响对象。** 哪种用户角色或使用场景会受影响。
3. **需要采取的行动。** 立即迁移、自愿启用，还是无需操作。

其他所有内容都应围绕这三点提供更多细节。

## 各渠道模板

### 变更日志条目（简洁）

```md
## v1.42.0 — 2026-05-26

### 新增
- <功能> — <一句话说明用户收益>。([#1234](link))

### 变更
- <变更> — <一句话说明影响>。([#1235](link))

### 修复
- <缺陷> — <一句话说明用户可见的问题>。([#1236](link))

### 弃用
- <项目>。由 <替代项目> 取代。计划在 v<x> 移除。

### 不兼容变更
- <变更>。**迁移方式：** <一句话说明> 或 <迁移指南链接>。
```

### 版本说明（面向采用者）

包含变更日志内容，并补充：

- 迁移指南章节，提供变更前后的代码。
- 兼容性表（版本、运行时、操作系统）。
- 已知问题和解决方法。
- 致谢（贡献者、报告已修复缺陷的人员）。

### 开发者博客文章（300–800 字）

- **引入（1 段）：** 描述此次版本在真实场景中解决的问题。
- **新增内容（3–5 个要点及其子段落）：** 介绍功能，每项配一个代码或截图示例。
- **升级（1 段）：** 说明如何升级以及需要检查什么。
- **后续方向：** 用一句话描述下一步方向，不要作出承诺。

### 应用内通知

- 1 句话。
- 1 个链接。
- 用户看过后自动关闭。

### 社交媒体帖子

- 一句话介绍。
- 1 个链接。
- 1 张图片或短视频。
- 不要为了制造互动而故意含糊或诱导回复。如果内容需要长帖，就改写成博客文章。

## 写作规则

- 以用户为主语，而不是团队。`现在你可以导出 CSV` 优于 `我们新增了 CSV 导出`。
- 用数字代替形容词。`冷启动速度提升 60%` 优于 `速度快了很多`，并引用测量方法。
- 展示内容，不要只用文字描述。一个代码片段和一张截图即可；过多内容会形成噪音。
- 标明发布日期。没有日期的版本内容最容易过时。
- 明确链接到迁移路径，不要把它埋在正文中。
- 不兼容变更使用 `**Breaking:**` 前缀标明，并在电子邮件/社交媒体渠道中重复说明。

## 避免事项

- 使用“我们很高兴宣布”之类的套话。
- 将用户可见变更和内部变更混在同一份清单中。
- 发布没有办法验证的营销宣传。
- 为尚未发布的工作承诺发布日期。
- 团队尚未承诺交付之前，提前宣布某项功能。

## 发布后检查清单

- 变更日志已与版本一同提交到源代码管理系统。
- 博客文章日期与实际发布日期一致。
- 所有链接都有效（版本标签、PR、文档章节）。
- 不兼容变更也写入升级指南，而不只写在公告中。
- 公告公开发布前通知内部团队，而不是发布后才通知。

## 另请参阅

- [可选技能](../../optional.md)——按类别整理的全部可选目录技能。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、分配和故障排查。
- [应用内置目录](../../../skills.md#3-app-shipped-catalog)——内置和可选目录技能的版本管理与更新方式。
- [技能指南](../../../../guides/org/skills.md)——界面操作指南和内置目录概览。
