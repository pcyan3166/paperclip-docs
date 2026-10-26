---
paperclip_version: v2026.1012.0
seo_title: 已保存的连接说明
seo_description: 为智能体提供来自已保存连接的指导，编辑或禁用提供商默认设置，并查看说明的适用时机，同时不改变工具权限。
---

# 已保存的连接说明

您可以将指导附加到已保存的连接，以便符合条件的智能体了解何时以及如何使用该连接。例如，记忆连接可以指示智能体在工作前回忆先前的决策，并在之后保存有用的事实。

说明用于指导行为。连接的访问规则和操作权限仍决定智能体可以执行的操作。

## 审查或编辑指导

五个[记忆连接器](memory-connectors.md)包含经过审查的默认设置。当连接器声明模板时，其设置界面会在现有屏幕上显示**智能体说明**；没有额外的设置步骤。

要更改已保存的连接：

1. 打开其**权限**页面，找到位于智能体访问和操作列表之间的**智能体说明**。
2. 保持**告知智能体使用 {provider}** 开启状态，以便在此连接可用于任务时包含该指导。
3. 选择**编辑说明**，编写您的指导，然后选择**保存说明**。文本最多可包含 2,000 个字符。
4. 若要再次采用提供商的当前模板，请选择**重置为默认值**并保存。重置操作会保留切换开关的状态。

关闭切换开关会保留您的文本和工具访问权限。通知会明确说明这一点：智能体仍可使用工具，但不会收到这些说明。保存失败会保留您的草稿，以便您可以重试。

重新连接、登录返回和操作刷新会保留您已保存的选择。较新的目录模板不会覆盖自定义文本，也不会将已退出的连接重新开启。

## 查看智能体接收的内容

打开智能体的说明，查找**来自连接**。这个只读部分显示已保存的指导，并带有链接回其源连接，包括针对已禁用说明的通知。

实际指导取决于任务。该连接必须对该智能体和负责人可用，具有当前的凭据授权，并且至少有一个可访问的工具。需要批准的工具仍可使指导可用。已撤销的访问权限、已禁用的工具或缺失的必要连接配置可能会阻止指导的提供。

编辑适用于后续轮次。正在运行的轮次会保留其开始时使用的指导；当已保存的指导发生变化时，Paperclip 会替换不兼容的会话。

## 编写有用的说明

保持文本具体，说明您希望如何使用该连接。指明预期的记忆上下文或工作流，描述何时使用它，并在保存或检索失败时要求提供证据。

```txt
Recall relevant project decisions before planning work.
Save concise facts that will help the next task, and cite what was saved.
Treat older memories as background context when the current task differs.
```

不要在说明文本中放入 API 密钥或密码。不要告诉智能体指导授予访问权限或覆盖批准要求。

## 在记忆之外使用说明

已保存的说明是一种通用的连接功能。自定义和非记忆连接可以通过连接 API 存储相同的 `agentInstructions` 设置，即使没有模板：

```json
{
  "agentInstructions": {
    "enabled": true,
    "text": "Read the release handbook before changing a deployment. Cite the checklist."
  }
}
```

当前编辑器仅适用于包含明确模板元数据的连接。它不是每个自定义服务器上的通用文本框。Paperclip 不会自动信任远程 MCP 服务器的初始化文本作为连接说明。

对于自定义 HTTP 或进程适配器，请将服务器提供的说明快照与智能体的常规提示一起传递，并将更改的快照视为会话更改。[HTTP 适配器](../reference/adapters/http.md#connection-instructions) 和 [进程适配器](../reference/adapters/process.md#connection-instructions) 参考文档描述了快照到达的确切方式。

## 相关指南

- [记忆连接器](memory-connectors.md)
- [连接器访问机制](access-model.md)
- [设置操作权限](action-permissions.md)
- [连接自定义 MCP 服务器](custom-mcp-servers.md)

## 来源

- [设置契约](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/shared/src/connection-instructions.ts) — 字段、文本限制、默认值和公共配置。
- [编辑器与智能体视图](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/ui/src/features/connections/ConnectionInstructions.tsx) — 标签、保存、重置和通知。
- [授权交付](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/server/src/services/connection-instructions.ts) — 任务依赖的可用性和说明快照。
