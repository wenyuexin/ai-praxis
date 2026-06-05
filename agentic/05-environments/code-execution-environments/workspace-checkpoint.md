# Workspace Checkpoint

> 适用范围：Agent 代码执行环境中的 workspace snapshot、checkpoint、rollback 与恢复路径
> 阶段状态：研究主干，持续补充框架案例与恢复机制证据
> 使用说明：本文件回答“workspace 为什么需要 checkpoint、checkpoint 针对什么状态生效、常见恢复路径有哪些、核心矛盾是什么”，不直接预设某一种实现方案为标准答案

## 一、定位

如果说 `workspace-structure.md` 讨论的是 workspace 是什么，那么本文件讨论的是：**workspace 状态如何被保存、恢复、比较与回退**。

在 Agent 的代码执行环境里，checkpoint 不应被狭义理解为“把目录打个压缩包”或“把容器做个快照”。

它更适合被理解为：**对任务工作域在某个决策点上的可恢复状态描述**。

这个定义里最重要的不是“存成什么格式”，而是两个问题：

- checkpoint 针对的是哪一组状态
- 恢复时系统到底要回到什么粒度的过去

因此，workspace checkpoint 是连接执行连续性、失败恢复与审计能力的关键专题，而不是附属于文件系统实现的小细节。

---

## 二、为什么它必须单列成主题

### 2.1 workspace 连续性不等于可恢复性

很多系统允许 agent 在同一 workspace 中连续修改代码、运行测试和生成 artifact，但“状态一直存在”并不等于“系统能够可靠恢复”。

连续性只说明历史状态还在累积；checkpoint 才回答：

从当前已核验的 OpenHands / `software-agent-sdk` 交叉证据看，还需要再补一道区分：**会话可恢复性也不等于文件系统可恢复性**。

也就是说，一个系统可能已经支持：

- 重新连回同一 `conversation_id` / `sandbox_id`
- 重新获取 `conversation_url` / `session_api_key`
- 对同一 runtime / container / remote session 执行 `resume`

但这仍不足以直接证明：

- 底层 `working_dir` 可以被独立重建
- 文件状态能仅靠 persisted events / trajectory / logical metadata 稳定恢复
- 没有原 runtime 连续性时仍能复原同一任务工作域

因此，workspace checkpoint 讨论里必须把“恢复连接”“恢复执行身份”“恢复文件状态”拆开。

- 如果某一步引入错误，回退到哪里
- 如果执行中断，恢复从哪里继续
- 如果多个分支尝试并行展开，如何保留可比较的中间状态

所以，workspace 是状态载体，checkpoint 才是状态治理机制。

### 2.2 它不应等同于容器快照

容器快照、文件系统 snapshot、Git commit、事件日志 replay 都可能参与恢复，但它们不是同一个概念。

checkpoint 真正关注的是：**对当前任务来说，哪些状态足以支持可靠恢复**。

这意味着：

- 有时只需要文件状态 checkpoint
- 有时还需要命令输出、测试结果、artifact metadata
- 有时还要记录任务步骤、依赖上下文和人工介入痕迹

如果把 checkpoint 直接写成“容器层快照”，会过早把概念绑死在某类基础设施上。

### 2.3 它天然连接 rollback、recovery 与 traceability

checkpoint 不是为了“存档”而存在，而是为了支持后续动作：

- rollback：回到更早状态
- recovery：从失败中恢复执行
- comparison：比较不同尝试分支
- audit：解释某个结果来自哪个状态

因此，workspace checkpoint 不只是恢复层话题，也会影响 trace、artifact attribution 和任务治理。

---

## 三、checkpoint 到底在保存什么

### 3.1 文件状态

最直接的一层是 workspace 中文件的可恢复状态，例如：

- 源代码内容
- 新增、修改、删除的文件
- 配置与脚本
- 任务生成的中间产物

这是很多 checkpoint 讨论最容易看到的一层，但不是全部。

### 3.2 任务状态指针

仅保存文件内容并不足以支持长链任务恢复。

很多场景还需要知道：

