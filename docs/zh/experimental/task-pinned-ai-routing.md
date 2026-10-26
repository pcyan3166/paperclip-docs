---
paperclip_version: v2026.1012.0
seo_title: 任务固定的 AI 连接池路由
seo_description: 让新任务轮流使用 AI 账户，同时使每项任务保持使用原账户。启用路由器、创建连接池，再将其分配给智能体。
---

# 任务固定的 AI 路由（连接池）

你可以将工作分散到多个 AI 账户，但不会让同一段对话中途切换账户。将已连接的账户组成一个**连接池**，然后将智能体指向此连接池。每项新任务都会轮到队列中的下一个账户，并且在任务的整个生命周期内，始终使用同一账户、执行框架和会话。

此功能属于实验性功能，默认关闭。它需要两个条件：实例级 **AI connection routers**（AI 连接路由器）标记（`enableAiConnectionRouters`）以及实际负责路由的已安装插件。仅安装插件不会开启路由，开启路由也不会自动启用任何连接池。

## 开始之前

- **路由器插件。** 连接池由声明 `ai.connections.route` 能力的插件提供。Paperclip Cloud 的连接池插件就是一个示例。没有此类插件，就没有可添加的连接池连接器。如果你要自行开发，请参阅 Plugin SDK 参考中的 [AI connection routers](../reference/plugins/sdk.md#ai-connection-routers)。
- **实例级标记。** 与大多数实验性功能不同，此功能即使已经开启，**Experimental 设置页面上也没有对应开关**。在自托管实例上，实例管理员需要通过 API 设置：

  ```sh
  curl -X PATCH https://YOUR-PAPERCLIP-HOST/api/instance/settings/experimental \
    -H "Content-Type: application/json" \
    -d '{"enableAiConnectionRouters": true}'
  ```

  发送请求时必须使用实例管理员凭据。在 Paperclip Cloud 上，此值由 Cloud 管理；如果需要更改，请联系 Paperclip。请参阅[实例管理员 API](../reference/api/instance-admin.md)。
- **已保存的 AI 账户。** 连接池只能使用已连接且你有权在此公司中使用的账户。请先连接账户，例如 [OpenAI](../connectors/openai.md)、[Anthropic](../connectors/anthropic.md)、[Grok](../connectors/xai.md) 或 [OpenRouter](../connectors/openrouter.md)。
- **Manage connections 权限。** 只有连接管理人员可以创建、编辑或移除连接池。

## 创建连接池

1. 打开 **Connectors**，找到插件添加的连接池连接器（例如 **AI connection pool**），然后选择 **Add connection pool**（添加连接池）。
2. 在 **Connections**（连接）步骤中，勾选要加入连接池的账户。你可以搜索列表；**Connect a new account**（连接新账户）会在新标签页中打开目录，不会丢失当前草稿。标记为 **Needs attention**（需要处理）的账户必须修复后才能加入。选择 **Continue**（继续）。
3. 在 **Order**（顺序）步骤中排列账户。新任务会按此顺序轮流分配。
4. 选择 **Create pool**（创建连接池）。页面会提示 *“连接池已创建并暂停。准备好后即可启用。”*
5. 打开连接池，勾选 **Enable this pool**（启用此连接池），然后选择 **Save changes**（保存更改）。

连接池中的每个账户都会关联到与其服务提供商相匹配的执行框架：OpenAI 账户使用 Codex，Anthropic 账户使用 Claude，Grok 账户使用 Grok，OpenRouter 账户使用 OpenCode。一个连接池中可以混用不同服务提供商的账户。

### 高级选项

打开连接池页面上的 **Advanced**（高级）以微调设置：

- **Skip connections near their usage limit**（跳过接近用量上限的连接）会在分配新任务前检查每个账户的用量，并跳过达到或超过 **Skip at … % usage**（用量达到 …% 时跳过）阈值的账户（默认为 90%）。关闭后，连接池只会按顺序轮换，不检查用量。
- **Model**（模型）和 **Effort (optional)**（推理力度，可选）用于为每个账户设置默认模型和推理力度。

连接池页面还会显示 **Used by**（使用者）：公司内所有设置为使用此连接池的智能体（包括已暂停的智能体），并提供指向其个人资料的链接。

## 为智能体指定连接池

1. 打开智能体设置，找到 **AI connection**（AI 连接）字段。
2. 选择连接池。它会显示为 *“<连接池名称> · Experimental pool”*。选择 **Individual account**（单个账户）即可恢复为普通的单连接方式。
3. 如果 Paperclip 询问 *“使用 <连接池名称>？”*，它是在提醒你：*“使用此连接池会重置现有使用池外账户的会话。”* 选择 **Use pool**（使用连接池），然后保存智能体设置。

之后会显示提醒：*“新任务将轮换账户。现有任务继续使用原账户。”* 智能体只能使用已启用且至少包含一个与其执行框架兼容账户的连接池。创建新智能体时不会提供连接池选项；智能体创建后，请在其设置中指定连接池。

## 路由行为

- **每个连接池共用一个轮换顺序。** 共享连接池的所有智能体都会共用同一个轮换位置，因此工作会分散到不同账户，而不是每个智能体都从第一个账户开始。
- **每项任务都会固定使用一个账户。** 任务第一次运行时会选择一个账户；此任务后续所有运行（包括会话重置或压缩后）都会使用同一账户和执行框架。编辑或移除池成员，只会影响尚未固定账户的任务。
- **模型和推理力度可以更改，账户不能更改。** 你仍可在任务编辑器中选择不同的模型或推理力度。如果固定账户不支持所选项，本次执行会使用该账户的默认值，并记录类似 *“此成员无法使用模型覆盖设置；正在使用其默认值。”* 的说明。
- **等待而非失败。** 如果所有账户都超过使用限额，任务会等待，而不是失败。任务横幅会显示 **Pool exhausted**（连接池额度耗尽）、下一次检查用量的时间以及 *“任务等待时仍会保留所选账户。用量允许后将恢复工作。”* 此等待不会消耗任务通常的重试额度。
- **可以查看使用了哪个账户。** 智能体页面上的执行详情会显示 *“任务固定的连接池账户 · <账户名称>”*。

如果固定账户需要重新连接，任务常规连接卡片会重新连接同一个账户，智能体仍会继续使用该连接池。

## 移除连接池

在连接池页面打开 **⋯** 菜单并选择 **Remove connection**（移除连接），或者从 **Connectors** 中将其移除。确认提示会说明：*“此连接池中的连接会保留。”* 移除连接池会阻止新的账户分配；已经运行的任务会继续使用已分配的账户。即使路由功能或插件后来被关闭，仍可以移除连接池。

## API

拥有 Manage connections 权限的董事会用户可以直接管理连接池：

| 端点 | 用途 |
| --- | --- |
| `GET /api/companies/{companyId}/ai-connection-pools` | 列出公司的连接池及其 `revision`。 |
| `POST /api/companies/{companyId}/ai-connection-pools` | 创建连接池；或者在请求中提供现有 `id` 和当前 `expectedRevision` 来更新连接池。请求正文还需包含所属的 `pluginKey`，以及包含 `name`、`enabled`、`mode`（`round_robin` 或 `usage_aware`）、`thresholdPercent` 和 `members` 的 `config`。 |
| `GET /api/companies/{companyId}/ai-connection-pools/{poolId}/inspection` | 读取每个连接池成员最近一次的用量观测结果。 |
| `DELETE /api/companies/{companyId}/ai-connection-pools/{poolId}` | 移除连接池。请求正文必须包含当前的 `expectedRevision`。 |

如果 `expectedRevision` 已过时，请求会被拒绝，因此两个人同时编辑同一连接池时不会相互覆盖。常规连接更新和删除端点会拒绝处理连接池。

## 故障排查

| 你看到的内容 | 含义 |
| --- | --- |
| *“请联系实例运维人员启用 AI 连接路由。”* | `enableAiConnectionRouters` 已关闭。实例管理员需要将其开启。 |
| *“请在 Plugins 中启用连接池插件。”* | 路由器插件已安装，但尚未运行。请在 Plugins 中启用它。 |
| 智能体的 **AI connection** 字段显示 *“连接池不可用——请启用路由和连接池”* | 智能体指定了一个已暂停、已移除或被实例级标记阻止的连接池。请启用连接池，或选择 **Individual account**。 |
| *“连接管理人员可以编辑此连接池。”* | 你没有 Manage connections 权限。 |
| 任务上显示 **Pool exhausted** | 所有账户都达到用量上限。用量允许后，任务会自动恢复。 |

## 相关指南

- [查看 AI 账户用量](../connectors/ai-usage.md)
- [连接器访问方式](../connectors/access-model.md)
- [Plugin SDK：AI 连接路由器](../reference/plugins/sdk.md#ai-connection-routers)
- [实验性功能概览](overview.md)
