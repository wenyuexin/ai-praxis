# Failure Attribution

> 适用范围：Agent 系统中失败如何被定位、归因、解释与反馈到后续修订
> 阶段状态：研究主干，持续补充 benchmark、产品案例与调试证据
> 使用说明：本文件回答“为什么 Agent 失败不能只看最终报错、failure attribution 到底在区分什么、它与 traceability / replay / debugging 的关系是什么”，不直接把某一种日志系统或追踪实现写成标准答案

## 一、定位

在 Agent 系统里，`failure attribution` 不应被狭义理解为“找到报错日志”或“确认哪一步失败了”。

更准确地说，它描述的是：**当任务失败、偏航、停滞或产出低质量结果时，系统是否能把失败合理归因到某一类来源、某一个阶段或某一组相互作用机制上**。

这里最关键的不是只有一个失败标签，而是三个问题：

- 失败是来自模型推理、工具执行、环境状态、权限策略、任务分解，还是人机协作关系
- 失败发生在表面可见步骤，还是更早的隐含前置环节
- 失败被解释清楚之后，系统能否把结论回流到 prompt、tool、memory、environment 或 orchestration 的修订

因此，failure attribution 是 evaluation 中连接 observability、debugging 与 iterative improvement 的基础主题，而不是错误日志收集的附属细节。

---

## 二、为什么它必须单列成主题

### 2.1 “失败了”远远不够支持系统改进

很多 Agent 任务的失败并不是单一报错，而是：

- 结果错了，但过程看似正常
- 任务没有完成，但系统也没有硬错误
- 中间步骤都成功了，最终交付物却不可用
- 人类接管后才发现更早阶段的偏航

这意味着只记录成功/失败或错误码，通常不足以支持改进。

### 2.2 失败归因会直接决定调试方向

同样是“任务没完成”，可能对应完全不同的调试路径：

- 如果是推理误判，可能要改 prompt 或 plan structure
- 如果是工具返回不稳定，可能要改 tool wrapper 或 retry policy
- 如果是环境漂移，可能要改 sandbox、workspace 或依赖管理
- 如果是任务委托粒度不对，可能要改 control surface 或 human-in-the-loop 设计

因此 failure attribution 不是事后总结，而是调试与演进的分流机制。

### 2.3 它与 traceability、replay、checkpoint 都高度耦合

如果没有足够的轨迹、状态锚点与中间 artifact：

- 失败很难稳定复盘
- replay 往往只能重跑，不能解释
- checkpoint 只能帮助回退，未必帮助理解原因

所以 failure attribution 必须在 `07-evaluation/` 中单列，而不能只散落在 observability 或 logging 主题里。

---

## 三、常见 failure attribution 维度

### 3.1 Model / Reasoning Failure

失败主要来自模型对任务、上下文或中间信息的误解与错误推理。

典型表现：

- 误解需求
- 选错策略
- 忽略关键约束
- 在多步推理中早期走偏

这类失败最容易被误判成“结果不好”，但真正的问题常常更早就出现了。

### 3.2 Tool / Action Failure

失败主要来自工具调用、命令执行或动作层结果异常。

典型表现：

- API 返回异常或结构不稳定
- 命令执行失败
- 写文件失败、路径错误、超时
- tool wrapper 行为与系统预期不一致

这类失败通常比 reasoning failure 更容易被显式记录，但未必更容易被正确解释。

### 3.3 Environment / State Failure

失败主要来自环境状态、依赖上下文或 workspace 条件异常。

典型表现：

- 环境漂移
- 依赖缺失或版本不一致
- workspace 污染
- 权限、网络、文件系统约束与任务不匹配

这类失败常常会伪装成 tool failure 或结果不稳定，因此需要更强的状态视角来识别。

### 3.4 Orchestration / Workflow Failure

失败主要来自任务拆分、步骤组织、上下文传递或多阶段协调本身。

典型表现：

- 计划不合理
- 上下文在阶段切换中丢失
- 子步骤顺序错误
- 多 agent / 多模块之间责任边界不清

这类失败不一定出现在任何单一步骤中，而是出现在整体执行结构上。

### 3.5 Human / Control Failure

失败主要来自委托方式、确认机制、接管时机或用户控制面的不合理设计。

典型表现：

- 用户给了过粗或过细的委托
- 应该确认的动作没有确认
- 人工介入过晚
- 系统状态展示不足，导致人类做错判断

这提醒我们：failure attribution 不只是模型和工具问题，也包含 human-agent interaction 层面的失败。

---

## 四、如何进行 failure attribution

### 4.1 Step Trace：先定位失败暴露点

