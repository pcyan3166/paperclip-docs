---
paperclip_version: v2026.609.0
seo_title: CLI 秘密管理命令
seo_description: 管理代理在运行时使用的 API 密钥、令牌和凭据，避免将原始值粘贴到代理配置中。
---

# 秘密管理命令

需要管理代理和项目在运行时使用的 API 密钥、令牌及其他凭据时，请使用这些命令，避免原始值进入 shell 历史或泄漏到日志中。`secrets` 命令组用于声明由 Paperclip 管理的秘密、连接外部提供商的保管库（例如 AWS Secrets Manager）、审查凭据的引用位置，并将内联环境变量值迁移为规范的秘密引用。

本页所有命令都限定在公司范围内，或针对特定秘密/提供商配置 ID 操作；同时接受[通用客户端选项](./common-options.md)（`--data-dir`、`--api-base`、`--api-key`、`--context`、`--profile`、`--json`）。公司范围的子命令必须提供 `-C, --company-id <id>`。

> **警告：** Paperclip 绝不会打印秘密值。`secrets list`、`declarations`、`usage` 和 `access-events` 只返回元数据——名称、密钥、提供商、状态和引用信息——不会返回明文。只有设置秘密时才会显示其值；即便如此，也建议使用 `--value-env`，让值不进入 shell 历史。请将此视为绝对保证，并据此设计脚本。

---

## 命令用途

| 命令 | 用途 |
|---|---|
| `secrets list` | 查看公司已有秘密的元数据。 |
| `secrets declarations` | 查看公司导出时生成的可移植环境变量声明（目标实例必须提供的内容）。 |
| `secrets create` | 保存新的 Paperclip 管理秘密值。 |
| `secrets link` | 注册由外部提供商保管的秘密，不将其值复制到 Paperclip。 |
| `secrets update` | 修改秘密元数据。 |
| `secrets rotate` | 替换由 Paperclip 管理的秘密值。 |
| `secrets usage` | 查找秘密的引用位置（哪些代理/项目使用了它）。 |
| `secrets access-events` | 审计秘密何时被读取以及由谁读取。 |
| `secrets delete` | 删除秘密（需要 `--yes` 和 `--confirm`）。 |
| `secrets doctor` | 通过 API 检查提供商健康状态。 |
| `secrets providers` | 列出公司可用的秘密提供商描述信息。 |
| `secrets provider-configs`（及子命令） | 管理公司级别的提供商保管库配置。 |
| `secrets remote-import` / `remote-import:preview` | 从已配置的远程来源导入秘密。 |
| `secrets migrate-inline-env` | 将代理环境变量中内联的敏感值转换为秘密引用。 |

---

## 列出和检查秘密

`secrets list` 会返回公司定义的所有秘密的元数据。人类可读输出会显示秘密的 `id`、`name`、`key`、`provider`、`status`、`managedMode`、`latestVersion` 及是否设置 `externalRef`——绝不会显示秘密值。

```sh
paperclipai secrets list --company-id <company-id>
paperclipai secrets list --company-id <company-id> --json
```

`secrets declarations` 从另一角度查看：它会询问公司的导出流程，了解导出公司包需要哪些可移植环境*输入*，从而准确列出导入公司运行前目标实例必须提供的密钥。每行包括 `key`、`scope`（`company`、`project:<slug>` 或 `agent:<slug>`）、`kind`（`secret` 或 `plain`）、`requirement`、`portability` 标志、是否 `hasDefault` 以及 `description`。

```sh
paperclipai secrets declarations --company-id <company-id>
paperclipai secrets declarations --company-id <company-id> --include agents,projects --kind secret
```

| 标志 | 用途 |
|---|---|
| `--include <values>` | 导出预览的逗号分隔包含项。可选一个或多个：`company,agents,projects,issues,tasks,skills`。默认值：`company,agents,projects`。 |
| `--kind <kind>` | 按类型筛选声明：`all`、`secret` 或 `plain`。默认值：`all`。 |

> **提示：** 将公司导出交给他人之前，应运行 `declarations`。它会告诉接收方需要自行提供哪些凭据，不会泄露你的凭据。

---

## 创建、链接、更新和轮换

### 创建 Paperclip 管理的秘密

`secrets create` 会保存由 Paperclip 加密并管理的值。必须且只能通过 `--value` 或 `--value-env` 之一提供值。始终优先使用 `--value-env`：它会从指定环境变量读取值，避免明文出现在 shell 历史、终端回滚缓冲区或进程参数列表中。

```sh
# Preferred: the value is read from the env var, not typed on the command line
export ANTHROPIC_API_KEY=sk-ant-...
paperclipai secrets create --company-id <company-id> --name anthropic-api-key --value-env ANTHROPIC_API_KEY
```

```sh
# Discouraged: --value puts the secret into shell history
paperclipai secrets create --company-id <company-id> --name anthropic-api-key --value 'sk-ant-...'
```

