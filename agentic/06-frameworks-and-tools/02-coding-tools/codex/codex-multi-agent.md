# Codex 多智能体边界核验

> **专题文档**。本文只整理目前能从公开产品页面与本地源码 `D:\github\codex` 中保守确认的多智能体相关事实，并明确哪些说法仍不宜写成定论。

> 截至当前，`subagent`、子线程生成、父子线程关系、并发线程上限、linked worktree 场景处理，都已有较明确锚点；但 `Manager` 调度器、`TaskGraph`、`TaskChannel`、冲突仲裁器、以及“每个 Subagent 自动创建 git worktree”之类说法，仍不应写成源码级事实。

---

## 一、当前可直接核验的事实

### 1.1 `subagent` 在协议层是显式概念

在本地源码中，`SessionSource`、`ThreadSource`、`SubAgentSource` 都是明确类型，而不是仅存在于产品文案里的抽象说法。

| 事实 | 一手锚点 |
|------|----------|
| 会话来源可标记为 `SubAgent(...)` | `D:\github\codex\codex-rs\protocol\src\protocol.rs` |
| 线程来源存在 `ThreadSource::Subagent` | `D:\github\codex\codex-rs\protocol\src\protocol.rs` |
| `SubAgentSource` 至少包含 `Review`、`Compact`、`ThreadSpawn`、`MemoryConsolidation` | `D:\github\codex\codex-rs\protocol\src\protocol.rs` |

这意味着：**“子代理 / 子线程”在 Codex 源码里是一级对象**，而不只是社区对并行执行现象的命名。

### 1.2 子线程生成机制可以直接看到

`codex-rs/core/src/agent/control.rs` 中可以直接看到：父线程会为新 agent 预留并发槽位，并在 `SubAgentSource::ThreadSpawn` 场景下生成带来源信息的新线程。

可直接引用的关键点包括：

- `spawn_agent_with_metadata(...)`
- `spawn_agent_internal(...)`
- `SessionSource::SubAgent(SubAgentSource::ThreadSpawn { ... })`
- `state.spawn_new_thread_with_source(...)`
- `thread_source: Some(ThreadSource::Subagent)`

因此，把 Codex 的一部分多智能体能力理解为**父线程拉起子线程执行子任务**，是有源码锚点支撑的。

### 1.3 delegate 路径中存在真实的异步转发机制

`codex-rs/core/src/codex_delegate.rs` 提供了更具体的子线程委托入口：

- `run_codex_thread_interactive(...)`
- `run_codex_thread_one_shot(...)`
- `async_channel::bounded(...)`
- `tokio::spawn(...)`
- `forward_events(...)`
- `forward_ops(...)`

从这里至少能保守确认两件事：

1. 父会话确实可以启动一个 `subagent` 子线程。
2. 子线程与父侧之间存在事件与操作的异步转发链路。

但这还**不足以推出**一套完整且稳定命名的“Manager → TaskGraph → TaskChannel → Subagent”内部架构。

### 1.4 系统对 `subagent` 生命周期有正式埋点

`codex-rs/analytics/src/events.rs` 中存在：

- `subagent_thread_started_event_request(...)`
- `subagent_source_name(...)`
- `subagent_parent_thread_id(...)`

这些符号说明：

- `subagent` 启动会被显式记录；
- 父线程 ID 与 `subagent_source` 会进入事件模型；
- `thread_spawn` 不是完全隐含的内部细节，而是被系统性追踪的对象。

### 1.5 并发线程上限是正式配置项

`codex-rs/core/config.schema.json` 中存在 `max_concurrent_threads_per_session`。

这能支撑一个较保守的判断：**Codex 至少在会话级别暴露了并发子线程数量的控制面**。这比“理论上可并行”更强，也比“任意多 Agent DAG 调度”更保守。

### 1.6 linked worktree 是明确考虑过的场景

`codex-rs/app-server/README.md` 中可以直接看到对 **linked Git worktrees** 的说明：同一项目的 hooks 定义以 root checkout 的 `.codex/` 为准，而不是以某个 linked worktree 的局部定义为准。

这能确认：

- 仓库明确支持或至少明确处理 linked worktree 场景；
- worktree 并非完全游离于 Codex 能力边界之外。

但这仍然**不能直接证明**：每个 `subagent` 都自动通过 `git worktree` 创建独立目录。

---

## 二、目前可保守成立的结构性理解

### 2.1 更接近“父线程 + 子线程委托”而不是强行写死 `Manager` 类型

如果要对当前源码做最保守的抽象，一个更稳妥的说法是：

- 父线程会在特定来源下拉起 `subagent` 子线程；
- 子线程带有来源、父线程关系、角色元数据等信息；
- 父侧与子侧之间存在事件转发、操作转发与审批路由；
- 并发度会受到会话级配置约束。

