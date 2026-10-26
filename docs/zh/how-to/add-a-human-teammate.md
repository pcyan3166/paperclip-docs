---
paperclip_version: v2026.1023.0
seo_title: 添加人类团队成员
seo_description: 将联合创始人、运营人员或只读查看者作为董事会成员引入您的公司，并选择与其需求相匹配的访问级别。
---

# 添加人类团队成员

Paperclip 公司并非单人游戏。您可以将其他人作为董事会成员引入——例如，一位关注相同智能体的联合创始人、一位负责处理收件箱的运营人员，或一位仅需仪表盘只读权限的查看者。这是从应用程序中完成的端到端流程：您创建一个邀请链接，他们打开它，即可加入。

整个往返过程只需几分钟。邀请仅支持**复制链接**——Paperclip 不会发送邮件，因此您需要自行分享链接。

**开始之前：** 创建邀请需要 `users:invite` 权限。所有者和管理员默认拥有此权限。如果您不确定自己拥有什么权限，请参阅 [角色与权限](../administration/roles-and-permissions.md)。

---

## 1. 决定他们应获得的角色

每个邀请都包含一个角色，这是您的团队成员接受邀请后立即获得的角色。请选择能让他们完成工作所需的最小角色：

- **查看者 (Viewer)** — 只读。适合利益相关者。
- **运营者 (Operator)** — 可以编辑公司工作并邀请人员，但不能批准加入或管理权限。这是默认选项，也是大多数动手型团队成员的合适选择。
- **管理员 (Admin)** — 还可以批准加入请求。
- **所有者 (Owner)** — 完全控制，包括管理其他成员的权限。

他们加入后您随时可以更改此设置（步骤 5），因此不必过度纠结。完整说明请参阅 [角色与权限](../administration/roles-and-permissions.md)。

---

## 2. 创建邀请链接

![成员页面的邀请选项卡，包含“邀请人员”卡片](../../user-guides/screenshots/light/company/invites.png)

打开 **设置 → 成员** 并切换到 **邀请** 选项卡。在 **邀请人员** 卡片中，在 **选择角色** 下选择步骤 1 中的角色——每个选项都显示该角色权限的简短描述——然后点击 **创建邀请**。Paperclip 会同时执行以下三项操作：

1. 基于您当前的 Paperclip 域名生成一个新的、一次性的邀请链接；
2. 将 URL 复制到您的剪贴板（如果浏览器允许——否则会通过提示通知您手动复制）；
3. 将链接放入 **最新邀请链接** 面板，并在下方的 **邀请历史** 表格中添加一行，带有 **活跃** 徽章。

如果需要再次复制，请使用 URL 旁边的 **复制链接** 按钮。该链接是**一次性**的，并在 72 小时后过期——如果链接失效，只需重新创建一个。完整指南请参阅 [公司管理 → 邀请](../administration/company.md#invites)。

---

## 3. 分享链接

通过您通常分享机密信息的方式发送 URL——私信、密码管理器或私有频道。接受活跃链接的人将以该链接携带的角色加入您的公司，因此请将其视为短期密码，不要在任何公开地方发布。如果您改变了主意，想要撤销一个尚未使用的邀请？在 **邀请历史** 表格中点击该行的 **撤销**。

---

## 4. 您的团队成员打开链接并加入

当他们打开 URL 时，会进入一个带有您公司名称和标志的 Paperclip 加入页面。如果他们还没有账户，页面会先引导他们完成注册，然后返回邀请页面。一旦他们登录并接受，就会**立即**以邀请中的角色加入——人类邀请没有审批步骤。他们会看到 **您已加入组织** 以及一个 **打开董事会** 按钮。

Paperclip 仍在后台记录一条加入请求，该请求已自动批准，因此邀请的 **加入请求** 列和审计跟踪保持完整。

> **如果他们看到“未启用电子邮件和密码注册”：** 您的实例已关闭自助注册，因此被邀请者无法创建接受邀请所需的账户。这由 `auth.disableSignUp` 配置字段（或 `PAPERCLIP_AUTH_DISABLE_SIGN_UP` 环境变量）控制。要让被邀请的团队成员注册，请确保已启用注册——将 `auth.disableSignUp` 设置为 `false`（其默认值）或确认 `PAPERCLIP_AUTH_DISABLE_SIGN_UP` 未设置为 `true`——然后重启实例并让他们重新打开邀请链接。完整认证模式设置请参阅 [启用多用户登录](./enable-multi-user-login.md)。

---

## 5. 微调其访问权限

您的团队成员已经是具有邀请角色的**活跃**成员。要调整其角色或授予额外权限，请打开 **设置 → 成员**，点击其行的 **编辑**，在授权网格中设置角色、状态和任何显式授权，然后保存。（即使您稍后更改其角色，显式授权仍会保留——请参阅 [角色与权限](../administration/roles-and-permissions.md#how-grants-combine-precedence)。）

> **仍然看到待处理请求？** 在人类邀请开始即时加入之前提交的请求可能仍在等待。如果您拥有 `joins:approve` 权限，成员列表上方会有一个 **待处理人类加入** 卡片，带有 **批准人类** / **拒绝人类** 按钮。完整的 **加入请求队列** (`/inbox/requests`) 涵盖人类和智能体请求，并带有 **状态** 和 **请求类型** 过滤器——目前它主要处理智能体请求。

![带有状态和请求类型过滤器的加入请求队列；当没有等待项时列表为空](../../user-guides/screenshots/light/company/join-requests.png)

---

## 6. 验证

他们现在出现在 **设置 → 成员** 中，带有 `active` 状态徽章和您赋予的角色。就这样——他们可以登录并查看公司。上述每个步骤（创建邀请、接受邀请、激活成员资格）都会写入 [活动日志](../guides/day-to-day/activity-log.md)，因此整个入职过程事后可审计。

---

## 偏好终端？CLI 等效命令

上述每个步骤都有对应的 CLI 命令，便于脚本化入职流程。它们与 UI 使用的同一 API 一一对应。确切负载请参阅 [访问 CLI 参考](../reference/cli/access.md#invites)。

```sh
# Create the invite (returns the invite URL)
paperclipai invite create --company-id <company-id> --payload-json '{"role":"operator"}'

# Human invites join on accept; agent requests (and any older
# pending ones) still need approval
paperclipai join list --company-id <company-id> --status pending
paperclipai join approve <request-id> --company-id <company-id>

# Confirm they're in
paperclipai member list --company-id <company-id>
```

使用 `paperclipai invite revoke <invite-id>` 撤销未处理的邀请（注意：这需要邀请 **ID**，而不是令牌）。

---

## 相关

- [公司管理](../administration/company.md) — 成员页面（含邀请选项卡）和加入请求的完整说明。
- [角色与权限](../administration/roles-and-permissions.md) — 每个角色和授权实际允许的操作。
- [移除成员](./offboard-a-member.md) — 当有人离开时的反向操作。
- [启用多用户登录](./enable-multi-user-login.md) — 如果您的实例仍处于本地信任模式，这是必需的第一步。