| 标志 | 用途 |
|---|---|
| `--name <name>` | 必填。秘密显示名称。 |
| `--key <key>` | 可移植秘密键（其他资源引用时使用的稳定键）。 |
| `--provider <provider>` | 秘密提供商 ID。省略时使用公司的默认提供商（本地托管加密）。 |
| `--value <value>` | 秘密值。不能与 `--value-env` 同时使用。 |
| `--value-env <name>` | 从指定环境变量读取值。不能与 `--value` 同时使用。变量未设置或为空时会报错。 |
| `--description <text>` | 自由文本描述。 |

同时传入 `--value` 和 `--value-env`，或两者都不传，都会报错。

### 链接外部提供商管理的秘密

`secrets link` 会通过引用注册存放在外部保管库（例如 AWS Secrets Manager）中的秘密。Paperclip 只记录指针——提供商、外部引用和可选版本引用——绝不将秘密值复制到自身存储。如果组织将托管保管库用作唯一可信来源，并希望 Paperclip 在运行时解析而非保留另一份副本，应选择此方式。

```sh
paperclipai secrets link \
  --company-id <company-id> \
  --name prod-stripe-key \
  --provider aws_secrets_manager \
  --external-ref arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/stripe-AbCdEf
```

| 标志 | 用途 |
|---|---|
| `--name <name>` | 必填。秘密显示名称。 |
| `--provider <provider>` | 必填。秘密提供商 ID，例如 `aws_secrets_manager`。 |
| `--external-ref <ref>` | 必填。提供商秘密 ARN、名称、路径或其他引用。 |
| `--key <key>` | 可移植秘密键。 |
| `--provider-version-ref <ref>` | 要固定到的提供商版本 ID 或标签。 |
| `--description <text>` | 自由文本描述。 |

链接的秘密会以 `managedMode: external_reference` 保存，因此 `secrets list` 会显示其 `externalRef=yes`。

### 更新元数据

`secrets update` 会修改秘密元数据。它接受原始 JSON 请求体，因此可以设置服务器 `UpdateSecret` schema 允许的任何字段。

```sh
paperclipai secrets update <secret-id> --payload-json '{"description":"Rotated quarterly by ops"}'
```

| 标志 | 用途 |
|---|---|
| `--payload-json <json>` | 必填。符合服务器更新秘密请求体 schema 的 JSON 对象。 |

> **注意：** `update` 修改的是元数据，而非秘密值。要更改值，请使用 `rotate`。

### 轮换秘密值

`secrets rotate` 会替换由 Paperclip 管理的秘密值并递增版本。与 `create` 一样，应通过 `--value` 或 `--value-env` 之一传入新值，并优先使用 `--value-env`。

```sh
export ANTHROPIC_API_KEY=sk-ant-new...
paperclipai secrets rotate <secret-id> --value-env ANTHROPIC_API_KEY
```

| 标志 | 用途 |
|---|---|
| `--value <value>` | 新的秘密值。不能与 `--value-env` 同时使用。 |
| `--value-env <name>` | 从指定环境变量读取新的秘密值。不能与 `--value` 同时使用。 |

---

## 审计使用和访问记录

轮换或删除凭据前，请查明哪些对象依赖它，以及哪些对象读取过它。

`secrets usage` 会报告秘密被引用的位置，即环境变量绑定指向该秘密的代理和项目。

```sh
paperclipai secrets usage <secret-id>
```

`secrets access-events` 会返回秘密的访问审计记录，方便确认哪些运行在何时读取过它。

```sh
paperclipai secrets access-events <secret-id> --json
```

> **提示：** 计划删除秘密时，务必先运行 `secrets usage`。如果代理环境变量仍引用该秘密，删除后会导致代理运行失败。

---

## 删除秘密

`secrets delete` 刻意设计得不易误操作。必须传入 `--yes`，并通过 `--confirm` 重复提供位置参数中的秘密 ID，两者必须完全匹配。

```sh
paperclipai secrets delete <secret-id> --yes --confirm <secret-id>
```

| 标志 | 用途 |
|---|---|
| `--yes` | 必填安全标志，确认执行破坏性操作。 |
| `--confirm <secretId>` | 必填。必须重复命令参数中传入的同一秘密 ID。 |

如果缺少 `--yes`，或 `--confirm` 与参数不匹配，命令会拒绝继续。

---

## 提供商健康状态和描述信息

`secrets doctor` 通过 Paperclip API 执行提供商健康检查，并为每个提供商打印一行结果，状态为 `ok`、`warn` 或 `error`，同时给出警告、缺少配置的提示、检测到的凭据来源以及备份建议。

```sh
paperclipai secrets doctor --company-id <company-id>
```

对 AWS 支持的秘密，`doctor` 会报告缺少的非秘密提供商环境变量，以及预期使用的 AWS SDK 运行时凭据来源。不要将 AWS 引导凭据存放在 Paperclip 秘密中——应让运行时通过标准 AWS 凭据链解析它们，并使用 `doctor` 确认该凭据链已正确配置。

