---
paperclip_version: v2026.618.0
seo_title: 代理浏览器技能
seo_description: 使用受监督的真实浏览器验证行为、采集证据，或读取静态抓取无法访问的 JavaScript 渲染内容。
---

# 代理浏览器

> 使用真实浏览器检查或交互式操作网页/应用——进行导航、截图、查看控制台和网络请求、填写简单表单——适用于验证任务，不适用于无人值守自动化。

使用受监督的真实浏览器验证行为、采集证据，或读取静态抓取无法获取的 JavaScript 渲染内容。本技能用于监督式页面/应用验证，不用于无人值守爬取。

这是一个**可选**目录技能，需要时再选择安装。关于安装、审计、更新、指派和重置目录技能，请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

## 适用情况

- 需要对已部署页面或本地开发服务器截图，确认 UI 改动。
- 需要读取 `curl` / `wget` 无法看到的 JavaScript 渲染内容。
- 用户报告 UI 缺陷，需要复现并采集控制台错误、网络请求或布局状态。
- 需要执行简短流程（加载、点击、观察）以验证验收标准。

### 不适用情况

- 页面可通过静态 HTML 访问——请使用 HTTP 抓取。
- 任务要求无人值守、大规模爬取——这应由专用爬虫完成。
- 网站需要你无权访问的身份认证，或其条款禁止自动化操作。

## 目录元数据

| 字段 | 值 |
|---|---|
| 目录 ID | `paperclipai:optional:browser:agent-browser` |
| 规范 key | `paperclipai/optional/browser/agent-browser` |
| 目录路径 | `catalog/optional/browser/agent-browser` |
| 类型 | `optional` |
| 类别 | `browser` |
| Slug | `agent-browser` |
| 入口文件 | `SKILL.md` |
| 信任级别 | `markdown_only` |
| 兼容性 | `compatible` |
| 默认安装 | `false` |
| 建议角色 | `qa`、`engineer`、`researcher` |
| 依赖 | — |
| 标签 | `browser`、`puppeteer`、`playwright`、`verification` |
| 文件数 | 1 |
| 内容哈希 | `sha256:eabb2c9f7b5e1a27ebb1e05a711d61433a266478154cd671a685e99e67aadea2` |
| 软件包 | `@paperclipai/skills-catalog@0.3.1` |

## 文件清单

| 路径 | 类型 | 字节数 |
|---|---|---:|
| `SKILL.md` | `skill` | 5,133 |

## 完整技能定义

以下内容是此技能完整且权威的 `SKILL.md`，也是代理在运行时实际加载的文件。可使用代码块右上角的控件复制或下载为 `SKILL.md`。

````markdown skill-source
---
name: agent-browser
description: Drive a real browser to inspect or interact with a web page or app — navigate, take screenshots, read console and network, fill simple forms — for verification tasks, not unattended automation.
key: paperclipai/optional/browser/agent-browser
recommendedForRoles:
  - qa
  - engineer
  - researcher
tags:
  - browser
  - puppeteer
  - playwright
  - verification
---

# Agent Browser

Use a controlled browser to verify behavior, capture evidence, or extract information from web pages that a static fetch cannot reach (SPAs, login-gated pages, dynamic content). This skill is about supervised verification, not unattended scraping.

## When to use

- You need a screenshot of a deployed page or a local dev server to confirm a UI change.
- You need to read JavaScript-rendered content that `curl`/`wget` will not see.
- A user reports a UI bug and you need to reproduce it interactively to capture console errors, network requests, or layout state.
- You need to walk through a short flow (load page, click, observe) to verify acceptance criteria.

## When not to use

- The page is reachable as static HTML. Use `curl`/HTTP fetch — it is cheaper, faster, and more reliable.
- The task is unattended large-scale scraping. That belongs to a dedicated scraper with rate limits, robots.txt handling, and a real user agent policy — not this skill.
- The site is behind authentication you do not own credentials for, or whose terms of service prohibit automation.
- The site involves sensitive accounts (banking, healthcare, government) where automation risks lockout or compliance issues.

