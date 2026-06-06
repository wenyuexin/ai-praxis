# Agentic AI 基础概念

> 适用范围：Agentic AI 的基础概念空间、系统模型层次与定义边界
> 阶段状态：概念总览，持续收敛
> 使用说明：本文件为 `01-foundations/` 建立整体认知框架，回答“Agentic AI 的基础概念由哪些维度组成、它们之间的层次关系是什么、当前哪些定义已经较稳定、哪些仍然需要持续收敛”，不替代各子目录的专题正文

## 一、定位

`01-foundations/` 是 `agentic/` 知识体系的概念底座。它不直接讨论单智能体的 planning 实现、多智能体的协调拓扑、环境层的沙箱设计或评估方法，而是为这些后续目录提供：

- **概念定义**：workflow、agentic workflow、agent 等术语在当前仓库中的最小工作定义
- **维度关系**：认知架构、执行编排、系统模型之间的层次结构
- **分类依据**：reactive / deliberative、单一 vs 多智能体、环境 vs 运行时的区分原则
- **边界提醒**：哪些概念在当前阶段仍不够稳定，不适合写成定论

因此，本文件的价值不在于给出终极分类法，而在于让 `02-single-agent/`、`03-multi-agent/`、`05-environments/`、`06-frameworks-and-tools/` 等目录在涉及上位概念时，有一个可统一引用的锚点——避免同一概念在各目录中被写出不同口径。

---

## 二、概念空间

Agentic AI 的基础概念至少由以下维度组成。这些维度不是并列的术语列表，而是围绕**“一个系统如何在反馈循环中持续执行任务”**这一核心问题展开的层次结构。

### 2.1 定义与分类（definition-and-taxonomy）

这个维度回答最基础的问题：什么算 agent，什么不算；workflow、agentic workflow、agent、multi-agent 之间的区分线在哪里。

当前较稳定的方向是把“是否具有持续的执行循环”作为 agent 与 workflow / tool calling 之间的首要分界。`02-single-agent/overview.md` 已经明确写出的判断是：agent 的核心特征不是“是否调用了 LLM”，而是系统是否能在环境反馈和中间状态的基础上不断调整后续步骤。这是一个较好的工作定义起点，但它并不等于最终的标准答案——因为产业界对 agent 的定义仍处于漂移中（见 `01-foundations/backlog.md` P1 3.3）。

### 2.2 Reactive / Deliberative 连续谱（behavioral-paradigms）

反应式（reactive）与审慎式（deliberative）代表两种极端设计哲学：

- 反应式系统在收到输入后直接触发动作，不经过显式规划或推理
- 审慎式系统在执行动作前先做显式建模、规划或推理

但在真实系统中，大多数 agent 落在两者之间：局部步骤是反应式的快速响应，全局任务依赖审慎式的步进拆解。因此更稳妥的表述是**连续谱**而非二分法。这个连续谱会直接影响 downstream 对 planning、reflection 与 autonomy structure 的理解——当前 `behavioral-paradigms/` 目录已建立但正文尚未展开，本文件只做站位说明。

### 2.3 认知架构（cognitive-architectures）

认知架构关注模型/认知层面的内部处理与信息流动结构。它不是指工程层面的 module 划分，而是指系统如何用外部设施分担模型不擅长处理的认知负担。

`cognitive-architectures/Externalization_2604.08224.md` 提供了一个较稳定的观察框架：它将 Agent 基础设施的演进统一解释为**外部化**——把模型不擅长解决的认知负担，通过基础设施转化为更简单的形式。其四个技术支柱（memory、skills、protocols、harness）为理解 agent 系统提供了一个超越"模型越大越强"的视角。

外部化框架在当前仓库中有较稳定的概念支撑（`Observed`，基于论文原文），但它能否作为 foundations 层的主要认知架构模型仍有待更多案例核验。

### 2.4 执行编排（orchestration / execution organization）

执行编排关注工程层面的控制流、调度与步骤组织。它不同于认知架构（关注模型如何处理信息），而是关注运行时如何组织节点/步骤的执行顺序、如何管理依赖、如何分流和聚合。

