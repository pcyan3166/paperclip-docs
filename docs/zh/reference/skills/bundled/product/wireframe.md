---
paperclip_version: v2026.618.0
seo_title: 线框图技能
seo_description: 将布局、层级和组件制作成独立 SVG 黑白低保真 UI 线框图，不添加颜色或视觉润色。
---

# 线框图

> 将低保真黑白 UI 线框图生成为独立 SVG 文件；可选打包为单页 HTML 查看器，并通过 here-now 技能发布。用户要求“为 X 绘制线框图”“画个屏幕草图”“起草布局”“低保真模型”“粗略草图”“制作可查看线框图的页面”“为这些屏幕构建查看器”或“部署/发布/托管线框图”时使用。用户需要生产级 UI 代码、品牌设计、高保真模型或带动画/交互的原型时**不要**使用此技能，应改用 frontend-design 等技能。

制作黑白低保真 UI 线框图，并将每个屏幕保存为独立 SVG 文件。重点展示布局、层级和组件，不添加颜色、品牌元素或视觉润色。其目的是在投入高保真设计前先对齐整体结构。

这是一个**内置**目录技能，属于基础技能套件。目录技能的安装、审计、更新、指派和重置方法请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

## 适用情况

- 需要在视觉设计前传达屏幕或流程的结构。
- 产品或工程讨论需要有具体布局供大家评议。
- 希望低成本并排探索多种布局方案。

### 不适用情况

- 任务需要可直接发布的高保真视觉设计——本技能刻意只制作低保真图。
- 已有明确布局，只需进行视觉润色。

## 目录元数据

| 字段 | 值 |
|---|---|
| 目录 ID | `paperclipai:bundled:product:wireframe` |
| 规范 key | `paperclipai/bundled/product/wireframe` |
| 目录路径 | `catalog/bundled/product/wireframe` |
| 类型 | `bundled` |
| 类别 | `product` |
| Slug | `wireframe` |
| 入口文件 | `SKILL.md` |
| 信任级别 | `assets` |
| 兼容性 | `compatible` |
| 默认安装 | `false` |
| 建议角色 | `designer`、`product`、`engineer` |
| 依赖 | — |
| 标签 | `design`、`wireframe`、`ux`、`prototyping`、`svg` |
| 文件数 | 7 |
| 内容哈希 | `sha256:0bd9a9fdc656d529e3f97c00cd504dcf72d3a4fecb8b0504ca2fe3e00d63287f` |
| 软件包 | `@paperclipai/skills-catalog@0.3.1` |

## 文件清单

| 路径 | 类型 | 字节数 |
|---|---|---:|
| `SKILL.md` | `skill` | 11,887 |
| `assets/site-template.html` | `asset` | 14,791 |
| `assets/template-mobile.svg` | `asset` | 925 |
| `assets/template.svg` | `asset` | 1,262 |
| `references/components.md` | `reference` | 14,806 |
| `references/examples.md` | `reference` | 16,108 |
| `references/grid-system.md` | `reference` | 4,581 |

## 完整技能定义

以下内容是此技能完整且权威的 `SKILL.md`，也是代理在运行时实际加载的文件。可使用代码块右上角的控件复制或下载为 `SKILL.md`。

````markdown skill-source
---
name: wireframe
description: Produce low-fidelity black-and-white UI wireframes as standalone SVG files, optionally bundled into a single-page HTML viewer and published via the here-now skill. Use when the user asks to "wireframe X", "sketch a screen for", "draft a layout", "low-fi mockup", "rough mock", "make a page to view the wireframes", "build a viewer for these screens", or to "deploy / publish / host the wireframes". Do NOT use when the user wants production UI code, branded designs, hi-fi mockups, or animated/interactive prototypes — use frontend-design or similar instead.
key: paperclipai/bundled/product/wireframe
recommendedForRoles:
  - designer
  - product
  - engineer
tags:
  - design
  - wireframe
  - ux
  - prototyping
  - svg
---

# Wireframe

Produce low-fidelity, black-and-white UI wireframes as **standalone SVG files**. The goal is to communicate **structure** — what goes where, in what order, at roughly what size — without committing to colour, brand, or polish.

## When to use

Trigger on phrases like:

