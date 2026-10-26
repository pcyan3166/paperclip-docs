---
paperclip_version: v2026.525.0
seo_title: 工作区差异查看器插件
seo_description: 查看智能体在工作区中所做的确切更改：文件移动和新增、已暂存与未暂存内容的区别，以及逐行补丁。
---

# 工作区差异查看器

如果你希望在不离开 Paperclip 控制面板的情况下查看智能体在工作区中实际更改了什么——哪些文件移动了、添加了什么、已暂存与未暂存内容有何区别，以及逐行补丁——`@paperclipai/plugin-workspace-diff` 插件就能实现这一点。它会在执行工作区和项目工作区中添加一个**更改**标签页，使用运行 Paperclip 的主机上的 Git 在本地计算差异，并通过 `@pierre/diffs` 渲染。

本页面向希望启用、配置或了解此“更改”标签页限制的**操作员和开发者**。

> Workspace diff 插件和插件运行时目前都处于 alpha 阶段。Paperclip 版本升级可能带来不兼容变化；依赖此插件时请固定版本。

---

## 快速安装

此插件以已发布的 npm 包形式提供，因此任何 Paperclip 安装方式（Docker、systemd、裸机部署等）都可以在配置了 Paperclip CLI 的主机上运行一条命令完成安装：

```sh
paperclipai plugin install @paperclipai/plugin-workspace-diff
```

也可以在控制面板中依次打开：**设置 → 插件 → 安装插件**，粘贴 `@paperclipai/plugin-workspace-diff` 并提交。

在 `paperclipai plugin examples` 中不会显示此插件——该命令只列出四个内置参考**示例**插件，而非所有第一方插件。安装“工作区差异查看器”时请使用完整包名。

如果你使用的是 monorepo 检出版本，请参阅[本地开发插件](../../how-to/develop-a-plugin-locally.md)，并将安装器指向 `packages/plugins/plugin-workspace-diff`。

首次启用时，插件会申请范围较小且明确的能力集合：`ui.detailTab.register`、`execution.workspaces.read` 和 `project.workspaces.read`。插件不会写入工作区——差异内容均为只读——也不会管理自己的数据库命名空间、智能体或例程。批准之前，请查看插件详情页中的权限卡片。

---

## 安装后会获得什么

安装并启用插件后，Paperclip 界面中会出现一项内容：

- **更改标签页**：显示在每个执行工作区和项目工作区的详情页中（插槽 ID 为 `workspace-changes-tab`，插槽类型为 `detailTab`，顺序为 `25`，实体类型为 `execution_workspace` 和 `project_workspace`）。打开任意工作区，即可在详情标签行中看到此标签页。

此标签页名称为**更改**（插件的 `displayName` 为 “Workspace Changes”）。其中提供熟悉的文件树加补丁布局：列出已更改文件及其 Git 状态（新增、修改、删除、重命名、复制、类型更改、未跟踪、未知），并为每个文件显示分别呈现**已暂存**、**未暂存**、**HEAD** 和**未跟踪**内容的补丁视图。

插件会直接在工作区路径上运行 Git。它没有后台工作进程、托管智能体、例程或迁移脚本；只有“更改”标签页打开时才会调用的一个小型数据端点。

---

## 差异的计算方式

插件工作进程会提供一个数据端点（`workspace-diff`）。“更改”标签页会携带工作区 ID 和查询参数调用该端点。工作进程会：

1. 根据实体类型，通过 SDK 的 `executionWorkspaces.get` 或 `projects.listWorkspaces` 查询工作区。
2. 解析一个**基准引用**：使用工作区自己的 `baseRef`，或者对于执行工作区使用父项目工作区的 `defaultRef` / `repoRef`。也可以在查询中传入明确的 `baseRef`。
3. 确认工作区的 `cwd` 是 Git 仓库内真实存在的绝对目录（通过 `O_NOFOLLOW` 检查和 `git rev-parse --show-toplevel` 验证）。
4. 使用 10 秒超时和 2 MiB 列表缓冲区运行 `git`，枚举已更改文件并生成补丁。

