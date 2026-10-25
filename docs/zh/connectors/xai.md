---
paperclip_version: v2026.1023.0
seo_title: Grok 连接器
seo_description: 通过订阅或 API 密钥为智能体提供 xAI Grok 模型访问权限。涵盖哪些智能体可以使用、凭据分配、测试运行及故障排除。
---

# Grok

Grok 连接为智能体提供用于运行 xAI Grok 模型的凭据。这是模型访问权限，而非一组工具——没有需要授权的操作。

有两种身份验证方式：通过 Grok CLI 进行订阅登录，或使用 xAI API 密钥。

## 连接前准备

- 要么拥有涵盖 Grok CLI 登录的订阅，要么拥有来自 [xAI 控制台](https://console.x.ai/) 的 xAI API 密钥。
- 一个运行 Grok 的智能体：要么使用 **Grok** 适配器，要么使用 **Paperclip Runner** 并将 **ACP agent** 设置为 **Grok Build**（参见 [Grok Build On Paperclip Runner](../../reference/adapters/grok-local.md#grok-build-on-paperclip-runner)）。这是一个容易让人出错的要求：指向其他任何提供方的运行时智能体都不会获取该凭据。
- 仅限订阅登录：需要一个登录环境——要么是 Paperclip 服务器主机，要么是支持交互式登录的沙箱环境。参见 [使用订阅](#with-a-subscription)。

## 选择登录方式

| 方式 | 适用场景 | 计费 |
| --- | --- | --- |
| **订阅** | 希望智能体运行使用包含 Grok CLI 访问权限的计划 | 适用您的 xAI 计划的条款和限制 |
| **API 密钥** | 希望按量计费、独立账单或无需交互式登录 | xAI 按令牌对密钥所属账户计费 |

不要假设这两种方式等价。订阅路径用于身份验证 Grok CLI；它不是一般的 xAI API 凭据。请与 xAI 确认当前计划的权益，而不是自行推断。

## 连接 Grok

1. 打开 **Connectors** 并选择 **Grok**。
2. 选择登录方式。

设置是一个单屏界面，没有单独的访问步骤。账户保存后，您可以从已保存的连接中更改哪些人员和智能体可以使用它。

### 使用 API 密钥

粘贴密钥。Paperclip 将其存储为机密，之后不可读取。

### 使用订阅

登录在 **登录环境** 中运行，有两种类型。您的实例提供哪些类型属于配置事项：

| 登录环境 | 所需条件 | 登录方式 |
| --- | --- | --- |
| **Paperclip 服务器主机** | 该主机上安装了 `grok` CLI，具有 shell 访问权限，且存在活动的本地环境 | Paperclip 显示一条命令，需粘贴到该主机的终端中（**复制登录命令**） |
| **沙箱环境** | 提供方的沙箱支持交互式登录，由管理员配置 | 登录发生在 Paperclip 提供的环境中 |

当两者都存在时，Paperclip 会显示 **登录环境** 选择器。登录环境可能与智能体后续运行的位置不同。

**在 Paperclip 服务器主机上：**

1. 选择 **Sign in**。Paperclip 显示一条命令，该命令将 `GROK_HOME` 设置为本次尝试的目录并运行 `grok login --device-auth`。
2. 在该主机的终端中运行该命令并完成 xAI 的设备登录。
3. Paperclip 检测到凭据并完成连接。该尝试在过期前保持开放 30 分钟。

> **注意：** 登录使用其自身的 `GROK_HOME`，因此它既不读取也不干扰该机器上您个人的 `grok` 登录。

> **注意：** 30 分钟限制是针对服务器主机尝试记录的；我们尚未确定沙箱登录的等效时间。

如果服务器主机登录不可用，您将看到 *"Server-host subscription sign-in is unavailable on this hosted instance. Choose a supported sign-in environment or use an API key."* 这限制了 **在主机上** 登录，而非限制订阅身份验证本身。

**在受支持的沙箱中：**

1. 如果显示选择器，在 **登录环境** 中选择沙箱，并选择订阅身份验证。
2. 等待 Paperclip 准备登录链接，然后使用 **Sign in to Grok** 打开它。
3. 如果 Paperclip 显示设备代码，请在提供方的登录页面上输入该代码。完成授权并返回 Paperclip。
4. 等待 Paperclip 完成连接。如果尝试过期或失败，请开始新的尝试；仅打开提供方页面并不能确认凭据已保存。

## 分配凭据

打开已保存的连接，在 **Personal default** 下使用 **Make default** 设置您自己的账户。在智能体的 **Connection** 选择器中，选择 **Responsible user’s default** 以使用每位责任人的账户，或选择具名的兼容的个人或公司共享账户。保存智能体配置。

- **将其设为您的默认** 提供方，配置为使用责任人连接的智能体将使用每个人自己的账户。
- **为智能体选择特定的兼容连接**，以便符合条件的运行使用该账户。

特定绑定并非无条件。通过它的每次运行必须满足以下所有三点，否则将被拒绝：

| 要求 | 缺少该项导致运行失败的原因 |
| --- | --- |
| 所选绑定与账户匹配 | 具名的 **Personal** 账户使用显式个人绑定；**Company shared** 账户使用共享绑定。两种选择都不绕过人工访问 |
| 运行具有 **责任人**，且该责任人可以使用凭据 | 人工共享受众控制每个绑定——*"This credential is not shared with the responsible user"* |
| 连接已 **为该智能体安装**（或全公司范围） | 否则 *"This connection is not permitted for this agent"* |

因此，绑定 **不能替代责任人**。

**Personal** 保持凭据属于您；**Company shared** 使一个账户可供符合条件的智能体使用，前提是运行的责任人在其人工受众中。模型在智能体配置中选择。

### 从智能体设置中连接

您还可以在不离开智能体的情况下连接账户。在智能体的 **Connection** 字段中：

- 当您当前的个人默认需要关注时，会出现 **Reconnect account**。它会再次登录您并修复该同一账户，因此其默认设置及其智能体访问权限保持不变。
- **Connect an account…** 打开 **Connect account**，它会预先告知您新账户 *"will become your default for this provider."* 您的任务将使用它；其他人保留他们自己的默认设置。

当您以此方式连接全新账户时，您还会看到 **Allow all agents in this company to use this account for my tasks**。如果您可以管理连接，它默认勾选。取消勾选可将该账户仅保留给此智能体。无论哪种方式，该账户仅支持您负责任的任务，且重新连接永远不会扩大账户已有的访问权限。如果登录成功但无法保存默认设置，**Retry default selection** 会在无需再次登录的情况下重试。

### 从任务中重新连接

如果任务显示身份验证卡片，请选择 **Fix connection** 以在不离开任务的情况下修复所选账户。账户所有者再次登录；卡片会说明何时需要其他所有者执行此操作。设置完成后，任务通过正常准入恢复。重新连接会保留账户的受众和智能体访问权限。

如果您是责任人且尚未连接自己的 Grok 账户，任务聊天将显示 **AI connection needed** 而非失败，并带有标题为 **Connect your Grok account** 的卡片。选择 **Connect Grok**，在卡片内连接您自己的账户，任务将自动恢复。Paperclip 永远不会借用队友的凭据来填补空缺。

## 检查使用限制

在将长任务交给智能体之前，您可以查看提供方配额剩余多少。打开已保存的账户，找到 **Usage**，然后选择 **Check usage**。这对 Grok 订阅有效。API 密钥账户会显示 *"Unavailable for this sign-in method."* 而非按钮，因为提供方没有可供读取的单密钥配额。

Paperclip 仅在您要求时读取限制。打开账户、列出连接和启动运行都不会触发检查。每个限制窗口显示已使用量及重置时间，并在提供方指示时标记为 **Limit reached** 或 **Blocked**。**Overage** 显示是否可用额外付费使用。提供方未提供的任何内容显示为 **Not reported**，绝不显示为零。成功检查后，按钮变为 **Refresh**。

Grok 并不总是报告已使用包含计划中的多少。当它省略此项时，Paperclip 显示 **Not reported** 而非猜测。

检查是只读的。它不会将工作转移到其他账户，不会在达到限制时停止运行，不会刷新凭据，也不会购买额度。

| 消息 | 含义 |
| --- | --- |
| *"Sign in again to check usage."* | 存储的凭据已过期。重新连接账户 |
| *"Usage access denied."* | 提供方拒绝共享此凭据的使用情况 |
| *"Too many checks. Try again later."* | 提供方对检查本身进行了速率限制。您的配额不受影响 |
| *"Reconnect to check usage."* | 账户当前不可用。请先重新连接 |

Board 用户可以通过 `GET /api/companies/{companyId}/ai-connections/{connectionId}/usage` 通过 API 运行相同的检查。

## 试用

检查智能体的适配器是否为 Grok，并记录其配置使用的 AI 连接。然后：

```txt
Reply with the single word: ready
```

**回复证明** Grok 凭据已被接受，适配器匹配，且上述检查已通过——Paperclip 拒绝不合格的绑定，而不是回退。**它不证明哪个账户被计费**；该归属信息未在每次运行中显示，因此请在 xAI 控制台中检查您预期账户的使用情况。

如果运行报告连接不兼容，请先检查适配器。

> **注意：** 示例任务，非记录的测试结果。Grok 运行未为此文档执行。

## 故障排除与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| *"Select an AI connection compatible with this harness and model"* | 智能体未设置为运行 Grok——最常见的是运行时智能体的 ACP agent 不是 Grok Build | 将智能体的适配器设置为 Grok，或将运行时的 **ACP agent** 设置为 **Grok Build** |
| *"Connect an account and choose your personal default"* | 智能体使用责任人的连接，但该人没有默认设置 | 连接账户并将其标记为默认 |
| *"This run needs a responsible user to select an AI connection"* | 运行没有责任人来评估凭据检查 | 为工作指定合格的责任人。绑定共享连接 **不能** 绕过此限制 |
| *"This credential is not shared with the responsible user"* | 连接与具名人员共享，但责任人不在其中 | 将该人员添加到连接的受众中 |
| 预览期间创建的订阅连接停止工作 | 预览时期的订阅凭据不可重用，必须重新建立 | 重新连接账户 |
| **Sign in** 不可用 | 此部署未提供登录环境 | 询问管理员是否可以启用沙箱登录环境；否则使用 API 密钥 |
| 登录命令无反应 | 您运行命令的主机上缺少 `grok` CLI，或您在错误的机器上运行了它 | 安装 CLI 并在 Paperclip 服务器主机本身的终端中运行命令 |
| 状态为 **expired** 或 **needs attention** | 凭据已轮换或密钥已撤销 | 重新连接账户 |
| 运行因配额错误失败 | xAI 的计划或密钥限制，而非 Paperclip 限制 | 对于订阅，在已保存的账户上选择 **Check usage** 以查看哪个窗口已耗尽及何时重置。否则通过 xAI 检查使用情况 |

限制：一个连接对应一个 xAI 账户，且不授予工具访问权限。Grok 适配器要求比其他模型提供方更窄——在围绕此连接器规划工作之前请确认这一点。

## 相关指南

- [连接器概述](https://paperclip.ing/product/connectors/xai/)

- [Anthropic](anthropic.md)、[OpenAI](openai.md)、[OpenRouter](openrouter.md) — 其他模型提供方。
- [自定义模型提供方](custom-model-providers.md) — 兼容网关、本地端点和 Bedrock。
- [检查 AI 账户使用情况](ai-usage.md)
- [连接器访问机制](access-model.md)
- [xAI 文档](https://docs.x.ai/)
