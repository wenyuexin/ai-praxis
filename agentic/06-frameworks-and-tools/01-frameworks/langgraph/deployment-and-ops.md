# LangGraph Deployment and Operations

> 本文件聚焦 LangGraph Server / Platform、部署、长任务运行、可观测性与运维边界
> Evidence 状态：当前以 `Observed` 为主；多节点协调、内部后台实现与某些平台细节仍保留 `Inferred / Unverified`

## 一、为什么需要部署层专题

LangGraph 作为框架不仅包含本地 SDK，还包含面向 long-running workflow 的部署与服务化能力。对于工程使用者，checkpoint、interrupt、streaming、thread、store 最终都要落到服务运行、存储后端、API、权限和 observability 上。

## 二、Observed 级能力分层

### 2.1 SDK / Server / Platform 三层边界

当前官方资料可直接支持以下三层划分：

| 能力层 | 名称 | 职责 | Evidence |
|---|---|---|---|
| 本地 SDK | `langgraph` + checkpointers | 图构建、编译、本地执行、可插拔持久化 | Observed |
| 本地 Server | LangGraph Server | 提供托管 API、长任务运行、streaming、HITL，依赖数据库做持久化与任务队列 | Observed |
| 托管平台 | LangGraph Platform / LangSmith Deployment | 完全托管的 Cloud / BYOC / Self-Hosted 控制平面与多租户能力 | Observed |

因此，后续写作必须始终区分：哪些能力属于 SDK，哪些属于 server，哪些只属于托管平台。

### 2.2 Checkpointer 在本地 SDK 与托管平台的语义不同

当前官方资料可直接支持：

- **本地 SDK**：编译 graph 时需要显式传入 checkpointer，例如 `MemorySaver`、`PostgresSaver` 等。
- **托管平台**：checkpointing 由平台的 managed checkpointer 自动处理。
- **托管平台不支持 custom checkpointer**。

这是一条非常重要的工程边界：平台自动管理 checkpoint，不等于本地 SDK 默认具备同样的后台机制。

### 2.3 Platform / Server 提供 thread / checkpoint 管理 API

当前官方资料可直接支持以下管理能力：

- 检索特定 `thread_id` 的执行历史与 checkpoint 列表，例如 `client.threads.get_history`。
- 通过 `client.threads.update_state` 修改某个 checkpoint 处的 graph state。
- 通过 `client.runs.wait` / `client.runs.stream` 并配合 `input=None`，从指定 `checkpoint_id` 恢复执行。

这表明在托管环境中，thread / checkpoint 不是隐藏实现细节，而是被提升为显式管理对象。

### 2.4 TTL 与持久化清理是 server / platform 级能力

当前官方资料可直接支持：

- `checkpointer.ttl`：控制 checkpoint 生命周期，并关联旧 checkpoint 与 thread metadata 的保留边界。
- `store.ttl`：控制 store items 的生命周期。
- 这类 TTL 配置出现在 server / platform 的部署配置语义中，而不是本地 memory saver 的默认能力。

因此，cleanup / retention / 自动清理的写法需要严格区分部署层级。

### 2.5 同一 thread 上的并发 run 处理属于 server / platform API 语义

当前官方 reference 已足以补入一条对工程使用很重要的边界：

- 在 server / platform 的 run API 中，官方暴露了同一 `thread_id` 存在 pending / inflight run 时的 **multitask strategy** 配置。
- 可观察到的策略至少包括：拒绝新 run、排队、打断当前 run，或把 thread 回到启动前状态后再启动新 run。
- 这说明“同一 thread 上的新请求如何处理”在托管 API 语义中是显式问题，而不是完全留给用户自己猜测。

但这里必须同时保守：这条能力首先证明的是 **server / platform API 对并发 run 的处理策略存在**，而不是已经证明本地 SDK、底层 saver 或多节点部署都自动具备同等级协调机制。

## 三、长任务运行与 checkpoint 的关系

当前官方资料可直接支持：

- LangGraph Server / Platform 的一个核心价值是支持长期运行、有状态的 agent workflow。
- checkpoint / thread / store 是这类 long-running workflow 的基础设施，而不是附带功能。
- pending writes 与 resume 共同支撑失败后继续执行，而不是简单重跑全部步骤。

## 四、关键边界

### 4.1 Platform capability 不等于 SDK default capability

不要把以下能力直接写成本地 SDK 默认具备：

- managed checkpointer
- TTL 自动清理
- cron / scheduled runs
- 多租户 / BYOC / SSO
- 平台级 thread / checkpoint 管理 API

### 4.2 Observability 不等于 recovery

可观测性、thread history、run history 或 trace API 可以帮助理解执行过程，但它们不自动等于：

- checkpoint recovery
- workspace rollback
- 外部副作用回滚

### 4.3 thread 级请求处理不等于底层分布式一致性保证

即使官方 server / platform API 已暴露同一 thread 上的 multitask strategy，也仍不能直接推出：

- 多个执行实体操作同一 `thread_id` 时，底层 saver 一定具备强一致分布式锁
- 分布式多节点 checkpoint 已有完整串行化保证
- 平台内部已经把所有 thread 冲突自动消解为用户无感问题

更稳妥的写法是：官方已经把“同一 thread 的并发 run 如何处理”提升为显式 API 语义，但其更深的锁、事务、一致性与多节点协调实现细节仍未补实。

## 五、仍待继续核验的问题

- LangGraph Server 内部如何加载 TTL 配置并执行清理。
- managed checkpointer 的内部实现与自定义 saver 的真实差异。
- 多节点 / Kubernetes / task queue 与 checkpoint 协调机制。
- server / platform 在 long-running workflow、sleep、webhook、cron 上的更细行为。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: LangGraph 官方 deployment / platform / server 文档、announcement、托管能力说明与 checkpoint / thread 管理 API；本轮整理见 `./notes/source.md` 与 `./notes/evidence.md`。
- Trace: 作为 LangGraph framework study 的 ops 层专题，避免只研究本地 SDK。
- Needs: 深读 server 配置、managed checkpointer 内部实现、多节点协调与平台后端行为。