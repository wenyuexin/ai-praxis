# LangGraph Human-in-the-loop and Interrupt

> 本文件聚焦 LangGraph interrupt、resume、approval、time travel 与人工介入边界
> Evidence 状态：当前以 `Observed` 为主；更深的平台 UI / metadata / 外部副作用一致性仍保留 `Inferred / Unverified`

## 一、为什么单列

LangGraph 的 interrupt / resume 是它区别于一次性 workflow runner 的重要能力。对 agent 系统来说，human-in-the-loop 不只是“暂停一下”，还涉及状态保存、人工审阅、state 修改、继续执行和审计追踪。

## 二、核心对象

当前官方文档与 API 资料可直接支持以下对象：

- `interrupt()`：在 node 内部动态触发中断。
- `interrupt_before` / `interrupt_after`：在编译时对指定节点设置静态中断点。
- `Command(resume=...)`：恢复 graph 执行。
- `update_state`：直接修改某个 checkpoint 处的 graph state。
- time travel：从历史 checkpoint replay 或 fork 新分支继续执行。
- checkpointer：中断恢复和 time travel 所依赖的持久化基础。

## 三、Observed 结论

### 3.1 interrupt 是 graph control flow 语义

当前官方资料直接把 interrupt 描述为：

- 暂停 graph execution
- 向客户端返回 payload
- 让外部输入或人工决策后再继续执行

这说明 interrupt 的定义范围处在 **graph control flow 层**，而不是 OS / process / container 管理层。

### 3.2 interrupt 不等于 process freeze / container pause

当前官方资料可直接支持以下边界：

- interrupt 不是类似 `input()` 的阻塞式等待。
- graph 执行不会保持一个被冻结的进程原地等待输入。
- 恢复时需要重新调用 graph，并依赖 checkpoint 中保存的状态继续执行。

因此，interrupt / resume 不应被写成 process freeze、container pause 或 workspace snapshot restore。

### 3.3 resume 通过 `Command(resume=...)` 完成，并从 node 开头重放

当前官方资料可直接支持：

- 恢复依赖 `Command(resume=value)`。
- `interrupt(...)` 返回值与 `resume` 传入值对应。
- graph 恢复时会从中断节点开头重新执行，而不是从某条 Python 语句的机器级暂停点继续。
- 如果同一 node 中有多个 interrupt，resume 值会按调用顺序匹配。

这进一步说明 LangGraph 的 resume 是 **graph step replay / continuation**，不是进程级恢复。

### 3.4 interrupt 依赖 checkpointer

当前官方资料明确要求：使用 interrupt 前必须启用 checkpointer。原因是 interrupt / resume 依赖持久化 graph state，而不是依赖内存中保留一个阻塞执行体。

### 3.5 动态 interrupt 与静态 breakpoint 是两类机制

当前资料可直接区分：

- `interrupt_before` / `interrupt_after`：编译时静态设置的 breakpoint。
- `interrupt()`：node 内部动态、可条件化触发的中断。

### 3.6 time travel = replay + fork

当前官方资料可直接支持：

- **Replay**：从先前 checkpoint 重新执行，不是只读回看缓存。
- **Fork**：从先前 checkpoint 出发，用修改后的 state 创建新的执行分支。

这意味着 time travel 不是单一语义，而是“历史重放 + 分支继续执行”的组合能力。

### 3.7 `update_state` / fork 的语义需要分层表述

当前材料已经能稳定支持以下较保守的写法：

- `update_state` 的 first-class use case 是修改 checkpoint 处的 graph state。
- 官方文档层把它描述为修改 state 后创建新的 branch checkpoint，并继续执行。
- 但当前 OSS runtime 主路径源码中，`update_state` 更接近在原 thread 的 checkpoint 链上做覆盖式写入，而不是显式新建 `checkpoint_id` / `thread_id`。

因此，这里应保持 stop-line：state edit / fork 更接近“从旧状态出发派生或继续执行”的 time-travel 语义，而不是环境回滚；但“branch checkpoint”与“覆盖式写入”之间的精确对应关系，当前仍需继续结合 server / platform 层核验。

### 3.8 approval / HITL 的最小闭环是 interrupt → state edit → resume

结合当前官方文档、API 说明与本地 runtime 源码，已经可以把 approval / human-in-the-loop 的最小流程写得更具体：

- 先通过 `interrupt()` 或 `interrupt_before` / `interrupt_after` 把 graph 停在一个可审阅的 checkpoint。
- runtime 会把 `INTERRUPT` 信号写入相应的 checkpoint / pending write 路径，使当前 state 对后续读取可见。
- 人可以在该 checkpoint 处检查 state，并通过 `update_state` 修改 graph state。
- 然后再通过 `Command(resume=...)` 或对应的 server / platform run API 恢复执行。
- 在当前 OSS runtime 主路径中，还可以直接观察到恢复阶段会跳过 `ERROR` / `INTERRUPT` 等特殊写入，并对可复用的成功 writes 做 re-apply。

这说明 LangGraph 的 approval 不只是“暂停等待确认”，还包含对 graph state 的显式编辑能力；但这类编辑仍然只作用于 graph state snapshot，而不是自动修复外部系统状态。

## 四、关键边界

### 4.1 Human approval 不自动保证 side-effect safety

在工具调用前 interrupt 可以让人审批，但审批后执行的外部副作用是否可回滚、是否幂等，仍取决于工具实现和应用层设计。

### 4.2 Time travel 不是完整环境回滚

即使 time travel 基于 checkpoint 恢复 graph state，也不默认回滚：

- 外部 API 调用
- 数据库写入
- 文件系统修改
- 其他未进入 graph state 的副作用

### 4.3 state edit 只作用于 graph state snapshot

`update_state` 修改的是 graph state snapshot 中的字段，不等于修改外部环境或自动修复外部系统不一致。

## 五、仍待继续核验的问题

- state edit 后 system metadata、trace metadata 如何传递。
- 平台 UI / approval API 与本地 SDK 的细节差异。
- 多轮 edit / fork / continue 的最佳实践与限制。
- 外部系统副作用不一致时是否有官方推荐处理模式。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: LangGraph 官方 interrupt / human-in-the-loop / time travel / `update_state` / API 参考资料；本轮整理见 `./notes/source.md` 与 `./notes/evidence.md`。
- Trace: 用于连接 LangGraph framework study 与环境层 recovery / governance 问题。
- Needs: 深读 fork branch、state metadata、approval UI / platform 语义与 side-effect consistency 指南。