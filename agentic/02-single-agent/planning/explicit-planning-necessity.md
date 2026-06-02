# Explicit Planning Necessity

> 适用范围：单智能体中显式 planner / plan representation 的必要性、适用边界与替代路径
> 阶段状态：研究主干，持续补充论文案例与工程证据
> 使用说明：本文件回答“为什么显式 planning 不能被默认写成 single-agent 的必要条件、它在哪些任务里更重要、哪些系统即使没有独立 planner 也仍然是 agent、显式 planning 与 reasoning loop 的关系是什么”，不直接把任一路径写成通用最优解

## 一、定位

在 `02-single-agent/` 中，planning 是核心能力轴之一，但这很容易被误读成另一个更强的说法：**高质量 single-agent 一定要有显式 planner**。

这正是本文件要拆开的地方。

更准确的问题不是“agent 有没有 planning”，而是：

- planning 是否必须以独立 planner 的形式存在
- 计划是否一定要在执行前被显式展开
- 没有 planner 的系统，是否仍可能通过 reasoning-action loop 表现出有效的局部规划
- 什么情况下显式 planning 真正带来不可替代的收益

因此，本主题讨论的不是“规划是否重要”，而是：**显式 planning 到底是不是 single-agent 的必要结构**。

---

## 二、为什么它必须单列成主题

### 2.1 不单列它，planning 目录很容易被误写成前提目录

如果只看到 `planning/` 目录，读者很容易自然推断：

- 既然 planning 是主目录，那 planner 应该是所有 agent 的基础部件
- 没有显式计划的系统，只是“弱一点的 agent”
- `ReAct` 这类模式只是“还没规划好”的过渡方案

这些推断都会把 single-agent 的真实连续谱压扁。

### 2.2 它是 `ReAct` 与 `plan-and-execute` 之间的上游分歧

很多文档会直接比较：

- `ReAct`
- `plan-and-execute`
- 各种分层 planner / executor 结构

但这些模式真正分歧的上游问题是：**系统是否需要把计划显式抽出来，作为独立对象或独立阶段管理**。

如果不先把这个问题单列出来，后续模式比较会缺少统一判断框架。

### 2.3 它会影响 agent 的定义与实现判断

如果把显式 planner 当作硬门槛，那么很多依靠持续 reasoning-action-observation loop 运行良好的系统都会被低估。

反过来，如果完全否认显式 planning 的价值，又会忽视复杂任务中：

- 全局步骤结构
- 子任务依赖
- 长链失败恢复
- 可解释执行计划

这些问题的真实需求。

---

## 三、为什么“没有显式 planner”不等于“没有 planning”

### 3.1 planning 可以以内生形式存在

很多 agent 并不会先输出一份独立计划，再开始执行。

它们更常见的是：

- 在当前 observation 基础上决定下一步
- 边执行边形成局部步骤结构
- 根据反馈即时修正后续动作

这种系统仍然具有 planning 成分，只是 planning 没有被抽成独立模块。

### 3.2 `ReAct` 不是“无规划”，而是规划与行动紧耦合

`ReAct` 常被误写成“没有 planner 的反面案例”，但更准确的说法是：

- planning 与 acting 交织在同一 loop 中
- 每一步 thought 都可能承担局部计划作用
- 计划结构更多以内联、短周期、局部更新的形式存在

所以它不是 absence of planning，而是 absence of explicit separated planner。

### 3.3 局部规划和全局规划不是一回事

很多任务只需要：

- 判断下一步干什么
- 根据新反馈调整后续动作
- 对局部失败做修正

这类任务未必需要完整全局 plan representation。

因此，“有没有 planning” 与 “有没有显式全局 planner” 必须分开讨论。

---

## 四、显式 planning 真正提供了什么

### 4.1 全局步骤结构

显式 planning 最直接的价值是把任务拆成：

- 可见的步骤
- 子任务依赖
- 执行顺序
- 阶段性目标

这对长链任务尤其重要。

### 4.2 更稳定的任务分解

当任务跨度较大、目标不止一个时，显式 plan 往往能帮助系统：

- 避免一开始就陷入局部路径
- 先建立整体框架
- 在复杂问题上减少盲目试错

### 4.3 更好的可解释性与可复盘性

显式 planning 让系统更容易回答：

- 为什么先做这一步
- 当前在整体任务中处于哪里
- 哪个步骤失败导致整体偏移

因此它不仅是执行机制，也和 traceability、debugging 紧密相关。

### 4.4 更适合与工具链和外部状态系统对接

当系统需要：

- 多工具 orchestration
- workspace state
- rollback / retry
- artifact lineage

显式 plan 往往更容易成为这些机制的上层骨架。

---

