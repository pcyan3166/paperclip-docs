---
paperclip_version: v2026.1012.0
seo_title: managed-agent 命令
seo_description: 配置并验证受限的 Anthropic 托管智能体及其运行环境，然后保存 Paperclip 用于运行智能体的公司配置。
---

# managed-agent 命令

`managed-agent` 用于配置远程、由服务提供商托管的运行时；公司可使用它替代本地执行的适配器。目前该命令面向 Anthropic Managed Agents beta：它会在 Anthropic 端创建（或接管）受限的 Agent 和 Environment，检查两者是否符合 Paperclip 严格的“无网络、无工具”安全配置，然后将**托管智能体配置**保存到公司中，供运行时派发工作时读取。

```sh
paperclipai managed-agent setup --company-id <company-id> \
  --profile-key <key> \
  --display-name "<name>" \
  --api-key-secret-id <uuid>
```

这是由控制面板操作员执行的一次性配置任务，而不是每次心跳运行都要执行的操作。你只需为公司配置一次托管服务提供商；之后由运行时负责派发工作。

> **注意：** 此命令会从 CLI 进程环境读取 `ANTHROPIC_API_KEY`，以便直接与 Anthropic 通信。运行前请导出有效密钥，否则命令会在执行其他操作前停止。

---

## `managed-agent setup`

`setup` 是唯一的子命令。它会创建或接管受限的 Anthropic Agent 和 Environment，验证两者，并保存生成的公司配置。

在后台，命令会根据你传入的 `--profile-key`（或显式指定的 ID）查找已有的 Anthropic 资源，找不到时则创建新资源，并应用固定的安全配置：Environment 网络访问受限、不允许任何主机且不安装任何软件包；Agent 不配置工具、MCP 服务器、技能或多智能体设置。如果接管的资源不符合此配置，`setup` 会拒绝使用，而不会冒险信任它们。

```sh
# Provision from scratch, acknowledging the beta's retention terms
paperclipai managed-agent setup --company-id <company-id> \
  --profile-key acme-prod \
  --display-name "Acme Production" \
  --api-key-secret-id 6f1c2e3a-1b2c-4d5e-8f90-0a1b2c3d4e5f \
  --acknowledge-retention

# Adopt an Anthropic Agent and Environment you already created
paperclipai managed-agent setup --company-id <company-id> \
  --profile-key acme-prod \
  --display-name "Acme Production" \
  --api-key-secret-id 6f1c2e3a-1b2c-4d5e-8f90-0a1b2c3d4e5f \
  --agent-id <anthropic-agent-id> \
  --environment-id <anthropic-environment-id> \
  --acknowledge-retention
```

### 必需选项

| 标志 | 用途 |
|---|---|
| `--profile-key <key>` | **必需。** 稳定的公司配置键。此值也会标记 Anthropic 资源，以便后续运行 `setup` 时再次找到它们。 |
| `--display-name <name>` | **必需。** 便于阅读的配置显示名称，也用于为 Anthropic Agent 和 Environment 命名。 |
| `--api-key-secret-id <id>` | **必需。** 现有公司密钥的 ID；该密钥保存 `ANTHROPIC_API_KEY`。必须是 UUID。 |

### 可选选项

| 标志 | 默认值 | 用途 |
|---|---|---|
| `--model <id>` | `claude-sonnet-5` | 固定使用的 Claude 模型。必须是已通过资格验证的 Managed Agents 模型 `claude-sonnet-5`。 |
| `--max-session-list-cost-usd <usd>` | `1.00` | 默认的会话费用硬上限。解析后的金额必须至少为 1 美分。 |
| `--agent-id <id>` | _未设置_ | 接管现有 Anthropic Agent，而非创建新 Agent。 |
| `--agent-version <version>` | _未设置_ | 固定使用指定版本的现有 Agent。 |
| `--environment-id <id>` | _未设置_ | 接管现有 Anthropic Environment，而非创建新环境。 |
| `--probe` | `false` | 仅进行只读资格验证。验证或发现资源后，打印**将要保存**的配置；不会创建或持久化任何内容。 |
| `--acknowledge-retention` | `false` | **实际配置时必需。** 确认接受 beta 的数据保留条款，并知悉该服务不符合 ZDR 和 HIPAA 要求。 |

`setup` 也接受标准客户端选项，并且按公司限定范围：必须通过标志或上下文配置文件提供 `-C, --company-id <id>`，因为最终配置会保存到该公司。详见[通用选项](./common-options.md)。

> **提示：** 建议先使用 `--probe`。它会完整执行资源发现和验证流程，并显示即将保存的准确配置，以便你在正式运行提交配置前确认模型、费用上限和接管的资源。

---

## 保存的内容

正式运行（非 probe）时，`setup` 会将配置保存到公司，记录配置键和显示名称、解析出的 Anthropic Agent ID 及固定版本、Environment ID、默认模型和会话费用上限、保存密钥的密钥 ID，以及描述已验证安全配置的资格验证记录。使用 `--probe` 时，会打印相同配置而不保存。

---

## 另请参阅

- [密钥命令](./secrets.md)——创建公司密钥，并将其 ID 传给 `--api-key-secret-id`。
- [适配器命令](./adapter.md)——与此托管服务提供商并列的本地执行适配器。
- [智能体命令](./agent.md)——配置完成后使用此配置运行的智能体。
- [通用选项](./common-options.md)——共享客户端标志和公司解析方式。
