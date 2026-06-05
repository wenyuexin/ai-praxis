# LangGraph Memory and Store

> 本文件聚焦 LangGraph Store、memory、cross-thread persistence 与 checkpoint 的区别
> Evidence 状态：当前以 `Observed` 为主；后端实现与高级检索能力仍保留 `Inferred / Unverified`

## 一、定位

LangGraph 中的 memory / store 与 checkpoint 容易混淆：checkpoint 关注某个 thread 的执行恢复，store 更关注跨 thread、跨会话或长期记忆。二者都属于状态持久化，但服务的问题不同。

## 二、核心对象

当前官方资料可直接支持以下对象：

- checkpointer：保存 thread-scoped workflow execution state。
- store：保存 cross-thread 可访问的数据或长期记忆。
- memory：应用层把用户偏好、事实、经验等写入 store 的实践形态。
- namespace / key：store 中组织长期记忆的基本方式。

## 三、Observed 结论

### 3.1 Checkpointer 与 Store 解决不同层级的问题

当前官方资料可直接支持：

- checkpointer 对应 thread-scoped 的短期执行状态。
- store 对应 cross-thread 的长期记忆与共享知识。
- checkpoint 与 store 在 API 层明确分离：前者围绕 `BaseCheckpointSaver`，后者围绕 `BaseStore`。

这意味着 LangGraph 从抽象层就把“执行恢复状态”和“长期记忆状态”拆成了两类对象。

### 3.2 Store 支持 cross-thread persistence

当前官方资料可直接支持：

- store 用于不同 conversation threads 之间共享数据。
- store 采用 namespace-keyed / document-style 的持久化组织方式。
- store 适合保存用户偏好、共享知识、长期上下文等不属于单个 thread 生命周期的数据。

### 3.3 Checkpoint 是 per-thread state，不是长期记忆总库

checkpoint 保存的是某个 thread 的 graph state snapshot，用于：

- durable execution
- interrupt / resume
- error recovery
- time-travel 类能力

这和 store 的长期知识、跨会话共享、全局状态并不是同一种问题。

### 3.4 Store / memory 不等于 execution trace 或 workspace recovery

当前官方资料可直接支撑以下边界：

- store 保存的是应用选择写入的长期信息。
- 它不是完整 execution trace。
- 它不是 workspace snapshot。
- 它不是 process / container / shell session 恢复机制。

因此，memory / store 的存在不能被误写成“LangGraph 可以恢复完整环境”。

## 四、关键边界

### 4.1 Store 不是 checkpoint

Store 可以保存长期信息，但它不一定记录：

- graph 当前执行进度
- pending node
- interrupt 状态
- reducer merge 历史

### 4.2 Memory 不是完整审计日志

Memory 保存的是应用主动写入的信息；没有写入的信息不会自动存在。不能把 store 中的 memory 误写成完整 trajectory、audit log 或系统级 trace。

### 4.3 Cross-thread 不等于跨环境

跨 thread 可访问的信息，不代表跨 sandbox、跨 container 或跨 workspace 自动共享文件和进程状态。

## 五、仍待继续核验的问题

- store 的 search / list / TTL / backend 细节。
- `InMemoryStore`、`PostgresStore` 或其他后端的实现差异。
- store 与 platform 托管能力之间的边界。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: LangGraph 官方 memory / store / concepts / API 参考资料；本轮整理见 `./notes/source.md` 与 `./notes/evidence.md`。
- Trace: 为区分 checkpoint、memory 和 workspace state 预留机制专题。
- Needs: 深读 store API、backend 实现、TTL 与 search 语义。