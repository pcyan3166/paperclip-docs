---
paperclip_version: v2026.1023.0
seo_title: ClickHouse 连接器
seo_description: 允许智能体查询 ClickHouse Cloud ClickStack 服务。查找服务 ID，为何不支持自托管，有界查询测试，以及故障排查。
---

# ClickHouse

智能体可以查询 ClickHouse Cloud ClickStack 服务——这对于让智能体回答来自可观测性或分析数据的问题非常有用。

> **警告：** 这是 ClickHouse Cloud 托管的 ClickStack 服务器。通过此连接器无法访问自托管或自管理的 ClickHouse 集群。如果您运行自己的 ClickHouse，请改为[连接您自己的 MCP 服务器](custom-mcp-servers.md)。

## 连接前准备

- 一个拥有 ClickStack 服务的 ClickHouse Cloud 账户。
- 该服务的 **ClickHouse Cloud 服务 ID**。从 **ClickStack → 团队设置 → API 与智能体** 中复制。它看起来像 `11e1031f-9a13-4cac-9bc7-d4ec9286ec17`。
- 您登录所用账户的数据库授权。智能体可以查询什么由 ClickHouse 自身的用户和授权决定，因此请在连接之前而非之后确定这些设置。

## 连接 ClickHouse

1. 打开 **连接器** 并选择 **ClickHouse**。
2. 阅读主按钮上方的访问行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择不同身份、缩小智能体范围或使用其他登录方法，请选择 **更改**。
3. 输入 **ClickHouse Cloud 服务 ID**。
4. 选择 **继续登录** 并完成浏览器登录。

## 选择访问权限

真正的控制权在 ClickHouse 一侧。可达范围取决于登录账户的数据库用户和授权所允许的内容——Paperclip 不会在其上添加表或数据库过滤器。

如果智能体应仅具备读取权限，则仅授予读取权限。只读的 ClickHouse 用户比仅依赖操作设置提供了更强且更清晰的边界，并且能够抵御工具目录的变更。

> **警告：** ClickHouse 查询可能代价高昂。对大表的无界查询会产生真实的计算成本，并可能影响使用该服务的所有人。请保持智能体查询有界，并优先使用具有合理配额的服务器或用户。

除非有特定原因，否则任何模式更改或数据更改操作应保持为 **关闭**。参见[设置操作权限](action-permissions.md)。

### `LIMIT` 不是成本控制

这是让您在让智能体接触大表之前值得内化的陷阱。`LIMIT` 限制查询*返回*的行数。它并不能可靠地限制为生成这些结果而*读取*的行数——对未过滤表的 `LIMIT 10` 仍可能扫描整个表。

ClickHouse 在其针对智能体生成查询的指导中直接指出：“始终使用 `max_rows_to_read` 或 `max_bytes_to_read` 限制扫描大小——仅使用 `LIMIT` 无法防止全表扫描。”

在 ClickHouse 一侧，通过以下设置来限制工作本身：

| 设置 | 限制内容 |
| --- | --- |
| `max_rows_to_read` | 物化前扫描的行数——真正起作用的护栏 |
| `max_bytes_to_read` | 扫描的字节数 |
| `max_execution_time` | 中断超过 N 秒的查询 |
| `max_estimated_execution_time` | 拒绝预计运行时间过长的查询 |

将这些设置为连接登录所用 ClickHouse 用户的配额或配置文件设置，以便它们适用于智能体编写的任何内容，而不是依赖智能体的记忆。ClickHouse 的[智能体查询安全指南](https://github.com/ClickHouse/agent-skills/blob/main/skills/clickhouse-best-practices/rules/agent-query-safety.md) 提供了推荐值。

## 尝试使用

仅使用**元数据**进行验证。元数据读取可以在完全不扫描表数据的情况下确认凭据和授权，这避免了针对应用表内容的查询，但仍会使用提供商请求和资源：

```txt
List the databases and tables the ClickHouse connection can see. Do not query
any table contents and do not change anything.
```

预期结果是一个与该用户在 ClickStack 中看到的授权相匹配的列表。

仅当上述步骤成功，且仅针对已知较小的表时，尝试有界读取——并依赖用户的扫描限制而非 `LIMIT` 子句来保持其有界。

不要通过对生产表进行计数来验证。“最小的表”不是足够的成本限制：智能体可能会运行额外的查询来识别它。

> **注意：** 这是示例任务，而非记录的测试结果。本文档未对任何 ClickHouse 服务执行查询。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 设置无法完成 | 服务 ID 缺失或格式错误 | 从 **ClickStack → 团队设置 → API 与智能体** 中复制 |
| 找不到服务 ID | 账户没有 ClickStack 服务，或者是自托管的 | 需要 ClickHouse Cloud 上的 ClickStack |
| 授权成功但未显示任何表 | 登录用户在任何数据库上都没有授权 | 在 ClickHouse 中为该用户授予访问权限 |
| 缺少某些表 | 授权仅覆盖部分数据库 | 调整授权；无需重新连接 |
| 查询被拒绝 | 用户为只读且查询尝试写入 | 如果您这样配置，这是预期行为 |
| 尽管有 `LIMIT`，查询仍缓慢或昂贵 | `LIMIT` 限制返回的行数，而非扫描的行数 | 在 ClickHouse 用户上设置 `max_rows_to_read` / `max_bytes_to_read` 和执行时间限制 |
| **需要关注** | 登录已过期或服务已被移除 | 选择 **重新连接** 并确认服务仍然存在 |

限制：每个连接仅限一个 ClickStack 服务。仅限 ClickHouse Cloud。访问控制是 ClickHouse 的授权，而非 Paperclip 资源过滤器。

## 相关指南

- [连接器概述](https://paperclip.ing/product/connectors/clickhouse/)

- [Supabase](supabase.md) — 另一个数据库连接器。
- [连接您自己的 MCP 服务器](custom-mcp-servers.md) — 用于自托管 ClickHouse。
- [设置操作权限](action-permissions.md)
- [ClickHouse 托管 ClickStack MCP 服务器](https://clickhouse.com/blog/announcing-managed-clickstack-mcp-server)
