# LangGraph Architecture

> 本文件用于整理 LangGraph 的架构层次和核心对象关系
> Evidence 状态：当前已混合 `Observed` / `Inferred` / `Unverified`；稳定边界来自官方概念文档与 API reference，更深内部实现仍待源码核验

## 一、架构视角

LangGraph 可以先按四层理解：

```text
Application / Agent Logic
  → graph definition: StateGraph, nodes, edges, conditional routing
Runtime Execution
  → compiled graph, invoke / stream / batch, task scheduling, retry, interrupt
State & Persistence
  → state schema, reducers, channels, checkpointer, thread, store
Deployment & Operations
  → LangGraph Server / Platform, long-running workflow, tracing, observability
```

这只是研究用分层，不代表上游源码的模块边界。后续源码核验时需要确认这些概念实际落在哪些 package、class 和 API 中。

## 二、核心对象

待核验对象包括：

- `StateGraph`：声明 state schema、nodes、edges 和 control flow 的构建入口。
- compiled graph：运行入口，提供 invoke / stream / batch 等执行方法。
- node：接收 state 并返回 partial state、command 或 route signal。
- edge / conditional edge：定义节点间控制流。
- reducer：定义多个节点或 parallel branch 更新同一 state key 时如何合并。
- checkpointer：保存和读取 graph state checkpoint。
- thread：区分同一 graph 的不同会话 / 执行上下文。
- store：跨 thread 的长期记忆或外部状态存储。
- interrupt / command：暂停、人工介入、恢复或跳转执行。

## 三、需要拆清的边界

### 3.1 Graph definition vs Runtime execution

Graph definition 是开发者声明工作流结构；runtime execution 是 compiled graph 根据 state、edge、checkpoint 和 runtime config 执行节点。后续要避免把“图定义可持久化”误写成“执行环境可恢复”。

### 3.2 Checkpoint vs Store

Checkpoint 更偏 thread-scoped execution state；store 更偏跨 thread 的长期信息。二者都不是默认的文件系统 snapshot，也不自动回滚工具副作用。

### 3.3 Interrupt vs Sandbox pause

Interrupt 是 graph control flow 的暂停点，主要用于 human-in-the-loop、approval、state edit 和 resume；它不等于容器暂停、进程冻结或 workspace 快照。

### 3.4 Subgraph / Send vs Isolation

Subgraph 和 `Send` 可能带来并行执行与局部 state 传递，但它们的隔离边界需要源码核验：是 state namespace、task boundary、thread boundary，还是只是 execution step 的逻辑分支。

## 四、当前已稳定的架构边界

基于第一轮官方文档 / API 资料补证，当前已能稳定保留以下架构边界：

- graph state、checkpoint、interrupt、store 是不同层级对象，不应混写成一个“状态系统”。
- checkpointer 面向 thread-scoped workflow state；store 面向 cross-thread persistent memory。
- interrupt / resume 建立在 graph state persistence 之上，而不是 OS / process / container restore 之上。
- reducer 是 state merge 语义的一部分，而不是 deployment / runtime 隔离机制。

## 五、第二轮已补实的边界

基于第二轮官方文档 / API 资料补证，当前还可以稳定加入：

- runtime 采用 Pregel / superstep 执行模型，checkpoint 边界与 superstep 紧密相关。
- pending writes 属于 checkpoint / recovery 协议的一部分，而不是单纯调试输出。
- SDK / Server / Platform 是三层不同能力对象，managed checkpointer 只属于托管平台语义。
- time travel 不是单纯“回看历史”，而是包含 replay 与 fork / `update_state` 分支继续执行。

## 六、待继续补证的问题

- compiled graph 的 runtime 对象和 graph definition 对象如何分离。
- checkpointer 的接口与实现是否属于 runtime 必选依赖，还是可选配置。
- state update / reducer / channel 的真实调用顺序。
- parallel branch 的调度、异常传播和 state merge 语义。
- LangGraph Server / Platform 与本地 SDK runtime 的更细内部边界。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: 当前已结合官方概念文档、API reference 和 deployment / runtime 说明补证架构边界；更深源码核验仍待继续。
- Needs: 阅读 LangGraph 官方 docs 与源码中的 graph、pregel、checkpoint、store、types、runtime 相关模块。