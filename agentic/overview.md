# Agentic AI Overview

> 适用范围：`agentic/` 顶层领域的整体认知框架
> 阶段状态：早期综述，基于现有文档整理，主题仍不完整
> 使用说明：本文件回答“Agentic AI 当前可以怎样从概念、能力、协作、环境、工具和评估层整体理解”，不替代各子目录专题，也不把尚未覆盖的主题写成已稳定结论

## 一、定位

Agentic AI 研究的是：当 LLM 不再只是生成一次回答，而是被放入一个可观察、可行动、可反馈的任务环境中时，系统如何获得持续行动能力，以及这种能力会带来哪些新的边界问题。

它的核心不只是“让模型会调用工具”，而是把开放目标转化为一个动态执行过程：系统需要理解目标、选择行动、调用外部能力、读取环境反馈、维护中间状态，并在失败、风险或不确定性出现时调整后续路径。由此，Agentic AI 的研究对象从单次推理扩展为一个 **model + state + tools + environment + control + evaluation** 的复合系统。

从领域级别看，当前可以先把 Agentic AI 拆成七个互相耦合的问题层：

- **概念层**：什么算 agent，什么只是 workflow、tool calling 或 assistant；持续执行循环、自主性和反馈适应是最小边界问题。
- **能力层**：单个 Agent 如何规划、行动、使用工具、维护记忆并进行反思；这里关注的是能力闭环，而不是工具清单。
- **协作层**：多个 Agent 是否、何时、如何带来净收益；协调拓扑、通信成本、共享状态和失败传播是核心变量。
- **人机关系层**：人类如何委托、约束、观察、批准、接管和校准信任；交互表面背后是控制关系。
- **环境层**：Agent 能看到什么、能做什么、在哪里执行、如何被隔离、追踪和恢复；环境不等于容器。
- **工程对象层**：框架、工具、产品和项目案例如何把上述能力组合成真实系统；对象研究提供证据，但不自动产生通用定论。
- **评估层**：如何判断系统是否真的有效、可靠、可复现、可调试且适合真实使用；benchmark 只是其中一类信号。

因此，本领域的顶层问题不是“有哪些 Agent 工具”或“哪种框架最好”，而是：**一个 LLM 驱动系统在不确定任务中如何安全、可控、可恢复、可评估地持续行动**。

---

## 二、当前阶段说明

本文件是早期版本，主要基于当前已有 `README.md`、各子目录 `overview.md`、`backlog.md` 和已形成的部分项目 / 框架研究归纳而来。

需要特别注意：

- 当前 `agentic/` 仍存在明显主题缺失，尤其是人机交互、评估、部分基础行为范式、浏览器环境、仿真环境和跨层对比专题仍不充分。
- 本文件只整理已经在主干中有一定支撑的结构，不用“当前缺少内容”反向证明目录结构已经合理。
- 对安全、恢复、性能、评估和多智能体收益等问题，只保留保守表述；未验证材料不写成主线定论。
- 具体框架和项目案例只作为观察样本，不外推为 Agentic AI 的通用实现路线。

---

## 三、Agentic AI 的核心系统图景

从当前文档体系看，Agentic AI 可以被理解为一个围绕持续执行循环展开的系统：

```text
目标 / 任务
  → 概念与系统模型
  → 单智能体能力闭环
  → 多智能体协作结构（可选）
  → 人类委托、控制与接管
  → 环境、权限、执行与恢复边界
  → 框架、工具与项目实现
  → 评估、调试与可靠性反馈
  → 回到系统修订
```

这个循环强调三点：

1. **Agentic 不等于一次 LLM 调用**：更关键的是持续执行循环，以及能否在反馈和中间状态基础上调整后续步骤。
2. **能力不等于可靠系统**：planning、memory、tool use、multi-agent、sandbox、UI、benchmark 都只是局部能力；它们需要被组织成可控、可恢复、可评估的系统。
3. **真实系统总是跨层耦合**：工具调用会受环境约束影响，多智能体收益会受评估基线影响，人机交互会改变 autonomy 和 permission 的边界。

---

## 四、七个层级的当前理解

### 4.1 Foundations：先定义边界，再讨论能力

基础层的核心作用是防止后续目录在概念上各说各话。

当前较稳定的工作口径是：

- workflow 更强调预定义或半定义的步骤结构。
- agent 更强调持续执行循环和基于反馈的调整能力。
- agentic workflow 位于两者交界处：保留流程约束，同时在关键节点释放局部自主判断。

