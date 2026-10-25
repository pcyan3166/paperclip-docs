---
paperclip_version: v2026.1005.0
seo_title: Browser Use Cloud 连接器
seo_description: 将网站任务委派给 Browser Use Cloud，在 Paperclip 中查看实时浏览器，并选择已保存的配置文件、智能体访问权限和支出限额。
---

# Browser Use Cloud

您可以为智能体分配网站工作，并在任务的 **Browser**（浏览器）选项卡中查看托管浏览器。Browser Use Cloud 负责执行浏览任务；Paperclip 负责控制访问权限、操作审批以及记录运行成本。

Browser Use Cloud 不需要启用实验性设置。

## 连接前准备

- 来自 [Browser Use 设置](https://cloud.browser-use.com/settings) 的 Browser Use Cloud 项目 API 密钥，需具备浏览器工作的读取和写入权限。
- 用于首次浏览测试的一个智能体和一个任务。
- 对于自托管实例，需要能够访问 `api.browser-use.com` 的出站 HTTPS 连接。您的浏览器还需要能够访问 `live.browser-use.com`；自定义内容安全策略（Content Security Policy）必须在 `frame-src` 中允许该源。

API 密钥存储在 Paperclip 的密钥库中。它不会传递给智能体，也不会显示在实时浏览器面板中。

## 连接 Browser Use Cloud

1. 打开 **Connectors**（连接器）并选择 **Browser Use Cloud**。
2. 阅读主按钮上方的访问权限说明。选择 **Change**（更改）以选择个人凭据或限制哪些智能体可以使用该凭据。
3. 输入 **API key**（API 密钥）并完成设置。连接检查会读取已保存的配置文件；它不会启动付费的浏览器工作。
4. 打开 **Permissions**（权限）。在 **Browser settings**（浏览器设置）下，如果您希望设置上限，请设置 **Maximum cost per browser run (USD)**（每次浏览器运行的最大成本（美元））。
5. 审查 **Allowed saved profiles**（允许的已保存配置文件），然后选择 **Save browser settings**（保存浏览器设置）。

默认使用全新浏览器。选择配置文件可让符合条件的智能体使用其已保存的网站登录信息。请在 Browser Use Cloud 中管理这些配置文件；Paperclip 不创建或导入它们。凭据所有者或共享连接管理器可以更改这些设置。

## 选择访问权限

暴露的操作使用常规的 **Allowed**（允许）、**Ask first**（先询问）和 **Off**（关闭）权限。由于启动或继续浏览任务可能会更改网站，因此 Paperclip 对这些操作采取保守的分类方式，同时包括取消操作和结束会话操作。

| 操作 | 您可以执行的操作 |
| --- | --- |
| `browser_start` | 启动托管任务，可选使用允许的配置文件和更低的成本上限。 |
| `browser_status` | 读取进度、结果和记录的运行成本。 |
| `browser_continue` | 为空闲的对话分配更多工作。 |
| `browser_cancel` | 停止托管的智能体工作，同时短暂保留浏览器可用。 |
| `browser_end` | 停止工作并关闭对话的浏览器。 |
| `browser_sessions` | 列出属于此任务、智能体和凭据授权的对话。 |
| `browser_profiles` | 列出此凭据授权允许的已保存配置文件。 |

浏览器工作需要实际的任务和智能体运行。连接器的测试界面可以列出配置文件，但不能启动付费浏览。符合条件的运行还会自动获得连接器的使用技能。

运行的成本上限采用适用的最低限制：请求的上限、您保存的浏览器上限，以及剩余的公司、智能体或项目硬性预算。记录的提供商运行费用会汇入 Paperclip 的成本和预算记录。它们不是实时发票，也不是跨并发运行的共享预留。

## 查看和继续工作

就绪的浏览器会在任务的右侧面板中打开。您可以与页面交互、切换选项卡，并从其任务卡片重新打开浏览器。关闭面板选项卡会隐藏查看器。

- **Stop browsing**（停止浏览）会取消托管智能体的工作。
- 在 **Browser options**（浏览器选项）中，**Close browser**（关闭浏览器）会停止浏览器本身，而 **Reconnect view**（重新连接视图）会重新加载断开的显示。
- **Keep browser open**（保持浏览器打开）会延长空闲窗口，具体取决于提供商的过期时间。接近结束时，倒计时旁边会出现 **Keep browsing**（继续浏览）。
- **Browser size**（浏览器大小）默认设置为 **Fit to pane**（适应面板）。如果您希望远程页面在面板大小改变时保持该视口大小，请选择固定大小。

后续工作应放入普通任务消息中，以便智能体可以继续同一对话。离开查看器会启动十分钟的空闲窗口。任务取消、访问权限丢失、智能体暂停、预算阻止以及类似的干扰会请求清理相关的托管工作。

实时查看器仅供符合条件的公司人类成员使用。请勿将其 URL 复制到任务、评论或工件中；它携带对会话的访问权限。

## 尝试使用

为符合条件的智能体创建一个任务：

```txt
Use Browser Use Cloud to visit paperclip.ing and report the main navigation links.
Use a fresh browser. Do not sign in, submit forms, or change anything.
```

在 **Browser**（浏览器）中查看页面，将答案与网站进行比较，并检查任务的连接器调用和成本记录。

> **注意：** 这是示例任务，并非记录的测试结果。托管工作可能会产生提供商费用。

## 故障排除和限制

| 问题 | 检查项 |
| --- | --- |
| 无法从连接器测试中启动浏览器操作 | 为符合条件的智能体分配一个真实任务。 |
| 已保存的登录信息不可用 | 允许此任务使用的凭据访问其配置文件。 |
| 实时视图为空白或断开连接 | 使用 **Reconnect view**（重新连接视图）并检查对查看器源的访问权限。 |
| 另一个查看器控制大小 | 如果您希望此面板控制视口，请使用 **Fit to this pane instead**（改为适应此面板）。 |
| 移除时提示关闭挂起 | 等待提供商关闭，然后重试移除连接。 |

此连接器委派托管任务。它不暴露任意的浏览器控制命令，不导入录制或文件，也不提供本地浏览器会话。如果付费启动的结果不确定，请在启动替代工作之前检查 Browser Use Cloud；Paperclip 不会自动重试。

## 相关指南

- [连接器访问权限如何工作](access-model.md)
- [设置操作权限](action-permissions.md)
- [Browser Use Cloud API 文档](https://docs.browser-use.com/cloud/api-v4-overview)

## 来源

- [连接器定义](https://github.com/paperclipai/paperclip/blob/467125fafb47a8520856504fecc48d6e32055db1/packages/shared/src/app-definitions/browser-use-cloud.json) — 设置方法和凭据字段。
- [浏览器设置](https://github.com/paperclipai/paperclip/blob/467125fafb47a8520856504fecc48d6e32055db1/ui/src/pages/apps/app-detail/BrowserUseSettingsPanel.tsx)、[浏览器控件](https://github.com/paperclipai/paperclip/blob/467125fafb47a8520856504fecc48d6e32055db1/ui/src/components/task-side-panel/TaskBrowserFooter.tsx) 和 [浏览器服务](https://github.com/paperclipai/paperclip/blob/467125fafb47a8520856504fecc48d6e32055db1/server/src/services/browser-use.ts) — 配置文件、查看器行为、生命周期和成本。
