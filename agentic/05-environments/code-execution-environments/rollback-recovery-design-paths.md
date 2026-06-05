# Rollback and Recovery Design Paths

> 适用范围：Agent 代码执行环境中的 rollback、recovery、replay、snapshot 与重建路径比较
> 阶段状态：研究主干，持续补充框架案例与恢复机制证据
> 使用说明：本文件回答“Agent 系统为什么不能把 rollback / recovery 理解为单一路径、常见恢复机制分别解决什么问题、它们的核心 trade-off 是什么”，不直接预设某一种实现方案为标准答案

## 一、定位

如果说 `workspace-checkpoint.md` 讨论的是 checkpoint 保存什么、如何保存以及与 workspace 的关系，那么本文件进一步讨论的是：**系统在需要回退、恢复、重试或重建任务状态时，究竟可以走哪些设计路径**。

在 Agent 的代码执行环境里，rollback / recovery 不应被狭义理解为“回到上一个版本”或“恢复容器快照”。

它更适合被理解为：**当任务执行失败、中断、污染、分叉或需要人工接管时，系统重新获得可推进状态的机制集合**。

这个定义里最重要的不是“有没有恢复”，而是三个问题：

- 系统要恢复的是文件状态、任务状态、执行上下文，还是它们的组合
- 恢复目标是“退回去”、 “继续做”、 “重新跑” 还是“重新构建”
- 恢复路径如何对齐任务语义、成本边界与审计要求

因此，rollback / recovery 更适合作为 code execution environment 中的结构性专题，而不是附属于某个 snapshot 机制的实现细节。

---

## 二、为什么它必须单列成主题

### 2.1 rollback 与 recovery 不是同一个问题

很多讨论会把两者混在一起，但它们解决的并不是完全相同的问题：

- rollback 更偏把系统退回某个较早、较稳定的状态
- recovery 更偏让任务在失败、中断或污染后重新具备推进能力

一个系统可以有 rollback 但 recovery 很弱，也可以几乎不强调回退，却依赖 replay 或 rebuild 完成恢复。

### 2.2 “恢复成功”不一定意味着“状态完全回到过去”

在长链 Agent 任务中，恢复并不总是要求精确复原每一比特状态。

很多场景真正关心的是：

从当前 OpenHands / `software-agent-sdk` 交叉源码看，这一点还可以再拆得更细：系统可能已经具备很强的 conversation-level recovery——例如恢复 `conversation_id`、`sandbox_id`、`conversation_url`、`session_api_key`，或者让同一 runtime / container / remote session 重新 `resume`——但这仍然不自动等于“底层 workspace 文件状态已经被完整回到过去”或“可脱离原执行实体独立重建”。

- 能否回到一个可继续执行的安全基线
- 能否重新拿到必要 artifact 与上下文
- 能否解释恢复前后发生了什么变化

所以 recovery 的评价标准往往是任务语义上的可继续性，而不只是底层状态的机械复制。

### 2.3 它天然连接 checkpoint、traceability、audit 与 isolation

恢复路径选择会反过来影响：

- checkpoint 需要记录多厚的状态
- traceability 需要保留多强的因果链
- audit 需要看到哪些恢复动作
- sandbox / workspace 生命周期是否要被绑定

如果不把 rollback / recovery 单列出来，很多环境设计问题会一直混写在 checkpoint 或 sandbox 话题里。

---

## 三、常见恢复目标

### 3.1 Return to Known Good State

把 workspace、任务步骤或执行上下文退回到已知稳定点。

典型场景：

- 某轮修改引入明显错误
- 某个子任务污染了共享状态
- 需要人工接管前先恢复安全基线

### 3.2 Resume Interrupted Work

在任务被中断、超时、崩溃或外部依赖失败后，从已有上下文继续推进。

典型场景：

- 长任务执行中断
- 外部服务短暂失败
- worker 重启后需要恢复任务状态

### 3.3 Reconstruct Equivalent State

不要求回到完全相同的物理状态，而是重建出足以继续工作的等价任务状态。

典型场景：

- runtime 或 sandbox 生命周期较短
- 状态跨进程、跨容器或跨机器迁移
- 系统更依赖逻辑状态而非物理快照

