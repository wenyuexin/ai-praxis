# Tool Orchestration

> 适用范围：单智能体中多工具调用序列的组织、约束与执行边界
> 阶段状态：研究主干，持续补充论文案例与工程证据
> 使用说明：本文件回答“为什么 tool use 不应只理解为单次工具选择、tool orchestration 解决什么问题、它与 planning / reasoning / environments 如何耦合、核心 trade-off 是什么”，不直接预设某一种 orchestration 方式为标准答案

## 一、定位

如果说 `tool-use/` 关注单智能体如何发现和调用单个工具，那么本文件进一步讨论的是：**当系统面对多个工具、多个步骤、多个前置条件时，这些工具调用序列究竟如何被组织起来**。

在 single-agent 里，tool orchestration 不应被狭义理解为“多调用几次工具”或“做一个工具路由器”。

更准确地说，它关注的是：

- 工具调用的顺序如何决定
- 多个工具之间的依赖如何表达
- 失败后是局部重试、替换工具，还是回到更高层重规划
- 工具调用结果如何反馈回 reasoning loop、memory 与后续执行

因此，tool orchestration 讨论的不是单个工具能力，而是**工具序列如何被系统化组织**。

---

## 二、为什么它必须单列成主题

### 2.1 tool selection 不能覆盖多步工具工作流

“选哪个工具”只解决单点决策问题，但真实任务经常要求：

- 先检索信息，再写代码，再运行测试
- 先读取文件，再调用 API，再做结果比较
- 先浏览网页，再提取结构化信息，再落盘

这些都不是单次 selection 能回答的，而是 sequence orchestration 问题。

### 2.2 工具一多，复杂度不再只属于 planning

很多人会自然把 orchestration 归到 planning，但这只对了一部分。

因为一旦工具进入执行，系统还必须面对：

- 工具失败如何恢复
- 中间产物如何传递
- 参数如何由上一步结果驱动生成
- 哪些工具适合串行，哪些可以并行

所以 tool orchestration 既和 planning 有关，也和 execution、reasoning、memory、traceability 深度耦合。

### 2.3 不单列它，tool-use 目录会被误缩成协议目录

如果只写 API calling、MCP、code interpreter、web browsing，`tool-use/` 很容易被理解成“工具类型列表”或“调用协议目录”。

单列 orchestration 的价值，就是提醒：单智能体的工具使用核心不只是接口，而是**如何围绕任务目标把工具组织成有效工作流**。

---

## 三、tool orchestration 到底在组织什么

### 3.1 调用顺序

最基础的一层是：

- 哪一步先调用哪个工具
- 哪一步必须等待前一步结果
- 哪一步是否可以跳过或替换

这决定了 agent 是否只是局部试错，还是有可理解的执行结构。

### 3.2 工具依赖

很多工具调用并不是独立动作，而是依赖链的一部分，例如：

- 搜索结果决定后续读取对象
- 文件解析结果决定后续代码修改范围
- 测试输出决定是否需要调用修复工具

这说明 orchestration 不是扁平列表，而是依赖结构。

### 3.3 中间状态与产物传递

多工具调用往往通过中间对象连接：

- 文件
- patch
- structured result
- logs
- task state
- memory update

如果这些对象没有稳定传递方式，工具再多也很难形成可靠工作流。

### 3.4 失败恢复路径

tool orchestration 还必须回答：

- 某个工具失败后，是否直接重试
- 是否换一个工具完成同一子任务
- 是否回到 planning 层重新安排步骤
- 是否终止整个执行链

因此 orchestration 不是“调用成功时的流程图”，而是包含失败分支的执行结构。

---

## 四、几种典型 orchestration 风格

### 4.1 Inline Tool Use

工具调用直接嵌在 reasoning-action loop 中，按 observation 驱动逐步决定下一次调用。

优点：

- 灵活
- 对未知任务更适应
- 与 `ReAct` 风格天然兼容

局限：

- 多工具依赖结构通常不够显式
- 长链任务容易漂移
- 回看时较难解释全局工作流

### 4.2 Planned Tool Sequence

系统先生成一组工具步骤，再按计划逐步执行。

优点：

- 更容易表达顺序与依赖
- 适合结构化任务
- 更利于 trace 与复盘

局限：

- 计划很容易在执行中失效
- 对动态环境适应性较弱

### 4.3 Conditional / Branching Orchestration

系统在步骤之间保留条件分支，根据工具结果进入不同路径。

优点：

- 更贴近真实任务分支
- 能较好处理失败与不同结果类型

局限：

- 状态管理更复杂
- 条件设计不清时会迅速失控

### 4.4 Hierarchical / Bilevel Orchestration

高层负责子任务或工具簇选择，低层负责具体调用与局部执行。

优点：

- 更适合大规模工具生态
- 能把全局结构与局部灵活性分开处理

局限：

