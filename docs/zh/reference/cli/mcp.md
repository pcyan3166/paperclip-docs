---
paperclip_version: v2026.1023.0
seo_title: MCP 设备登录和代理命令
seo_description: 通过浏览器设备授权让助手登录，然后使用 CLI 私有凭据存储，通过本地 stdio 暴露 Paperclip 工具。
---

# MCP 命令

使用 `mcp` 命令，以你的人类 Paperclip 账户连接助手。设备登录无需本地浏览器回调监听器；代理则允许仅支持 stdio 的 MCP 客户端访问已授权的远程工具。

助手连接属于实验功能，默认关闭。使用这些命令需要启用了[助手连接（MCP）](../../experimental/assistant-connections.md)的身份验证实例。

```sh
paperclipai mcp login --device --url https://YOUR-PAPERCLIP-HOST/mcp/paperclip
paperclipai mcp proxy --url https://YOUR-PAPERCLIP-HOST/mcp/paperclip
```

## 使用设备登录授权

```sh
paperclipai mcp login --device --url <url> [--company <id>]
```

| 选项 | 用途 |
|---|---|
| `--device` | 必填。在不启动回调监听器的情况下开始浏览器审批流程。 |
| `--url <url>` | 必填。实例显示的确切规范 MCP URL。 |
| `--company <id>` | 将授权范围限制为指定的组织 UUID。 |

保持命令运行。它会显示审批 URL 和供人类核对的验证码，然后继续等待。打开 URL、登录、比对验证码、查看组织和权限，然后批准或拒绝。请求十分钟后过期；命令过期时请重新开始。

审批 URL 会打开实例的 `/mcp-device` 页面，并自动填入验证码。如果你自行打开 `/mcp-device`，请在 **Enter the code shown by your assistant** 下方输入验证码。批准后，命令会打印包含 MCP URL 的 `Connected to` 行，以及包含获批组织 ID 的 `Organization:` 行。

CLI 会请求读取、写入、配置和刷新权限。你实际获得哪些权限范围取决于授权结果和你的组织角色。保存凭据前，登录流程会通过 `paperclip_connection` 验证获批组织。

请使用以 `/mcp/paperclip` 结尾的 HTTPS URL，不得包含凭据、查询参数或片段。只有 localhost 或回环地址可使用 HTTP。设置页面 URL `/mcp/setup` 不是 MCP 资源 URL。

## 运行 stdio 代理

```sh
paperclipai mcp proxy --url <url>
```

将此命令配置为助手中的本地 stdio MCP 服务器。请使用登录时的准确 URL。代理支持 `initialize`、`ping`、`tools/list` 和 `tools/call`；不提供 Events 订阅接口。

例如，接受命令和参数配置的客户端可以这样启动：

```json
{
  "command": "paperclipai",
  "args": ["mcp", "proxy", "--url", "https://YOUR-PAPERCLIP-HOST/mcp/paperclip"]
}
```

客户端外围配置格式可能有所不同。开始工作前，请从已连接的客户端调用 `paperclip_connection`。

## 凭据存储与撤销

凭据保存在 `$PAPERCLIP_HOME/mcp` 下，默认路径为 `~/.paperclip/mcp`。CLI 使用权限受限的目录和文件，以原子方式替换凭据，并在多个进程间串行刷新凭据。访问权限绑定到精确的资源 URL；重定向会被拒绝。

登录替换同一实例的现有凭据时，CLI 会尝试撤销旧连接。如果撤销失败，会提示你在 Paperclip 中手动撤销。你也可以随时从 **Connectors → Assistant Connection (MCP)** 或 `/assistant-connections` 撤销连接。

请勿在聊天或项目配置中公开 token 和私有设备码。显示的验证码仅供人类在授权时进行比对。

## 故障排查

| 问题 | 后续步骤 |
|---|---|
| URL 被拒绝 | 从 Paperclip 复制规范的 `/mcp/paperclip` URL，并检查 HTTPS 或回环地址要求。 |
| 设备授权过期 | 重新开始登录，并在命令仍运行时完成批准。 |
| 代理提示要先连接 | 使用同一操作系统账户，对该准确 URL 运行设备登录。 |
| 访问已撤销或过期 | 重新登录。系统会再次检查成员关系和授权。 |
| 写入报告连接失败 | 重试前检查任务。更改可能已经完成；代理不会自动重试。 |

## 另请参阅

- [连接助手](../../experimental/assistant-connections.md)
- [Assistant MCP 参考](../api/assistant-mcp.md)
- [身份验证](authentication.md)：其他 CLI 身份验证方式。