### 3.4 Replay or Retry from Structured History

通过事件、步骤、patch、tool calls 或任务轨迹重放执行，重新得到可用状态。

典型场景：

- 需要比较不同恢复尝试
- 希望保留清晰的因果链
- 适合结构化执行循环而非任意副作用环境

---

## 四、常见设计路径

### 4.1 Checkpoint-Based Rollback

以 checkpoint 或 snapshot 为状态锚点，回退到某个既有恢复点。

优点：

- 恢复语义直观
- 适合高确定性回退
- 对人工理解较友好

代价：

- 依赖足够好的 checkpoint 设计
- 状态越厚，成本越高
- 对高频分叉或长任务未必轻量

### 4.2 Transactional Snapshot / Overlay Revert

把一轮执行视为事务性修改，失败时丢弃 overlay、临时层或事务窗口内的副作用。

优点：

- 更适合控制局部副作用
- 对试探性执行较友好
- 有利于在共享基线上做隔离实验

代价：

- 需要稳定定义事务边界
- 跨事务依赖更难处理
- 对外部副作用与非文件状态不一定充分

### 4.3 Event Replay / Step Replay

通过事件流、步骤记录、patch 序列或 task trace 重放到目标状态。

当前 OpenHands 相关核验结果恰好说明了为什么这一路径不能被提前写成既成事实：虽然本地仓库中已经能确认 trajectory 配置项、前端下载 / 拉取入口、event 持久化，以及 SDK / agent-server 的 conversation restore 语义，但仍未在当前已核验范围内定位到一个可直接指认、足以把 event / observation / tool call / artifact / workspace file change 串成完整文件系统重建链的 replay engine。因此，“存在 replay 入口”与“replay 足以重建工作域”之间必须显式留出证据空档。

LangGraph 则提供了另一种对照：它已经能稳定支持 replay、fork、`update_state`、interrupt / resume 和 durable execution，但其 replay / time-travel 语义仍然围绕 graph state checkpoint 与 workflow control flow 展开，而不是默认重建 workspace 文件系统或 execution environment。也就是说，结构化 replay 的存在只能先证明“某层状态可被重放 / 分支 / 继续执行”，不能自动证明“系统所有相关状态都能被重建”。这提醒我们：replay-based recovery 必须明确说明自己重放的是哪一层状态。

优点：

- 更利于解释因果链
- 适合与 traceability、audit 结合
- 可以支持不同恢复分支比较

代价：

- 重放过程本身可能不稳定
- 对外部依赖漂移较敏感
- 对执行步骤结构化程度要求更高

### 4.4 Logical Reconstructability

通过基线版本、patch 集合、artifact 清单、任务状态指针等逻辑元数据重建工作状态。

优点：

- 更适合跨 runtime / 跨 sandbox 迁移
- 更贴近任务语义
- 对物理环境耦合更弱

代价：

- 重建正确性强依赖元数据质量
- 确定性可能弱于物理快照
- 调试失败时定位更复杂

### 4.5 Hybrid Recovery Model

组合使用 snapshot、overlay、replay 与 logical metadata。

典型方式包括：

- 稳定阶段用 checkpoint，局部试探阶段用 overlay revert
- 保留文件 snapshot，同时记录 trace 与 task state 用于解释和重建
- 对高价值节点做 full snapshot，其余节点依赖 replay 或 logical reconstruction

这类设计通常更符合真实系统，但也最容易带来边界膨胀与治理复杂度。

---

## 五、如何比较不同路径

### 5.1 恢复的是哪一层状态

不同路径的第一分界，不是技术名字，而是它们主要恢复什么：

- 文件与目录状态
- 任务步骤与控制流状态
- artifact 与依赖上下文
- 执行环境与 runtime 状态
- 可解释性与审计上下文

如果不先回答这一层，技术路径比较会很容易失焦。

### 5.2 恢复语义是“回到过去”还是“重新变得可用”

有些机制强调精确回退，有些机制强调继续推进。

再进一步，继续推进本身也可能存在不同语义层次：

- 等当前动作自然结束后进入可恢复状态
- 立即中断当前执行，再把会话停在可恢复状态
- 重新连回既有会话或 runtime 身份，但不承诺完整文件系统重建

