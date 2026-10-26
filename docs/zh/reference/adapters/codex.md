---
paperclip_version: v2026.1023.0
seo_title: Codex 适配器
seo_description: 在 Paperclip 主机上将 OpenAI Codex CLI 作为本地编程智能体运行，并为每个智能体持久保存会话状态和受管理的 CODEX_HOME。
---

# Codex

`codex_local` 会在与 Paperclip 相同的计算机上运行 OpenAI Codex CLI。如果你需要具有持久会话状态、受管理 `CODEX_HOME`，并能将 Paperclip 技能注入 Codex 技能目录的本地编程智能体，请使用此适配器。

---

## 适用场景

- 你已经在主机上使用 Codex CLI。
- 希望会话在多次心跳之间保持连续。
- 希望 Paperclip 尽可能按公司管理 Codex 主目录。
- 希望仓库指令、Paperclip 指令和 Codex 自身的运行时行为协同工作。

## 不适用场景

- 运行时位于 webhook 或远程 API 后面。请使用 [HTTP](./http.md)。
- 只需要 shell 命令或脚本。请使用 [Process](./process.md)。
- 尚未安装 Codex CLI，或 Paperclip 主机无法执行该 CLI。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `cwd` | 否 | 智能体的绝对工作目录。实际使用中建议设置。省略后，适配器会回退到当前进程工作目录。有权限时 Paperclip 会创建该路径。 |
| `engine` | 否 | Codex 的运行方式：`auto`（默认，运行 ACP）、`acp`（始终使用 Agent Client Protocol）或 `cli`（始终使用经典 Codex CLI）。请参阅 [ACP 引擎](#acp-引擎)。 |
| `model` | 否 | Codex 模型 ID。请参阅[模型](#模型)。留空时适配器会省略 `--model`，由 Codex CLI 使用自身默认值。 |
| `promptTemplate` | 否 | 执行时使用的提示词模板。 |
| `instructionsFilePath` | 否 | 添加到发送给 `codex exec` 的 stdin 提示词前面的 Markdown 文件。 |
| `modelReasoningEffort` | 否 | 通过 Codex 配置传递的推理力度覆盖值。选项取决于模型，请参阅[推理力度](#推理力度)。 |
| `search` | 否 | 使用 `--search` 运行 Codex。 |
| `fastMode` | 否 | 通过设置 `service_tier="fast"` 和 `features.fast_mode=true` 启用 Codex Fast 模式。支持 `gpt-6.1-sol`、`gpt-6-astra`、`gpt-6-sol`、`gpt-6-luna`、`gpt-5.6-sol`、`gpt-5.6-terra`、`gpt-5.6-luna`、`gpt-5.5` 和 `gpt-5.4`，也会传递给手动输入的模型 ID。如果配置的模型无法使用此模式，Paperclip 会忽略设置并说明原因。 |
| `dangerouslyBypassApprovalsAndSandbox` | 否 | 使用 `--dangerously-bypass-approvals-and-sandbox` 运行 Codex，使无人值守执行不会因审批而停止。留空时默认开启——请参阅[权限](#权限)。 |
| `command` | 否 | 默认为 `codex`。 |
| `extraArgs` | 否 | 追加到 Codex 调用中的额外 CLI 参数。 |
| `env` | 否 | 传递给运行时的环境变量。支持密钥引用。 |
| `timeoutSec` | 否 | 执行超时秒数。在本地和 SSH 目标上，`0` 表示适配器不设置墙钟超时。在沙盒目标上，`0` 或未设置时采用沙盒默认值 14,400 秒；使用正数覆盖该值，使用负数禁用适配器超时。 |
| `graceSec` | 否 | 强制停止前的宽限时间。 |
| `outputInactivityTimeoutMs` | 否 | 适配器等待 Codex 输出多久后，才将运行判定为卡住。Codex 每次输出都会重置计时器（Linux 上，如果进程显示实际 CPU、磁盘或子进程活动，也会重置），因此活跃运行不会触发超时。未设置时默认为 30 分钟（`1800000`）。设为 `null` 可完全关闭监控——只应对确定会长时间无输出的任务这样做，因为 Paperclip 平台仍会在静默运行一小时后触发安全保护。触发时，适配器会停止 Codex 进程，并将运行报告为失败；错误信息类似 `monitor: no codex activity (output or process) for 30m 0s`。 |
| `workspaceStrategy` | 否 | 执行工作区策略，例如 `git_worktree`。 |
| `workspaceRuntime` | 否 | 保留供工作区运行时使用的元数据。 |

> **注意：** Codex 通过 stdin 接收提示词，并使用 `codex exec --json`。在尝试真实心跳之前，适配器环境测试会同时检查命令和身份验证方式。

---

<a id="permissions"></a>
## 权限

Codex 智能体默认以全自动模式运行。如果不设置 `dangerouslyBypassApprovalsAndSandbox`，Paperclip 会传入 `--dangerously-bypass-approvals-and-sandbox`，使 Codex 能够处理任务，而不会因无人响应的审批请求停下来。

你可以通过显式设置来控制行为。出现以下任一情况时，Paperclip 会跳过完全绕过权限的设置——让 Codex 保持在可写工作区沙盒中，或使用你选择的模式。后面三项只在你未设置 `dangerouslyBypassApprovalsAndSandbox` 时生效：

- 将 `dangerouslyBypassApprovalsAndSandbox` 设为 `false`。
- 通过 `extraArgs` 自行选择沙盒模式或配置档，例如 `--sandbox`、`--profile`、`--full-auto` 或 `sandbox_mode=` 配置覆盖。
- 通过 `extraArgs` 设置 `approval_policy=` 覆盖，或使用 `sandbox_workspace_write.network_access=false` 关闭网络访问。
- 执行目标拒绝网络访问。

此绕过功能仅适用于经典 Codex CLI 路径（`engine: "cli"`）。ACP 引擎会让 Codex 保持在可写工作区沙盒中，并在每轮执行中开启网络访问。

> **请注意：** 早期版本会让 Codex 始终留在工作区沙盒中，除非你主动启用绕过。如果你依赖旧行为，请明确将 `dangerouslyBypassApprovalsAndSandbox` 设为 `false`。

---

## ACP 引擎

Codex 可通过 ACP 或经典 Codex CLI 两种引擎运行，由 `engine` 字段选择：

- **`auto`（默认）——ACP。** Paperclip 通过 Agent Client Protocol（ACP）运行 Codex。如果主机不满足 ACP 前置条件，执行会失败并显示说明原因的设置错误——Paperclip 不会自行切换引擎。
- **`acp`——始终使用 ACP。** 强制使用 Agent Client Protocol 路径。
- **`cli`——始终使用 Codex CLI。** 强制使用经典 CLI 包装器，完全跳过 ACP。

ACP 可提供更丰富的结构化实时执行记录：会话身份、带上下文窗口用量的状态、Assistant 和思考过程的令牌增量，以及会随进度更新并最终折叠为单张卡片的工具调用信息。你在查看沙盒执行的实时输出时，额外细节尤为有用。

当引擎解析为 ACP（`auto` 或 `acp`）时，以下额外字段适用：

| 字段 | 默认值 | 说明 |
|---|---|---|
| `agentCommand` | 软件包本地的 `codex-acp` | 可选，用于覆盖 Codex ACP 服务器命令。 |
| `mode` | `persistent` | `persistent` 会在多次执行之间保留 ACP 会话状态；`oneshot` 则每次运行都重新开始。 |
| `nonInteractivePermissions` | `deny` | ACP 智能体在交互式会话外请求输入时如何处理——拒绝请求（`deny`）或令执行失败（`fail`）。 |
| `stateDir` | 由 Paperclip 管理 | 可选的 ACP 会话状态目录。默认位于 Paperclip 按公司和智能体划分的存储目录中。 |
| `warmHandleIdleMs` | `0` | 两次运行之间保持 ACP 进程热运行的时间，单位为毫秒。`0` 表示每次运行后关闭进程，但仍保留持久会话状态。 |

> **请注意：** 旧版独立 `acpx_local` 适配器的能力现在由 ACP 提供。该适配器已退役——请选择 `codex_local`（或 `claude_local` / `gemini_local`），并将 `engine` 保持为 `auto`，即可默认使用 ACP。

### 在沙盒环境中使用 ACP

如果智能体在 Paperclip 沙盒环境中运行，可以将 `engine` 保持为 `auto`。如果沙盒提供 Paperclip 双向进程会话，Paperclip 就会继续使用 ACP 引擎及其结构化实时执行记录；无需在适配器配置中添加单独的桥接设置。

只运行一次性命令的环境无法承载 ACP 会话，因此此类环境中的 `auto` 执行会失败并显示设置错误，而不会切换到 Codex CLI。如果你希望使用 CLI 路径（例如在此类环境中），请设置 `engine: "cli"`。

---

<a id="models"></a>
## 模型

在 `model` 字段中选择已知的 Codex 模型 ID。选择器会按 ChatGPT 应用对 Codex 模型的排序方式显示：最新版本优先，同一版本内能力更强的模型优先，旧模型排在末尾：

- `gpt-6.1-sol`
- `gpt-6-astra`
- `gpt-6-sol`
- `gpt-6-luna`
- `gpt-5.6-sol`（适配器默认值）
- `gpt-5.6-terra`
- `gpt-5.6-luna`
- `gpt-5.5`
- `gpt-5.4`
- `gpt-5.4-mini`
- `gpt-5`
- `gpt-5-mini`
- `gpt-5-nano`
- `o3`
- `o4-mini`
- `o3-mini`
- `codex-mini-latest`

这是一个经过筛选的列表。即使提供了 OpenAI API 密钥，选择器也不会从 OpenAI 模型目录自动填充，因为该目录还包含 Codex 无法运行的图像、音频和嵌入模型。如果希望在下拉列表中添加其他模型（例如通过自己的网关提供的模型），请使用 [`PAPERCLIP_ADAPTER_MODELS`](../deploy/environment-variables.md#adding-models-to-the-model-picker) 声明它们。

你也可以输入列表中没有的模型 ID。Paperclip 无法识别的值会视为手动模型 ID，并原样传给 Codex CLI。

> **请注意：** 默认值现在是具体模型标识 `gpt-5.6-sol`。OpenAI 没有为简写 `gpt-5.6` 发布元数据，因此 Paperclip 会自动将其改写为 `gpt-5.6-sol`——仍使用旧别名配置的智能体会继续工作，也不会再触发 CLI 的“找不到模型元数据”警告。

> **提示：** 将 `model` 留空即可让 Codex 自行选择。没有设置模型时，适配器会完全省略 `--model` 标记，使 Codex CLI 使用其自身默认模型。

### 推理力度

`modelReasoningEffort` 可选值取决于你选择的模型：

| 模型 | 推理力度级别 |
|---|---|
| `gpt-6.1-sol`、`gpt-6-astra`、`gpt-6-sol`、`gpt-5.6-sol`、`gpt-5.6-terra` | `low`、`medium`、`high`、`xhigh`、`max`、`ultra` |
| `gpt-6-luna`、`gpt-5.6-luna` | `low`、`medium`、`high`、`xhigh`、`max` |
| 其他模型 | `minimal`、`low`、`medium`、`high`、`xhigh` |

如果切换到不支持当前所选推理力度的模型，智能体表单会清除该值并回退到 Auto。

---

## 模型配置档

模型配置档是智能体主要模型设置的具名替代项，因此可以让同一智能体使用不同执行路径，而无需重写适配器配置。Codex 自带一个配置档，由适配器提供（`source: "adapter_default"`），因此每个 `codex_local` 智能体都可以使用：

| 键 | 标签 | 说明 |
|---|---|---|
| `cheap` | Cheap（低成本） | 使用明确配置的低成本 Codex 模型，而不更改主模型。 |

关键在于“明确配置”：随附的 `cheap` 配置档 `adapterConfig` 为空，不会替你选择模型。启用它并不会自动改变行为，必须先填写低成本执行路径所用设置——通常是 `model`，还可以选择覆盖 `modelReasoningEffort`。由你决定此智能体使用哪个模型作为低成本模型，而适配器配置中的主 `model` 保持不变。

> **请注意：** 过去 `cheap` 配置档会为你硬编码特定 Codex 模型和推理力度，现在不会了。如果你依赖旧行为，请在配置档中自行设置需要的模型。

新建 Codex 智能体时，此配置档默认关闭。创建智能体时，如果其运行时配置中没有 `cheap`，Paperclip 会写入 `modelProfiles.cheap = { enabled: false }`——配置档存在但处于休眠状态，等待你配置并启用。

---

## 会话持久化

Codex 会保留 `previous_response_id` 链，使心跳可以继续同一段对话，而不必每次都从头开始。

如果存在以下位置提示，会话编解码器也会保留它们：

- `cwd`
- `workspaceId`
- `repoUrl`
- `repoRef`

如果工作目录发生变化，适配器会启动新会话，而不是复用旧响应链。

### 在 Paperclip Runner 上原生运行 Codex

Codex 也可以通过 [Paperclip Runner](./paperclip-runner.md) 原生运行（`paperclip_runner` 并设置 `provider: "codex"`），而不是通过 `codex_local`。原生路径有以下特定行为：

- **重启不会丢失对话。** 如果服务器重启导致原生 Codex 轮次中途终止，Paperclip 会先确认旧进程确实已经退出，然后在同一对话和工作区中重试该请求一次，并要求智能体继续之前先检查未完成的命令或外部操作。它不会重新发送任务。执行日志会使用 `provider_turn_lost_on_restore` 记录中断轮次。仍无法核实的内容会暂停并交由人员处理。
- **Codex 保留自己的基础提示词。** 在线程启动或恢复时，Paperclip 会将任务上下文作为附加的 `developerInstructions` 发送，因此 Codex 的标准基础指令仍然保留。此变更之前创建的对话在恢复时仍会使用保存的提示词；如果希望恢复标准基础指令，请重置这些会话。
- **ChatGPT 账户无权使用模型时会显示明确提示。** Codex 使用 ChatGPT 登录时，如果拒绝所选模型，任务会显示 **Model unavailable.**（模型不可用），并提示 *“当前 ChatGPT 连接不支持所选模型。请选择受支持的模型或兼容的 AI 连接。”* Paperclip 不会自动重试该请求。请选择受支持的模型或清除任务的模型覆盖设置，然后重试。
- **繁忙模型会自动重试。** 所选模型容量已满时，Paperclip 会显示 **Model at capacity**（模型容量已满），并分别在一分钟后和两分钟后重试。请参阅[执行需要关注时](../../experimental/task-chat.md#when-a-run-needs-attention)。

---

## 技能注入

Codex 会将 Paperclip 技能注入有效的 `CODEX_HOME/skills` 目录。

在默认的受管理主目录模式下，Paperclip 会在当前 Paperclip 实例下为每家公司使用独立 Codex 主目录，并从共享 Codex 主目录导入身份验证信息，以延续登录状态。

如果 Paperclip 运行在受管理工作树实例中，此 Codex 主目录也会按工作树隔离，避免会话、日志和技能在不同检出目录之间泄漏。

如果要在 Paperclip 之外手动使用本地 CLI，请运行：

```sh
pnpm paperclipai agent local-cli codexcoder --company-id <company-id>
```

该命令会安装缺少的技能、创建智能体 API 密钥，并打印可用于以该智能体身份运行 Codex 的 shell 导出语句。

---

## 指令文件

如果设置了 `instructionsFilePath`，Paperclip 会读取文件，并在每次运行中将其添加到发送给 `codex exec` 的 stdin 提示词前面。

这与 Codex 自身发现仓库指令的机制相互独立。如果工作目录中存在 `AGENTS.md`，Codex 仍会按正常行为加载该文件。

> **提示：** Paperclip 管理的指令请使用 `instructionsFilePath`。如果希望 Codex 自然地从工作区读取仓库指令，请使用仓库本地的指令文件。

---

## 身份验证和按智能体划分的主目录

每个 `codex_local` 智能体都使用自己受管理的 Codex 主目录，因此不同智能体绝不会使用彼此的登录凭据，也不会共享 Codex 状态。由于主目录最初为空，Paperclip 会在启动 Codex 前为其准备凭据：

- **默认继承主机上的 Codex 登录。** 如果你已在 Paperclip 主机上登录 Codex（使用 ChatGPT 订阅登录），Paperclip 会自动将该登录关联到智能体的受管理主目录。无需为每家公司或每个智能体重复登录。
- **设置后，按智能体配置的 API 密钥优先。** 如果智能体的 `env` 中包含 `OPENAI_API_KEY`，Paperclip 会将其写入受管理主目录并用作 API 密钥身份验证，而不会借用主机登录。在 ACP 引擎上，`CODEX_API_KEY` 也可作为该密钥的另一个名称。
- **不会改动你自己的 `CODEX_HOME`。** 如果将适配器指向 Paperclip 受管理公司目录之外的 `CODEX_HOME`，Paperclip 会将其视为自行管理，并且不会注入或覆盖内容。
- **不会静默地在无凭据状态下执行。** 如果受管理主目录中没有可用登录信息且未配置 API 密钥，执行会立即以清晰的适配器错误失败，而不是启动 Codex 后再收到服务提供商返回的 `401`。

### 在沙盒中登录

智能体在没有可用 Codex 登录的 Paperclip 沙盒环境中运行时，环境测试会报告缺少身份验证；Paperclip 还可为你启动交互式设备登录。在浏览器中完成登录后，Paperclip 会将生成的订阅凭据转为公司范围的凭据，供之后的心跳复用。

Paperclip 还会维护按公司范围和身份密钥索引的订阅凭据缓存（每个账户保留一个可用凭据），因此从沙盒执行中恢复的较新登录可以更新主机上已保存的账户。此缓存不会向空白主机注入凭据，也不会切换智能体使用的账户。默认启用；如需关闭，可在 Paperclip 主机上将 `PAPERCLIP_CODEX_AUTH_CACHE` 设为假值（`0`、`false`、`no` 或 `off`）。

### 将 CODEX_HOME 同步到沙盒

智能体在 Paperclip 沙盒中运行时，适配器不会将整个 Codex 主目录复制到沙盒。它会显式暂存允许列表中的内容，确保只有运行真正需要的材料会跨越边界，同时不会带上主机本地的运行时状态。同步内容包括：

- `config.json`
- `config.toml`
- `instructions.md`
- `auth.json`
- `skills`

标准 `codex` 二进制文件在运行时写入的所有内容——会话数据库（`*.sqlite`、`*-wal`）、`plugins/`、`cache/`、`sessions/`、`shell_snapshots/` 等——都会留在主机上，不会同步。此允许列表只适用于沙盒执行；本地和 SSH 目标会直接使用受管理主目录。

---

## 环境测试

**Test Environment**（测试环境）按钮会检查：

- Codex CLI 已安装且可执行。
- 工作目录是可用的绝对路径。
- 已通过 `OPENAI_API_KEY` 或 Codex 自身的登录状态完成身份验证。
- 已安装的 Codex CLI 版本足以支持所选模型。使用 ChatGPT 登录时，`gpt-6.1-sol` 需要 Codex `0.159.0` 或更高版本，`gpt-6-sol` 和 `gpt-6-luna` 需要 `0.157.0` 或更高版本。测试会在智能体实际运行位置（本地、SSH 或沙盒）执行 `codex --version`，并报告 `codex_cli_version_compatible` 或 `codex_cli_version_incompatible`。如果 CLI 过旧，就会跳过 hello 探测。
- hello 探测可以通过提示词 `Respond with hello.` 运行 `codex exec --json -`。

如果探测失败，请先修复命令或身份验证路径，再保存适配器。如果 Codex 后端拒绝 ChatGPT 账户使用所选模型，测试会报告 `codex_hello_probe_model_rejected`，并说明可能原因：Codex CLI 过旧、订阅计划不包含该模型，或该模型尚未开放到你的账户。在沙盒中，如果 CLI 过旧，解决方法是发布包含 Paperclip 固定 Codex 版本的镜像。向模型目录添加模型不会更新现有镜像中预装的 CLI。

---

## 自定义服务提供商和网关

Codex 没有 CLI 标记或环境变量可以指定自定义的兼容 OpenAI 端点。自定义端点应在 `$CODEX_HOME/config.toml` 中以 `[model_providers.<id>]` 表格的形式配置。如果不想手动编辑该文件，可在 Paperclip 主机上设置环境变量 `PAPERCLIP_CODEX_PROVIDERS`，将 Codex 路由到自己的网关或服务提供商。

`PAPERCLIP_CODEX_PROVIDERS` 是一个映射到 Codex `config.toml` 结构的 JSON 对象：

```json
{
  "providers": {
    "my-gateway": {
      "name": "My gateway",
      "base_url": "https://gateway.example.com/v1",
      "env_key": "OPENAI_API_KEY",
      "wire_api": "responses"
    }
  },
  "model_provider": "my-gateway"
}
```

- `providers` 下的每个键都会变成一个 `[model_providers.<id>]` 表格。常见字段包括 `name`（可选的显示名称）、`base_url`（兼容 OpenAI 的端点）、`env_key`（Codex 在请求时读取 Bearer 密钥的环境变量）和 `wire_api`（Codex 与服务提供商通信时使用的协议）。Codex 支持的其他字段也会原样传递，包括 `query_params`、`http_headers`、`env_http_headers` 和 `request_max_retries`。
- `model_provider` 是可选字段。设置后会选择顶层服务提供商，并且必须与 `providers` 中的条目一致。然后由 `model` 字段选择该服务提供商下的模型。
- 字符串值可以使用 `{env:VAR}` 占位符。Paperclip 会在服务器端根据执行环境变量和 `process.env` 展开占位符；这对 `http_headers` 等必须提供字面值的字段很有用。Bearer 密钥应优先使用 Codex 自带的 `env_key` 间接引用方式。无法解析的占位符会保持原样。

Paperclip 会将这些定义写入按公司受管理的 Codex 主目录中 `config.toml` 的受管理区域，该区域由 `# >>> paperclip codex providers ... managed, do not edit >>>` 标记注释界定。你现有的配置会保留；冲突定义会被移除，以便受管理的定义生效；执行完成后会恢复原文件。

> **注意：** 只有受管理的 Codex 主目录才会执行此合并。如果适配器配置明确设置了 `env.CODEX_HOME`，Paperclip 会保持该用户管理的主目录不变，并显示说明，而不会进行合并。

---

## 示例

```json
{
  "adapterType": "codex_local",
  "adapterConfig": {
    "cwd": "/Users/me/projects/paperclip-workspace",
    "model": "gpt-5.6-sol",
    "instructionsFilePath": "/Users/me/projects/paperclip-workspace/INSTRUCTIONS.md",
    "modelReasoningEffort": "medium",
    "search": false,
    "dangerouslyBypassApprovalsAndSandbox": true,
    "env": {
      "OPENAI_API_KEY": {
        "type": "secret_ref",
        "secretId": "secret-id",
        "version": "latest"
      }
    },
    "timeoutSec": 300,
    "graceSec": 15
  }
}
```

---

## 后续阅读

- [创建适配器](./creating-an-adapter.md)
- [适配器界面解析器契约](./adapter-ui-parser.md)
- [外部适配器](./external-adapters.md)
