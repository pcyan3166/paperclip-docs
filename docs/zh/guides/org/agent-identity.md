---
paperclip_version: v2026.1023.0
seo_title: 智能体的密码学身份
seo_description: 查找智能体的公钥，了解哪些运行环境会获得私钥，并在备份实例时保留原有身份。
---

# 智能体的密码学身份

每位智能体都有一对持久的 Ed25519 密钥。你可以用公钥识别该智能体在不同运行中生成的签名。无论重命名、更改模型、暂停还是重启实例，智能体身份都会保留。

此功能不需要启用实验性开关，也不会取代智能体调用 Paperclip API 时使用的 bearer 凭据。

## 查找智能体的公钥

打开智能体页面并找到 **Identity（身份）** 面板。面板会显示经过缩短的指纹，并允许你复制 PEM 格式的完整公钥。指纹用于识别密钥；密码学验证程序需要使用 PEM 公钥。

新建智能体时会同时获得身份。在此功能推出前创建的智能体，会在 Paperclip 准备该智能体下一次受管理的运行时获得身份。在此之前，面板会显示 **Not created yet（尚未创建）**。仅打开页面、重启实例或读取 API 都不会创建身份。

也可以通过 [`GET /api/agents/:id/identity`](../../../reference/api/agents.md#public-identity) 读取智能体的公开身份。尚未配置时该接口返回 `null`；配置后会返回算法、密钥 ID、公钥 PEM 和创建时间。该接口使用与读取对应智能体相同的访问检查。

## 运行环境会收到什么

Paperclip 会向受支持的受管理运行环境提供三个环境变量：

| 变量 | 值 |
|---|---|
| `PAPERCLIP_AGENT_KEY_ID` | 公钥指纹，以 `sha256:` 开头。 |
| `PAPERCLIP_AGENT_PUBLIC_KEY` | 多行公钥 PEM。 |
| `PAPERCLIP_AGENT_PRIVATE_KEY` | 用于签名的多行私钥 PEM。 |

这些值由 Paperclip 管理；智能体的环境配置无法替换它们。智能体可通过常规密码学库生成签名。例如，在 Node.js 中：

```js
import { sign } from "node:crypto";

const signature = sign(
  null,
  challengeBytes,
  process.env.PAPERCLIP_AGENT_PRIVATE_KEY,
);
```

使用方必须定义挑战内容、签名授权的操作，以及授权如何过期或撤销。Paperclip 不提供签名端点、私钥下载、Git 签名集成、密钥轮换界面或导入外部密钥功能。

> **安全提示：** 受管理的运行环境会拿到私钥，其中也包括 Cursor Cloud 等受管理的远程运行环境。请选择一个你信任、能够保管智能体持久身份的运行主机。该主机和智能体都可能保留此密钥，并在 Paperclip 之外使用它签名。

在当前版本中，由用户自行托管的 HTTP 和 gateway 智能体（包括由 API 托管的 Claude Managed 和 AWS AgentCore）不会收到私钥。新建智能体记录仍会保存身份；已有智能体记录则会在下次受支持的受管理运行时配置身份。

## 恢复时保留身份

私钥使用现有的 `local_encrypted` 机密信息提供程序加密后存入实例数据库。恢复身份需要数据库**以及与之匹配的加密密钥**：可以是 `PAPERCLIP_SECRETS_MASTER_KEY`，也可以是实例的 secrets 主密钥文件。请将该密钥与数据库备份分开保存。

如果加密密钥错误、密文损坏或公钥与私钥不匹配，运行会停止。Paperclip 不会悄悄替换身份。请先恢复匹配的数据库和加密密钥，再重试。

复制智能体或导入公司/模板时会创建新身份。开发种子数据不会包含身份记录，以便复制出来的智能体获取新的身份。灾难恢复时恢复数据库会保留原身份，因此应将这种方式用于恢复同一个实例，而不是创建独立的开发智能体。

有关便携式导出和实例恢复之间的区别，请参阅[备份和恢复公司](../../../how-to/back-up-and-restore-a-company.md)；运行时环境变量见[环境变量](../../../reference/deploy/environment-variables.md#agent-runtime)。

## 来源

- [身份服务](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/server/src/services/agent-identity.ts)——密钥创建、加密、验证和公开身份。
- [身份约定](https://github.com/paperclipai/paperclip/blob/a6306ba606eb87c89b9ef0344e9fe8e0025580f9/doc/AGENT-IDENTITY.md)——运行时交付和恢复边界。
