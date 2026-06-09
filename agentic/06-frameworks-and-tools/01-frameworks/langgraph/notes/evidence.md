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
| time travel 文档层语义包含 replay 与 fork；当前 OSS runtime 已能观察到 `source=fork` checkpoint 路径，但 `update_state`、同一 thread checkpoint 链与 server / platform branch 语义之间仍需分层核验。 | Observed / Unverified | 官方 time travel / fork / `update_state` 文档 + 本地 `_loop.py` / `put_writes` 路径源码 | 继续核验 server / platform 是否在 API 入口创建 fork checkpoint | replay / fork 主路径可标为 Observed；`update_state` 与 platform branch 的精确对应仍保持 Unverified，不等于 environment rollback。 |
| approval / HITL 的最小闭环是 interrupt → state edit → resume。 | Observed | 官方 interrupt / HITL / `update_state` / resume 文档 + 本地 `_runner.py` / `_loop.py` 源码 | 深读 metadata、审计与多轮 edit 边界 | 适合回填 `human-in-the-loop.md`；但 state edit 之后的 branch / metadata 语义仍需与上条冲突项分开处理。 |
| saver backend 的核心主路径可稳定写到 InMemory / SQLite / Postgres 及其 async 变体；社区 / 扩展实现应与正文主路径分开处理。 | Observed / Inferred | 官方 saver docs、reference、backend 包说明 | 深读社区/扩展 saver 的官方支持边界 | 正文优先写主路径，扩展实现先保留在线索或 notes。 |
| saver backend 的 migration、TTL 到期清理、并发 thread 协调与更深事务语义仍未补实。 | Unverified | 官方 changelog、server/platform 文档、backend 包说明 | 深读 backend 实现与事务逻辑 | 适合列为后续需要澄清的点，不适合写成完整生产保证。 |

## Workflow state / checkpoint / replay boundary pass（2026-06-09）

Version Basis：本轮源码观察基于本地 LangGraph checkout `/Users/wenyuexin/github/langgraph` 的 `libs/checkpoint/` 与 `libs/langgraph/` 当前工作区状态。
Observed At：2026-06-09。
Scope：仅覆盖 OSS `BaseCheckpointSaver`、checkpoint schema、Pregel loop / apply_writes 主路径；不覆盖 LangGraph Platform 托管后台的未公开实现。
Drift Risk：LangGraph checkpoint / delta channel / platform API 仍在演进，源码字段与官方文档口径可能继续变化。

