---
paperclip_version: v2026.1023.0
seo_title: Hugging Face 连接器
seo_description: 让智能体搜索 Hugging Face 模型、数据集和论文。涵盖发现功能范围、为何不运行或托管模型、搜索测试及故障排查。
---

# Hugging Face

智能体可以搜索 Hugging Face Hub——查找模型、数据集和论文，并阅读其文档和元数据。

> **注意：** 这是发现，而非执行。通过此连接器找到模型并不意味着运行它、将其下载到工作区、部署它，或让智能体对其进行推理。运行模型与阅读关于模型的文档是两回事。

如果您希望智能体*使用*某个模型，那属于模型提供商连接——请参阅 [Anthropic](anthropic.md)、[OpenAI](openai.md)、[OpenRouter](openrouter.md) 或 [Grok](xai.md)。

## 连接前须知

对于公共 Hub 内容，您无需任何配置——不需要账户、密钥或登录。选择 **Use public Hugging Face Hub** 即可完成。

对于私有或受限（gated）仓库，您需要一个已拥有这些仓库访问权限的 Hugging Face 账户。请使用属于您公司的账户进行连接；访问权限将跟随该账户。

## 连接 Hugging Face

有两种接入方式。请选择符合智能体需要查看内容的方式。

| 方法 | 您获得的功能 | 所需条件 |
| --- | --- | --- |
| **Use public Hugging Face Hub** | 对公共模型、数据集、仓库详情和文档的只读查询 | 无——这是匿名访问 |
| **Sign in with Hugging Face** | 授权账户的访问范围，包括其可打开的私有仓库，以及仓库和作业工具 | 一个 Hugging Face 账户 |

1. 打开 **Connectors** 并选择 **Hugging Face**。
2. 阅读主按钮上方的访问说明行。它说明了连接所属的身份以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他方法，请选择 **Change**。
3. 选择您的方法：
   - **Use public Hugging Face Hub** 会立即连接。无需登录任何内容。
   - **Sign in with Hugging Face** 会打开浏览器登录。Paperclip 会自动注册其客户端，因此无需在开发者控制台中进行任何配置。

公共方法是目录中最简短的连接器设置，因此适合作为首次尝试。

## 选择访问权限

内容分为两个层级，其区别值得明确说明：

| 内容 | 谁可以查看 |
| --- | --- |
| 公共模型、数据集和论文 | 任何人。连接不会为这些内容增加特权访问权限 |
| 私有或受限仓库 | 仅当授权账户已拥有访问权限时 |

**公共方法** 仅为公共 Hub 提供只读工具，因此它是风险最低的连接器之一。有两点需要注意：

- Hugging Face 对匿名请求实施速率限制。繁忙的智能体可能会触及限制；如果这成为问题，请切换到已登录的连接。
- 查询将按输入内容发送至 Hugging Face。请仅使用公共信息——不要包含机密、个人数据、私有仓库名称或专有细节。

**已登录方法** 需要谨慎处理。由于此处的大部分价值在于公共内容，一旦关闭其写入权限，它便是风险较低的连接器之一。例外情况是拥有私有组织仓库访问权限的账户——该账户的访问范围将成为智能体的访问范围。

对于已登录连接，Paperclip 向 Hugging Face 请求 `read-mcp`、`read-repos`、`contribute-repos` 和 `jobs` 作用域。这涵盖了读取，但也包括向仓库贡献内容和运行作业，因此默认情况下这不是只读连接。请阅读 **Permissions** 选项卡上的操作列表，并将任何非读取操作设置为 **Ask first** 或 **Off**，直到您需要它们为止。请参阅 [设置操作权限](action-permissions.md)。

> **注意：** 已登录连接针对 Hub 自身的工具，且 Paperclip 请求端点时已关闭 Gradio 工具——因此托管在 Spaces 上的交互式应用程序不属于此连接的界面范围。

## 试用

```txt
Search Hugging Face for recent text embedding models under 500M parameters and summarize the top three, including their licences.
```

预期结果可在 Hub 上验证。许可证（Licence）是一个值得询问的有用信息，因为人们在选择模型时往往忽略这一细节。

> **注意：** 这是示例任务，而非记录的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 未找到私有仓库 | 授权账户没有访问权限 | 在 Hub 上授予访问权限，或使用拥有该权限的账户进行连接 |
| 受限模型的详情不可用 | 受限仓库要求在 Hub 上接受条款 | 使用授权账户接受条款 |
| 智能体声称可以运行其找到的模型 | 它不能——此连接器仅用于发现 | 使用模型提供商连接 |
| 公共连接在重度使用期间开始失败 | Hugging Face 对匿名请求实施速率限制 | 等待并重试，或使用 **Sign in with Hugging Face** 进行连接 |
| 在公共连接上未找到私有仓库 | 公共方法仅能看到公共内容 | 使用拥有访问权限的账户通过 **Sign in with Hugging Face** 进行连接 |
| 结果似乎过时 | Hub 不断变化；智能体读取的是当时返回的内容 | 重新运行搜索 |
| **Needs attention** | 授权已被撤销 | 选择 **Reconnect** |

限制：仅用于发现和元数据。无推理、无托管、无 Spaces 应用程序。私有内容需要已登录连接，且其账户已拥有访问权限；公共方法是匿名、只读且受速率限制的。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/hugging-face/)

- [Anthropic](anthropic.md)、[OpenAI](openai.md)、[OpenRouter](openrouter.md)、[Grok](xai.md) — 实际运行模型的连接。
- [设置操作权限](action-permissions.md)
- [Hugging Face MCP documentation](https://huggingface.co/docs/hub/agents-mcp)
