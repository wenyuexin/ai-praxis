# Mem0 删除与重置：哪些状态会被清理

> 最后更新：2026-11-03
> Version Basis：`mem0ai/mem0@94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`，Python OSS `v2.2.1`
> Observed At：2026-11-02；2026-11-03 固定来源复核与证据登记重建
> Scope：固定 Python 同步与 AsyncMemory 路径；Qdrant 仅作已核对的后端实例。Platform `delete_linked` 仅保留公开契约边界。未运行删除 API。
> Drift Risk：high

Mem0 的单条 `delete()`、按 scope 的 `delete_all()` 和 `reset()` 不是同一种清理操作。正常走完删除步骤时，前两者移除主记录并追加删除 history，但不清空 SQLite messages；批量路径若遇到错误或重复批次保护，可能只完成一部分。`reset()` 则尝试重置 SQLite 表、向量存储和已初始化的实体存储。同步与异步路径还存在批量失败处理和 reset 顺序差异，因此不能将接口成功或名称直接理解成跨存储原子清除。

## 一条记录的 delete

公共 `delete(memory_id)` 先按 ID 读取主记录；找不到时抛 `ValueError`。之后内部路径读取正文、时间和 scope，按顺序执行：

1. 从 vector store 删除该 ID。
2. 向 SQLite history 插入 `event="DELETE"`、`new_memory=None`、`is_deleted=1` 的记录，并保留旧正文及相关时间、actor、role。
3. 尝试从实体索引清理关联 ID。

history 不是被删除的对象，而是此次删除留下的事件；SQLite messages 不在这条路径中。成功时公共入口返回 `{"message": "Memory deleted successfully!"}`。

vector 删除在 history 写入之前，二者没有共同事务。若 vector 调用抛错，后续步骤不执行；若 vector 删除已返回而 history 插入失败，异常会向外传播，但没有代码回滚 vector 删除。实体维护是 best-effort：未初始化时跳过，普通清理异常会被记录并吞掉，因此实体残留不必然导致 delete 报错。以上失败状态由控制流推演，未注入故障验证。

单条实体清理按原 memory 的 user／agent／run scope 扫描最多 10,000 条实体记录；找到关联后移除对应 memory ID，若没有剩余 ID 则删除实体记录，否则重写实体向量与 payload。扫描不是任意规模的全库保证，且具体向量后端行为仍取决于 provider。

## 按 scope 批量 delete_all

`delete_all()` 不是无条件清空入口。它只接受 `user_id`、`agent_id`、`run_id` 三类可选 scope；值会先规范化，至少一个有效非空 scope 是必需条件，否则抛 `ValueError` 并提示使用 `reset()`。传入多个 scope 时组成同一 filters 对象；具体筛选操作符由 vector store 实现决定，固定 Qdrant 对平铺条件采用 AND。

同步路径反复按 filters 列出最多 1,000 条主记录，再逐条走与 `delete()` 相同的 vector → DELETE history → entity cleanup 顺序。重复返回同一批 ID 时会停止循环以避免无限循环；这个保护会正常跳出循环，不抛错，随后仍走常规成功返回，因此重复批次下可能有匹配记录尚未删除。逐条异常没有被捕获，也可能在部分记录已经删除后中止整个方法。两种情况都可能留下部分结果。这条路径不清理 messages。

异步路径也按 1,000 条分页并检测重复批次；检测到重复批次时会正常停止分页，仍继续后续实体清理与常规成功返回，因此也可能有匹配主记录未删除。每批用 `gather(..., return_exceptions=True)` 并行执行删除；单条失败会记录并继续，而不是让整批异常中断。它跳过逐条实体清理，在向量批次完成后按 scope 批量删除最多 10,000 条实体记录。实体超出该扫描范围时不保证全清。无论重复批次还是单条删除失败，都不能仅凭返回值确认所有主记录均已删除。

两种 delete_all 都为每条成功到达 history 步骤的记录写 DELETE 事件，而不是移除已有 history，也不主动删除 SQLite messages。向量删除和各条 history 插入彼此不构成一个跨存储事务。

保留 messages 还会影响后续写入：若相关旧消息仍位于同一 `session_scope` 的最近十条缓存内，后续普通 `add()` 会将其作为短上下文提供给抽取模型。固定提示词要求这些旧消息仅帮助理解新消息、抽取应来自新消息，因此不能直接断言已删事实会被重新抽取；但删除主记录也不等于消除了所有后续上下文影响。换 scope、消息淘汰和模型输出都会改变结果。这是从读取／清理路径连接得出的潜在风险，不是已复现的旧值重现，也不是 search 从 history 中恢复被删记录。

## reset 的范围和顺序

SQLite `reset()` 在一个 SQLite 事务中删除 `history` 与 `messages` 两张表；它不删除数据库文件。Memory 随后关闭并重建 SQLite manager，因此配置的数据库路径仍被复用。

