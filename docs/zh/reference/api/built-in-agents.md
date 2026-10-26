---
paperclip_version: v2026.720.0
seo_title: 内置智能体 API
seo_description: Paperclip 随附、可以按名称配置到任意公司的第一方智能体，无需每次都从头配置同类角色。
---

# 内置智能体

Paperclip 随附的第一方智能体，可随时配置到任意公司。

内置智能体是 Paperclip 根据名称识别的普通智能体。你不必从零招聘并自行设置角色、目的和指令，而是可以从服务器维护的小型注册表中配置它。每个内置智能体都有稳定的 `key`，例如 `briefs` 或 `learning`；Paperclip 使用此键解析智能体，而不是依赖硬编码的数据库 ID，因此后端功能可以引用“此公司的 Briefs 智能体”，无需固定其 ID。

如果你希望为公司启用 Paperclip 随附的智能体、检查其设置状态、指定适配器，或（对于捆绑的内置智能体）控制其运行计划，请使用此 API。

如果你还没有阅读，请先查看 [API 概览](./overview.md)，了解基础 URL、身份验证和公司范围规则。本页所有接口都遵循这些约定。

---

## 内置智能体的特点

从底层来看，内置智能体与招聘的其他智能体一样，都是 `agents` 表中的普通记录。使其与众不同的是存储在智能体 `metadata` 中、不可变的 `metadata.paperclipBuiltInAgent` 标记。此标记记录注册表 `key` 和该智能体支持的功能键。Paperclip 之后会根据此标记查找智能体，同时也会保护它：无法通过常规智能体创建和更新路由伪造、移动或移除此标记。只有内置智能体服务可以写入此标记。

由于内置智能体由注册表管理，它们已经带有合理的默认值，包括显示名称、简短用途、默认角色，以及允许使用的适配器类型。最后一步由你完成：选择适配器并配置它。

---

## Paperclip 随附的内置智能体

目前 Paperclip 随附三种内置智能体。下表内容直接来自注册表。

| Key | 显示名称 | 用途 | 默认角色 | 允许的适配器类型 |
|---|---|---|---|---|
| `briefs` | Briefs Agent | 为看板和智能体公司准备简明的运营简报。 | `general` | `codex_local`、`claude_local`、`gemini_local`、`opencode_local`、`process` |
| `learning` | Learning Agent | 根据已完成工作和重复出现的模式，维护可复用的公司经验。 | `general` | `codex_local`、`claude_local`、`gemini_local`、`opencode_local`、`process` |
| `reflection-coach` | Reflection Coach | 针对智能体近期工作进行有证据支持的复盘，提出小幅改进指令和技能的建议，并在应用更改前请求审批。 | `general` | `claude_local`、`codex_local`、`gemini_local`、`opencode_local`、`process` |