- "wireframe a [screen / page / flow] for X"
- "low-fi / lo-fi mockup of X"
- "draft a layout for X"
- "rough sketch of the [dashboard / settings / login / ...] page"
- "show me how X would lay out before I build it"

Skip and defer to `frontend-design` (or similar) when the request mentions: brand, polish, real components, "production-ready", colour palettes, hi-fi, Figma export, or actual code/HTML/React deliverables.

## House style — non-negotiable

Wireframes are diagnostic, not decorative. Lock these tokens on every output:

| Token            | Value                                             | Notes                                  |
| ---------------- | ------------------------------------------------- | -------------------------------------- |
| Stroke           | `#000` width `1.5`                                | All borders, dividers, outlines        |
| Fill (boxes)     | `#fff`                                            | Default for cards/containers           |
| Placeholder fill | `#e6e6e6`                                         | Image/avatar/empty-state regions       |
| Text colour      | `#000` for labels, `#666` for placeholder text    | No other colours                       |
| Accent           | `#d33` (dashed) — annotation layer ONLY           | Never inside real UI elements          |
| Font             | `font-family="-apple-system, system-ui, sans-serif"` | Single typeface across the whole file  |
| Type scale       | `12` caption · `14` body · `20` heading · `28` title | No other sizes                         |
| Grid             | 8px snap, 24px gutter                             | All x/y/w/h must be multiples of 8     |
| Default canvas   | `1280×800` desktop, `375×812` mobile, `768×1024` tablet | Pick one and state it in the comment   |

If you need to highlight a specific region for a callout, use the **annotation layer** (red dashed). Never colourise the wireframe itself.

## Workflow

1. **Confirm scope.** What screen(s)? Which viewport (desktop / tablet / mobile)? Single screen or multi-screen flow? If unclear, ask one question, then proceed with the most likely default.
2. **Pick a canvas** from the table above. State the viewport in your reply.
3. **Compose from primitives.** Read `references/components.md` and assemble the screen from the primitive snippets. Snap every coordinate to 8px.
4. **Write the SVG to a file.** Default path: `wireframes/<slug>.svg` in the working directory. Filename slug describes the screen (`login.svg`, `dashboard.svg`, `settings-account.svg`).
5. **Emit a textual annotation list** in your reply, mapping each numbered region in the SVG to a one-line description ("1 — primary nav, 2 — search input, 3 — list of recent items"). This makes the wireframe accessible, queryable, and reviewable in text-only channels.
6. **For multi-screen flows**, produce one SVG per screen and a summary `flow.svg` that arranges thumbnails left-to-right with arrows between them.

## Quick start — minimal SVG

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="1280" height="800" viewBox="0 0 1280 800"
     font-family="-apple-system, system-ui, sans-serif" fill="#fff" stroke="#000" stroke-width="1.5">
  <!-- canvas border -->
  <rect x="0" y="0" width="1280" height="800" />

  <!-- example: a button -->
  <g transform="translate(48, 48)">
    <rect width="120" height="40" rx="4" />
    <text x="60" y="25" font-size="14" text-anchor="middle" stroke="none" fill="#000">Continue</text>
  </g>
</svg>
```

Two house-style gotchas worth memorising:

- Always set `stroke="none"` on `<text>` elements (text inherits the parent stroke and gets a halo otherwise).
- Always wrap text fill explicitly (`fill="#000"`) since the parent group fill is `#fff` for boxes.

## Primitive library

The full set of reusable primitives lives in `references/components.md`. Load it whenever you need a primitive whose exact markup you do not have in working memory. Do not re-derive primitives from scratch — copy the snippet and adjust coordinates.

Primitives provided:

- **Inputs:** button (filled, outlined, icon), text input, textarea, dropdown, checkbox, radio, toggle, search input
- **Layout:** card, section divider, sidebar, two-column, three-column
- **Navigation:** navbar, tab bar, breadcrumb, pagination, sidebar nav
- **Content:** heading, paragraph block, list row, table, key-value pair, metric tile
- **Media:** image placeholder, avatar (circle/square), video placeholder
- **Overlay:** modal, drawer, toast, tooltip, dropdown menu (open state)
- **Annotation:** numbered callout, dashed region highlight, arrow connector

## Grid, palette, and type scale

For exact pixel values, palette tokens, and type sizes, see `references/grid-system.md`.

## Worked examples

