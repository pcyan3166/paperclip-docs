---
seo_title: 反馈与投票
seo_description: 对智能体评论和文档选择“有帮助”或“需要改进”。投票默认只保存在本机；是否导出或分享由你决定。
---

# 反馈与投票

智能体发布评论或修改文档时，你可以选择 **Helpful（有帮助）**（点赞）或 **Needs work（需要改进）**（点踩）。投票是最快的方式，能让智能体以及维护 Paperclip 的人员知道哪些结果有效、哪些需要改进。

投票默认只保存在你的本机。除非你主动选择分享特定反馈，否则任何数据都不会离开你的实例。

---

## 如何投票

1. 在任意智能体评论或文档修订上，点击 **Helpful（有帮助）** 或 **Needs work（需要改进）**。
2. 如果选择了 **Needs work（需要改进）**，系统会显示一个可选提示：*“What could have been better?（哪些地方可以改进？）”* 输入原因，也可以关闭提示。
3. 首次投票时，系统会询问你要将投票保留在本机，还是与外部分享。你的选择会按公司保存，之后可以更改。

### 会保存哪些数据

每次投票都会在本地生成两条记录：

| 记录 | 内容 |
|---|---|
| **Vote（投票）** | 投票方向（赞成/反对）、可选原因、分享偏好、同意版本和时间戳 |
| **Trace bundle（跟踪数据包）** | 完整上下文快照：被投票的评论或修订、任务标题、智能体信息、投票结果和原因，足以单独理解这条反馈 |

所有数据都保存在你的本地 Paperclip 数据库中。如果投票标记为*分享*，Paperclip 会立即尝试将跟踪数据包上传至已配置的遥测后端。上传内容会经过压缩，让完整数据包保持在网关限制以内。如果上传失败，跟踪记录会进入可重试的 `failed` 状态，以便稍后重新发送。

---

## 查看你的投票

### 在终端中快速生成报告

```sh
pnpm paperclipai feedback report
```

该命令会打印带颜色的摘要：投票计数、每条跟踪记录的详情和原因，以及导出状态。

常用参数：

```sh
# Installed CLI
paperclipai feedback report

# Point to a specific server or company
pnpm paperclipai feedback report --api-base http://127.0.0.1:3000 --company-id <company-id>

# Include raw payloads
pnpm paperclipai feedback report --payloads
```

### API 端点

所有端点都需要董事会用户权限（本地开发时会自动满足）。

**某项任务的投票：**
```sh
curl http://127.0.0.1:3102/api/issues/<issueId>/feedback-votes
```

**某项任务的跟踪数据包（含完整内容）：**
```sh
curl 'http://127.0.0.1:3102/api/issues/<issueId>/feedback-traces?includePayload=true'
```

**公司范围内的全部跟踪记录：**
```sh
curl 'http://127.0.0.1:3102/api/companies/<companyId>/feedback-traces?includePayload=true'
```

**单条跟踪记录封装：**
```sh
curl http://127.0.0.1:3102/api/feedback-traces/<traceId>
```

**单条跟踪记录的完整导出包：**
```sh
curl http://127.0.0.1:3102/api/feedback-traces/<traceId>/bundle
```

#### 筛选参数

跟踪记录端点支持以下参数：

| 参数 | 取值 | 说明 |
|---|---|---|
| `vote` | `up`、`down` | 按投票方向筛选 |
| `status` | `local_only`、`pending`、`sent`、`failed` | 按导出状态筛选 |
| `targetType` | `issue_comment`、`issue_document_revision` | 按被投票内容类型筛选 |
| `sharedOnly` | `true` | 仅显示标记为分享的投票 |
| `includePayload` | `true` | 包含完整上下文快照 |
| `from` / `to` | ISO 日期 | 按日期范围筛选 |

---

## 导出你的数据

```sh
pnpm paperclipai feedback export
```

命令会生成带时间戳的目录和相应的 zip 文件：