`definition-and-taxonomy/orchestration-paradigms.md` 提供了一个从自主性-协作性二维坐标系出发的四种范式分类：Workflow、Agent、Agentic Workflow、Multi-Agent。这个分类在工程讨论中有较好的实用价值（`Observed`，基于当前框架生态的观察归纳），但它是否适合作为 foundations 层的正式分类依据仍需持续核验——特别是 workflow 与 agentic workflow 的边界在真实工程中经常模糊处理。

### 2.5 系统模型（agent-system-modeling）

系统模型试图把认知架构、执行编排、工具系统、记忆系统、协议、环境、治理等放在同一个上位解释框架中。

当前在这个方向上最重要的信号来自 harness 视角。harness 正在从**工程术语**（"给模型套个壳"）演化为**系统模型概念**：能统一解释 runtime substrate、observability、execution control、memory 协调与权限治理的上位层。这一演进轨迹在多篇论文和工程实践中有独立证据（`candidates.md` 列出了 7 篇 harness 相关论文），但当前仍然处于候选研究对象收集阶段，尚未收敛为 foundations 层正式专题。

---

## 三、几个关键的工作定义

以下定义是当前仓库中**可临时使用的口径**，不宣称已经得到最终标准答案。这些定义会随着 01-foundations 继续建设而收敛。

### 3.1 Workflow

Workflow 是一组**预先定义或半定义**的执行步骤，步骤之间的依赖关系和分支条件相对确定。workflow 的核心特征是**结构先于执行**：可执行性在运行前即可判断。

### 3.2 Agentic Workflow

Agentic Workflow 是在一定程度上保留 workflow 的结构约束，同时允许 LLM 在结构内的关键节点（分支条件、质量评估、路由决策等）做自主判断。它不是纯 workflow，也不是纯 agent，而是**有约束的自主**。

### 3.3 Agent

Agent 的核心特征是**持续的执行循环**：接收目标或上下文 → 推理/规划 → 执行动作（工具调用、代码执行等）→ 接收反馈/观察 → 更新内部状态 → 进入下一轮。它不是"一次 LLM 调用"，也不是"一个 tool calling workflow"。

当前较稳妥的最小工作界定是：**系统是否具备持续的执行循环，能否在环境反馈和中间状态的基础上不断调整后续步骤**（`02-single-agent/overview.md`）。

### 3.4 三者的关系

- workflow 保证可预测的执行路径
- agent 保证在不确定环境中的持续适应
- agentic workflow 是两者的交集：保留结构约束，释放局部自主

> 注意：这三者的边界在产业界仍处于漂移中（`backlog.md` P1 3.3）。本文件给出的定义适合当前 `agentic/` 目录内部使用，但不适合作为通用标准推广。

---

## 四、认知架构、执行编排与系统模型

这三层关系是 foundations 层面最容易混淆的核心结构。当前先把它们的区分写清楚：

| 层次 | 关注问题 | 典型对象 | 归属目录 |
|------|---------|----------|----------|
| 认知架构 | 模型如何处理信息、承担什么认知负担 | memory、skills、protocols 的外部化模型 | `cognitive-architectures/` |
| 执行编排 | 工程运行时如何组织步骤、管理依赖 | StateGraph、superstep、orchestrator | 分散在 `02/03/05/06` |
| 系统模型 | 能把上述两者放进同一框架的上位视角 | harness、runtime substrate、4-pillar 模型 | `agent-system-modeling/` |

**认知架构**回答的是**模型和外部设施如何分担认知任务**。Externalization 论文的四个支柱（memory 外化时间状态、skills 外化程序性知识、protocols 外化交互结构、harness 作为统一协调层）是当前该层最完整的观察框架。但它是一个**论文视角**（`Observed`），不是已验证的行业共识。