`references/examples.md` contains four complete wireframes you can copy and adapt:

1. Login screen (mobile, 375×812)
2. Admin dashboard (desktop, 1280×800)
3. Settings page with form (desktop, 1280×800)
4. Modal confirmation overlay (desktop, 1280×800)

When the user's request is close to one of these, start from the example and modify, rather than building from blank.

## Output convention

Every wireframe response should include:

1. The SVG file written to disk (path stated explicitly).
2. The SVG inlined in your reply (so it renders in markdown previews).
3. A short numbered annotation list mapping regions to intent.
4. Any explicit assumption you made (viewport, signed-in state, empty/populated, dark/light not applicable since monochrome).

## If the user requests a website / viewer page

Trigger on phrases like "make a page that shows the screens", "single page I can scroll", "build a viewer", "let me click through the wireframes", "show them all on one page", or any request to bundle multiple wireframes into a browsable artifact (not a production site).

Build **one static `index.html`** that loads the SVG wireframes directly. Do not turn this into a React app or component library — it's a review surface, not product UI.

**File layout** (default):

```
design/<task-slug>/
  index.html
  wireframes/    # the SVGs from this skill
  screenshots/   # any reference screenshots
```

**Page anatomy** (start from `assets/site-template.html` and adjust — do not re-derive the CSS):

- **Sticky sidebar TOC** (240px on desktop) listing every screen with anchor links. Group by Flow / Screens / Open questions.
- **Hero header** at the top: crumb (issue id), title, one-paragraph summary, pill row of meta tags (`12 screens`, `Lo-fi · monochrome`, `Click any wireframe to zoom`).
- **One section per screen** with a 2-column grid: wireframe on the left, reference image + numbered annotations + a "Why this changes" callout on the right. The wireframe `<img>` points to the SVG file directly — do not inline it.
- **Click-to-zoom lightbox** for any element marked `[data-zoom]`. Esc and backdrop click both close.
- **Flow diagram** section near the top that loads `wireframes/flow.svg` full-width.
- **Open questions** section at the bottom for unresolved decisions.

**House style for the viewer** (matches the wireframes themselves):

- Palette: `--bg: #fafaf8`, `--panel: #fff`, `--ink: #111`, `--muted: #666`, `--line: #e5e5e0`, `--accent: #d33` (red dashed callouts only).
- System font stack only: `-apple-system, system-ui, "Segoe UI", sans-serif`. No web fonts.
- 8px-based spacing, `border-radius: 8px` on cards, 1px `--line` borders, no shadows except the hover lift on `.wire`.
- The viewer chrome is allowed to be slightly more polished than the wireframes (subtle hover, rounded cards) — but never colourful. The wireframes themselves stay strictly monochrome.

**Responsive** (verify before reporting done):

- ≥980px: two-column grid, sidebar TOC visible.
- 900–980px: grid stacks to one column, TOC still sidebar.
- <900px (tablet/phone): TOC collapses to a sticky `<details>` disclosure at the top of the page, defaults closed; tapping a link auto-closes it. Sections get `scroll-margin-top: 80px` so anchor jumps clear the sticky bar.
- <560px (phone): tighter type/spacing scale, hero shrinks, lightbox switches from flex-centered to block layout at full viewport width with `touch-action: pinch-zoom` so users can pinch in further.

**Verification** before handing off: open the file in a browser and walk it at 1440×900, 768×1024, and 390×844. Confirm anchor jumps land cleanly, lightbox opens/closes, and SVGs render at the right aspect.

## If the user asks to deploy / publish / host the wireframes

Defer to the **`here-now` skill** — it owns publishing, anonymous vs. permanent sites, claim tokens, and credentials. Do not roll your own hosting.

Load the `here-now` skill and follow its `publish.sh` recipe. The shape is:

```bash
cd design/<task-slug>
{path-to-here-now}/scripts/publish.sh .
# → https://{adjective-noun-suffix}.here.now/
```

If `here-now` isn't installed for the current agent, install it (`npx skills add heredotnow/skill --skill here-now -g`) or escalate to whoever owns the agent's skill set. Do not roll your own hosting.

Things to remember when invoking it:

