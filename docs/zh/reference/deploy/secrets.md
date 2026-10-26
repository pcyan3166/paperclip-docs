---
paperclip_version: v2026.525.0
seo_title: 部署中的密钥
seo_description: 避免在适配器配置中明文存放敏感值，同时确保智能体可在运行时使用这些值。介绍默认密钥后端及其更改方法。
---

# 密钥

密钥可以避免在适配器配置中明文存放敏感值，同时确保智能体能够在运行时使用这些值。

如果你需要了解默认密钥服务提供商、配置严格模式，或将内嵌凭据迁移到托管密钥引用，请阅读本页。

---

## 默认服务提供商

内置服务提供商为 `local_encrypted`。

它使用本地主密钥存储密钥材料，密钥位于：

```txt
~/.paperclip/instances/default/secrets/master.key
```

该密钥会在引导过程中自动创建；除非你另行指定，否则只保存在本机。

> **注意：** 默认服务提供商面向本地和单机使用。其他部署方式可以沿用相同的密钥模型，但请仔细核对外围基础设施。

---

## 服务提供商密钥库

*服务提供商密钥库*是按公司划分的路由记录，用于告知 Paperclip 如何连接外部密钥存储服务（目前支持 AWS Secrets Manager；GCP Secret Manager 和 HashiCorp Vault 只能保存为草稿元数据）。每个密钥库保存服务提供商所需的地址、命名空间和前缀，绝不会保存凭据。服务器端凭据仍来自主机的常规凭据链（环境变量、实例角色、`~/.aws/credentials`）。

在 **Company Settings → Secrets → Provider vaults** 中管理密钥库。页面会针对每个服务提供商显示一个区域，并提供 **Add vault** 按钮；每个卡片还会显示健康状态、默认状态和移除控件。在 UI 中移除密钥库只会删除 Paperclip 侧的路由记录。对于 AWS Secrets Manager，确认对话框会明确说明：“此操作不会删除远程 AWS Secrets Manager 密钥库、密钥或任何 AWS 数据。”

AWS Secrets Manager 的创建表单包含 **Find existing AWS values** 发现步骤。它会扫描你指定区域中 `secretsmanager:ListSecrets` 返回的元数据，并根据你选中的候选项预填命名空间、名称前缀、KMS 密钥 ID 和标签字段。它不会读取密钥值。完整运维流程见[连接 AWS Secrets Manager 密钥库](../../how-to/connect-aws-secrets-vault.md)。

REST 接口与 UI 对应：`GET/POST /companies/{companyId}/secret-provider-configs`；AWS 扫描使用 `POST /companies/{companyId}/secret-provider-configs/discovery/preview`；单个密钥库的修改使用 `/secret-provider-configs/{id}` 上的 `PATCH`、`DELETE`、`POST .../default` 和 `POST .../health`。

---

## 配置密钥

常规设置流程：

```sh
pnpm paperclipai onboard
```

更新现有安装：

```sh
pnpm paperclipai configure --section secrets
```

验证配置：

```sh
pnpm paperclipai doctor
```

---

## 环境变量覆盖

| 变量 | 含义 |
|---|---|
| `PAPERCLIP_SECRETS_MASTER_KEY` | 32 字节密钥，可使用 base64、十六进制或原始字符串格式。 |
| `PAPERCLIP_SECRETS_MASTER_KEY_FILE` | 本地密钥文件的自定义路径。 |
| `PAPERCLIP_SECRETS_STRICT_MODE` | 要求服务端环境绑定使用密钥引用。不适用于 `paperclipai configure --section llm` 或 `config.llm.apiKey`。 |

通过以下方式启用严格模式：

```sh
PAPERCLIP_SECRETS_STRICT_MODE=true
```

如果希望阻止新的敏感值以内嵌明文的方式进入配置，请启用严格模式。

---

## 严格模式

严格模式是针对已保存环境绑定的服务端强制策略。启用后，`*_API_KEY`、`*_TOKEN` 和 `*_SECRET` 等敏感环境变量必须使用密钥引用；系统会拒绝内嵌的明文值。

它适用于服务器保存以下内容时：

- 智能体的 `adapterConfig.env`；
- 项目的 `env`；
- 包含适配器环境变量的招聘智能体审批正文；
- 包含智能体适配器环境变量的公司导入。

除了本地可信安装之外，其他情况都更适合启用此模式。

严格模式**不适用于** `paperclipai configure --section llm`。无论 `PAPERCLIP_SECRETS_STRICT_MODE` 如何设置，该命令都会将明文写入 `config.llm.apiKey`。

> **警告：** 严格模式会阻止新的敏感明文值，但不会自动迁移已经保存的值。

---

## 实例级密钥

`config.llm.apiKey` 是普通字符串字段，目前不支持 `secret_ref`，也没有 CLI 或 API 路径可以将实例级密钥保存到密钥存储中。

如果你希望全流程都只使用密钥引用，请将 `config.llm.apiKey` 留空，并按下文示例在 `adapterConfig.env` 中为每个智能体适配器 API 密钥绑定 `secret_ref`。

---

## 迁移内嵌密钥

如果现有智能体配置中仍包含内嵌的 API 密钥或 token，请将其迁移到托管密钥中：

```sh
pnpm secrets:migrate-inline-env
pnpm secrets:migrate-inline-env --apply
```

如果想先进行模拟运行，请先在不带 `--apply` 的情况下执行命令。

> **注意：** `pnpm secrets:migrate-inline-env` 是仓库源码脚本，只能从 Paperclip Git 检出目录中使用，无法通过已安装的 `paperclipai` CLI 使用。

此脚本只会扫描 `agent.adapterConfig.env` 中与敏感关键词正则表达式匹配的键，例如 `api_key`、`token`、`secret`、`password` 和 `credential`。它不会迁移 `config.llm.apiKey`。

---

## 智能体配置中的密钥引用

适配器环境变量应引用密钥，而不是直接嵌入明文：

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

运行时，服务器会解析密钥，通过已配置的服务提供商解密，并将明文注入智能体进程的环境。

对于预期会轮换的值，请使用 `version: "latest"`。只有在需要固定历史版本时，才指定数字版本。

> **提示：** 如果某个配置值不适合打印到日志中，它通常也应该使用密钥引用。

---

## 例程的 `env` 绑定

例程除了智能体适配器环境变量外，还可以有自己的 `env` 映射（`routines.env`）。格式相同：每个值可以是普通字符串，也可以是指向公司密钥存储的 `{ "type": "secret_ref", "secretId": "...", "version": "latest" }` 引用。严格模式同样适用于例程环境变量；保存敏感键时必须使用 `secret_ref`。

每次例程运行都会通过 `routine_runs.routine_revision_id` 固定到其执行时对应的例程修订版本，因此即使之后编辑例程，历史运行仍保留派发时使用的环境变量。UI 和 API 的设置步骤见[创建例程 — env 映射](../../how-to/create-a-daily-routine.md#4-optional-give-the-routine-an-env-map)。
