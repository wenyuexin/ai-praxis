# Agentic AI 基础概念

> 适用范围：Agentic AI 的基础概念空间、系统模型层次与定义边界
> 阶段状态：概念总览，持续收敛
> 使用说明：本文件为 `01-foundations/` 建立整体认知框架，回答“Agentic AI 的基础概念由哪些维度组成、它们之间的层次关系是什么、当前哪些定义已经较稳定、哪些仍然需要持续收敛”，不替代各子目录的专题正文

## 一、定位

`01-foundations/` 是 `agentic/` 知识体系的概念底座。它不直接讨论单智能体的 planning 实现、多智能体的协调拓扑、环境层的沙箱设计或评估方法，而是为这些后续目录提供：

- **概念定义**：workflow、agentic workflow、agent 等术语在当前仓库中的最小工作定义
- **维度关系**：认知架构、执行编排、系统模型之间的层次结构
- **分类依据**：reactive / deliberative 连续谱、单一 vs 多智能体、能力层 vs 对象层的区分原则
- **边界提醒**：哪些概念在当前阶段仍不够稳定，不适合写成定论

因此，本文件的价值不在于给出终极分类法，而在于让 `02-single-agent/`、`03-multi-agent/`、`05-environments/`、`06-frameworks-and-tools/` 等目录在涉及上位概念时，有一个可统一引用的锚点——避免同一概念在各目录中被写出不同口径。

---

## 二、概念空间与工作定义

Agentic AI 的基础概念不是并列术语列表，而是围绕**“一个系统如何在反馈循环中持续执行任务”**这一核心问题展开的结构化概念空间。

### 2.1 定义与分类（definition-and-taxonomy）

这个维度回答最基础的问题：什么算 agent，什么不算；workflow、agentic workflow、agent、multi-agent 之间的区分线在哪里。

当前可临时使用的工作定义如下：

- **Workflow**：一组**预先定义或半定义**的执行步骤，步骤之间的依赖关系和分支条件相对确定。它的核心特征是**结构先于执行**。
- **Agentic Workflow**：在保留 workflow 结构约束的同时，允许 LLM 在关键节点（如路由、评估、分支）做自主判断。它是**有约束的自主**。
- **Agent**：具备**持续的执行循环**：接收目标或上下文 → 推理/规划 → 执行动作 → 接收反馈/观察 → 更新内部状态 → 进入下一轮。它不是“一次 LLM 调用”，也不是“一个 tool calling workflow”。

当前较稳妥的最小工作界定是：**系统是否具备持续的执行循环，能否在环境反馈和中间状态的基础上不断调整后续步骤**。这已经是当前 `agentic/` 目录内部比较稳定的使用口径，但仍不应当作产业界最终标准，因为外部对 agent 的定义仍在漂移（见 `01-foundations/backlog.md` P1 3.3）。

三者之间的关系可以暂时理解为：

- workflow 保证可预测的执行路径
- agent 保证在不确定环境中的持续适应
- agentic workflow 是两者的交集：保留结构约束，释放局部自主

### 2.2 Behavioral Paradigms（behavioral-paradigms）

Reactive 与 deliberative 代表两种极端设计哲学：

- **反应式（reactive）**：收到输入后直接触发动作，不经过显式规划或推理
- **审慎式（deliberative）**：执行动作前先做显式建模、规划或推理

但真实系统很少落在纯粹两端，更稳妥的表述是**连续谱**：局部步骤可能是反应式的，全局任务则依赖审慎式的规划或反思。因此 foundations 层更关心的不是“二分法”，而是这个连续谱如何影响对 planning、reflection 与 autonomy structure 的理解。当前 `behavioral-paradigms/` 目录已建立，但正文尚未展开，本文件只建立概念锚点。

### 2.3 认知架构（cognitive-architectures）

认知架构关注模型/认知层面的内部处理与信息流动结构。它不是工程层面的 module 划分，而是系统如何用外部设施分担模型不擅长处理的认知负担。

`cognitive-architectures/Externalization_2604.08224.md` 提供了一个较稳定的观察框架：Agent 基础设施的演进可以理解为**外部化**——把模型不擅长解决的认知负担，通过基础设施转化为更简单的形式。其四个技术支柱（memory、skills、protocols、harness）为理解 agent 系统提供了一个超越“模型越大越强”的视角。

