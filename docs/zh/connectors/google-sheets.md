---
paperclip_version: v2026.1005.0
seo_title: Google Sheets 连接器
seo_description: 连接 Google Sheets 的两种方式：通过 Google 登录，或将指定名称的电子表格共享给 Paperclip 机器人账户。包括分组、读取测试及修复方法。
---

# Google Sheets

> **警告：** **Google 验证待定。** Paperclip 尚未完成 Google 应用验证。在授权过程中，您可能会看到“未验证应用”的警告。如果 Google 提供 **Advanced**（高级）选项以继续前往 Paperclip，在审查所请求的访问权限后，您可以选择继续。并非所有账户都提供此选项；Workspace 管理员限制和其他 Google 访问要求仍然适用。如果您无法连接，请联系 [support@paperclip.ing](mailto:support@paperclip.ing)。

> **注意：** 在验证待定期间，Google Workspace 连接器以及任何已保存的 Google 账户会暂时从 **Connectors**（连接器）页面隐藏。这仅将其从列表中隐藏。现有的 Google 连接将继续运行，其访问权限和权限保持不变，需要 Google 服务的智能体仍可在其任务中通过连接卡片向您请求该服务。

智能体可以读取电子表格的值和结构，在写入连接中还可以更新它们。

Sheets 是唯一一个具有两种真正不同设置方式的连接器：使用 Google 登录，或将指定名称的电子表格共享给 Paperclip 机器人账户。它们在智能体可以访问的内容和可以执行的操作上有所不同，因此请在开始之前进行选择。

## 您想要哪种设置？

| 如果您希望 | 使用 | 访问范围 |
| --- | --- | --- |
| 智能体能够跨您 Google 账户已能打开的电子表格进行工作 | **Google 登录** | 该账户能打开的所有内容 |
| 将智能体限制在简短、明确的电子表格列表中 | **Paperclip 机器人账户** | 仅您粘贴的电子表格 |

机器人账户是更强的边界：Paperclip 会将每次调用与连接的电子表格列表进行核对，因此访问范围不依赖于提供方的同意屏幕。它还暴露了更广泛的操作集，包括行删除。Google 登录路径可以访问更多的电子表格，但无法删除任何内容。

> **注意：** 机器人路径的电子表格允许列表是特定于该方法的。不要假设其他连接器提供相同的资源过滤器。

## 选项 A：Google 登录

