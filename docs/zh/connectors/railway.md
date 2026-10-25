---
seo_title: Railway 连接器
seo_description: 通过工作区同意连接 Railway，审查部署权限，并了解待定的实时资格验证以及独立的容器 SSH 访问。
---

# Railway

Railway 将智能体连接到选定的工作区以执行基础设施工作。Paperclip 的定义包括：当 Railway 接受用于 API 访问的连接时，执行直接服务、部署和受限的日志操作。

> **未经验证的资格：** 在固定的产品定义中，Railway 的实时资格验证仍在进行中。仅通过浏览器登录并不能证明直接 API 操作有效。

## 连接前准备

- 一个具有目标工作区访问权限的 Railway 账户。
- 关于哪些智能体可以访问服务数据并执行部署操作的决策。

此托管连接不支持项目令牌。容器命令需要在连接上单独设置 SSH。

## 连接 Railway

> **未经验证的设置：** 此流程遵循固定的 Paperclip 定义和 Railway 的文档。尚未使用实时 Railway 连接进行测试。以下每个步骤均未经验证。

1. **未经验证：** 打开 **Connectors**，选择 **Railway**，然后选择 **Connect Railway**。
2. **未经验证：** 在 **Access** 中，选择身份以及可以使用该身份的 智能体。
3. **未经验证：** 通过托管的 `https://mcp.railway.com` 连接进行登录，并在同意时选择 智能体 可以使用的工作区。
4. **未经验证：** 完成授权并检查操作列表。在依赖直接服务或部署操作之前，先通过读取操作确认 API 访问权限。
5. **未经验证：** 在允许操作之前，审查操作权限以及所需的工作区、项目、环境和 服务 选择。

## 选择访问权限

Railway 强制执行在同意时选择的工作区。Paperclip 还要求此 连接器 进行资源选择。请将选定的范围保持在允许工作的最小范围内。

选定的操作初始状态为 **Allowed**。在适当情况下，将部署更改和容器操作设置为 **Ask first** 或 **Off**。日志和容器命令可能会暴露应用程序数据和密钥。

通用的 Railway 智能体 和提交暂存更改在 Paperclip 中不可用，因为其内部更改无法单独审查。不要假设 Railway 自身服务器文档中描述的每个操作在此处都可用。

## 尝试使用

```txt
List the Railway projects this connection can reach. Do not deploy, restart, roll back, or run container commands.
```

与已同意的工作区进行比较，并检查 连接器 调用。成功的项目读取并不能证明部署或 SSH 访问权限。

> **未经验证的检查：** 示例任务，而非记录的测试结果。

## 故障排除与限制

| 问题 | 检查项 |
| --- | --- |
| 缺少工作区 | 审查在 Railway 同意时选择的工作区。 |
| Railway 拒绝 API 访问 | 使用所需的权限重新连接。Paperclip 不会回退到另一个 凭据。 |
| 操作需要目标 | 选择所需的工作区、项目、环境和 服务。 |
| 容器命令不可用 | 检查单独的 SSH 设置及其权限。 |
| 缺少通用 Railway 智能体 | 它在 Paperclip 中是故意不可用的。 |

实时兼容性仍未经验证。Paperclip 的可用操作和边界与直接将 Railway 连接到编辑器的情况不同。

## 相关指南

- [连接器概述](https://paperclip.ing/product/connectors/railway/)

- [连接器访问工作原理](access-model.md)
- [设置操作权限](action-permissions.md)
- [验证与故障排除](verify-and-troubleshoot.md)
- [Railway 服务器文档](https://docs.railway.com/ai/mcp-server)

## 来源

- [Paperclip 连接器定义](https://github.com/paperclipai/paperclip/blob/3166e93a7eee315e3bfbda622e080044ec5c343d/packages/shared/src/app-definitions/railway.json#L17) — 在固定的产品版本中，包括方法名称、身份验证、端点以及 连接器 特定的限制。提供商设置文档链接见上文。
