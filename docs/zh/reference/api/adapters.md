---
paperclip_version: v2026.824.0
seo_title: 适配器 API
seo_description: 服务端适配器注册表的控制平面接口：Paperclip 可连接的智能体运行时、配置方式和软件包安装流程。
---

# 适配器 API

适配器 API 是**服务端适配器注册表**的控制平面接口：Paperclip 可以连接哪些智能体运行时、如何配置这些运行时，以及如何在实例级别安装外部适配器软件包。

有关各适配器配置语义的说明，请参见[参考 → 适配器 → 概览](../adapters/overview.md)。本页介绍 REST 端点。

> 本页所有路由都挂载在 `/api` 下，作用范围是实例管理员/看板用户；它们配置的是服务器本身，而非某个公司。

---

## 列出已安装的适配器

```
GET /api/adapters
```

返回 Paperclip 当前识别的服务端适配器注册表，其中包括内置适配器类型和安装在此实例上的外部适配器。

## 安装外部适配器

```
POST /api/adapters/install
```

在实例上安装外部适配器软件包。适配器管理器 UI 和 `paperclipai adapter install` CLI 工作流都会使用此端点。

## 更新适配器的实例配置

```
PATCH /api/adapters/:type
```

保存由 `:type` 指定的适配器的实例级配置，例如 `claude_local`、`codex_local`、`openclaw_gateway`。

## 覆盖适配器行为

```
PATCH /api/adapters/:type/override
```

设置由运维人员控制的适配器覆盖配置块，可用于强制禁用适配器、固定其行为或覆盖注册表默认值，而不必直接编辑实例配置。

## 删除已安装的适配器

```
DELETE /api/adapters/:type
```

从注册表中移除外部适配器。不能删除内置适配器类型。

## 重新加载适配器

```
POST /api/adapters/:type/reload
```

在当前进程中卸载并重新加载适配器模块。修改磁盘上的配置后，无需重启服务器即可使用此端点。

## 重新安装适配器

```
POST /api/adapters/:type/reinstall
```

重新执行外部适配器安装流程（重新获取、解压并注册）。

## 读取适配器配置 schema

```
GET /api/adapters/:type/config-schema
```

返回 JSON schema，适配器管理器会使用它来呈现适配器设置表单。

## 适配器 UI 解析器资源包

```
GET /api/adapters/:type/ui-parser.js
```

提供适配器附带的 UI 解析器资源包。Paperclip UI 会获取此脚本，以呈现该适配器的运行输出。线路契约详见 [Adapter UI Parser](../adapters/adapter-ui-parser.md)。

---

## 适配器设备登录

某些适配器通过**设备登录**进行身份验证：工具会输出代码和 URL，你在浏览器中批准后，CLI 会完成登录。以下路由会在 Paperclip 中为某个公司驱动此流程，并限定在一个环境内。目前这些路由适用于 Codex 适配器。

| 路由 | 用途 |
|---|---|
| `POST /api/companies/{companyId}/adapters/{type}/login-sessions` | 启动设备登录会话。正文：`{ "environmentId": "<id>", "ttlSeconds"?: <number> }`。返回 `201`；如果已有会话正在进行，则返回 `409`。 |
| `GET /api/companies/{companyId}/adapters/{type}/login-sessions/{sessionId}` | 读取会话状态，以及在浏览器中完成登录所需的代码和 URL。 |
| `POST /api/companies/{companyId}/adapters/{type}/login-sessions/{sessionId}/cancel` | 取消会话。 |

这些是看板路由：调用方必须是看板用户、有权访问该公司，并获准管理该公司的适配器。`type` 必须是支持设备登录的适配器（目前为 Codex）；其他类型会被拒绝。