- Publish the **directory containing `index.html` at its root**, not the parent. `index.html` must be at the root of the published tree.
- Without saved credentials the site is **anonymous and expires in 24 hours**. With a saved API key, it's permanent. If the user wants a permanent URL, follow the `here-now` skill's sign-in-code flow — don't fake your way around it.
- For an update, pass `--slug {existing-slug}` so the URL stays stable across review rounds (the script auto-loads the claim token from `.herenow/state.json`).
- Read `publish_result.*` lines from script stderr to determine `auth_mode` and the claim URL — do not read `.herenow/state.json` and present its contents as the source of truth.
- Always share the `siteUrl` from the current run; if anonymous, also share the claim URL and the 24h expiry warning.

## What this skill is NOT for

- **Production UI code** — use `frontend-design` or write React/HTML directly.
- **Hi-fi or branded mockups** — use Figma or a design tool, not this.
- **Interactive prototypes** — SVG is static; export multi-screen flows as a flow.svg.
- **Diagrams of system architecture, sequence flows, or data models** — use mermaid or plantuml.
- **Illustrations or art** — use `example-skills:canvas-design` or `algorithmic-art`.

## Bundled assets

- `assets/template.svg` — blank desktop canvas with hidden 8px-grid guides; copy as a starting point.
- `assets/template-mobile.svg` — same for 375×812 mobile.
- `assets/site-template.html` — minimal review-viewer page (sticky TOC + responsive collapse + lightbox). Copy to `design/<task-slug>/index.html` and fill in the sections when the user requests a website.
````

## 中文译文

# 线框图

生成低保真、黑白的 UI 线框图，并分别保存为 SVG 文件。目标是表达界面**结构**——各部分的位置、顺序和大致尺寸，而不是确定颜色、品牌或视觉润色。

## 适用场景

用户提出以下请求时使用：

- “为 X 绘制一个[屏幕/页面/流程]线框图”。
- “为 X 做一份低保真/粗略原型”。
- “起草一个布局”。
- “粗略画一下[仪表板/设置/登录/... ]页面”。
- “在实现之前先展示 X 的布局”。

如果用户提到品牌、精致视觉、真实组件、“可用于生产”、配色方案、高保真稿、Figma 导出或实际代码/HTML/React 交付物，应转而使用 `frontend-design` 或类似技能。

## 固定的线框图风格

线框图用于诊断，不用于装饰。每个输出都必须固定使用以下设计令牌：

| 令牌 | 值 | 说明 |
|---|---|---|
| 描边 | `#000`，宽度 `1.5` | 用于所有边框、分隔线和轮廓 |
| 填充（框） | `#fff` | 卡片/容器默认填充色 |
| 占位填充 | `#e6e6e6` | 图片、头像和空状态区域 |
| 文字颜色 | 标签使用 `#000`，占位文字使用 `#666` | 不使用其他颜色 |
| 强调色 | `#d33`（虚线） | **仅**用于注释图层，不得出现在真实 UI 元素中 |
| 字体 | `font-family="-apple-system, system-ui, sans-serif"` | 整个文件只使用一种字体 |
| 字号 | `12` 注释、`14` 正文、`20` 标题、`28` 页面标题 | 不使用其他字号 |
| 网格 | 8px 对齐，24px 间距 | 所有 x/y/w/h 坐标必须是 8 的倍数 |
| 默认画布 | 桌面 `1280×800`、移动端 `375×812`、平板 `768×1024` | 选择一种并在注释中说明 |

若要突出某个区域供读者参考，请使用红色虚线的**注释图层**。不得给线框图本身上色。

## 工作流程

1. **确认范围。** 需要绘制哪些屏幕？视口是桌面、平板还是移动端？是单屏还是多屏流程？若不明确，先问一个问题，然后采用最可能的默认值继续。
2. **选择画布。** 从上表中选择一种画布，并在回复中说明视口。
3. **使用基础组件组合。** 阅读 `references/components.md`，使用其中的基础片段组装屏幕。所有坐标都按 8px 网格对齐。
4. **将 SVG 写入文件。** 默认保存在工作目录的 `wireframes/<slug>.svg`。文件名 slug 应描述屏幕，例如 `login.svg`、`dashboard.svg`、`settings-account.svg`。
5. **在回复中给出文字注释列表。** 将 SVG 中每个编号区域映射到一句描述（例如“1 — 主导航，2 — 搜索框，3 — 最近项目列表”）。这样在纯文本渠道中也能访问、查询和审核线框图。
6. **多屏流程**应为每个屏幕各生成一个 SVG，并生成一份 `flow.svg`，用箭头从左到右排列各缩略图。

