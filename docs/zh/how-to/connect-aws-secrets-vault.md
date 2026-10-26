---
paperclip_version: v2026.525.0
seo_title: 连接 AWS Secrets Manager 保险库
seo_description: 让 Paperclip 通过区域、命名空间和名称前缀自动发现您已使用的保险库，而无需手动复制 ARN。
---

# 连接 AWS Secrets Manager 保险库

将 AWS Secrets Manager 接入 Paperclip 的“密钥”页面，使其能够自动发现您已使用的保险库——包括区域、命名空间和名称前缀——而无需您手动复制 ARN。完成本指南后，您将拥有一个已保存的提供商保险库，智能体可以针对该保险库进行轮换，且输入字段已根据您的实时 AWS 账户预填。

如果您只需要了解概念模型——严格模式、本地加密提供商、环境变量覆盖——请先阅读 [密钥](../reference/deploy/secrets.md)。本页是面向运维人员的设置操作指南。

---

## 开始之前

在运行 Paperclip 服务器的主机上，您需要准备以下三项内容：

- 进程可以读取的 AWS 凭证（通常的凭证链：环境变量、实例角色或 `~/.aws/credentials`）。这些凭证需要 `secretsmanager:ListSecrets` 权限——这是发现功能唯一调用的权限。在运行时读取和写入密钥值使用密钥解析到的 ARN 上通常的 `GetSecretValue` / `PutSecretValue` 权限；如果您希望[将值写入关联的密钥](#change-the-value-of-a-linked-aws-secret)，还需要 `secretsmanager:UpdateSecretVersionStage` 权限——该权限用于支持写入失败时的回滚操作。
- 您打算从中读取数据的区域，例如 `us-east-1`。
- 在 Paperclip 中公司的所有者角色——只有所有者才能写入 `secret_provider_config` 行。

Paperclip 的发现功能仅读取**元数据**。它不会调用 `GetSecretValue`，因此候选项预览永远不会拉取明文。

---

## 1. 打开“提供商保险库”选项卡

1. 登录并打开 **Company Settings → Secrets**（公司设置 → 密钥）。
2. 切换到 **Provider vaults**（提供商保险库）选项卡。您应该看到每个提供商对应一个部分；我们需要的是 **AWS Secrets Manager** 部分。
3. 在 AWS 行上点击 **Add vault**（添加保险库）。

保险库表单以*创建*模式打开。发现面板仅在此处显示——一旦保险库存在，再次打开同一表单时将处于编辑模式，不再显示发现面板。

---

## 2. 输入区域并运行发现

保险库表单包含三个头部字段（提供商、显示名称、状态）以及一组 AWS 路由字段主体。

1. 保持 **Provider**（提供商）设置为 AWS Secrets Manager。
2. 输入一个 **Display name**（显示名称），该名称将显示在每个密钥的保险库选择器中，例如 `prod-us-east-1`。
3. 在 **AWS region**（AWS 区域）中填写您想要检查的区域，例如 `us-east-1`。区域是唯一的必填 AWS 字段——在设置区域之前，发现功能处于禁用状态。
4. 如果您已经知道组织密钥所用的斜杠分隔路径，可选填 **Namespace**（命名空间）和 **Secret name prefix**（密钥名称前缀）。发现功能将使用您输入的内容作为初始查询，但仅使用区域也能正常工作。
5. 点击 **Find existing AWS values**（查找现有 AWS 值）。

Paperclip 使用草稿字段调用 `POST /companies/{companyId}/secret-provider-configs/discovery/preview` 并内联渲染结果。在请求进行中时，按钮会显示加载指示器——在大型账户上，元数据扫描可能需要几秒钟。

> **提示：** 如果按钮呈灰色且下方显示“Enter an AWS region before discovery.”（请先输入 AWS 区域再进行发现），说明您忘记填写区域字段。

---

## 3. 选择候选项并预填表单

当发现功能返回结果时，面板会切换到以下三种状态之一：

- **找到候选项。** 每个推断出的保险库对应一行，每行标注了 Paperclip 检测到的命名空间和名称前缀，以及样本数量和第一个采样密钥的名称（*值*不会显示）。点击与您想要注册的保险库匹配的那一行上的 **Use values**（使用值）。上方的 AWS 字段——区域、命名空间、密钥名称前缀、KMS 密钥 ID、所有者和环境标签——将从该候选项中填充，如果您未填写显示名称，它也会从候选项中设置。
- **无候选项。** 您将看到“No AWS vault metadata candidates found. Manual entry is still available.”（未找到 AWS 保险库元数据候选项。仍可进行手动输入）。请自行输入路由字段；除区域外，它们均为可选。
- **警告。** 黄色行会显示诸如节流或部分扫描等问题。保险库仍然可以安全保存——Paperclip 只是无法采样到期望的数量。稍后重新运行发现通常可以清除节流警告。

您可以在编辑字段后重新运行发现——例如通过不同的命名空间进行缩小范围——并选择不同的候选项。每次点击 **Use values** 时，表单都会重新填充。

随本版本发布的 `provider-vaults-tab.png` 截图展示了面板的填充状态。

---

## 4. 保存保险库

1. 如果希望此保险库成为 Paperclip 在智能体轮换基于 AWS 的密钥且未明确选择保险库时写入的目标，请勾选 **Default for AWS Secrets Manager**（AWS Secrets Manager 默认值）复选框。
2. 除非您正在捕获尚未希望解析的内容，否则保持 **Status**（状态）为 **Ready**（就绪）。下拉菜单有四种状态：**Ready**（就绪，实时可用）、**Warning**（警告，实时可用但健康检查发现问题——通常自动设置）、**Disabled**（已禁用，保留元数据但阻止 Paperclip 使用它）以及 **Coming soon**（即将推出，用于未实现的提供商——GCP Secret Manager、HashiCorp Vault）。
3. 点击 **Create vault**（创建保险库）。

服务器在保存时运行健康检查（与每个保险库的健康按钮绑定的检查相同）。如果 AWS 拒绝凭证或区域不可达，保险库仍会保存，但会进入带有提供商消息的警告状态——修复底层凭证后，点击卡片上的 **Check health**（检查健康）即可。

---

## 5. 将密钥关联到新保险库

现在保险库已存在，该公司的任何密钥都可以使用它。

1. 切换回 **Secrets**（密钥）选项卡。
2. 在任何现有的基于 AWS 的密钥上，打开更新对话框并点击 **Update value**（更新值）（如果密钥是无法接受值写入的外部引用，则点击 **Update reference**（更新引用）——见下文）。对话框中有一个 **Provider vault**（提供商保险库）下拉菜单——选择您刚刚创建的保险库，Paperclip 将通过它向 AWS 写入新版本。下拉菜单的默认选项标记为 **Deployment default**（部署默认值），任何具有 `blockReason`（已禁用、健康检查失败）的保险库都会列出但不可选择。
3. 对于新密钥，相同的保险库选择器也会出现在创建对话框中。

在此保险库创建之前已存在的密钥仍指向它们原本指向的保险库。下次轮换密钥时，通过 **Update value** 重新指向新保险库。

---

<a id="change-the-value-of-a-linked-aws-secret"></a>
## 更改关联 AWS 密钥的值

过去，关联密钥是单向的：Paperclip 可以读取 AWS 值并重新指向链接，但如果凭证本身发生变化，您必须前往 AWS 控制台并自行放入新值。现在不再需要这样做了。AWS Secrets Manager 支持将值写入关联的密钥，因此您可以从“密钥”页面轮换凭证。

在关联的 AWS 密钥上打开更新对话框，您将获得两个选项卡：

- **Write new value**（写入新值）——粘贴新凭证。Paperclip 将其写入您的引用已指向的 AWS 密钥。该字段下方显示它将写入的确切引用，以便您在提交前双重检查目标。
- **Change reference**（更改引用）——原始行为。将密钥指向不同的 AWS 密钥，而不向 AWS 写入任何内容。

详情面板会告诉您该密钥支持哪种操作。支持值写入的关联密钥显示为“*Paperclip resolves this provider reference and can write new values to it via Update value.*”（Paperclip 解析此提供商引用，并可通过“更新值”向其写入新值。）不支持的则显示为“*Paperclip resolves this provider reference but does not rotate the provider value.*”（Paperclip 解析此提供商引用，但不轮换提供商值。）——其菜单操作保持标记为 **Update reference**（更新引用）而非 **Update value**（更新值）。

在使用之前，有两点需要了解：

- **新值对所有人成为当前值。** Paperclip 写入新的 AWS 版本并将其设为 `AWSCURRENT`，这与在 AWS 控制台中进行轮换的效果完全相同。读取该 AWS 密钥的其他任何内容——Lambda、ECS 任务、其他团队的服务——也会获取新值。这通常是目的所在，但这意味着这不是仅针对 Paperclip 的更改。
- **不能同时执行两项操作。** 写入值和重新指向引用是独立的操作。选择一个选项卡，保存，如果需要第二个更改，则将其作为单独的更新运行。

写入值需要在目标 ARN 上使用 `PutSecretValue` 权限，这是在解析所需的 `GetSecretValue` 权限之上的。同时授予 `secretsmanager:UpdateSecretVersionStage` 权限：如果写入中途失败，Paperclip 会将 `AWSCURRENT` 移回之前当前的版本，如果没有该权限，回滚将无法运行——即使更新报告失败，新版本仍会保持实时状态。

> **提示：** Paperclip 继续跟踪 `AWSCURRENT`，而不是固定其刚刚写入的版本。稍后直接在 AWS 中轮换同一密钥，Paperclip 仍会解析最新值——写入路径不会将链接锁定到某个版本。

---

## 移除不再需要的保险库

当保险库在 AWS 中退役，或您错误地注册了它时，可以从 Paperclip 中移除它，而无需触及 AWS 本身。

1. 在 **Provider vaults**（提供商保险库）选项卡上，找到该卡片并使用其菜单选择 **Remove from Paperclip**（从 Paperclip 移除）。
2. 确认对话框明确说明了针对 AWS 的具体后果：“*This does not delete the remote AWS Secrets Manager vault, secrets, or any AWS data.*”（这不会删除远程 AWS Secrets Manager 保险库、密钥或任何 AWS 数据。）它仅丢弃 Paperclip 存储的路由元数据。
3. 点击 **Remove from Paperclip**（从 Paperclip 移除）。

引用已移除保险库的密钥将失去关联并回退到部署默认值。只要底层 AWS 值仍可读，它们将继续解析——然而，轮换将在下一次尝试时要求您选择新的保险库。随本版本发布的 `remove-provider-vault-confirmation.png` 截图展示了确切的措辞。

---

## 故障排除

**发现返回“AccessDenied”或类似错误** — 服务器运行的凭证无法列出该区域的密钥。确认角色具有 `secretsmanager:ListSecrets` 权限，然后重新运行发现。错误消息会逐字显示 AWS 的原因。

**发现找到零个候选项，但您知道密钥存在** — 要么区域错误，要么账户中的每个密钥都具有 Paperclip 无法聚类的非默认名称模式。回退到手动输入——自行填写区域、命名空间和密钥名称前缀，然后保存。发现功能是一种便利，而非必需。

**“Find existing AWS values”（查找现有 AWS 值）始终无法启用** — 该按钮依赖于非空区域。相同的限制也适用于 **Create vault**（创建保险库）——没有区域无法保存 AWS 保险库。

**更新值下拉菜单显示保险库但呈灰色** — 其状态为已禁用或最后一次健康检查失败。点击保险库卡片的 **Check health**（检查健康）按钮重新测试，或以编辑模式打开它并将状态切换回就绪。

**您希望在不写入 AWS 的情况下对其进行测试** — 以 **Disabled**（已禁用）状态保存保险库。元数据已存储，编辑时仍可见发现结果，且在您切换回状态之前，没有轮换可以针对它。

**“… does not support writing values to external reference secrets”（… 不支持向外部引用密钥写入值）** — 该消息指出了拒绝写入的提供商。AWS Secrets Manager 实现了写入透传，因此看到此消息意味着密钥由不支持该功能的提供商支持——GCP Secret Manager 和 HashiCorp Vault 在当前构建中为占位符。将凭证移至基于 AWS 的密钥，或在该提供商自己的控制台中更改值，并将 Paperclip 密钥保留为普通引用。

**“Provide either a new value or a new external reference, not both”（请提供新值或新外部引用，而非两者兼有）** — 更新同时携带了新值*并*指向了不同的引用。请逐一执行：写入值，保存，然后在第二次更新中更改引用（或反之亦然）。

**关联密钥仅提供“Update reference”（更新引用）** — 要么密钥尚未存储外部引用，要么其提供商未提供写入密钥值的选项。仅支持该功能的提供商才会获得 **Write new value**（写入新值）选项卡。

---

## 相关

- [密钥](../reference/deploy/secrets.md) — 密钥存储模型、严格模式以及密钥引用在运行时如何解析。
- [密钥 API](../reference/api/secrets.md#rotating-an-external-reference-secret) — 通过 API 执行相同的值写入，包括所有验证规则。
- [更新或轮换提供商 API 密钥](rotate-provider-api-key.md) — 路径 B 使用如上创建的提供商保险库。