**执行编排**回答的是**运行时如何组织步骤的执行**。LangGraph 的 Pregel/superstep 模型（`06-frameworks-and-tools/01-frameworks/langgraph/runtime-and-execution.md`）、OpenHands 的 sandbox service / app server 分层（`06-frameworks-and-tools/03-project-studies/openhands/architecture.md`）都属于这一层。

**系统模型**回答的是**能否用一个统一概念框架解释 agent 系统中认知、工具、记忆、协议、环境、治理之间的相互作用**。harness 作为该层候选视角的价值在于：它不是简单的编排器，而是认知架构与执行编排的共同 carriers——既管理 memory/skills/protocols/policies 的生命周期，也提供 permission、observability、context budget 等横切机制。但也正因为如此，它容易变成"模型之外的一切"，需要持续警惕边界膨胀（见 `candidates.md` 和 `backlog.md` P0 2.2）。

---

## 五、Reactive / Deliberative / Hybrid 连续谱

纯反应式系统与纯审慎式系统在实际工程中两端都很少见。大多数 agent 系统落在一个混合连续谱上：

- **纯反应式**：输入 → 直接映射到动作。适合可预测、低风险、快速响应的子任务
- **纯审慎式**：输入 → 显式建模/规划 → 执行。适合高复杂度、高风险、多步骤任务
- **混合连续谱**：大多数系统在局部步骤做反应式处理，在全局任务上保留审慎式的规划或反思

这个连续谱的重要性在于：

- 影响 planning 的设计：反应式系统不需要显式 planner；审慎式系统需要；混合系统需要一个 "when to plan" 的自适应决策
- 影响 reflection 的价值：在审慎式端，reflection 往往更有效；在反应式端，reflection 的成本可能覆盖收益
- 影响 autonomy 的定义：一个 "偏反应式" 的系统和 "偏审慎式" 的系统即使在同一个任务上做相同的事，其自主性的来源也不同

当前 `behavioral-paradigms/` 目录已建立但正文未展开。本文件只在概念层面站位。

---

## 六、当前最容易混淆的概念边界

以下边界在 `02-single-agent/`、`03-multi-agent/`、`05-environments/`、`06-frameworks-and-tools/` 中反复出现，但在 foundations 层还没有形成统一锚点。

### 6.1 Workflow vs Agent vs Tool Calling

| 概念 | 判断标准 |
|------|----------|
| 单纯 tool calling | 无持续执行循环，无状态保持 |
| workflow | 有预定义步骤结构，执行不依赖 LLM 自主判断 |
| agent | 有持续执行循环，在环境反馈中调整行为 |
| agentic workflow | 有 workflow 框架 + 内部节点允许 LLM 自主判断 |

当前 `02-single-agent/overview.md` 已给出很好的边界说明。主要风险在于把 "tool calling workflow" 直接写成 agent。

### 6.2 Cognitive Architecture vs Orchestration Architecture

| 层面 | 关注 |
|------|------|
| 认知架构 | 模型与外部设施如何分配认知负担（memory、skills、protocols） |
| 编排架构 | 工程运行时如何组织执行步骤（graph、superstep、sandbox lifecycle） |

这两者常被混写，因为同一个系统（如 LangGraph）同时提供了 graph runtime（编排层）和 checkpoint/store（可以被视为认知层的基础设施）。更稳妥的写法是交叉引用但不混淆。

### 6.3 Environment vs Execution Container

已在 `05-environments/overview.md` 中充分讨论。核心区别在于：

- environment 至少包括 permission layer、execution sandbox、observability/recovery、governance
- execution container 只覆盖 execution isolation

### 6.4 Orchestration vs Coordination vs Collaboration

| 概念 | 关注 |
|------|------|
| orchestration | 集中式步骤编排与调度 |
| coordination | 多参与者之间的依赖管理、冲突消解、全局一致性 |
| collaboration | 多 agent 共同分工完成一个任务 |

当前 `03-multi-agent/coordination/README.md` 已有较清晰的边界说明。主要风险是把 coordination 与 collaboration 混写——前者更偏调度与冲突消解，后者更偏任务分工与角色耦合。

---

## 七、与其他目录的关系