- 设计成本高
- planner、executor、tool state 之间接口更难稳定

### 4.5 Hybrid Orchestration

现实系统往往不是纯一种模式，而是：

- 高层 plan-and-execute
- 局部步骤内使用 `ReAct` 风格
- 对某些高风险工具再加入确认或 recovery gate

这种方式更接近工程现实，但也更容易带来边界混乱。

---

## 五、它与其他能力轴如何耦合

### 5.1 与 Planning 的关系

tool orchestration 常被 planner 驱动，但不等于 planning 本身。

planning 更关心目标结构与步骤设计；orchestration 更关心步骤落地时工具如何串起来。

因此可以说：

- planning 决定“做哪些步骤”
- orchestration 决定“这些步骤如何通过工具链实际完成”

### 5.2 与 Reasoning-Acting 的关系

在紧耦合 agent 中，orchestration 很多时候直接长在 reasoning loop 里。

这意味着：

- reasoning 决定是否发起下一次工具调用
- observation 决定是否修改调用路径
- orchestration 在局部循环里动态演化

### 5.3 与 Memory 的关系

多工具工作流如果不借助 memory，很难稳定跨步骤保留：

- 已完成哪些子任务
- 哪些结果已被验证
- 哪些工具尝试已经失败
- 哪些 artifact 可以复用

因此 orchestration 的可靠性往往受 memory 设计直接影响。

### 5.4 与 Environments 的关系

工具序列一旦涉及代码执行、文件修改、网络访问、浏览器操作，它就立刻受到环境层约束：

- 哪些工具被允许
- 工具在哪个 runtime 执行
- 副作用如何隔离
- 失败后如何回滚

所以 orchestration 不能脱离 `05-environments/` 单独讨论。

---

## 六、最重要的结构性矛盾

### 6.1 Tool Selection Simplicity vs Workflow Structure

单次工具选择越简单，系统越轻；但任务一旦变复杂，没有显式工作流结构就很容易失控。

### 6.2 Flexibility vs Predictability

动态 orchestration 更灵活，但也更难预测、复盘和限制；显式 orchestration 更稳定，但容易牺牲适应性。

### 6.3 Local Optimization vs Global Task Efficiency

局部看，每一步都选“当下最合理”的工具似乎足够；但全局看，这可能导致重复调用、无效探索和状态漂移。

### 6.4 Rich Orchestration vs System Overhead

orchestration 越丰富，越能表达复杂依赖与失败分支；但状态管理、traceability、实现复杂度与 token 成本也会同步上升。

---

## 七、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- tool use 的问题主要就是选哪个工具
- orchestration 只是 planning 的附属细节
- 多工具任务一定需要显式 workflow DAG
- `ReAct` 风格不具备 tool orchestration
- orchestration 做得越重越好

这些说法都会过早把工具使用问题压扁成单轴设计，掩盖任务类型、工具规模与执行环境之间的差异。

---

## 八、与其他专题的关系

- 与 `tool-use/README.md`：后者定义 tool-use 目录边界；本文件进一步强调工具使用不只是协议与接口，还包括多工具工作流组织。
- 与 `tool-invocation-reliability.md`：后者更聚焦单次与多次调用为什么会失真、如何做参数约束、结果校验与失败恢复；本文件则更强调工作流层的组织结构。
- 与 `bilevel-planning-tool-navigation.md`：后者把 hierarchical / bilevel orchestration 从“风格类型”进一步展开成独立主题，重点讨论高层规划与低层工具导航如何分层协作。
- 与 `overview.md`：本文件同时承接其中 `Planning-Execution 耦合 vs 解耦` 与 `Tool-centric Design vs Monolithic Agent` 两条关键主轴。
- 与 `planning-vs-execution.md`：工具规模和依赖复杂度会直接推动 planning / execution 从紧耦合走向更显式 orchestration。
- 与 `reasoning-and-acting-loop/README.md`：紧耦合 orchestration 往往直接嵌在 reasoning-action-observation 循环中。
- 与 `memory/`：多工具链的中间状态、失败记录与结果复用依赖 memory 机制。
- 与 `../05-environments/`：代码执行、浏览器、权限与恢复机制会直接塑造 orchestration 的可行边界。
- 与 `backlog.md`：对应 `Tool Selection vs Tool Orchestration` 这一高优先级缺口，也和 `Bilevel Planning for Tool Navigation` 线索相连。

---

## 九、当前最值得继续补证的方向

- 主流 coding agent 与 research agent 在 orchestration 风格上有何典型分布
- 工具依赖图、条件分支与中间产物传递是否存在可复用抽象
- orchestration 应更多由 planner 负责、executor 负责，还是由 reasoning loop 内生演化
- bilevel planning、tool navigation、tool graph 等中间形态是否已经形成稳定模式
- tool orchestration 与 traceability / recovery 应如何共同设计，避免长链任务失控
