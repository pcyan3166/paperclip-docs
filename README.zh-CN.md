# Paperclip 文档

[English](README.md) | 简体中文

本仓库是 [Paperclip](https://github.com/paperclipai/paperclip) 文档网站的源文件，包含用户指南、API 与 CLI 参考、适配器文档和部署说明。

**在线阅读：**[docs.paperclip.ing](https://docs.paperclip.ing/)

## 文档内容

- **指南（Guides）**：快速入门、核心概念和日常使用。
- **API 参考**：控制平面端点和请求数据结构。
- **CLI 参考**：`paperclip` 命令行工具。
- **适配器（Adapters）**：插件开发说明和集成目录。
- **部署（Deploy）**：自托管、升级和运维指南。

网站是一个静态外壳，直接渲染 `docs/` 中的 Markdown 文件。阅读原始文档不需要框架或构建步骤。

## 构建与部署

在本地构建发布包：

```sh
npm run docs:build
```

本地预览请通过 HTTP 服务打开，不能直接双击 `site/index.html`（`file://`）或 `.site/index.html`。浏览器会阻止本地页面读取站点所需的 `content.json` 和 Markdown 文件。构建后运行：

```sh
npm run docs:serve
```

然后访问 <http://localhost:4321/>。

Cloudflare Pages 通过 GitHub 直接连接到 `paperclipai/paperclip-docs`，本仓库通常不使用 Wrangler 发布：

- 推送到 `main` 会部署生产站点：[https://docs.paperclip.ing/](https://docs.paperclip.ing/)。
- 推送到其他分支会创建 Cloudflare Pages 预览/金丝雀部署。
- Cloudflare 会为每次部署创建金丝雀网址，例如 `https://92b9a99c.paperclip-docs-74t.pages.dev`。请以 Cloudflare Pages 部署列表或 GitHub 中对应分支和提交的部署/检查所显示的网址为准。

自动部署并非总能成功；曾出现推送到 `main` 后部署没有触发的情况。面向读者的改动上线后，请检查线上站点是否已更新；若内容仍旧，可手动重新发布：

```sh
npm run docs:build
npx wrangler pages deploy .site --project-name paperclip-docs --branch main
```

## 参与贡献

发现拼写错误、失效链接或可以改进的内容时，可使用文档页脚提供的两种方式：

- **建议修改（Suggest an edit）**：在 GitHub 网页编辑器中打开对应 Markdown 文件，修改后提交拉取请求。
- **报告问题（Report an issue）**：创建预填了页面网址的[文档反馈问题](.github/ISSUE_TEMPLATE/03-docs-feedback.yml)。

较大的改动可 Fork 本仓库，然后向 `main` 分支提交拉取请求。截图位于 `docs/user-guides/screenshots/{light,dark}/`；替换界面截图时应同时提供浅色和深色主题版本。

## 社区

- [Discord](https://discord.gg/m4HZY7xNG3)
- [GitHub 讨论区](https://github.com/paperclipai/paperclip/discussions)

## 许可证

Copyright (c) 2026 Paperclip Community.

Paperclip **软件**为开源软件；许可证请见主 [paperclip 仓库](https://github.com/paperclipai/paperclip)。

本仓库中的**文档**采用[知识共享署名—非商业性使用—禁止演绎 4.0 国际许可协议（CC BY-NC-ND 4.0）](https://creativecommons.org/licenses/by-nc-nd/4.0/)授权。你可以在注明出处的前提下为非商业目的阅读和分享文档。任何形式的商业使用都需事先取得版权所有者的书面许可。完整条款请见 [`LICENSE`](LICENSE)。

### 未列入导航的托管版 Beta 指南

`docs/hosted-beta.md` 由 `site/build-release.mjs` 构建到 `/hosted-beta/`。该页面复用完整的文档外壳并初始化页面工具，但不会加入导航或搜索清单。

请勿将该页面加入 `site/content.json`：它有意不出现在导航、搜索、首页列表、上一页/下一页链接或站点地图中，并带有 `noindex` 元标签。该页面可公开访问，并非访问受限文档。

运行 `node scripts/verify-unlisted-beta.mjs` 可验证根路径和子路径构建以及该页面不会进入发现列表。可使用 `npm run docs:build` 构建预览包，并用 `npm run docs:serve` 预览。