| Claim | Status | Sources | Notes | Needs |
|---|---|---|---|---|
| 当前 OSS `Checkpoint` schema 直接包含 `v`、`id`、`ts`、`channel_values`、`channel_versions`、`versions_seen`、`updated_channels`；`CheckpointTuple` 再携带 `metadata`、`parent_config` 与 `pending_writes`。 | Observed | 本地 `libs/checkpoint/langgraph/checkpoint/base/__init__.py` | 这能补强“checkpoint 是 graph runtime state snapshot”的结论；正文中把 `next` / `tasks` 写成 checkpoint 内容时需要注明它们更接近调试 / runtime view，不应与当前 OSS `Checkpoint` TypedDict 混为一个 schema。 | 对照当前官方 docs / API reference 是否仍把 `next` / `tasks` 作为 checkpoint 展示字段。 |
| `CheckpointMetadata.source` 显式区分 `input`、`loop`、`update`、`fork`。 | Observed | 本地 `libs/checkpoint/langgraph/checkpoint/base/__init__.py` | 支持把普通运行 checkpoint、手动 state update 与 time-travel fork 分层讨论。 | 继续核验 server / platform 是否在 API 层额外包装 branch / fork metadata。 |
| `BaseCheckpointSaver` 文档字符串明确写出 `thread_id` 是保存和检索 checkpoint 的 primary key，且没有它不能保存 state、从 interrupt resume 或启用 time-travel debugging。 | Observed | 本地 `libs/checkpoint/langgraph/checkpoint/base/__init__.py` | 可把 `thread_id` 稳定写成 workflow identity / persistence key，而不是 workspace identity。 | 无需继续补证即可回填正文；更深并发协调仍另列。 |
| `Command(resume=...)` 在 Pregel loop 主路径中要求存在 checkpointer；没有 checkpointer 会抛出 runtime error。 | Observed | 本地 `libs/langgraph/langgraph/pregel/_loop.py` | 这把“resume 依赖持久化 graph state”从文档结论补强为源码路径。 | 可补入 `human-in-the-loop.md`。 |
| time-travel replay 主路径会在特定条件下写入 `source=fork` 的 checkpoint；`source=update` / `source=fork` 的路径被单独排除。 | Observed | 本地 `libs/langgraph/langgraph/pregel/_loop.py` | 这减弱了此前“文档 fork vs OSS 覆盖式写入”的简单冲突说法：源码中存在明确 fork checkpoint 路径，但 `update_state` 与平台 branch 语义仍需分层。 | 继续核验 `update_state` 入口和 platform / server API 对 branch 的定义。 |
| superstep 结束后，Pregel loop 通过 `apply_writes` 更新 channels、清空 pending writes，并以 `source=loop` 写 checkpoint。 | Observed | 本地 `libs/langgraph/langgraph/pregel/_loop.py`、`libs/langgraph/langgraph/pregel/_algo.py` | 支持“checkpoint 按 workflow superstep / graph runtime 语义推进”，不是 node 内任意语句级 checkpoint。 | 继续核验 durability mode 对保存频率的影响。 |
| `apply_writes` 会按 task path 排序、更新 `versions_seen`、按 channel 聚合 writes，并通过 channel `update()` 决定版本推进与 updated channels。 | Observed | 本地 `libs/langgraph/langgraph/pregel/_algo.py` | 这说明 replay / checkpoint 的状态推进依赖 channel / reducer 规则；不能写成无条件覆盖整个 Python 对象图。 | 对照 reducer 文档和冲突错误路径。 |
| pending writes 恢复路径会跳过 `ERROR`、`INTERRUPT`、`RESUME` 等控制写入，只把可复用成功 writes re-apply 到任务。 | Observed | 本地 `libs/langgraph/langgraph/pregel/_loop.py` | 支持“failure recovery without re-running completed work”的工程边界；不等于 side-effect rollback。 | 深读 saver backend 对 pending writes 的事务边界。 |
| 本轮源码能支持 workflow state / checkpoint / replay / fork 的 graph runtime 语义，但不能推出 workspace filesystem recovery、进程级恢复、外部副作用回滚或跨实例一致性。 | Inferred | 以上源码观察 + 现有官方 persistence / interrupt / time-travel 文档 | 这是 stop-line 归纳，应在正文中保留为边界，不写成 LangGraph 官方直接命名。 | 若要进入 `05-environments`，需再对比 workspace / sandbox 类系统。 |

## `update_state` / branch boundary pass（2026-06-09）

Version Basis：本轮源码观察基于本地 LangGraph checkout `/Users/wenyuexin/github/langgraph` 的 `libs/langgraph/`、`libs/sdk-py/` 与 `libs/sdk-js/` 当前工作区状态。
Observed At：2026-06-09。
Scope：覆盖 OSS `Pregel.update_state` / `bulk_update_state` 主路径、Python SDK `threads.update_state` REST wrapper、server runtime 类型说明和相关测试；未覆盖 LangGraph API server 私有实现。
Drift Risk：`update_state`、server runtime 和 platform branch 语义仍可能随 LangGraph API 演进。