## Before launching the browser

- Confirm the URL and what state should be true after navigation.
- Decide what evidence is needed: full-page screenshot, viewport screenshot, console log, network trace, HTML snapshot, extracted text.
- Decide the viewport size that matters for the task (mobile vs desktop). Default to a desktop size unless the task is mobile-specific.
- For local dev servers, confirm the server is running and the port is what you expect.

## Driving the browser

A typical verification session:

1. **Launch with a real-looking user agent** when the target is the public internet; an unrealistic UA flags automation traffic.
2. **Set a sane viewport** (e.g., 1366×768 desktop, 390×844 iPhone-ish).
3. **Navigate and wait for the right signal.** Prefer waiting for a specific selector or network-idle over arbitrary sleeps.
4. **Capture evidence immediately** after the wait condition succeeds, before any interaction perturbs the state.
5. **Interact deliberately.** One click at a time, with a wait between actions; re-screenshot after each meaningful state change.
6. **Read the console and network panels** for unexpected errors, 4xx/5xx responses, or slow requests.
7. **Close the browser cleanly** when done. Long-running browser sessions leak memory and hold ports.

## What evidence to record

For a verification task, deliver:

- A full-page or viewport screenshot of each meaningful state.
- The console log, filtered to warnings/errors.
- Any non-2xx network response with the URL, status, and a short response body excerpt.
- A short narration: "Navigated to X, observed Y, clicked Z, observed W."

For a UI bug repro, also record:

- The exact reproduction steps the user can follow.
- Viewport size and (where relevant) device pixel ratio.
- Whether the bug reproduces on first load vs after interaction.

## Login-gated pages

- Prefer programmatic auth (API token, magic link) over UI login.
- If UI login is the only path, the user must provide credentials explicitly for this run. Never reuse credentials outside the session.
- Do not store credentials in the session log, screenshot, or returned output.

## Performance and politeness

- Throttle to one navigation per few seconds when touching shared infra.
- Respect `robots.txt` for public sites you are inspecting at any volume.
- Cancel navigations if a page exceeds a reasonable timeout (e.g., 30s); the page is broken or rate-limiting you.
- Do not retry forever on failure. Retry once with a longer timeout, then escalate.

## Common failure modes

- **Selector not found.** Page changed, or you are waiting before render. Take a screenshot to see actual state; adjust the selector.
- **Click does nothing.** The element is offscreen, covered by a modal, or in a shadow DOM. Scroll into view or pierce the shadow root.
- **Headless detection.** Some sites detect headless Chrome and serve a different page. Use a non-headless mode or a fingerprint-realistic configuration only when authorized.
- **Cross-origin iframe blocking.** Iframes you do not own cannot be inspected; the page must offer the data outside the iframe or the task is infeasible.

## Anti-patterns

- Long unsupervised browser sessions that drift from the original task.
- Scraping behind authentication you do not own.
- Captioning a screenshot with "looks good" without saying what state was loaded and what selectors confirmed it.
- Treating a passing screenshot as proof of correctness across viewports you did not actually test.
````

## 中文译文

# 智能体浏览器

使用受控浏览器验证行为、采集证据或从静态抓取无法访问的网页中提取信息，例如单页应用、需要登录的页面和动态内容。本技能用于有人监督的验证，不用于无人值守抓取。

## 适用场景

- 需要对已部署页面或本地开发服务器截图，以确认 UI 变更。
- 需要读取 `curl` / `wget` 无法获取的 JavaScript 渲染内容。
- 用户报告 UI 错误，需要交互复现并采集控制台错误、网络请求或布局状态。
- 需要走查一段简短流程（加载页面、点击、观察），以验证验收标准。

## 不适用场景