这个抽象已经足够解释公开产品中“并行线程”“子代理”“线程来源”等现象，而不必提前发明一个源码里未明确出现的 `Manager` 总控类型。

### 2.2 多智能体能力更像“线程化委托 + 会话并发控制”

结合 `control.rs`、`codex_delegate.rs`、`protocol.rs`、`config.schema.json`，当前更稳的理解是：

- Codex 至少具备把任务分派到子线程的能力；
- 子线程不是匿名 worker，而是带 `subagent source` 的正式对象；
- 并行能力首先体现为**每会话可并发的线程数**，而不是公开可见的 DAG 调度系统；
- 审批与部分事件会回到父会话处理，而不是完全由子线程自治。

这类判断可以写成 `Inferred` 或“候选解释”，但比旧版文档中的通道图更贴近现有源码。

---

## 三、当前未找到的关键说法

下列说法在本地仓库 `D:\github\codex` 中**未找到足够强的一手源码锚点**，因此不宜继续写成正文定论：

| 不宜写死的说法 | 当前状态 |
|----------------|----------|
| 存在明确的 `Manager` 多智能体调度核心类型 | 未找到 |
| 存在 `TaskGraph` 数据结构 | 未找到 |
| 存在 `TaskChannel<T>` 作为固定通道抽象 | 未找到 |
| `Manager → Subagent` 主要通过 `Tokio mpsc` 通信 | 未找到直接证据 |
| `Subagent` 结果通过 `oneshot` 回传 | 未找到直接证据 |
| 存在独立的冲突仲裁器 / arbitration 模块 | 未找到 |
| 每个 `Subagent` 都自动创建独立 `git worktree` | 未找到直接实现证据 |
| 存在稳定公开的 Automations 核心实现模块 | 本地源码中未见清晰对象级锚点 |

这并不意味着这些能力绝对不存在，而是表示：**在当前证据状态下，正文不该把它们写成已经核实的内部实现。**

---

## 四、Worktree 与 Automations 的边界

### 4.1 Worktree：可以确认“支持场景”，不能确认“固定隔离机制”

结合官方产品页面与本地源码，目前最稳妥的边界是：

- 官方产品叙述明确出现过 `Worktree` 与并行线程协作；
- 本地源码与 app-server 文档也明确考虑 linked worktree 场景；
- 但尚不足以把“每个 Subagent = 一个独立 git worktree”写成稳定实现事实。

因此，关于 worktree 的更稳写法应是：**Codex 产品层明确支持 worktree 场景，但子代理与 worktree 的一一对应关系仍待补证。**

### 4.2 Automations：更适合继续停留在产品层描述

本地仓库里能看到与 `automations` 相关的标签或外围痕迹，但尚未找到足够清晰、适合在本专题中当作内部多智能体核心机制引用的稳定模块。

所以在 `codex-multi-agent.md` 中，更合适的做法是：

- 保留“官方产品层存在 Automations 能力”的说法；
- 不把 Automations 直接等同为某个已核实的多智能体调度内核；
- 若后续要继续深挖，应另行在 Automations 专题中补对象级锚点。

---

## 五、当前可接受的一句话总结

> 截至当前，更稳妥的说法是：Codex 已明确具备 `subagent` 子线程、父子线程来源建模、异步委托与会话级并发控制等多智能体相关能力；但 `Manager`、`TaskGraph`、冲突仲裁、以及“每个子代理固定对应一个 worktree”等更细粒度机制，仍不应写成已核实事实。

---

## Evidence

- Status: Mixed
- Sources: OpenAI 官方产品页面；本地源码仓库 `D:\github\codex` 中的 `codex-rs/core/src/agent/control.rs`、`codex-rs/core/src/codex_delegate.rs`、`codex-rs/protocol/src/protocol.rs`、`codex-rs/analytics/src/events.rs`、`codex-rs/core/config.schema.json`、`codex-rs/app-server/README.md`。
- Trace: 本文已从“社区推断型架构图”收缩为“源码可核验事实 + 保守解释 + 未找到项”三层结构；凡是未在本地仓库中找到稳定符号的内部机制，一律不再写成定论。
- Needs: 若后续继续补证，可进一步核验 `spawn_agent` 周边测试、线程上限配置的实际生效路径、以及 linked worktree 与产品 Worktree 能力之间是否存在更直接的源码连接。

## 参考来源

- `D:\github\codex\codex-rs\core\src\agent\control.rs`
- `D:\github\codex\codex-rs\core\src\codex_delegate.rs`
- `D:\github\codex\codex-rs\protocol\src\protocol.rs`
- `D:\github\codex\codex-rs\analytics\src\events.rs`
- `D:\github\codex\codex-rs\core\config.schema.json`
- `D:\github\codex\codex-rs\app-server\README.md`
- https://developers.openai.com/codex
- https://developers.openai.com/codex/app/features

*最后更新: 2026-06-04*