这一口径适合当前仓库内部使用，但不应写成外部公认标准。外部关于 agent、workflow、harness、runtime substrate、orchestrator 等术语仍在漂移。

### 4.2 Single-Agent：能力闭环，而不是工具清单

单智能体层关注一个 Agent 如何围绕目标形成闭环：规划、行动、调用工具、接收反馈、更新记忆、必要时反思和修正。

当前最重要的结构性矛盾包括：

- context completeness vs retrieval noise
- planning-execution 耦合 vs 解耦
- tool-centric design vs monolithic agent
- reflection 收益 vs token / latency / 执行成本

因此，单智能体不是“能 tool call 就算 Agent”，而是一个围绕任务持续推进的能力组合。

### 4.3 Multi-Agent：收益来自结构，成本也来自结构

多智能体层关注多个决策单元共同工作时出现的新问题。它的收益通常来自并行化、专业化、视角互补和局部错误隔离；代价则来自协调开销、状态同步、角色漂移、重复劳动和隐性 token / latency 成本。

当前最需要保守表达的结论是：multi-agent 并不天然优于 single-agent。是否有收益取决于任务类型、基线能力、协调拓扑、共享状态方式和评估设定。

### 4.4 Human-Agent Interaction：交互表面背后是控制关系

人机交互层不只是 UI 或聊天界面，而是人类如何委托、约束、观察、接管和信任 Agent 系统。

当前可用的四个视角是：

- delegation and control：人类把什么权力交给 Agent。
- human-in-the-loop：人在执行闭环中何时介入、以什么角色介入。
- interaction surfaces：系统如何呈现计划、状态、动作、日志和 artifact。
- trust and alignment：系统如何表达能力边界、不确定性和风险。

这一层与环境层和评估层强耦合：权限确认、可见性、接管机制和可信评估往往共同决定 Agent 是否能被安全使用。

### 4.5 Environments：环境不等于容器

环境层定义 Agent 能看到什么、能做什么、实际在哪里执行、失败后如何被追踪和恢复。

当前较稳定的三层理解是：

- permission layer：限制 Agent 被允许做什么。
- execution sandbox：限制动作实际在哪里发生、副作用能扩散到哪里。
- observability and recovery：记录 Agent 做了什么，以及失败后如何解释和恢复。

还需要加一条重要 stop-line：connection recovery 不等于 filesystem reconstruction。conversation restore、runtime resume、workflow checkpoint、trajectory replay 或 sandbox identity 恢复，都不能自动外推为 workspace 文件系统可独立重建。

### 4.6 Frameworks and Tools：对象研究不是能力 taxonomy

框架与工具层采用对象视角，研究具体框架、编码工具、项目案例、Skill / Tool 系统和横向比较。

它不负责重新讲 planning、memory、tool-use、multi-agent、environment 或 evaluation 的基础理论，而是观察这些能力如何在真实系统中被实现、组合和产品化。

当前需要守住的边界是：对象研究应保持内聚，跨主题目录只回填稳定且必要的证据；不要把单个框架或项目的实现方式写成 Agentic 通用结构。

当前还需要额外区分工具协议层与编排运行时层：tool schema、tool discovery、tool execution governance、workflow state、result normalization 是不同层的问题。一个对象暴露了工具调用能力，只能说明它提供了某种可调用接口，不能自动推出它已经解决权限、超时、取消、审计、状态恢复和结果归一化等完整编排契约问题。

### 4.7 Evaluation：评估不是排行榜

评估层回答的是 Agent 是否有效、可靠、可复现、可解释、可调试，而不是只看 benchmark 分数。

当前五个核心视角是：

- task completion and outcome quality
- benchmark design and representativeness
- observability and debugging
- human evaluation and trustworthy use
- safety, robustness and failure boundary

最重要的矛盾包括 benchmark score vs real-world utility、outcome metrics vs process understanding、automatic evaluation vs human judgment、reproducibility vs ecological validity。

---

## 五、当前最重要的跨层矛盾

### 5.1 Autonomy vs Control

Agentic 系统越自主，越能减少人工操作负担；但自主性越高，权限、可见性、接管和责任边界也越重要。

这条矛盾同时出现在：

- 单智能体的 planning / reflection 设计
- 人机交互中的 delegation / takeover
- 环境层的 permission / governance
- 评估层的 safety / robustness

### 5.2 Capability vs Reliability

更强的模型、更长上下文、更多工具或更多 Agent，并不自动带来更可靠的系统。

可靠性往往来自：

- 工具调用前后的约束和校验
- 记忆写回与污染控制
- 环境权限和副作用隔离
- 轨迹、日志、artifact 与 checkpoint 的可追溯性
- 失败归因和可恢复路径