- 页面可通过静态 HTML 访问。此时使用 `curl` / HTTP 获取内容，成本更低、更快且更可靠。
- 任务要求无人值守的大规模抓取。应使用带有速率限制、`robots.txt` 处理和真实用户代理策略的专用抓取器，而不是本技能。
- 网站要求登录，但你没有获准使用的凭据，或其服务条款禁止自动化。
- 网站涉及银行、医疗、政府等敏感账户，自动化可能导致锁定账户或违反规定。

## 启动浏览器前

- 确认 URL，以及导航后应满足的状态。
- 决定需要哪些证据：整页截图、视口截图、控制台日志、网络跟踪、HTML 快照或提取文本。
- 决定相关视口尺寸（移动端还是桌面端）。若任务未特别要求移动端，默认使用桌面尺寸。
- 对于本地开发服务器，确认服务器正在运行，且端口符合预期。

## 操作浏览器

典型验证过程：

1. 访问公共互联网时，使用看起来真实的用户代理；不自然的用户代理可能会被识别为自动化流量。
2. 设置合理的视口，例如桌面端 `1366×768`，或近似 iPhone 的 `390×844`。
3. 导航并等待正确的信号。优先等待特定选择器或网络空闲，不要任意等待固定时长。
4. 等待条件满足后立即采集证据，避免后续交互改变状态。
5. 有目的地进行交互。每次只点击一处，操作之间等待；每次重要状态变化后重新截图。
6. 查看控制台和网络面板，检查意外错误、4xx/5xx 响应或缓慢请求。
7. 完成后正常关闭浏览器。长时间运行的浏览器会占用内存和端口。

## 应记录的证据

验证任务应提供：

- 每个重要状态的整页或视口截图。
- 过滤后的控制台警告和错误日志。
- 每个非 2xx 网络响应的 URL、状态码和简短响应正文摘录。
- 简短操作记录，例如：“导航到 X，看到 Y，点击 Z，随后看到 W。”

复现 UI 问题时，还应记录：

- 用户可按步骤复现问题的精确操作流程。
- 视口尺寸，以及相关时的设备像素比。
- 问题是在首次加载时出现，还是交互后才出现。

## 需要登录的页面

- 优先使用 API 令牌或 magic link 等程序化认证方式，而不是 UI 登录。
- 若只能通过 UI 登录，用户必须明确提供本次运行所需的凭据。不得在本会话之外复用凭据。
- 不要把凭据保存到会话日志、截图或返回结果中。

## 性能与礼貌

- 访问共享基础设施时，将导航频率控制在每几秒一次。
- 以任何规模检查公共网站时，都应遵循其 `robots.txt`。
- 若页面超过合理时限（30 秒）仍未加载，应取消导航；页面可能故障或正在限流。
- 不要无限重试。使用更长超时时间重试一次后仍失败，就应升级处理。

## 常见失败模式

- **找不到选择器。** 页面可能已变化，或等待发生在渲染前。先截图检查实际页面，再调整选择器。
- **点击没有效果。** 元素可能在屏幕外、被模态框遮住，或位于 shadow DOM 中。将其滚动到可视区域，或进入 shadow root。
- **检测到无头浏览器。** 某些网站会检测无头 Chrome 并返回不同页面。只有在获得授权时，才使用非无头模式或符合真实浏览器特征的配置。
- **跨源 iframe 阻止访问。** 无法检查不属于你的 iframe；网站必须在 iframe 外提供数据，否则该任务无法完成。

## 反面模式

- 长时间无人监督地运行浏览器，导致操作偏离原任务。
- 抓取你无权访问的认证页面。
- 仅用“看起来没问题”描述截图，却不说明加载了什么状态、哪些选择器确认了结果。
- 把某个视口的截图通过当成未实际检查的其他视口也正确的证明。

## 另请参阅

- [可选技能](../../optional.md)——按类别分组的所有可选目录技能。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、技能指派和故障排查。
- [应用内置目录](../../../skills.md#3-app-shipped-catalog)——内置和可选目录技能的版本管理和更新机制。
- [技能指南](../../../../guides/org/skills.md)——UI 操作指南和内置技能目录概览。
