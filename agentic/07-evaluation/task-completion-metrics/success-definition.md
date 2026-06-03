# Success Definition and Partial Completion

> 适用范围：Agent 任务中“成功”如何定义、部分完成如何判断、质量阈值如何划分
> 阶段状态：研究主干，持续补充 benchmark 与真实任务案例证据
> 使用说明：本文件回答“为什么 Agent 任务不能只看 success/fail 二元结果、partial completion 到底意味着什么、当前最重要的 trade-off 是什么”，不直接把某个 benchmark 的通过条件写成普适标准

## 一、定位

在 Agent 系统里，`success` 不应被狭义理解为“最终输出了一个看起来像答案的结果”或“benchmark 判定通过”。

更准确地说，success 应被理解为：**系统在给定任务语境、质量要求、成本约束与风险边界下，是否达到了可接受的完成状态**。

这里最关键的不是只有一个二元标签，而是三个问题：

- 成功的判定是结果导向、过程导向，还是二者结合
- 什么叫“部分完成”，它是有价值中间态，还是失败的另一种说法
- 质量、成本、时间、风险是否共同构成 success definition

因此，success definition 是 Agent evaluation 中的基础主题之一，而不是 benchmark 打分逻辑的附属细节。

---

## 二、为什么它必须单列成主题

### 2.1 很多 Agent 任务天然不是二元结果

传统分类任务更容易用正确/错误来判断，但 Agent 任务常常包含：

- 多步骤子目标
- 开放式结果
- 可恢复中间态
- 需要人与系统共同完成的交付物

这意味着只用 success/fail 往往会掩盖大量真实差异。

### 2.2 “部分完成”既可能是价值信号，也可能是误导

部分完成有时意味着：

- 已完成前半段工作，后续只需少量人工补齐
- 已产出可复用 artifact
- 已经把错误范围缩小，利于恢复

但也可能意味着：

- 结果不可交付
- 系统在关键步骤上失败，只留下表面进展
- 用户被“看起来做了一些事”误导

所以 partial completion 不能被自动当成正向指标。

### 2.3 它会直接影响 benchmark、human evaluation 与 debugging

如果 success definition 不清楚：

- benchmark 的通过率会失去解释力
- 人工评估会缺乏稳定标准
- 系统调试时很难判断问题是“没做完”还是“做错了但表面完成了”

因此它不是某个子目录的小问题，而是 `07-evaluation/` 的基础判断框架之一。

---

## 三、常见 success 定义路径

### 3.1 Binary Completion

只判断任务是否最终完成。

典型场景：

- benchmark 中的 pass/fail 判定
- 单轮工具调用是否成功
- 明确可验证的目标任务

优点：

- 简单
- 易比较
- 易统计

代价：

- 难表达部分进展
- 难表达质量差异
- 容易掩盖恢复价值与中间产物价值

### 3.2 Graded Outcome Quality

把结果质量分成多个等级，而不是简单通过/失败。

典型场景：

- 文档、分析、规划、研究型任务
- 代码修复中的“能运行 / 能通过测试 / 工程质量可接受”分层
- 开放任务中的可用性评估

优点：

- 更接近真实任务判断
- 能区分低质量完成与高质量完成
- 更适合 human evaluation 结合

代价：

- 标准更难统一
- 主观性更强
- 自动化评测更困难

### 3.3 Milestone-Based Completion

按任务中的关键里程碑判断完成度。

典型场景：

- 长链任务
- 多阶段工作流
- 包含检索、分析、执行、验证的任务

优点：

- 能表达部分完成的结构位置
- 更适合分析卡住在哪里
- 有利于恢复和接管

代价：

- 里程碑定义本身有争议
- 容易把任务切得过细或过粗
- 未必总能映射到最终交付价值

### 3.4 Utility-Based Completion

用“结果是否足够有用”来判断任务是否成功，而不是只看形式完成。

典型场景：

- 用户真实工作流中的任务支持
- 研究辅助、编程辅助、分析建议等场景
- 人与 Agent 协作完成的开放任务

优点：

- 更贴近真实使用价值
- 能容纳 partial completion 的有效情形
- 更利于 human-centered evaluation

代价：

