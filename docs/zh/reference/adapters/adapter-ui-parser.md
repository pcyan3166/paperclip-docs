---
seo_title: 适配器界面解析器契约
seo_description: Paperclip 会将适配器 stdout 实时传输到执行查看器。提供界面解析器，将结构化输出渲染为工具调用和结果，而不是原始文本。
---

# 适配器界面解析器契约

Paperclip 会将适配器 stdout 实时传输到执行查看器。如果你的适配器输出结构化行，请提供界面解析器，让查看器能够渲染工具调用、工具结果、思考区块和系统消息，而不是将所有内容都当作普通 Assistant 文本。

以下情况请参阅本页：

- 你正在开发外部适配器；
- 你的运行时会输出结构化 stdout；
- 你希望执行查看器显示 shell 执行记录以外的内容。

---

## 解决的问题

如果没有自定义解析器，界面会回退到通用 shell 解析器。这意味着：

- 工具调用会显示为普通文本；
- 工具耗时和结果会丢失；
- 更难区分 stderr 和系统标记。

使用解析器后，执行查看器就能显示更丰富的执行记录，并清楚呈现执行过程。

> **提示：** 如果运行时只输出纯文本，可以完全跳过解析器，让通用 shell 解析器处理即可。

---

## 加载方式

1. 适配器软件包在 `package.json` 中导出 `./ui-parser`。
2. 服务器读取此模块，并通过 `GET /api/adapters/:type/ui-parser.js` 提供该模块。
3. 用户打开某次执行时，浏览器会获取此模块。
4. 界面会在浏览器沙盒中执行解析器，并为该适配器类型注册它。

```text
package.json -> dist/ui-parser.js -> GET /api/adapters/:type/ui-parser.js -> browser eval -> transcript rendering
```

> **注意：** 解析器模块必须独立自包含。不能导入运行时模块、访问 DOM 或调用 Node API。

---

## 软件包契约

你的软件包应声明解析器契约版本：

```json
{
  "paperclip": {
    "adapterUiParser": "1.0.0"
  }
}
```

主机加载解析器前会检查此值。

| 主机要求 | 适配器声明 | 结果 |
|---|---|---|
| `1.x` | `1.0.0` | 加载解析器 |
| `1.x` | `2.0.0` | 主机记录警告，并回退到通用解析器 |
| `1.x` | 缺失 | 暂时仍会加载解析器，但未来版本可能要求此字段 |

---

## 导出形式

你的 `dist/ui-parser.js` 必须至少导出以下形式之一：

### `parseStdoutLine(line: string, ts: string): TranscriptEntry[]`

如果每一行都可以独立解析，请使用此形式。

```ts
export function parseStdoutLine(line: string, ts: string): TranscriptEntry[] {
  if (line.startsWith("[my-agent]")) {
    return [{ kind: "system", ts, text: line }];
  }

  return [{ kind: "assistant", ts, text: line }];
}
```

### `createStdoutParser(): { parseLine(line, ts): TranscriptEntry[]; reset(): void }`

如果需要跨行保留状态，例如处理多行工具输出或续行，请使用此形式。

```ts
let counter = 0;

export function createStdoutParser() {
  let awaitingResult = false;

  function parseLine(line: string, ts: string): TranscriptEntry[] {
    const trimmed = line.trim();
    if (!trimmed) return [];

    if (trimmed.startsWith("[tool]")) {
      const toolUseId = `tool-${++counter}`;
      awaitingResult = true;
      return [{ kind: "tool_call", ts, name: "shell", input: {}, toolUseId }];
    }

    if (awaitingResult) {
      awaitingResult = false;
      return [{ kind: "tool_result", ts, toolUseId: `tool-${counter}`, content: trimmed, isError: false }];
    }

    return [{ kind: "assistant", ts, text: trimmed }];
  }

  function reset() {
    awaitingResult = false;
  }

  return { parseLine, reset };
}
```

如果两种导出形式都存在，会优先使用带状态的解析器。

---

## 执行记录条目

查看器可识别以下条目类型：

```ts
{ kind: "assistant"; ts: string; text: string; delta?: boolean }
{ kind: "thinking"; ts: string; text: string; delta?: boolean }
{ kind: "user"; ts: string; text: string }
{ kind: "tool_call"; ts: string; name: string; input: unknown; toolUseId?: string }
{ kind: "tool_result"; ts: string; toolUseId: string; content: string; isError: boolean }
{ kind: "system"; ts: string; text: string }
{ kind: "stderr"; ts: string; text: string }
{ kind: "stdout"; ts: string; text: string }
```

使用 `toolUseId` 将调用与结果关联起来。界面会将这些配对渲染为可折叠卡片。

> **警告：** 切勿让解析器抛出异常。如果无法解析某一行，请改为返回普通 `stdout` 条目。

---

## 约束

1. 不得导入运行时模块。
2. 不得产生顶层副作用。
3. 不得访问 DOM 或 Node API。
4. 对相同的 `(line, ts)` 输入，输出必须确定且一致。
5. 必须能够容错处理每一行。
6. 保持软件包体积小。每次运行都会提供并执行解析器。

---

## 测试

发布软件包前，请使用示例执行记录测试解析器。

```ts
import { createStdoutParser } from "./dist/ui-parser.js";

const parser = createStdoutParser();

for (const line of [
  "[my-agent] Starting session abc123",
  "Thinking about the task...",
  "[tool] read /src/main.ts",
  "const main = () => {}",
]) {
  const entries = parser.parseLine(line, new Date().toISOString());
  console.log(entries);
}
```

只要解析器可在浏览器中安全运行且导出形式正确，Paperclip 就能在运行时动态加载它。

---

## 何时可以跳过

以下情况可以不提供解析器：

- 运行时只输出纯文本；
- 工具调用边界对用户不重要；
- 通用 shell 执行记录已经足够易读。

对于简单的适配器和命令包装器，这是完全合理的选择。

---

## 后续阅读

- [外部适配器](./external-adapters.md)
- [创建适配器](./creating-an-adapter.md)
