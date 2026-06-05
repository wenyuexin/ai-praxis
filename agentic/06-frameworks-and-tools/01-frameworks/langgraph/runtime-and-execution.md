# LangGraph Runtime and Execution

> 本文件聚焦 LangGraph 的执行模型、并发、streaming、subgraph 与错误路径
> Evidence 状态：当前以 `Observed` 为主；`Send` / subgraph merge 路径与更深 runtime 内核细节仍保留为 `Inferred / Unverified`

## 一、研究边界

LangGraph runtime 需要回答“graph 定义之后如何实际运行”：节点如何调度、边如何触发、并行分支如何执行、streaming 如何暴露中间状态、失败如何传播，以及 checkpoint 与执行 step 如何耦合。

## 二、Observed 级运行时骨架

### 2.1 LangGraph 运行时采用 Pregel / superstep 模型

当前官方运行时文档可直接支持：

- `Pregel` 是 LangGraph 的核心运行时引擎。
- 它采用受 Google Pregel 启发的消息传递式图执行模型。
- **superstep** 是最小的原子执行单元。
- 一个 superstep 内允许多个节点并行执行。

这意味着 LangGraph 的 runtime 不是简单的“按节点逐个串行跑”，而是围绕 superstep、channel update 和节点调度组织执行。

### 2.2 Checkpoint 按 every superstep 写入

当前官方资料可直接支持：

- 当 graph 编译时配置 checkpointer，checkpoint 保存发生在 **every superstep**。
- 这不是 node-level checkpoint，也不只是 graph 退出时统一保存。
- 如果某个 superstep 中并行执行多个节点，这些节点的状态更新会在该 superstep 完成后进入 checkpoint 持久化路径。

因此，LangGraph checkpoint 与其 runtime 执行模型是强耦合的：checkpoint 边界更接近 superstep 边界，而不是函数调用边界。

### 2.3 `tasks`、`pending writes`、`channel_versions`、`versions_seen` 的角色

当前官方资料可直接支持以下运行时对象：

- `tasks`：记录在某个 checkpoint 处被调度执行的 graph nodes，包含节点身份与可能的中断信息。
- `pending writes`：failure recovery 的关键机制。当同一 superstep 中某个节点失败时，其他已成功节点产生的 writes 会被保留下来，恢复时不必全部重跑。
- `channel_versions`：跟踪各 channel 的版本。
- `versions_seen`：记录节点已见过哪些 channel 版本。

这些对象共同说明 LangGraph checkpoint 不是单纯“存一份 state 字典”，而是保存与执行推进、失败恢复和版本一致性相关的运行时结构。

### 2.4 pending writes 是 durable execution 的工程核心之一

当前官方资料可直接支持：

- pending writes 不是普通 debug 信息，而是为了 **failure recovery without re-running completed work**。
- 在同一 superstep 中，如果部分节点已成功、部分节点失败，LangGraph 会保留已成功节点的 writes。
- 后续恢复时，runtime 可以读取这些 pending writes，避免把已完成工作全部再执行一遍。

因此，pending writes 是“LangGraph 为什么能把 checkpoint 做成工程可用的 durable execution”这一问题的核心答案之一。

### 2.5 `Send` / subgraph / reducer 的并行 merge 边界

当前官方文档、reference 与本地 `pregel/` / `graph/` 源码已足以补入几条更细但仍相对稳定的边界：

- `Send` 对应的是 **dynamic fan-out**：运行时可以把工作发送给多个并行分支，而不是只靠静态 edge 预先写死所有分支。
- 在当前 OSS runtime 主路径中，`Send` 会先被写入 `TASKS` 通道，再由 `prepare_next_tasks()` 消费并展开为 push task，随后进入并行执行与后续 merge 路径。
- 这些并行分支仍运行在 LangGraph 的 superstep / synchronization 模型里：先并行推进，再把更新带回 state merge 路径。
- 当前源码可直接观察到：真正的 merge 核心落在 `apply_writes()`，它会先按 `task_path_str` 排序，再按 channel 分组，并调用对应 channel 的 `update(...)` 做 reducer 驱动的合并。
- 当多个并行节点写入同一 state key 时，必须由 **per-key reducer** 指定如何合并；否则会触发并发更新冲突，而不是由 runtime 自动猜测 merge 语义。
- subgraph 与父图共享 state keys 时，返回的 partial state 会沿父图已有的 reducer / channel 规则合并回去；如果父图与子图 schema 不同，则需要显式做状态映射。
- `Command` 与 `Send` 不应混写：`Send` 更偏运行时 fan-out，`Command(update=..., goto=...)` 更偏在单个控制点同时做 state update 与 routing。

这些结论已经足以把 `Send` / subgraph 讨论从“只有概念名词”推进到“已知 merge stop-line”；但它们仍不足以证明更深层的内部调度顺序、原子性或事务语义。

## 三、关键边界

### 3.1 并行不等于隔离

LangGraph 支持 superstep 内并行节点执行，但当前稳定证据只足以支持：

- 多个节点可并行运行。
- 并发产生的 state updates 走 per-key reducer merge 路径。

这**不等于**可以直接推出：

- 每个分支有独立 execution environment
- tool side effects 自动隔离
- 并发更新天然没有业务冲突
- `Send` / subgraph 的 merge 路径已经完全清楚

### 3.2 Streaming 不等于持久化

stream / astream 可以暴露执行过程，但当前稳定证据还不足以证明：

- 所有 stream events 都进入 checkpoint
- stream event 本身构成完整 trace persistence

因此 streaming 仍应与 checkpoint / durable execution 分开写。

### 3.3 Retry 不等于 side-effect safety

即使 runtime 支持恢复、重放或重试，也不能直接推出 node 内部外部副作用具备幂等或回滚保障。外部 API、文件写入、数据库写入的一致性仍取决于应用层设计。

## 四、仍待继续核验的问题

- `Send` 和 subgraph 的具体 merge 路径。
- pregel runtime 内部调度顺序与错误传播细节。
- stream events 与 checkpoint / trace 的精确关系。
- concurrent branch 的更深事务语义。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: LangGraph 官方 Pregel runtime 文档、CompiledGraph / checkpoint / pending writes 相关文档与 API 说明；本轮整理见 `./notes/source.md` 与 `./notes/evidence.md`。
- Trace: 为 LangGraph framework study 的 execution 层提供第一轮稳定结论，再决定是否回填环境层。
- Needs: 深读 `pregel/` 相关源码，尤其是 superstep、pending writes、subgraph / `Send` merge 路径。