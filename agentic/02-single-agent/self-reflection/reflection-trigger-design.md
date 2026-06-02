# Reflection Trigger Design

> 适用范围：单智能体中 self-reflection 的触发信号、停止条件、预算控制与恢复分流设计
> 阶段状态：研究主干，持续补充任务类型与系统案例证据
> 使用说明：本文件回答“为什么 reflection 不应默认常开、触发机制通常依赖哪些信号、系统如何决定何时反思、何时停止、何时改走 retry / replanning / human handoff”，不直接把某一种 trigger policy 写成标准答案

## 一、定位

如果说 `reflection-cost-tradeoff.md` 讨论的是：**reflection 值不值得、成本来自哪里、哪些任务更可能从中获益**，那么本文件进一步讨论的是另一个更工程化的问题：**既然 reflection 不是默认净收益能力，系统到底应该在什么时点触发它**。

这意味着我们不再只问：

- reflection 会不会提高质量

而要继续问：

- 什么信号说明“现在值得反思”
- 什么信号说明“继续反思已经没有意义”
- 什么时候应该改走 retry、tool substitution、replanning 或 human handoff
- 触发策略如何与风险、预算、任务结构和外部反馈耦合

因此，本主题讨论的不是 reflection 能不能做，而是：**reflection 在执行闭环里如何被触发、约束和终止**。

---

## 二、为什么它必须单列成主题

### 2.1 收益边界明确之后，下一个问题必然是触发边界

一旦已经承认 reflection 带成本，那么系统就不能把它当作默认常开能力。

真正的问题会变成：

- 简单任务要不要反思
- 失败后先反思还是先重试
- 高风险步骤是不是应该预先反思
- 反思几轮后该停止

也就是说，收益/成本分析只是上游，trigger design 才是落地控制层。

### 2.2 它和 stopping rule 一起决定 loop 是否稳定

反思系统如果没有触发逻辑，很容易：

- 该反思时不反思
- 不该反思时过度计算
- 在局部失败上不断自检却不收敛

因此 trigger 与 stopping logic 不是附属细节，而是 reflection loop 是否稳定的关键。

### 2.3 它是 self-reflection 与 recovery design 的连接点

reflection 不只是 self-reflection 目录内部问题。

它还会直接碰到：

- tool failure 后是否先反思
- 任务偏航后是否回到 planning
- 错误已经结构性失效时是否应该直接 replanning
- 高风险场景下是否需要人类接管

所以 trigger design 实际上是 self-reflection 与 tool-use / planning / environments 的桥梁问题。

---

## 三、为什么 reflection 不能默认常开

### 3.1 固定每步都反思会快速放大成本

如果每一步都插入 reflection：

- token 开销增加
- latency 变长
- 执行链更复杂
- 状态管理负担更重

对很多短任务来说，这几乎一定是过度计算。

### 3.2 不是所有失败都适合先做 reflection

有些问题其实更适合：

- 直接重试
- 更换工具
- 重新读取上下文
- 回到 planning 层重构路径

如果系统把所有失败都先送进 reflection，就会把 recovery path 弄得过于单一。

### 3.3 有些任务的最佳反馈根本不来自自我批评

例如：

- 测试结果
- 编译器错误
- 返回码
- 结构化校验结果
- 用户显式反馈

这些信号往往比语言层 self-critique 更硬、更直接。

因此 trigger design 的核心之一，就是分辨**什么时候该优先相信外部反馈，而不是继续内部反思**。

---

## 四、常见触发信号到底有哪些

### 4.1 Tool Failure Trigger

最直接的一类是工具失败触发，例如：

- 调用报错
- 返回码异常
- 输出结构不合法
- 关键字段缺失

这种信号的优点是明确，但问题在于：工具失败后未必总该先反思，也可能更适合 retry 或 tool substitution。

### 4.2 Constraint Violation Trigger

当输出不满足明确约束时触发 reflection，例如：

- 格式不合规
- 漏掉必需字段
- 未满足用户指定条件
- 生成结果与 task spec 冲突

这类触发通常比较稳定，因为它有显式 success criteria。

### 4.3 Inconsistency Trigger

系统检测到内部或外部不一致时触发，例如：

- 前后结论冲突
- 计划与当前动作不一致
- 工具结果与已有状态冲突
- 多个证据源相互矛盾

这是很常见但也较复杂的一类 trigger，因为它要求系统先具备 inconsistency detection 能力。

### 4.4 High-Risk Step Trigger

在高风险步骤前后主动触发 reflection，例如：

- 执行不可逆修改前
- 提交关键结果前
- 权限敏感操作前
- 人工接管前的最后检查

这类触发更接近治理策略，而不是单纯纠错策略。

### 4.5 Task Complexity Trigger

系统依据任务复杂度、步骤跨度或依赖规模决定是否触发 reflection。

这种方式的优点是前置控制，但难点在于：复杂度判断本身未必稳定。

### 4.6 Confidence / Uncertainty Trigger

如果系统能够估计不确定性，也可能在：

- 自信度过低
- 多候选路径分歧较大
- 当前观察不足以支撑下一步

时触发 reflection。

这类方法很吸引人，但往往也更依赖额外校准机制。

---

## 五、触发之后，系统还要决定什么

### 5.1 是 local reflection 还是 global reconsideration

触发 reflection 之后，首先要分辨：

- 只修当前输出
- 还是重新审视更高层计划或状态

如果问题根源已经在任务结构层，继续局部 reflection 只会浪费成本。

### 5.2 是 self-critique 还是 separate critic

不是所有 reflection 都必须由同一个 agent 角色执行。

系统还要决定：

- 由当前 executor 自反
- 还是切换到单独 critic 阶段 / 角色

这会直接影响成本、解释性与协调复杂度。