这也是为什么 `pause`、`interrupt`、`resume` 的差异不能只被看成 API 细节，而应被视为 recovery design path 的组成部分。

前者更偏：

- known-good rollback
- deterministic snapshot restore

后者更偏：

- resumability
- rebuildability
- replay-based recovery

两类目标都合理，但不要混成一个指标体系。

### 5.3 成本不只来自存储

恢复路径的成本通常至少包括：

- 状态记录成本
- 恢复执行成本
- 运行时耦合成本
- 调度与编排成本
- 实现复杂度与维护成本
- 人工理解与排障成本

OpenHands 的 runtime / sandbox / workspace 调度风险说明，“恢复执行成本”本身也应继续拆开：sandbox resume 轮询、远端 runtime API 调用、workspace bridge HTTP 往返、event store 全量读取、setup script 重跑和清理失败处理，都会影响 recovery 的可用性。压缩存储或减少 checkpoint 文件，并不自动降低这些编排链路的尾延迟。

所以“更省空间”不等于“更优恢复设计”。

### 5.4 恢复路径会反过来塑造系统边界

选择某种恢复设计后，系统往往会被反向塑形：

- checkpoint-first 系统更需要稳定的状态锚点
- replay-first 系统更需要结构化任务轨迹
- logical reconstruction 更需要清晰的对象模型与 metadata
- overlay/transaction 设计更需要明确副作用边界

因此 rollback / recovery 不是后加模块，而是会反向影响环境层架构。

---

## 六、最重要的结构性矛盾

### 6.1 Recovery Fidelity vs Operational Cost

恢复越接近精确复原，往往越需要记录更多状态、维持更多约束，操作成本也越高。

### 6.2 Fast Rollback vs Rich Explainability

快速回退机制通常强调效率；而高可解释恢复路径往往需要更多 trace、metadata 与步骤结构。

### 6.3 Runtime Coupling vs Portability

与容器、进程或底层 snapshot 深度绑定的恢复方案通常更直接；但跨环境迁移、跨 worker 重建时可移植性更差。

### 6.4 Localized Undo vs End-to-End Recovery

局部撤销更轻、更快；但很多真实失败跨越文件、artifact、任务步骤与外部调用，单点回退不足以提供完整恢复。

---

## 七、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- rollback 的合理实现只有一种
- 只要有 checkpoint，recovery 就自动成立
- replay 一定比 snapshot 更灵活
- 物理快照一定比逻辑重建更可靠
- 恢复设计只属于底层执行层，不影响任务语义与审计边界

这些说法都会把恢复问题过度简化，掩盖不同任务形态、状态层次与工程成本之间的差异。

---

## 八、与其他专题的关系

- 与 `workspace-checkpoint.md`：前者讨论 checkpoint 记录什么；本文件进一步比较系统实际如何利用这些状态锚点完成 rollback / recovery。
- 与 `workspace-traceability.md`：恢复路径越依赖 replay、rebuild 或错误归因，就越依赖强 traceability 支撑。
- 与 `workspace-structure.md`：workspace 的共享/隔离方式会直接塑造恢复边界与恢复成本。
- 与 `../overview.md`：本文件展开其中 `rollback / recovery` 这一组仍需深化的核心主题。
- 与 `../conflict.md`：对应 `Rollback 的合理路径是否唯一` 这一条目，并为后续比较不同恢复路径的任务适配边界提供主干支撑。
- 与 `../sandboxing-and-safety/sandbox-layers.md`：恢复机制与 `state / recovery layer` 紧密相关，但其实现常与 execution isolation 与 permission control 发生耦合。

---

## 九、当前最值得继续补证的方向

- 哪些 Agent 系统更偏 checkpoint restore，哪些更偏 replay / reconstruction，它们背后的任务假设分别是什么
- full snapshot、overlay revert、event replay、logical reconstruction 在代码任务中的恢复成本与失败模式比较
- recovery 机制应如何覆盖文件状态之外的 artifact、依赖、权限上下文与人工接管痕迹
- 多 agent / 多 subtask 场景下，恢复应优先局部回退、分支重建，还是全局一致性修复
- rollback / recovery 与 auditability、memory writeback、strategy revision 之间的反馈闭环应如何表达