同步 `Memory.reset()` 的顺序是：

1. 重置 SQLite 表并重建 manager。
2. 调用 vector adapter 的 `reset()`（若支持），否则删除 collection 并创建新的 adapter。
3. 若 entity store 已初始化，尝试 reset；异常只记录，之后清空当前实例上的 entity store 引用。

异步 `AsyncMemory.reset()` 使用另一顺序：先直接删除 vector collection，运行垃圾回收并关闭可关闭的 client；再重置并重建 SQLite，随后构造新的 vector adapter；最后尝试 reset 已初始化的 entity store，失败只记录并清空引用。它不调用 vector adapter 的 `reset()`。除实体 reset 外，这些步骤没有外围的 catch-and-continue；中途异常会向外传播，后续步骤不再执行。

这两条 reset 路径没有跨 SQLite、主向量存储和实体存储的统一事务。若同步路径的 SQLite reset 已完成而 vector reset 失败，或异步路径的 collection 删除已完成而 SQLite reset 失败，就可能留下不同步状态；这些是控制流推演，未注入故障。entity reset 的异常被捕获，不会仅凭该异常阻止流程继续。返回值没有逐 store 清理报告。reset 后是否确实清空，仍受 adapter/provider 的具体实现影响。

在固定 Qdrant adapter 中，sync adapter `reset()` 删除并重建 collection；async Memory 路径则直接调用 `delete_col()` 后创建新 adapter。adapter 还可能使用注入的 client 或不同部署，不能把固定 Qdrant 实例的清理细节推广到所有 backend，也不能把 reset 当作删除本地数据库文件或所有外部副作用的保证。

## 清理矩阵

| 入口 | 主记录／向量 | SQLite history | 实体索引 | SQLite messages |
|---|---|---|---|---|
| `delete(id)` | 删除单个 ID | 追加 `DELETE`，保留旧正文，`is_deleted=1` | 按 scope 尝试移除关联 ID；best-effort，最多扫描 10,000 条 | 不清理 |
| 同步 `delete_all(scope)` | 按 filters 分批删除；逐条异常可中止，重复批次会正常停止并仍成功返回 | 对每条成功走到 history 的记录追加 DELETE history | 每条单独 best-effort 清理 | 不清理 |
| 异步 `delete_all(scope)` | 按 filters 分批并发删除；单条异常记录后继续，重复批次会正常停止并仍成功返回 | 对每条成功走到 history 的记录追加 DELETE history | 批次后按 filters 批量删除最多 10,000 条实体记录 | 不清理 |
| 同步 `reset()` | 调 adapter reset，或删除 collection 并重建 | SQLite reset 删除 history 表 | 若已初始化则尝试 reset；异常吞掉后丢弃引用 | SQLite reset 删除 messages 表 |
| 异步 `reset()` | 先删除 collection，再创建新 adapter | SQLite reset 删除 history 表 | 若已初始化则尝试 reset；异常吞掉后丢弃引用 | SQLite reset 删除 messages 表 |

矩阵描述固定 SDK 调用路径，不代表任意 provider、注入 client 或线上部署都具有相同底层删除和可见性语义。

## Platform 边界

固定 Mem0 commit 中的 Platform OpenAPI 声明 `delete_linked=true` 可删除 linked memories，响应可包含 `cascade_count`；同一快照的 SDK changelog 描述沿 v3 `linked_memory_ids` 的旧 superseded 链传递删除。这是公开契约与客户端说明，不是服务端实现观察。本篇未确认 `replaced_by` 与 `linked_memory_ids` 的映射、遍历方向与范围、环处理、原子性、失败残留或索引刷新；OSS 的 delete/reset 路径不能用来补齐这些未知项。

## Evidence

- **Status**：固定 Python 同步／异步调用顺序、SQLite 表操作及 Qdrant adapter 调用为 `Observed`；部分失败后可能出现的跨存储状态为 `Inferred`，未做故障复现；Platform `delete_linked` 为 `Verified`（公开契约）和 `Observed`（客户端说明），服务端执行为 `Unverified`。
- **Sources**：固定源码锚点及范围见 [`notes/source.md §10`](./notes/source.md#10-删除与重置来源重建2026-11-03)；Claim 对照见 [`notes/evidence.md`](./notes/evidence.md) M36–M39（本轮重新核对后的重建登记）。后续抽取上下文与提示词限定见该节 M39 来源及普通写入 source §6。
- **Trace**：从固定 commit 的 `Memory`、`AsyncMemory`、SQLite、vector adapter 与 entity helper 静态路径提炼。未运行真实 API、未注入故障、未删除任何外部数据。
- **Needs**：具体 backend／client 的删除可见性和故障行为；Platform `delete_linked` 服务端遍历、事务与失败语义。
