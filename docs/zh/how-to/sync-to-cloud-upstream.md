---
paperclip_version: v2026.609.0
seo_title: 将本地公司同步至云端上游
seo_description: 将公司的智能体、项目、议题、技能及可移植文件迁移至已连接的云端堆栈：预览、解决冲突、推送，然后验证。
---

# 将本地公司同步至 Paperclip 云端上游

> **已弃用 — 即将移除。** 主机到主机的云端同步功能已在上游退役。若要将公司迁移至另一实例（包括托管实例），请将其导出为可移植包并在目标实例中导入——替代方案具有完整保真度，支持本地到本地、通过 URL 或从 GitHub 进行迁移。请参阅 [备份和恢复公司](./back-up-and-restore-a-company.md)。此页面暂时保留，供使用旧版本构建的用户参考，将在未来版本中移除。

云端上游同步会将本地 Paperclip 公司——包括其智能体、项目、议题、技能和可移植文件——迁移至已连接的 Paperclip 云端堆栈。您可以预览将要发生的变更，解决任何冲突，推送工作，然后决定哪些实体类型（智能体、例程、监视器）应在云端侧激活。

该流程为可选参与、实验性质，并受实例设置控制。在您开启标志并授权连接之前，任何数据都不会离开您的本地实例。

首次执行干运行推送所需时间：约 15 分钟。

---

## 所需条件

- 一个正在运行的本地 Paperclip 实例，您可以以董事会用户身份访问。
- 一个 Paperclip 云端堆栈 URL，该堆栈在 `/.well-known/paperclip-upstream` 处宣传 `cloud_sync` 传输功能标志。
- 您想要推送的本地 `companyId`。
- 在云端堆栈上拥有足够的权限以批准授权授予——CLI 请求 `upstream_import:preview`、`upstream_import:write` 和 `upstream_import:read` 范围。

---

## 1. 开启实验性设置

在您选择加入之前，云端上游同步功能是隐藏的。在标志翻转为 `true` 之前，服务器会以 `Cloud sync is not enabled` 拒绝所有云端路由，CLI 也会以相同消息拒绝推送。

该标志位于实例实验性设置中：

```bash
curl -X PATCH "$PAPERCLIP_API_URL/api/instance/settings/experimental" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "enableCloudSync": true }'
```

您也可以通过董事会设置面板进行切换——一旦标志开启，云端上游条目就会显示出来。

---

## 2. 连接至云端堆栈

CLI 会生成一个新的源身份（一个 ed25519 密钥对加一个指纹），打开浏览器访问云端堆栈的 PKCE 授权 URL，并将生成的令牌存储在 `~/.paperclip/<instance>/secrets/cloud-upstream-connections.json` 下。该文件以 `0600` 模式写入——请保持此模式不变。

```bash
paperclipai cloud connect https://cloud.example.com
```

如果主机没有浏览器，请回退到设备代码流程：

```bash
paperclipai cloud connect https://cloud.example.com --no-browser
```

您将看到一个验证 URL 和一个一次性用户代码。在拥有浏览器的机器上打开该 URL，粘贴代码，批准授予，然后 CLI 将接管。

董事会 UI 在云端上游页面也有一个连接入口。它调用 `POST /api/cloud-upstreams/connect/start` 来生成 `pendingConnectionId` 和 `authorizationUrl`，然后在重定向落地后通过 `POST /api/cloud-upstreams/connect/finish` 完成。

无论哪种方式，最终都会得到一个存储的连接，供后续步骤复用。

---

## 3. 预览推送

预览是一次干运行。CLI 通过可移植服务导出本地公司，构建传输清单，并询问云端堆栈将会发生什么变化。远程端不会写入任何内容。

```bash
paperclipai cloud push \
  --company "$LOCAL_COMPANY_ID" \
  --dry-run
```

输出会打印运行 ID、清单哈希值以及一行摘要：

```
create=12 update=3 adopt=0 skip=1 conflict=0 staleMapping=0
```

任何警告（来自本地导出或远程端）以及前十个冲突会打印在摘要下方。如果您想要完整的有效载荷，请添加 `--json`。

从董事会端，相同的预览可通过 `POST /api/cloud-upstreams/{connectionId}/push-runs/preview` 获取。响应是一个 `CloudUpstreamPreview`，云端上游董事会页面将其渲染为每种实体类型的计数、警告列表（带有 `severity` `warning` 或 `blocker`）以及冲突列表（带有 `plannedAction` 为 `create`、`update`、`skip` 或 `blocked`）。

预览中的 `schemaCompatible: false` 意味着云端堆栈处于不同的传输架构主版本。请停止并升级其中一侧——在这种情况下，CLI 将以代码 `3` 退出，并显示清晰的 `Cloud upstream schema mismatch` 消息。

---

## 4. 在应用前解决冲突

冲突是指云端堆栈表示“我已经拥有具有此自然键的内容，且不会静默覆盖它”。每个 `CloudUpstreamConflict` 包含：

