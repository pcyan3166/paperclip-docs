---
paperclip_version: v2026.1023.0
seo_title: 自定义 AI 端点与 Bedrock
seo_description: 连接 Responses、Messages、Chat Completions、本地或 Bedrock 端点。使其 API 格式与智能体框架匹配，并显式选择模型。
---

# 自定义模型提供商

使用现有的模型网关、智能体环境中的模型服务器或 Amazon Bedrock 来提供模型访问。您将路由和凭据保存为 AI 连接，然后在兼容的智能体上选择该连接。

这些连接无需启用实验性设置即可使用。它们提供模型访问，但不添加工具操作或 **权限** 操作目录。

## 选择 API 格式

格式必须与运行您智能体的框架相匹配。从其他提供商选择模型不会更改框架。

| 连接或 API 格式 | 兼容的框架 |
| --- | --- |
| **Responses API** / **OpenAI Responses · Codex** | Codex |
| **Messages API** / **Anthropic Messages · Claude** | Claude |
| **Chat Completions API** / **Chat Completions · OpenCode, Hermes** | OpenCode 或 Hermes |
| **本地端点** | 与其所选 Responses、Messages 或 Chat Completions 格式匹配的框架 |
| **Amazon Bedrock** | Claude |
| **OpenRouter** | 使用路由 OpenRouter 连接的 Claude、Codex、OpenCode 或 Hermes；参见 [OpenRouter](openrouter.md)。 |

Paperclip Runner 对其 Codex、OpenCode 和 ACP Claude 提供商使用相同的兼容性规则。其 Cursor 和 Grok 配置文件不会因选择自定义端点而变得兼容。

## 连接网关或本地端点

1. 打开 **连接器** 并选择 **Responses API**、**Messages API**、**Chat Completions API** 或 **本地端点**。
2. 输入 **提供商 URL**，使用该提供商预期的基础 URL。
3. 审查 **API 格式** 和 **身份验证**。
4. 当需要身份验证时，输入凭据。
5. 可选地，使用提供商模型 ID 或网关别名添加 **模型 ID（逗号分隔）**。
6. 选择连接的人类受众和智能体访问权限，然后选择 **连接**。

通用的模型提供商设置还将 **自定义网关** 和 **本地端点** 分组在 **高级提供商** 下。

| 身份验证 | 可用性 |
| --- | --- |
| **Bearer token** | Responses、Messages 和 Chat Completions |
| **API key · x-api-key** | 仅限 Messages |
| **无身份验证** | 不需要密钥的网关和本地路由 |

远程网关 URL 需要 HTTPS。它们不能包含嵌入的凭据、查询参数或片段。本地端点需要 localhost 或回环地址，并且可以使用 HTTP，例如 `http://localhost:11434/v1`。

本地地址是相对于智能体的执行环境的。如果您的智能体在沙箱中运行，`localhost` 指的是该沙箱，而不是您的笔记本电脑或 Paperclip 服务器。在运行智能体之前，请确保模型服务在该处可用。

## 连接 Amazon Bedrock

选择 **Amazon Bedrock**，输入 **AWS 区域**，选择 **Bedrock API key** 身份验证，并提供密钥。您可以添加想要使用的 Bedrock 模型 ID。

此连接支持 Bedrock API 密钥和区域。它不接受自定义基础 URL 或 AWS 访问密钥对。Paperclip 通过 Bedrock 路由将连接投射到 Claude 框架中。

## 选择连接和模型

打开智能体的 **连接** 选择器并选择命名的兼容账户。自定义路由连接需要显式选择；您不能将其设为责任用户的提供商默认值。

当您创建新智能体时，**连接模型** 步骤并排显示三个磁贴：提供商的订阅、提供商的 API 密钥和 **高级**（标记为 **自定义网关**）。选择 **高级** 以选择已保存的兼容连接，或者在离开智能体设置之前设置新的 OpenRouter、Amazon Bedrock、**自定义网关** 或 **本地端点** 连接。然后选择 **使用连接**。

个人账户可以被显式选择，或者连接管理器可以与公司共享账户。每次运行仍然需要一个可以使用该凭据的责任用户，并且连接必须对智能体可用。参见 [连接器访问如何工作](access-model.md)。

使用端点提供的精确模型 ID。保存在连接上的 ID 成为模型选择器的选项；您也可以在智能体上输入 ID。Paperclip 不会从任意网关、本地服务器或 Bedrock 自动获取模型目录。没有保存模型 ID 的 OpenRouter 连接使用 OpenRouter 的公共目录并提供刷新功能。

OpenCode 对自定义端点使用 `paperclip/<model-id>`，对 OpenRouter 使用 `openrouter/<model-id>`。选择器添加框架前缀；Claude、Codex 和 Hermes 使用原始上游 ID。

## 在执行环境中验证

保存自定义连接并不能证明其端点、凭据或所选模型有效。给配置好的智能体一个简短的任务并观察其运行：

```txt
Reply with the single word: ready
```

这可能消耗提供商的使用量。首先检查智能体上选择的账户和模型。在浏览器中可用的本地服务可能仍然无法从智能体的环境中访问。

> **注意：** 这是一个示例测试，而非记录的实时结果。

## 重新连接和故障排查

重新连接会替换凭据，同时保留端点、API 格式、模型列表和访问权限。若要更改路由本身，请创建另一个连接并在智能体上选择它。

| 问题 | 下一步 |
| --- | --- |
| 连接不兼容 | 使 API 格式与上表中的框架匹配。 |
| 模型列表为空 | 向连接添加提供商 ID 或在智能体上输入模型。 |
| 无法访问本地端点 | 在执行环境中启动服务并检查其回环 URL。 |
| 网关拒绝密钥 | 检查其选定的身份验证格式并使用有效凭据重新连接。 |
| 任务需要身份验证 | 如果您拥有该账户，请使用任务连接卡片重新连接；否则，其所有者必须恢复访问权限。 |

## 相关指南

- [OpenRouter](openrouter.md)
- [检查 AI 账户使用情况](ai-usage.md)
- [验证和故障排查](verify-and-troubleshoot.md)
- [Paperclip Runner](../../reference/adapters/paperclip-runner.md)