## 快速开始：最简 SVG

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="1280" height="800" viewBox="0 0 1280 800"
     font-family="-apple-system, system-ui, sans-serif" fill="#fff" stroke="#000" stroke-width="1.5">
  <!-- canvas border -->
  <rect x="0" y="0" width="1280" height="800" />

  <!-- example: a button -->
  <g transform="translate(48, 48)">
    <rect width="120" height="40" rx="4" />
    <text x="60" y="25" font-size="14" text-anchor="middle" stroke="none" fill="#000">Continue</text>
  </g>
</svg>
```

需要准确标记时，请直接复制该示例并仅调整坐标。需牢记两个常见问题：

- `<text>` 元素必须设置 `stroke="none"`，否则文字会继承父级描边并出现光晕。
- 必须显式为文字设置 `fill="#000"`，因为框的父级组默认填充为 `#fff`。

## 基础组件库

所有可复用基础组件都位于 `references/components.md`。若不确定某个组件的精确标记，应加载该文件。不要从头重新推导组件；复制片段后调整坐标即可。

组件包括：

- **输入控件：** 按钮（实心、描边、图标）、文本输入框、多行文本框、下拉菜单、复选框、单选框、开关、搜索框。
- **布局：** 卡片、分区分隔线、侧边栏、双栏、三栏。
- **导航：** 导航栏、标签栏、面包屑、分页、侧边导航。
- **内容：** 标题、段落块、列表行、表格、键值对、指标卡片。
- **媒体：** 图片占位框、圆形/方形头像、视频占位框。
- **浮层：** 模态框、抽屉、提示条、工具提示、下拉菜单（展开状态）。
- **注释：** 编号标注、虚线区域高亮、箭头连接线。

## 网格、配色和字号

精确的像素值、配色令牌和字号请查阅 `references/grid-system.md`。

## 完整示例

`references/examples.md` 中有四份可复制和调整的完整线框图：

1. 登录屏幕（移动端，`375×812`）。
2. 管理仪表板（桌面端，`1280×800`）。
3. 带表单的设置页面（桌面端，`1280×800`）。
4. 模态确认浮层（桌面端，`1280×800`）。

用户需求接近其中某个示例时，应从该示例开始修改，而不是从空白画布开始。

## 输出约定

每次回复都应包含：

1. 写入磁盘的 SVG 文件，并明确说明路径。
2. 在回复中内嵌 SVG，使其能在 Markdown 预览中显示。
3. 简短的编号注释列表，说明各区域的用途。
4. 明确列出所作的假设（视口、登录状态、空/有数据状态；由于使用单色设计，不适用深色/浅色模式）。

## 用户要求制作网站/查看器页面时

用户说“制作一个展示这些屏幕的页面”“做一个可滚动的单页”“构建查看器”“让我能点击切换线框图”“把它们都放在一个页面上”，或要求将多个线框图打包成可浏览产物时，使用本节。

构建**一个静态 `index.html`**，直接加载 SVG 线框图。不要将其构建为 React 应用或组件库——这只是审核界面，不是产品 UI。

默认文件布局：

```text
design/<task-slug>/
  index.html
  wireframes/    # 本技能生成的 SVG
  screenshots/   # 参考截图（如有）
```

页面结构应从 `assets/site-template.html` 开始并按需调整，不要重新推导 CSS：

- **固定侧边目录**：桌面端宽 240px，列出每个屏幕的锚点链接，并按 Flow / Screens / Open questions 分组。
- **顶部介绍区**：包含面包屑（issue ID）、标题、一段摘要，以及元信息标签行（例如 `12 screens`、`Lo-fi · monochrome`、`Click any wireframe to zoom`）。
- **每屏一个章节**：使用双栏网格，左侧显示线框图；右侧显示参考图片、编号注释和“Why this changes”说明。线框图的 `<img>` 应直接指向 SVG 文件，不要内嵌 SVG。
- **点击放大灯箱**：任何带 `[data-zoom]` 的元素都可打开灯箱。按 Esc 键和点击背景都应关闭。
- **流程图章节**：靠近页面顶部，全宽加载 `wireframes/flow.svg`。
- **开放问题**：在页面底部列出尚未解决的决定。

