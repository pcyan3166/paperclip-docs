---
paperclip_version: v2026.1012.0
seo_title: 更新或轮换提供商 API 密钥
seo_description: 替换正在运行的智能体所使用的凭证——在提供商轮换密钥、密钥过期，或从订阅登录迁移到控制台颁发的密钥之后。
---

# 更新或轮换提供商 API 密钥

您已有一个正在运行的智能体，且其使用的 API 密钥需要更改——可能是提供商进行了轮换，可能是密钥已过期，也可能是您从订阅登录（如 Claude.ai）迁移到了控制台颁发的密钥，导致旧凭证失效。本指南将指导您原地替换密钥，而无需重建智能体。

如果您是首次设置智能体，请遵循 [雇佣您的第一个智能体](../guides/getting-started/your-first-agent.md)——该指南涵盖了从零开始创建密钥并进行配置。

---

## 存储 API 密钥的两种方式

Paperclip 允许您以两种形式将提供商密钥绑定到智能体的 `adapterConfig.env` 中。了解您使用的是哪种形式，决定了您在哪里更改该值。

| 存储形式 | 外观 | 轮换位置 |
|---|---|---|
| **纯值** | 密钥直接粘贴到智能体的环境变量字段中。 | 在智能体本身——打开适配器表单，粘贴新值，保存。 |
| **密钥引用** | 智能体的环境变量条目指向 Paperclip 密钥存储中的某个密钥，通常使用 `version: "latest"`。 | 在 **公司设置 → 密钥** 中——只需轮换一次密钥，所有绑定到该密钥的智能体都会获取新值。 |

> **提示：** 如果您在同一个提供商上有多个智能体，请将密钥存储为密钥引用。这样轮换就变成了一次编辑，而不是每个智能体都要操作一次。

---

## 路径 A — 在智能体上轮换（纯值）

当密钥是直接粘贴到适配器表单中时，使用此路径。

1. 打开 **智能体 → {agent}**，并在智能体侧边栏中选择 **Harness / Runtime**。
2. 滚动到 **环境变量** 卡片。
3. 找到提供商的密钥名称（例如 `ANTHROPIC_API_KEY`），并将值替换为新密钥。
4. 点击 **保存更改**。
5. 在底部的 **测试您的智能体** 卡片中，点击 **运行测试**。适配器会启动对提供商的检查；**连接成功** 表示新密钥已通过身份验证。

下一次心跳将获取新值。无需重启。

![智能体 Harness / Runtime 页面上的环境变量卡片，其中 GITHUB_TOKEN 行绑定到密钥，并有一个添加变量链接](../../user-guides/screenshots/light/agents/test-environment-fail.png)

> **警告：** 纯值存储在智能体记录中。如果您正在使用 `PAPERCLIP_SECRETS_STRICT_MODE=true`，服务器将拒绝任何名称匹配敏感关键词正则表达式（`api_key`, `token`, `auth`, `authorization`, `bearer`, `secret`, `password`, `credential`, `jwt`, `private_key`, `cookie`, `connectionstring`）的环境键的新内联值。您可以暂时关闭严格模式，或使用路径 B。

---

## 路径 B — 轮换密钥（密钥引用）

当智能体上的环境变量条目看起来像这样时，使用此路径：

```json
{
  "env": {
    "ANTHROPIC_API_KEY": {
      "type": "secret_ref",
      "secretId": "8f884973-c29b-44e4-8ea3-6413437f8081",
      "version": "latest"
    }
  }
}
```

1. 打开 **公司设置 → 密钥**。
2. 找到智能体引用的密钥——它通常以提供商命名（`anthropic-key`, `openai-key` 等）。
3. 点击 **轮换**，粘贴新值并确认。Paperclip 将其存储为新版本，并递增 `latestVersion`。
4. 打开智能体并点击 **测试环境** 以确认新值正在被注入。

