# Planning vs Execution Coupling

> 适用范围：单智能体中 planning 与 execution 的耦合/解耦关系、典型模式与适用边界
> 阶段状态：研究主干，持续补充论文案例与工程证据
> 使用说明：本文件回答“为什么 planning 与 execution 的关系是 single-agent 的关键设计轴、紧耦合与解耦分别解决什么问题、核心 trade-off 是什么”，不直接预设某一种模式为标准答案

## 一、定位

如果说 `planning/` 关注单智能体如何生成计划，`reasoning-and-acting-loop/` 关注推理—行动—观察循环，那么本文件进一步讨论的是：**planning 与 execution 在系统中究竟应该多紧地耦合在一起**。

在 single-agent 里，这不是一个小的实现细节，而是一条非常核心的设计轴。

因为很多看似不同的模式，本质上都在回答同一个问题：

- 计划应该在执行前尽量展开，还是在执行过程中逐步生成
- 执行反馈应只局部修补当前动作，还是反向改变全局计划
- 系统更像一个持续局部决策循环，还是一个“先定结构、再逐步落实”的过程

因此，planning vs execution 讨论的不是“有没有 planner”，而是**规划结构与执行循环之间的关系如何被组织**。

---

## 二、为什么它必须单列成主题

### 2.1 这决定 single-agent 的基本工作方式

单智能体的很多关键差异，并不在“是否会调用工具”本身，而在于：

- 工具调用前有没有明确步骤结构
- 失败后是局部重试还是全局重规划
- 观察反馈是立即影响下一步，还是先回到 planner 再统一修正

也就是说，同样是 tool-using agent，可能因为 planning / execution 关系不同而形成完全不同的执行风格。

### 2.2 这是 ReAct 与 Plan-and-Execute 分野的上游问题

很多讨论会直接拿 `ReAct` 和 `plan-and-execute` 做对比，但如果不抽象出更上层的设计轴，容易把它们写成两个离散模式。

更合适的理解是：

- `ReAct` 更偏 planning 与 acting 紧耦合
- `plan-and-execute` 更偏 planning 与 execution 显式分离

而大量真实系统其实分布在这两端之间。

### 2.3 一旦不单列，planning、reasoning、tool-use 会反复混写

如果不把这个主题拆出来，后续很容易出现三种混乱：

- 把 planner 当成所有 agent 的必要部件
- 把 reasoning loop 当成没有 planning 的系统
- 把 tool orchestration 问题误写成单纯工具选择问题

单列这一主题的价值，就是先把“计划结构”和“执行循环”这条主轴单独澄清。

---

## 三、两种典型极端

### 3.1 Tightly Coupled Planning-Execution

这一端的典型形态是：系统在每一步边想边做，把规划、行动、观察与局部修正交织在同一循环中。

典型特征：

- 计划粒度较局部
- 每轮 observation 很快进入下一轮决策
- planner 往往不是独立阶段，而是嵌在 action loop 里
- 更接近 `thought → action → observation → next thought`

代表性风格包括：

- `ReAct`
- 一些轻量 tool-using agent loop
- 偏交互式、探索式任务中的局部决策模式

### 3.2 Loosely Coupled / Decoupled Planning-Execution

另一端的典型形态是：系统先形成较明确的计划结构，再按步骤执行，并在必要时回到 planning 层修正整体结构。

典型特征：

- 计划粒度更全局
- execution 负责落实已有步骤
- 失败或偏差常触发 re-plan，而不是只做局部动作修补
- planner 更像显式模块，而不是隐含在每一步 thought 中

代表性风格包括：

- `plan-and-execute`
- 分阶段 orchestration workflow
- 对长任务、复杂工具序列更敏感的执行系统

---

## 四、这条轴上真正变化的是什么

### 4.1 计划粒度

系统规划得越早、越细，越接近解耦端；越依赖即时判断，越接近紧耦合端。

关键问题包括：

- 计划是高层目标列表，还是可直接执行步骤
- 计划是否包含工具、参数与前置条件
- 计划是否允许执行中持续增删结构

### 4.2 反馈注入位置

execution 的反馈可以注入不同层次：

- 只影响下一步局部动作
- 触发当前子计划修正
- 触发全局计划重建

反馈注入层次越高，planning 与 execution 越趋向解耦；越低，则越趋向紧耦合。

