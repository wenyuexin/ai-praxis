# Agent vs Tool-Use Workflow Boundary

> 适用范围：单智能体边界定义、最小执行闭环与 tool calling workflow 区分
> 阶段状态：研究主干，持续补充框架口径与系统案例证据
> 使用说明：本文件回答“为什么 tool-use workflow 不应直接等同于 agent、单智能体的最小执行闭环通常包含哪些结构、哪些系统更接近 agent、哪些系统更接近工具增强式 workflow、这个边界为什么长期不稳定”，不直接把某一套定义写成唯一标准

## 一、定位

`02-single-agent/` 一开始就面临一个基础问题：**只要系统能调工具，它就已经是 agent 吗**。

这个问题表面上像术语争执，实际上会影响整个目录的边界：

- `tool-use/` 到底是在描述 agent 的一部分，还是已经足以构成 agent
- `reasoning-and-acting/` 中的循环结构是不是 agent 的必要成分
- `planning/`、`memory/`、`self-reflection/` 这些能力到底是增强项，还是某种最小闭环的组成部分
- 如何区分单次 function calling application 与真正具有持续执行能力的系统

因此，这个主题讨论的不是“agent 这个词怎么营销”，而是：**什么样的系统已经形成了最小执行闭环，什么样的系统仍然更适合被理解为 tool-augmented workflow**。

---

## 二、为什么它必须单列成主题

### 2.1 不单列它，single-agent 的目录边界会不断漂移

如果不明确讨论 agent 边界，`02-single-agent/` 很容易被两种相反方式误写：

- 一种是写得过宽，只要有 function calling 就都算 agent
- 另一种是写得过窄，仿佛一定要有 planner、memory、reflection、multi-step autonomy 才算 agent

这两种写法都会让目录组织失真。

### 2.2 它是多个能力轴的共同前提

这个问题并不只属于 `tool-use/`。

因为一旦讨论“是不是 agent”，就会同时牵涉：

- 是否存在持续 reasoning-action-observation loop
- 是否会根据环境反馈调整后续行为
- 是否保有跨步骤状态
- 是否具备一定程度的自主 continuation，而不是一次调用结束

所以它天然横跨 `reasoning-and-acting/`、`tool-use/`、`planning/`、`memory/`。

### 2.3 它直接影响后续很多专题的判断标准

例如：

- `planning-vs-execution.md` 讨论的是 agent 内部如何耦合/解耦，而不是普通 workflow 是否也有计划步骤
- `tool-orchestration.md` 讨论的是 agent 如何组织多工具链，而不是任意脚本流水线都算 orchestration
- `tool-centric-vs-monolithic.md` 讨论的是两种 agent 组织路径，而不是所有 LLM 应用的分类

如果没有边界文档，这些主干专题会缺少统一前提。

---

## 三、为什么 tool-use workflow 不能直接等同于 agent

### 3.1 单次工具调用只说明“能接外部能力”

一个系统能调用搜索、数据库、代码执行器或浏览器，只能说明它具备 external capability access。

但这还没有回答：

- 它是否会根据结果继续推进任务
- 它是否能重新安排下一步行动
- 它是否有持续状态与中间目标管理
- 它是否形成真正的执行闭环

所以 tool use 是常见特征，但不是自动充分条件。

### 3.2 很多 tool workflow 仍然是一次性线性应用

现实里有大量系统虽然用了 function calling 或外部 API，但整体仍是：

- 用户提问
- 模型选一次工具
- 工具返回结果
- 模型整合并结束

这类系统可以非常有用，但它们很多时候更接近“工具增强式单轮应用”，而不是持续自治的 agent loop。

### 3.3 Agent 更强调 feedback-conditioned continuation

agent 与普通 tool workflow 最关键的差别，通常不在“有没有工具”，而在：

- 有没有后续步骤的持续生成
- 后续步骤是否受到环境反馈影响
- 系统是否会在中间结果基础上修正行为

也就是说，agent 更像一个会持续推进的执行体，而不是一次回答里顺手调了一个工具。

---

## 四、单智能体最小执行闭环通常包含什么

### 4.1 目标承接

系统首先要承接一个任务或目标，而不是只做局部函数映射。

这意味着它至少要知道：

- 当前要完成什么
- 什么结果算完成
- 当前输出只是最终答案，还是中间步骤

### 4.2 局部判断与动作选择

系统需要在某个时点决定：

- 直接回答
- 调工具
- 读取环境反馈
- 继续下一步
- 停止当前路径并换一种做法

如果没有这一层，它更像预定义流水线，而不是 agentic loop。

### 4.3 观察与反馈吸收

动作之后，系统需要读取新的观察：

- 工具结果
- 外部报错
- 检索内容
- 文件变化
- 测试输出

并把这些反馈纳入后续决策。

### 4.4 状态更新与继续执行

最关键的一步是：系统不能把每一步都当成独立调用，而需要更新“我现在处在任务的哪里”。

这类状态可能体现在：

- 上下文历史
- 临时记忆
- 中间 artifact
- 当前 step / subgoal
- 已失败尝试记录

### 4.5 停止条件

一个闭环还必须知道何时结束。

也就是说，agent 不只是会继续，还必须会：

- 判断任务是否完成
- 判断当前路径是否失败
- 判断是否需要交还给用户或人工接管

没有 stopping rule 的系统，只是无界 continuation，不构成稳定 agent 结构。

---

## 五、哪些系统更接近 agent，哪些更接近 tool workflow

