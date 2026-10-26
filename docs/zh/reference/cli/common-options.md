---
paperclip_version: v2026.609.0
seo_title: CLI 通用选项和连接
seo_description: 几乎所有控制平面命令共享的连接标志、API 基础地址解析规则和上下文处理方式。
---

# 通用选项和连接

几乎所有与控制平面通信的 Paperclip CLI 命令都使用相同的一组连接标志、API 基础地址解析规则和上下文配置文件机制。读完本页后，你就会了解其他参考页面如何解析 `--api-base`、`--api-key` 和公司作用范围——它们都链接到此处，而不重复说明。命令无法连接、使用了错误的身份，或你不想每次都重复输入相同标志时，请查阅本页。

---

## 共享客户端标志

客户端命令会注册一组通用选项。CLI 进行 API 调用的任何命令都接受这些标志：

| 标志 | 用途 |
|---|---|
| `-d, --data-dir <path>` | Paperclip 数据目录根路径。将所有本地状态（配置、上下文、数据库、日志、存储、密钥）与 `~/.paperclip` 隔离。用于干净的测试实例和每个工作树独立配置时不可或缺。 |
| `--api-base <url>` | Paperclip API 的基础 URL。优先级最高，用于覆盖 CLI 的连接目标。 |
| `--api-key <token>` | 身份验证调用使用的 Bearer 令牌。优先级最高的凭据；传入此标志会禁用交互式控制面板身份验证恢复（见下文）。 |
| `--context <path>` | 要读取和写入的 CLI 上下文文件路径。覆盖默认的 `~/.paperclip/context.json` 查找路径。 |
| `--profile <name>` | 要使用的上下文配置文件名称。默认使用上下文中的当前配置文件。 |
| `-c, --config <path>` | Paperclip 配置文件路径。只有在未指定其他 API 基础地址时，才用于推断本地服务器端口。 |
| `--json` | 输出原始 JSON，而不是人类可读的内联格式。详见[输出和脚本](output-and-scripting.md)。 |

按公司限定范围的命令还会增加一个标志：

| 标志 | 用途 |
|---|---|
| `-C, --company-id <id>` | 公司 ID。覆盖此命令所使用配置文件中的默认公司。 |

> **注意：** 只有选择了公司作用范围的命令才会注册 `--company-id` 的简写 `-C`。并非所有命令都按公司限定范围；如果不确定，请查看相应命令的参考页面。

---

## API 基础地址的解析方式

CLI 按以下顺序检查各个来源，并使用第一个非空值作为 API 基础地址。随后会规范化解析结果（去掉末尾斜杠）：

1. 命令行中的 `--api-base <url>`。
2. `PAPERCLIP_API_URL` 环境变量。
3. 所选上下文配置文件的 `apiBase`。
4. 从 Paperclip 配置文件推断的本地地址：`http://<host>:<port>`。主机来自 `PAPERCLIP_SERVER_HOST`（默认值为 `localhost`），端口依次来自 `PAPERCLIP_SERVER_PORT` 和配置文件中的 `server.port`。
5. 最后的回退地址：`http://localhost:3100`。

由于采用此顺序，在本地实例上通常无需额外配置即可运行 CLI，同时也可以通过单个标志或环境变量指向远程服务器。连接失败时，错误会显示实际尝试的 URL，并提示检查 `GET /api/health`，以确认服务器可访问。

> **提示：** 如果连接到了错误的地址，可先为单条命令明确设置 `--api-base`，确认目标服务健康；然后将可用地址保存到配置文件中，避免继续受解析顺序影响。

---

## 凭据的解析方式

API 密钥与 API 基础地址分别解析，同样采用第一个匹配项优先的规则：

| 顺序 | 来源 | `authSource` |
|---|---|---|
| 1 | `--api-key <token>` | `explicit` |
| 2 | `PAPERCLIP_API_KEY` 环境变量 | `env` |
| 3 | 调用时读取配置文件的 `apiKeyEnvVarName` **所指名的**环境变量 | `profile_env` |
| 4 | 以解析出的 API 基础地址为键保存的控制面板凭据 | `stored_board` |

配置文件绝不保存明文令牌。它保存的是环境变量的*名称*（`apiKeyEnvVarName`）；CLI 运行时再读取该变量的值。这是将智能体密钥关联到配置文件的推荐方式——密钥留在环境变量或密钥管理器中，上下文文件则可以安全提交或分享。