每个通过 `version: "latest"` 引用此密钥的智能体，现在将在其下一次心跳时解析为新值。固定到特定数字版本（`"version": "3"`）的智能体将保持该历史值，直到您重新指向它们。

> **注意：** 每次轮换都会自动创建新版本——没有“原地编辑值”的选项。旧版本仍可由任何固定到它的绑定解析；新绑定和 `latest` 消费者将向前移动。

---

## 各提供商注意事项

大多数提供商遵循相同的模式——在提供商控制台中生成新密钥，复制一次（大多数控制台仅在创建时显示密钥），然后将其放入上述的路径 A 或路径 B。区别在于环境变量名称和密钥格式。

<!-- tabs: Anthropic (Claude), OpenAI, Google (Gemini), xAI (Grok), Cursor, OpenCode -->

<!-- tab: Anthropic (Claude) -->

**环境变量：** `ANTHROPIC_API_KEY`
**密钥前缀：** `sk-ant-`
**控制台：** [console.anthropic.com](https://console.anthropic.com) → **API 密钥** → **创建密钥**

> **信息：** 正在从 Claude.ai 订阅登录迁移到控制台 API 密钥吗？这是打开本文档的最常见原因。`claude_local` 的旧版本接受 Claude Code 的订阅身份验证；新版本需要来自 [console.anthropic.com](https://console.anthropic.com) 的显式 `ANTHROPIC_API_KEY`。生成控制台密钥，通过路径 A 或路径 B 存储，适配器将开始使用它而不是订阅会话。

`claude_local` 适配器也可以通过 Bedrock 设置或 Claude Code 自身的登录状态进行身份验证。如果您切换到其中一种，请清除 `ANTHROPIC_API_KEY` 条目，而不是保留旧值。参见 [Claude Code](../reference/adapters/claude-code.md)。

<!-- tab: OpenAI -->

**环境变量：** `OPENAI_API_KEY`
**密钥前缀：** `sk-`
**控制台：** [platform.openai.com](https://platform.openai.com) → 个人资料菜单 → **API 密钥** → **创建新密钥**

被 `codex_local` 使用，当配置的模型是 OpenAI 模型时也被 `opencode_local` 使用。Codex 自身的 CLI 登录状态也满足适配器要求——如果您已运行 `codex login`，可以留空 `OPENAI_API_KEY`。

<!-- tab: Google (Gemini) -->

**环境变量：** `GEMINI_API_KEY` 或 `GOOGLE_API_KEY`（两者均可接受）
**控制台：** [aistudio.google.com](https://aistudio.google.com) → **获取 API 密钥**

被 `gemini_local` 使用。适配器还接受 Google 账户登录或 Gemini CLI 身份验证，因此在开发机器上您可能根本不需要密钥。参见 [Gemini CLI](../reference/adapters/gemini-cli.md)。

> **注意：** 测试环境确认二进制文件已安装且环境变量已设置。它不会检测配额耗尽或 Google 账户被撤销——通过的检查并不能保证运行会成功。

<!-- tab: xAI (Grok) -->

**环境变量：** `XAI_API_KEY`
**控制台：** [console.x.ai](https://console.x.ai) → **API 密钥**

被 `grok_local` 使用。将其与其他提供商密钥同等对待——生成，存储为密钥引用，绑定到适配器的环境变量中。参见 [Grok Local](../reference/adapters/grok-local.md)。

<!-- tab: Cursor -->

**本地 (`cursor`)：** 没有 API 密钥字段。本地适配器使用 Cursor Agent CLI 自身的登录会话——通过在主机终端上运行 `cursor login` 重新进行身份验证。无需更改 Paperclip。

**云端 (`cursor_cloud`)：** 环境变量 `CURSOR_API_KEY`。通过路径 A 或路径 B 轮换；从您的 Cursor 账户设置中生成密钥。参见 [Cursor Cloud](../reference/adapters/cursor-cloud.md)。

<!-- tab: OpenCode -->

`opencode_local` 是一个多提供商运行时。您轮换的密钥是与适配器上配置的 `provider/model` 匹配的那个：

- `anthropic/...` → `ANTHROPIC_API_KEY`
- `openai/...` → `OPENAI_API_KEY`
- `google/...` → `GEMINI_API_KEY` 或 `GOOGLE_API_KEY`
- `xai/...` → `XAI_API_KEY`

如果智能体使用两个提供商（例如一个模型提供商和一个嵌入提供商），请轮换两者。参见 [OpenCode](../reference/adapters/opencode.md)。

<!-- /tabs -->

---

## 验证新密钥是否有效

1. 打开智能体并点击 **测试环境**。这将确认二进制文件可访问，环境变量已解析，且提供商接受该凭证。
2. 触发一次心跳——要么开启心跳并等待下一次触发，要么为智能体分配一个小任务。观察运行日志以确认首轮成功。
3. 如果您轮换的是共享密钥，请在引用该密钥的第二个智能体上重复检查，以确保轮换已生效。

---

## 故障排除

**“我添加了新密钥，但智能体仍然返回 401”** — 在 **公司设置 → 密钥** 中创建密钥 *不会* 自动重新连接现有智能体。智能体仍然使用您上次保存时配置在其 `ANTHROPIC_API_KEY`（或等效项）环境变量行中的内容。打开智能体 → **编辑** → **环境变量**，将该行的来源更改为 **密钥**，选择新密钥，然后点击 **测试环境**。如果该行已设置为 **纯值** 且包含旧密钥，则该纯值会遮蔽您在其他地方创建的任何新密钥——将其切换为 **密钥** 即可修复。

**轮换后提供商返回 401** — 提供商（而非 Paperclip）正在拒绝该凭证，这意味着请求携带了 *某个* 密钥到达了提供商。两个常见原因：(1) 密钥值本身错误——周围粘贴了空白字符，从错误的账户/工作区复制，或新的控制台账户尚未充值；(2) 智能体获取了旧的纯值而非轮换后的密钥。从提供商控制台重新复制密钥，暂时将其粘贴为纯值以隔离哪一侧出错，然后在测试环境通过后将其移回密钥引用。

**“API 密钥无效”** — 将环境变量 *名称* 与提供商预期的名称进行比较（`ANTHROPIC_API_KEY`，而非 `ANTHROPIC_KEY`）。检查密钥前缀是否与提供商颁发的匹配（`sk-ant-`, `sk-` 等）。在密钥前后粘贴空白字符是常见原因。

**轮换后测试环境仍然失败** — 智能体可能持有缓存的进程。停止并重新启用心跳，以便下一次运行以新环境启动新的子进程。对于 `cursor` 等长期运行的适配器，结束现有会话，以免恢复缓存重用旧身份验证。

**严格模式拒绝内联值** — 您正在使用 `PAPERCLIP_SECRETS_STRICT_MODE=true`，这会阻止持久化环境变量绑定中的内联敏感值。请改用路径 B（密钥引用）。参见 [密钥](../reference/deploy/secrets.md)。

**旧订阅登录持续被使用** — Claude Code、Codex CLI 和 Gemini CLI 各自在 Paperclip 外部缓存其登录状态。如果您希望 API 密钥优先，请在主机上注销 CLI（`claude logout`, `codex logout` 等），以便适配器回退到环境变量。

**多个智能体仍然失败** — 如果您轮换了一个密钥，但只有部分智能体获取了新值，请检查失败智能体的环境变量绑定。固定到特定版本（`"version": "3"`）的智能体不会跟随 `latest`；请重新指向它们或更新绑定。

---

## 相关

- [智能体适配器](../guides/org/agent-adapters.md) — 每个适配器的完整配置参考。
- [密钥](../reference/deploy/secrets.md) — 密钥存储的工作原理及严格模式的行为。
- [雇佣您的第一个智能体](../guides/getting-started/your-first-agent.md) — 首次设置，包括生成 API 密钥。
