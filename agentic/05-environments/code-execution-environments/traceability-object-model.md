# Traceability Object Model

> 适用范围：Agent 代码执行环境中的 traceability 最小必要对象集、对象关系与归因结构
> 阶段状态：研究主干，持续补充产品案例与对象建模证据
> 使用说明：本文件回答“workspace traceability 到底应把哪些实体当作一等对象、这些对象之间如何关联、为什么不能只保存事件日志、核心 trade-off 是什么”，不直接预设某一种 tracing 存储或实现框架为标准答案

## 一、定位

如果说 `workspace-traceability.md` 讨论的是 traceability 为什么重要、常见组织视角有哪些，那么本文件进一步讨论的是：**为了让 traceability 真正支持归因、恢复、审计与排障，系统至少应把哪些对象显式建模**。

在 Agent 的代码执行环境里，traceability 不应被狭义理解为“保留一串事件”或“把命令日志落盘”。

更准确地说，它需要一个**对象模型**：用一组稳定实体和它们之间的关系，把动作、状态、产物和任务语义绑定起来。

这个定义里最重要的不是“数据存在哪里”，而是三个问题：

- 哪些实体必须被视为 traceability 的一等对象
- 对象之间最关键的关系是什么
- 哪些对象是核心状态，哪些只是派生视图或展示层拼装结果

因此，traceability object model 不是一个纯实现细节，而是 workspace traceability 能否形成可解释结构的基础。

---

## 二、为什么它必须单列成主题

### 2.1 只有事件流，通常不足以支持稳定归因

单纯的 command log、tool log 或 event stream 可以回答“发生过什么”，但很难稳定回答：

- 某个文件改动究竟来自哪一步动作
- 某个 artifact 属于哪次任务尝试
- 某个失败应归因到哪个状态、哪个工具或哪个子任务
- rollback 后哪些结果仍有效，哪些应该失效

这意味着 traceability 不能只围绕“事件”建模，还必须围绕对象与对象关系建模。

### 2.2 对象模型决定后续很多能力是否成立

traceability object model 一旦太弱，后续很多能力都会变得脆弱或依赖临时推断：

- artifact attribution
- checkpoint relevance
- retry / replay comparison
- auditability
- memory writeback
- strategy revision

所以对象模型不是为了“数据库设计好看”，而是为了让这些能力拥有稳定支点。

### 2.3 不单列它，workspace traceability 很容易停留在视角讨论

`workspace-traceability.md` 已经建立了 event-first、artifact-centric、task-centric、hybrid 四种视角。

但如果没有对象模型层的补充，这些仍然更像“观察视角”，而不是“系统应该至少记录哪些实体”。

因此需要把“视角”进一步下沉为“对象集 + 关系集”。

---

## 三、最小必要对象集应该包含什么

这里给出的不是唯一理论标准，而是当前更适合 Agent 代码执行环境的工作性最小集合。

### 3.1 Task

`task` 是最上层的归属对象。

它回答：

- 当前 trace 归属于哪个任务
- 当前执行目标是什么
- 哪些步骤、子任务、workspace 和 artifact 属于同一个任务边界

如果没有 task，trace 容易退化成全局事件池，缺少语义归属。

### 3.2 Step / Attempt

`step` 或 `attempt` 负责表达任务内部的局部推进单元。

它回答：

- 某次动作属于哪个任务步骤
- 某个结果是第一次尝试、重试还是分支尝试
- 某段 trace 应与哪个 checkpoint 或人工确认节点对齐

对于多轮修改、失败重试或多 agent 协作场景，这个对象通常不可缺。

### 3.3 Action

`action` 是最接近执行面的对象，表示一次明确发生的动作，例如：

- 命令执行
- tool call
- 文件读写请求
- 网络访问请求
- 子任务触发

它回答“做了什么”，但不自动回答“影响了什么”。

因此 action 很重要，但它不是唯一核心对象。

### 3.4 Workspace State / Checkpoint

traceability 需要有稳定的状态锚点，否则动作与结果很难绑定到“当时的环境”。

因此通常需要把以下至少之一视为对象：

- workspace state
- checkpoint
- baseline snapshot
- branch / overlay state

它回答：

- 某个 action 发生时系统处于什么状态
- 某个 artifact 对应哪一轮状态
- rollback 或 replay 的参照点是什么

### 3.5 Artifact

`artifact` 是代码任务里最关键的一类对象之一，例如：

- 文件改动
- patch
- 测试输出
- 报告
- 下载产物
- 生成脚本

artifact 之所以必须单列，是因为很多真实问题最终都不是在事件层，而是在产物层暴露出来。

### 3.6 Actor / Executor

在多工具、多 agent、多 runtime 场景里，仅知道“动作发生了”还不够，通常还需要知道“是谁做的”。

这里的 `actor` 可以是：

- 主 agent
- sub-agent
- tool runtime
- user / human override
- external service

这有助于把错误归因从“发生了问题”推进到“问题更可能由谁引入”。

### 3.7 Decision / Authorization Context

很多高价值 traceability 不只是记录动作本身，还要知道动作为什么会被允许发生。

因此在 agent 环境里，经常还需要显式记录：

- permission decision
- confirmation result
- escalation record
- mode / policy context

没有这层，很多行为虽然被记录了，但难以解释“为什么系统当时认为这一步合理”。

---

## 四、对象之间最关键的关系是什么

### 4.1 Containment：谁属于谁

