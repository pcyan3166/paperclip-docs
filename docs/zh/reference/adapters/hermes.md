---
paperclip_version: v2026.626.0
seo_title: Hermes 适配器
seo_description: 在 Paperclip 主机上运行 Nous Research 的 Hermes Agent，为智能体提供持久记忆和每次运行均可使用的 30 多种工具。
---

# Hermes

`hermes_local` 会在与 Paperclip 相同的计算机上运行 [Hermes Agent](https://github.com/NousResearch/hermes-agent)，这是 Nous Research 开发的全功能 AI 智能体。如果你希望通过一个适配器获得持久记忆、30 多种工具、80 多项可加载技能、多服务提供商模型路由和 MCP 客户端支持，请使用它。

---

## 适用场景

- Hermes 是内置适配器，无需安装插件。Paperclip 开箱即提供 `hermes_local`。
- 需要持久记忆、FTS5 会话搜索或子智能体委派。
- 希望路由到多个推理服务提供商（Anthropic、OpenRouter、OpenAI、Nous、OpenAI Codex、Copilot、Copilot ACP、HuggingFace、ZAI、Kimi Coding、MiniMax、Kilocode）。
- 希望使用文件系统检查点确保可安全回滚。

## 不适用场景

- 运行时位于 webhook 或 API 后面。请使用 [OpenClaw Gateway](./openclaw-gateway.md) 或 [HTTP](./http.md)。
- 只需要运行一次性的 shell 命令。请使用 [Process](./process.md)。
- 计算机上尚未安装 Hermes Agent（需要 Python 3.10 或更高版本）。

---

## 常见字段

### 核心字段

| 字段 | 必填 | 默认值 | 说明 |
|---|---:|---|---|
| `provider` | 否 | `auto` | API 服务提供商。接受 `auto`、`openrouter`、`nous`、`openai-codex`、`copilot`、`copilot-acp`、`anthropic`、`huggingface`、`zai`、`kimi-coding`、`minimax`、`minimax-cn`、`kilocode`。通常请保留 `auto`——Hermes 会根据模型或 `~/.hermes/config.yaml` 推断服务提供商。 |
| `timeoutSec` | 否 | `1800` | 执行超时秒数。 |
| `graceSec` | 否 | `10` | SIGTERM 发出后，等待多长时间再终止 Hermes 进程。 |

<a id="tools"></a>
### 工具

| 字段 | 必填 | 默认值 | 说明 |
|---|---:|---|---|
| `toolsets` | 否 | 全部 | 以逗号分隔的工具集，例如 `terminal,file,web`。 |
| `maxTurnsPerRun` | 否 | — | 可选的 Hermes `--max-turns` 限制，用于限制工具调用轮数。 |

### 会话和工作区

| 字段 | 必填 | 默认值 | 说明 |
|---|---:|---|---|
| `persistSession` | 否 | `true` | 通过 Hermes `--resume` 在多次心跳之间恢复会话。 |
| `worktreeMode` | 否 | `false` | 使用 Git 工作树隔离。 |
| `checkpoints` | 否 | `false` | 启用文件系统检查点，以便回滚。 |

### 高级设置

| 字段 | 必填 | 默认值 | 说明 |
|---|---:|---|---|
| `verbose` | 否 | `false` | 显示详细输出。 |
| `quiet` | 否 | `true` | 使用简洁输出，不显示横幅或旋转图标。 |
| `promptTemplate` | 否 | 内置 | 自定义提示词模板（见下文）。 |
| `paperclipApiUrl` | 否 | — | 可选的 API 基础 URL 覆盖值。默认使用 `PAPERCLIP_API_URL`。 |

---

## 模型选择

Hermes 自行管理模型。你需要在 `~/.hermes/config.yaml` 或 Hermes 自身的配置中设置模型；Paperclip 适配器配置中没有 `model` 字段。这样，“添加 Hermes 智能体”就只需点击一次：如果 Hermes 已在计算机上配置好，设置便完成了。

---

## 会话持久化

当 `persistSession=true`（默认值）时，每次运行都会通过 Hermes 的 `--resume` 标记恢复会话。这可以保留：

- 对话上下文；
- 记忆；
- 工具状态。

会话会标记为 `tool` 来源，避免污染用户的交互式 Hermes 历史记录。适配器的 `sessionCodec` 会在不同运行之间验证和迁移会话状态。

---

<a id="skills-integration"></a>
## 技能集成

适配器会扫描两个技能来源，并在界面中合并显示：

- **Paperclip 管理的技能**——随适配器一同提供，可通过看板界面切换。
- **Hermes 原生技能**——来自 `~/.hermes/skills/`，只读且始终加载。

`listSkills` 和 `syncSkills` 会提供统一快照，因此 Paperclip 界面可以在一个视图中显示这两类技能。

---

## 提示词模板变量

在 `promptTemplate` 中使用 `{{variable}}` 语法：

| 变量 | 说明 |
|---|---|
| `{{agentId}}` | Paperclip 智能体 ID |
| `{{agentName}}` | 智能体显示名称 |
| `{{companyId}}` | 公司 ID |
| `{{companyName}}` | 公司名称 |
| `{{runId}}` | 当前心跳执行 ID |
| `{{taskId}}` | 已分配任务/问题 ID |
| `{{taskTitle}}` | 任务标题 |
| `{{taskBody}}` | 任务指令 |
| `{{projectName}}` | 项目名称 |
| `{{paperclipApiUrl}}` | Paperclip API 基础 URL |
| `{{commentId}}` | 评论 ID（由评论唤醒时提供） |
| `{{wakeReason}}` | 触发此次运行的原因 |

条件区段：

- `{{#taskId}}...{{/taskId}}` — 仅在分配了任务时包含。
- `{{#noTask}}...{{/noTask}}` — 仅在普通心跳检查时包含。
- `{{#commentId}}...{{/commentId}}` — 仅在由评论唤醒时包含。

---

## 执行详情

适配器会以单次查询模式启动 Hermes Agent（`hermes chat -q ...`）。Hermes 使用完整工具集处理任务，然后退出。适配器会：

1. 捕获 stdout/stderr 并解析令牌用量、会话 ID 和费用。
2. 将原始输出解析为结构化 `TranscriptEntry` 对象（带状态图标的工具卡片）。
3. 将 Hermes 的 ASCII 横幅、setext 标题和 `+--+` 表格边框后处理为规范的 GFM Markdown。
4. 重新分类无害的 stderr（MCP 初始化消息、结构化日志），避免它们被显示为错误。
5. 将会话标记为 `tool` 来源。
6. 将结果连同费用、用量和会话状态报告回 Paperclip。

---

## 示例

```json
{
  "adapterType": "hermes_local",
  "adapterConfig": {
    "toolsets": "terminal,file,web",
    "persistSession": true,
    "checkpoints": true,
    "timeoutSec": 1800,
    "graceSec": 10
  }
}
```

---

## 后续阅读

- [创建适配器](./creating-an-adapter.md)
- [适配器界面解析器契约](./adapter-ui-parser.md)