- 难以标准化
- 很依赖上下文和用户角色
- 容易与偏好评价混在一起

---

## 四、partial completion 到底意味着什么

### 4.1 结构上完成了多少

一种最直接的理解是：任务的关键步骤完成到了什么位置。

例如：

- 已完成检索和分析，但未执行修改
- 已完成代码修改，但未通过测试
- 已产出报告草稿，但未完成核验

这属于 **里程碑层面的 partial completion**。

### 4.2 结果上有多少可交付价值

有些任务虽然没有完全完成，但已经产生了可复用价值，例如：

- 给出了正确方向的修复 patch
- 提供了高质量的排障线索
- 产出了可继续加工的结构化结果

这属于 **utility 层面的 partial completion**。

### 4.3 恢复上离成功还有多远

partial completion 还可以被理解为恢复成本问题：

- 继续做需要从头开始，还是只补最后一步
- 人工接管时需要完全重做，还是只需微调
- 系统重试时能否复用中间状态和 artifact

这说明 partial completion 与 recovery、handoff 和 debugging 紧密相关。

---

## 五、如何比较不同 success 定义

### 5.1 任务类型

不同任务天然适合不同 success definition：

- 封闭可验证任务更适合 binary completion
- 长链任务更适合 milestone-based completion
- 开放结果任务更适合 graded 或 utility-based completion

### 5.2 风险与成本约束

有些任务只要最终成功即可；有些任务则必须同时满足：

- 成本不过高
- 时间不过长
- 风险不越界
- 过程可审计

因此 success 不是只看结果本身，还要看任务约束下是否“可接受”。

### 5.3 谁在做判断

success 也受到评判者影响：

- benchmark evaluator 更偏形式化规则
- human reviewer 更偏可用性与可信度
- system developer 更关心可复现性与 failure attribution

如果不区分评判主体，就很容易混淆不同 success 标准。

---

## 六、最重要的结构性矛盾

### 6.1 Simplicity vs Fidelity

success definition 越简单，越容易比较；但越简单，也越容易失真。

### 6.2 Binary Clarity vs Partial Value Recognition

二元标准更清晰，但会忽略中间产物和可恢复价值；承认 partial completion 更贴近现实，但也更复杂。

### 6.3 Objective Metrics vs Contextual Usefulness

越客观的指标越易统一；越接近真实使用价值的标准，越依赖上下文和用户目标。

### 6.4 Benchmark Comparability vs Real-World Validity

统一 success 标准便于 benchmark 排名；但真实任务往往需要更丰富的完成定义。

---

## 七、最容易被误写成定论的问题

当前阶段尤其要避免以下误解：

- success/fail 足以覆盖绝大多数 Agent 任务
- partial completion 一定应该记作正向结果
- benchmark pass rate 就代表真实可用性
- 有输出就说明任务已经完成
- success definition 只是评分细节，不影响系统设计与调试

这些说法都会让评估体系停留在表面结果，而忽视任务结构、恢复语义与真实交付价值。

---

## 八、与其他专题的关系

- 与 `../overview.md`：本文件把 evaluation 目录中的结果质量视角下沉为 success definition 与 partial completion 框架。
- 与 `../backlog.md`：对应 `Success Definition and Partial Completion` 这一 P0 缺口。
- 与 `../agent-benchmarks/`、`../swe-benchmarks/`：benchmark 的通过条件必须建立在更清楚的 success definition 之上。
- 与 `../human-evaluation/`：graded outcome 与 utility-based completion 往往需要人工评估补充。
- 与 `../observability-and-debugging/`：partial completion 是否有价值，常常取决于中间状态是否可追溯、可复用、可恢复。
- 与 `../../04-human-agent-interaction/`：人类接管、人工验收和协作体验会影响“部分完成是否足够好”的判断。

---

## 九、当前最值得继续补证的方向

- 不同 benchmark 中 pass/fail 与 graded success 的真实差异
- software engineering、research、tool-use 等任务中的 partial completion 典型形态
- utility-based completion 是否可以形成更稳定的评估 rubric
- milestone-based completion 如何和 replay、handoff、recovery 结合
- success definition 是否应进一步拆成 result quality、completion structure、handoff readiness 三个独立专题