### 4.3 状态管理方式

紧耦合系统通常把更多状态保留在当前 loop 中；解耦系统则更依赖显式的计划表示、步骤状态与执行记录。

因此这条轴也直接影响：

- traceability 怎么做
- memory 更新在哪里发生
- failure recovery 是局部还是全局

### 4.4 工具组织方式

当工具数量和依赖关系变复杂时，planning / execution 的关系会被进一步放大：

- 少量工具下，局部决策往往足够
- 大量工具、多前置条件或多阶段任务下，更显式的 orchestration 往往更重要

所以这条轴并不是抽象偏好，而会被工具生态规模推着变化。

---

## 五、紧耦合端解决什么，不解决什么

### 5.1 优势

- 更灵活
- 对未知环境与探索式任务更友好
- observation 能快速回流到下一步行动
- 实现相对直接，不一定需要厚 planner

### 5.2 局限

- 全局结构容易不稳定
- 长任务中更容易漂移
- 多工具序列与前置依赖更难显式管理
- 回头分析“为什么这样做”时，结构解释性通常较弱

因此紧耦合端并不是“没有 planning”，而是 planning 被压缩到了局部循环里。

---

## 六、解耦端解决什么，不解决什么

### 6.1 优势

- 更适合长链任务
- 更容易表达阶段性目标和前置依赖
- 更利于 trace、checkpoint、step state 管理
- 更适合把失败定位到计划层还是执行层

### 6.2 局限

- 前期规划成本更高
- 计划可能很快过时
- 对动态环境和探索式任务未必足够灵活
- planner / executor 接口一旦设计不好，会引入额外复杂度

因此解耦端也不是“更高级”，而是更偏结构化任务组织。

---

## 七、最重要的结构性矛盾

### 7.1 Local Reactivity vs Global Coherence

紧耦合端更容易保持局部反应速度；解耦端更容易维持全局结构一致性。

### 7.2 Planning Overhead vs Execution Drift

前置规划越多，启动和维护成本越高；前置规划越少，执行中越容易漂移和反复试错。

### 7.3 Explicit Structure vs Adaptive Flexibility

显式结构更利于管理、审计和复盘；适应性更强的局部循环更适合动态环境，但也更难稳定控制。

### 7.4 Planner Simplicity vs Orchestration Depth

planner 越薄，系统越轻；但任务和工具越复杂，缺少显式 orchestration 往往会让复杂度转移到 loop 内部，最后变得更难理解。

---

## 八、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- 高质量 single-agent 一定需要显式 planner
- `ReAct` 没有 planning
- planning 与 execution 解耦一定优于紧耦合
- 长任务一定只能靠 plan-and-execute
- tool orchestration 只是 planning 的附属问题

这些说法都会过早把单一模式写成普适结论，掩盖任务形态、工具规模与环境动态性之间的差异。

---

## 九、与其他专题的关系

- 与 `overview.md`：本文件展开其中 `Planning-Execution 耦合 vs 解耦` 这一核心结构性矛盾。
- 与 `planning/README.md`：后者给出 planning 目录边界；本文件进一步讨论 planning 在系统中如何与 execution 组织关系。
- 与 `reasoning-and-acting-loop/README.md`：紧耦合端的大量模式会落到 reasoning-action loop 的局部循环中。
- 与 `planning/plan-and-execute/README.md`：该目录对应解耦端的重要代表模式之一。
- 与 `architectural-patterns/react/README.md`：该目录对应紧耦合端的重要代表模式之一。
- 与 `tool-use/README.md`：工具规模与调用复杂度会直接推动 planning / execution 关系的变化。
- 与 `conflict.md`：对应 `Planning 是否必须显式存在` 这一条目，也为后续讨论 tool orchestration 提供主干支撑。

---

## 十、当前最值得继续补证的方向

- 哪些任务最适合紧耦合 loop，哪些任务更需要显式 plan-and-execute 结构
- bilevel planning、tool orchestration 等中间形态是否已经形成稳定模式
- 计划粒度、反馈层级与 recovery 机制之间是否存在可复用组合框架
- planning / execution 解耦后，memory、traceability、reflection 分别应挂在哪一层
- 主流 coding agent 与 research agent 在这条轴上的典型分布有何差异
