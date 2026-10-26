---
paperclip_version: v2026.1023.0
seo_title: test-drive 命令
seo_description: 启动一个隔离且自动初始化的 Paperclip 本地实例，其中包含可直接使用的公司和 CEO 智能体，方便体验产品。
---

# test-drive 命令

`test-drive` 是在自己的电脑上体验 Paperclip 的最快方式。只需一条命令，就能启动全新的本地实例、为你创建公司和 CEO 智能体、为智能体配置所需的服务提供商凭据，并在浏览器中打开控制面板。它适合快速试用：这是一个临时环境，可以创建、体验，然后弃用。

```sh
paperclipai test-drive
```

此命令会像 `run` 一样运行一个**本地 Paperclip 实例**，然后在其上初始化入门内容。它不会连接到你已经运行的远程控制平面，而是单独启动一个实例。

> **注意：** `test-drive` 适用于评估和演示，不适用于准备长期使用的公司。它以 `local_trusted` 模式绑定到回环地址，并使用视为临时的数据目录。若要建立可长期使用的本地环境，请使用[`run`](./run.md)或[设置命令](./setup-commands.md)。

---

## 执行步骤

运行 `test-drive` 时，它会：

1. **首先隔离环境。** 在 CLI 加载常规配置和 `.env` 文件之前，`test-drive` 会清除进程中的所有 `PAPERCLIP_*` 变量（以及 `DATABASE_URL`、`HOST`、`PORT` 等少数其他变量），避免 shell 中意外导出的变量将其指向其他实例。唯一保留的是你通过 `--api-key-env` 指定的环境变量（如果提供了该选项）。
2. **隐藏 API 密钥。** 如果传入 `--api-key`，命令会立即从进程参数列表中清除该值，避免其出现在日志、遥测、诊断信息或崩溃输出中。
3. **确保数据库隔离。** 如果数据目录连接到外部 PostgreSQL 数据库（通过 `DATABASE_URL`、`DATABASE_MIGRATION_URL` 或配置中的 `database.mode: postgres`），命令会拒绝运行。因此试用实例始终使用自己的内嵌数据库。
4. **启动实例。** 命令会在回环端口启动本地服务器，修复数据目录，并在不提示用户的情况下采用默认值。重用数据目录时，如果其保存的端口仍可用，就会继续使用该端口，以确保回调和注册 URL 继续工作。否则，从 `3100` 开始选择第一个空闲端口。
5. **初始化公司和智能体。** 如果数据目录为空，命令会创建公司、定义并保存服务提供商密钥，并创建一个配置了相应适配器的 CEO 智能体。如果目录中已经有公司，则会重用该公司，并告知你初始化标志已被忽略。
6. **打开控制面板。** 命令会打印 URL 并在浏览器中打开（除非传入 `--no-browser`）。

> **注意：** Paperclip 退出时会保留数据目录。如果未传入 `--data-dir`，`test-drive` 会为你创建一个全新的临时目录并打印其路径；使用完成后请自行清理。

---

## 选择运行框架和模型

`--harness` 标志决定为 CEO 智能体创建哪个适配器，以及因此需要哪种服务提供商凭据：

| Harness | 适配器 | 所需凭据 |
|---|---|---|
| `claude`（默认值） | `claude_local` | `ANTHROPIC_API_KEY` |
| `codex` | `codex_local` | `OPENAI_API_KEY` |
| `opencode` | `opencode_local` | `OPENROUTER_API_KEY` |

`--model` 用于设置智能体的初始模型；对 `claude` 和 `codex` 是可选项，对 `opencode` 则是必填项，并且必须采用 `openrouter/<model>` 格式（例如 `openrouter/anthropic/claude-sonnet-4`），路径段不能为空。

```sh
# Try it with Codex instead of Claude
paperclipai test-drive --harness codex

# OpenCode requires an openrouter/<model> id
paperclipai test-drive --harness opencode --model openrouter/anthropic/claude-sonnet-4
```

---

## 提供凭据

智能体需要服务提供商密钥。可通过以下三种方式提供给 `test-drive`，优先级依次递减：

- **`--api-key <value>`**——直接传入密钥，使用方便，但该值可能短暂出现在 shell 历史和进程参数中，因此 `test-drive` 会发出警告，并从自身的参数列表中清除它。
- **`--api-key-env <variable>`**——指定已保存密钥的环境变量名称。上述隔离步骤会保留此变量。
- **不传凭据选项**——`test-drive` 会回退到运行框架对应的默认变量（`ANTHROPIC_API_KEY`、`OPENAI_API_KEY` 或 `OPENROUTER_API_KEY`）。

`--api-key` 和 `--api-key-env` 互斥。如果找不到凭据，命令会在创建任何内容之前停止，并明确告知你应该设置哪个变量。

```sh
# Key already exported in your shell
export ANTHROPIC_API_KEY=sk-ant-...
paperclipai test-drive

# Key in a differently named variable
paperclipai test-drive --api-key-env MY_TEAM_ANTHROPIC_KEY
```

---

## 选项

| 标志 | 默认值 | 用途 |
|---|---|---|
| `-d, --data-dir <path>` | 新建临时目录 | 要创建或重用的 Paperclip 数据目录。省略时会使用隔离的临时目录。 |
| `--company-name <name>` | `Test Company` | 要创建的公司名称。 |
| `--agent-name <name>` | `CEO` | 要创建的 CEO 智能体名称。 |
| `--harness <harness>` | `claude` | 智能体初始运行框架：`claude`、`codex` 或 `opencode`。 |
| `--model <model-id>` | _适配器默认值_ | 智能体初始模型。对于 `opencode`，必须提供 `openrouter/<model>` 格式的模型。 |
| `--api-key-env <variable>` | _运行框架对应的默认变量_ | 从此环境变量读取服务提供商密钥。与 `--api-key` 互斥。 |
| `--api-key <value>` | _未设置_ | 直接传入服务提供商 API 密钥。与 `--api-key-env` 互斥。 |
| `--no-browser` | 自动打开浏览器 | 不在浏览器中打开初始化后的实例，只打印 URL。 |

> **提示：** 初始化标志（`--company-name`、`--agent-name`、`--harness`、`--model` 和凭据标志）仅在数据目录为空时生效。若将 `--data-dir` 指向现有的试用目录，命令会重用其中已有的配置。

---

## 另请参阅

- [run 命令](./run.md)——启动本地实例，但不执行临时试用初始化。
- [设置命令](./setup-commands.md)——用于长期使用的首次设置流程。
- [适配器命令](./adapter.md)——各运行框架所使用的 `claude_local`、`codex_local` 和 `opencode_local` 适配器。
- [通用选项](./common-options.md)——共享客户端标志及其解析顺序。