`secrets providers` 会列出公司可用的提供商描述信息，即 `create`、`link` 和提供商配置命令可以选择的提供商集合。

```sh
paperclipai secrets providers --company-id <company-id>
```

---

## 提供商保管库配置

一家公司可以配置多个提供商保管库（例如多个 AWS Secrets Manager 实例），并将其中一个设为默认项。这些配置也可在看板 UI 的 `Company Settings → Secrets → Provider vaults` 中管理，但 CLI 提供所有相同操作。修改类子命令接受原始 `--payload-json`，因此与服务器契约保持一致。

```sh
# List existing configs
paperclipai secrets provider-configs --company-id <company-id>

# Create a config
paperclipai secrets provider-config:create --company-id <company-id> --payload-json '{ ... }'

# Preview which secrets a vault would discover, without importing them
paperclipai secrets provider-config:discovery-preview --company-id <company-id> --payload-json '{ ... }'

# Inspect, update, default, health-check, or delete a single config
paperclipai secrets provider-config:get <config-id>
paperclipai secrets provider-config:update <config-id> --payload-json '{ ... }'
paperclipai secrets provider-config:default <config-id>
paperclipai secrets provider-config:health <config-id>
paperclipai secrets provider-config:delete <config-id>
```

| 子命令 | 范围 | 必需标志 | 用途 |
|---|---|---|---|
| `provider-configs` | 公司 | `-C` | 列出公司的提供商保管库配置。 |
| `provider-config:create` | 公司 | `-C`、`--payload-json` | 创建保管库配置。 |
| `provider-config:discovery-preview` | 公司 | `-C`、`--payload-json` | 预览保管库发现的秘密，不导入。 |
| `provider-config:get <configId>` | 配置 | — | 显示单个配置。 |
| `provider-config:update <configId>` | 配置 | `--payload-json` | 修改单个配置。 |
| `provider-config:default <configId>` | 配置 | — | 将此配置设为公司的默认保管库。 |
| `provider-config:health <configId>` | 配置 | — | 检查单个配置的健康状态。 |
| `provider-config:delete <configId>` | 配置 | — | 删除单个配置。 |

---

## 远程导入

配置提供商保管库后，可以将选定秘密导入 Paperclip 注册表。请始终先预览：`remote-import:preview` 显示*将要*导入的内容，`remote-import` 才实际执行选定的导入。两者都接受 `--payload-json`，用于描述来源和选择范围。

```sh
paperclipai secrets remote-import:preview --company-id <company-id> --payload-json '{ ... }'
paperclipai secrets remote-import --company-id <company-id> --payload-json '{ ... }'
```

| 子命令 | 必需标志 | 用途 |
|---|---|---|
| `remote-import:preview` | `-C`、`--payload-json` | 预览将从远程导入哪些秘密。 |
| `remote-import` | `-C`、`--payload-json` | 导入选定的远程秘密。 |

---

## 将内联环境变量迁移为秘密引用

旧版代理配置可能会将敏感值直接写入适配器环境变量（例如字面值形式的 `API_KEY` 字符串）。`secrets migrate-inline-env` 会查找这些敏感内联值，为其创建或轮换 Paperclip 管理的秘密，并改写代理环境变量绑定，让其改为引用相应秘密。

默认情况下这是一次试运行。先在不带 `--apply` 的情况下运行，查看将执行的操作；确认后再添加 `--apply` 提交变更。

```sh
# Dry run: report agents to update, secrets to create, secrets to rotate
paperclipai secrets migrate-inline-env --company-id <company-id>

# Commit the migration
paperclipai secrets migrate-inline-env --company-id <company-id> --apply
```

| 标志 | 用途 |
|---|---|
| `--apply` | 持久保存变更。不传入时，命令只报告计划的迁移。 |

试运行结果会报告 `agentsToUpdate`、`secretsToCreate`、`secretsToRotate` 和完整候选列表。添加 `--apply` 后，新建秘密使用本地加密管理提供商；同名现有秘密会轮换为当前内联值；受影响代理的环境变量绑定会替换为指向 `version: latest` 的 `secret_ref`。

> **注意：** 系统根据环境变量键名判断其是否“敏感”，例如匹配 `*_token`、`api_key`、`password`、`secret`、`private_key`、`jwt` 等模式的键。未被标记为敏感的内联值会保持原样；应用迁移前请先检查试运行结果。

---

## 另请参阅

- [通用选项](./common-options.md)——所有秘密管理命令都接受的共享客户端标志。
- [输出与脚本](./output-and-scripting.md)——使用 `--json` 将秘密元数据传给其他工具。
- [代理命令](./agent.md)——环境变量绑定会引用秘密的代理。
- [项目命令](./project.md)——使用秘密的项目环境声明。
- [公司命令](./company.md)——导出/导入命令会生成 `secrets declarations` 所预览的环境变量声明。
- [访问命令](./access.md)——更广泛的授权和凭据上下文。
