# LangGraph Checkpoint and Persistence

> 本文件聚焦 LangGraph checkpoint、thread、persistence 与 recovery 语义
> Evidence 状态：当前以 `Observed` 为主；更深 saver backend、pending writes 实现细节与并发 thread 协调仍保留为 `Inferred / Unverified`

## 一、定位

Checkpoint / persistence 是 LangGraph 对环境层研究最有价值的机制：它提供 graph state 的持久化与恢复能力，但这个恢复能力默认属于 workflow runtime 层，不应直接等同于 workspace filesystem recovery、container resume 或 process checkpoint。

## 二、核心对象与接口

本文件优先回答：

- checkpoint 保存哪些 graph state。
- checkpoint 与 `thread_id` / `checkpoint_id` 的关系。
- checkpointer 的职责和基础接口边界。
- checkpoint 在 durable execution 中如何支撑 interrupt / resume。
- 为什么 checkpoint 不是 workspace / process / container snapshot。

### 2.1 Checkpoint 保存的内容

当前官方文档与 API 资料可直接支持的结论是：checkpoint 是 **graph state 在特定时间点的 snapshot**。其中至少包括：

- `channel_values`：状态通道的值。
- `channel_versions`：各状态通道的版本。
- `versions_seen`：节点已见过的版本信息。
- `next`：下一步待执行的节点。
- `tasks`：与该 checkpoint 关联的任务信息。

这说明 checkpoint 保存的是 graph runtime 的结构化状态，而不是文件系统、容器或进程级快照。

### 2.2 thread / thread_id / checkpoint_id

当前官方文档与 API 资料表明：

- **thread** 是一组 checkpoint 的逻辑归属对象。
- 使用 checkpointer 时，必须在 `configurable` 中提供 `thread_id`，如 `{"configurable": {"thread_id": "1"}}`。
- `thread_id` 是保存和检索 checkpoint 的主键。
- `checkpoint_id` 是可选字段，用于在某个 thread 的历史 checkpoint 中定位一个具体状态，并从中间点继续执行。

因此，`thread_id` 更接近 workflow identity，而不是 sandbox identity 或 workspace identity。

### 2.3 Checkpointer 的职责边界

Checkpointer 是 LangGraph persistence layer 的实现组件，遵循 `BaseCheckpointSaver` 抽象接口。当前可直接观察的核心方法包括：

- `put`：保存 checkpoint。
- `put_writes`：保存 pending writes。
- `get_tuple`：按 thread / checkpoint 读取 checkpoint。
- `list`：列出可用 checkpoint。
- `delete_thread`：删除某个 thread 关联的 checkpoint。

当前官方资料中可观察到的核心主路径实现类型包括：

- `InMemorySaver`
- `SqliteSaver`
- `AsyncSqliteSaver`
- `PostgresSaver`
- `AsyncPostgresSaver`

在当前整理范围内，这些是最适合稳定写入正文的 saver / checkpointer 主路径对象；其他社区或扩展实现若缺少同等级官方文档与 reference 支撑，更适合先保留在线索或 notes 层。

这说明 LangGraph 已把 checkpoint persistence 明确抽象成独立组件，而不是隐含在 graph definition 或 node logic 中。

### 2.4 `BaseCheckpointSaver` 的最小接口语义

结合当前官方文档 / reference，可把几个核心方法的最小职责边界写得更清楚：

- `put`：保存某个 checkpoint 的完整状态锚点，包括 configuration、metadata 与该时刻的 graph state snapshot。
- `put_writes`：保存与某个 checkpoint 关联的 **intermediate writes / pending writes**，它们是完整 checkpoint 之外的增量恢复信息。
- `get_tuple`：按给定 configuration 读取某个 checkpoint tuple，而不是只读一份裸 state。
- `list`：按 thread / filter 枚举可恢复的 checkpoint 历史。
- `delete_thread`：删除某个 thread 关联的数据，是 thread 级清理接口，而不是单个 node 级 API。

这里最重要的边界是：checkpoint saver 不只是“存一个 state blob”，而是同时承担 checkpoint 历史、pending writes 与 thread 级检索 / 清理的 persistence 协议角色。

### 2.5 pending writes 的接口位置

当前官方资料与本地 saver / runtime 源码可直接支持：

- `put_writes` 是 `BaseCheckpointSaver` 的核心接口之一。
- pending writes 不是普通附属字段，而是 checkpoint persistence 协议中的一部分。
- 它服务于 failure recovery，而不是完整 trace persistence。
- 在当前 OSS runtime 主路径中，可以直接观察到 `PendingWrite` 结构、`PregelLoop.put_writes()` 的追加与持久化路径、`after_tick()` 中的清空，以及 `_reapply_writes_to_succeeded_nodes()` 在恢复时对成功 writes 的重放逻辑。
- 这说明 pending writes 不只是概念说明，而是 runtime / saver 共同参与的恢复中间层。

