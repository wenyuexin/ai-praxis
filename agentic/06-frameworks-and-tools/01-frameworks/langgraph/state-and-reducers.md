# LangGraph State and Reducers

> 本文件聚焦 LangGraph 的 state schema、channels、reducers 与状态合并边界
> Evidence 状态：当前以 `Observed` 为主；更深运行时细节仍保留 `Inferred / Unverified`

## 一、为什么单列

LangGraph 的核心不是普通 DAG，而是围绕 shared state 运行的 graph runtime。state 如何定义、节点如何更新 state、并行分支如何合并 state，是理解 checkpoint、interrupt、human-in-the-loop 和 durable execution 的前提。

## 二、核心对象

当前官方文档与 API 资料可直接支持以下对象：

- state schema：定义 graph 的共享状态结构。
- node：读取当前 state，返回 partial update。
- reducer：多个节点更新同一 state key 时的合并规则。
- channels：runtime 按 state key 组织状态通道和值。
- partial update：节点只返回需要更新的字段子集。

## 三、Observed 结论

### 3.1 StateGraph 节点签名是 `State -> Partial<State>`

当前官方资料明确把 LangGraph 的节点定义为：

- 读取当前 `State`
- 返回 `Partial<State>`

这意味着节点默认不是直接替换整个状态对象，而是提交局部更新，由 runtime 按字段合并进全量 state。

### 3.2 Reducer 是按字段的 merge 机制

官方资料明确说明：

- 每个 state key 都可以配置 reducer。
- reducer 的职责是聚合同一 key 的多个更新值。
- reducer 函数在 Python 和 TypeScript 中都以“旧值 + 新值 → 合并后值”为基本签名。
- 内置的 `add_messages` 就是一个典型 reducer，用于把新增消息合并进已有消息列表。

因此，reducer 的真实作用是 **per-field merge policy**，不是泛泛的“状态合并概念”。

### 3.3 没有 reducer 的并发更新会触发冲突

当前官方资料可直接支持：当多个节点并发返回同一 state key 的更新，而该 key 没有合适 reducer 时，会出现 `INVALID_CONCURRENT_GRAPH_UPDATE` 之类的冲突错误。

这说明 reducer 不只是语法糖，而是 LangGraph 并发 state update 语义中的关键约束。

### 3.4 TypeScript Annotation 系统把 schema、default、reducer 绑定在一起

当前资料可直接支持：在 TypeScript 中，`Annotation.Root()` 用于定义 state schema；每个 field 可以配置：

- reducer
- default
- 对应 state / update 类型推导

这表明 LangGraph 在 JS/TS 生态中把 reducer 设计成 schema definition 的一部分，而不是后置补丁。

## 四、关键边界

### 4.1 State 是 graph runtime state，不是 workspace

LangGraph state 是 graph runtime 的数据结构。即使 state 中包含文件路径、工具调用记录或 message history，也不能自动推出它保存了：

- workspace 文件内容
- 进程状态
- 外部服务状态
- execution environment 的完整快照

### 4.2 Reducer 是 merge policy，不是 conflict-free guarantee

Reducer 可以定义如何合并值，但它不自动保证：

- 业务层没有冲突
- 外部副作用可回滚
- 多次 retry 一定幂等
- 并发写入一定符合预期语义

因此 reducer 解决的是 state merge 语义，不是所有并发一致性问题。

### 4.3 Message history 不是完整 trace

message state 可以保存 agent / chat history，但不等于完整 execution trace。工具副作用、文件系统变化、外部 API 调用仍需单独证据链。

## 五、仍待继续核验的问题

- channels 在 runtime 内部的精确数据结构与更新顺序。
- invalid update 的完整错误路径与类型校验细节。
- failed node 的 partial state 是否会进入 checkpoint。
- state update 与 checkpoint 写入的精确先后顺序。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: LangGraph 官方 low-level concepts、StateGraph API 参考、TypeScript `Annotation` 与 reducer 文档；本轮整理见 `./notes/source.md` 与 `./notes/evidence.md`。
- Trace: 为 checkpoint / persistence 研究提供 state 层前置概念。
- Needs: 深读 channels、invalid update、message reducer 和 runtime merge 细节。