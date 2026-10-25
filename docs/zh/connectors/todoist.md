---
paperclip_version: v2026.1023.0
seo_title: Todoist 连接器
seo_description: 允许智能体读取和更新 Todoist 任务。账户与项目范围、共享项目行为、限制写入操作、读取测试及故障排查。
---

# Todoist

智能体可以处理您的 Todoist 任务——查找任务、读取详情，以及创建或完成任务。

## 连接前准备

- 一个包含您希望智能体使用的项目的 Todoist 账户。

仅此而已。这是最简单的连接器之一，设置起来非常简便。

## 连接 Todoist

1. 打开 **Connectors** 并选择 **Todoist**。
2. 阅读主按钮上方的访问权限说明行。该行说明了连接将以谁的身份登录，以及哪些智能体可以使用该连接。若要选择其他身份、缩小智能体范围或使用其他登录方法，请选择 **Change**。
3. 选择 **Continue to Todoist** 并完成浏览器登录。

Paperclip 会自动注册其客户端，因此无需在开发者控制台中进行任何配置。

## 选择访问权限

访问范围（Reach）是指授权 Todoist 账户的：其自身的项目，以及其已加入的任何共享项目。Paperclip 中没有项目选择器。

共享项目值得特别留意。个人 Todoist 账户通常属于与伴侣、团队或客户共享的项目。拥有此连接的智能体可以读取和更改所有这些项目中的任务，而这些更改在其他成员看来将显示为来自您本人。

如果这很重要，请连接一个属于较少共享项目的账户，或者保持写入操作受限。

若要限制写入操作，请在连接的 **Permissions** 选项卡上设置创建、更新和完成操作，同时保持读取操作为 **Allowed**。这两个设置并不等效，且在此处差异至关重要：

- **Off** 表示只读。智能体完全无法更改您的任务。
- **Ask first** 并非只读。智能体仍然可以更改您的任务；它只是必须等待人工批准每次调用。如果您批准，编辑将会执行。

如果您希望智能体仅查阅您的任务列表而从不编辑，请选择 **Off**。如果您希望智能体提出您签署批准的编辑建议，请选择 **Ask first**。参见 [Set action permissions](action-permissions.md)。

> **注意：** 在 Todoist 中完成任务属于写入操作，对于重复任务，它会推进重复周期。请将完成操作视为一种更改，而非无害的确认。

## 试用

```txt
What Todoist tasks are due today? List them with their projects. Do not complete or change anything.
```

与 Todoist 进行对比。读取今日任务可在不触碰您任务列表的情况下，确认凭据和智能体的权限。

> **注意：** 此为示例任务，并非记录在案的测试结果。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 某项目的任务缺失 | 授权账户未加入该共享项目 | 在 Todoist 中加入该项目；无需重新连接 |
| 智能体更改了共享项目中的任务 | 访问范围包含共享项目，且允许写入 | 将写入设置为 **Ask first**，或连接权限更有限的账户 |
| 重复任务意外移动 | 完成重复任务会推进其重复周期 | 在 Todoist 中调整该任务，并限制完成操作 |
| 写入操作被拒绝 | 账户在共享项目中的角色不允许此操作 | 检查 Todoist 中该项目的共享设置 |
| 某些字段不可用 | 某些 Todoist 功能受套餐限制 | 检查您的 Todoist 套餐 |
| **Needs attention** | 授权已被撤销 | 选择 **Reconnect** |

限制：每个连接仅限一个 Todoist 账户。Paperclip 内部无项目过滤器。共享项目随账户一同存在。

## 相关指南

- [Connector overview](https://paperclip.ing/product/connectors/todoist/)

- [Linear](linear.md)、[Jira](jira.md)、[Asana](asana.md) —— 用于团队工作跟踪，而非个人任务。
- [Set action permissions](action-permissions.md)
- [How connector access works](access-model.md)
- [Todoist developer documentation](https://developer.todoist.com/)
