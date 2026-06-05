# LangGraph Evidence Notes

> 本文件用于整理 LangGraph framework study 的 Claim / Status / Sources / Needs 对照，服务正文中的 Evidence 段落
> 当前状态：已建立第一轮 Claim Tracker，并补入第二轮 runtime / deployment / time-travel / saver backend 线索

## Claim Tracker

| Claim | Status | Sources | Needs | Notes |
|---|---|---|---|---|
| LangGraph checkpoint 保存 graph runtime state，而不是默认 workspace filesystem snapshot。 | Observed | 官方 persistence / checkpoint 文档、checkpoint 结构与 saver API | 深读 saver 实现与序列化边界 | 当前可稳定回填正文。 |
| `thread_id` 通过 `configurable` 传递，是保存与读取 checkpoint 的关键标识。 | Observed | 官方 persistence / thread / config 文档 | 深读 `checkpoint_id` 与 thread 历史管理 | 更接近 workflow identity，而非 sandbox identity。 |
| checkpoint / durable execution 解决的是 workflow 恢复，而不是 OS / process / container 恢复。 | Observed | 官方 durable execution 与 interrupt 文档 | 深读 side-effect recovery 与 idempotency 指南 | 可与 OpenHands / SWE-agent 做恢复语义对照。 |
| LangGraph 节点签名是 `State -> Partial<State>`，reducer 是 per-key merge policy。 | Observed | 官方 `StateGraph` / reducers / Annotation 文档 | 深读 channels 与 invalid update 细节 | 当前已足以支撑 state/reducer 正文。 |
| 没有合适 reducer 时，并发更新同一 state key 会产生冲突。 | Observed | 官方 reducers / error 文档 | 核验更多并行 branch 错误路径 | 是 parallel update 研究入口。 |
| `interrupt()` / `Command(resume=...)` 属于 graph control flow 语义，不等于 process freeze 或 container pause。 | Observed | 官方 interrupt / human-in-the-loop 文档与 API reference | 深读 time travel 与 state edit | 当前已足以支撑 HIL 正文。 |
| interrupt 依赖 checkpointer；恢复时从中断 node 开头继续重放。 | Observed | 官方 interrupt / checkpoint 文档 | 深读多次 interrupt 的匹配逻辑 | 不应误写成原地继续执行同一进程。 |
| store / memory 与 checkpoint 解决的是不同层级的状态持久化问题。 | Observed | 官方 memory / store concepts 与 `BaseStore` reference | 深读 backend、TTL、search 与 namespace 语义 | 当前可稳定回填正文。 |
| store 提供 cross-thread persistent memory，但不等于完整 execution trace 或 workspace recovery。 | Observed | 官方 memory / store 文档 | 深读 platform 托管与 search 行为 | 防止误写成完整环境恢复。 |
| LangGraph runtime 采用 Pregel / superstep 模型，checkpoint 按 every superstep 写入。 | Observed | 官方 Pregel runtime / CompiledGraph / checkpoint 文档 | 深读 pregel 源码与 subgraph / `Send` merge 路径 | 当前可稳定回填 `runtime-and-execution.md`。 |
| pending writes 是 failure recovery 的核心机制之一，用于避免重跑已完成 work。 | Observed | 官方 checkpoint / pending writes 文档 + 本地 `pregel/_loop.py` / saver 实现源码 | 深读 saver backend 事务处理 | 不等于完整 trace persistence；源码侧还能直接看到 `PendingWrite` 结构、`put_writes()`、`after_tick()` 清空与 `_reapply_writes_to_succeeded_nodes()` 的恢复路径。 |
| `Send` 对应 dynamic fan-out；并行分支对同一 state key 的 merge 依赖 per-key reducer。 | Observed | 官方 `Send` / subgraph / reducer / error 文档 + 本地 `pregel/_write.py` / `_algo.py` / `_runner.py` 源码 | 深读 superstep 内部调度顺序与更深 merge 原子性 | 当前足以回填 merge stop-line；源码侧已能直接定位 `Send → TASKS channel → prepare_next_tasks → apply_writes` 主路径，但不宜外推内部事务语义。 |
| subgraph 与父图共享 keys 时沿既有 reducer / channel 规则合并；schema 不同则需显式状态映射。 | Observed | 官方 subgraph 文档与示例 + 本地 `_utils.py` / `_algo.py` / `graph/state.py` 源码 | 深读 parent / child checkpoint 原子性 | 适合补入 runtime / state 相关正文；更深 parent/child checkpoint 原子性仍未补实。 |
| OSS saver 主路径（InMemory / SQLite / Postgres 及 async 变体）都显式实现 `put`、`put_writes`、`get_tuple`、`list`、`delete_thread`。 | Observed | 本地 `BaseCheckpointSaver`、memory/sqlite/postgres saver 源码 | 深读扩展 saver 与 backend 差异 | 适合补强 persistence 接口边界；不要外推为社区/扩展 saver 的统一语义。 |
| 当前 OSS saver 主路径可直接观察到实例级锁、幂等写入路径与无内建 TTL/自动清理；这些现象不等于跨进程并发安全或整个平台缺失 TTL。 | Observed / Inferred | 本地 sqlite/postgres saver 源码 + server/platform 文档 | 深读跨进程并发、Platform TTL 行为与事务边界 | 其中“实例级锁存在”“OSS saver 无内建 TTL/自动清理”可记为 Observed；跨进程安全、Platform 行为和生产影响仍应保留为 Inferred / Unverified。 |
| LangGraph Platform / LangSmith Deployment 使用 managed checkpointer，本地 SDK 需显式传入 checkpointer。 | Observed | 官方 deployment / platform / managed checkpointer 文档 | 深读后台自动处理实现 | 不要混写平台能力与 SDK 默认能力。 |
| server / platform run API 显式暴露同一 `thread_id` 上 pending / inflight run 的 multitask strategy。 | Observed | 官方 SDK / server / platform run API reference | 深读本地 SDK 支持范围与后台实现 | 证明 thread-level run handling 存在，但不等于已验证底层分布式一致性保证。 |
| time travel 文档层语义包含 replay 与 fork；但 `update_state` 在 OSS runtime 主路径中表现为原 thread 上的覆盖式写入，两层口径当前存在张力。 | Conflicting | 官方 time travel / fork / `update_state` 文档 + 本地 `_loop.py` / `put_writes` 路径源码 | 继续核验 server / platform 是否在 API 入口创建 fork checkpoint | 不等于 environment rollback；当前不宜把“创建新 branch”或“纯覆盖式写入”任何一侧单独写成 LangGraph 全局定论。 |
| approval / HITL 的最小闭环是 interrupt → state edit → resume。 | Observed | 官方 interrupt / HITL / `update_state` / resume 文档 + 本地 `_runner.py` / `_loop.py` 源码 | 深读 metadata、审计与多轮 edit 边界 | 适合回填 `human-in-the-loop.md`；但 state edit 之后的 branch / metadata 语义仍需与上条冲突项分开处理。 |
| saver backend 的核心主路径可稳定写到 InMemory / SQLite / Postgres 及其 async 变体；社区 / 扩展实现应与正文主路径分开处理。 | Observed / Inferred | 官方 saver docs、reference、backend 包说明 | 深读社区/扩展 saver 的官方支持边界 | 正文优先写主路径，扩展实现先保留在线索或 notes。 |
| saver backend 的 migration、TTL 到期清理、并发 thread 协调与更深事务语义仍未补实。 | Unverified | 官方 changelog、server/platform 文档、backend 包说明 | 深读 backend 实现与事务逻辑 | 适合列为后续需要澄清的点，不适合写成完整生产保证。 |

