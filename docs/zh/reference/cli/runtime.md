---
paperclip_version: v2026.1023.0
seo_title: 安装 Paperclip Runner 运行时
seo_description: 在受支持的主机上安装并验证 Paperclip 固定版本的 Cursor 和 Pi 运行时，导入 Linux Pi 配套运行时，然后配置 Runner 智能体。
---

# 运行时命令

使用实验性 [Paperclip Runner](../adapters/paperclip-runner.md) 准备主机，以便运行 Cursor 或 Pi。运行时设置会下载并验证固定版本的运行时，不会调用模型。

```sh
paperclipai runtime setup cursor
paperclipai runtime setup pi
```

## 设置 Cursor

请在运行 Paperclip 的同一操作系统用户身份下，于智能体实际执行的机器上运行命令。此操作要求已安装 Paperclip 服务器软件包。支持 macOS ARM64、macOS x64 和 Linux x64。

| 命令 | 效果 |
|---|---|
| `paperclipai runtime setup cursor` | 下载、展开并验证适用于当前主机的固定版本 Cursor 发行包，或验证已有安装。 |

受支持的服务提供商参数为 `cursor` 和 `pi`。此命令没有特定服务提供商的标志。

安装文件使用操作系统账户在 `~/.paperclip/runtimes/cursor` 下的缓存。运行时身份包含平台、架构和经过验证的内容哈希。已有安装无效时会被拒绝，不会自动替换。执行镜像随附的打包运行时始终优先。

<a id="set-up-pi"></a>
## 设置 Pi

`paperclipai runtime setup pi` 会下载、展开并验证适用于当前主机的固定版本 Pi 发行包，或验证已有安装。请从已安装的 Paperclip 发行包中，以运行 Paperclip 的操作系统用户身份执行。支持 macOS ARM64、macOS x64 和 Linux x64。其他平台会以 `Pi is not qualified for platform <target>` 失败。

Pi 文件保存在已安装的服务器软件包内部，而不是用户主目录中。下载时会透传代理和证书设置（`HTTPS_PROXY`、`HTTP_PROXY`、`NO_PROXY`、`SSL_CERT_FILE`、`SSL_CERT_DIR`、`NODE_EXTRA_CA_CERTS`），但绝不透传服务提供商密钥、npm 配置或 `HOME`。如果命令未完成，请查看报告的错误；已有的无效安装不会自动替换。

### 导入 Linux Pi 配套运行时

对于远程 Linux 执行，可以导入已下载的 Pi 配套运行时，整个流程不会再下载文件：

```sh
paperclipai runtime import-remote <directory> --sha256 <digest>
```

| 参数或标志 | 用途 |
|---|---|
| `<directory>` | 存放已解包配套运行时的目录，其中必须包含 `companion.json`。 |
| `--sha256 <digest>` | 必填。`companion.json` 的 SHA-256 值，取自可信发行包。 |

导入前，此命令会根据清单检查每个文件，并以 JSON 格式输出结果。按 Ctrl+C 可取消。

## 配置智能体

完成设置后，如果部署要求启用 Runner，请在 Experimental 设置中开启 **Paperclip Runner**。选择 `paperclip_runner` 适配器、**ACP agents** 和 **Cursor**，然后选择明确的模型和 Cursor 会话模式。

设置过程不会让你登录 Cursor、修改智能体适配器或迁移旧版 `cursor` 智能体。请单独配置 Cursor 凭据，并在开始工作前使用 **Test Environment**。参见 [Paperclip Runner 上的 Cursor](../adapters/cursor-local.md#cursor-on-paperclip-runner)。

## 故障排查

| 问题 | 后续步骤 |
|---|---|
| 找不到命令 | 使用包含 `runtime setup` 的 CLI 构建版本。 |
| 无法解析服务器软件包 | 从已安装且包含已发布服务器目录结构的 Paperclip 发行包中运行命令。 |
| 平台没有固定版本发行包 | 使用受支持的执行目标平台之一。 |
| 验证失败 | 查看报告的安装错误。重复运行设置不会覆盖无效文件。 |
| 智能体仍要求身份验证 | 安装运行时与 Cursor 身份验证是两个独立步骤。 |

## 另请参阅

- [Paperclip Runner](../adapters/paperclip-runner.md)
- [Cursor Local](../adapters/cursor-local.md)
- [Pi](../adapters/pi.md)
- [环境](../../experimental/environments.md)