- **`02-single-agent/`**：本文件的概念定义直接服务于单智能体的执行循环描述；single-agent 的 planning / tool-use / reflection 等能力都是在执行循环中发挥作用的。
- **`03-multi-agent/`**：multi-agent 系统的 topology、failure modes、何时引入的判断，需要先有 "什么是 agent" 的稳定定义。
- **`04-human-agent-interaction/`**：委托粒度、控制面设计、trust 与 alignment 的讨论建立在 "agent 能做什么、不能做什么" 的判断之上。
- **`05-environments/`**：environment 的定义（不等于 execution container）需要 foundations 层的支持；permission/execution/observability 三层框架可以在概念层面找到更上位的对应。
- **`06-frameworks-and-tools/`**：框架和工具是这些概念的具体工程实现。foundations 层的定义帮助判断一个框架是否真的算 "agent framework"。
- **`07-evaluation/`**：评估方法依赖 "什么算一个成功的 agent" 的判定标准。

---

## 八、当前最值得继续补证的方向

以下方向从 overview 视角看**还不适合直接写成定论**，但值得在 `backlog.md` 或专题文件继续跟踪：

1. **Agent 工作定义的稳定性**：随着产业界对 agent 定义的漂移，"持续执行循环" 这一工作定义是否足够区分 agent 与 agentic workflow，需要持续检验。更适合进入 `definition-and-taxonomy/` 专题或 `backlog.md`。

2. **Harness 作为系统模型的收敛**：当前 harness 从工程术语走向系统模型概念的信号已足够明确，但仍主要在 candidate 阶段。如果后续有更多案例支持或正式专题产出，可以考虑升级为 foundations 层正式专题。

3. **认知架构与编排架构的统一表述**：Externalization 的 4-pillar 框架、orchestration-paradigms 的 4-paradigm 框架、topology 的 4-type 框架之间是否存在更上位的统一结构，当前还不清楚。更适合在 `backlog.md` 中持续跟踪。

4. **Reactive / Deliberative 连续谱的工程映射**：连续谱的概念框架已有，但如何映射到具体的 planning/reflection/autonomy 设计决策，仍然缺乏系统化整理。适合放入 `behavioral-paradigms/` 专题。

5. **Terminology 漂移的记录机制**：当前仓库对 workflow / agent / agentic workflow 的定义是基于当前状态的便利工作定义。如果产业界在这些术语上的口径持续变化，需要有一个轻量机制来记录不同来源的口径差异。适合进入 `definition-and-taxonomy/`。

---

## 九、用途边界

- 本文件是 `01-foundations/` 的第一份 overview，不是 foundations 层理论体系的最终答案。
- 本文件中的工作定义适合当前 `agentic/` 目录内部统一引用，但不应被写为外部公认标准。
- 如果后续 subdirectories 的专题正文导致某条定义需要更新，优先更新本文件，再同步其影响的下游 overview 引用。
- 本文件不替代 `backlog.md` 中的问题缺口职责；后者继续记录 "尚待专题化" 的问题。

---

## Evidence

- Status: `Observed / Inferred`
- Sources: `02-single-agent/overview.md`（agent 最小工作定义、执行循环描述）、`03-multi-agent/coordination/topology.md`（topology 主分类轴论证结构）、`05-environments/overview.md`（environment ≠ execution container 边界）、`cognitive-architectures/Externalization_2604.08224.md`（外部化 4-pillar 框架）、`definition-and-taxonomy/orchestration-paradigms.md`（4 范式分类）、`agent-system-modeling/candidates.md`（harness 候选对象集）、`01-foundations/backlog.md`（P0 缺口定义）
- Trace: 从 `round5-feedback.md` 的 "首选方向 = 01-foundations 概念底座补齐" 出发，基于当前已有积累完成 foundations 层第一份全貌文件
- Needs: 随着 02–07 继续建设，本文件的定义可能需要同步更新；特别是 agent 工作定义和 harness 概念的系统化程度

---

*最后更新: 2026-06-06*