## Out of Scope / Next

| Topic | Status | Notes |
|---|---|---|
| `Send` / subgraph 的精确 merge 路径与更深原子性 | Unverified | 已补到 `Send → TASKS → prepare_next_tasks → apply_writes` 主路径；更深调度顺序、parent/child checkpoint 原子性仍待继续源码核验 |
| 多个执行实体操作同一 `thread_id` 的协调机制 | Unverified | 当前只确认到 saver 实例级锁与 API 层 multitask strategy；底层跨进程/跨实例协调仍不能提前写成平台保证 |
| managed checkpointer 的内部实现与 server 后台细节 | Inferred / Unverified | 需继续读 server / platform 文档或源码 |
| saver backend 的 migration path、TTL 到期清理、pending writes 事务细节 | Unverified | 已确认 OSS saver 无内建 TTL/自动清理、`delete_thread` 为显式清理接口；Platform TTL 行为与事务边界仍需继续核验 |
| 社区 / 扩展 saver（如 MongoDB / Redis）的生产语义与并发行为 | Unverified | 暂不纳入正文主路径，先保留为后续澄清线索 |
| `update_state` 的 branch 语义、state edit 后 metadata / trace metadata 传递 | Conflicting / Unverified | 文档层与 OSS runtime 实现层对 branch 语义存在张力；metadata 自动传播仍需实验或更深 server/platform 核验 |

## 使用说明

- 当某个 Claim 获得官方文档或源码直接支持时，可标为 `Observed`。
- 当不同版本、文档或源码之间冲突时，改为 `Conflicting`，并在 Notes 说明冲突点。
- 当某个 Claim 需要运行实验或部署验证时，保持 `Unverified` 或在 Needs 标明实验条件。
- 正文中的 `## Evidence` 应优先摘要引用本文件中的稳定结论，而不是要求读者自行从 `source.md` 还原。