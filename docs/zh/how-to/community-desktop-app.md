---
paperclip_version: v2026.707.0
seo_title: 社区桌面应用（非官方）
seo_description: 一款非官方的 macOS 应用，封装了完整的 Paperclip 实例——拖入“应用程序”文件夹即可运行，无需终端。了解它是什么，以及依赖它之前需要注意的事项。
---

# 社区桌面应用（非官方）

> **警告：** Paperclip 桌面应用是一个**由社区维护的非官方项目**。它并非由 Paperclip 团队构建、发布或提供支持。该项目位于 [`aronprins/paperclip-desktop`](https://github.com/aronprins/paperclip-desktop)，其打包和版本管理与 Paperclip 本身是分开的。如果您遇到应用外壳（安装、启动、更新）方面的问题，请在该仓库中报告——而不是在主 Paperclip 问题跟踪器中。

社区桌面应用将完整的 Paperclip 实例封装在一个标准的 macOS 应用程序中：将其拖入“应用程序”文件夹，打开它，Paperclip 即可运行——无需终端。有些人喜欢这种便利性，本页面汇总了安装、运行和更新该应用所需的一切信息。

**官方支持的运行 Paperclip 方式**是 [终端和服务器 / VPS 安装](../guides/getting-started/installation.md)。本文档中的其他所有内容均假设您已采用其中一种方式。不过，一旦应用运行起来，产品体验是完全一致的——无论实例是如何启动的，公司、智能体、例程和预算的工作方式都相同。

---

## 安装应用

### 步骤 1 — 确认您的 Mac 类型

该应用根据 Mac 的芯片提供两个版本。如果您不确定自己使用的是哪种：

1. 点击屏幕左上角的 **Apple 菜单** ()
2. 点击 **关于本机**
3. 查看标有 **芯片** 或 **处理器** 的那一行
   - 如果显示 **Apple M1**、**M2**、**M3** 或 **M4** — 您使用的是 Apple Silicon
   - 如果显示 **Intel Core** — 您使用的是 Intel Mac

### 步骤 2 — 下载安装程序

前往社区 [发布页面](https://github.com/aronprins/paperclip-desktop/releases/latest)，下载适用于您 Mac 的安装程序：

| 我的 Mac 是… | 下载此文件 |
|---|---|
| Apple Silicon (M1/M2/M3/M4) | `Paperclip.Desktop-[version]-arm64.dmg` |
| Intel | `Paperclip.Desktop-[version].dmg` |

> **注意：** 文件名中的版本号将与最新发布的版本匹配。如果您看到多个 `.dmg` 文件，名称中包含 `arm64` 的是为 Apple Silicon 准备的。没有该标识的是为 Intel 准备的。

### 步骤 3 — 安装并打开

1. 打开您下载的 `.dmg` 文件——它将像磁盘映像一样挂载并打开一个窗口
2. 将 **Paperclip** 图标拖入 **应用程序** 文件夹
3. 从您的应用程序文件夹中打开 Paperclip，或按 **Cmd+Space** 并输入 **Paperclip**

> **警告：** 首次打开应用时，macOS 可能会显示一个提示，称无法验证开发者。这是 macOS 对从 App Store 之外下载的应用的标准警告——由于这是社区构建版本，在点击 **打开** 之前，您应自行判断是否信任该应用。

首次启动需要 10–30 秒，因为应用在后台启动其捆绑的 Paperclip 服务器。后续启动速度会更快。

---

## 选择本地或远程模式

应用打开时，会询问您希望如何连接：

**本地模式** 直接在您的 Mac 上运行完整的 Paperclip 实例。您的智能体在您的机器上运行，所有数据保持本地化，不会发送到外部服务器（除了您的智能体向 Anthropic 或 OpenAI 发起的 API 调用）。

**远程模式** 将应用连接到运行在另一台机器上的 Paperclip 实例——可以是团队服务器、云主机或同事的电脑。应用成为该实例的一个轻量级 UI 外壳。

选择本地模式后，应用完成启动并引导您进入引导流程。接下来，请遵循常规指南：在智能体能够执行任何工作之前，您需要从 Anthropic 或 OpenAI 获取一个 [API 密钥](../guides/getting-started/installation.md)，然后您可以 [创建您的第一个公司](../guides/getting-started/your-first-company.md)。

### 连接到您部署的服务器

如果您的团队运行 [服务器 / VPS 安装](../guides/getting-started/installation.md)，任何人都可以将应用指向该服务器：

1. 安装应用（按上述步骤操作）。
2. 在首次启动屏幕上，选择 **远程**。
3. 输入您的实例 URL（例如 `https://paperclip.example.com`）。
4. 使用您在该实例上的账户登录。

以这种方式连接的每个人都会看到相同的公司、智能体和例程——服务器是数据的唯一真实来源。

---

## 更新

应用会自动更新自身；您无需运行任何命令。

1. 启动时，它会检查 GitHub Releases 是否有 [`paperclip-desktop`](https://github.com/aronprins/paperclip-desktop/releases) 的新版本。
2. 如果存在新版本，它会提示您下载。请批准下载。
3. 更新将在 **您下次退出应用时** 安装。退出并重新打开应用即可进入新版本。

> **注意：** 自动下载是可选的（您需要点击提示），但一旦批准下载，退出时安装是自动执行的。如果您跳过了下载提示，应用将在下次启动时再次提供该选项。

**若要强制立即检查**，请打开 macOS 菜单栏中的 **Paperclip** 菜单并点击 **检查更新**。

**若要验证当前版本**，请打开 **Paperclip** 菜单 → **关于 Paperclip**，或将鼠标悬停在 UI 左侧边栏底部的小 **`v`** 徽章上。

> **提示：** 社区发布版本遵循与 Paperclip CLI 相同的日历版本，但它们单独打包，可能会比 CLI 晚一两天。如果您需要在修复发布后立即获取，[终端安装](../guides/getting-started/installation.md) 始终是最新的。

---

## 故障排除

**应用从不提示更新** — 请确保您已连接网络，且 [paperclip-desktop/releases](https://github.com/aronprins/paperclip-desktop/releases) 上确实存在新版本。如果发布源为空，更新器会记录警告并静默跳过。在新发布日重新打开应用。

**您已更新但想回滚** — 从 [发布页面](https://github.com/aronprins/paperclip-desktop/releases) 下载上一个安装程序，并覆盖安装当前应用。对于代码而言，回滚是安全的，但单向迁移可能已经重写了您的数据库——如果是这种情况，请恢复更新前的数据库备份（参见 [备份和恢复公司](./back-up-and-restore-a-company.md)）。

**其他任何问题** — 对于应用外壳本身的问题，请在 [社区仓库](https://github.com/aronprins/paperclip-desktop/issues) 中提交 issue。对于 Paperclip 产品本身的问题（智能体、例程、预算、UI），本文档的其余部分和 [主仓库](https://github.com/paperclipai/paperclip) 照常适用。

---

## 相关

- [安装](../guides/getting-started/installation.md) — 官方支持的终端和服务器 / VPS 安装。
- [更新 Paperclip](./update-paperclip.md) — 更新受支持的安装路径。
- [备份和恢复公司](./back-up-and-restore-a-company.md) — 无论您以何种方式运行 Paperclip，都可以对数据进行快照。
