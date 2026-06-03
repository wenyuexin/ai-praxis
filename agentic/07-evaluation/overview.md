# Evaluation Overview

> 适用范围：Agent 系统评测、调试、可观测性与可靠性验证的整体认知框架
> 阶段状态：研究综述，持续补充
> 使用说明：本文件回答“Agent 应该如何被评价、benchmark 与 observability 分别解决什么问题、当前最重要的结构性矛盾是什么”，不直接等同于某一个具体 benchmark 的介绍

## 一、定位

`07-evaluation/` 研究的不是“跑几个 benchmark 分数”这么狭义的问题，而是：**当 Agent 能够规划、调用工具、跨步骤执行、与人协作甚至多 Agent 联动时，我们究竟用什么标准判断它是否有效、可靠、可复现、可解释**。

这里的关键不只是分数，而是评价框架本身：

- 评测的是任务结果、执行过程，还是失败恢复能力
- benchmark 的环境、工具和数据分布是否足够代表真实任务
- 可观测性与调试能力如何帮助解释“为什么失败”
- 人工评估、自动指标与过程轨迹之间如何互相补足

因此，本目录更适合被理解为 Agent 系统的 **quality judgment / debugging / validation layer**，而不是附属于某几个榜单的资料集合。

---

## 二、建议采用的五个核心视角

### 2.1 Task Completion and Outcome Quality

这一视角回答：**Agent 最终有没有把任务做成，以及结果质量如何**。

典型问题包括：

- 任务完成的判定标准是什么
- 正确率、通过率、成功率、完成时间、成本是否需要联合看待
- 开放任务中如何定义“足够好”
- 输出质量是否应与过程稳定性分开评价

它强调的是 **结果层面的有效性判断**。

### 2.2 Benchmark Design and Representativeness

这一视角回答：**一个 benchmark 到底在测什么，以及它是否代表真实世界问题**。

典型问题包括：

- benchmark 测的是知识问答、工具使用、长程规划还是软件工程任务
- 是否依赖特定环境、特定工具链或特定数据分布
- benchmark 的成功标准是否与真实使用场景对齐
- leaderboard 分数能否直接外推到真实任务能力

它强调的是 **评测任务与真实任务之间的映射关系**。

### 2.3 Observability and Debugging

这一视角回答：**当 Agent 出错时，我们是否知道它是怎么错的**。

典型问题包括：

- 是否有足够的轨迹、日志、checkpoint、artifact 与步骤状态可供复盘
- 错误是来自推理、工具、环境、权限、数据，还是协作失配
- 调试能力是否只是事后查看日志，还是能支持系统性归因和重试
- 可观测性如何反馈到 prompt、tool、memory 或 orchestration 的修订

它强调的是 **解释失败与支持迭代改进的能力**。

### 2.4 Human Evaluation and Trustworthy Use

这一视角回答：**用户、评审者或操作者是否真的认为系统可用、可信且可协作**。

典型问题包括：

- 人工评估关注的是正确性、可理解性、协作体验还是风险感知
- 主观偏好与自动指标之间是否一致
- 系统是否在高风险场景中表现出可接受的行为边界
- 用户是否会因为错误的能力印象而过度信任系统

它强调的是 **人类使用语境下的质量判断**，而不只是纯自动分数。

### 2.5 Safety, Robustness and Failure Boundary

这一视角回答：**系统在扰动、攻击、异常工具环境与长链任务下是否仍能维持可接受行为**。

典型问题包括：

- 对 prompt injection、工具错误、环境漂移和任务分布偏移是否鲁棒
- 失败是局部可恢复，还是会沿着任务链放大
- 安全性评估与普通任务成功率之间是否存在显著张力
- 鲁棒性是否只是“平均分不低”，还是“坏情况下仍可控”

它强调的是 **系统边界条件下的可靠性与风险控制能力**。

---

## 三、当前最重要的结构性矛盾

### 3.1 Benchmark Score vs Real-World Utility

benchmark 分数更高，不一定意味着真实任务里更好用。

原因包括：

- benchmark 常依赖封闭任务定义
- 真实任务更受环境、权限、工具、数据质量和用户协作影响
- 某些系统擅长刷分，但并不擅长开放世界执行

### 3.2 Outcome Metrics vs Process Understanding

只看最终结果，可能看不出系统是稳健完成还是侥幸完成；只看过程轨迹，又可能丢失实际交付价值。

因此“结果评价”和“过程解释”不能互相替代。

### 3.3 Automatic Evaluation vs Human Judgment

自动指标更便于规模化比较；人工评估更接近真实使用感受。

但两者经常不一致：

- 自动指标可能忽略可解释性与协作体验
- 人工评估可能引入主观偏差与成本瓶颈

### 3.4 Reproducibility vs Ecological Validity

越可复现的 benchmark，往往越倾向受控环境；越贴近真实世界的任务，往往越难稳定复现。

这是一条长期存在的 trade-off。

---

## 四、最容易被误写成定论的问题

当前阶段尤其要避免以下误解：

- Agent 评估主要就是 leaderboard 排名
- 任务成功率足以概括系统质量
- 有日志就等于可调试、可观测
- 人工评估只是锦上添花，不影响主结论
- 评估问题只属于 benchmark 设计，不涉及环境、交互与系统架构

这些说法都会把评估目录从系统质量判断层，误写成指标列表或榜单汇总。

---

## 五、核心子主题如何落位

### 5.1 Task Completion Metrics

聚焦任务成功定义、结果质量指标、成本和效率衡量。

### 5.2 Agent Benchmarks

聚焦通用 Agent 任务、多工具任务与长程任务 benchmark。

### 5.3 SWE Benchmarks

聚焦软件工程任务中的定位、修复、测试与仓库级评测问题。

### 5.4 Safety and Robustness

聚焦攻击、异常条件、对抗扰动与失败边界下的可靠性评价。

### 5.5 Human Evaluation

聚焦主观评估、协作体验、可信度判断与人工审查机制。

### 5.6 Observability and Debugging

聚焦轨迹、日志、回放、归因与调试支持。

### 5.7 Papers

聚焦评估相关论文、综述与方法演进材料。

---

## 六、与其他目录的关系

- 与 `../02-single-agent/`：本目录评估单智能体能力在真实任务中的有效性与稳定性。
- 与 `../03-multi-agent/`：多 Agent 系统的协调成本、收益边界与失败模式需要单独评估。
- 与 `../04-human-agent-interaction/`：人工评估、信任校准与交互体验验证与人机协作目录紧密相关。
- 与 `../05-environments/`：环境决定 benchmark 是否可信、执行是否可复现、日志是否可解释。
- 与 `../06-frameworks-and-tools/`：具体框架、产品与项目案例为评估方法提供真实映射对象。

---

## 七、当前最值得补齐的专题

当前目录骨架已经建立，但主干正文仍较少。优先值得补齐的方向包括：

- `task-completion-metrics/` 下的 success definition、cost-quality tradeoff 与 partial completion
- `agent-benchmarks/` 下的 benchmark taxonomy、task realism 与 transferability
- `observability-and-debugging/` 下的 traceability、failure attribution 与 replay-based debugging
- `human-evaluation/` 下的 rubric design、trust calibration 与 collaborative usefulness
- `safety-and-robustness/` 下的 adversarial evaluation、distribution shift 与 failure boundary

这些更适合先拆成专题正文或 backlog 项，而不是继续停留在目录占位阶段。