### 5.1 更接近 tool-augmented workflow 的系统

通常表现为：

- 工具调用次数少
- 路径基本线性
- 中间状态很弱
- 调完工具后即结束
- 没有显式或隐式的持续执行循环

这类系统常见于：

- 单轮问答接搜索
- 一次性 function calling
- 预定义 API bridge
- prompt → tool → answer 的轻量流程

### 5.2 更接近 single-agent 的系统

通常表现为：

- 会持续生成后续步骤
- 能根据观察结果改变行动
- 具备某种状态延续机制
- 可能在失败后重试、重规划、改调别的工具
- 任务结束不是第一次动作后立刻到来，而是经过多轮推进

这类系统更接近：

- `ReAct` 类执行循环
- coding agent
- browser / research agent
- 带 workspace、memory、recovery 的任务执行系统

### 5.3 边界是连续谱，不是绝对分割线

很多系统都处在中间地带：

- 平时像单轮 workflow，遇到失败才进入多轮修正
- 默认一步完成，但复杂任务时会展开多步 loop
- 有多个工具，但仍主要由单次大模型整合主导

因此更好的问题不是“是不是 agent”二元判定，而是：**它在多大程度上已经形成反馈驱动的执行闭环**。

---

## 六、哪些能力是常见特征，哪些不应被误写成硬门槛

### 6.1 常见但不必然充分的特征

以下特征经常出现在 agent 中，但单独存在时都不应被当作充分条件：

- tool use
- multi-step output
- long context
- memory store
- reflection
- planner

因为这些能力都可能出现在非 agent 系统里。

### 6.2 更接近必要结构的成分

相比之下，以下成分更接近 agent 的核心边界：

- 持续执行循环
- feedback-conditioned continuation
- 中间状态延续
- 基于观察调整后续动作
- 明确或隐式的 stopping logic

这些成分不一定都以同样形式实现，但缺少其中大部分时，系统通常更难称为 agent。

### 6.3 显式 planner 不是硬门槛

很多人会把 agent 直接等同于 planner-driven system，但这会误伤 `ReAct` 这类紧耦合路径。

是否是 agent，更关键的不是有没有独立 planner，而是有没有持续 loop 与反馈驱动调整。

### 6.4 memory 不是硬门槛，但状态延续几乎总是必要

系统未必一定有独立 memory module。

但如果完全没有跨步骤状态延续，就很难形成真正的多步 agent loop。

因此更准确的表述不是“agent 必须有 memory 子系统”，而是“agent 几乎总要有某种状态保持机制”。

---

## 七、最重要的结构性矛盾

### 7.1 Broad Labeling vs Precise Boundary

把大量 tool-use 应用都称作 agent，概念传播更轻松；但知识组织会迅速失真。

### 7.2 Capability View vs Loop View

一种视角按“能力列表”定义 agent；另一种视角按“是否形成持续执行闭环”定义 agent。二者会导向不同边界。

### 7.3 Product Framing vs Research Framing

产品叙事往往倾向更宽泛地使用 agent 一词；研究与系统分析更需要可操作的结构边界。

### 7.4 Simplicity of Demo vs Stability of System

单次 tool calling demo 很容易展示 agent 感；但真正稳定的 agent 系统往往需要状态、反馈、停止条件与恢复机制共同支撑。

---

## 八、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- 只要调用工具就是 agent
- 只要能多轮对话就是 agent
- 没有 planner 就不算 agent
- 没有 memory 数据库就不算 agent
- 只要有 autonomy，就必须是完全无人干预系统

这些说法都会把边界问题压平，忽略 loop、feedback、state 与 stopping logic 的组合关系。

---

## 九、与其他专题的关系

- 与 `overview.md`：本文件展开其中“单智能体并不等于任何 tool calling workflow”这一基础边界，并为“最小执行闭环”提供更明确表达。
- 与 `reasoning-and-acting/README.md`：持续 reasoning-action-observation loop 是区分 agent 与普通 workflow 的关键结构之一。
- 与 `tool-use/README.md`：tool use 是常见 agent 能力，但本文件强调它不是自动充分条件。
- 与 `planning-vs-execution.md`：只有先确认系统已构成 agent loop，才有意义继续讨论其 planning / execution 如何耦合。
- 与 `tool-orchestration.md`：orchestration 关注多工具链如何组织；本文件更关注什么样的组织程度已经跨过普通 workflow 边界。
- 与 `tool-centric-vs-monolithic.md`：无论更偏 tool-centric 还是 monolithic，本文件都先回答“它是否已经构成 agent”这个更基础的问题。
- 与 `backlog.md`：本文件对应当前 P0 缺口 `Agent 是否等于 Tool-Use Workflow`，并为后续 long context、planning necessity 等主题提供边界基线。
- 与 `../05-environments/`：环境提供反馈、可执行性与恢复能力，也会影响某个系统能否稳定维持持续执行闭环。

---

## 十、当前最值得继续补证的方向

- 主流框架与厂商文档如何定义 agent，与研究论文定义差异有多大
- 单次 tool workflow、轻量 loop 与重型 agent system 之间是否存在更稳定的中间分层
- 在 coding / browser / research 场景中，哪些最小结构一旦出现，就足以把系统从 workflow 推向 agent
- 长上下文与更强模型能力是否会削弱“显式 loop”作为边界特征的重要性
- 是否能抽象出一套更稳的 agent boundary checklist，用于区分 demo、workflow 与 agent system