每个列表中排在最前的适配器类型，是不指定适配器进行配置时服务器会选择的默认值。Reflection Coach 与另外两个智能体略有不同：它以**捆绑组件**的形式随附，初始状态为暂停。详见下文[Reflection Coach](#reflection-coach)。

---

## 生命周期状态

每种内置智能体都有状态，根据公司和键对应的带标记智能体记录计算。列出内置智能体或读取其状态时，会返回以下值之一：

| 状态 | 含义 |
|---|---|
| `not_provisioned` | 此公司和键对应的内置智能体尚不存在。配置它即可开始使用。 |
| `pending_approval` | 你已请求内置智能体，但公司要求新智能体经过看板审批，因此它会等待决定后才启用。 |
| `needs_setup` | 智能体记录已存在，但适配器配置不完整，例如缺少模型或命令。在你重新提交适配器详情完成设置之前，它不会运行。参见[完成首次设置](#完成首次设置)。 |
| `ready` | 适配器配置完整，且智能体未暂停，可以开始工作。 |
| `paused` | 智能体已暂停。计划任务或后台工作应跳过它，不要针对它排队。 |

需要使用内置智能体的后端功能会在安排工作前调用内部辅助函数。如果智能体不存在或仍为 `needs_setup`，该函数会返回 **HTTP 412** 和 `code: "built_in_agent_not_configured"`。请将此视为需要运维人员处理的设置任务，而不是服务器故障。如果智能体为 `paused`，功能会在智能体旁边收到 `built_in_agent_paused` 警告，以便记录跳过并继续，不会误将其视为就绪状态。

---

## 配置与权限

配置内置智能体**不需要**经过完整的智能体招聘流程。无需额外获取招聘审批权限；这些端点只要求常规的 `agents:create` 权限，与创建任何智能体所需权限相同。内置智能体是注册表管理的系统能力，因此 Paperclip 将启用它视为配置，而非专门的招聘流程。

需要注意一点：如果公司开启了“新智能体需要看板审批”，配置新的内置智能体仍会进入常规待处理审批。在这种情况下，配置端点会返回 `202 Accepted`，内置智能体状态为 `pending_approval`，直到看板作出决定。如果公司不要求看板审批，内置智能体会立即配置完成并返回 `200 OK`。

### 完成首次设置

部分内置智能体可能会先于配置到达。获批招聘后，你可能会看到智能体记录已经创建，但其 `adapterConfig` 仍为空；这正是 `needs_setup` 状态。通常的下一步是打开智能体设置对话框，选择适配器类型，填写模型或命令，然后保存。

如果公司开启了“新智能体需要看板审批”，过去尝试此操作会卡住：请求返回 `409 Conflict`、`code: "built_in_agent_reconfiguration_requires_approval"`，并显示消息“应用内置智能体适配器更改前需要看板审批。”如果你自己就是看板，就没有其他人可以再次批准你已经拥有的审批，因此设置无法完成。

此问题现已修复。对于尚未完成适配器设置的内置智能体，发送适配器详情会被视为首次配置，并立即生效，与关闭看板审批时的流程相同。调用常规端点：

`POST /api/companies/{companyId}/built-in-agents/{key}/provision`

此请求会返回 `200 OK`，不会创建审批；内置智能体状态从 `needs_setup` 变为 `ready`。

附近还有两种不同情况，这都是有意为之：

- **更改已配置内置智能体的适配器**：例如状态为 `ready` 或 `paused` 的智能体。这属于真正的重新配置，仍会返回 `409 Conflict` 和 `code: "built_in_agent_reconfiguration_requires_approval"`。
- **内置智能体仍处于 `pending_approval` 时提交适配器详情**：会返回 `409 Conflict`、`code: "built_in_agent_pending_approval"` 和消息“内置智能体设置已在等待看板审批。”请等待看板作出决定后再配置智能体。

适配器配置何时算“完成”取决于适配器类型。对于 `process` 和 `command` 适配器，Paperclip 会检查是否提供了 `command` 或 `script`。对于 `http`，会检查 `url`、`endpoint` 或 `webhookUrl`。对于 `openclaw_gateway` 和 `hermes_gateway`，会检查 `baseUrl` 或 `url`。其他所有适配器（包括随附内置智能体默认使用的本地适配器）则检查 `model`。

> **注意：** 内置智能体受实例功能开关控制。实例未启用此功能时，本页每个路由都会返回 `404 Not Found` 和消息“Built-in agents are not enabled.”

例程控制端点（启用、禁用、运行）的门槛更高：它们会安排并触发实际工作，因此要求调用方是拥有 `tasks:assign` 权限的**看板运维人员**。

---

## API

所有路由都限定在公司范围内，且位于 `/api` 下。本页 `{key}` 均表示注册表键，例如 `briefs`、`learning` 或 `reflection-coach`。

### 列出内置智能体

`GET /api/companies/{companyId}/built-in-agents`

返回注册表中的所有定义及其在此公司中的当前状态。运维人员可以使用此端点呈现“内置智能体”面板。列表视图会对适配器配置和运行时配置脱敏。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -s \
  "http://localhost:3100/api/companies/{companyId}/built-in-agents" \
  -H "Authorization: Bearer <token>"
```

<!-- tab: JavaScript -->

```js
const res = await fetch(
  `http://localhost:3100/api/companies/${companyId}/built-in-agents`,
  {
    headers: { Authorization: `Bearer ${token}` },
  },
);

const builtIns = await res.json();
```

<!-- tab: Python -->

```python
import requests

res = requests.get(
    f"http://localhost:3100/api/companies/{companyId}/built-in-agents",
    headers={"Authorization": f"Bearer {token}"},
)
built_ins = res.json()
```

<!-- /tabs -->

### 获取状态

`GET /api/companies/{companyId}/built-in-agents/{key}/status`

返回单个内置智能体的当前状态。配置或重置后，可使用此端点确认智能体最终处于哪个状态。

### 配置

`POST /api/companies/{companyId}/built-in-agents/{key}/provision`

为公司创建内置智能体，或更新已有内置智能体，并为其指定适配器。要求拥有 `agents:create` 权限。

请求正文可选：可以发送空正文接受注册表默认值，也可以提供下列任何字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| `adapterType` | 字符串 | 必须是此内置智能体允许的适配器类型之一。默认使用允许列表中的第一个类型。 |
| `adapterConfig` | 对象 | 适配器专属配置，例如根据适配器类型提供 `model`、`command` 或 `url`。 |
| `budgetMonthlyCents` | 整数 | 月预算，单位为美分。默认采用注册表默认值（目前所有随附内置智能体均为 `0`）。 |

直接配置内置智能体时返回 `200 OK`；若公司要求看板审批且请求因此创建了待处理审批，则返回 `202 Accepted`。

如果内置智能体仍处于 `needs_setup`，也使用此端点完成适配器设置。即使公司启用了看板审批，首次配置也会直接生效并返回 `200 OK`。详见[完成首次设置](#完成首次设置)，其中说明了仍需审批的两种情况。

<!-- tabs: cURL, JavaScript, Python -->

<!-- tab: cURL -->

```bash
curl -s -X POST \
  "http://localhost:3100/api/companies/{companyId}/built-in-agents/briefs/provision" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "adapterType": "claude_local",
    "adapterConfig": {
      "model": "claude-sonnet-4-20250514",
      "cwd": "/Users/me/projects/company"
    }
  }'
```

<!-- tab: JavaScript -->

```js
const res = await fetch(
  `http://localhost:3100/api/companies/${companyId}/built-in-agents/briefs/provision`,
  {
    method: "POST",
    headers: {
      Authorization: `Bearer ${token}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      adapterType: "claude_local",
      adapterConfig: {
        model: "claude-sonnet-4-20250514",
        cwd: "/Users/me/projects/company",
      },
    }),
  },
);

const state = await res.json();
```

<!-- tab: Python -->

```python
import requests

res = requests.post(
    f"http://localhost:3100/api/companies/{companyId}/built-in-agents/briefs/provision",
    headers={
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json",
    },
    json={
        "adapterType": "claude_local",
        "adapterConfig": {
            "model": "claude-sonnet-4-20250514",
            "cwd": "/Users/me/projects/company",
        },
    },
)
state = res.json()
```

<!-- /tabs -->

### 协调

`POST /api/companies/{companyId}/built-in-agents/{key}/reconcile`

确保内置智能体存在，且由注册表管理的组成部分已经实体化，但不会更改你配置的适配器。发送空正文即可。这是一种静默、幂等的方式，可确保内置智能体（以及捆绑内置智能体的指令、技能和例程）已配置到位。要求拥有 `agents:create` 权限。

### 重置

`POST /api/companies/{companyId}/built-in-agents/{key}/reset`

将内置智能体恢复到注册表管理的默认值，包括显示名称、用途、角色，以及捆绑的指令、技能或例程，同时**保留适配器设置**，避免丢失你配置的本地模型或命令。要求拥有 `agents:create` 权限。

请求正文可选。要重置全部内容，请发送空正文；要只重置捆绑内容的一部分，请传入 `resources` 数组：

| 字段 | 类型 | 说明 |
|---|---|---|
| `resources` | 数组 | 可以包含 `"agent"`、`"instructions"`、`"skill"`、`"routine"`。省略时重置全部内容。 |

### 例程控制（捆绑内置智能体）

捆绑内置智能体随附一个默认禁用的计划例程。以下路由用于启用、禁用它或触发一次性运行。需要拥有 `tasks:assign` 权限的看板运维人员。在每条路径中，`{routineKey}` 是例程的键；对于 Reflection Coach，该键为 `recent-agent-reflection`。

`POST /api/companies/{companyId}/built-in-agents/{key}/routines/{routineKey}/enable`

`POST /api/companies/{companyId}/built-in-agents/{key}/routines/{routineKey}/disable`

`POST /api/companies/{companyId}/built-in-agents/{key}/routines/{routineKey}/run`

- **enable**：启用例程及其计划；如果智能体已暂停，也会恢复运行。
- **disable**：暂停计划和例程。
- **run**：立即触发一次运行，不受计划控制。返回 `202 Accepted`。

---

<a id="the-reflection-coach"></a>
## Reflection Coach

值得特别介绍 Reflection Coach，因为它既是内置智能体，**也是**捆绑技能。配置后，Paperclip 会为公司创建三个组成部分：智能体指令、`reflection-coach` 技能和每周运行的 `recent-agent-reflection` 例程（默认禁用，需手动启用）。它默认也处于**暂停**状态，因此在你主动完成配置和启用之前不会运行。

它的工作是提供辅导。每次运行时，它都会检查另一个智能体最近完成的工作，并提出对该智能体指令或技能的一项最小且持久的改进建议。其关键设计原则是：不会在同一次运行中热替换任何内容。所有更改都是**经审核且受门控控制的提案**——教练必须先展示差异、等待已接受的任务交互，然后只在单独的后续运行中应用更改。不会在同一次运行中直接编辑。

此门控通过权限实现。Reflection Coach 获得的是两个仅供审核的权限键，而不是直接写入权限：

- `agents:suggest-changes`：提出修改智能体设置的建议以供审核，但不直接应用。
- `skills:suggest-changes`：提出修改公司技能的建议以供审核，但不直接应用。

这两个键是直接写入权限 `agents:configure` 和 `skills:create` 的较温和对应权限。Reflection Coach 只持有“建议”权限，因此其提案始终会经过审核，然后才会生效。完整权限模型（包括各角色持有的权限键）见[角色与权限](../../administration/roles-and-permissions.md)。

---

## 后续阅读

- [智能体 API](./agents.md)：内置智能体所基于的完整智能体生命周期、适配器和配置。
- [角色与权限](../../administration/roles-and-permissions.md)：了解本页提到的 `agents:create`、`tasks:assign` 和 `*:suggest-changes` 权限键。
- [技能参考](../skills.md)：了解 Reflection Coach 等捆绑技能的打包、导入和版本管理方式。