| Claim | Status | Sources | Notes | Needs |
|---|---|---|---|---|
| `Pregel.update_state()` 是 `bulk_update_state(config, [[StateUpdate(...) ]])` 的薄封装；真正逻辑在 `bulk_update_state` / `abulk_update_state`。 | Observed | 本地 `libs/langgraph/langgraph/pregel/main.py`、`pregel/protocol.py` | 入口要求存在 checkpointer，否则抛出 `ValueError("No checkpointer set")`。 | 无需继续补证即可回填正文。 |
| `update_state` 对普通节点更新会运行该节点的 channel writers，而不是执行完整 node function 或沿 edges 继续运行 graph。 | Observed | 本地 `libs/langgraph/langgraph/pregel/main.py`、`libs/sdk-py/langgraph_sdk/runtime.py` | server runtime 文档明确 `threads.update` / `graph.aupdate_state` does not execute node functions or evaluate edges，只应用 writers、reducers 和 channel triggers。 | 可作为 HITL / approval 边界回填。 |
| 普通 `update_state` 会把 channel writes 先关联到当前 checkpoint，再通过 `apply_writes` 更新 checkpoint，最后以 metadata `source=update` 写入新 checkpoint。 | Observed | 本地 `libs/langgraph/langgraph/pregel/main.py` | 这支持“state edit 创建新的 graph state checkpoint”，但不是执行环境 snapshot。 | 继续核验不同 saver backend 的事务边界。 |
| `as_node="__copy__"` 是显式 copy / fork 路径，会以 `source=fork` 写 checkpoint；如果 values 是列表，还会在 fork 后继续调用 `perform_superstep` 做 update。 | Observed | 本地 `libs/langgraph/langgraph/pregel/main.py`、`libs/langgraph/tests/test_time_travel.py` | 测试文件把 `update_state` 后 invoke 描述为 fork，并验证多个 fork from same checkpoint 互不污染。 | 需要对照官方文档是否把这一路径暴露为公开语义。 |
| Python SDK `threads.update_state` 只向 `/threads/{thread_id}/state` POST `values`、`as_node`、`checkpoint` / `checkpoint_id`，返回 latest `checkpoint`；SDK schema 的 `ThreadState` 公开 `checkpoint` 与 `parent_checkpoint`。 | Observed | 本地 `libs/sdk-py/langgraph_sdk/_async/threads.py`、`_sync/threads.py`、`schema.py` | SDK 暴露的是 thread/checkpoint API 表面，不直接暴露名为 `branch` 的接口。 | 继续核验 API server 是否在内部把 checkpoint update 命名为 branch。 |
| server runtime 类型说明把 `threads.update` 归为 write context，但明确它不需要 execution runtime，也不应依赖外部资源初始化。 | Observed | 本地 `libs/sdk-py/langgraph_sdk/runtime.py` | 这进一步支持 `update_state` 是 graph state write，不是 run execution 或 environment operation。 | 可回填 stop-line。 |
| integration test 观察到 interrupted thread 上 `threads.update_state` 会持久化 mutation；脚本说明该操作会提交新 checkpoint 并 consume outstanding interrupt，后续 `run.respond` 会失败为 `no_such_interrupt`。 | Observed / Inferred | 本地 `libs/sdk-py/integration/scripts/test_update_state.py`、`libs/sdk-py/tests/integration/test_update_state.py` | “mutation persists” 有测试支持；“consume outstanding interrupt”来自脚本说明和集成行为描述，仍需 server 私有实现补证。 | 若写正文，避免扩大成所有 interrupt 场景通用保证。 |
| 当前证据可以把此前 `update_state` branch 张力收敛为：OSS 中同时存在 `source=update` 的 state edit checkpoint 和 `source=fork` 的 copy/time-travel checkpoint；Platform / API 文档中的 branch 更可能是基于 checkpoint / parent_checkpoint 的对外抽象，而不是单一字段。 | Inferred | 以上 OSS runtime、SDK schema、time travel tests | 这不是官方命名结论，只是当前源码边界归纳。 | 若要完全闭合，仍需 LangGraph API server 实现或官方 docs 明确说明。 |

## Out of Scope / Next

| Topic | Status | Notes |
|---|---|---|
| `Send` / subgraph 的精确 merge 路径与更深原子性 | Unverified | 已补到 `Send → TASKS → prepare_next_tasks → apply_writes` 主路径；更深调度顺序、parent/child checkpoint 原子性仍待继续源码核验 |
| 多个执行实体操作同一 `thread_id` 的协调机制 | Unverified | 当前只确认到 saver 实例级锁与 API 层 multitask strategy；底层跨进程/跨实例协调仍不能提前写成平台保证 |
| managed checkpointer 的内部实现与 server 后台细节 | Inferred / Unverified | 需继续读 server / platform 文档或源码 |
| saver backend 的 migration path、TTL 到期清理、pending writes 事务细节 | Unverified | 已确认 OSS saver 无内建 TTL/自动清理、`delete_thread` 为显式清理接口；Platform TTL 行为与事务边界仍需继续核验 |
| 社区 / 扩展 saver（如 MongoDB / Redis）的生产语义与并发行为 | Unverified | 暂不纳入正文主路径，先保留为后续澄清线索 |
| interrupted thread 上的 `update_state`、platform UI / branch 叙述与 state edit 后 metadata / trace metadata 传递 | Observed / Unverified | OSS runtime 已观察到 `source=update` 与 `source=fork` metadata 路径，SDK 已确认 thread state endpoint 形态；server 私有实现、platform UI 包装和 metadata 自动传播仍需实验或更深核验 |

## 使用说明

- 当某个 Claim 获得官方文档或源码直接支持时，可标为 `Observed`。
- 当不同版本、文档或源码之间冲突时，改为 `Conflicting`，并在 Notes 说明冲突点。
- 当某个 Claim 需要运行实验或部署验证时，保持 `Unverified` 或在 Needs 标明实验条件。
- 正文中的 `## Evidence` 应优先摘要引用本文件中的稳定结论，而不是要求读者自行从 `source.md` 还原。