这个框架在当前仓库中有较稳定的概念支撑（`Observed`，基于论文原文），但它能否成为 foundations 层的主要认知架构模型，仍有待更多案例核验。

### 2.4 执行编排（orchestration / execution organization）

执行编排关注工程层面的控制流、调度与步骤组织。它不同于认知架构（关注模型如何处理信息），而是关注运行时如何组织节点/步骤的执行顺序、如何管理依赖、如何分流和聚合。

`definition-and-taxonomy/orchestration-paradigms.md` 提供了一个从自主性—协作性二维坐标出发的四种范式分类：Workflow、Agent、Agentic Workflow、Multi-Agent。这个分类在工程讨论中有较好的实用价值（`Observed`，基于当前框架生态的观察归纳），但它是否适合作为 foundations 层的正式分类依据仍需持续核验，特别是 workflow 与 agentic workflow 的边界在真实工程中经常模糊处理。

### 2.5 系统模型（agent-system-modeling）

系统模型试图把认知架构、执行编排、工具系统、记忆系统、协议、环境、治理等放在同一个上位解释框架中。

当前在这个方向上最重要的信号来自 harness 视角。harness 正在从**工程术语**演化为**系统模型概念**：它尝试统一解释 runtime substrate、observability、execution control、memory 协调与权限治理的上位层。这一演进轨迹在多篇论文和工程实践中有独立证据；当前仓库已不只是候选对象排队阶段，而是进入了**候选对象跟踪 + 多篇论文笔记成形 + 继续复核剩余证据边界**的状态（见 `agent-system-modeling/candidates.md`、`agent-system-modeling/2605.13357_AI_Harness_Engineering.md` 与 `agent-system-modeling/ETCLOVG_Harness_Survey.md`），但仍未收敛为 foundations 层正式专题。

---

## 三、关键结构关系

### 3.1 认知架构、执行编排与系统模型

这三层关系是 foundations 层最容易混淆的核心结构：

| 层次 | 关注问题 | 典型对象 | 当前归属 |
|------|---------|----------|----------|
| 认知架构 | 模型如何处理信息、承担什么认知负担 | memory、skills、protocols 的外部化模型 | `cognitive-architectures/` |
| 执行编排 | 工程运行时如何组织步骤、管理依赖 | StateGraph、superstep、orchestrator | 分散在 `02/03/05/06` |
| 系统模型 | 能把上述两者放进同一框架的上位视角 | harness、runtime substrate、4-pillar 模型 | `agent-system-modeling/` |

- **认知架构**回答的是：模型和外部设施如何分担认知任务。
- **执行编排**回答的是：运行时如何组织步骤的执行。
- **系统模型**回答的是：能否用一个统一概念框架解释 agent 系统中认知、工具、记忆、协议、环境、治理之间的相互作用。

更直白地说：同一个系统可以同时拥有认知层解释、执行层组织和系统层建模，但三者不应被写成同一件事。LangGraph、OpenHands 等系统之所以容易引发混写，正是因为它们在同一工程对象中同时承载了多个层次。

### 3.2 Reactive / Deliberative / Hybrid 连续谱

纯反应式系统与纯审慎式系统在实际工程中两端都很少见。大多数 agent 系统落在一个混合连续谱上：

- **纯反应式**：输入 → 直接映射到动作，适合可预测、低风险、快速响应的子任务
- **纯审慎式**：输入 → 显式建模/规划 → 执行，适合高复杂度、高风险、多步骤任务
- **混合连续谱**：局部步骤偏反应式，全局任务保留审慎式规划或反思

这个连续谱的重要性在于：

- 它影响 planning 的设计：不是所有系统都需要显式 planner
- 它影响 reflection 的价值：反思收益和执行成本并不均衡
- 它影响 autonomy 的定义：系统的自主性既来自“能不能做”，也来自“如何做”

因此，behavioral paradigms 在 foundations 层不是“一个附属标签”，而是理解下游 planning / reflection / autonomy 设计分歧的上位轴。

---

## 四、当前最容易混淆的核心边界

### 4.1 Workflow vs Agent vs Tool Calling

| 概念 | 判断标准 |
|------|----------|
| 单纯 tool calling | 无持续执行循环，无状态保持 |
| workflow | 有预定义步骤结构，执行不依赖 LLM 自主判断 |
| agent | 有持续执行循环，在环境反馈中调整行为 |
| agentic workflow | 有 workflow 框架 + 内部节点允许 LLM 自主判断 |

