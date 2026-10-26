---
paperclip_version: v2026.1023.0
seo_title: 插件 SDK
seo_description: Paperclip 插件的 worker 端编写工具包。可在 worker 入口点中导入，用于声明插件并订阅主机事件。
---

# 插件 SDK

`@paperclipai/plugin-sdk` 是 Paperclip 插件的 worker 端编写工具包。在插件 worker 入口点中导入它，即可声明插件、订阅主机事件、注册作业和数据馈送、对主机执行 RPC，并访问托管数据库、密钥、状态以及 Paperclip API 的其余部分。

本页面向**插件作者**：负责编写并打包插件代码的开发者。如果你只是运行插件（安装、配置、启用或停用），请参阅[管理 → 插件](../../administration/plugins.md)。

> 插件运行时目前处于 alpha 阶段。SDK 在不同 Paperclip 版本之间仍可能发生破坏性变更；请固定 `@paperclipai/plugin-sdk` 和 `@paperclipai/shared` 的版本，并在升级时重新阅读本页。

---

## 适用场景

需要以下能力时可使用插件 SDK：

- 添加可长期运行并对 Paperclip 事件作出响应的 worker（例如 `issue.created`、`agent.run.completed` 等）。
- 在 Paperclip 界面中提供新页面、小组件、启动器或设置。
- 注册定时作业、webhook、工具，或由插件管理的代理和例程。
- 在插件代码旁提供一个托管数据库命名空间。
- 将新的环境驱动程序（自定义沙箱／执行后端）接入 Paperclip。

## 不适用场景

- **让 Paperclip 支持新的 AI 运行时。** 请改用[适配器](../adapters/creating-an-adapter.md)：适配器使用逐次运行的线路协议，而插件扩展的是服务器。
- **添加代理应遵循的指令。** 请编写[公司技能](../../how-to/write-a-company-skill.md)：技能是代理在运行时加载的 Markdown 文件，不是服务器代码。
- **一次性脚本。** 插件需要安装、启用和管理。临时自动化优先使用 REST API 或 CLI。

---

## 软件包接口

SDK 软件包提供两个入口点：

