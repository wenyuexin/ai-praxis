# Tool-Centric vs Monolithic Agent

> 适用范围：单智能体能力组织方式中的工具中心化路径与统一大模型路径比较
> 阶段状态：研究主干，持续补充系统案例与工程证据
> 使用说明：本文件回答“单智能体为什么会在 tool-centric 与 monolithic 之间形成主轴分化、两种路径分别在组织什么能力、各自适合什么任务与环境、核心 trade-off 在哪里”，不直接把其中任一方案写成通用最优解

## 一、定位

如果说 `tool-use/` 关注单智能体如何调用外部能力，`architectural-patterns/` 关注这些能力如何被组合成工作流，那么本文件进一步讨论的是：**单智能体究竟应更像一个围绕外部工具与状态系统组织起来的 orchestrator，还是一个尽量把更多判断与生成都收进统一模型内部的大一体系统**。

这条主轴之所以重要，不只是因为“要不要多接几个工具”，而是因为它会改变系统对下列问题的理解方式：

- 能力主要来自模型本身，还是来自外部可组合部件
- 任务状态主要放在长上下文里，还是分散在 memory、tool outputs 与 artifacts 中
- 失败后更适合继续让模型重想，还是切换工具、重构工作流或依赖外部反馈
- 系统更像一个统一生成器，还是一个围绕 trusted components 协调执行的控制层

因此，tool-centric vs monolithic 不是产品包装差异，而是 single-agent 系统组织方式上的核心分岔。

---

## 二、为什么它必须单列成主题

### 2.1 它不是单纯的 tool use 子问题

如果只把这个问题塞进 `tool-use/`，很容易把它误写成“工具多一点还是少一点”。

但真实分歧更深：

- tool-centric 路径会把模型视为 orchestrator
- monolithic 路径会尽量让模型内部承担更多理解、推理与状态整合
- 二者对 memory、planning、feedback 与 environments 的依赖方式都不同

所以这不是单一能力点，而是跨目录结构性主轴。

### 2.2 它直接影响 single-agent 的边界定义

当系统大量依赖外部工具、memory、protocol 与 runtime 时，agent 更像一个协调层；而当系统主要依赖大模型内部能力和长上下文时，agent 更像一个统一推理体。

这会进一步影响：

- 什么算“能力增强”
- 什么算“外部依赖”
- 什么问题应由模型内部解决，什么问题应由外部系统承担

### 2.3 不单列它，会让 overview 中的主轴停留在口号层

`overview.md` 已经指出 `Tool-centric Design vs Monolithic Agent` 是关键矛盾，但如果没有专门主题，后续很容易只在不同目录零散碰到：

- `tool-orchestration.md` 讨论工具链组织
- `memory/` 讨论外部状态保持
- `self-reflection/` 讨论模型内部修正

缺少一个总览文档时，这些材料难以被收束成同一条组织逻辑。

---

## 三、两种路径分别在强调什么

### 3.1 Monolithic Agent

monolithic 路径更接近“尽量强的统一模型 + 大上下文”的思路。

它通常强调：

- 尽量把更多判断留在模型内部完成
- 借助更强推理能力减少外部流程拆分
- 用长上下文承接更多状态与历史
- 用单一连续 loop 维持任务一致性

在这种视角里，外部工具当然可以存在，但通常更像能力补丁，而不是系统真正的骨架。

### 3.2 Tool-Centric Agent

tool-centric 路径更接近“模型 + trusted tools + external memory + explicit workflow”的思路。

它通常强调：

- 模型未必要独自完成所有能力
- 外部工具可以承担检索、执行、验证、修改、浏览、运行等关键步骤
- 状态可以分散保存在 artifacts、tool outputs、workspace 与 memory 中
- agent 的核心价值在于选择、组织、协调和恢复，而不是包办所有计算

在这种视角里，模型更像控制层，而不是唯一能力来源。

### 3.3 二者并不是“有工具”与“没工具”的二元对立

现实系统往往位于连续谱上，而不是纯粹二选一。

例如：

- 一个大模型为主、偶尔调用搜索的系统，仍偏 monolithic
- 一个围绕代码执行、文件操作、测试反馈和检索系统构建的 coding agent，通常更偏 tool-centric
- 高层依赖统一模型做全局判断，局部又 heavily 依赖工具链，也可能是中间形态

因此更准确的理解方式是：**哪一侧承担主导性的能力组织责任**。

---

## 四、它们到底在把能力放到哪里

### 4.1 推理与决策位置

monolithic 路径倾向于：

- 把更多决策留在模型内部连续推理中
- 减少显式状态切换与外部流程拆分

tool-centric 路径倾向于：

- 把一部分决策前移到工作流层
- 让模型在工具选择、参数生成、错误恢复中承担控制职责

### 4.2 状态保持位置

monolithic 路径更依赖：

- 长上下文
- 对话历史
- 模型内部连续生成的一致性

tool-centric 路径更依赖：

- external memory
- artifacts
- workspace state
- tool outputs 与 trace

