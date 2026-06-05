# LangGraph Source Notes

> 本文件记录 LangGraph framework study 的源码入口、官方文档入口、长引用线索、失败搜索与局部核验
> 当前状态：已建立第一轮来源记录，并补入第二轮 runtime / deployment 深挖线索

## 一、第一轮优先资料入口

本轮主要使用的官方资料入口包括：

- LangGraph persistence / checkpoint / thread 相关官方文档
- LangGraph durable execution 官方文档
- LangGraph interrupt / human-in-the-loop 官方文档与 API reference
- LangGraph memory / store 官方 concepts 与 API reference
- LangGraph Python / TypeScript 的 `StateGraph`、reducers、annotation 相关 API reference
- `BaseCheckpointSaver`、checkpoint schema 与 saver 实现相关资料

## 二、来源记录

建议后续继续按同样格式补充。

- 资料：LangGraph persistence / checkpoint 官方文档
- 路径 / URL：官方 LangGraph docs 中 persistence / checkpoint 页面
- 观察：checkpoint 被定义为 thread 在某一时刻的 graph state snapshot；与 `thread_id` / `checkpoint_id` 绑定
- 对应正文：`checkpoint-and-persistence.md`
- Status：Observed
- Notes：重点支持“checkpoint ≠ workspace snapshot”边界

- 资料：LangGraph durable execution 官方文档
- 路径 / URL：官方 LangGraph docs 中 durable execution 页面
- 观察：durable execution 依赖 checkpoint persistence，在系统故障或 HIL 中断后从最后记录状态恢复
- 对应正文：`checkpoint-and-persistence.md`、`human-in-the-loop.md`
- Status：Observed
- Notes：不要外推成 OS / process 恢复

- 资料：LangGraph interrupt / human-in-the-loop 文档与 interrupt API reference
- 路径 / URL：官方 LangGraph docs 与 reference 中 interrupt 页面
- 观察：`interrupt()` 暂停 graph execution，`Command(resume=...)` 恢复，且恢复时从 node 开头重放
- 对应正文：`human-in-the-loop.md`
- Status：Observed
- Notes：支持“graph control flow ≠ process freeze”边界

- 资料：LangGraph `StateGraph` / reducers / TypeScript annotation 参考资料
- 路径 / URL：官方 Python / TypeScript reference 中 `StateGraph`、`Annotation`、reducers 页面
- 观察：节点签名为 `State -> Partial<State>`；reducer 是 per-key merge policy；并发更新同一 key 且无 reducer 时会冲突
- 对应正文：`state-and-reducers.md`
- Status：Observed
- Notes：支持 state merge 与 reducer 机制

- 资料：LangGraph memory / store concepts 与 `BaseStore` reference
- 路径 / URL：官方 concepts / reference 中 memory 与 store 页面
- 观察：checkpointer 对应 thread-scoped execution state；store 对应 cross-thread persistent memory；二者 API 抽象分离
- 对应正文：`memory-and-store.md`
- Status：Observed
- Notes：支持“store ≠ checkpoint”边界

## 三、第二轮深挖入口

- 资料：Pregel runtime / CompiledGraph 官方文档
- 路径 / URL：官方 LangGraph runtime / Pregel / CompiledGraph 页面
- 观察：runtime 采用 Pregel / superstep 模型，checkpoint 按 every superstep 写入
- 对应正文：`runtime-and-execution.md`
- Status：Observed
- Notes：支撑 runtime 层第一轮稳定结论

- 资料：Checkpoint API 参考（含 pending writes）
- 路径 / URL：官方 checkpoint API reference
- 观察：`tasks`、`pending writes`、`channel_versions`、`versions_seen` 都是 checkpoint / recovery 语义的一部分
- 对应正文：`runtime-and-execution.md`、`checkpoint-and-persistence.md`
- Status：Observed
- Notes：pending writes 是 failure recovery 关键机制

- 资料：LangGraph deployment / platform / server 文档与 announcement
- 路径 / URL：官方 deployment / platform / LangSmith Deployment 文档与公告
- 观察：SDK / Server / Platform 三层能力分离；托管平台使用 managed checkpointer，不支持 custom checkpointer
- 对应正文：`deployment-and-ops.md`
- Status：Observed
- Notes：不要混写平台能力和本地 SDK 能力

- 资料：Time Travel / `update_state` 官方文档
- 路径 / URL：官方 time travel、fork、state update 页面
- 观察：time travel 包含 replay 与 fork；`update_state` 修改 checkpoint 处的 graph state 并创建新 branch
- 对应正文：`human-in-the-loop.md`
- Status：Observed
- Notes：不等于 environment rollback

- 资料：Saver packages / v0.2 changelog / backend docs
- 路径 / URL：官方 checkpointer 包、change log、backend 文档
- 观察：InMemory / SQLite / Postgres saver 及 async 变体存在；`checkpoint_id` / `parent_checkpoint_id` 为新版字段；TTL 主要是 server / platform 层配置能力
- 对应正文：`checkpoint-and-persistence.md`、`deployment-and-ops.md`
- Status：Observed / Inferred
- Notes：migration path 与并发 thread 协调仍待深读源码

## 四、失败搜索与边界

当前已知需要继续留意：

- 某些部署 / platform / agent server 语义更多出现在产品文档而不是本地 SDK 文档中。
- JS/TS 与 Python 在 API 命名和 schema 表达上有细微差异，正文需避免混淆。
- 非官方镜像站、第三方博客、下游包说明只能当线索，不能当主证据。
- `Send` / subgraph 的 merge 路径、并发同一 `thread_id` 的协调机制仍需更深源码核验。

## 五、提醒

- 本文件可以保留过程材料与长摘录，但不要直接把这些内容当作正文结论。
- 从本文件回填正文时，需要补 Evidence 状态和最小 Trace。