如果前三个来源均未找到明确的密钥，CLI 会回退使用与解析后 API 基础地址关联的已存控制面板凭据（通过 `auth login` 或连接向导创建）。如果调用仍返回 `401`，或返回表示需要控制面板访问权限或实例管理权限的 `403`，并且当前是交互式 TTY、同时你没有传入 `--api-key`，CLI 会尝试在命令执行过程中通过交互式控制面板身份验证恢复来生成新令牌。传入 `--api-key` 会完全选择退出此恢复流程；在脚本和 CI 中应如此设置。

> **警告：** 交互式身份验证恢复只会在 TTY 环境中触发。无头环境中不会出现提示；请预先通过 `--api-key`、`PAPERCLIP_API_KEY` 或配置文件的 `apiKeyEnvVarName` 提供有效凭据，否则命令会返回底层身份验证错误。

---

## 公司作用范围的解析方式

按公司限定范围的命令按以下顺序解析公司 ID：

1. `--company-id <id>`（别名 `-C`）。
2. `PAPERCLIP_COMPANY_ID` 环境变量。
3. 所选配置文件的 `companyId`。

如果命令需要公司，但以上来源都未设置，则会报错：

```
Company ID is required. Pass --company-id, set PAPERCLIP_COMPANY_ID, or set context profile companyId via `paperclipai context set`.
```

---

## 上下文配置文件

配置文件保存在 CLI 上下文文件中。默认文件为 `~/.paperclip/context.json`，其版本为 2，包含一个 `currentProfile` 字段及一个以名称为键的 `profiles` 映射表。上下文路径依次通过 `--context`、`PAPERCLIP_CONTEXT` 环境变量和从当前目录向上查找最近的 `.paperclip/context.json` 来确定；都未找到时使用默认主目录路径。该文件权限设置为 `0600`。

每个配置文件都包含身份类型信息，可保存以下字段：

| 字段 | 含义 |
|---|---|
| `apiBase` | 此配置文件的默认 API 基础 URL。 |
| `companyId` | 默认公司 ID。 |
| `persona` | `board` 或 `agent`——配置文件使用的身份类型。 |
| `agentId` | 默认智能体 ID（智能体身份）。 |
| `agentName` | 默认智能体显示名称（智能体身份）。 |
| `apiKeyEnvVarName` | 保存 API 密钥的环境变量名称（建议使用此方式，不要直接存储令牌）。 |

两种身份类型有重要区别：`board` 配置文件在整个实例中具有完整的控制面板权限；`agent` 配置文件则精确限定在一个公司和一个智能体中。各类凭据的生成方式见[身份验证](./authentication.md)。

### 管理配置文件

```sh
# Inspect the active profile and the full store
paperclipai context show

# List every profile with its key fields
paperclipai context list

# Switch the active profile
paperclipai context use default
```

`context set` 会将值写入某个配置文件（默认是当前配置文件；也可通过 `--profile` 指定其他配置文件），如果该配置文件不存在则会创建：

```sh
# A board-operator profile pointed at a local instance
paperclipai context set \
  --api-base http://localhost:3100 \
  --company-id <company-id> \
  --persona board \
  --use

# An agent profile that reads its key from an env var
paperclipai context set --profile my-agent \
  --persona agent \
  --agent-id <agent-id> \
  --agent-name "Ops Agent" \
  --api-key-env-var-name PAPERCLIP_API_KEY \
  --use
```

`context set` 接受 `--api-base`、`--company-id`、`--persona`（必须是 `board` 或 `agent`）、`--agent-id`、`--agent-name` 和 `--api-key-env-var-name`。添加 `--use` 可同时将配置文件设为当前活动配置文件。为字段传入空值会将其从配置文件中清除。

> **提示：** 不要将密钥写入文件。在配置文件中设置 `--api-key-env-var-name PAPERCLIP_API_KEY`，并在 shell 中导出实际令牌：
>
> ```sh
> paperclipai context set --api-key-env-var-name PAPERCLIP_API_KEY
> export PAPERCLIP_API_KEY=...
> ```

---

## 使用 `--data-dir` 隔离状态

`--data-dir` 会将 Paperclip 所有本地状态（配置、上下文、数据库、日志、存储和密钥）重定向到你指定的目录。需要共享同一个隔离根目录的命令都应传入此标志：

```sh
paperclipai run --data-dir ./tmp/paperclip-dev
paperclipai issue list --data-dir ./tmp/paperclip-dev
```

这样可以避免实验、测试实例和并行工作树与真实的 `~/.paperclip` 状态相互冲突。

---

## 另请参阅

- [身份验证](./authentication.md)——生成控制面板令牌和智能体密钥，以及连接向导。
- [输出和脚本](output-and-scripting.md)——`--json` 和机器可读输出。
- [安装](installation.md)——安装 `paperclipai` 二进制文件。
- [概览](overview.md)——CLI 在整体操作模式中的作用。
