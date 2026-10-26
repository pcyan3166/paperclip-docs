---
paperclip_version: v2026.1012.0
seo_title: Cloud Sync（已退役）
seo_description: 本版本已移除 Cloud Sync（可将本地公司推送到 Paperclip Cloud）。本文说明替代方案，以及旧链接现在的去向。
---

# Cloud Sync（已退役）

> **此版本已移除。** Paperclip v2026.1012.0 已不再提供 Cloud Sync。**Cloud upstream**（Cloud 上游）页面、**Settings → Experimental**（设置 → 实验性功能）中的 **Cloud Sync** 开关、`paperclipai cloud push` 命令以及 cloud-upstream API 均已移除。若要将公司迁移到其他实例（包括托管实例），请改用公司的 **Export**（导出）和 **Import**（导入）。请参阅[备份和恢复公司](../how-to/back-up-and-restore-a-company.md)。

## 此功能原先的用途

在旧版本中，**Cloud Sync** 会将本地运行的 Paperclip 实例连接到 Paperclip Cloud 上游。你可以预览推送、解决冲突、将本地公司（智能体、项目、问题、技能、可移植文件）推送到 Cloud 堆栈，然后选择在 Cloud 端唤醒哪些智能体、例程和监控器。

## 现在会看到什么

如果你打开旧链接或书签中的 Cloud 上游页面地址（`/company/settings/cloud-upstream`），Paperclip 会将你带到 **Export** 页面：

![在此版本中，旧的 Cloud upstream 地址会打开 Export 页面](../../user-guides/screenshots/light/experimental/cloud-upstream.png)

在该页面中，你可以将公司导出为可移植软件包，然后导入到另一个实例。Import/Export 可迁移与 Cloud Sync 相同的内容，并支持本地到本地、通过 URL 或从 GitHub 导入；它不需要保持上游连接，也不需要启用实验性功能。

## 后续阅读

- [备份和恢复公司](../how-to/back-up-and-restore-a-company.md)：替代工作流程。
- [公司导出与导入](../administration/company.md)：详细了解 Export 和 Import 页面。
- [实验性功能概览](overview.md)