### 5.3 反思后走 revise、retry、replan 还是 handoff

reflection 真正有价值的部分，不只是“看出哪里错了”，而是能把后续分流决定清楚：

- revise 当前结果
- retry 当前动作
- 换工具
- 回到 planning 层
- 终止并交给人

所以 trigger design 必须和 recovery routing 一起看。

---

## 六、停止条件为什么和触发条件同样重要

### 6.1 没有 stopping rule 的 reflection 很容易失控

如果系统只会触发，不会停止，就容易出现：

- 一轮接一轮自检
- 反思发现问题，却继续反思而不行动
- 成本不断累加但收益边际递减

### 6.2 常见 stopping 方式

常见停止条件包括：

- 达到最大轮数
- 达到预算上限
- 当前结果已满足约束
- 连续两轮没有产生实质修正
- 已判断问题需要升级到 replanning 或 handoff

### 6.3 stopping rule 本质上也是治理机制

stopping 不只是节省 token，而是在回答：

- 什么时候应该结束内部自我修正
- 什么时候应该把控制权交给其他 recovery path

这让它和 `agent-vs-tool-workflow-boundary.md` 中的 stopping logic 形成直接呼应。

---

## 七、几种典型 trigger policy 风格

### 7.1 Fixed Triggering

在固定步骤或固定阶段插入 reflection。

优点：

- 简单
- 可预测
- 便于预算控制

局限：

- 对任务差异不敏感
- 容易在简单任务上浪费成本

### 7.2 Event-Driven Triggering

由失败、约束违背、不一致等具体事件触发。

优点：

- 更贴近真实执行信号
- 更容易把 reflection 与 recovery 结合

局限：

- 依赖事件检测质量
- 可能漏掉“尚未显性失败但已在偏航”的情况

### 7.3 Risk-Aware Triggering

对高风险步骤额外提高 reflection 频率或严格度。

优点：

- 更贴近工程治理需求
- 与安全、审计、handoff 更容易结合

局限：

- 风险分级本身需要稳定标准

### 7.4 Budget-Aware Triggering

把 reflection 明确视作预算化能力：

- 只对高价值任务开放
- 超预算后降级
- 在一定轮数或 token 上限后切换路径

这类策略尤其适合实际系统，而不只是 benchmark setting。

---

## 八、哪些场景里 trigger design 更关键

### 8.1 Coding Agent

代码任务里既有丰富外部反馈，又有高代价错误，因此系统尤其需要决定：

- 测试失败后先反思还是先重试
- patch 生成后是否需要先做自检
- 哪些修改前后需要高风险触发

### 8.2 Browser / Research Agent

在信息不完整、页面动态变化、证据来源多样的场景里，reflection trigger 很容易和 inconsistency detection 耦合。

### 8.3 高风险执行任务

只要任务带有显著副作用或高可信要求，reflection trigger 就不再只是性能优化，而是治理机制的一部分。

---

## 九、最重要的结构性矛盾

### 9.1 More Checking vs More Throughput

触发越积极，潜在质量越高；但吞吐、延迟与交互流畅度也会同步下降。

### 9.2 Local Reflection vs Structural Recovery

有些问题适合局部反思修补；有些问题应直接进入 replanning、tool substitution 或 handoff。

### 9.3 Early Trigger vs Late Trigger

过早触发可能浪费成本；过晚触发则可能让错误已扩散到更难修复的状态。

### 9.4 Generic Policy vs Task-Specific Policy

统一 trigger policy 更简单；但不同任务、不同风险等级、不同工具链往往需要不同 trigger logic。

---

## 十、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- 失败后总该先做 reflection
- 只要检测到错误就应该继续多轮反思
- 高风险任务一定要默认增加更多 reflection 轮数
- 有了 trigger policy 就不再需要 stopping rule
- reflection trigger 主要只是 self-reflection 目录内部细节

这些说法都会低估 reflection 与 retry、replanning、tool recovery、human handoff 之间的分流关系。

---

## 十一、与其他专题的关系

- 与 `self-reflection/README.md`：后者定义 self-reflection 目录边界；本文件进一步展开“何时触发、何时停止、何时分流”的控制层问题。
- 与 `reflection-cost-tradeoff.md`：后者解释 reflection 为什么不是默认净收益能力；本文件则解释一旦承认这一点，系统应如何设计 trigger 与 stopping logic。
- 与 `tool-invocation-reliability.md`：工具失败、前置条件不满足、结果校验异常等信号，常常会成为 reflection 的重要触发源，但不必然总是最优分流路径。
- 与 `planning-vs-execution.md`：当问题根源位于更高层任务结构时，reflection trigger 应让位于 replanning，而不是停留在局部修补。
- 与 `agent-vs-tool-workflow-boundary.md`：reflection trigger 与 stopping rule 都是持续执行闭环稳定性的一部分，不只是局部优化手段。
- 与 `../05-environments/`：在高风险环境里，reflection trigger 也可能承担治理与安全检查作用，但不能替代权限、回滚与隔离机制。
- 与 `backlog.md`：本文件对应当前 P0 缺口 `Reflection Trigger Design`，并为后续继续展开 critic routing、budget policy 与 risk-aware control 提供基础主轴。

---

## 十二、当前最值得继续补证的方向

- coding / browser / research 三类 agent 在 reflection trigger 上是否存在稳定差异
- 工具失败、约束违背、不一致、高风险步骤四类触发信号，哪些最常带来净收益
- reflection 触发后，revise / retry / replan / handoff 的最优分流策略是否存在稳定模式
- stopping rule 的收益更多来自成本控制，还是来自避免 loop drift
- 是否能形成一套更稳的 reflection trigger checklist，用于比较不同 agent 系统
