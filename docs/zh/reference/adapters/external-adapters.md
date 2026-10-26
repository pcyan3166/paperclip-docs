---
paperclip_version: v2026.609.0
seo_title: 外部适配器
seo_description: 如果你需要独立管理版本、单独分发，或使用不应放在 Paperclip 仓库中的私有运行时，可将适配器作为独立 npm 软件包或本地目录发布。
---

# 外部适配器

外部适配器允许你将 Paperclip 适配器作为独立 npm 软件包或本地目录发布。如果你需要独立管理版本、单独分发，或运行时不应放在 Paperclip 仓库中，请使用外部适配器。

---

## 适用场景

- 希望将适配器独立于 Paperclip 进行分发。
- 正在为团队或工作区开发内部适配器。
- 需要一种本地插件，可以在不编辑 Paperclip 源代码的情况下安装和重新加载。

## 不适用场景

- 适配器应随 Paperclip 一同发布。请使用内置适配器。
- 只需要运行一次性的本地命令。请使用 [Process](./process.md)。
- 此运行时已有对应的内置本地适配器。

---

## 内置适配器与外部适配器

| 方面 | 内置适配器 | 外部适配器 |
|---|---|---|
| 源代码 | Paperclip 仓库 | 独立 npm 软件包或本地目录 |
| 安装方式 | 随 Paperclip 一同发布 | 通过 Adapter Manager 或 `POST /api/adapters/install` 安装 |
| 更新方式 | 需要发布 Paperclip 新版本 | 独立管理软件包版本 |
| 界面解析器 | 在 Paperclip 中静态导入 | 可选导出 `./ui-parser` |
| 注册方式 | 在主机中硬编码注册 | 启动时从适配器插件存储加载 |

> **提示：** 如果你希望独立于 Paperclip 应用管理适配器发布周期，外部适配器通常是合适的选择。

---

## 软件包结构

最小软件包布局：

```text
my-adapter/
  package.json
  tsconfig.json
  src/
    index.ts
    server/
      index.ts
      execute.ts
      test.ts
    ui-parser.ts
```

`src/index.ts` 应导出适配器元数据和服务器工厂：

```ts
export const type = "my_adapter";
export const label = "My Adapter";
export const models = [{ id: "model-a", label: "Model A" }];
export const agentConfigurationDoc = `# my_adapter agent configuration

Use when:
- ...

Don't use when:
- ...
`;

export { createServerAdapter } from "./server/index.js";
```

`package.json` 应公开运行时入口；如果你附带解析器，也要公开界面解析器软件包：

```json
{
  "name": "my-paperclip-adapter",
  "version": "1.0.0",
  "type": "module",
  "paperclip": {
    "adapterUiParser": "1.0.0"
  },
  "exports": {
    ".": "./dist/index.js",
    "./server": "./dist/server/index.js",
    "./ui-parser": "./dist/ui-parser.js"
  }
}
```

---

## 安装

可以从 Board 界面或通过 API 安装外部适配器：

```http
POST /api/adapters/install
Content-Type: application/json
```

请求正文接受以下字段：

| 字段 | 必填 | 说明 |
|---|---:|---|
| `packageName` | 是 | npm 软件包名称或本地路径。 |
| `version` | 否 | 可选的 npm 版本。 |
| `isLocalPath` | 否 | 本地检出目录应设为 `true`。 |

示例：

```json
{
  "packageName": "@henkey/hermes-paperclip-adapter",
  "version": "latest",
  "isLocalPath": false
}
```

安装后，适配器会出现在 `GET /api/adapters` 返回结果和 Adapter Manager 界面中。

---

## 服务器契约

主机会从你的软件包根目录调用 `createServerAdapter()`。该工厂必须返回一个至少包含以下项目的 `ServerAdapterModule`：

- `type`
- `execute()`
- `testEnvironment()`
- `models`
- `agentConfigurationDoc`

服务器工厂是外部适配器的核心，也是将软件包转换为可运行适配器类型的组件。

---

## 界面解析器契约

如果适配器输出结构化 stdout，请将 `./ui-parser` 作为自包含且可在浏览器安全运行的模块一并发布。

请参阅：

- [适配器界面解析器契约](./adapter-ui-parser.md)

界面解析器并非必需，但它决定了你看到的是原始日志文本，还是易读的执行记录。

---

## 运行时生命周期

1. Paperclip 启动时加载适配器软件包。
2. 插件存储会记录软件包名称、本地路径和安装版本。
3. 适配器会在 Board 界面和服务器注册表中启用。
4. 之后可以通过 Adapter Manager 禁用、重新加载、重新安装或移除适配器。

常用管理端点：

- `GET /api/adapters`
- `PATCH /api/adapters/:type`
- `POST /api/adapters/:type/reload`
- `POST /api/adapters/:type/reinstall`
- `DELETE /api/adapters/:type`

> **注意：** 内置适配器受到保护，不能删除。仍可以安装一个**覆盖**内置类型的外部适配器（见下文）；不过内置适配器会保留为回退实现。只有在存在外部覆盖时，才可以删除相应的内置适配器。

---

## 覆盖内置适配器

可以安装一个 `type` 与内置适配器相同的外部适配器，例如发布自己的 `hermes_local`，以使用较新版本或组织专属实现。`POST /api/adapters/install` 接受此覆盖实现；外部适配器会优先使用，内置适配器则保留为回退实现。

需要了解以下几点：

- 首次安装某内置类型的覆盖实现时，无需重启。如果已有该类型的外部插件记录，重新安装已有覆盖实现是唯一会报告 `requiresRestart` 的情况。
- 在 Adapter Manager 中暂停覆盖实现后，该类型的所有智能体会在下一次运行时改回内置实现；恢复覆盖实现后又会改回外部适配器。
- 移除覆盖实现会注销外部适配器并回退到内置适配器。内置适配器本身永远不会被删除。

面向运维人员的暂停、恢复和移除覆盖实现操作指南位于[设置 → 实例：适配器](../../administration/settings.md)。

---

## 示例工作流程

1. 创建软件包并导出 `createServerAdapter()`。
2. 如果附带自定义解析器，则添加 `paperclip.adapterUiParser`。
3. 通过 Adapter Manager 或 `POST /api/adapters/install` 安装软件包。
4. 使用 `GET /api/adapters/:type/config-schema` 驱动界面表单。
5. 如果附带解析器，则使用 `GET /api/adapters/:type/ui-parser.js`。
6. 在本地迭代期间重新加载适配器。

---

## 实用说明

- 开发适配器时，优先通过本地路径安装。
- 希望软件包像其他依赖一样工作时，请使用 npm 安装。
- 保持软件包自包含。主机预期适配器无需修改 Paperclip 源代码即可顺利加载。
- 适配器自身配置文档应以适配器软件包为准。

---

## 后续阅读

- [创建适配器](./creating-an-adapter.md)
- [适配器界面解析器契约](./adapter-ui-parser.md)
- [Claude Code](./claude-code.md)
- [Codex](./codex.md)