通过 `view` 查询参数支持两种视图：

- `working-tree`（默认值）——相对于 `HEAD` 查看已暂存、未暂存和未跟踪的更改。
- `head`——比较 `HEAD` 与已解析的基准引用。

如果请求 `head` 视图时没有提供基准引用，“更改”标签页会回退到 `working-tree`，而不会失败；即使尚未选择基准，也始终可以渲染差异。

还可以传入 `paths` / `path`，将差异范围限制在指定文件；并通过 `includeUntracked`（默认 `true`）控制是否包含未跟踪文件。

---

## “更改”标签页中的控件

标签页工具栏提供两个切换项和一个基准引用输入框：

- **并排 / 统一**——在并排补丁视图和内联补丁视图之间切换。默认采用并排视图。
- **工作树 / 与引用比较**——选择 `working-tree`（默认值）检查尚未提交的更改，或选择**与引用比较**来比较 `HEAD` 和已解析的基准引用。
- **基准引用**——“与引用比较”视图选中时，会显示在视图切换项旁边。如果留空，标签页会预填由工作进程解析的工作区 `defaultBaseRef`，并自动切换到**与引用比较**视图，因此常见情况无需手动切换即可使用。手动修改任一控件后，本次会话会保留你的选择。

向下滚动时，工具栏会**固定在标签页顶部**，因此查看大型差异时仍可使用切换项和基准引用输入框。

---

## 需要了解的限制

插件设置了硬上限，避免异常庞大的差异导致控制面板无响应。默认值来自 `WORKSPACE_DIFF_CAPS`：

| 上限 | 默认值 |
|---|---|
| `maxFiles` | 200 个已更改文件 |
| `maxFileBytes` | 每个文件 512 KiB（跳过二进制文件和超大文件） |
| `maxPatchBytes` | 每个文件的补丁最多 256 KiB |
| `maxTotalPatchBytes` | 所有补丁合计最多 1 MiB |

达到上限时不会直接失败；插件会在可计算的结果旁附上结构化**警告**。可能出现的警告代码包括：

- `base_ref_missing`、`base_ref_invalid`——无法使用已解析的基准引用。
- `binary_file`、`file_oversized`——文件无法以内联补丁方式呈现，因此已跳过。
- `file_count_truncated`、`patch_truncated`——超出总量限制。
- `git_command_failed`——Git 命令返回非零退出码（响应中会包含标准错误）。
- `missing_cwd`、`non_git_workspace`、`workspace_path_invalid`——工作区路径不可用。
- `path_filter_invalid`、`symlink_target_outside_workspace`——触发路径安全防护。

这些警告会显示在“更改”标签页中，让操作员明确知道内容为何缺失，而不是面对空白列表。

---

## 提示和常见用法

- **用于审核智能体编写的代码。** 智能体在执行工作区完成工作后，打开“更改”标签页即可在合并或移交前查看补丁；整个流程与代码审核界面类似，无需离开 Paperclip。
- **在项目工作区设置 `baseRef`。** 插件会回退使用项目工作区的 `defaultRef`，再回退到 `repoRef`。因此配置一次后，该项目下所有执行工作区都会自动获得合适的差异比较基准。
- **工作区必须是真实的 Git 仓库。** 不支持非 Git 工作区；此时会显示 `non_git_workspace` 警告。如果执行工作区只是普通文件夹，“更改”标签页会提示你初始化 Git，或不要在该工作区使用插件。
- **插件只读取内容。** 它不会暂存、提交、重置或以其他方式修改仓库。如果要处理差异，请在主机 shell 中操作，或使用其他插件。

---

## 相关内容

- [管理 → 插件](../../administration/plugins.md)——面向操作员的插件管理器。
- [操作指南 → 本地开发插件](../../how-to/develop-a-plugin-locally.md)——让 Paperclip 使用本地检出的插件代码。
- [参考 → 插件 SDK](./sdk.md)——插件开发接口，可用于扩展差异查看器或编写自己的详情标签页插件。
