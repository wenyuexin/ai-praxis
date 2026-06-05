# LangGraph

LangGraph 是一个基于图编排的 Agent workflow framework / runtime，由 LangChain 团队开发。本目录把 LangGraph 作为重要框架对象系统研究，重点关注它如何组织 graph state、checkpoint / persistence、interrupt / resume、parallel execution、memory / store、deployment / ops，以及这些机制与 `05-environments` 中 workspace lifecycle、checkpoint、recovery 边界的对照关系。

## 研究边界

本目录优先回答：

- LangGraph 的 `StateGraph`、compiled graph、node / edge / subgraph 等核心抽象如何组织 agent workflow。
- graph state、channels、reducers 与 message state 如何定义和合并。
- `checkpoint` / `checkpointer` / `thread` 的生命周期：检查点保存什么、何时创建、如何恢复、过期策略和清理语义。
- `interrupt` / `interrupt_before` / `interrupt_after`：暂停与恢复的控制语义，与 sandbox / workspace lifecycle 的 pause/resume 有何异同。
- `Send` API、subgraph 和 parallel branch 的执行与状态隔离边界。
- store / memory 与 checkpoint 的区别：长期记忆、cross-thread persistence 和 execution recovery 分别解决什么问题。
- LangGraph Server / Platform 的部署、长任务运行、可观测性与运维边界。

不在本目录：

- 泛化 LangChain 生态（Model I/O、Chains、Memory、Tools 等已有 `langchain-agents/` 单独覆盖）。
- LangGraph 的 LLM 调用策略、prompt 模板设计或具体 Agent 实现模式。
- 把 graph state checkpoint 外推为完整 workspace 文件系统恢复定论。

## 当前阶段

- **阶段一：完整目录骨架**：已建立。当前先按重要框架对象预留对象总览、架构、state、checkpoint、runtime、human-in-the-loop、memory/store、deployment/ops 等机制专题。
- **阶段二：第一轮证据补强**：已完成。`checkpoint-and-persistence.md`、`state-and-reducers.md`、`human-in-the-loop.md`、`memory-and-store.md` 已拿到第一批 `Observed` 级稳定结论。
- **阶段三：runtime / deployment 第二轮深挖**：已完成。`runtime-and-execution.md`、`deployment-and-ops.md` 已补入 Pregel / superstep、pending writes、SDK / Server / Platform 边界、managed checkpointer、time travel / fork / `update_state` 等稳定结论。
- **阶段四：第三轮深水区边界收紧**：已完成一轮保守回填。当前已把 `Send` / subgraph / reducer 的 merge stop-line、`BaseCheckpointSaver` 的最小接口语义、approval/HITL 的 `interrupt → state edit → resume` 闭环，以及 server / platform run API 上同一 `thread_id` 的 multitask strategy 边界写回正文与 `notes/evidence.md`。
- **阶段五：继续深水区源码核验**：待开始。后续优先核验 `Send` / subgraph 的精确 merge 路径、同一 `thread_id` 并发协调的底层实现、saver backend 的 transaction / migration / pending writes 细节，以及 state edit 后 metadata / trace metadata 的传递。

## 目录结构

```text
langgraph/
├── notes/
├── README.md
├── overview.md
├── architecture.md
├── state-and-reducers.md
├── checkpoint-and-persistence.md
├── runtime-and-execution.md
├── human-in-the-loop.md
├── memory-and-store.md
└── deployment-and-ops.md
```

## 阅读入口

推荐阅读顺序：

1. `overview.md`：建立 LangGraph framework study 的整体地图。
2. `architecture.md`：理解核心对象、分层和研究边界。
3. `state-and-reducers.md`：先理解 state / reducer，再进入 checkpoint。
4. `checkpoint-and-persistence.md`：聚焦 checkpoint、thread、persistence 与 recovery 语义。
5. `runtime-and-execution.md`：理解 invoke / stream / batch、parallel branch、subgraph 和错误路径。
6. `human-in-the-loop.md`：理解 interrupt / resume / time travel 的控制语义。
7. `memory-and-store.md`：区分 store / memory 与 checkpoint。
8. `deployment-and-ops.md`：补本地 SDK 之外的 server / platform / ops 边界。

`notes/` 作为研究辅助材料层保留，但不是主要阅读入口；只有在需要追溯源码摘录、长引用、失败搜索或 Claim 对照时再进入。

## 与 05-environments 的对照价值

LangGraph 的 checkpoint 机制提供了一个与 OpenHands / SWE-agent 不同的状态管理样本：

- OpenHands 围绕 conversation / sandbox identity 做长期可恢复会话，checkpoint 隐含在事件持久化和 sandbox resume 中。
- SWE-agent 围绕 per-instance repo reset + trajectory，明确不支持 workspace level checkpoint / resume。
- LangGraph 在 graph state 层面提供显式 checkpoint / persistence API，但 graph state checkpoint 不应直接写成 workspace filesystem checkpoint。

因此，LangGraph 适合用来拆清：workflow state recovery、execution environment recovery、workspace file recovery、external side-effect recovery 这几层恢复语义。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: 已完成前三轮官方文档 / API reference 导向的补证与保守回填，涵盖 persistence / checkpoint / thread / reducer / interrupt / time travel / deployment / platform，以及 `Send` / subgraph merge stop-line、`BaseCheckpointSaver` 最小接口语义、approval 闭环与 thread-level multitask strategy 边界；更深源码核验仍待继续。
- Trace: 从 `05-environments/candidates.md` 的 LangGraph checkpoint 条目出发，建立独立的 langgraph framework study 入口；后续稳定结论再小范围回填环境层专题。
- Needs: 继续深读 runtime / pregel、server / platform、saver backend 与并发协调相关源码，尤其是 merge 原子性、底层锁/事务与 metadata 传递。
