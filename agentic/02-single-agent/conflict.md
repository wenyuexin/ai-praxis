# Single-Agent Conflict

> 适用范围：`02-single-agent/` 范围内的术语冲突、事实分歧、结论矛盾与待核验问题
> 使用说明：本文件只记录会影响 `planning / memory / tool-use / reasoning-and-acting-loop / self-reflection / patterns` 知识组织的问题；它是研究输入，不是主干定论

## 一、待核验问题

### 1.1 Agent 是否等于 tool-use workflow

- **冲突类型**：术语 / 定义
- **冲突描述**：不同资料对“只要有 tool calling 是否就算 agent”存在明显分歧。有的把 function calling workflow 直接称为 agent；有的则要求具备持续执行循环、环境反馈、状态更新与自主调整。
- **涉及范围**：`overview.md`、`tool-use/`、`reasoning-and-acting-loop/`
- **为什么重要**：这会直接影响 `02-single-agent/` 的边界，以及 agent 与普通工具增强式 LLM application 的区分方式。
- **待核验问题**：agent 的最小必要条件应如何定义？tool use 在其中是必要条件、充分条件，还是常见但非本质特征？
- **建议处理方向**：优先查看 agent 系统综述、ReAct 论文、主流框架官方定义，整理“执行循环”与“单次工具调用”的边界。

### 1.2 Memory Taxonomy 不兼容

- **冲突类型**：术语
- **冲突描述**：`short-term / long-term`、`working / episodic / semantic`、`token-level / parametric / latent`、`factual / experiential / working` 等分类体系之间难以直接映射。
- **涉及范围**：`memory/`、`overview.md`
- **为什么重要**：若不显式记录，`memory/` 子目录会被误解为稳定 taxonomy，而实际上该领域仍在快速重构。
- **待核验问题**：这些 taxonomy 各自适用于什么研究语境？哪些分类是在“功能”层，哪些是在“存储形式”或“动态过程”层？
- **建议处理方向**：优先整理近年综述论文，区分不同 taxonomy 的维度来源，避免强行一一映射。

### 1.3 Long Context 是否可以替代独立 Memory System

- **冲突类型**：结论
- **冲突描述**：一类观点认为大 context window 足以减少额外 memory mechanism 的必要性；另一类观点认为长上下文仍然无法解决忠实性衰减、检索噪声与状态管理问题。
- **涉及范围**：`memory/`、`tool-use/`、`overview.md`
- **为什么重要**：会影响 `memory/` 的定位，以及是否把 memory 视为一等能力还是长上下文的附属优化。
- **待核验问题**：context window 扩展是否真的改变了 memory system 的必要性，还是只是改变了 memory 设计形式？
- **建议处理方向**：收集长上下文模型研究、agent memory 综述、实际系统案例，区分“容量提升”和“可靠性提升”。

### 1.4 Planning 是否必须显式存在

- **冲突类型**：结论 / 定义
- **冲突描述**：部分资料默认认为高质量 single-agent 必须包含显式 planner 或 DAG；另一类实践则表明 `ReAct` 式推理—行动循环已能覆盖大量任务。
- **涉及范围**：`planning/`、`reasoning-and-acting-loop/`、`architectural-patterns/react/`
- **为什么重要**：会影响 `planning/` 在目录中的表达方式，以及是否把 planner 误写成所有 single-agent 的基础前提。
- **待核验问题**：显式 planning 在哪些任务中是必要结构，哪些任务中只是可选增强？
- **建议处理方向**：对比 ReAct、plan-and-execute、bilevel planning 等代表路径，按任务复杂度与工具规模分类讨论。

### 1.5 Reflection 的收益是否稳定大于成本

- **冲突类型**：结论
- **冲突描述**：self-reflection 通常被认为能提升复杂任务表现，但其收益边界、最优反思轮数和 token / latency 开销并没有稳定共识。
- **涉及范围**：`self-reflection/`、`overview.md`
- **为什么重要**：如果不记录冲突，容易把 reflection 写成默认正向能力，而忽略其显著成本。
- **待核验问题**：reflection 在什么任务类型中真正带来净收益？何时会变成过度计算？
- **建议处理方向**：优先查看 reflection 论文、agent benchmark 结果与系统实证，建立收益/成本同时观察的视角。

---

## 二、维护规则

- 若问题已获得较稳定证据并形成主干结论，应同步更新相关文档并移除或改写对应条目。
- 若问题扩展到 `03-multi-agent/` 或 `05-environments/`，应在对应目录的 `conflict.md` 建立镜像条目，而不是长期只保留在本文件。
- 若问题只影响临时架构判断、不适合进入主干目录范围，可先保留为临时冲突整理；一旦进入本目录稳定讨论面，应尽快收束到当前 `conflict.md`。