这也是为什么该主轴会直接碰到 `memory/` 与 `05-environments/`。

### 4.3 能力来源位置

monolithic 路径更相信：

- 更强模型
- 更好 prompt
- 更长上下文
- 更强内生推理

tool-centric 路径更相信：

- 工具链可组合性
- 外部验证信号
- 显式状态管理
- 专门系统对专门任务的支撑

### 4.4 错误修复位置

monolithic 路径出错后，通常更容易走向：

- 再想一轮
- 再反思一轮
- 在同一上下文里继续修正

tool-centric 路径出错后，通常更容易走向：

- 重试工具
- 更换工具
- 读取外部反馈
- 回退到上一步 artifact 或 workflow 节点

---

## 五、各自更适合什么场景

### 5.1 更偏 monolithic 的场景

以下任务通常更适合 monolithic 倾向：

- 以语言理解和生成质量为核心的任务
- 外部可验证反馈较少的任务
- 步骤结构相对短、工具依赖较弱的任务
- 需要保持整体语义风格连续性的任务

例如高层分析、长文本整理、开放式解释与单轮综合判断。

### 5.2 更偏 tool-centric 的场景

以下任务通常更适合 tool-centric 倾向：

- 多步执行任务
- 强依赖外部反馈与环境交互的任务
- 需要检索、执行、验证、回滚的任务
- 工具能力明显优于纯语言推理的任务

例如 coding agent、research agent、browser agent、需要严格文件或系统操作的工作流。

### 5.3 高风险任务通常会把系统推向 tool-centric

当错误代价较高时，系统往往不愿只依赖模型内部“再想一轮”。

它更可能要求：

- 外部验证
- traceability
- rollback
- 明确的权限边界
- 可审计的 artifact 流

这使得很多高风险系统天然更靠近 tool-centric 路径。

---

## 六、最重要的结构性矛盾

### 6.1 Unified Reasoning vs External Composability

monolithic 更强调统一推理体验；tool-centric 更强调把不同能力拆给外部可组合部件。

### 6.2 Context Cohesion vs State Explicitness

monolithic 依赖连续上下文带来的整体一致性；tool-centric 则更倾向把状态显式外化，换取可复盘与可恢复性。

### 6.3 Simplicity vs Reliability Infrastructure

monolithic 往往系统表面更简单；tool-centric 往往需要更多 memory、trace、recovery 与 orchestration 设施。

### 6.4 Model Capability Scaling vs System Engineering Scaling

monolithic 更依赖模型本身能力扩张；tool-centric 更依赖系统工程把不同部件组织起来。

---

## 七、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- 模型越强，就越不需要工具
- 只要有很多工具，就一定更像 agent
- tool-centric 天然比 monolithic 更工程化
- monolithic 一定不可靠，tool-centric 一定可控
- 长上下文已经可以替代 external memory 与 workflow design

这些说法都把连续谱问题写成了绝对二选一，忽略了任务类型、风险等级、环境约束和模型能力阶段差异。

---

## 八、与其他专题的关系

- 与 `overview.md`：本文件展开其中 `Tool-centric Design vs Monolithic Agent` 这一主轴，并解释它为何会贯穿 memory、tool-use、patterns 与 environments。
- 与 `tool-orchestration.md`：后者更关注多工具工作流如何组织；本文件更关注系统为什么会选择把能力骨架放在工具链还是统一模型内部。
- 与 `memory-taxonomy-conflicts.md`：当系统更靠近 tool-centric 时，memory 往往更外显；更靠近 monolithic 时，长上下文与内部连续性的重要性会提升。
- 与 `long-context-vs-external-memory.md`：后者进一步展开“状态到底更多留在上下文里还是外部系统里”这条主轴，是本主题在 memory 维度上的直接延伸。
- 与 `reflection-cost-tradeoff.md`：monolithic 路径更容易依赖内部 reflection 修正；tool-centric 路径更容易把修正建立在外部反馈、工具验证与 recovery 上。
- 与 `planning-vs-execution.md`：当系统走向 tool-centric，planning 与 execution 往往更容易显式分层；当系统更 monolithic，二者更可能继续紧耦合。
- 与 `../05-environments/`：权限、执行隔离、traceability 与 rollback 会显著影响 tool-centric 路径的可行性与收益。
- 与 `backlog.md`：本文件对应当前 P0 缺口 `Tool-Centric Design vs Monolithic Agent`，并为后续补充长上下文、tool reliability 与 agent 边界等主题提供上位框架。

---

## 九、当前最值得继续补证的方向

- 主流 coding agent、research agent 与通用 assistant 在这条连续谱上的典型落点
- 更强长上下文模型是否真的会把系统重新推回 monolithic 路径
- tool-centric 系统的可靠性收益，哪些来自工具本身，哪些来自 trace / rollback / workflow design
- 哪些任务虽然频繁使用工具，但本质上仍由统一模型主导
- 是否存在稳定的中间形态，既保留 monolithic 一致性，又不过度依赖重型 orchestration
