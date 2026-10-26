---
paperclip_version: v2026.609.0
seo_title: Cloud 同步命令（已移除）
seo_description: Cloud Sync 命令已移除。请改用完整保真度的公司导入和导出功能，它可以捕获 Cloud Sync 推送的同一可移植数据包。
---

# Cloud 同步命令（已移除）

> **已移除。** 上游已停用主机到主机的 Cloud Sync。`paperclipai cloud` 命令组（包括 `cloud connect` 和 `cloud push`）已不存在，实验性设置 `enableCloudSync` 也已一并移除。本页暂时保留，供从旧链接或旧版本进入的用户查看指引；未来版本会删除此页。

## 替代方式

要在不同 Paperclip 实例之间迁移公司，请使用完整保真度的**公司导入/导出**功能。它会捕获与 Cloud Sync 过去推送相同的可移植数据包，包括公司清单、智能体、项目、任务、技能及其以 Markdown 保存的文件；不同之处在于，数据包作为文件从一个实例导出后再导入另一个实例，无需管理云端基础设施、发现握手或签名密钥。

- [`paperclipai company export` / `paperclipai company import`](./company.md)：CLI 可移植性命令。
- [备份与恢复公司](../../how-to/back-up-and-restore-a-company.md)：分步操作指南。

如果你依赖脚本化的 `cloud push`，等效流程是在源实例上执行 `company export`，再在目标实例上执行 `company import`。两者都支持自动化所需的 `--json`。

## 另请参阅

- [公司命令](./company.md)：现在用于公司迁移的导入/导出流程。
- [通用选项](./common-options.md)：共享客户端标志和 API 基础地址解析方式。
- [输出与脚本](./output-and-scripting.md)：在自动化中使用 `--json` 和退出代码。