第一步通常是确认失败首先在哪个步骤暴露出来：

- 是计划阶段就偏了
- 还是执行阶段报错
- 还是结果验收阶段才暴露问题

这回答的是“失败在哪里被看到”，但还不是“失败为什么发生”。

### 4.2 Causal Chain：再追踪导致失败的因果链

真正的 attribution 需要往前追：

- 这个失败是单点失误，还是早期偏航的结果
- 中间有没有关键上下文被误读或丢失
- 哪个 checkpoint 之后系统开始不可恢复地偏离

这一步决定系统能否从“报错定位”提升到“因果解释”。

### 4.3 State and Artifact View：检查状态与产物是否支持解释

很多失败并不能只靠日志解释，还要看：

- 当时的 workspace state 是什么
- 关键 artifact 是否已经过时或失配
- 哪个结果与哪个 code state / task state 对应

这说明 failure attribution 与 traceability object model 紧密相关。

### 4.4 Repair Mapping：把归因映射到修订动作

failure attribution 的价值在于它能指导下一步修订：

- 改 prompt
- 改 tool
- 改 retry / timeout policy
- 改 environment isolation
- 改 delegation granularity 或 approval design

如果不能映射到修订动作，再细的 attribution 也容易停留在分析展示层。

---

## 五、它与 observability / debugging / replay 的关系

### 5.1 observability 提供证据，不等于 attribution

有日志、有轨迹、有 checkpoint，并不自动意味着系统已经理解失败来源。

observability 更像证据基础设施；attribution 是在这些证据之上建立解释结构。

### 5.2 debugging 依赖 attribution 做优先级分流

debugging 往往涉及很多可能的修复方向，而 attribution 的价值就在于缩小范围、提高修复命中率。

### 5.3 replay 能帮助复现，但不一定能帮助解释

replay 可以帮助重演失败路径，但如果没有足够的状态标签、artifact 关系和因果链，重放仍可能只得到“又失败了一次”。

所以 replay 是归因的重要支撑，而不是归因本身。

---

## 六、最重要的结构性矛盾

### 6.1 Attribution Fidelity vs Instrumentation Cost

归因越细，越需要更强的埋点、对象建模与状态记录；但这也会带来更高的系统成本。

### 6.2 Local Error Detection vs Systemic Cause Analysis

局部错误检测更快、更易自动化；系统性因果分析更有解释力，但也更复杂。

### 6.3 Replayability vs Interpretability

能重放失败路径，不一定能清楚解释失败机制；能解释失败机制，也未必总能稳定重放。

### 6.4 Developer Utility vs User Understandability

开发者需要更细的 attribution；普通用户可能只需要知道“哪里不可靠、下一步该怎么做”。

这意味着 failure attribution 的输出层也需要分层。

---

## 七、最容易被误写成定论的问题

当前阶段尤其要避免以下误解：

- 有错误日志就等于已经完成失败归因
- 最后一步报错的位置就是失败根因
- 只要能 replay，就足以支持 debugging
- failure attribution 主要是工程日志问题，不涉及任务结构和人机协作
- 一次失败只能归因给一个来源

这些说法都会让调试体系停留在表面错误，而忽视真正的因果结构。

---

## 八、与其他专题的关系

- 与 `../overview.md`：本文件把 evaluation 中的 observability / debugging 视角下沉为 failure attribution 框架。
- 与 `../backlog.md`：对应 `Failure Attribution and Replayable Debugging` 这一 P0 缺口。
- 与 `../task-completion-metrics/success-definition.md`：只有先定义什么算失败、什么算部分完成，failure attribution 才有稳定对象。
- 与 `../../05-environments/code-execution-environments/workspace-traceability.md`：failure attribution 依赖 workspace 中动作、状态与 artifact 的可追溯性。
- 与 `../../05-environments/code-execution-environments/traceability-object-model.md`：归因是否成立，高度依赖 task、step、action、artifact、checkpoint 等对象是否被显式建模。
- 与 `../../04-human-agent-interaction/delegation-and-control/delegation-granularity.md`：某些失败并非系统能力不足，而是委托粒度和控制边界设计不当。

---

## 九、当前最值得继续补证的方向

- 不同 benchmark 与真实产品中 failure attribution 的最小对象集与最小证据集是什么
- 多 agent / 多阶段工作流中的 failure attribution 是否需要单独的层级化框架
- replay、checkpoint、artifact lineage 与 failure attribution 的最佳组合方式
- attribution 输出是否应区分开发者视图、评估视图与用户视图
- failure attribution 是否应进一步拆成 root-cause taxonomy、repair mapping、handoff readiness 三个独立专题
