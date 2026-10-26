---
paperclip_version: v2026.831.1
seo_title: 移除成员
seo_description: 当成员离职时，移除其访问权限。Paperclip 通过更改成员状态而非删除记录来实现此操作，从而确保其历史记录完整且可审计。
---

# 移除成员

当有人离开——例如承包商结束合作、团队成员角色变更或账户需要停用——您需要将其访问权限从公司中移除。Paperclip 通过更改其**成员状态**而非删除记录来实现此操作，因此审计轨迹得以保留。目前尚无自助式的“离开公司”功能；需由所有者（Owner）或管理员（Admin）在应用中执行移除操作。

**开始之前：** 您需要 `users:manage_permissions` 才能编辑或归档成员（所有者拥有此权限；管理员没有）。移除实例级访问权限需要实例管理员权限。

---

## 1. 选择：暂停或归档

根据该人员是否可能返回，分为两个级别：

- **暂停（Suspend）** — 可逆的暂停。成员记录保留，状态切换为 `suspended`，其访问权限被切断。稍后将其切换回 `active` 即可恢复访问。适用于休假或临时锁定场景。
- **归档（Archive，即移除）** — 彻底退出。它会暂停访问权限，**清除该成员的显式权限授予**，并**重新分配其未完成的工作**，以确保没有任务被搁置。适用于人员确实已离开的情况。

您可以先暂停，稍后再归档；没有规定必须直接跳转到归档。

---

## 2. 暂停成员（可逆）

![成员页面](../../user-guides/screenshots/light/company/access.png)

打开 **设置 → 成员**，点击该成员行的 **编辑**。在 **编辑成员** 对话框中，将 **成员状态** 设置为 **已暂停** 并保存。该行仍可见，并带有 `suspended` 徽章——历史记录得以保留——且其对公司资源的访问权限立即被切断。

若需稍后恢复其访问，重复上述步骤并将状态改回 **活跃**。

---

## 3. 归档成员并移交其工作

归档是完整的移除操作，并在同一步骤中处理未完成的工作。在 **设置 → 成员** 页面，对该成员行使用 **移除** 操作。将弹出一个对话框，标题处有关于在用户从分配字段中隐藏之前移动活跃分配的说明，并提供一个 **任务重新分配** 选择器，列出公司的活跃成员和智能体。

选择谁应继承离开成员的未完成工作并确认。随后 Paperclip 将：

- 将其任何 **进行中** 的问题重置为 `todo` 并清除活跃检出状态，以便新受让人从零开始；
- 将其其他未解决的问题 **重新分配** 给您选择的人员或智能体；
- 清除该成员的显式权限授予并归档成员资格。

系统将通过提示消息确认已移动多少项分配。请在归档前安排好接替者，以确保工作持续流动。（暂停操作不会重新分配任何内容——因此，如果您在任务中途暂停某人，其问题将保留在其名下，直到您手动重新分配或稍后归档。）

---

## 4. 理解实际切断访问权限的机制

这一点值得精确说明：**访问权限由成员状态控制，而非由该人员仍持有的任何令牌决定。** 一旦其成员状态变为 `suspended` 或 `archived`，其发起的所有公司范围请求都将通过访问检查失败——即使他们仍拥有有效的登录会话或配置文件中存在的个人看板 API 密钥。不存在“仍登录因此仍允许”的漏洞；授权层会在每次请求时重新检查成员资格。

因此，暂停或归档成员资格是持久的控制手段。其个人看板 API 密钥是用户自行管理的用户范围凭证；一旦成员资格消失，这些密钥对*此*公司立即失效。

---

## 5. 如果他们有实例级访问权限，也需移除

公司移除操作**不会**触及实例管理员权限。如果该人员是实例管理员，或您在实例级别授予了其公司访问权限，请在 **设置 → 实例：访问** 中单独清理（需要实例管理员权限）：

![实例访问页面](../../user-guides/screenshots/light/settings/instance-access.png)

1. 在左侧窗格中搜索该用户并点击以打开其详细信息视图。
2. 如果其带有绿色的实例管理员盾牌图标，点击 **移除实例管理员**。
3. 在下方的 **公司访问** 网格中，取消勾选其不应再访问的所有公司，并点击 **保存公司访问**。检查下方的 **当前成员资格** 以确认结果。

实例管理员可以访问*每个*公司，无论其个人成员资格如何，因此如果适用，此步骤至关重要。参见 [设置 → 实例：访问](../administration/settings.md#instance-access) 和 [角色与权限](../administration/roles-and-permissions.md#instance-admin-the-layer-above-companies)。

---

## 6. 验证

回到 **设置 → 成员** 页面，该成员将显示 `suspended` 徽章或已从活跃列表中消失。成员记录及其历史仍保留以供审计——每次状态变更都会写入 [活动日志](../guides/day-to-day/activity-log.md)——因此您始终可以查看谁被移除、何时移除以及由谁移除。

---

## 偏好使用终端？CLI 等效命令

相同的移除步骤可通过命令行执行，适用于脚本化去配置：

```sh
# Suspend (reversible)
paperclipai member update <member-id> --company-id <company-id> --payload-json '{"status":"suspended"}'

# Archive, reassigning open work to another member or agent
paperclipai member archive <member-id> --company-id <company-id> \
  --payload-json '{"reassignment":{"assigneeUserId":"<replacement-user-id>"}}'

# Instance-level cleanup (instance admin)
paperclipai admin user demote <user-id>
paperclipai admin user company-access <user-id>
paperclipai admin user company-access:update <user-id> --payload-json '{"companyIds":[...]}'

# Confirm
paperclipai member list --company-id <company-id>
```

---

## 相关

- [角色与权限](../administration/roles-and-permissions.md) — 这些状态变更所接入的访问模型。
- [公司管理](../administration/company.md#members) — 成员页面。
- [设置](../administration/settings.md#instance-access) — 实例访问屏幕。
- [添加人类队友](./add-a-human-teammate.md) — 入职方向。