- `entityType` — 发生冲突的实体类型。
- `sourceLabel` / `targetLabel` — 本地和远程行的可读名称。
- `plannedAction` — 如果您允许通过，导入器将执行的操作（`create`、`update`、`skip` 或 `blocked`）。
- `reason` — 导入器的理由。

逐行决定是重命名还是移除本地实体，然后重新运行预览。干运行成本很低；您可以按需运行多次。

CLI 在 `stderr` 上显示冲突，并在存在至少一个冲突或过时映射时以代码 `2`（非致命提示）退出。脚本可以根据该退出代码进行分支处理，而无需解析输出。

---

## 5. 应用运行

一旦预览干净——或者您已接受导入器允许处理的冲突——请移除 `--dry-run`：

```bash
paperclipai cloud push --company "$LOCAL_COMPANY_ID"
```

CLI 上传分块的实体有效载荷（默认每块 100 个实体；可通过 `--max-entities-per-chunk` 调整），通知远程端应用，然后获取尾随事件日志。摘要行和最后十个事件会打印到标准输出。

从 API 角度，等效操作是使用本地 `companyId` 的 `POST /api/cloud-upstreams/{connectionId}/push-runs`。响应是一个 `CloudUpstreamRun`，您可以通过 `GET /api/cloud-upstreams/{connectionId}/push-runs/{runId}` 轮询，直到 `status` 从 `running` 翻转为 `succeeded`、`failed` 或 `cancelled`。`activeStep` 字段会经历 `connect`、`scan`、`preview`、`push`、`verify`、`activate`，以便董事会 UI 可以显示进度，而无需自行发明阶段。

如果您需要在运行中途放弃，`POST /api/cloud-upstreams/{connectionId}/push-runs/{runId}/cancel` 将其标记为已取消并通知远程端停止。重试时，在创建调用中通过 `retryOfRunId` 引用失败的运行。

---

## 6. 在云端侧激活实体

成功的推送会将所有内容以 **暂停** 状态落地在远程端。智能体不会领取议题，例程不会触发，监视器不会发出警报——云端堆栈等待您显式开启每种实体类型。

每次激活一种类型：

```bash
curl -X POST \
  "$PAPERCLIP_API_URL/api/cloud-upstreams/$CONNECTION_ID/push-runs/$RUN_ID/activation" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "companyId": "'$LOCAL_COMPANY_ID'", "entityType": "agents" }'
```

`entityType` 接受 `agents`、`routines` 或 `monitors`——其他任何值将返回 `400`。响应是一个 `CloudUpstreamActivationDecision`，包含已切换的计数和 `activatedAt` 时间戳。

董事会的云端上游页面暴露了三个按钮，映射到相同的调用。先激活智能体，然后例程，最后监视器是安全的默认顺序：智能体是执行者，例程触发工作，监视器只有在有工作可监视时才有意义。

---

## 验证落地情况

连接的 `lastRunId` 指向最近的运行，因此单次获取即可告知您所有信息：

```bash
curl -sS "$PAPERCLIP_API_URL/api/cloud-upstreams?companyId=$LOCAL_COMPANY_ID" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY"
```

`CloudUpstreamsState` 响应包含该公司的连接列表和最近运行列表。抽查确认：

- 连接的 `tokenStatus` 为 `connected`（而非 `expired` 或 `revoked`），
- 运行的 `status` 为 `succeeded`，
- `progressPercent` 为 `100`，
- 且 `summary` 与您接受的预览相匹配。

跟随 `targetUrl` 进入云端堆栈中的运行，并确认激活决定已生效。

---

## 常见故障

| 症状 | 含义 |
|---|---|
| `Cloud sync is disabled.` (CLI) 或 HTTP `404 Cloud sync is not enabled` | `enableCloudSync` 实验性标志为 `false`。请翻转它（步骤 1）并重试。 |
| `Remote URL is not a Paperclip Cloud upstream target.` | 远程端未提供带有 `schema: "paperclip-upstream-discovery-v1"` 的有效 `/.well-known/paperclip-upstream` 文档。 |
| `Cloud upstream schema mismatch: local major X, remote supports Y.` (CLI 退出代码 `3`) | 两侧处于不同的传输架构主版本。请升级其中一侧。 |
| `Remote Paperclip Cloud stack does not advertise the cloud_sync transfer flag.` | 云端堆栈可达，但未启用云端同步。请要求堆栈管理员宣传 `cloud_sync` 功能标志。 |
| `cloud push` 后 CLI 退出代码 `2` | 至少有一个冲突或过时映射行需要人工关注。使用 `--dry-run` 重新运行以检查，然后要么在本地解决，要么接受并重新应用。 |
| `No cloud connection found.` | CLI 无法在本地存储中匹配 `--remote-url`（或默认当前连接）。请重新运行 `paperclipai cloud connect`。 |

---

## 另请参阅

- [CLI 参考中的云端命令](../../reference/cli/cloud.md) — `paperclipai cloud connect` 和 `paperclipai cloud push` 的所有标志。
- [备份和恢复公司](./back-up-and-restore-a-company.md) — 云端推送所包装的本地可移植导出。
- [术语表](../guides/welcome/glossary.md) — 公司、智能体、例程、监视器的定义。