### 5.3 Coordination Gain vs Coordination Cost

多智能体和复杂工作流都可能带来任务拆分与并行收益，但它们也会放大协调、状态同步、消息冗余和执行延迟。

因此，Agentic 系统设计不能只问“能否拆分”，还要问“拆分后是否仍有净收益”。

### 5.4 Recovery Semantics vs Recovery Claims

恢复能力必须拆开讨论：

- connection / conversation recovery
- workflow state recovery
- execution environment recovery
- trajectory replay
- workspace filesystem recovery
- external side-effect recovery

当前已有材料支持把这些语义区分开，但不支持把其中一种恢复能力外推为全部恢复能力。尤其要避免把 conversation state、workflow / graph state 和 workspace artifact 混写成同一种 state，也不能把 trace / observability 直接写成 checkpoint / replay / state persistence。

### 5.5 Benchmark Score vs System Utility

Agent benchmark 可以提供重要信号，但真实可用性还取决于环境、权限、工具、交互、成本、失败恢复和任务分布。

因此，评估结果需要能回流到系统层：失败到底来自模型推理、工具调用、记忆、环境、协调、人类交互，还是 benchmark 设定本身。

---

## 六、容易误写成定论的问题

当前阶段尤其要避免以下误写：

- tool use 等于 agent。
- tool schema / tool discovery 等于完整编排契约。
- multi-agent 总比 single-agent 强。
- Docker sandbox 足以定义 Agent 安全。
- 更长上下文可以替代外部记忆和状态管理。
- reflection 一定带来净收益。
- conversation / workflow / runtime 恢复等于 workspace 文件系统恢复。
- benchmark 排名足以代表真实可用性。
- 某个框架或项目的实现路径代表 Agentic AI 的通用路线。

这些问题应优先进入 `backlog.md`、`conflict.md` 或对应专题继续补证，而不是在顶层综述中写成单向结论。

---

## 七、与主干文档的关系

- `README.md` 负责目录定向、边界说明和导航。
- 本文件负责顶层领域综述和跨层问题地图。
- `backlog.md` 记录跨目录主题缺口，不追踪任务进度。
- 子目录 `overview.md` 负责各层自身的整体认知。
- 子目录 `backlog.md`、`candidates.md`、`conflict.md` 分别承接问题缺口、候选对象和冲突口径。
- 具体论文、框架、工具和项目案例应优先保持对象内聚，不在顶层重复展开。

---

## 八、当前仍明显缺失的主题

本文件只是早期版本，后续至少还需要继续补齐：

- 顶层 Agentic AI 与传统 automation / workflow / assistant 的边界专题。
- `04-human-agent-interaction/` 中 human-in-the-loop、delegation granularity、trust calibration 的正文专题。
- `07-evaluation/` 中 failure attribution、benchmark transferability、partial completion 与 human evaluation rubrics。
- `05-environments/` 中 browser environment、simulated environment 与 evaluation environment 的更多主线材料。
- `02-single-agent/` 中 memory writeback、tool reliability failure taxonomy、reflection trigger 的更多证据。
- `03-multi-agent/` 中 hidden cost、specialization granularity、structured protocol vs emergent coordination 的更多对照。
- `06-frameworks-and-tools/` 中更多对象研究如何稳妥回填到 `02/03/05/07` 的案例边界。

这些缺口应继续由 `agentic/backlog.md` 和各子目录 backlog 承接；在证据足够前，不应把它们提前写成顶层定论。

---

## Evidence

- Status: `Observed / Inferred`
- Sources: `agentic/README.md`、`01-foundations/overview.md`、`02-single-agent/overview.md`、`03-multi-agent/overview.md`、`04-human-agent-interaction/overview.md`、`05-environments/overview.md`、`06-frameworks-and-tools/overview.md`、`07-evaluation/overview.md`、`agentic/backlog.md`、`agentic/temp/chat/feedback.md`
- Trace: 从顶层 backlog 中“Agentic 顶层 overview 缺口”出发，先基于已有主干文档形成早期综述；本次仅从 `agentic/temp/chat/feedback.md` 回填保守问题意识，不回填具体对象判断、契约草案或未经核验的横向结论；主题缺失和待补证方向继续保留在 `agentic/backlog.md` 与各子目录 backlog 中
- Needs: 随着人机交互、评估、浏览器环境、仿真环境、框架案例回填和跨层冲突处理继续推进，本文件需要持续修订

---

*最后更新: 2026-06-08*
