---
paperclip_version: v2026.1012.0
seo_title: 创建自定义适配器
seo_description: 如果内置适配器不适用于你的运行时，可以自行开发。本页介绍代码结构和适配器必须满足的运行时契约。
---

# 创建适配器

如果内置本地适配器不适用于你的运行时，可以创建自定义适配器。本页介绍代码结构和运行时契约；如果你要打包可安装插件，请先阅读[外部适配器](./external-adapters.md)。

> **提示：** 如果你使用 Claude Code 搭建适配器，`.agents/skills/create-agent-adapter` 技能可以通过交互方式引导你完成相同的结构。

---

## 内置适配器与外部适配器

| 方面 | 内置适配器 | 外部适配器 |
|---|---|---|
| 源代码 | 位于 Paperclip 仓库中 | 位于独立软件包中 |
| 注册方式 | 添加到主机注册表 | 通过适配器插件存储加载 |
| 界面解析器 | 静态导入 | 可选导出 `./ui-parser` |
| 适用场景 | 核心适配器和由主机拥有的运行时 | 独立分发和本地插件 |

对于大多数新的运行时集成，请先开发外部适配器软件包。只有 Paperclip 本身需要随产品一同发布适配器时，才将其改为内置适配器。

---

## 建议的软件包布局

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
    cli/
      format-event.ts
```

核心规则很简单：保持软件包自包含，并在软件包根目录导出元数据和服务器工厂。

`cli/format-event.ts` 是可选文件。只有在你希望为 `paperclipai run --watch` 提供自定义实时监视格式化器时，才添加它。

---

## 根目录元数据

主机会导入 `src/index.ts`，因此该文件应尽量少依赖其他模块。

```ts
export const type = "my_adapter";
export const label = "My Adapter";
export const models = [{ id: "model-a", label: "Model A" }];
export const agentConfigurationDoc = `# my_adapter agent configuration

Use when:
- ...

Don't use when:
- ...

Core fields:
- ...
`;

export { createServerAdapter } from "./server/index.js";
```

用户配置适配器时，界面会显示 `agentConfigurationDoc` 字符串。

---

## 服务器工厂

`createServerAdapter()` 是服务器端入口。它应返回一个 `ServerAdapterModule`，将执行逻辑与环境测试连接起来。

```ts
import type { ServerAdapterModule } from "@paperclipai/adapter-utils";
import { execute } from "./execute.js";
import { testEnvironment } from "./test.js";

export function createServerAdapter(): ServerAdapterModule {
  return {
    type: "my_adapter",
    execute,
    testEnvironment,
    models: [{ id: "model-a", label: "Model A" }],
    agentConfigurationDoc,
  };
}
```

Paperclip 会依赖该模块来执行适配器生命周期契约。

---

## 执行

`execute()` 接收 `AdapterExecutionContext` 并返回 `AdapterExecutionResult`。

请按以下步骤使用它：

1. 使用 `@paperclipai/adapter-utils/server-utils` 中安全的辅助函数读取配置。
2. 使用 `buildPaperclipEnv(agent)` 构建运行时环境。
3. 从 `runtime.sessionParams` 解析或恢复会话状态。
4. 使用 `renderTemplate()` 渲染提示词模板。
5. 启动命令或调用远程服务。
6. 返回用量、费用、会话和结果元数据。

你可能会用到以下辅助函数：

| 辅助函数 | 用途 |
|---|---|
| `runChildProcess()` | 启动本地命令，并提供流式日志和超时控制。 |
| `buildPaperclipEnv()` | 注入标准的 `PAPERCLIP_*` 环境变量。 |
| `renderTemplate()` | 替换 `{{agentId}}` 等模板变量。 |
| `asString()`、`asNumber()`、`asBoolean()` | 安全读取配置值。 |

> **注意：** 将适配器输出视为不可信数据。应进行防御性解析，切勿盲目执行其 stdout。

### 连接说明

连接可以携带使用说明，所有获得授权的智能体轮次都应查看这些说明。服务器会为每次运行解析连接说明，并将它们放入执行上下文的 `context.connectionInstructions`：该值是 `{ text, digest }` 对象；如果没有适用内容，则为 `null`。请将 `text` 附加到模型提示词中；不要直接传递整个对象。

如果你通过将 `context.paperclipWake` 传给 `@paperclipai/adapter-utils/server-utils` 中的 `renderPaperclipWakePrompt()` 来构造唤醒提示词，连接说明会自动附加。内置的 [HTTP](./http.md#connection-instructions) 和 [Process](./process.md#connection-instructions) 适配器会将同一对象转发给你的服务或脚本。

---

## 环境测试

`testEnvironment()` 会在执行开始前验证适配器配置。

可使用它检查：

- 命令或端点是否存在；
- 工作目录是否有效；
- 必需的身份验证信息或环境变量是否存在；
- 轻量级 hello 探测是否成功。

请返回 `info`、`warn` 和 `error` 检查结果，让界面能够说明哪些项目已就绪，以及哪些项目仍需处理。

---

## 会话持久化

如果运行时可以跨心跳恢复状态，请将状态保存在 `sessionParams` 中，并在下次唤醒时恢复。

```ts
export const sessionCodec = {
  deserialize(raw) {
    // Validate the raw payload and convert it into session params.
  },
  serialize(params) {
    // Convert session params back into a storable shape.
  },
  getDisplayId(params) {
    // Return a human-readable label for the session, if available.
  },
};
```

运行时报告先前会话无法恢复时，请使用 `clearSession: true`。

---

## 注入技能

适配器应让运行时能够访问 Paperclip 技能，同时避免污染用户工作区。

首选方案：

1. 创建临时技能目录，并通过 CLI 标记传递该目录。
2. 在运行时的全局技能位置创建符号链接。
3. 通过环境变量让运行时使用受管理的技能目录。
4. 只有当运行时不支持更好的方法时，才回退到将技能内容注入提示词。

内置本地适配器已经通过各自特定运行时的方式实现了此功能。

---

## 界面解析器

如果你需要比通用 shell 解析器更丰富的执行记录呈现方式，可以提供自包含的 `ui-parser.ts`。

请参阅：

- [适配器界面解析器契约](./adapter-ui-parser.md)

对于内置适配器，界面解析器可以位于 Paperclip 源代码中。对于外部适配器，它必须独立且可在浏览器中安全运行。

---

## 注册内置适配器

只有内置适配器才应添加到主机注册表：

1. `server/src/adapters/registry.ts`
2. `ui/src/adapters/registry.ts`
3. `cli/src/adapters/registry.ts`

外部适配器应通过插件加载器自行注册。

---

## 安全

- 将密钥放在环境变量或密钥引用中，不要写入提示词。
- 将运行时输出视为不可信输入。
- 设置超时和宽限期。
- 界面解析器不得使用 DOM 或 Node API。

---

## 后续阅读

- [外部适配器](./external-adapters.md)
- [适配器界面解析器契约](./adapter-ui-parser.md)
- [智能体如何工作](../../guides/org/agents.md)