### 2.6 TTL / thread 删除 / state edit 的稳定边界

当前可稳定写入正文的只是较窄的边界：

- `delete_thread` 已经进入 checkpointer / saver 的基础接口语义，说明 thread 级清理是 persistence 层显式考虑的问题。
- TTL 配置当前更稳定地出现在 server / platform 层，而不是本地 SDK 的通用默认能力。
- `update_state` / state edit 的精确持久化语义需要分层表述：官方文档层把它描述为修改 state 后创建新的 branch checkpoint，而当前 OSS runtime 主路径源码更接近在原 thread 的 checkpoint 链上做覆盖式写入。

但这里仍要保持 stop-line：TTL 到期是否自动触发 `delete_thread`、不同 saver 的清理策略是否一致，以及 `update_state` 的 branch checkpoint 语义是否依赖 server / platform 包装，当前都还不适合写成定论。

## 三、Observed 边界

### 3.1 Checkpoint 保存 graph state，不默认保存文件系统

当前最稳定的结论是：LangGraph checkpoint 保存的是 graph state snapshot，而不是默认的 workspace filesystem snapshot。直接依据包括：

- 官方文档将 checkpoint 定义为某个 thread 在给定时间点的状态快照。
- checkpoint 结构围绕 `channel_values`、`channel_versions`、`versions_seen`、`next`、`tasks` 组织。
- 这些字段都属于 graph runtime state，而不是 OS / process / container / filesystem 层。

因此，即使 checkpoint 能恢复 workflow execution，也不能直接推出以下状态被保存：

- workspace 文件内容
- 容器 / 进程 / shell session
- 外部数据库或 API 副作用
- 未显式进入 state / store 的工具内部缓存

### 3.2 Durable execution 是 workflow 层恢复

官方 durable execution 语义可直接支持以下判断：

- LangGraph 通过 checkpoint 持久化 graph state，使 workflow 在系统故障或 human-in-the-loop 中断后可以从最后记录的状态恢复。
- 这种恢复依赖 graph state persistence，而不是 OS 级 checkpoint / restore。
- interrupt / resume、error recovery 与 durable execution 是同一 persistence 语义链上的能力。

### 3.3 pending writes 支撑 failure recovery

当前官方资料可直接支持：

- 在某个 superstep 中，如果部分节点已成功、部分节点失败，成功节点的 writes 可作为 pending writes 被保留。
- 后续恢复时，LangGraph 可以利用 pending writes 避免把已完成工作全部重跑。

这说明 pending writes 是 durable execution 的工程核心之一，但它仍不等于完整 execution trace 或事务型 environment rollback。

### 3.4 Checkpoint 不等于环境恢复

当前官方资料虽未逐条写出“不会恢复容器/文件系统”，但它对 checkpoint 的定义始终限定在 graph state snapshot 层。这足以支撑一个稳定的 `Observed` 级边界：**LangGraph checkpoint 的官方定义范围没有扩展到 execution environment 层。**

## 四、仍保持保守的点

以下内容当前仍不应写成定论：

- checkpoint 存储的 `channel_values` 是否覆盖所有运行时对象，尤其是不可序列化对象、闭包或临时资源句柄。
- 不同 saver（memory / SQLite / Postgres）在 pending writes、cleanup、TTL、migration 上的完整差异。
- durable execution 对外部工具副作用、幂等与回滚的一致性保证。
- 多个执行实体并发操作同一 `thread_id` 时的协调语义。
- `update_state` 在文档层的 branch checkpoint 语义与 OSS runtime 主路径中的覆盖式写入之间的对应关系。
- state edit 后 metadata / trace metadata 的传递规则。

## 五、与 05-environments 的关系

后续如果回填 `05-environments/code-execution-environments/workspace-checkpoint.md`，只能小范围引用稳定结论：

- graph state checkpoint 与 workspace checkpoint 是不同层级。
- workflow resume 与 execution environment resume 是不同语义。
- checkpoint 不自动解决外部副作用、文件系统变化和资源 cleanup。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: LangGraph 官方 persistence / checkpoint / durable execution 文档；`BaseCheckpointSaver`、checkpoint 结构、pending writes 与 saver API 参考资料；本轮整理见 `./notes/source.md` 与 `./notes/evidence.md`。
- Trace: 从 `05-environments/candidates.md` 的 LangGraph checkpoint 候选进入 framework study，再决定是否回填环境层。
- Needs: 深读 saver 实现细节、pending writes、TTL、并发 thread 协调与 schema evolution。