最基础的一组关系通常是：

- task contains steps
- step contains actions
- task owns workspaces and artifacts

这组关系决定 trace 的层级归属。

### 4.2 Causality：谁导致了谁

比 containment 更关键的是因果关系，例如：

- action produced artifact
- action updated workspace state
- step selected checkpoint
- decision authorized action

traceability 真正的价值，往往建立在这类关系之上。

### 4.3 Versioning / Lineage：谁从谁演化而来

很多对象并不是一次性静态存在，而是会演化：

- artifact derived from baseline
- checkpoint supersedes previous checkpoint
- attempt branches from earlier step
- report corresponds to a specific code state

如果没有 lineage，系统只能看到孤立对象，很难解释演变路径。

### 4.4 Validity / Scope：对象在什么边界内有效

并不是所有对象在任何时刻都同样有效。

例如：

- 某份测试结果只对某个 checkpoint 有效
- 某个 artifact 只属于某次尝试
- 某个授权只在当前模式和当前任务下有效

因此 traceability object model 还需要表达对象的有效范围，而不只是存在本身。

---

## 五、什么不一定需要成为一等对象

### 5.1 展示视图

很多 dashboard、timeline、session transcript 都很有价值，但它们更适合作为派生视图，而不是对象模型本体。

否则系统会把展示结构误当成核心结构。

### 5.2 原始日志行

原始终端输出、调试日志、底层 telemetry 当然值得保留，但它们更适合作为 action 或 artifact 的附属证据，而不是最上层对象。

### 5.3 临时 UI 状态

某些交互式系统中的展开面板、选择态、局部消息缓存，对产品很重要，但未必应该进入环境层 traceability object model。

对象模型需要优先服务归因、恢复与审计，而不是服务所有交互细节。

---

## 六、几种常见对象模型偏向

### 6.1 Event-First Model

核心对象偏向 action / event。

优点：

- 简单
- 易采集
- 时序清晰

局限：

- artifact attribution 往往较弱
- 任务语义和状态绑定不够稳定

### 6.2 Artifact-First Model

核心对象偏向文件、patch、报告、测试结果等 artifact。

优点：

- 更贴近代码任务结果
- 更利于结果解释与审计

局限：

- 动作过程与中间失败路径容易丢细节
- 状态演化捕获要求更高

### 6.3 Task-First Model

核心对象偏向 task / step / attempt。

优点：

- 语义层级清晰
- 适合多步骤与多 agent 场景

局限：

- 底层 action 和 artifact 细节可能被抽象过度

### 6.4 Hybrid Object Model

更现实的方案通常是混合建模：

- task / step 负责语义归属
- action 负责执行痕迹
- workspace state / checkpoint 负责状态锚点
- artifact 负责结果归因
- decision / authorization 负责治理上下文

这类模型更复杂，但通常更能支撑恢复、审计与长期治理。

---

## 七、最重要的结构性矛盾

### 7.1 Minimal Object Set vs Attribution Strength

对象集越小，系统越轻；但对象太少时，归因与恢复能力会迅速退化。

### 7.2 Rich Relationship Graph vs Modeling Complexity

关系越完整，解释能力越强；但建模、采集、存储和维护复杂度也会显著提升。

### 7.3 Stable Core Model vs Task-Specific Extensions

核心对象模型越稳定，系统越可迁移；但不同任务可能需要补充特殊对象，过度通用又会损失表达力。

### 7.4 Auditability vs Noise Explosion

记录越全，审计越强；但如果所有对象都被一等化，trace 系统会迅速膨胀，反而难以理解。

---

## 八、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- 只要有 event log，就不需要对象模型
- artifact 只是附属结果，不需要一等建模
- checkpoint 只属于 recovery，不需要进入 traceability object model
- task / step / attempt 三层一定都必须同时存在
- 统一对象模型一定优于分层对象模型

这些说法都会让 traceability 从结构化归因系统退化回日志堆积，或过早把某一种建模方式写成唯一标准答案。

---

## 九、与其他专题的关系

- 与 `workspace-traceability.md`：前者讨论 traceability 的视角与作用；本文件进一步把这些视角下沉为对象集与关系集。
- 与 `workspace-checkpoint.md`：checkpoint 在这里不只是恢复锚点，也是一类重要 traceability 对象。
- 与 `workspace-structure.md`：workspace 作为 task-specific working context，决定对象归属和状态边界。
- 与 `rollback-recovery-design-paths.md`：恢复路径越依赖 replay、rebuild 与比较，就越需要稳定的对象模型支撑。
- 与 `../conflict.md`：对应 `Workspace Traceability 应以什么为核心组织单位` 这一冲突条目，并为后续收敛最小必要对象集提供主干支撑。
- 与 `../overview.md`：本文件属于 observability / recovery 一侧的进一步下沉专题。

---

## 十、当前最值得继续补证的方向

- 主流 agent 系统是否真的显式建模了 task、step、action、artifact、checkpoint、decision 这些对象，还是主要依赖日志拼装
- object model 中哪些关系最值得优先建模：causality、lineage、validity、containment，还是别的关系
- 多 agent / 多 subtask 场景下 actor、executor 与 task ownership 应如何表达
- object model 应如何同时支持 auditability、recovery、memory writeback 与 strategy revision
- 对代码执行环境之外的 browser / simulated environments，是否需要复用同一核心对象模型，还是分环境扩展
