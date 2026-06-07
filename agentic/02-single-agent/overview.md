# Single-Agent Overview

> 适用范围：单智能体能力层的整体认知框架
> 阶段状态：研究综述，持续补充
> 使用说明：本文件回答“单智能体系统在能力层面由哪些关键问题构成、这些问题之间如何耦合、当前有哪些核心矛盾”，不负责记录具体框架实现细节

## 一、定位

`02-single-agent/` 关注单个智能体如何在任务执行中形成一个相对闭环的能力系统：接收目标、组织上下文、规划行动、调用工具、接收反馈、更新状态并在必要时自我修正。

这里的“单智能体”并不等于“只调用一次模型”，也不等于“任何 tool calling workflow”。更关键的边界在于：系统是否具有持续的执行循环，能在环境反馈和中间状态基础上不断调整后续步骤。

因此，本目录要解决的问题不是“有哪些单 Agent 框架”，而是：

- 单智能体最小闭环由哪些能力组成？
- 这些能力之间通过什么机制相互耦合？
- 哪些问题已经形成相对稳定共识，哪些仍然存在明显分歧？
- 当单智能体扩展到更复杂任务时，瓶颈通常出现在哪里？

---

## 二、核心能力轴

### 2.1 Planning

规划负责把目标转换成可执行步骤，包括任务拆解、步骤排序、局部修正与失败后的重规划。

需要注意：

- 不是所有高性能单智能体都需要显式 planner。
- `ReAct` 一类模式中，规划常与行动交织在同一循环里。
- 当任务复杂度或工具规模上升时，planning 与 execution 的耦合/解耦会成为关键设计轴。

### 2.2 Reasoning and Acting

推理—行动循环决定单智能体如何在每一步进行局部判断、触发动作、读取观察并更新后续决策。

这一层往往是单智能体最直观的“执行引擎”，也是 tool use、reflection、memory 真正发生耦合的位置。

### 2.3 Tool Use

工具使用并不只是“能不能调接口”，而是涉及：

- 如何发现和选择工具
- 如何表达参数和约束
- 如何处理工具失败与异常反馈
- 如何在工具调用前后保持正确的任务语境

当前一个重要观察是：tool use 的瓶颈很多时候不在模型本身，而在于任务上下文组织、调用边界和执行反馈设计。

### 2.4 Memory

记忆是单智能体跨步骤、跨阶段甚至跨任务保留和调用信息的机制。

当前该领域最值得强调的事实不是“已有标准 taxonomy”，而是**分类体系仍然碎片化**。`short-term / long-term`、`working / episodic / semantic`、`parametric / contextual / latent` 等视角并不完全兼容。

因此，本目录对 memory 的组织应保持开放，不应过早把某一套分类写成定论。

### 2.5 Self Reflection

自我反思负责识别错误、批评中间产物、执行修正与迭代 refinement。

它的重要性已经被广泛认识，但收益边界并不稳定：

- reflection 往往能提升复杂任务表现
- 但它也会增加 token cost、latency 和执行链长度
- 当前仍缺乏关于“何时值得反思、反思几轮最优”的稳定共识

### 2.6 Patterns

模式层关注常见的工作流抽象，如 `ReAct`、`AutoGPT pattern`、`RA.Aid` 等。它的价值不在于树立“标准答案”，而在于帮助比较不同能力组合方式的收益与代价。

---

## 三、当前最重要的结构性矛盾

### 3.1 上下文完整性 vs 检索噪声

memory 系统必须在两种需求之间平衡：

- 尽量保留更多历史上下文，避免信息丢失
- 控制上下文体积和检索噪声，避免 token 预算和忠实性崩溃

这不是临时工程问题，而是单智能体长期存在的根本矛盾。

### 3.2 Planning-Execution 耦合 vs 解耦

一端是 `ReAct` 式紧耦合循环：简单直接、局部灵活；另一端是更显式的规划—执行分离：全局结构更清晰，但也引入更高状态管理成本。

这个矛盾会随着工具规模和任务跨度增加而不断放大。

### 3.3 Tool-centric Design vs Monolithic Agent

单智能体到底应该被理解为：

- 一个尽量强大的统一模型，通过大上下文完成更多判断
- 还是一个围绕 trusted tools、external memory 和 external protocols 组织起来的 orchestrator

这是 single-agent 研究和工程实践中都非常关键的一条主轴。

### 3.4 Reflection 收益 vs 计算成本

reflection 常常有效，但不是零成本能力。它与 token、latency、执行复杂度直接相关，当前更像一条需要持续测量的 trade-off，而不是已经收敛的最佳实践。

---

## 四、最容易被误写成定论的问题

以下内容在当前阶段更适合保持谨慎：

- `tool use = agent`
- “显式 planner 是 agent 的必要组成部分”
- 某一套 memory taxonomy 已经稳定成为标准
- 长 context window 可以替代独立 memory system
- reflection 一定带来净收益

这些问题都更适合在 `conflict.md` 或专题文档中保留冲突与证据差异，而不是在 overview 中写成单向结论。

---

## 五、建议的目录理解方式

可把 `02-single-agent/` 理解为围绕执行循环展开的能力系统：

```text
目标
  → planning
  → reasoning and acting
  → tool use
  → observation / feedback
  → memory update
  → self reflection
  → 再次进入下一轮执行
```

其中：

- `planning/` 决定步骤结构
- `reasoning-and-acting-loop/` 决定局部循环
- `tool-use/` 决定外部能力边界
- `memory/` 决定跨时间状态保持
- `self-reflection/` 决定修正与迭代能力
- `architectural-patterns/` 负责总结这些能力如何被组合

---

## 六、与其他领域的概念关系

单智能体层关注执行循环中的能力闭环，因此会持续引用 foundations 层的 agent 定义与系统模型，并与多智能体、环境、框架工具和评估等主题形成稳定的概念耦合：例如环境会改变工具执行与状态保持方式，框架工具会把这些能力组合成具体系统，而评估则检验这些能力闭环在真实任务中的有效性与成本边界。

具体目录分工与导航入口由 `agentic/README.md` 和各目录 `README.md` 承担。

---

## 七、当前最值得补齐的专题

- planning 与 execution 解耦程度
- tool orchestration 与 tool selection 边界
- memory taxonomy 冲突与统一框架
- reflection 的收益/成本边界
- tool-centric vs monolithic 的适用场景比较

这些内容更适合作为后续专题文档、`backlog.md` 或 `conflict.md` 的输入，而不是现在就在 overview 中写死答案。