当前最主要的风险不是概念本身不存在，而是把 “tool calling workflow” 直接写成 agent，或者把任何带 LLM 判断的流程都笼统写成 agentic。foundation 层在这里的职责是提供最小可用区分线，而不是给出永恒标准答案。

### 4.2 Cognitive Architecture vs Orchestration

| 层面 | 关注 |
|------|------|
| 认知架构 | 模型与外部设施如何分配认知负担（memory、skills、protocols） |
| 编排架构 | 工程运行时如何组织执行步骤（graph、superstep、sandbox lifecycle） |

这两者常被混写，因为同一个系统可能同时提供 graph runtime、checkpoint、store、tool execution、policy control 等能力。更稳妥的写法是：承认它们在真实系统中会耦合，但在概念层面仍保持区分。

---

## 五、当前最值得继续补证的方向

以下方向从 overview 视角看**还不适合直接写成定论**，但值得在 `backlog.md` 或专题文件继续跟踪：

1. **Agent 工作定义的稳定性**：随着产业界对 agent 定义的漂移，“持续执行循环”这一工作定义是否足够区分 agent 与 agentic workflow，需要持续检验。更适合进入 `definition-and-taxonomy/` 专题或 `backlog.md`。
2. **Harness 作为系统模型的收敛**：当前 harness 从工程术语走向系统模型概念的信号已足够明确，且已进入多篇论文笔记成形与剩余证据边界继续复核阶段；但它仍未收敛为 foundations 层正式专题。如果后续有更多案例支持，或 `agent-system-modeling/` 下形成更稳定的总论，再考虑升级为正式专题。
3. **认知架构与编排架构的统一表述**：Externalization 的 4-pillar 框架、orchestration-paradigms 的 4-paradigm 框架、topology 的 4-type 框架之间是否存在更上位的统一结构，当前还不清楚。更适合在 `backlog.md` 中持续跟踪。
4. **Reactive / Deliberative 连续谱的工程映射**：连续谱的概念框架已有，但如何映射到具体的 planning / reflection / autonomy 设计决策，仍然缺乏系统化整理。适合放入 `behavioral-paradigms/` 专题。
5. **Terminology 漂移的记录机制**：当前仓库对 workflow / agent / agentic workflow 的定义是基于当前状态的便利工作定义。如果产业界在这些术语上的口径持续变化，需要有一个轻量机制来记录不同来源的口径差异。适合进入 `definition-and-taxonomy/`。

---

## 六、用途边界

- 本文件是 `01-foundations/` 的概念总览，不是 foundations 层理论体系的最终答案。
- 本文件中的工作定义适合当前 `agentic/` 目录内部统一引用，但不应被写为外部公认标准。
- 本文件只保留概念层面的上游关系；目录分工与导航入口由 `agentic/README.md` 和 `01-foundations/README.md` 承担。
- 如果后续专题正文导致某条定义需要更新，优先更新本文件，再同步其影响的下游 overview 引用。
- 本文件不替代 `backlog.md` 中的问题缺口职责；后者继续记录“尚待专题化”的问题。

---

## Evidence

- Status: `Observed / Inferred`
- Sources: `02-single-agent/overview.md`（agent 最小工作定义、执行循环描述）、`cognitive-architectures/Externalization_2604.08224.md`（外部化 4-pillar 框架）、`definition-and-taxonomy/orchestration-paradigms.md`（4 范式分类）、`agent-system-modeling/candidates.md`（harness 候选对象集与阅读顺序）、`agent-system-modeling/2605.13357_AI_Harness_Engineering.md`（runtime substrate 方向论文笔记）、`agent-system-modeling/ETCLOVG_Harness_Survey.md`（生态分类方向论文笔记）、`01-foundations/backlog.md`（P0 缺口定义）
- Trace: 从 `round5-feedback.md` 的“首选方向 = 01-foundations 概念底座补齐”出发，基于当前已有积累完成 foundations 层概念总览，并在后续重构中收缩重复与越界内容
- Needs: 随着 `02`–`07` 继续建设，本文件的定义可能需要同步更新；特别是 agent 工作定义和 harness 概念的系统化程度

---

*最后更新: 2026-06-08*