## 五、哪些场景里显式 planning 更可能重要

### 5.1 长跨度多步任务

任务越长、步骤越多，系统越容易因为局部 loop 失去全局方向。

这时显式 planning 往往更有价值。

### 5.2 存在明显子任务依赖的任务

例如：

- 先检索，再整理，再写代码，再测试
- 先收集材料，再生成结构，再填充内容
- 先分析错误，再决定修复路线，再验证结果

这种任务更适合把依赖关系外显化。

### 5.3 高代价失败任务

如果一次错误代价较高，系统常常希望：

- 在行动前先看清整体路径
- 控制步骤顺序
- 减少盲目探索

因此高风险场景通常更容易受益于显式 planning。

### 5.4 需要协作、审查或人工接管的任务

如果系统不是纯黑盒自动执行，而是要和人、审查流程或外部系统交互，显式 plan 会更容易成为共享接口。

---

## 六、哪些场景里它可能不是必要条件

### 6.1 短任务或低分支任务

如果任务很短，显式 planning 可能只是把本来简单的 loop 额外形式化。

### 6.2 高动态环境任务

当外部环境变化过快时，先写出的全局计划很容易迅速过时。

这种情况下：

- 计划维护成本会上升
- 局部反馈驱动调整可能更有效

### 6.3 探索性任务

有些任务本来就不是先有稳定路径再执行，而是通过不断观察逐步发现路径。

这类任务常常更适合紧耦合 reasoning-action loop，而不是重型 upfront planning。

### 6.4 工具与反馈足够强的任务

如果系统每一步都能快速获得外部反馈，例如：

- 编译/测试
- 检索结果
- 工具返回码
- 页面状态

那么局部 loop 本身就可能承担大部分导航功能，显式 plan 不一定是必要前提。

---

## 七、最重要的结构性矛盾

### 7.1 Global Structure vs Local Adaptivity

显式 planning 提供全局结构；紧耦合 loop 提供局部适应性。二者往往此消彼长。

### 7.2 Plan Visibility vs Plan Maintenance Cost

计划一旦外显，可解释性会提升；但环境越动态，维护这份计划的成本越高。

### 7.3 Upfront Decomposition vs Incremental Discovery

有些任务适合先拆后做；有些任务必须边做边发现正确路径。

### 7.4 Planner Clarity vs System Overhead

独立 planner 让系统分工更清晰；但也会增加状态同步、计划失效和 replanning 复杂度。

---

## 八、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- 没有显式 planner 就不算 agent
- `ReAct` 没有 planning
- 任务越复杂，越一定要先生成完整计划
- 有了显式 plan，就天然比局部 loop 更可靠
- 显式 planning 与 tool orchestration 是同一个问题

这些说法都会把 planning 的实现形式与 planning 的存在本身混为一谈。

---

## 九、与其他专题的关系

- 与 `overview.md`：本文件展开其中“不是所有高性能单智能体都需要显式 planner”这一提醒，并把它从一句 caution 变成完整专题。
- 与 `planning/README.md`：后者定义 planning 目录边界；本文件进一步澄清 planning 重要，不等于显式 planner 必然是硬门槛。
- 与 `patterns/react/README.md`：`ReAct` 是 planning 与 acting 紧耦合的重要代表，说明“无独立 planner”不等于“无规划能力”。
- 与 `planning/plan-and-execute/README.md`：后者代表 planning 显式分离的一端；本文件解释为什么它是路径之一，而不是唯一标准答案。
- 与 `planning-vs-execution.md`：后者讨论 planning 与 execution 的耦合/解耦；本文件更基础地讨论 planning 是否必须以独立 planner 形式存在。
- 与 `agent-vs-tool-workflow-boundary.md`：系统是否是 agent，关键在持续闭环而非是否有 planner；本文件在此基础上继续细化 planner 的必要性边界。
- 与 `tool-orchestration.md`：复杂工具链常推动显式 planning 出现，但 orchestration 与 planner 不是同一层问题。
- 与 `backlog.md`：本文件对应当前 P0 缺口 `Planning 是否必须显式存在`，并为后续继续补充 bilevel planning、tool navigation 等线索提供判断基线。

---

## 十、当前最值得继续补证的方向

- `ReAct`、plan-and-execute、bilevel planning 等代表路径在任务复杂度上的稳定分界是否存在
- 哪些任务一旦出现长链依赖，就会显著受益于显式 plan representation
- 显式 planner 的收益，哪些来自更好分解，哪些来自更强 traceability 与人工可见性
- 更强长上下文模型是否会降低独立 planner 的必要性，还是只是把 planner 内联化
- 是否能形成一套更稳的判断准则，用于区分“planning 存在”与“explicit planner 必需”
