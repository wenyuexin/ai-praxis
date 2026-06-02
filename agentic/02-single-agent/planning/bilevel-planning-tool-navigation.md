# Bilevel Planning for Tool Navigation

> 适用范围：单智能体中高层规划与低层工具导航分层设计、适用边界与中间形态
> 阶段状态：研究主干，持续补充大规模工具生态与复杂任务案例证据
> 使用说明：本文件回答“为什么在工具数量和依赖复杂度上升时会出现 bilevel / hierarchical planning、它与 `ReAct` 和 plan-and-execute 分别是什么关系、它主要解决什么问题、代价在哪里”，不直接把这种分层结构写成唯一标准答案

## 一、定位

如果说 `planning-vs-execution.md` 讨论的是 planning 与 execution 的耦合/解耦主轴，`tool-orchestration.md` 讨论的是多工具工作流如何被组织，那么本文件进一步讨论的是一个更具体的中间形态：**当工具生态变大、依赖关系变复杂时，系统为什么会把“高层任务结构”和“低层工具导航”拆成两层来处理**。

这里的关键点不是简单地“多加一个 planner”。

更准确地说，bilevel planning 要回答的是：

- 高层负责决定什么子任务、什么阶段结构、什么工具簇
- 低层负责在当前子任务内具体选工具、组参数、执行与恢复
- 两层之间如何传递状态、失败信号与重规划触发条件

因此，本主题讨论的不是一般意义上的 planning，也不是单纯的 tool routing，而是：**规划层与工具导航层如何分层协作**。

---

## 二、为什么它必须单列成主题

### 2.1 它不是单纯的“更复杂 orchestration”

很多人会把 bilevel / hierarchical tool navigation 直接当成 orchestration 的一个重型版本。

但它真正特殊的地方在于：

- 高层不直接决定每一个具体调用
- 低层也不完全脱离全局目标自由探索
- 系统需要在 planner 与 executor 之间保留明确接口

这已经不是“多写几条工具步骤”的量变，而是结构上的分层。

### 2.2 它处在 `ReAct` 与 plan-and-execute 之间的关键中间地带

如果只用两端描述 single-agent：

- 一端是紧耦合 `ReAct`
- 一端是显式 plan-and-execute

那很多真实系统会被压扁。

而 bilevel planning 的价值就在于，它提供了一个更贴近现实的中间层：

- 高层有全局结构
- 低层保留局部灵活性
- 失败不一定直接全局重来

### 2.3 它直接回应大规模工具生态下的扩展问题

工具少的时候，单层 loop 往往足够。

但一旦系统面对：

- 多种工具簇
- 多阶段任务
- 明显前置条件依赖
- 多种恢复路径

单一层次的 planning 或 routing 就很容易失控。

因此这条主题本质上是在回答：**single-agent 如何在工具规模上升后仍维持可管理的结构**。

---

## 三、什么叫 bilevel planning for tool navigation

### 3.1 高层：任务结构与子任务分配

高层通常更关注：

- 当前任务要拆成哪些子任务
- 哪些子任务需要哪类工具能力
- 执行顺序与阶段边界是什么
- 哪些步骤失败后要局部修补，哪些必须回到上层重构

也就是说，高层更像在组织任务骨架，而不是直接发出每一条 invocation。

### 3.2 低层：局部工具导航与执行

低层通常更关注：

- 当前子任务选哪个具体工具
- 参数如何构造
- 工具失败后是否先重试或换工具
- 当前局部结果是否足以返回高层

这一层更贴近真实 invocation 层面。

### 3.3 两层之间的关键接口

bilevel planning 是否成立，关键不只是“两层都存在”，而是接口是否清楚：

- 高层交给低层的到底是目标、步骤、约束还是候选工具簇
- 低层回传给高层的是结果、状态摘要、失败类型还是重规划信号
- 哪些失败在低层自消化，哪些失败必须上浮

如果接口不稳定，分层很容易变成额外负担。

---

## 四、它与其他常见模式的关系

### 4.1 与 `ReAct` 的关系

`ReAct` 更接近：

- 局部 thought-action-observation loop
- planning 与 acting 紧耦合
- 工具导航内生在当前执行循环中

而 bilevel planning 则会把一部分规划职责上提：

- 低层仍可能局部使用 `ReAct`
- 但高层开始负责更全局的子任务与工具簇结构

所以 bilevel 不是 `ReAct` 的反面，更像是它在复杂工具生态下的一种上层扩展。

### 4.2 与 plan-and-execute 的关系

plan-and-execute 更容易被理解为：

- 先有明确计划
- 再按步骤执行
- 执行偏差时回到 planner

bilevel planning 则通常更细：

- 上层 plan 未必细到每个具体工具调用
- 低层仍有局部导航空间
- 某些偏差只在低层修复，不必立刻升级为全局 replan

所以它经常可以被看作 plan-and-execute 的细粒度中间版本。

### 4.3 与 hierarchical orchestration 的关系

`tool-orchestration.md` 里提到的 hierarchical / bilevel orchestration，本文件就是把其中那一类模式从“风格”进一步提升成“专题”。

重点不再只是“有上下层”，而是讨论：

- 为什么分层会出现
- 分层到底在分什么
- 分层带来的收益和代价如何平衡

---

## 五、它主要解决什么问题

### 5.1 大工具空间下的搜索压力

当可用工具、工具簇或执行路径变多时，单层 loop 每一步都在全量空间里局部选择，会很容易：

- 搜索过宽
- 决策不稳定
- 重复试错
- 难以形成全局方向

高层先缩小子任务与工具簇范围，可以显著降低低层搜索压力。

### 5.2 任务结构与 invocation 细节分离

很多任务的真正难点并不在同一层，例如：