- `@paperclipai/plugin-sdk`：本页介绍的 worker 端接口。`definePlugin`、`runWorker`、`PluginContext`、协议辅助函数以及所有清单／协议类型均默认从此入口导入。
- `@paperclipai/plugin-sdk/ui`：用于插件 UI 扩展的界面包入口。本页不主要介绍这部分；面向操作者的说明请参阅[管理 → 插件](../../administration/plugins.md)，两个应用级插槽请参阅下文[覆盖整个应用的 UI 插槽](#覆盖整个应用的-ui-插槽)。

以下标识符均从 `@paperclipai/plugin-sdk` 导出，是权威名称；复制时请保持拼写不变。

### 覆盖整个应用的 UI 插槽

大多数 UI 插槽只挂载在某个页面或实体上。`PLUGIN_UI_SLOT_TYPES` 中有两种插槽类型会直接挂载到已登录用户的应用外壳：

- **`appShellOverlay`**：与整个应用并存的常驻 UI，例如浮动面板。它需要 `ui.action.register` 能力，并接收与小组件相同的主机上下文。用户在页面间导航时，该 UI 会保持挂载；用户切换账户或公司、退出登录或进入引导流程时才会卸载。因此组件卸载时要取消请求或订阅。面板的无障碍支持由插件负责。
- **`organizationSwitcher`**：用自定义组件替换侧边栏顶部的组织菜单。它需要 `ui.sidebar.register` 能力，并且必须是 React 组件（此插槽不支持自定义元素导出）。主机传入 `PluginOrganizationSwitcherProps`；其中的 `organizationSwitcher` 对象包含 `currentCompany`（当前公司的 `name` 和 `logoUrl`）、侧边栏的 `collapsed` 和 `open` 状态及 `onOpenChange`、离开页面前用于关闭移动端导航的 `onNavigate`、使用主机退出登录流程的 `onSignOut` 和 `signingOut`，以及按主机样式绘制公司图标的 `renderIcon`。

组织切换器采用谨慎的回退策略。插件加载时主机会预留空间；如果没有插件提供该插槽、提供者不止一个、发现过程失败或组件抛出异常，主机就会退回内置菜单，确保用户始终能访问自己的组织。`currentCompany` 描述的是此 Paperclip 实例中的公司；如果菜单还要显示外部账户或组织，应从所属服务获取相关信息。对于这两种插槽，都应把 props 和主机上下文视为显示信息，而非用户身份的证明；敏感操作必须由所属服务进行身份验证。替换菜单只会改变外观，不会改变用户对各公司的访问权限。

---

## 公共 API

### 插件定义

| 导出项 | 说明 | 适用时机 |
|---|---|---|
| `definePlugin` | 将 `PluginDefinition` 包装为 `PaperclipPlugin` 的工厂函数。应从 worker 入口点默认导出其结果。 | 始终使用：每个插件 worker 都从 `definePlugin({...})` 开始。 |
| `runWorker` | 使用给定插件和 `import.meta.url` 启动 worker 的 JSON-RPC 循环。 | 在 worker 入口点底部、`definePlugin` 之后调用。 |
| `startWorkerRpcHost` | 更底层的入口，返回可由你自行管理的 `WorkerRpcHost`（适用于测试或自定义运行框架）。 | 将 worker 嵌入非默认传输方式时，例如进程内测试。 |

类型：`PluginDefinition`、`PaperclipPlugin`、`PluginHealthDiagnostics`、`PluginConfigChangeContext`、`PluginConfigValidationResult`、`PluginWebhookInput`、`PluginApiRequestInput`、`PluginApiResponse`、`RunWorkerOptions`、`WorkerRpcHostOptions`、`WorkerRpcHost`。

#### 确认哪个公司的配置发生了变化

操作者保存插件配置时，主机会调用可选的 `onConfigChanged` 钩子，让插件无需重启 worker 即可应用更改。Paperclip 的插件配置按公司隔离，而每个插件只启动一个 worker（并非每家公司各启动一个），所以关键问题始终是：“这是谁的配置？”

`onConfigChanged` 现在通过第二个参数提供答案：

```ts
onConfigChanged?(
  newConfig: Record<string, unknown>,
  context?: PluginConfigChangeContext,
): Promise<void>;
```

`PluginConfigChangeContext` 只有一个字段：`companyId: string | null`，代表配置发生变化的公司；对于未绑定特定公司的实例级／全局保存，该值为 `null`。

这个钩子触发的时间可能比你预期得更早。worker 启动时，主机会通过与操作者保存配置相同的 `configChanged` 路径，重放每家已配置公司的存储配置。这样插件就能主动在 `setup()` 中处理公司级工作，而不必等待事件；因此它没有公司范围的调用可供读取 `ctx.config.get(companyId)`。主机还会将插件已配置的公司授权为 worker 的主动作用范围，因此插件通过自己的定时器和循环调用主机时会解析到这些公司之一，不会因缺少公司上下文而被拒绝。其他公司仍然无权访问。

启动时的重放尽力而为，只是提前通知，不构成保证：未实现 `onConfigChanged` 的 worker 仍可在运行时通过 `ctx.config.get(companyId)` 读取配置。相同配置可能重放多次，因此 `onConfigChanged` 必须具备幂等性。

#### 使用 `multiCompanyConfig` 声明多公司支持

由于启动重放会遍历每家已配置公司，如果插件只维护一份 worker 全局配置，最终就会悄悄以最后处理的公司配置运行。这会造成跨租户混用（把某家公司的令牌用于另一家公司的流量），因此主机会**默认拒绝**这种情况。

如果 worker 确实要在同一进程中服务多家公司，请在插件定义中设置 `multiCompanyConfig: true`，并以 `context.companyId` 为键保存各公司的状态：

```ts
import { definePlugin, runWorker, type PluginLogger } from "@paperclipai/plugin-sdk";

const configByCompany = new Map<string, Record<string, unknown>>();
let logger: PluginLogger | null = null;

const plugin = definePlugin({
  multiCompanyConfig: true,

  async setup(ctx) {
    logger = ctx.logger;
  },

  async onConfigChanged(newConfig, context) {
    if (!context?.companyId) {
      // Instance-wide save — nothing company-specific to rebind.
      return;
    }
    configByCompany.set(context.companyId, newConfig);
    logger?.info("Applied config for one company", { companyId: context.companyId });
  },
});

export default plugin;
runWorker(plugin, import.meta.url);
```

如果未设置 `multiCompanyConfig`（默认值），插件会被视为单租户。此时“默认拒绝”有明确含义：`configChanged` 已应用某家公司的配置后，如果收到另一家公司的不同配置，主机会以 `PLUGIN_RPC_ERROR_CODES.CROSS_TENANT_CONFIG` 拒绝，而不会覆盖现有配置。若不同公司 ID 对应的配置完全相同，则视为幂等重放，仍会被允许，因此包含相同配置的重复作用域记录不会造成问题。

`companyId` 一直存在于线路数据中，即 `ConfigChangedParams` 上的可选字段 `companyId?: string | null`。变化之处在于 SDK 现在会将它传给钩子，而不再丢弃。

<a id="required-agent-lifecycle-work"></a>
#### 必需的代理生命周期工作

有些插件负责管理代理工作所需的资源，例如沙箱、邮箱或其他系统中的账户。这些资源必须在代理接到第一个任务前就绪，并且必须在终止操作报告完成前清理完毕。为此，插件可以成为代理生命周期中的必需步骤。

在清单中声明 `agentLifecycle: true`，并同时声明 `agents.lifecycle.manage` 能力（若缺少该能力，清单会因“Agent lifecycle participation requires agents.lifecycle.manage”而被拒绝），然后实现 `onAgentLifecycle(request)`：

```ts
import { definePlugin, runWorker } from "@paperclipai/plugin-sdk";

const plugin = definePlugin({
  async setup() {},
  async onAgentLifecycle(request) {
    // Your own provider code. Make it idempotent and fence older versions.
    const done = await reconcileAgentResources(request);
    return {
      operationId: request.operationId,
      version: request.version,
      status: done ? "complete" : "pending",
    };
  },
});

export default plugin;
runWorker(plugin, import.meta.url);
```

每条生命周期命令提交后，主机都会调用此钩子。请求（`AgentLifecycleRequest`）包含 `companyId`、`agentId`、`operationId`、`version` 和当前 `phase`。在 `AgentLifecycleResult` 中原样返回 `operationId` 和 `version`；外部操作仍在进行时返回 `status: "pending"`（Paperclip 会再次调用），确认操作完成后返回 `"complete"`。接受删除请求不等于资源已经删除，所以只有资源确实清除后才能报告完成。若步骤失败，请抛出异常；代理页面会显示固定的失败提示（不会显示异常文本）和 **Retry** 按钮。

| `phase` | 插件应执行的操作 |
|---|---|
| `preparing` | 创建或协调代理资源，并等待资源就绪。 |
| `verifying` | 主机测试代理已保存的运行时配置后，完成剩余检查。 |
| `pausing` | 主机确认代理执行已停止后，停止相关资源。 |
| `resuming` | 重新启动资源并等待其就绪。 |
| `terminating` | 主机撤销代理密钥并停止执行后，停止相关资源。 |
| `cleaning_up` | 移除由插件管理的资源并等待移除完成。 |

在所有必需步骤完成前，分配给代理的工作都会保持排队，因此插件确实能够控制代理何时开始工作。

实现时请考虑以下规则：

- **调用可能重复。** 超时或重启后，同一阶段可能再次到达。请按公司和代理记录已处理的最高 `version`，忽略旧版本，并确保外部调用幂等。
- **生命周期权限范围有限。** 此能力不会授予写入代理配置或读取代理密钥的权限。如需调用 `ctx.agents.get`，请另行申请 `agents.read`。
- **代理雇用时就确定插件选择。** Paperclip 会在首次选择插件时记录代理所需插件（并跳过在该公司被停用的插件）。停用或移除已选插件会阻止该代理继续推进，直到插件恢复；之后安装的插件不会自动添加到已有代理。
- **这与资源事件收件箱相互独立。** 确认[持久资源生命周期事件](#持久资源生命周期钩子)不会完成生命周期步骤，反之亦然。

#### 通过 `onIdleDrain` 让实例进入休眠

在没有活动时会自动休眠的实例上，已启用的插件 worker 会阻止实例休眠，除非它实现了 `onIdleDrain(signal)`。准备休眠时，主机会停止向 worker 发送新工作，并询问此钩子是否仍有插件工作正在进行：

- 只有当定时器、套接字、分离运行的操作和清理工作都已结束，且在 `signal` 中止前不会再启动新工作时，才返回 `"none"`。
- 仍有工作时返回 `"present"`；无法确认是否结束时返回 `"unknown"`。这两种结果都会让实例保持唤醒。

不要为了看起来空闲而取消有用的工作。SDK 已经会统计仍在处理的请求和通知；若休眠被取消或超时，主机会释放保持唤醒状态（中止 `signal`）。因此，若钩子暂停了自己的后台工作，应在此时恢复。缺少钩子、使用旧版 SDK 或钩子抛出异常，都会按“保持唤醒”处理。无论钩子返回什么，已启用的作业和投递等持久工作仍会让实例保持唤醒。

### 插件上下文

`PluginContext` 是传给 `setup(ctx)` 的参数。它按功能提供客户端；以下类型均从 `@paperclipai/plugin-sdk` 导入：

| 客户端 | 用途 |
|---|---|
| `PluginConfigClient` | 读取并观察解析后的插件实例配置。 |
| `PluginLocalFoldersClient` | 检查和配置已声明的本地文件夹挂载（包括 `PluginLocalFolderStatus`、`PluginLocalFolderListing`、`PluginLocalFolderProblem`，以及用于 `configure`／`list` 调用的新导出结构 `PluginLocalFolderConfigureInput`、`PluginLocalFolderListOptions` 和 `PluginLocalFolderEntry`）。 |
| `PluginEventsClient` | 订阅主机事件（`ctx.events.on(...)`），并手动读取持久代理／项目生命周期钩子（`ctx.events.listLifecycle(...)`、`ctx.events.acknowledgeLifecycle(...)`）；详见[持久资源生命周期钩子](#持久资源生命周期钩子)。 |
| `PluginJobsClient` | 为已声明的作业注册处理程序（`ctx.jobs.register(...)`）。 |
| `PluginLaunchersClient` | 注册启动器的渲染和操作处理程序（`PluginLauncherRegistration`）。 |
| `PluginHttpClient` | 由主机策略控制的出站 HTTP。 |
| `PluginSecretsClient` | 解析实例配置中声明的密钥引用。 |
| `PluginActivityClient` | 将 `PluginActivityLogEntry` 写入主机活动日志。 |
| `PluginStateClient` | 在 `ScopeKey` 下读写有作用域限制的键值状态。 |
| `PluginEntitiesClient` | 对插件拥有的实体执行 upsert 和查询（`PluginEntityUpsert`、`PluginEntityQuery`、`PluginEntityRecord`）。 |
| `PluginProjectsClient`、`PluginExecutionWorkspacesClient`、`PluginCompaniesClient`、`PluginIssuesClient`、`PluginIssueRelationsClient`、`PluginIssueSummariesClient`、`PluginAgentsClient`、`PluginAgentSessionsClient`、`PluginGoalsClient`、`PluginSkillsClient` | 通过主机访问 Paperclip 核心领域数据，进行读写操作。 |
| `ctx.approvals` | 读取并处理公司审批；详见[响应交互和审批](#响应交互和审批)。`list`／`get` 需要 `approvals.read`，`decide` 需要 `approvals.respond`。SDK 源码中的接口名为 `PluginApprovalsClient`，但目前没有单独导出；可通过 `PluginContext` 使用。 |
| `ctx.routines` | 解析和协调由插件管理的 Paperclip 例程（`ctx.routines.managed`）。需要 `routines.managed` 能力。接口类型目前没有单独导出，但可通过 `PluginContext` 使用。 |
| `ctx.execution` | 流式接收长时间运行的 `execute` 调用的实时输出；详见[流式传输实时命令输出](#流式传输实时命令输出)。SDK 源码中的接口名为 `PluginExecutionClient`，但目前没有单独导出；可通过 `PluginContext` 使用。 |
| `ctx.access` | 读取和管理公司成员及邀请（`ctx.access.members`、`ctx.access.invites`）；详见[访问与授权](#访问与授权)。客户端为 `PluginAccessClient`，其下包含 `PluginAccessMembersClient` 和 `PluginAccessInvitesClient`。需要相应的 `access.*` 能力。 |
| `ctx.authorization` | 读取和管理授权许可、策略摘要、分配预览及授权审计记录；详见[访问与授权](#访问与授权)。客户端为 `PluginAuthorizationClient`，需要相应的 `authorization.*` 能力。 |
| `ctx.loginPty` | 在环境驱动程序 worker 中流式传输一个实时登录伪终端的输出和退出状态；详见[登录 PTY 与双工通道流](#登录-pty-与双工通道流可选)。SDK 源码中的接口名为 `PluginLoginPtyClient`，但目前没有单独导出；可通过 `PluginContext` 使用。 |
| `ctx.duplexChannel` | 在环境驱动程序 worker 中流式传输一个持久双工通道的数据和退出状态；详见[登录 PTY 与双工通道流](#登录-pty-与双工通道流可选)。SDK 源码中的接口名为 `PluginDuplexChannelClient`，但目前没有单独导出；可通过 `PluginContext` 使用。 |
| `PluginDataClient` | 注册供 UI 查询的数据馈送（`ctx.data.register(...)`）。 |
| `PluginActionsClient` | 注册可由主机调用的操作。 |
| `PluginStreamsClient` | 流式主机 API。 |
| `PluginToolsClient` | 注册清单中声明的工具实现（`ToolRunContext`、`ToolResult`）。 |
| `PluginMetricsClient`、`PluginTelemetryClient` | 发送指标和遥测数据。 |
| `PluginTracer`、`PluginSpan` | 为插件工作创建追踪 span（`ctx.tracer.startSpan(...)`）。 |
| `PluginLogger` | 结构化日志记录器（`ctx.logger.info/warn/error`）。 |
| `PluginDatabaseClient` | 访问插件声明的托管 Postgres 命名空间。 |

通过 `PluginMetricsClient` 调用 `metrics.write` 发送指标，或通过 `PluginLogger` 调用 `log` 写入日志时，可以传入可选的 `companyId`，将记录限定到某家公司；删除公司时，这些记录会级联删除。省略该字段或传入 `null`，则记录保留在实例级作用域。

`ctx.tracer` 可用于将一段工作包装在 span 中，使其出现在主机追踪记录里。调用方式与 `ctx.logger` 类似：调用 `ctx.tracer.startSpan("my-work", { attributes: { … } })`，在返回的 `PluginSpan` 上设置属性或状态，并在完成后调用 `end()`。这是一个刻意保持精简且不依赖 OpenTelemetry 的契约：SDK 不会引入 `@opentelemetry/api`，实际记录由主机完成。请注意两点：在追踪关闭时（或不存在活动主机追踪上下文时）创建的 span 是无害的空操作，不会记录任何内容，因此无需先检查就可以创建；主机还会在信任边界重新限制每个属性，未列入主机允许清单的属性不会进入已记录的 span。若需要显式使用默认空操作实现（例如避免测试依赖真实 tracer），可导入共享的 `NOOP_PLUGIN_TRACER` 和 `NOOP_PLUGIN_SPAN`。

问题领域辅助类型：`PluginIssueMutationActor`、`PluginIssueRelationSummary`、`PluginIssueCheckoutOwnership`、`PluginIssueWakeupResult`、`PluginIssueWakeupBatchResult`、`PluginIssueRunSummary`、`PluginIssueApprovalSummary`、`PluginIssueCostSummary`、`PluginBudgetIncidentSummary`、`PluginIssueInvocationBlockSummary`、`PluginIssueOrchestrationSummary`、`PluginIssueSubtreeOptions`、`PluginIssueAssigneeSummary`、`PluginIssueSubtree`、`IssueDocumentSummary`。

`ctx.executionWorkspaces` 的工作区元数据类型：`PluginExecutionWorkspaceMetadata`。

代理会话辅助类型：`AgentSession`、`AgentSessionEvent`、`AgentSessionSendResult`。调用 `ctx.agents.sessions.sendMessage(...)` 发送消息后，收到的 `eventType: "done"` 类型 `AgentSessionEvent` 中，`message` 字段是该次运行面向用户的规范最终回复；若运行没有生成回复文本，该字段为 `null`。应将此字段转发给提出请求的人，无需再从 `"chunk"` 事件重新拼接回复。`eventType: "error"` 事件中的 `message` 则包含运行状态字符串。

工作区、事件和作用域辅助类型：`PluginWorkspace`、`PluginEvent`、`EventFilter`、`ScopeKey`、`PluginJobContext`、`ResourceLifecycleEvent`。

#### 持久资源生命周期钩子

如果插件需要为每个代理或项目在 Paperclip 之外配置资源（如邮箱、云文件夹或其他工具中的席位），并在代理终止或项目归档时清理资源，那么 `ctx.events.on(...)` 的即发即弃模式就不够可靠：错过事件意味着漏掉资源配置。生命周期钩子提供按插件隔离的持久收件箱。每当代理或项目创建或状态变化时，Paperclip 都会写入日志；插件完成自己的操作后确认事件。未确认的事件会再次投递。

通过 `ctx.events` 上的两个调用读取收件箱。二者都受 `events.subscribe` 能力控制，并限定在单家公司：

- `listLifecycle(companyId, limit?, afterId?)` 返回最多 `limit` 个待处理事件（默认 `50`，必须是 `1` 到 `100` 之间的整数），类型为 `ResourceLifecycleEvent[]`。分页时，将上次看到的最后一个 `id` 作为 `afterId` 传入。
- `acknowledgeLifecycle(companyId, eventId)` 将一条事件标记为此插件已处理。只有外部操作确实成功后才能调用。

若插件尚未就绪，或已在该公司停用，这两个调用都会被拒绝。

每个 `ResourceLifecycleEvent` 都包含 `id`、`companyId`、`resourceId`、`createdAt`，以及一组 `resourceType` 和 `action`：

| `resourceType` | `action` | Paperclip 记录事件的时机 |
|---|---|---|
| `agent` | `create` | 代理无需审批即可创建，或待处理的雇用申请获批。 |
| `agent` | `pause` | 代理被暂停（由人员暂停或因预算停止）。 |
| `agent` | `resume` | 暂停的代理重新变为活动状态。 |
| `agent` | `terminate` | 代理被终止。 |
| `project` | `create` | 项目被创建。 |
| `project` | `update` | 项目详情或目标关联变化；项目工作区被添加、编辑或移除；项目关联仓库完成同步；或已归档项目被恢复。 |
| `project` | `archive` | 项目被归档。 |

事件刻意不包含资源内容：它只说明**哪个**资源以何种方式发生变化，不说明资源当前具体状态。执行操作前，应通过 `ctx.agents` 或 `ctx.projects` 重新读取代理或项目。`id` 是不透明字符串，不是数字。

事件消费者需要遵守以下投递规则：

- **每个资源一次只投递一条事件，创建事件优先。** 对每个资源，只会看到最早的一条未确认事件，且 `create` 始终排在其他事件之前。不同资源的事件按 `id` 顺序返回。如果先确认同一资源较晚的事件，调用会报错：“Acknowledge earlier lifecycle events for this resource first”。
- **至少投递一次。** 事件会一直重复投递，直到确认。因此，在外部创建资源时应使用按公司隔离的幂等键，例如公司 ID 加事件 `id`。
- **每轮轮询都从头分页。** 每轮扫描开始时不传 `afterId`，只在该轮内部用它分页。这样失败事件和延迟提交的事件都能在后续轮次重新读到。
- **确认记录按插件隔离。** 两个插件读取同一公司时，各自都会收到每条事件的独立副本。

此功能首次发布时，Paperclip 会一次性写入基线事件，确保插件不会漏掉已有资源：为每个现存项目和每个未处于待审批或已终止状态的代理写入 `create`，然后为当前处于相应状态的代理和项目写入 `pause`、`terminate` 或 `archive`。之后安装的插件也能看到同一批积压事件，因此首次扫描就能补齐整家公司现有资源。

定时作业很适合扫描收件箱：

```ts
ctx.jobs.register("sync-lifecycle", async () => {
  for (const company of await ctx.companies.list()) {
    let afterId: string | undefined;
    for (;;) {
      const events = await ctx.events.listLifecycle(company.id, 100, afterId);
      if (events.length === 0) break;
      for (const event of events) {
        try {
          await provisionOrTearDown(event); // your provider call, keyed on event.id
          await ctx.events.acknowledgeLifecycle(company.id, event.id);
        } catch (err) {
          ctx.logger.warn("lifecycle event failed; will retry next sweep", { id: event.id });
        }
      }
      afterId = events[events.length - 1].id;
    }
  }
});
```

### 清单类型

插件清单会根据从 `@paperclipai/shared` 重新导出的类型进行校验。从 SDK 导入这些类型即可避免额外依赖。

顶层清单字段 `agentLifecycle: true` 会将插件设为[必需的生命周期参与者](#必需的代理生命周期工作)（需要 `agents.lifecycle.manage`）。

| 类型 | 声明内容 |
|---|---|
| `PaperclipPluginManifestV1` | 顶层清单结构。 |
| `PluginJobDeclaration` | 定时／触发式作业。 |
| `PluginWebhookDeclaration` | 入站 webhook 端点。 |
| `PluginToolDeclaration` | 向代理公开的工具。 |
| `PluginEnvironmentDriverDeclaration` | 环境／沙箱驱动程序。 |
| `PluginManagedAgentDeclaration`（及 `PluginManagedAgentResolution`） | 插件管理的代理。 |
| `PluginManagedProjectDeclaration`（及 `PluginManagedProjectResolution`） | 插件管理的项目。 |
| `PluginManagedRoutineDeclaration`（及 `PluginManagedRoutineResolution`） | 插件管理的例程。 |
| `PluginManagedSkillDeclaration`（及 `PluginManagedSkillFileDeclaration`、`PluginManagedSkillResolution`） | 插件管理的公司技能。 |
| `PluginUiDeclaration`（及 `PluginUiSlotDeclaration`） | UI 界面。 |
| `PluginLauncherDeclaration`（及 `PluginLauncherActionDeclaration`、`PluginLauncherRenderDeclaration`） | 启动器位置和行为。 |
| `PluginDatabaseDeclaration` | 托管 Postgres 命名空间。 |
| `PluginApiRouteDeclaration`（及 `PluginApiRouteCompanyResolution`） | 插件挂载的 REST 路由。 |
| `PluginLocalFolderDeclaration` | 通过 `PluginLocalFoldersClient` 展示的本地文件夹挂载。 |
| `PluginObjectReferenceProviderDeclaration`（及 `PluginObjectReferenceRefreshPolicy`） | 外部对象引用提供方；详见[外部对象引用提供方](#外部对象引用提供方)。 |
| `PluginMinimumHostVersion` | 所需的主机版本范围。 |
| `PluginCompanySettings`、`PluginRecord`、`PluginDatabaseNamespaceRecord`、`PluginMigrationRecord`、`PluginConfig`、`CompanySkill`、`PluginManagedResourceKind`、`PluginManagedResourceRef` | 持久化记录和共享基础类型。 |

常量枚举类型：`PluginStatus`、`PluginCategory`、`PluginCapability`、`PluginUiSlotType`、`PluginUiSlotEntityType`、`PluginLauncherPlacementZone`、`PluginLauncherAction`、`PluginLauncherBounds`、`PluginLauncherRenderEnvironment`、`PluginStateScopeKind`、`PluginJobStatus`、`PluginJobRunStatus`、`PluginJobRunTrigger`、`PluginWebhookDeliveryStatus`、`PluginDatabaseCoreReadTable`、`PluginDatabaseMigrationStatus`、`PluginDatabaseNamespaceMode`、`PluginDatabaseNamespaceStatus`、`PluginApiRouteAuthMode`、`PluginApiRouteCheckoutPolicy`、`PluginApiRouteMethod`、`PluginEventType`、`PluginBridgeErrorCode`、`JsonSchema`。

### 托管资源

“托管资源”是指由插件拥有、由主机按公司实例化的 Paperclip 记录的统称，包括托管的**代理**、**项目**、**例程**和**技能**。你只需在清单顶层的 `agents[]`、`projects[]`、`routines[]` 和 `skills[]` 中声明一次；运行时，主机会针对当前 `companyId` 创建资源、重新关联资源，或返回已存在的记录。

当插件需要操作者能在看板上看到的持久业务对象时，适合使用托管资源，例如具名 worker、用于存放插件生成问题的稳定项目空间、会生成可见任务记录的定期例程，或展示在托管代理上的可复用技能。对于不需要在看板上显示任务记录的插件运行时维护工作，请继续使用 `jobs[]`。

每种资源都需要各自的能力（`agents.managed`、`projects.managed`、`routines.managed`、`skills.managed`），并通过 `PluginContext` 上专用的客户端访问：

```ts
await ctx.projects.managed.reconcile("research", companyId);
await ctx.agents.managed.reconcile("researcher", companyId);
await ctx.routines.managed.reconcile("weekly-brief", companyId);
await ctx.skills.managed.reconcile("weekly-brief-skills", companyId);
```

相关方法包括 `get()`、`reconcile()` 和 `reset()`；例程还提供 `update()` 和 `run()`。`reconcile()` 会创建缺失资源、重新关联可恢复的绑定，或返回现有资源。操作者想恢复插件建议的配置时，`reset()` 会重新应用清单默认值。

托管资源之间的依赖通过 `PluginManagedResourceRef` 声明，例如例程的 `assigneeRef` 和 `projectRef`。协调例程前，应先协调其引用的代理和项目；如果引用仍然缺失，例程解析结果会报告 `missing_refs`，而不会自行猜测。

键决定稳定身份。发布后重命名 `agentKey`、`projectKey`、`routineKey` 或 `skillKey`，在主机看来都会创建一个新的托管资源。

完整清单示例和编写规则请参阅上级目录中的 `doc/plugins/PLUGIN_AUTHORING_GUIDE.md`；上文[清单类型](#清单类型)列出的声明类型，是各托管条目可接受字段的权威定义。

### 响应交互和审批

如果插件将 Paperclip 接入人们日常使用的聊天应用、工单工具等服务，关键环节是把处理结果再传回 Paperclip：代理提出问题或请求确认，有人在看到请求的地方作出答复，然后将决定写回看板，效果应与对方直接在 Paperclip 中点击按钮完全相同。

`ctx.issues` 和 `ctx.approvals` 可用于完成这类往返操作，也能读取过程中附带的文件。所有功能都受能力控制；写入操作还要求注明该决定所属的人类用户。

注意类型的导入位置：这些方法返回的领域类型（`IssueThreadInteraction`、`Approval`、`IssueAttachment`）来自 `@paperclipai/shared`，不在 SDK 自身的导出列表中。请从该包导入。

#### 读取并处理问题讨论串交互

交互是代理发布到问题讨论串中的决策卡片，例如建议任务、问题和确认请求。插件原本就可以创建交互，现在也可以读取并处理它们。

通过 `ctx.issues.createInteraction` 创建交互时，载荷类型为 SDK 导出的 `CreateIssueThreadInteractionInput`；`ctx.issues.askUserQuestions` 使用相同的 `ask_user_questions` 结构，但不含 `kind`。这是主机验证的输入结构，因此无需自行构造完整的存储形式，可省略由主机填入默认值的字段。

| 方法 | 签名 | 所需能力 |
|---|---|---|
| `ctx.issues.listInteractions` | `listInteractions(issueId, companyId)` → `Promise<IssueThreadInteraction[]>` | `issue.interactions.read` |
| `ctx.issues.respondInteraction` | `respondInteraction(issueId, interactionId, input, companyId)` → `Promise<{ interaction: IssueThreadInteraction; applied: boolean }>` | `issue.interactions.respond` |

`input` 的结构为 `{ action: "accept" | "reject"; actorUserId?: string; reason?: string | null }`。

```ts
const { interaction, applied } = await ctx.issues.respondInteraction(
  issueId,
  interactionId,
  { action: "accept", actorUserId, reason: null },
  companyId,
);

if (!applied) {
  ctx.logger.info("Interaction was already resolved", { status: interaction.status });
}
```

实现时应考虑三点：

- **类型中 `actorUserId` 可选，但实际调用必须提供。** 处理交互属于看板用户操作，因此缺少此字段时主机会拒绝调用。主机也不会盲目信任传入值：执行时会重新验证该用户是否为问题所属公司的有效人类成员，并拒绝 `membershipRole` 为 `viewer` 的成员，因为 Web 应用在这些路由上同样将 viewer 视为只读。插件只能代表在 Paperclip 中本来就有权执行同一操作的身份行事。
- **`applied` 表示本次调用是否实际执行了操作。** 若本次调用完成交互处理则为 `true`；若交互此前已处于已处理状态则为 `false`。因此，通过聊天转发的重复按钮点击会安全地成为空操作，而不是报错；同时仍会返回已处理的交互，供你呈现其最终状态。
- **接受交互可能唤醒代理。** 已处理交互的 `continuationPolicy` 为 `wake_assignee` 时，主机会唤醒问题负责人；如果策略是 `wake_assignee_on_accept` 且决定为接受，也会唤醒代理，使其恢复工作，效果与在 Web 应用中作出决定相同。

#### 作出审批决定

`ctx.approvals` 是用于公司审批的新客户端。

| 方法 | 签名 | 所需能力 |
|---|---|---|
| `ctx.approvals.list` | `list({ companyId, status? })` → `Promise<Approval[]>` | `approvals.read` |
| `ctx.approvals.get` | `get(approvalId, companyId)` → `Promise<Approval \| null>` | `approvals.read` |
| `ctx.approvals.decide` | `decide(approvalId, input, companyId)` → `Promise<{ approval: Approval; applied: boolean }>` | `approvals.respond` |

`decide` 的 `input` 结构为 `{ action: "approve" | "reject"; actorUserId?: string; decisionNote?: string | null }`。`actorUserId` 的规则和 `applied` 的含义与上文 `respondInteraction` 相同。

还需了解以下两点：

- **返回载荷会经过脱敏。** `list` 和 `get` 返回的审批，其 `payload` 已由主机脱敏，并与 Paperclip 自己的审批读取界面保持一致；桥接层不会向插件泄露 Web 应用对审批读取者隐藏的密钥。
- **新作出的决定会唤醒请求者。** 当 `applied` 为 `true` 且审批有 `requestedByAgentId` 时，主机会唤醒该代理，使其在决定作出后恢复工作。

#### 读取附件

| 方法 | 签名 | 所需能力 |
|---|---|---|
| `ctx.issues.listAttachments` | `listAttachments(issueId, companyId)` → `Promise<IssueAttachment[]>` | `issue.attachments.read` |
| `ctx.issues.getAttachmentContent` | `getAttachmentContent(attachmentId, companyId, options?)` → `Promise<PluginIssueAttachmentContent \| null>` | `issue.attachments.read` |

`options` 结构为 `{ maxBytes?: number | null }`。`PluginIssueAttachmentContent` 在 SDK 的 `types.ts` 中定义，但目前没有以具名类型重新导出；可通过 `getAttachmentContent` 的返回类型使用。它包含以下字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| `attachmentId` | `string` | 附件 ID。 |
| `contentType` | `string` | 存储的 MIME 类型。 |
| `byteSize` | `number` | 实际读取的字节数。 |
| `sha256` | `string` | 已存储对象的内容哈希。 |
| `originalFilename` | `string \| null` | 上传时记录的原始文件名；未记录时为 `null`。 |
| `contentBase64` | `string` | 使用 Base64 编码的原始字节。 |

```ts
const attachments = await ctx.issues.listAttachments(issueId, companyId);

for (const attachment of attachments) {
  const content = await ctx.issues.getAttachmentContent(attachment.id, companyId, {
    maxBytes: 5 * 1024 * 1024,
  });
  if (!content) continue;

  ctx.logger.info("Read attachment", {
    originalFilename: content.originalFilename,
    contentType: content.contentType,
    byteSize: content.byteSize,
  });
}
```

`getAttachmentContent` 返回的是**字节，不是 URL**：读取经由受能力限制的主机桥接完成，结果本身不提供 URL。由此需要注意：

- **未知 ID 和跨公司 ID 无法区分。** 按设计，两者都会返回 `null`，因此桥接层不会成为跨公司探测资源是否存在的工具。
- **`maxBytes` 超限时会拒绝，而不是截断。** 如果存储大小超过上限，主机会抛出异常；如果读取过程中发现对象超过上限，也会再次抛出异常。你不会收到一个看似完整的部分文件。
- **读取操作会写入审计日志。** 每次成功读取内容，都会在主机活动日志中记录到相应问题下。

#### 五个新增能力字符串

在清单的 `capabilities` 中声明以下能力；否则主机能力校验器会拒绝调用这些方法的插件：

| 能力 | 授予权限 |
|---|---|
| `issue.interactions.read` | `ctx.issues.listInteractions` |
| `issue.interactions.respond` | `ctx.issues.respondInteraction` |
| `issue.attachments.read` | `ctx.issues.listAttachments` 和 `ctx.issues.getAttachmentContent` |
| `approvals.read` | `ctx.approvals.list` 和 `ctx.approvals.get` |
| `approvals.respond` | `ctx.approvals.decide` |

这五项都属于 `PLUGIN_CAPABILITIES`。如果需要在运行时检查能力值，SDK 会重新导出该常量。

### 访问与授权

两个新的上下文客户端允许插件读取和管理公司成员身份及其权限，与 Web 应用公开的成员、邀请、权限授权和策略界面相同。两者都受能力控制；执行写入时，主机会像 SDK 中其他写操作一样重新验证操作身份，而不会只相信插件提供的信息。

#### 使用 `ctx.access` 管理成员和邀请

`ctx.access` 是包含两个子客户端的 `PluginAccessClient`：

- `ctx.access.members`（`PluginAccessMembersClient`）提供 `list({ companyId, includeArchived? })`、`get(memberId, companyId)` 和 `update(memberId, patch, companyId)`。每条记录为 `PluginAccessMember`，包含 `id`、`companyId`、`principalType`（`PrincipalType`）、`principalId`、`status`（`MembershipStatus`）、`membershipRole` 和 `grants`（`PrincipalPermissionGrant[]`）。`update` 的 patch 只允许修改 `membershipRole`（`HumanCompanyMembershipRole | null`）和 `status`（`"pending" | "active" | "suspended"`）。读取需要 `access.members.read`，更新需要 `access.members.write`。
- `ctx.access.invites`（`PluginAccessInvitesClient`）提供 `list({ companyId, state?, limit?, offset? })`（返回 `{ invites, nextOffset }`）、`create({ companyId, allowedJoinTypes?, humanRole?, defaultsPayload?, agentMessage? })` 和 `revoke(inviteId, companyId)`。每个邀请都是 `PluginAccessInvite`，其 `state` 为 `"active" | "revoked" | "accepted" | "expired"` 之一；`create` 会返回邀请及一次性 `token`。`list` 需要 `access.invites.read`；`create`／`revoke` 需要 `access.invites.write`。

#### 使用 `ctx.authorization` 管理授权、策略和审计

`ctx.authorization` 是一个 `PluginAuthorizationClient`，按 `grants`、`policies` 和 `audit` 分组：

- `grants.list(...)` 和 `grants.set(...)` 可读取并替换主体的 `PrincipalPermissionGrant[]`，以 `permissionKey`（`PermissionKey`）和可选 `scope` 为键。分别需要 `authorization.grants.read` 和 `authorization.grants.write`。
- `policies.summary(companyId)` 返回 `PluginAuthorizationPolicySummary`（成员数和授权数，`permissionsMode: "simple"`）。`policies.get(...)`／`policies.update(...)` 读取或写入 `PluginAuthorizationPolicyRecord`，其 `resourceType` 为 `"company" | "agent" | "project" | "issue"` 之一。`policies.previewAssignment(...)` 和 `policies.explainAssignment(...)` 接收 `PluginAssignmentPreviewInput`，并返回 `PluginAuthorizationDecisionResult`（包含 `allowed`、`action`、`explanation`、`reason` 和匹配到的 `grant`）。需要 `authorization.policies.read`／`authorization.policies.write`。
- `audit.search(...)` 返回 `PluginAuthorizationAuditEntry[]`。需要 `authorization.audit.read`。

这些方法使用的领域结构（`PrincipalType`、`MembershipStatus`、`HumanCompanyMembershipRole`、`PermissionKey`、`PrincipalPermissionGrant`）均由 SDK 重新导出，可与其他类型一起从 `@paperclipai/plugin-sdk` 导入。对应的运行时常量数组（`PRINCIPAL_TYPES`、`MEMBERSHIP_STATUSES`、`HUMAN_COMPANY_MEMBERSHIP_ROLES`、`HUMAN_COMPANY_MEMBERSHIP_ROLE_LABELS`、`PERMISSION_KEYS`）也会重新导出；详见[重新导出的内容](#重新导出的内容)。

### 操作处理程序上下文

`ctx.actions.register(key, handler)` 现在会向处理程序传入不可变的第二个参数 `context: PluginPerformActionContext`。该上下文由主机解析，而不是取自调用方参数。它包含 `actor`（`Readonly<PluginPerformActionActorContext>`，其字段类型为 `type: PluginPerformActionActorType`，取值 `"user" | "agent" | "system"`，并带有 `userId`、`agentId`、`runId` 和 `companyId`），以及作为 `actor.companyId` 便捷别名的 `companyId`。请从此上下文读取操作身份；不要信任操作自身 `params` 中传入的 actor ID。

### 外部对象引用提供方

**外部对象引用提供方**可教会 Paperclip 识别指向其他系统工作的 URL（例如 GitHub PR 或 Linear 问题），并在 Paperclip 问题旁保留带状态的对象引用。操作者将受支持的 URL 粘贴到问题内容后，主机会识别 URL、请求插件解析远端当前状态，并按计划刷新该引用。这样，在问题的各个界面中会显示实时且带状态的标记，而不是普通链接。

在清单顶层的 `objectReferences[]` 中声明提供方，并在传给 `definePlugin({...})` 的对象中实现可选生命周期处理程序。声明 `objectReferences` 需要 `external.objects.detect` 和 `external.objects.read` 两项能力；批量刷新处理程序还需要 `external.objects.refresh`。

#### 声明提供方

每个条目都是一个 `PluginObjectReferenceProviderDeclaration`：

| 字段 | 类型 | 说明 |
|---|---|---|
| `providerKey` | `string` | 稳定的提供方键，例如 `"github"`、`"linear"` 或 `"mocktracker"`。 |
| `displayName` | `string` | 在面向操作者的界面中显示的可读名称。 |
| `objectTypes` | `string[]` | 此插件可识别和解析的提供方对象类型。 |
| `urlPatterns?` | `string[]` | 此提供方可识别的可读 URL 模式。这些是供操作者和文档查看的元数据；实际识别仍由 worker 执行。 |
| `refreshPolicy?` | `PluginObjectReferenceRefreshPolicy` | 此提供方可选的默认刷新策略。 |
| `webhookEndpointKeys?` | `string[]` | 可用于刷新这些对象、并在 `webhooks` 中声明的可选 webhook 端点键。每个键都必须与已声明的 `PluginWebhookDeclaration` 端点匹配。 |

`PluginObjectReferenceRefreshPolicy` 用于控制已解析对象被视为新鲜状态的时长：

| 字段 | 类型 | 说明 |
|---|---|---|
| `defaultTtlSeconds?` | `number` | 此提供方已解析对象的默认新鲜度时长。 |
| `staleAfterSeconds?` | `number` | UI 显示对象过时所用的时长。核心仍会将存活状态与远端状态分开存储。 |

#### 识别 → 解析 → 刷新的生命周期

在插件定义中实现三个可选钩子来完成生命周期。每个钩子都受各自能力控制。

`onDetectExternalObjects(params)`：Paperclip 扫描问题、评论或文档内容时，会调用此钩子，询问经过清理的 URL 候选项是否属于你的提供方。除非已显式计算提供方安全的身份组件哈希，否则主机已去除 URL 中的用户信息、查询字符串和片段。需要 `external.objects.detect`。

- 接收 `DetectExternalObjectsParams`：包含 `companyId`、`PluginExternalObjectUrlCandidate` 数组（字段为 `sanitizedCanonicalUrl`、`sanitizedDisplayUrl`、`canonicalIdentityHash`、`canonicalIdentity`、`redactedMatchedText`），以及 `PluginExternalObjectSourceContext`（字段为 `companyId`、`sourceIssueId`、`sourceKind`、`sourceRecordId`、`documentKey`、`propertyKey`）。
- 返回 `DetectExternalObjectsResult`：结构为 `{ detections }`。每个 `PluginExternalObjectDetection` 包含 `urlIdentityHash`、`providerKey`、`objectType`、`externalId`，以及可选的 `displayKey`、`iconKey`、`displayTitle` 和 `confidence`。

`onResolveExternalObject(params)`：当 Paperclip 需要获取由已声明提供方管理的某个外部对象当前规范化状态时，会调用此钩子。需要 `external.objects.read`。

- 接收 `ResolveExternalObjectParams`：包含 `companyId`、`providerKey`、`objectType`、`externalId`，以及当前 `object`（类型为 `PluginExternalObjectRecordSnapshot`，即已持久化的行，字段包括 `id`、`companyId`、`providerKey`、`objectType`、`externalId`、`sanitizedCanonicalUrl`、`canonicalIdentityHash`、`displayKey`、`iconKey`、`displayTitle`、`statusKey`、`statusLabel`、`statusIconKey`、`statusCategory`、`statusTone`、`liveness`、`isTerminal`、`data`、`remoteVersion`、`etag`）。
- 返回判别联合类型 `PluginExternalObjectResolveResult`：
  - `{ ok: true, snapshot }`，其中 `snapshot` 为 `PluginExternalObjectResolvedSnapshot`，包含已刷新的 `statusCategory` 和 `statusTone`，以及可选的 `displayKey`、`iconKey`、`displayTitle`、`statusKey`、`statusLabel`、`statusIconKey`、`isTerminal`、`data`、`remoteVersion`、`etag` 和 `ttlSeconds`。
  - `{ ok: false, liveness, errorCode, errorMessage?, retryAfterSeconds? }`，其中 `liveness` 仅允许为 `"auth_required"` 或 `"unreachable"`。可用它报告令牌已过期或远端无法访问，同时保留对象引用。

`onRefreshExternalObjects(params)`：可选的批量解析器，适用于能比逐一调用 `onResolveExternalObject` 更高效地刷新多个对象的提供方。需要 `external.objects.refresh`。

- 接收 `RefreshExternalObjectsParams`：包含 `companyId` 和 `PluginExternalObjectRecordSnapshot` 类型的 `objects` 数组。
- 返回 `RefreshExternalObjectsResult`：结构为 `{ results }`，数组项为 `{ objectId, result }`，其中 `result` 的结构与单对象解析返回的 `PluginExternalObjectResolveResult` 完全一致。

如果只实现 `onResolveExternalObject`，主机会按照 `refreshPolicy` 规定的时间窗口逐个刷新对象；实现 `onRefreshExternalObjects` 则可将它们合并成一次往返请求。具体实现示例请参阅上级目录中的 `server/src/services/github-external-object-provider.ts`。

<a id="ai-connection-routers"></a>
### AI 连接路由器

插件可以作为[任务固定 AI 路由](../../experimental/task-pinned-ai-routing.md)的路由决策器，决定连接池中下一项任务使用哪个已保存 AI 账户。其他工作（授权、将任务固定到账户、存储连接池，以及设置和管理界面）都由 Paperclip 完成，因此路由器插件只需少量代码。

在清单中同时声明以下两项：

```ts
capabilities: ["ai.connections.route"],
aiConnectionRouter: {
  name: "AI connection pool",            // 1–100 characters
  description: "Use existing AI connections.", // 1–500 characters
},
```

如果没有 `ai.connections.route` 能力，清单校验器会拒绝 `aiConnectionRouter`。Paperclip 会在 **Connectors** 目录中列出你的路由器，并提供 **Add connection pool** 操作；无需额外创建插件页面或侧边栏条目。只有实例启用了实验性标记 `enableAiConnectionRouters` 且插件已就绪时，路由器才可用。

然后在插件定义中实现 `onRouteAiConnection`。没有固定账户的任务需要账户时，主机会调用它（对应 worker 方法 `routeAiConnection`）：

```ts
export default definePlugin({
  async setup() {},
  async onRouteAiConnection(request) {
    const next = request.candidates[0];
    return next
      ? { kind: "selected", memberId: next.member.id }
      : { kind: "unavailable", skipped: {} };
  },
});
```

`AiConnectionRouterRequest` 包含 `companyId`、连接池 `pool`（类型为 `AiConnectionPool`）、`agentId`、`taskKey`、`lastMemberId`、`cursorVersion`、连接池的 `memberOrder`、可选的 `pinnedMemberId`、主机已确认该代理有权使用的 `candidates`（每项包含 `member`、`runtimeConfig`、`notes` 以及可选的 `usage`），以及 `now`。其中不含凭据或未经处理的提供方响应。

返回一个 `AiConnectionRouterResult`：

| `kind` | 含义 |
|---|---|
| `selected` | 使用 `memberId`。可选地在 `skipped` 中说明跳过其他成员的原因；该字段是成员 ID 到原因的映射。 |
| `exhausted` | 所有候选账户均超出限制。Paperclip 会将任务延后至 `retryAt`，并显示 **Pool exhausted**。请包含 `skipped`。 |
| `unavailable` | 没有候选账户可执行此任务。请包含 `skipped`。 |

你的返回值只是建议。主机会重新检查授权，不会被迫使用负责人无权使用的账户。SDK 会重新导出相关类型：`AiConnectionPool`、`AiConnectionPoolConfig`、`AiConnectionPoolMember`、`AiConnectionRouterRequest`、`AiConnectionRouterResult` 和 `AiConnectionRouterSelection`。

### JSON-RPC 协议

SDK 使用 JSON-RPC 2.0 在主机和 worker 之间通信。大多数插件作者不会直接调用这些接口，但 SDK 仍会导出它们，供高级用途（自定义传输、测试、重放工具）使用。

辅助函数和常量：

- `JSONRPC_VERSION`、`MESSAGE_DELIMITER`
- `JSONRPC_ERROR_CODES`、`PLUGIN_RPC_ERROR_CODES`
- `HOST_TO_WORKER_REQUIRED_METHODS`、`HOST_TO_WORKER_OPTIONAL_METHODS`
- `createRequest`、`createSuccessResponse`、`createErrorResponse`、`createNotification`
- `isJsonRpcRequest`、`isJsonRpcNotification`、`isJsonRpcResponse`、`isJsonRpcSuccessResponse`、`isJsonRpcErrorResponse`
- `serializeMessage`、`parseMessage`
- `JsonRpcParseError`、`JsonRpcCallError`
- `LOGIN_PTY_OUTPUT_NOTIFICATION`、`LOGIN_PTY_EXIT_NOTIFICATION`、`DUPLEX_CHANNEL_DATA_NOTIFICATION`、`DUPLEX_CHANNEL_EXIT_NOTIFICATION`：登录 PTY 和双工通道流的 worker 到主机通知方法名称；详见[登录 PTY 与双工通道流（可选）](#登录-pty-与双工通道流可选)。
- `encodeChannelBytes`、`decodeChannelBytes`：将双工通道原始 `Uint8Array` 字节转换为线路协议使用的 Base64 格式，或反向转换。

`PLUGIN_RPC_ERROR_CODES` 包含 `CROSS_TENANT_CONFIG`。当 `configChanged` 投递会导致单租户 worker 切换到第二家公司的配置时，worker 会抛出此错误码；详见[使用 `multiCompanyConfig` 声明多公司支持](#使用-multicompanyconfig-声明多公司支持)。

它还包含 `INVOCATION_SCOPE_DENIED`，当 worker 到主机的调用请求访问当前顶层调用未授权公司的公司范围数据时就会抛出。该作用域通过线路协议中的 `PluginInvocationContext` 传递（其 `scope` 是只包含一个 `companyId` 的 `PluginInvocationScope`），由主机针对每次调用签发；worker 将其视为不透明数据，在嵌套的 worker 到主机调用中只回传其 `id`，不会自行创建或修改它。主机端处理程序会通过 `WorkerHostCallContext` 接收回传的作用域。对应的 JSON-RPC 信封结构为 `JsonRpcInvocationScope` 和 `JsonRpcInvocationContext`；调用超出作用域时，主机客户端工厂会抛出 [`InvocationScopeDeniedError`](#主机客户端工厂)。

worker 到主机的方法表 `WorkerToHostMethods` 新增了支撑上述客户端的调用：`issues.listInteractions`、`issues.respondInteraction`、`issues.listAttachments`、`issues.getAttachmentContent`、`approvals.list`、`approvals.get` 和 `approvals.decide`。

协议类型：`JsonRpcId`、`JsonRpcInvocationScope`、`JsonRpcInvocationContext`、`JsonRpcRequest`、`JsonRpcSuccessResponse`、`JsonRpcError`、`JsonRpcErrorResponse`、`JsonRpcResponse`、`JsonRpcNotification`、`JsonRpcMessage`、`JsonRpcErrorCode`、`PluginRpcErrorCode`；作用域结构 `PluginInvocationScope`、`PluginInvocationContext` 和 `WorkerHostCallContext`；各 RPC 方法的参数结构 `InitializeParams`、`InitializeResult`、`ConfigChangedParams`、`ValidateConfigParams`、`OnEventParams`、`RunJobParams`、`GetDataParams`、`PerformActionParams`、`PluginPerformActionActorType`、`PluginPerformActionActorContext`、`PluginPerformActionContext`、`ExecuteToolParams`；以及主机方法表 `HostToWorkerMethods`／`HostToWorkerMethodName`／`WorkerToHostMethods`／`WorkerToHostMethodName`／`HostToWorkerRequest`／`HostToWorkerResponse`／`WorkerToHostRequest`／`WorkerToHostResponse`／`WorkerToHostNotifications`／`WorkerToHostNotificationName`。

外部对象协议结构：`PluginExternalObjectUrlCandidate`、`PluginExternalObjectSourceContext`、`DetectExternalObjectsParams`、`PluginExternalObjectDetection`、`DetectExternalObjectsResult`、`PluginExternalObjectRecordSnapshot`、`ResolveExternalObjectParams`、`PluginExternalObjectResolvedSnapshot`、`PluginExternalObjectResolveResult`、`RefreshExternalObjectsParams`、`RefreshExternalObjectsResult`。使用这些类型的生命周期请参阅[外部对象引用提供方](#外部对象引用提供方)。

环境驱动程序协议结构：`PluginEnvironmentDiagnostic`、`PluginEnvironmentDriverBaseParams`、`PluginEnvironmentValidateConfigParams`、`PluginEnvironmentValidationResult`、`PluginEnvironmentProbeParams`、`PluginEnvironmentProbeResult`、`PluginEnvironmentLease`、`PluginEnvironmentAcquireLeaseParams`、`PluginEnvironmentResumeLeaseParams`、`PluginEnvironmentReleaseLeaseParams`、`PluginEnvironmentDestroyLeaseParams`、`PluginEnvironmentRealizeWorkspaceParams`、`PluginEnvironmentRealizeWorkspaceResult`、`PluginEnvironmentExecuteParams`、`PluginEnvironmentExecuteResult`、`PluginEnvironmentRunnerIngressEndpointParams`、`PluginEnvironmentRunnerIngressEndpoint`、`PluginSyncFileMapping`、`PluginPostUploadCommand`、`PluginSyncOperation`、`PluginEnvironmentSyncInParams`、`PluginEnvironmentSyncOutParams`、`PluginEnvironmentSyncResult`、`PluginEnvironmentInteractiveSetupStatus`、`PluginEnvironmentInteractiveSetupConnectionType`、`PluginEnvironmentTemplateRefKind`、`PluginEnvironmentInteractiveSetupConnectionSummary`、`PluginEnvironmentInteractiveSetupConnectionPayload`、`PluginEnvironmentInteractiveSetupSession`、`PluginEnvironmentStartInteractiveSetupParams`、`PluginEnvironmentGetInteractiveSetupParams`、`PluginEnvironmentCaptureTemplateParams`、`PluginEnvironmentCaptureTemplateResult`、`PluginEnvironmentCancelInteractiveSetupParams`、`PluginEnvironmentCancelInteractiveSetupResult`、`PluginEnvironmentDeleteTemplateParams`、`PluginEnvironmentDeleteTemplateResult`、`PluginEnvironmentTemplateConfigBinding`。`PluginSync*` 和 `PluginEnvironmentSync*` 结构用于下文介绍的可选沙箱文件同步钩子；交互式设置和模板捕获结构则用于相应的设置钩子。

#### 流式传输实时命令输出

驱动程序通过 `execute` 运行长时间命令时，无需等命令结束后才显示输出。`execute` 调用仍在运行期间，每当命令产生新的输出片段，就调用 `ctx.execution.log(stream, chunk)`；主机会在片段到达时立即转发给该调用的日志回调，无需等到 `execute` 返回最终结果。主机会根据消息信封中由主机签发的调用 ID，将每个片段关联到当前运行的 `execute` 调用。因此，驱动程序只需在片段到达时上报，匹配工作由主机完成。

该客户端只有一个方法：`log(stream: "stdout" | "stderr", chunk: string): void`。`stream` 传入 `"stdout"` 或 `"stderr"`，`chunk` 传入新输出文本（字符串，而非原始字节）。在读取命令输出的循环中调用此方法是安全的：

- **默认实现是永不抛错的空操作。** 不支持流式输出的驱动程序保持原有行为；无需增加保护逻辑。没有可接收输出的目标时，`log` 不执行任何操作。
- **主机会丢弃格式错误的内容。** `stream` 不是严格等于 `stdout` 或 `stderr` 时会被丢弃；空 `chunk` 或过大的 `chunk` 也会被丢弃。无效输入不会进入日志回调，也不会反向导致调用抛错。

因此，支持实时输出的提供方只需在运行命令每次读取到新片段时调用 `ctx.execution.log("stdout", chunk)`（或 `"stderr"`）；不支持流式输出的提供方可完全忽略该客户端。

#### 创建失败后的清理

有些沙箱提供方会先分配机器，再等待机器启动。如果等待失败，提供方自己的 SDK 可能在返回资源句柄前就抛出异常，导致沙箱仍在运行却无人知晓。插件 SDK 为驱动程序提供了一种方式，即使发生重启，也能向主机提供足够证据以清理这项分配。

如果 `environmentAcquireLease`（或 `environmentDestroyLease`）在资源可能已经分配后失败，请抛出 `PluginEnvironmentCreationCleanupError`，而不是普通错误。其构造函数接收底层错误、错误消息，以及说明本次尝试创建内容的 `PluginEnvironmentCreationCleanup` 记录：

- `providerLeaseId`：创建资源时使用的名称或 ID；如果观察到所有权匹配的真实提供方 ID，则还包括 `observedProviderLeaseId`。
- `companyId`、`environmentId`、可选的 `runId`，以及本次创建尝试唯一的 `attemptId`。
- `accountFingerprint`：提供方账户的 64 字符小写十六进制指纹。
- `labels`：为资源加上的所有权标签，最多 16 项，每项键都必须以 `paperclip-` 开头。

worker 只会将经过验证的记录发回主机，不会发送提供方异常本身（异常可能含凭据）。主机会将记录存为待清理项，并重试拆除所有权匹配的那一项精确资源，因此不会删除其他尝试创建的资源。未通过验证的记录会直接丢弃。无论是否提供清理信息，资源获取都会失败；清理错误不会产生租约。

两个辅助函数用于处理此信息：`readEnvironmentCreationCleanupError(error)` 从错误中读取经过验证的 `PluginEnvironmentCreationCleanup`，若无效则返回 `null`；`environmentCreationCleanupErrorData(error)` 构造 worker 附加到 JSON-RPC 错误中的线路载荷。如果驱动程序不可能留下此类未清理分配，则可忽略这整套功能。

为了让操作者了解资源获取失败的**阶段**，可向构造函数传入可选的第四个参数 `PluginEnvironmentAcquisitionDiagnostic`。该参数包含最先失败的 `phase`（`create`、`workspace`、`shell`、`expiry` 或 `sentinel`）、从开始获取到失败的 `elapsedMs`、当时适用的 `budgetMs`，以及可选的 `providerState`（在创建截止时间后观察到的状态，只能是 `ENVIRONMENT_ACQUISITION_PROVIDER_STATES` 中的值：`absent`、`creating`、`pending_build`、`building_snapshot`、`pulling_snapshot`、`starting`、`started`、`error`、`build_failed`、`other`、`unavailable`）。主机只在获取失败时保留此信息，通过 `readEnvironmentAcquisitionDiagnostic(error)` 重新验证；未知的 `providerState` 会被丢弃，但不会连带丢弃整条记录。此诊断只描述失败步骤，不说明原因，也不会授权清理。

#### 停止但不删除沙箱（可选）

有时主机需要停止沙箱并保留其中的文件，例如从异常结束的运行中恢复工作区导出，或可复用沙箱完成一个工作轮次时。普通的释放钩子无法保证这一点，因为它遵循环境释放策略，而临时沙箱会在释放时被删除。

实现 `onEnvironmentStopLease(params: PluginEnvironmentReleaseLeaseParams): Promise<PluginEnvironmentTerminationReceipt>` 即可支持此行为。只有定义此钩子时，worker 才会公布对应的 `environmentStopLease` RPC 方法；这样主机能够识别旧版驱动，并在具备能力的驱动可用前暂缓清理。主机调用时，`params.resourceDisposition` 为 `"stop_and_retain"`。

以下两条规则可确保操作安全：

- **无论释放策略如何，都要停止那一项精确分配。** 返回 `PluginEnvironmentTerminationReceipt`，其 `state: "stopped"`，并包含已停止资源的 `providerLeaseId`。
- **无法确认停止时就抛出异常。** 绝不能退回到删除沙箱。停止失败会让清理保持待处理状态，主机会稍后重试，不会改用释放或销毁操作。

如果提供方无法在不删除沙箱的情况下停止资源，就不要实现此钩子。这样主机便不会要求驱动保留沙箱。

#### 在持久会话之外运行命令

`PluginEnvironmentExecuteParams` 包含可选字段 `bypassSession?: boolean`。只有驱动程序会打开**持久会话**（一个跨租约命令保持活动的 shell 或连接）时，该字段才有意义。

运行中的代理开始工作前，主机会为必须预先运行的命令设置 `bypassSession: true`；典型情况是工作区配置命令。看到此标记时，应以一次性方式运行命令，并保持持久会话关闭：不要为了它打开会话。这样，持久会话会在运行中第一个真实命令到达时自然开启，预配置过程也就不会泄漏到代理实际工作的会话中。

标记缺失或为 `false` 时，行为不变：普通的运行中命令会像以前一样打开并复用持久会话。完全不保留持久会话的驱动程序可忽略 `bypassSession`；它没有需要绕开的会话。

`PluginEnvironmentExecuteParams` 还可包含 `commandTimeoutMs`，即整条命令的总时间预算（毫秒）。如果驱动在会话中运行命令并轮询输出，应将此预算与每次轮询的超时分开处理。`0` 表示禁用截止时间；未提供该字段时，命令沿用会话已有的生命周期。

#### 沙箱文件同步（可选）

默认情况下，主机会使用 Base64 编码的 `environmentExecute` 回退方案，在租用的沙箱和主机之间传输逐字节一致的文件。如果驱动能够提供更好的方式（例如提供方原生批量上传、内部 tar 流或逐文件枚举），可在传给 `definePlugin({...})` 的对象中实现一对匹配的钩子，接管文件传输：

- `onEnvironmentSyncIn(params: PluginEnvironmentSyncInParams): Promise<PluginEnvironmentSyncResult>`：执行前调用，将主机文件和目录放到沙箱中的目标路径。
- `onEnvironmentSyncOut(params: PluginEnvironmentSyncOutParams): Promise<PluginEnvironmentSyncResult>`：执行后调用，将沙箱文件和目录复制回主机上的目标路径。

这两个钩子均为可选且需显式启用，但必须成对实现：**两个都要定义**，worker 才会公布 `environmentSyncIn`／`environmentSyncOut` 方法，主机也才会通过你的驱动传输文件。若不定义，仍使用 Base64 回退方案；只负责租用和执行的驱动可以完全忽略这些钩子。

每个参数对象都包含当前的 `PluginEnvironmentLease`，以及一个有序的 `PluginSyncOperation` 数组。操作按数组顺序执行；每个操作包含不透明且不敏感的 `operationId`（由编排器生成，驱动不得解析其含义）和 `PluginSyncFileMapping` 文件映射列表。每条映射描述一次源到目标的传输：

| 字段 | 类型 | 说明 |
|---|---|---|
| `sourcePath` | `string` | 传输源的绝对路径；syncIn 时为主机路径，syncOut 时为沙箱路径。 |
| `targetPath` | `string` | 传输目标的绝对路径；syncIn 时为沙箱路径，syncOut 时为主机路径。沙箱路径采用 POSIX 格式。 |
| `kind` | `"file" \| "directory"` | 映射传输的是单个普通文件还是整个目录树。 |
| `mode?` | `number` | 应用于目标的 POSIX 文件模式（例如密钥材料使用 `0o600`）。设定后，创建目标时就必须应用此模式，并且不能出现任何人可读的时间窗口：应使用指定模式创建，或先 chmod 再写入字节，绝不能写完后才修改权限。 |
| `exclude?` | `string[]` | 当 `kind` 为 `"directory"` 时要跳过的 glob 模式。 |
| `followSymlinks?` | `boolean` | 目录传输时的符号链接处理方式。值为假时保留链接本身；为 `true` 时解引用并传输目标字节（效果与 tar 的 `-h` 一致）。 |

返回 `PluginEnvironmentSyncResult`：其中 `operations` 数组回传每个 `operationId` 及其 `filesTransferred` 和 `bytesTransferred` 计数，供主机观测。

sync-out 失败时，worker 会向主机报告简短且安全的诊断信息，例如已知错误码 `ENOENT` 或 `ETIMEDOUT`、HTTP 状态码或退出码，绝不包含原始错误文本。你可以用导出的辅助函数提供更多诊断信息：使用 `withEnvironmentSyncTransferStep(step, operation)` 包装各阶段（如 `sandbox_access`、`file_download`、`archive_create`、`archive_extract`），以标明失败阶段；调用 `recordEnvironmentSyncError(error, diagnostic)` 附加 `transferFailureKind`（如 `download_missing` 或 `unsafe_archive`）；如果再用自己的错误包装错误，则使用 `preserveEnvironmentSyncErrorDiagnostic(wrapper, source)` 保留已有证据。已知列表之外的值都会被丢弃。诊断只为操作者提供证据，不会授权清理或重试。此契约与提供方无关：只要可观测结果符合映射定义，目录可以用任何方式传输。完整编写规则请参阅上级目录中的 `doc/plugins/SANDBOX_FILE_SYNC_HOOKS.md`。

##### 文件写入后运行命令

有些文件写入后不能立即使用：依赖清单需要安装，归档需要解包，检出的树可能需要修正后才能开始运行。因此，sync 操作现在可以附带一组简短 shell 命令，在该操作的文件全部写入沙箱**之后**执行。

`PluginSyncOperation` 新增了一个可选字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| `postUploadCommands?` | `PluginPostUploadCommand[]` | 在此操作的文件写入后按数组顺序运行的有序控制命令；遇到失败立即中止。 |

未提供该字段表示“没有命令”。不含此字段的操作与加入此契约前逐字节相同，因此驱动即使完全不读取该字段，行为也与之前一致。

每个条目都是从 `@paperclipai/plugin-sdk` 导出的 `PluginPostUploadCommand`：

| 字段 | 类型 | 说明 |
|---|---|---|
| `command` | `string` | 不透明的、由适配器编写的 shell 命令，在上传后运行。提供方必须原样执行。 |
| `cwd?` | `string` | 命令的工作目录。若提供，必须是位于此操作允许的沙箱目标根目录内的 POSIX 绝对路径。 |
| `timeoutMs?` | `number` | 可选的单条命令超时（毫秒）。 |

顺序规则很简单，必须严格遵守：

- **先传文件，再运行命令。** 每个操作中的所有映射都放置完毕后，再执行一次命令列表；不能在每个文件放好后分别执行。
- **按数组顺序执行，遇错即停。** 首个非零退出码或超时都会中止操作。不要跳过失败项继续执行，也不要静默地以部分成功结束。

###### 必须遵守的安全契约

这一部分尤其重要，因为驱动会把 `command` 字符串交给 shell 执行。Paperclip 通过限制命令的来源确保安全，而提供方也必须履行该契约的另一半。

**`command` 是由 Paperclip／适配器编写的控制操作。** 只有核心或适配器代码可以提供它。它绝不会来自服务器路由、问题或评论内容、项目或工作区文件内容、提供方插件回调，或任意适配器配置。命令中出现的路径由适配器／核心辅助函数根据已限制范围的路径生成，并且在到达驱动前就已进行 shell 引号转义。

**将命令视为不透明内容。** 你可以执行或拒绝它，但绝不能改写、拼接它，或在其中附加自己的 shell 片段。具体来说，工作目录必须作为结构化参数传递，不能将 `cd … &&` 前缀粘到命令字符串上。

**执行前重新验证 `cwd`。** 若提供了 `cwd`，必须使用放置文件时相同的沙箱根目录和防护规则，确认其位于根目录内；在执行任何命令前，遇到 `..`、绝对路径逃逸或符号链接逃逸都必须 fail-closed 拒绝。若未提供 `cwd`，默认使用已解析的同步远端／运行时根目录。绝不能回退到进程默认工作目录。

两个原生提供方展示了具体实现方式；若在编写自己的驱动，建议阅读：

- **Daytona**（`packages/plugins/sandbox-providers/daytona/src/file-sync.ts`）会对已提供的 `cwd` 重新运行词法检查 `assertConfinedSandboxPath`，并调用真实路径／符号链接防护 `assertSandboxPathsConfined`；之后执行 `sandbox.process.executeCommand(command.command, cwd, undefined, commandTimeoutSeconds)`。命令字符串原样传入，工作目录作为独立参数传递。
- **Kubernetes**（`packages/plugins/sandbox-providers/kubernetes/src/file-sync.ts`）会先在主机上对 `cwd` 做词法验证，然后在 Pod 中运行固定包装脚本。脚本通过 realpath 解析目录，通过 `/proc/self/fd` 固定目录，再切换到固定的 inode，最后才执行命令。目录和命令均作为位置参数（`$1` 和 `$2`）传入；包装脚本不会把二者插入自身脚本文本，因此命令会逐字节按编写内容执行。若尝试逃逸，会在运行命令前以 `42` 退出。

Kubernetes 与 Daytona 一样会执行这些命令，而不是静默丢弃它们：原生提供方若忽略此字段就会形成 fail-open，这正是该契约旨在防止的结果。

两个原生提供方都只在**传入**时运行这些命令，即在 `performSyncIn` 的每操作循环中、该操作的映射全部写入后运行；sync-out 时都不会运行。自己的驱动也应采用同样的位置。

#### 交互式设置和可复用模板（可选）

环境驱动插件除了租用和执行以外，还可以交互式创建沙箱，让操作者登录并配置妥当，然后将当前沙箱捕获为可复用的“自定义镜像”模板，供后续租用使用。这些钩子都是**可选的**：仅当驱动支持交互式设置或捕获可复用环境模板时才需实现。只提供租用和执行功能的驱动可完全忽略它们。

在传给 `definePlugin({...})` 的对象上实现以下可选方法：

- `onEnvironmentStartInteractiveSetup(params: PluginEnvironmentStartInteractiveSetupParams): Promise<PluginEnvironmentInteractiveSetupSession>`：启动交互式设置沙箱并返回脱敏的连接元数据。
- `onEnvironmentGetInteractiveSetup(params: PluginEnvironmentGetInteractiveSetupParams): Promise<PluginEnvironmentInteractiveSetupSession>`：读取设置状态；获得授权后还可读取一次性连接载荷。
- `onEnvironmentCaptureTemplate(params: PluginEnvironmentCaptureTemplateParams): Promise<PluginEnvironmentCaptureTemplateResult>`：从正在运行的设置沙箱捕获可复用的提供方模板。
- `onEnvironmentCancelInteractiveSetup(params: PluginEnvironmentCancelInteractiveSetupParams): Promise<PluginEnvironmentCancelInteractiveSetupResult>`：取消并清理设置沙箱，不将其提升为模板。
- `onEnvironmentDeleteTemplate(params: PluginEnvironmentDeleteTemplateParams): Promise<PluginEnvironmentDeleteTemplateResult>`：可选的尽力清理操作，用于删除已捕获的提供方模板。

典型流程为：开始 → 获取（轮询状态，获得授权后读取一次性连接载荷）→ 捕获。取消操作可用于退出；之后还可以通过删除模板进行清理。

#### 为 runnerd 提供私有入口（可选）

`onEnvironmentRunnerIngressEndpoint(params: PluginEnvironmentRunnerIngressEndpointParams)` 可让驱动返回一个经过身份验证的私有 WebSocket，供主机上的 runner 连接到租用的沙箱。参数包含当前 `PluginEnvironmentLease`，以及沙箱内部的 `port` 和 `path`；返回值为 `PluginEnvironmentRunnerIngressEndpoint`，包含 `kind: "authenticated_websocket"`、`websocketUrl`、`secretHeaders` 数组（元素为 `{ name, value }`）和 `generation` 字符串。只有当提供方将沙箱置于经过身份验证的隧道之后时才实现此钩子；直接执行命令的驱动可忽略它。

#### 登录 PTY 与双工通道流（可选）

有些环境驱动工作是实时双向流，而不是一次性命令，例如操作者需要输入的 Claude `setup-token` 登录提示，或主机通过管道传送数据的长期进程。有两组相配的可选钩子分别处理这两类流；只有实现后，相应方法才会公布。只提供租用和执行功能的驱动可以完全忽略这两组钩子。

**登录伪终端。** 实现 `onLoginPtyOpen`、`onLoginPtyInput`、`onLoginPtyStop` 和 `onLoginPtyClose`，即可运行一个实时登录终端。worker 通过 `ctx.loginPty` 流式传回终端输出及最终退出信息：每个新片段调用 `output(hostRouteId, workerSessionId, chunk)`，子进程结束时调用 `exit(hostRouteId, workerSessionId, exitCode)`；这些内容不会作为 RPC 回复发送。主机根据你回传的标识符将片段关联到对应路由，丢弃未知或不匹配 ID 的内容，且绝不记录原始字节。默认客户端是永不抛错的空操作。

**持久双工通道。** 对持久双向通道实现 `onDuplexChannelOpen`、`onDuplexChannelWrite`、`onDuplexChannelStop` 和 `onDuplexChannelClose`。worker 通过 `ctx.duplexChannel` 流式传输：调用 `data(hostRouteId, workerSessionId, chunk)` 发送数据；通道结束时调用 `exit(hostRouteId, workerSessionId, exitCode, transportClosed?)`。与登录 PTY 片段不同，通道片段是原始 `Uint8Array` 字节，因此 worker 到主机的传输过程会使用 Base64 编码。导出的辅助函数 `encodeChannelBytes(bytes)` 和 `decodeChannelBytes(value)` 可在原始字节与线路格式间转换；如果帧格式错误，`decodeChannelBytes` 会返回 `null`，让信任边界上的调用方将其视为协议错误，而不是静默替换为空字节。首选传输方式是 HTTP/2；`queue_v1` 是逐步弃用的后备方式。

worker 到主机的四种通知方法名称以常量形式导出：`LOGIN_PTY_OUTPUT_NOTIFICATION`、`LOGIN_PTY_EXIT_NOTIFICATION`、`DUPLEX_CHANNEL_DATA_NOTIFICATION` 和 `DUPLEX_CHANNEL_EXIT_NOTIFICATION`。每个钩子的参数结构（如 `PluginLoginPtyOpenParams`、`PluginDuplexChannelOpenParams` 等）定义在 SDK 的 `protocol.ts` 中，可通过钩子签名访问，但不会作为具名类型重新导出。

启动器渲染结构：`PluginModalBoundsRequest`、`PluginRenderCloseEvent`、`PluginLauncherRenderContextSnapshot`。

### 主机客户端工厂

如需在测试或自定义集成中嵌入桥接的主机端，可使用：

- `createHostClientHandlers`：构造主机响应 worker 到主机 RPC 调用所需的处理程序映射。
- `getRequiredCapability`：查询某个 worker 到主机调用所需的能力。
- `CapabilityDeniedError`：插件缺少所需能力时，主机处理程序抛出的错误。
- `InvocationScopeDeniedError`：worker 到主机调用请求访问当前顶层调用未授权公司的公司范围数据时，主机处理程序抛出的错误。其 `code` 为 `PLUGIN_RPC_ERROR_CODES.INVOCATION_SCOPE_DENIED`。

类型：`HostServices`、`HostClientFactoryOptions`、`HostClientHandlers`。

`HostServices` 按领域归纳了必须提供的处理程序。除现有的 `issues` 组（现在还需提供 `listInteractions`、`respondInteraction`、`listAttachments` 和 `getAttachmentContent`）外，还包括 `approvals` 组（`list`、`get`、`decide`），以及两个新增组：`access`（`listMembers`、`getMember`、`updateMember`、`listInvites`、`createInvite`、`revokeInvite`）和 `authorization`（`listGrants`、`setGrants`、`policySummary`、`getPolicy`、`updatePolicy`、`previewAssignment`、`explainAssignment`、`searchAudit`）。每个调用需要哪项能力，可通过 `getRequiredCapability` 查询。

### 打包和开发服务器

插件构建流程的辅助函数：

- `createPluginBundlerPresets`：返回类似 esbuild 和 rollup 的预设，固定插件包所需的 external 配置和入口结构。
- `startPluginDevServer`：插件 UI 包的本地开发服务器。
- `getUiBuildSnapshot`：读取当前 UI 构建快照，可用于测试。

类型：`PluginBundlerPresetInput`、`PluginBundlerPresets`、`EsbuildLikeOptions`、`RollupLikeConfig`、`PluginDevServer`、`PluginDevServerOptions`。

### 测试工具

SDK 提供了一流的测试框架，无需启动真实主机即可进行测试：

- `createTestHarness`：基础测试框架，可针对内存中的主机桩测试插件。
- `createEnvironmentTestHarness`：用于测试环境驱动插件的框架。驱动选项接受 `onStopLease` 钩子；框架还提供对应的 `stopLease(params)` 调用，因此无需真实提供方即可测试“停止并保留”行为。
- `createFakeEnvironmentDriver`：用于断言的模拟驱动实现。
- `filterEnvironmentEvents`、`assertEnvironmentEventOrder`、`assertLeaseLifecycle`、`assertWorkspaceRealizationLifecycle`、`assertExecutionLifecycle`、`assertEnvironmentError`：环境驱动流程的断言辅助函数。

类型：`TestHarness`、`TestHarnessOptions`、`TestHarnessLogEntry`、`EnvironmentTestHarness`、`EnvironmentTestHarnessOptions`、`EnvironmentEventRecord`、`FakeEnvironmentDriverOptions`。

要测试交互、审批和附件流程，可通过 `harness.seed({ ... })` 注入数据；它现在接受三个新键：

| 键 | 类型 |
|---|---|
| `issueInteractions` | `IssueThreadInteraction[]` |
| `issueAttachments` | `Array<IssueAttachment & { contentBase64?: string }>` |
| `approvals` | `Approval[]` |

要测试生命周期消费者，可用 `harness.seed({ lifecycleEvents: [...] })`（`ResourceLifecycleEvent[]`）注入日志条目。框架中的 `ctx.events.listLifecycle` 和 `ctx.events.acknowledgeLifecycle` 遵循与主机相同的规则：检查 `events.subscribe` 能力、限制数量为 `1` 到 `100`、创建事件优先且每个资源一次只投递一条，以及乱序确认时抛出错误。

测试框架会严格执行主机的写入要求，不会放宽条件：缺少 `actorUserId`、该用户不是公司的有效 `user` 成员，或其 `membershipRole` 为 `viewer` 时，`respondInteraction` 和 `approvals.decide` 都会抛出异常。要提供测试中使用的成员，请通过 `harness.seed({ accessMembers: [...] })` 注入；这样测试就无法通过生产环境本会拒绝的身份归属验证。框架还会在 `getAttachmentContent` 中执行 `maxBytes` 限制，并在重放时返回 `applied: false`，因此无需真实主机即可测试幂等行为。

### 重新导出的内容

- `z`：重新导出 `zod`，插件作者无需再添加单独依赖。可用于声明 `instanceConfigSchema` 和工具的 `parametersSchema`。
- `NOOP_PLUGIN_TRACER`、`NOOP_PLUGIN_SPAN`：默认的空操作 tracer 和 span（是值，不是类型）。例如测试中想显式使用不执行任何操作的默认实现时，可用作替代项。
- 从 `@paperclipai/shared` 导出的常量：`PLUGIN_API_VERSION`、`PLUGIN_STATUSES`、`PLUGIN_CATEGORIES`、`PLUGIN_CAPABILITIES`、`PLUGIN_UI_SLOT_TYPES`、`PLUGIN_UI_SLOT_ENTITY_TYPES`、`PLUGIN_RESERVED_COMPANY_SETTINGS_ROUTE_SEGMENTS`、`PLUGIN_STATE_SCOPE_KINDS`、`PLUGIN_JOB_STATUSES`、`PLUGIN_JOB_RUN_STATUSES`、`PLUGIN_JOB_RUN_TRIGGERS`、`PLUGIN_WEBHOOK_DELIVERY_STATUSES`、`PLUGIN_EVENT_TYPES`、`PLUGIN_BRIDGE_ERROR_CODES`。
- 从 `@paperclipai/shared` 导出的访问和权限常量数组，与 [`ctx.access` 和 `ctx.authorization`](#访问与授权) 使用的领域类型对应：`PERMISSION_KEYS`、`HUMAN_COMPANY_MEMBERSHIP_ROLES`、`HUMAN_COMPANY_MEMBERSHIP_ROLE_LABELS`、`MEMBERSHIP_STATUSES`、`PRINCIPAL_TYPES`。
- 从 `@paperclipai/shared` 重新导出的 AI 连接路由器类型，供[路由器插件](#ai-连接路由器)使用：`AiConnectionPool`、`AiConnectionPoolConfig`、`AiConnectionPoolMember`、`AiConnectionRouterRequest`、`AiConnectionRouterResult`、`AiConnectionRouterSelection`。
- 从 `@paperclipai/shared` 重新导出的连接提供方结构，供实现连接意图的插件使用：`ConnectionIntentInteraction`、`ConnectionIntentPayload`、`ConnectionIntentResult`、`ConnectionIntentSetupOptions`、`ConnectionRequestResult`、`ConnectionsSearchResult`。

---

## 示例

以下是一个最简 worker 入口点示例，其中注册了事件订阅、作业和数据馈送：

```ts
// dist/worker.ts
import { definePlugin, runWorker, z } from "@paperclipai/plugin-sdk";

const plugin = definePlugin({
  async setup(ctx) {
    ctx.logger.info("Plugin starting up");

    ctx.events.on("issue.created", async (event) => {
      ctx.logger.info("Issue created", { issueId: event.entityId });
    });

    ctx.jobs.register("full-sync", async (job) => {
      ctx.logger.info("Starting full sync", { runId: job.runId });
      // ... sync implementation
    });

    ctx.data.register("sync-health", async ({ companyId }) => {
      const state = await ctx.state.get({
        scopeKind: "company",
        scopeId: String(companyId),
        stateKey: "last-sync-at",
      });
      return { lastSync: state };
    });
  },

  async onHealth() {
    return { status: "ok" };
  },
});

export default plugin;
runWorker(plugin, import.meta.url);
```

上述结构是 SDK 自身 `index.ts` 文件头部的规范示例。清单类型和能力标记的对应信息，请参阅上文列出的 `Plugin*Declaration` 类型。

---

## Worker 入口点校验

只有在调用 `runWorker(plugin, import.meta.url)` 的文件就是进程入口点时，该调用才会启动 JSON-RPC 主机。此检查刻意兼容通过符号链接组织的软件包布局；本地插件开发中很常见这种布局，例如由 `pnpm` 链接的 SDK 或工作区链接的插件可能经过一层或多层符号链接。

实现此逻辑并导出的辅助函数为 `isWorkerEntrypoint(entry, moduleUrl)`：

- 它接收 `process.argv[1]`（Node 启动时使用的路径）和传给 `runWorker` 的 `import.meta.url`。
- 它通过 `fs.realpathSync.native` 解析两侧路径；若 realpath 调用抛出异常（例如路径尚不存在），则退回使用普通的 `path.resolve`。
- 它比较解析后的真实路径是否相等。相等时表示当前文件是入口点，`runWorker` 会调用 `startWorkerRpcHost({ plugin })`。如果不相等，`runWorker` 会静默返回；当同一模块也会被测试或重新导出用的 shim 导入时，这一行为很有用。

实际影响如下：

- **符号链接的插件包可正常运行。** 主机使用 `node /Users/you/.../dist/worker.js` 启动、而该路径经过符号链接解析时，真实路径比较仍会与 `import.meta.url` 匹配，worker 能正常启动。
- **进程内测试会跳过检查。** 在 `RunWorkerOptions` 中同时传入 `stdin` 和 `stdout`，`runWorker` 就会直接启动主机，而不检查 `process.argv[1]`。测试框架（`createTestHarness`、`createEnvironmentTestHarness`）会使用此路径。
- **重新导入 worker 文件是安全的。** 从其他文件导入 worker 模块（例如由 `worker-bootstrap.ts` 自行调用 `startWorkerRpcHost`）不会导致 RPC 主机重复启动，因为此时 `process.argv[1]` 指向 bootstrap 文件，而不是 worker 模块。

---

## 相关内容

- [管理 → 插件](../../administration/plugins.md)：以操作者身份安装、启用、配置和卸载插件。
- [操作指南 → 编写公司技能](../../how-to/write-a-company-skill.md)：代理加载的指令，**不是**服务器代码。
- [参考 → 创建适配器](../adapters/creating-an-adapter.md)：为新的 AI 运行时扩展 Paperclip 时应使用的扩展点。
- [参考 → 技能](../skills.md)：插件的 `PluginManagedSkillDeclaration` 所接入的技能文件结构和安装流程。