当前 OpenHands / `software-agent-sdk` 交叉源码正好提供了一个很典型的例子：系统可以显式保存或恢复 `conversation_id`、`sandbox_id`、`conversation_url`、`session_api_key`、持久化 conversation 目录等“任务状态指针”，从而让会话重新可寻址、可重连、可 resume；但这些指针本身并不自动等于完整文件快照。

- 当前执行到哪个步骤
- 上一个成功节点是什么
- 哪些 artifact 属于本轮尝试
- 哪个 checkpoint 是推荐恢复点

因此 checkpoint 往往不只是状态本体，也包括状态索引与恢复入口。

### 3.3 执行结果与环境上下文

如果一次失败与测试、命令输出或依赖变化有关，恢复往往需要更多上下文，例如：

- 最近一次测试结果
- 命令执行日志摘要
- 关键环境变量或依赖版本信息
- 与当前 checkpoint 绑定的 trace / artifact metadata

这说明 checkpoint 的讨论不能完全脱离 observability。

OpenHands 的调度风险也提示，checkpoint / recovery 的工程成本不能只看“状态是否保存下来”。如果 event persistence 需要全量读取、过滤、排序，或者恢复路径必须等待 sandbox resume、workspace backend 健康检查与远端命令轮询，那么恢复点即使存在，也可能因为读取、重连和验证链路过长而难以形成低延迟的可用恢复能力。

### 3.4 人工介入与决策痕迹

在 human-in-the-loop 的系统里，人工批准、手动修复、选择分支等动作也可能改变“可恢复状态”的含义。

如果恢复后丢失这些信息，系统虽然回到了旧文件，但不一定回到了同一个任务语义状态。

LangGraph 的第一轮与第二轮补证又提供了一个很好的反例边界：它已经能稳定支持 thread-scoped graph state checkpoint、durable execution、interrupt / resume、time travel replay / fork 与 `update_state` 分支继续执行；但这些能力保存和恢复的是 graph runtime state，而不是默认的 workspace 文件系统快照。也就是说，系统完全可能已经具备“workflow state recovery”，却仍不具备“workspace file recovery”。这进一步说明，本专题必须把 graph state、task control state、workspace file state 和 execution environment state 分层讨论，而不能把所有“checkpoint”写成一类东西。

---

## 四、常见 checkpoint 组织路径

### 4.1 Full Workspace Snapshot

对整个 workspace 在某一时刻做完整快照。

优点：

- 概念直观
- 恢复语义清晰
- 适合高确定性回滚

代价：

- 存储成本更高
- checkpoint 频率受限
- 对大型代码库与高频操作不够轻量

### 4.2 Incremental / Diff-Based Checkpoint

只保存相对前一状态的差异。

优点：

- 更节省存储
- 适合高频 checkpoint
- 更利于比较不同尝试路径

代价：

- 恢复链路更复杂
- 长链 diff 更容易导致回放成本上升
- 对一致性要求更高

### 4.3 Logical Checkpoint

不直接完整保存底层状态，而是保存“足以重建任务状态”的逻辑描述，例如：

这里尤其要警惕一种常见误写：把“支持 resume / restore 的逻辑元数据”直接写成“已经足以重建 workspace 文件系统”。当前 OpenHands / `software-agent-sdk` 的源码更适合支撑一种较弱但更准确的表述：它已经证明了 conversation restore、workspace backend pause/resume、agent-server conversation persistence 等逻辑恢复能力存在；但尚未证明这些逻辑元数据在脱离原 runtime / volume / working_dir 连续性的情况下，足以独立重建完整任务工作域。

- 当前基线版本
- 已应用 patch 集合
- 关键 artifact 列表
- 执行步骤与恢复入口

优点：

- 更抽象
- 更适合跨 runtime 或跨 sandbox 场景
- 更贴近任务语义

代价：

- 依赖重建过程正确
- 恢复确定性可能弱于物理快照
- 更容易受外部环境漂移影响

### 4.4 Hybrid Model

组合使用物理快照与逻辑元数据。

典型方式包括：

- 文件快照 + trace metadata
- 基线 snapshot + 后续 diff
- 只对关键阶段做 full snapshot，其余阶段记录 logical checkpoint

这类设计通常更符合真实系统，但也更考验边界定义。