```
feedback-export-20260331T120000Z/
  index.json                    # manifest with summary stats
  votes/
    PAP-123-a1b2c3d4.json       # one vote record
  traces/
    PAP-123-e5f6g7h8.json       # Paperclip feedback envelope
  full-traces/
    PAP-123-e5f6g7h8/
      bundle.json               # full export manifest
      ...raw adapter files      # codex/claude/opencode session artifacts when available
feedback-export-20260331T120000Z.zip
```

默认执行完整导出。`traces/` 包含 Paperclip 封装；`full-traces/` 保存更完整的逐条跟踪数据包以及可恢复的适配器原生文件。

```sh
# Custom server and output directory
pnpm paperclipai feedback export --api-base http://127.0.0.1:3000 --company-id <company-id> --out ./my-export
```

### 阅读导出的跟踪记录

`traces/` 中的文件示例如下：

```json
{
  "id": "trace-uuid",
  "vote": "down",
  "issueIdentifier": "PAP-123",
  "issueTitle": "Fix login timeout",
  "targetType": "issue_comment",
  "targetSummary": {
    "label": "Comment",
    "excerpt": "The first 80 chars of the comment that was voted on..."
  },
  "payloadSnapshot": {
    "vote": {
      "value": "down",
      "reason": "Did not address the root cause"
    },
    "target": {
      "body": "Full text of the agent comment..."
    },
    "issue": {
      "identifier": "PAP-123",
      "title": "Fix login timeout"
    }
  }
}
```

打开 `full-traces/<issue>-<trace>/bundle.json` 可查看扩展导出内容：捕获说明、适配器类型、完整性元数据和原始文件清单。

`bundle.json.files[]` 中的每个条目都在 `contents` 中包含实际文件内容，而不仅仅是路径。文本按 UTF-8 存储；二进制文件使用 base64 编码，并带有 `encoding` 标记。

内置本地适配器会直接导出原生会话文件：

- `codex_local`：`adapter/codex/session.jsonl`
- `claude_local`：`adapter/claude/session.jsonl`，以及可用时的 `adapter/claude/session/...` 辅助文件和 `adapter/claude/debug.txt`
- `opencode_local`：`adapter/opencode/session.json`、`adapter/opencode/messages/*.json` 和 `adapter/opencode/parts/<messageId>/*.json`，还可能包含 `project.json`、`todo.json` 和 `session-diff.json`

---

## 分享偏好

首次投票时会出现同意对话框：

- **Keep local（仅保留在本机）**——投票只留在本地（`sharedWithLabs: false`）。
- **Share this vote（分享此投票）**——标记该投票以供分享（`sharedWithLabs: true`）。

分享偏好按公司保存，你可以随时在反馈设置中修改。选择“仅保留在本机”的投票不会进入导出队列。

---

## 数据生命周期

| 状态 | 含义 |
|---|---|
| `local_only` | 仅保存在本地，未标记为分享 |
| `pending` | 已标记为分享，等待立即上传 |
| `sent` | 已成功发送 |
| `failed` | 上传失败；稍后的刷新流程会重试 |

无论分享状态如何，本地数据库都会保留完整的投票和跟踪记录。

---

## 远程同步

你选择分享的投票会随投票请求立即发送到遥测后端。后台刷新工作进程会稍后重试上传失败的跟踪记录。后端会验证请求，并将数据包写入配置好的对象存储。

- **应用服务器：** 构建数据包，将其 POST 到遥测后端，并更新跟踪状态。
- **遥测后端：** 验证请求身份和内容，压缩并存储数据包，然后返回最终对象键。
- **重试：** 上传失败的记录会变成 `failed`，并在 `failureReason` 中保存错误；工作进程会在之后的运行周期中重试。
- **默认端点：** 未配置后端 URL 时，Paperclip 会使用 `https://telemetry.paperclip.ing`。
- **快照说明：** 上传的对象是投票时刻的数据包。如果之后本地重新生成数据包，而底层适配器会话有所增长，那么本地数据包可能比先前上传的快照更大。

导出对象采用确定性的键名格式，便于检查：

```text
feedback-traces/<companyId>/YYYY/MM/DD/<exportId-or-traceId>.json
```
