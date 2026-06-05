# LangGraph Overview

> 本文件是 LangGraph framework study 的对象总览，先建立研究地图，不替代更深源码核验
> Evidence 状态：当前已混合 `Observed` / `Inferred` / `Unverified`；稳定结论来自两轮官方文档 / API reference 补证，更深 runtime / deployment 细节仍待继续核验

## 一、定位

LangGraph 是 LangChain 生态中的 graph-based agent workflow framework / runtime，核心价值不只是“把节点连成图”，而是把 **state、checkpoint、interrupt、resume、streaming、subgraph、store 与 deployment** 组合成可持久运行的 agent workflow。

本目录把 LangGraph 当作重要框架对象研究，而不是只当作 `05-environments` 的 checkpoint 证据。它需要同时回答：

- 开发者如何用 LangGraph 建模 agent workflow。
- LangGraph runtime 如何管理 graph state、checkpoint、thread 与 resume。
- 它的持久化与 human-in-the-loop 能力边界在哪里。
- 它与 OpenHands / SWE-agent 这类 code agent runtime 的状态管理差异是什么。

## 二、研究主线

当前目录按以下主线组织：

1. **架构与抽象层**：StateGraph、CompiledGraph、node、edge、subgraph、command、send。
2. **状态模型**：state schema、channels、reducers、message state、state merge。
3. **持久化与恢复**：checkpointer、thread、checkpoint namespace、store、durable execution。
4. **运行与并发**：invoke / stream / batch、parallel branch、Send API、retry、error path。
5. **人工介入**：interrupt、resume、approval、time travel、修改 state 后继续执行。
6. **部署与运维**：LangGraph Server / Platform、deployment API、long-running workflow、observability。

## 三、当前已稳定的第一轮结论

当前已通过第一轮官方文档 / API 资料补证，拿到几条可稳定使用的边界：

- checkpoint 保存的是 graph runtime state snapshot，而不是默认的 workspace filesystem snapshot。
- `thread_id` 是 thread-scoped checkpoint 的关键标识，更接近 workflow identity，而不是 sandbox identity。
- `interrupt()` / `Command(resume=...)` 属于 graph control flow 语义，不等于 process freeze / container pause。
- LangGraph 节点默认是 `State -> Partial<State>`，reducer 是 per-key merge policy。
- store / memory 与 checkpoint 解决的是不同层级的状态持久化问题：前者偏 cross-thread persistent memory，后者偏 per-thread durable execution。

这些结论当前仍主要建立在官方文档、API reference 和概念说明上；但 runtime / deployment 第二轮深挖已进一步补入：Pregel / superstep 模型、every superstep checkpoint、pending writes、SDK / Server / Platform 三层边界、managed checkpointer，以及 replay / fork / `update_state` 的 time travel 语义。最近一轮保守回填又把几条更细的 stop-line 写实了：`Send` / subgraph / reducer 的 merge 边界、`BaseCheckpointSaver` 的最小接口语义、approval / HITL 的 `interrupt → state edit → resume` 闭环，以及 server / platform run API 上同一 `thread_id` 的 multitask strategy 边界。更深的 saver backend、并发协调与 merge 路径仍待继续核验。

## 四、当前仍需保守处理的点

以下问题仍不应写成框架定论：

- `Send` / subgraph 的精确 merge 路径与更深原子性语义。
- 多个执行实体操作同一 `thread_id` 时，API 层 multitask strategy 之下的底层协调机制。
- managed checkpointer 的内部实现。
- saver backend 的 migration path、pending writes 事务细节。
- `update_state` 在文档层的 branch 语义与 OSS runtime 主路径中的覆盖式写入张力，以及 state edit 后 metadata / trace metadata 的传递。

## 五、与环境层研究的关系

LangGraph 对 `05-environments` 的价值主要在于校正“状态恢复”语义：

- LangGraph checkpoint 保存的是 graph runtime state，而不是默认保存 workspace 文件系统。
- thread / checkpoint 可以支持 workflow resume，但不等于 sandbox / container / process resume。
- store / memory 可以保存跨 thread 信息，但不等于 execution environment 的外部副作用回滚。
- interrupt / resume 是 graph control flow 语义，不能直接等同 OpenHands 的 sandbox pause / resume。

这些结论当前已可作为第一批稳定研究边界；但若要继续上升为更细的工程定论，仍需要补 runtime / server / saver backend 相关源码核验。

## 六、当前不覆盖

本目录暂不系统覆盖：

- LangChain 全生态的 model I/O、tool registry、prompt template。
- 具体业务 agent template 的 prompt 和策略细节。
- LangSmith tracing 的完整产品能力，除非与 LangGraph observability / deployment 直接相关。
- 把 LangGraph checkpoint 直接写成 workspace filesystem recovery。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: 当前已结合三轮官方文档 / API reference 导向的补证与保守回填，覆盖 checkpoint / thread / reducer / interrupt / time travel / deployment / platform，并补入 `Send` / subgraph merge stop-line、`BaseCheckpointSaver` 最小接口语义、approval 闭环与 thread-level multitask strategy 边界；更深源码核验仍待继续。
- Trace: 从 framework study 角度扩展 `agentic/06-frameworks-and-tools/01-frameworks/langgraph/`，后续稳定结论再小范围回流 `05-environments`。
- Needs: 继续深读 runtime / pregel、server / platform、saver backend 与并发协调相关源码，尤其是 merge 原子性、底层锁/事务与 metadata 传递。
