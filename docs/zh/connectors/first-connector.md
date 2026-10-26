---
paperclip_version: v2026.1012.0
seo_title: 连接您的第一个连接器
seo_description: 将仅一个只读服务连接到 Paperclip，并分配一个使用该服务的任务给智能体，同时不授予任何写入权限。
---

# 连接您的第一个连接器

完成本指南后，您将拥有一个已配置的连接器，一个被允许使用该连接器的智能体，所有写入操作均已关闭，以及一个智能体实际调用了该服务的已完成任务。大部分耗时在于提供方的登录界面和智能体自身的运行过程。

本示例采用 **Hugging Face**，因为它提供了简单的设置：免费账户、无需注册 OAuth 应用的浏览器登录，且无需管理员批准。其登录请求同时申请读取和写入权限，这正是步骤 4 要求您在智能体接触该服务之前关闭写入权限的原因。

> **注意：** 替换为其他连接器是可以的，但不要期望这些确切步骤能直接套用。这种形态——带有浏览器登录的应用工具——是目录中最简单的一种。其他连接器可能需要您自行注册的 OAuth 应用、管理员权限，或 Google Developer Preview 访问权限；消息通道和模型提供方的工作方式又有所不同，且根本没有可配置的操作列表。[连接器访问机制](access-model.md) 对这些形态进行了分类，每个提供方的页面也列出了其自身的前置条件。

## 开始之前

- 运行中的 Paperclip，以及一个您可以管理的公司。参见 [安装](../guides/getting-started/installation.md)。
- 至少一个已雇佣的智能体。参见 [雇佣您的第一个智能体](../guides/getting-started/your-first-agent.md)。
- **仅当您遵循 Hugging Face 示例时：** 一个免费的 [Hugging Face](https://huggingface.co) 账户，并在同一浏览器中登录。如果您选择其他首个连接器，请遵循该连接器的前置条件；您不需要 Hugging Face 账户。

## 1. 打开目录

在左侧边栏中，选择 **Connectors**（连接器）。

目录默认显示 **Paperclip**。使用 **Search connectors**（搜索连接器）找到 **Hugging Face**，然后选择 **Connect**（连接）。**Installed**（已安装）显示已保存的账户；**All**（全部）还包括通过外部提供方提供的应用。原生账户和导入账户按所属应用分组。

## 2. 检查归属对象

设置是一个单屏界面。Paperclip 会为连接归属对象和哪些智能体可以使用该连接填充合理的默认值，并在主按钮上方的一行中说明这些设置——例如，*“为组织中的所有人连接，所有智能体可用。”* 您可以现在或稍后在连接器的 **Permissions**（权限）选项卡中更改任何设置。

对于首个连接器，请缩小范围。选择 **Change**（更改）：

- 在 **Which humans can use this credential?**（哪些人员可以使用此凭据？）下，选择 **Just me**（仅我）。这将使其成为您的个人凭据：智能体仅在您是负责人的运行中使用它。这是首个连接器的正确默认设置，因为您在此处的操作不会影响公司中的其他人。[为人员和智能体使用独立账户](separate-accounts.md) 解释了何时应选择更广泛的范围。
- 在 **Which agents can use this connection?**（哪些智能体可以使用此连接？）下，选择 **Just agents I pick**（仅我选择的智能体）并选择您计划用于测试的那一个智能体。

按钮上方的行会相应更新，例如 *“以您的身份连接，1 个选定智能体可用。”*

## 3. 登录

选择 **Continue to Hugging Face**（继续前往 Hugging Face）。Paperclip 会向 Hugging Face 的授权服务器注册自身，并打开提供方的同意界面。请在那里批准。

您无需创建 OAuth 应用，也无需粘贴令牌。如果提供方拒绝这种自动注册，Paperclip 会停止并要求您提供客户端凭据，而不是静默失败——该路径在 [连接自定义 MCP 服务器](custom-mcp-servers.md) 中说明。

## 4. 关闭所有非读取操作

打开连接器的 **Permissions**（权限）选项卡。您将看到 Hugging Face 发布操作列表，每一行被分类为 **read**（读取）、**write**（写入）或 **destructive**（破坏性），并且每一项都设置为 **Allowed**（允许）、**Ask first**（先询问）或 **Off**（关闭）。

活动操作初始状态为 **Allowed**（允许），包括写入操作。在首个智能体任务之前，将每个非读取操作设置为 **Off**（关闭）。最快的方法是 **Write**（写入）标题旁边的 **Set all**（全部设置）菜单：选择 **Set all: Off**（全部设置：关闭），所有写入操作将同时更改。Hugging Face 的登录包含仓库和作业访问权限，因此预计会有一些写入操作被关闭。请检查列表而非假设，因为该列表来自提供方，可能会随其更改而改变。

保持读取操作为 **Allowed**（允许）。现在您拥有一个只能查询信息的连接器。

## 5. 给智能体分配一个任务

创建一个任务并将其分配给您选择的智能体。选择一个狭窄且可验证的任务：

```txt
Using the Hugging Face connector, find the three most-downloaded
text-classification models published in the last month. For each one,
report the model id, the download count, and the license.
Do not create, modify, or delete anything.
```

当运行结束时，智能体的答案应包含您可以在 huggingface.co 上手动检查的模型 ID。这就是首个只读任务的意义所在：您可以判断连接器是否正常工作，而无需信任智能体对其自身行为的总结。

## 6. 确认是连接器在执行工作

有两个地方可以查看：

- 连接器的 **Permissions**（权限）选项卡显示操作列表以及哪些条目已启用。
- 连接器的活动记录显示智能体针对此连接发起的调用。

如果运行未产生任何结果，或连接器行显示 **Needs attention**（需要关注），请前往 [验证连接器并修复损坏的连接器](verify-and-troubleshoot.md)。

## 您现在拥有的

这是一个仅限您使用、供单个智能体使用且只有读取权限的连接。这是一个故意保持较小的影响范围，在您学习智能体如何使用新工具时，应保持这种形态。

## 下一步

- [连接器访问机制](access-model.md) — 步骤 2 和 4 中的选择实际控制的内容，以及其他连接器形态的差异。
- [设置操作权限](action-permissions.md) — 开启写入操作，并将 **Ask first**（先询问）作为中间设置。
- [连接器](../connectors.md) — 目录中的其余部分。

## 来源

- [连接设置](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/features/connections/ConnectionSetupFlow.tsx) — 单屏访问选择和设置行为。
- [远程 MCP 设置](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/features/connections/remote-mcp/RemoteMcpConnectionSetup.tsx) — 特定于提供方的连接控制。