查看器的风格应与线框图一致：

- 配色：`--bg: #fafaf8`、`--panel: #fff`、`--ink: #111`、`--muted: #666`、`--line: #e5e5e0`、`--accent: #d33`（仅供红色虚线标注使用）。
- 只使用系统字体栈：`-apple-system, system-ui, "Segoe UI", sans-serif`。不要使用 Web 字体。
- 间距以 8px 为基准，卡片使用 `border-radius: 8px` 和 1px 的 `--line` 边框；除 `.wire` 悬停抬升效果外不要加阴影。
- 查看器外框可以比线框图略精致（例如轻微悬停效果和圆角卡片），但不能使用彩色设计。线框图本身必须严格保持单色。

响应式要求（交付前必须验证）：

- `≥980px`：双栏网格，显示侧边目录。
- `900–980px`：网格堆叠成单栏，侧边目录仍显示。
- `<900px`（平板/手机）：目录折叠为页面顶部固定的 `<details>` 展开控件，默认关闭；点击链接后自动关闭。章节设置 `scroll-margin-top: 80px`，避免锚点跳转被固定栏遮挡。
- `<560px`（手机）：缩小字号和间距；灯箱改为全视口宽度的块布局，并设置 `touch-action: pinch-zoom`，方便用户继续双指缩放。

交付前应在浏览器中分别以 `1440×900`、`768×1024` 和 `390×844` 检查页面。确认锚点跳转位置正确、灯箱能正常打开/关闭，且 SVG 按正确宽高比显示。

## 用户要求部署/发布/托管线框图时

应交由 `here-now` 技能处理；该技能负责发布、匿名与永久站点、认领令牌和凭据。不要自行实现托管。

加载 `here-now` 技能，并按其 `publish.sh` 流程操作。命令形态如下：

```bash
cd design/<task-slug>
{path-to-here-now}/scripts/publish.sh .
# → https://{adjective-noun-suffix}.here.now/
```

若当前智能体未安装 `here-now`，应安装它（`npx skills add heredotnow/skill --skill here-now -g`），或联系负责该智能体技能集的人。不要自行实现托管。

调用时牢记：

- 发布目录应是根目录中直接包含 `index.html` 的目录，而不是其父目录。发布树的根目录必须有 `index.html`。
- 没有保存的凭据时，站点是匿名的，24 小时后过期；使用已保存的 API key 时，站点永久有效。用户要求永久 URL 时，应按照 `here-now` 技能中的登录码流程操作，不要伪造流程。
- 更新现有站点时传入 `--slug {existing-slug}`，以便审核轮次之间保持 URL 稳定（脚本会自动加载 `.herenow/state.json` 中的认领令牌）。
- 应从脚本 stderr 中的 `publish_result.*` 行读取 `auth_mode` 和认领 URL；不要读取 `.herenow/state.json` 并将其内容作为事实依据。
- 始终分享本次运行得到的 `siteUrl`；若为匿名站点，还应提供认领 URL 和 24 小时过期说明。

## 本技能不适用于

- **生产 UI 代码**——使用 `frontend-design`，或直接编写 React/HTML。
- **高保真或品牌化原型**——使用 Figma 或其他设计工具。
- **交互式原型**——SVG 是静态的；多屏流程应导出为 `flow.svg`。
- **系统架构图、时序图或数据模型图**——使用 mermaid 或 plantuml。
- **插画或艺术作品**——使用 `example-skills:canvas-design` 或 `algorithmic-art`。

## 内置资源

- `assets/template.svg`——包含隐藏的 8px 网格辅助线的空白桌面画布，可作为起点。
- `assets/template-mobile.svg`——对应 `375×812` 移动端画布的模板。
- `assets/site-template.html`——包含固定目录、响应式折叠和灯箱的最简审核查看器页面。用户要求网站时，将它复制到 `design/<task-slug>/index.html` 并填充各章节。

## 另请参阅

- [内置技能](../../bundled.md)——按类别分组的所有内置目录技能。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、技能指派和故障排查。
- [应用内置目录](../../../skills.md#3-app-shipped-catalog)——内置和可选目录技能的版本管理和更新机制。
- [技能指南](../../../../guides/org/skills.md)——UI 操作指南和内置技能目录概览。