> **警告：** 此路径在授权前需要 Google Workspace 开发者预览版注册。Google 必须注册用于登录的 Workspace 电子邮件，如果您使用自己的客户端，还必须注册拥有 OAuth 客户端的 Cloud 项目。请先在 [Google Workspace Developer Preview](https://developers.google.com/workspace/preview) 处申请。

### 连接前准备

- 一个已经能够打开您希望智能体使用的电子表格的 Google Workspace 账户，且已确认完成开发者预览版注册。
- 如果您使用自己的客户端，请完成 [Google OAuth 设置](google-setup.md)，包括 API 启用、同意设置、作用域和回调 URI。

### 能力分组

| 分组 | 智能体可以执行的操作 | 请求的作用域 |
| --- | --- | --- |
| **仅读取** | 读取电子表格的值和结构 | `spreadsheets.readonly` |
| **读取与编辑** | 上述操作，外加更新值、公式和维度 | `spreadsheets` |

两个分组都不请求 Google Drive 访问权限。仅 Sheets 作用域就足以读取和编辑该账户能打开的电子表格（通过电子表格 ID）。使用早期版本建立的连接请求了更多权限。如果在此更改之前建立的 **Connect with Paperclip** 连接停止工作，请选择 **Reconnect**（重新连接）以使用较小的权限集进行登录。使用您自己的 OAuth 应用的连接在重新连接之前会保留其早期的授权。

已审查的操作：两个分组中均包含 `get-spreadsheet` 和 `get-values`；仅编辑分组中包含 `update-spreadsheet`、`update-values`、`update-formulas` 和 `insert-dimension`。此路径上没有删除操作。

### 步骤

1. 打开 **Connectors**（连接器）并选择 **Google Sheets**。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择其他身份、缩小智能体范围或使用其他登录方法，请选择 **Change**（更改）。
3. 检查能力分组。设置默认从可以执行更改的分组开始；若要仅读取连接，请选择 **Change** 并在 **What should Paperclip be able to do?**（Paperclip 应该能够做什么？）下选择它。当您的实例提供时，Paperclip 使用 **Connect with Paperclip**。若要改用您自己的客户端，请选择 **Use your own Google OAuth app**（使用您自己的 Google OAuth 应用）并提供来自 [Set up your own Google OAuth app](google-setup.md) 的客户端 ID 和密钥。
4. 使用已注册的 Workspace 账户完成 Google 的同意屏幕。

> **注意：** 连接器的指南建议电子表格更新应经过批准。请将写入操作保持在 **Ask first**（先询问）状态。

## 选项 B：Paperclip 机器人账户

代替连接 Google 身份，您将单个电子表格共享给实例拥有的机器人账户。然后，智能体只能访问这些确切的电子表格，而不能访问其他任何内容。

此路径要求实例管理员已配置服务账户。如果未配置，Paperclip 将报告 *"Google Sheets is not available on this instance yet."*（Google Sheets 在此实例上尚不可用。），且该选项无法使用。它不需要开发者预览版注册。

### 步骤

1. 打开 **Connectors**（连接器）并选择 **Google Sheets**。选择 **Change**（更改），并在 **What should Paperclip be able to do?**（Paperclip 应该能够做什么？）下选择 **Share selected sheets**（共享选定的电子表格）——即 **Use the Paperclip robot account**（使用 Paperclip 机器人账户）方法。
2. 阅读主按钮上方的访问行，它说明了哪些智能体可以使用该连接。选择 **Change**（更改）以缩小智能体范围。此方法不要求提供个人 Google 身份。
3. Paperclip 显示机器人账户的电子邮件地址。在 Google Sheets 中，将每个电子表格与该地址共享：
   - **Viewer**（查看者）足以进行读取。
   - **Editor**（编辑者）是追加、更新、添加工作表、清除值或删除行所必需的。
4. 粘贴每个已共享电子表格的链接。至少需要一个链接，Paperclip 会拒绝任何非 Google Sheets 链接的内容。
5. 完成设置。Paperclip 验证其能否访问您列出的每个电子表格。

### 智能体在此路径上可以执行的操作

| 操作 | 类别 |
| --- | --- |
| `list_spreadsheets`, `get_spreadsheet_info`, `read_values`, `search_rows` | 读取 |
| `append_rows`, `update_values`, `add_sheet_tab` | 写入 |
| `clear_values`, `delete_rows` | 破坏性 |

这些操作中的每一个都限制在连接的列表中的电子表格内。稍后添加电子表格意味着编辑连接的列表——仅将其共享给机器人账户是不够的。

> **警告：** `clear_values` 和 `delete_rows` 会移除数据，且无法从 Paperclip 恢复。除非智能体确实需要它们，否则请保持它们处于 **Off**（关闭）状态，并依赖 Google Sheets 的版本历史进行恢复。

## 选择访问权限

在 Google 登录路径中，访问范围是授权账户能打开的内容，Paperclip 不会对其进行缩小。在机器人路径中，访问范围是粘贴的列表，Paperclip 会强制执行它。

选择哪些智能体可以使用任一连接。Google 登录方法还询问谁可以使用该凭据；机器人账户方法使用实例配置的服务账户，而不是个人 Google 登录。参见 [连接器访问机制说明](access-model.md)（连接器访问权限如何工作）。

## 尝试使用

使用一个您自己能打开的电子表格，通过其 URL 标识，并选择一个您已知其值的单元格。替换两者：

```txt
In this spreadsheet — https://docs.google.com/spreadsheets/d/YOUR_SPREADSHEET_ID/edit
— read cell B2 of the "Headcount" tab and tell me its value. Do not change anything.
```

预期得到您已知的值。提供 URL 而不是标题很重要：已审查的读取操作接受电子表格标识符，要求智能体“按标题”查找文档测试的是发现功能，而不是连接本身。

在机器人路径上，首先运行 `list_spreadsheets` 作为更廉价的冒烟测试——但要狭义地阅读它所证明的内容。**它返回连接的配置列表，因此它确认的是设置，而不是 Google 是否会提供某个单元格。** 即使电子表格出现在那里，如果它从未实际共享给机器人账户，读取仍可能失败。请随后执行上述的单元格读取。

> **注意：** 这是说明性任务，而非记录的测试结果。

## 故障排除与限制

| 问题 | 可能原因 | 修复方法 |
| --- | --- | --- |
| **Use the Paperclip robot account**（使用 Paperclip 机器人账户）不可用 | 实例上未配置服务账户 | 询问管理员；消息为 *"Google Sheets is not available on this instance yet."*（Google Sheets 在此实例上尚不可用。） |
| 粘贴的链接被拒绝 | 它不是 Google Sheets 链接 | 使用电子表格本身的 URL，而不是 Drive 文件夹或发布到网络的链接 |
| 机器人路径：智能体无法看到您共享的电子表格 | 它不在连接的列表中 | 将链接添加到连接中；仅共享并不授予访问权限 |
| 机器人路径：读取有效但写入失败 | 机器人账户具有 **Viewer**（查看者）权限，而非 **Editor**（编辑者）权限 | 在 Google Sheets 中更改共享角色 |
| Google 登录：在同意前授权被拒绝 | 开发者预览版注册未完成 | 完成注册并重试 |
| Google 登录：缺少写入操作 | 连接是使用 **Read only**（仅读取）建立的 | 使用 **Read & edit**（读取与编辑）建立连接 |
| **Needs attention**（需要关注） | 凭据已过期或授权已被撤销 | 选择 **Reconnect**（重新连接） |

限制：两条路径都不创建或删除整个电子表格。Google 登录路径完全没有删除操作。开发者预览版仅适用于 Google 登录路径。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/google-sheets/)（连接器概述）

- [Google Drive](google-drive.md) — 查找和创建文件。
- [连接器访问机制说明](access-model.md)（连接器访问权限如何工作）
- [验证连接器并修复故障连接器](verify-and-troubleshoot.md)（验证连接器并修复损坏的连接器）
- [Google Sheets API MCP reference](https://developers.google.com/workspace/sheets/api/reference/mcp)（Google Sheets API MCP 参考）
