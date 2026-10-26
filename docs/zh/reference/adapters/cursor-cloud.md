---
paperclip_version: v2026.512.0
seo_title: Cursor Cloud 适配器
seo_description: 通过官方 SDK 运行 Cursor Cloud Agents，由 Cursor 负责远程代码执行，Paperclip 则保留任务状态和历史记录。
---

# Cursor Cloud

`cursor_cloud` 通过官方 Cursor SDK 运行 Cursor Cloud Agents，由 Cursor 负责远程代码执行，同时让 Paperclip 保留任务状态。如果你希望远程 Cursor 智能体会话能够跨 Paperclip 心跳持续存在，请使用此适配器。

---

## 适用场景

- 希望 Paperclip 通过官方 Cursor SDK 运行 Cursor Cloud Agents。
- 希望远程 Cursor 智能体会话能够跨 Paperclip 心跳持续存在。
- 希望由 Cursor 负责远程代码执行，同时由 Paperclip 保留任务状态。

## 不适用场景

- 需要在与 Paperclip 相同的计算机上本地执行。请改用 [Cursor Local](./cursor-local.md)。
- 没有 Cursor API 密钥，或 Cursor 账户不允许使用 Cloud Agents。
- 智能体要执行的工作无法通过检出代码仓库来表达。

---

## 常见字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `repoUrl` | 是 | Cursor 应打开的 Git 仓库 URL。 |
| `repoStartingRef` | 否 | 仓库的起始引用。 |
| `repoPullRequestUrl` | 否 | 用于将智能体关联到某个拉取请求的 URL。 |
| `runtimeEnvType` | 否 | `cloud`、`pool` 或 `machine`。 |
| `runtimeEnvName` | 否 | 具名云环境、资源池或计算机目标。 |
| `workOnCurrentBranch` | 否 | 在当前分支上继续工作，而不创建新分支。 |
| `autoCreatePR` | 否 | 允许 Cursor 自动创建拉取请求。 |
| `skipReviewerRequest` | 否 | 对自动创建的 PR 不请求审阅者。 |
| `instructionsFilePath` | 否 | 添加到提示词前面的智能体指令文件。 |
| `promptTemplate` | 否 | 心跳提示词模板。 |
| `bootstrapPromptTemplate` | 否 | 仅首次运行时使用的引导提示词模板。 |
| `model` | 否 | Cursor 模型 ID。省略时使用账户默认模型。 |
| `env.CURSOR_API_KEY` | 是 | Cursor API 密钥。请使用 Paperclip 密钥引用。 |
| `env.*` | 否 | 注入 Cloud 智能体 shell 的其他环境变量。 |

---

## 会话持久化

只要仓库和运行时身份仍匹配，Paperclip 就会在多次心跳之间复用持久化的 Cursor 智能体。每次 Paperclip 心跳都会映射到该持久智能体上的一次 Cursor 运行。如果仓库 URL、起始引用或运行时环境发生更改，Paperclip 就会启动新的 Cursor 智能体。

---

## 执行详情

- Paperclip 通过 Cursor SDK 驱动智能体，而不是使用本地 CLI。
- Paperclip 通过 Cursor SDK 的 `cloud envVars` 通道，将 `PAPERCLIP_*` 运行时环境变量注入 Cloud 智能体 shell。
- 任务和问题状态仍以 Paperclip 为准；Cursor 提供远程执行界面。

---

---

## 示例

```json
{
  "adapterType": "cursor_cloud",
  "adapterConfig": {
    "repoUrl": "https://github.com/example-org/example-repo",
    "repoStartingRef": "main",
    "runtimeEnvType": "cloud",
    "workOnCurrentBranch": false,
    "autoCreatePR": true,
    "skipReviewerRequest": false,
    "promptTemplate": "You are the engineering lead. Work carefully and report progress.",
    "bootstrapPromptTemplate": "Bootstrap the working environment, then await instructions.",
    "env": {
      "CURSOR_API_KEY": {
        "type": "secret_ref",
        "secretId": "secret-id",
        "version": "latest"
      }
    }
  }
}
```

---

## 相关内容

- [Cursor Local](./cursor-local.md) — 同一智能体服务商，改用本地执行。
- [创建适配器](./creating-an-adapter.md) — 如果内置适配器都不适用，可自行开发适配器。