---

## 五、它与 rollback / recovery 的关系

### 5.1 checkpoint 不等于 rollback

checkpoint 是被回退或恢复所引用的状态锚点；rollback 是一种动作。

也就是说：

- 没有 checkpoint，rollback 很难可靠进行
- 有 checkpoint，也不代表只能做简单回退

系统还可能基于 checkpoint 做分支恢复、重试或人工接管。

### 5.2 rollback 关注“退回去”，recovery 关注“继续做”

两者相关但不相同：

- rollback 更偏把 workspace 退回某个较早状态
- recovery 更偏在故障、中断或错误后恢复任务推进能力

因此一个系统可以：

- 先 rollback 到稳定 checkpoint
- 再从该 checkpoint 继续执行后续步骤

### 5.3 checkpoint 粒度会直接塑造恢复路径

如果 checkpoint 粒度过粗：

- 恢复成本更高
- 有效中间状态更少
- 人工排障时回退跨度更大

如果粒度过细：

- 管理成本更高
- 状态树更复杂
- 容易产生大量“技术上存在、语义上无意义”的恢复点

所以问题不只是“有没有 checkpoint”，而是“checkpoint 如何对齐任务语义”。

---

## 六、最重要的结构性矛盾

### 6.1 Recovery Fidelity vs Checkpoint Cost

恢复越精确，通常越需要记录更多状态；记录越多，存储、计算与管理成本越高。

这是一条典型 trade-off，没有免费解。

### 6.2 Frequent Checkpointing vs Execution Throughput

高频 checkpoint 能提升回退粒度与容错能力，但会打断执行流、增加 I/O 与状态管理开销。

### 6.3 Physical Snapshot vs Logical Reconstructability

物理快照恢复更直接，逻辑重建更灵活；前者偏确定性，后者偏可移植性与抽象表达。

### 6.4 State Completeness vs Boundary Clarity

记录的状态越完整，恢复与审计越强；但如果把日志、权限、会话、文件、缓存、依赖全部塞进 checkpoint，概念边界就会变得模糊。

所以必须持续回答：哪些属于 workspace checkpoint 的核心状态，哪些只是关联元数据。

---

## 七、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- checkpoint 就是文件系统快照
- rollback 的合理实现只有一种
- 只要保留 Git diff，就足以支持任务恢复
- checkpoint 越频繁越好
- workspace checkpoint 只属于 recovery，不涉及 traceability 和 audit

这些说法都会把状态恢复问题过度简化，掩盖任务语义、执行边界与可审计性的差异。

---

## 八、与其他专题的关系

- 与 `workspace-structure.md`：前者定义 workspace 是任务工作域；本文件进一步讨论这个工作域如何被保存与恢复。
- 与 `../overview.md`：本文件展开其中 `checkpoint / snapshot / rollback / recovery` 这一组核心子主题。
- 与 `../conflict.md`：对应 `Rollback 的合理路径是否唯一` 这一核心冲突，也补充 `Workspace 是否等于 Sandbox 内文件系统路径` 的恢复侧含义。
- 与后续 `workspace-traceability.md`：本文件只讨论 checkpoint 与恢复；后者继续讨论日志、轨迹、artifact 归因和审计边界。
- 与 `../sandboxing-and-safety/sandbox-layers.md`：checkpoint 更偏 `state / recovery layer`，但其实现常与 execution isolation 耦合，二者需要保持概念区分。

---

## 九、当前最值得继续补证的方向

- 主流 agent 系统的 checkpoint 触发时机：按步骤、按风险、按人工确认，还是按失败后补救
- full snapshot、diff checkpoint、logical checkpoint 三类模型在代码任务中的恢复成本比较
- checkpoint metadata 是否需要显式纳入 artifact、trace 与任务状态指针
- workspace checkpoint 与容器 snapshot、Git patch、event replay 的边界该如何清晰拆分
- 多 agent / 多 subtask 场景下 checkpoint 是否应共享基线、独立分支，还是分层组合
- “恢复连接 / 恢复会话 / 恢复 sandbox 身份” 与 “恢复文件系统状态” 在真实系统里是否经常分离，哪些系统提供前者但不提供后者