- 高层难点是阶段结构与依赖顺序
- 低层难点是具体工具选择、参数生成与失败恢复

如果把这两种问题混在一个 loop 里，系统会很容易在全局结构和局部执行之间来回打架。

### 5.3 失败分层处理

bilevel 结构的一个重要价值是：

- 局部调用失败，不必立刻推翻整个全局计划
- 但结构性失败也不应永远在低层反复修补

这使得 recovery path 可以更有层次：

- 低层 retry / substitution
- 中层子任务重排
- 高层整体 replanning

### 5.4 更适合与 traceability / workspace state 对接

一旦任务有多个阶段与多个工具簇，显式分层会更容易：

- 记录当前处在哪个阶段
- 标记某个子任务是否完成
- 识别失败发生在结构层还是 invocation 层
- 和 workspace / artifact / checkpoint 对齐

---

## 六、它的代价在哪里

### 6.1 接口设计复杂

bilevel 系统最大的问题之一，就是 planner、executor、tool state 之间接口不容易稳定。

如果接口太粗：

- 高层给低层的信息不够
- 低层容易偏航

如果接口太细：

- 高层又会重新退化成控制每一个调用的重型 planner

### 6.2 状态同步更难

一旦分层，系统就需要回答：

- 低层最新状态如何同步给高层
- 高层计划变更如何反映到低层
- 哪些中间 artifact 应共享，哪些只在局部有效

这使得它比单层 loop 更依赖显式状态管理。

### 6.3 失败升级边界不清时容易抖动

如果系统不知道某类失败应该：

- 继续在低层修补
- 还是升级到高层 replanning

就会出现两种坏情况：

- 过早上浮，系统变得笨重
- 过晚上浮，低层陷入无效循环

### 6.4 成本高于轻量 loop

分层往往意味着：

- 更多控制逻辑
- 更多状态对象
- 更多 trace 与接口维护
- 更高实现复杂度

所以它只在复杂度足够高时才更容易体现净收益。

---

## 七、哪些场景更可能需要它

### 7.1 大规模工具生态

当系统要在多个工具簇之间导航时，bilevel 结构通常更自然，例如：

- 先决定用浏览、检索还是代码执行工具簇
- 再在当前簇内部做具体调用

### 7.2 多阶段任务

例如：

- 先搜集信息，再整理结构，再生成产物，再验证结果
- 先理解代码库，再定位修改点，再改动，再运行验证

这类任务天然存在阶段边界，适合高层/低层分层。

### 7.3 高失败成本与强恢复需求任务

如果错误代价高，系统通常更希望：

- 局部错误先局部修补
- 结构性错误再升级到更高层

而不是让所有失败都在同一层次处理。

---

## 八、最重要的结构性矛盾

### 8.1 Global Guidance vs Local Flexibility

高层结构越清晰，低层自由度越受限；低层越灵活，高层越难维持整体方向。

### 8.2 Search Reduction vs Interface Overhead

通过分层减少搜索空间是收益；但新增接口、状态同步与升级逻辑是成本。

### 8.3 Local Recovery vs Global Replanning

哪些失败该在低层消化，哪些失败该升级到高层，是 bilevel 系统最难稳定的一条边界。

### 8.4 Structural Clarity vs Engineering Complexity

分层让系统更可解释、更适合 trace；但也明显提高实现复杂度。

---

## 九、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- 工具一多，就一定需要 bilevel planning
- hierarchical orchestration 天然优于 `ReAct`
- 高层 planner 越细越好
- 只要有两层结构，就自然解决 tool navigation 问题
- bilevel planning 只是 plan-and-execute 的重命名

这些说法都会忽略任务类型、工具规模、失败成本与状态管理能力之间的真实差异。

---

## 十、与其他专题的关系

- 与 `planning/README.md`：后者定义 planning 目录边界；本文件进一步讨论 planning 在复杂工具生态中如何通过高低层分工落地。
- 与 `tool-use/README.md`：后者定义 tool-use 目录边界；本文件则把工具导航问题提升到“高层结构如何约束低层调用”的层面。
- 与 `planning-vs-execution.md`：本文件可以视为其在复杂工具场景下的一种中间形态展开，解释紧耦合与解耦之间为什么会出现分层结构。
- 与 `tool-orchestration.md`：后者先提出 hierarchical / bilevel orchestration 风格；本文件进一步讨论其为什么出现、如何分层、代价在哪里。
- 与 `explicit-planning-necessity.md`：本文件并不假定显式 planner 总是必要，而是说明在某些高复杂度工具导航任务里，高层显式 planning 更可能产生净收益。
- 与 `tool-invocation-reliability.md`：低层 tool navigation 的可靠性，决定了 bilevel 结构能否真正成立；否则高层结构再清晰也会被底层调用失真拖垮。
- 与 `../05-environments/`：workspace state、traceability、rollback 与 recovery 机制，会直接影响高层/低层之间状态同步与失败升级的可行性。
- 与 `backlog.md`：本文件对应当前 P0 缺口 `Bilevel Planning for Tool Navigation`，并为后续继续补 input reformulation、state persistence 与更细粒度 navigation topic 提供上位框架。

---

## 十一、当前最值得继续补证的方向

- coding agent、browser agent、research agent 在 bilevel planning 上的典型分层方式是否存在稳定模式
- 高层到底更适合输出子任务、工具簇，还是带约束的局部 plan skeleton
- 哪些失败最适合在低层消化，哪些失败应强制升级到高层 replanning
- bilevel 结构的主要收益，究竟来自搜索空间缩小，还是来自更好的 recovery / traceability
- 是否能形成一套更稳的 bilevel planning checklist，用于比较不同 agent 系统
