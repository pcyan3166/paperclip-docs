---
paperclip_version: v2026.1012.0
seo_title: Mem0 连接器
seo_description: 使用 Mem0 为智能体提供持久化记忆存储。明确记忆归属、命名空间工作原理、范围受限的读取测试以及故障排查。
---

# Mem0

Mem0 为智能体提供持久化记忆存储，其生命周期超越单次任务——这是一个用于记录事实并在后续运行中检索它们的地方。

在连接之前需要确定的问题是智能体正在读取和写入的是*谁的*记忆。Mem0 根据您在调用时提供的标识符来组织记忆，因此边界是您强制执行的约定，而非 Paperclip 进行检查的内容。

## 连接前准备

- 开启 **Memory connectors**（记忆连接器）实验性设置。该设置默认关闭，在开启之前，Mem0 不会出现在 **Connectors**（连接器）列表中——参见 [Connections and Apps](../experimental/connections-apps.md)。稍后关闭该设置会阻止新的精选设置；已保存的 Mem0 连接将继续运行，并可重新连接或轮换其密钥。有关完整的提供商系列，请参见 [Memory connectors](memory-connectors.md)。
- 一个 Mem0 账户及其仪表板中的 API 密钥。
- 关于如何为记忆命名空间的决定——按智能体、按用户、按项目——以及智能体应使用哪些标识符。

## 连接 Mem0

1. 打开 **Connectors**（连接器）并选择 **Mem0**。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他登录方法，请选择 **Change**（更改）。
3. 粘贴 **Mem0 API key**（Mem0 API 密钥）。Paperclip 将其存储为机密信息。

## 选择访问权限

> **警告：** API 密钥可访问整个 Mem0 项目，而非单个命名空间。拥有此连接的智能体原则上可以通过传递不同的标识符来读取和写入该项目中的任何记忆。智能体或用户之间的隔离是智能体调用工具时的约定，而非强制执行的边界。

两个实际后果：

- 如果不同的智能体不应看到彼此的记忆，请为它们提供 **使用各自的 Mem0 项目和密钥分别创建连接**。不要仅依赖标识符纪律。
- 请谨慎决定写入内容。记忆会在任务之间持久存在，并且持有密钥的任何内容稍后都可以读取，因此写入 Mem0 的个人数据或机密信息将比产生它们的对话存在更久。

活动操作初始状态为 **Allowed**（允许），包括写入和删除。

记忆写入容易进行且会累积。如果在每次运行时都写入的智能体将构建一个无人整理的存储库，这既会增加成本，也会降低检索质量。建议在开始时将写入设置为 **Ask first**（先询问），以便您可以查看智能体想要记住什么。参见 [设置操作权限](action-permissions.md)。

删除操作会移除记忆；如果智能体不应遗忘已存储的信息，请将其设置为 **Off**（关闭）。

该连接在 **Agent instructions**（智能体指令）下包含可编辑的召回和保存指南。您可以关闭它而不移除工具，或者编辑它以描述您的记忆上下文。参见 [Saved connection instructions](connection-instructions.md)。

## 尝试使用

使用一个已包含已知记忆的命名空间。将示例标识符替换为您在 Mem0 中使用的标识符：

```txt
Search Mem0 for onboarding memories under user_id "YOUR_KNOWN_USER_ID".
Return the stored fact and its memory ID. Do not add, update, or delete anything.
```

将结果与现有记忆进行比较，并检查任务的连接器调用。如果您的集成使用智能体或项目标识符，请提供该确切标识符。

空结果并不能证明已读取了预期的命名空间。在更改凭据之前，请检查标识符、项目和工具结果。如果存储库为空，请推迟内容验证，或在测试项目中明确授权一次性的写入/读取测试；不要仅为了测试连接而向生产记忆中添加种子数据。

> **注意：** 这是说明性任务，而非记录的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 搜索返回空结果 | 存储库为空，或智能体查询的标识符与其写入时使用的标识符不同 | 检查智能体正在使用的标识符约定 |
| 一个智能体读取了另一个智能体的记忆 | 标识符是约定，而非强制执行的边界 | 使用单独的 Mem0 项目和单独的连接器 |
| 检索质量随时间下降 | 存储库中积累了低价值记忆 | 在 Mem0 中整理存储库；限制智能体可以写入的内容 |
| 调用被拒绝 | 密钥已被撤销或属于其他项目 | 使用当前密钥重新连接 |
| 写入了敏感信息 | 记忆在任务结束后仍持久存在 | 在 Mem0 中删除它，并缩小智能体可以存储的内容范围 |
| **Needs attention**（需要关注） | 密钥停止工作 | 选择 **Reconnect**（重新连接） |

限制：每个连接对应一个 Mem0 项目，且密钥可访问该项目的所有内容。Paperclip 不强制执行命名空间。Mem0 自身的计划限制适用。

## 相关指南

- [设置操作权限](action-permissions.md)
- [连接器访问机制说明](access-model.md)
- [Mem0 MCP 文档](https://docs.mem0.ai/platform/mem0-mcp)

## 来源

- [Mem0 definition](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/app-definitions/mem0.json) — API 密钥设置和默认指南。
- [Feature defaults](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/feature-catalog.ts) — 默认关闭的实验性设置。
