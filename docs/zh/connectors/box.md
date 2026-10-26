---
paperclip_version: v2026.1005.0
seo_title: Box 连接器
seo_description: 让智能体搜索并读取 Box 内容。首先需要 Box 管理员进行配置，涉及企业限制、读取测试及故障排查。
---

# Box

智能体可以搜索并处理 Box 中的文件和文件夹。

> **警告：** 在任何人能够连接之前，此连接器需要 Box 管理员进行配置。管理员必须在 Box 中创建 OAuth 集成并为该企业启用 AI 访问。若缺少这两项中的任何一项，授权要么会失败，要么会成功但指向一个空账户。

## 连接前准备

- 需要一位 Box 管理员来创建 OAuth 集成并为该企业启用 AI 访问。如果这不是您本人，请先联系他们——在 Paperclip 中进行的任何工作都无法替代这一前提。
- 来自该 Box 集成的客户端 ID 和密钥，以及在该集成上注册的 Paperclip 回调 URI。Paperclip 会在设置过程中显示该 URI。
- 一个具有访问权限的 Box 账户，以便智能体能够使用您希望其访问的内容。

Box 不支持此服务器的自动客户端注册，因此必须使用您自己的 OAuth 集成，而非可选配置。

## 连接 Box

此过程的大部分操作需在 Box 的管理控制台中完成，且需要管理员权限。此处只需先在 Paperclip 中读取回调 URI。

### 1. 读取 Paperclip 的回调 URI

打开 **Connectors** → **Box**。由于 Box 需要一个您自行注册的 OAuth 应用，因此设置界面将显示 **Box needs its own OAuth app** 并展示回调 URI。复制该 URI 并提供给执行第 3 步的人员。

### 2. 为企业启用 Box AI

在 Box **Admin Console** 中：

1. 选择 **Box AI**，然后选择 **Settings**。
2. 确保 **AI API** 已启用。
3. 确保已选中 **Official Box Integrations is enabled for all users**。

> **警告：** 若未执行上述操作，授权可能会成功，但智能体仍无法执行任何有效操作。如果连接状态看似正常，但每次调用均失败或返回空结果，请在排查 Paperclip 中的任何问题之前，先返回此步骤进行检查。

### 3. 创建集成凭据

仍在管理控制台中：

1. 在左侧边栏中选择 **Integrations**。
2. 按 **MCP** 类别筛选，或搜索您所需的 Box MCP 服务器。
3. 在所选 MCP 服务器旁，选择其状态并将其启用。
4. 对于未列出的服务器，将鼠标悬停在 **Custom Box MCP Server** 上，然后选择 **Configure**。
5. 在 **Additional Configuration** 下，选择 **+ Add Integration Credentials**。
6. 复制生成的 **Client ID** 和 **Client Secret**。
7. 输入第 1 步中获取的 **Redirect URI**。
8. 在 **Scopes** 下，确保已选中 **Manage AI**。

### 4. 在 Paperclip 中完成设置

返回设置界面，提供客户端 ID 和密钥，然后以智能体应访问其内容的账户身份在 Box 中完成授权。

Box 的官方参考文档为 [Managing Box MCP Servers](https://support.box.com/hc/en-us/articles/43847256139923-Managing-Box-MCP-Servers)。

## 选择访问权限

访问范围取决于授权 Box 账户的权限：即该账户已经能够打开的文件、文件夹和共享内容。Paperclip 没有文件夹选择器，因此通过授权一个权限更少的账户来限制访问范围。

Box 企业通常会在上述基础上施加更多限制——分类标签、共享链接策略和应用访问规则均可能导致内容无法通过集成访问，即使用户在 Box 网页应用中可以看到该内容。这通常是智能体在看似正常的设置后找不到任何内容的主要原因。

文件写入和删除操作具有重大影响，且并非总能从 Paperclip 中恢复。请将其设置为 **Ask first** 或 **Off**，并依赖 Box 的回收站和版本历史进行恢复。参见 [设置操作权限](action-permissions.md)。

## 尝试使用

```txt
Search Box for a file named "supplier agreement" and tell me its folder and last modified date. Do not download, move, or change anything.
```

将结果与 Box 网页端显示的内容进行核对。元数据读取可在不将文档内容拉取到任务记录中的情况下，确认凭据及企业的 AI 访问权限。

> **注意：** 此为示例任务，并非记录在案的测试结果。请替换为您自己能够找到的文件名。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 授权失败 | Box OAuth 集成不存在，或回调 URI 不匹配 | 让管理员创建该集成并注册确切的 URI |
| 授权成功但未找到任何内容 | 未为企业启用 AI 访问，或企业策略阻止了该应用 | 联系 Box 管理员启用 AI 访问并检查应用访问规则 |
| 部分文件夹可见，部分不可见 | 授权账户不是这些文件夹的协作者，或应用了分类规则 | 将该账户添加为协作者，或检查分类策略 |
| 文件已列出但无法读取 | 该内容受企业限制，或为不支持的文件类型 | 在 Box 中检查该文件的分类 |
| 意外发生了删除或移动操作 | 写入操作被设置为 **Allowed** | 从 Box 回收站恢复，然后收紧操作设置 |
| **需要关注** | 授权已被撤销，或集成已被禁用 | 选择 **Reconnect**；检查集成是否仍然存在 |

限制：每个连接仅支持一个 Box 账户。Paperclip 内部无文件夹限制。企业策略可能会覆盖账户看似拥有的权限。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/box/)

- [Google Drive](google-drive.md) — 另一个文件存储连接器。
- [设置操作权限](action-permissions.md)
- [连接器访问机制说明](access-model.md)
- [Managing Box MCP servers](https://support.box.com/hc/en-us/articles/43847256139923-Managing-Box-MCP-Servers)
