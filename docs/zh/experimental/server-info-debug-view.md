---
paperclip_version: v2026.1012.0
seo_title: 服务器信息调试视图
seo_description: 直接在应用中查看服务器当前运行的提交。此功能会在账户抽屉中添加 Server 区域，方便支持和开发工作。
---

# 服务器信息调试视图

在许多支持和开发对话中，第一个问题都是：“这台服务器实际运行的是哪个提交？”**Server Info Debug View**（服务器信息调试视图）可让你直接在应用中找到答案：它会在账户抽屉中添加 **Server** 区域，显示服务器最近一次重启时间、当前运行的提交，以及检出目录是否包含本地更改。

![账户抽屉中的 Server 区域](../../user-guides/screenshots/light/experimental/server-info.png)

## 启用此功能

1. 前往 **Settings → Experimental**（设置 → 实验性功能）。
2. 开启 **Server Info Debug View**，说明文字为：*“在账户抽屉中显示 Server 区域，并提供当前服务器重启时间和正在运行的提交。”* 此设置位于页面下方的 **Paperclip Developer Mode** 分组中。

## 使用方法

点击侧边栏底部的头像打开账户抽屉。**Server** 区域位于抽屉底部，在菜单行（**Settings**、在你有权邀请人员时显示的 **Invite**、**Documentation**、**Switch to dark mode** / **Switch to light mode** 切换项，以及在要求登录的实例上显示的 **Sign out**）下方。该区域显示三行信息：

- **Last restarted**（最近重启时间）——如果有开发服务器重启时间，则显示该时间；否则显示进程启动时间。
- **Running commit**（当前运行的提交）——显示短 SHA 和提交说明；没有 Git 元数据的部署会显示 *“无法获取提交信息”*。
- **Checkout state**（检出状态）——显示 *“检出目录干净”* 或 *“存在本地更改（N 项已暂存、N 项未暂存、N 项未跟踪）”*。

每次打开抽屉时都会重新获取这些值，因此服务器重启后会立即反映出来。在开发服务器实例（`pnpm dev:once`）上，只要抽屉保持打开，此区域还会每隔几秒实时轮询一次。

## 此功能的用途

这是一种轻量级的问题排查工具，在本地和开发实例上尤其有用：无需通过 shell 登录机器，即可快速确认重启是否加载了你的更改、服务器是否运行在预期提交上，以及检出目录是否有未提交更改。它与[空闲时自动重启开发服务器](auto-restart-dev-server.md)配合使用很方便。

## 关闭此功能时

此区域不会渲染，也不会发生轮询。无论是否开启，都不会涉及存储数据。

## 后续阅读

- [空闲时自动重启开发服务器](auto-restart-dev-server.md) — 配套标记，用于保持开发服务器启动状态为最新。
- [实验性功能概览](overview.md)
