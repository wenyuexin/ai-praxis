# Environments Overview

> 适用范围：Agent 执行环境与安全边界的整体认知框架
> 阶段状态：研究综述，持续补充
> 使用说明：本文件回答“Agent 环境由哪些层构成、它们分别解决什么问题、当前最重要的矛盾与误区是什么”，不直接等同于具体 sandbox 技术选型

## 一、定位

`05-environments/` 研究的是 Agent 所运行、交互、执行和被约束的环境。

这里的“环境”不应只被理解为：

- 一个 Docker 容器
- 一个浏览器
- 一个代码执行 runtime

更合适的理解是：环境负责定义 Agent **能看到什么、能做什么、实际在哪里执行、如何被观察与回滚**。

因此，本目录的价值不在于列举执行容器技术，而在于明确：

- 环境如何塑造 agent 的能力边界
- 隔离、权限、状态管理和审计如何共同构成环境层
- 为什么 agent 环境与传统程序运行时有本质不同

---

## 二、建议采用的三层理解框架

### 2.1 Permission Layer

权限层回答：**agent 被允许做什么**。

典型问题包括：

- 哪些工具、命令、网络访问或文件路径可用
- 是否需要用户确认
- 是否存在 capability whitelist / blacklist
- 如何避免越权操作、误执行和高风险行为扩散

这提醒我们：agent 安全不只是“把代码放到容器里跑”。

### 2.2 Execution Sandbox

执行层回答：**agent 实际在哪里执行**。

包括但不限于：

- 本地进程
- subprocess / venv
- container
- microVM
- remote runtime
- browser environment
- simulated environment

这一层的核心不只是技术选择，而是明确隔离强度、资源模型、启动延迟和副作用边界。

### 2.3 Observability and Recovery

可观测与恢复层回答：**agent 做了什么、如何追踪、如何恢复**。

典型问题包括：

- logs / trajectory / trace
- checkpoint / snapshot
- rollback / recovery
- workspace artifact management
- auditability

这一层对长期任务、自动化修改、代码执行和自治系统尤其关键。

同时需要加一条 stop-line：**恢复连接能力不等于文件系统可重建能力**。一个系统可能已经支持 conversation restore、runtime resume、agent-server 重连或 sandbox identity 恢复，但这并不自动证明底层 workspace 文件状态能脱离原 runtime / volume / working_dir 连续性被独立 checkpoint / replay / restore。

### 2.4 Minimal Safety Composition

如果只停留在“permission / execution / observability-recovery”三层结构，仍容易产生一个误解：好像只要某一层做得足够强，系统就已经安全。

更稳妥的理解是，Agent 环境通常需要一个**最小安全组合**：

- **permission / capability control**：限制哪些动作被允许发生
- **execution isolation**：限制动作一旦发生，副作用能扩散到哪里
- **recovery / traceability**：保证失败后能解释发生了什么，并恢复到可继续状态
- **governance / mode control**：决定系统默认何时自动执行、何时停下来请求人类确认

其中前三层仍可视为 `05-environments/` 的基础分析框架；而 `governance` 更适合作为一个横跨它们的工作模式维度。也就是说：

- 安全不只是“把代码放进容器里跑”
- 安全不只是“给高风险动作加确认”
- 安全也不只是“出了问题可以回滚”

只有把这些维度组合起来，才更接近 Agent 系统中的真实安全结构。

### 2.5 Environment Boundary Stop-Line

在 Agent 语境里，`environment` 不应被简化成 execution container，也不应无限膨胀为一切运行相关概念。

一个更稳妥的停线规则是：environment 至少覆盖以下四类边界条件——agent 能看到什么、能做什么、实际在哪里执行、失败后如何被追踪和恢复；但不应把 tool executor 之外的所有控制逻辑都吞进去。

在恢复语义上，还应额外记住一个边界提醒：environment 层可以稳定讨论 persistence、traceability、checkpoint、rollback / recovery hooks，但不应在证据不足时把“会话 / sandbox / runtime 可恢复”直接写成“workspace 文件系统可独立重建”。前者属于 environment 的可恢复能力，后者则涉及更强的状态保存粒度与恢复证明。

因此可以把以下内容视为 environment 的稳定组成部分：

- **observation boundary**：agent 能读取哪些文件、页面、状态与上下文
- **action space under permission**：agent 在授权约束下可调用哪些工具、命令、接口
- **execution isolation**：动作在本地进程、容器、microVM 还是 remote runtime 中发生
- **state persistence / recovery hooks**：checkpoint、traceability、rollback / recovery 如何支撑持续执行

与之相对，以下内容更适合作为与 environment 紧耦合但不应直接混同的外围层：

- 独立的 policy engine
- 更高层的 evaluation logic
- 面向交互层的 UI / conversation orchestration

这样处理的好处是：既能稳定写下 `environment != execution container`，又能避免把 environment 扩张成“模型之外的一切”。

---

## 三、核心子主题如何落位

### 3.1 Code Execution Environments

聚焦代码执行、依赖管理、测试运行、workspace 生命周期和文件副作用边界。

### 3.2 Sandboxing and Safety

聚焦隔离、权限控制、资源限制、回滚、审计和 agent-specific safety 问题。

### 3.3 Browser Environments

聚焦网页交互环境、DOM 观测、事件触发、会话状态与浏览器安全边界。

### 3.4 Simulated Environments

聚焦可重复实验环境、任务世界模型、仿真交互与受控评测场景。

### 3.5 Benchmarking Frameworks

聚焦环境相关评测框架及其如何定义任务、状态、观察、动作与成功标准。

---

## 四、当前最重要的结构性矛盾

### 4.1 隔离强度 vs 执行效率

更强的隔离通常意味着更高的启动成本、更复杂的运行时管理和更多执行开销。

从 OpenHands 的 runtime / sandbox / workspace 调度路径看，这个成本不只来自“隔离技术本身”，还来自执行环境编排链路：sandbox start / resume 轮询、Docker / remote runtime 同步调用、workspace bridge 的 HTTP 往返、event persistence 的全量读取，以及清理回收路径的串行等待，都会把隔离边界转化成尾延迟和吞吐压力。

这条 trade-off 没有免费解：安全性、速度、复杂度、可恢复性和可维护性始终相互牵制。

### 4.2 “在哪里执行” vs “允许做什么”

传统 sandbox 讨论往往只关注执行位置（container / VM / remote runtime），但 agent 环境必须额外关注 capability-level control。

也就是说：

- sandbox 解决的是运行位置与破坏范围
- permission policy 解决的是行动授权与能力边界

两者不能互相替代。

### 4.3 Interactive Safety vs Headless Autonomy

一类系统强调高风险操作前的人类确认；另一类系统追求无人工介入的自治执行。

这不是简单实现细节，而是两种不同的环境哲学：

- 安全优先
- 自治优先

现实系统往往需要在两者之间做取舍。

### 4.4 Workspace Sharing vs Workspace Isolation

共享 workspace 能降低状态传递成本，但也更容易带来污染、干扰和副作用扩散；完全隔离则更干净，但成本更高、协作更复杂。

### 4.5 Connection Recovery vs Filesystem Reconstruction

系统越强调 conversation restore、runtime resume、sandbox reuse 或 agent-server 重连，越容易给人一种“恢复能力已经足够强”的印象；但如果没有额外证据，这并不等于底层 workspace 文件状态已经可被独立重建。

调度性能也会放大这一区分：恢复连接通常还要经过 sandbox resume、健康检查、WebSocket 重连、workspace backend 可用性确认和 event 读取等步骤；这些路径变慢时，用户感知到的是“恢复慢”，但根因可能是执行实体重连、状态读取或工作域重建中的某一层，而不是单一 recovery 机制失效。

也就是说：

- 会话 / 连接恢复回答的是“还能否继续交互与执行”
- 文件系统重建回答的是“脱离原执行实体后，能否把同一任务工作域重新造出来”

LangGraph 的补证结果进一步强化了这条 stop-line：一个系统可以稳定支持 thread-scoped checkpoint、durable execution、interrupt / resume、time travel replay / fork 与 `update_state`，从而让 workflow state 在中断后继续推进；但这仍不等于它默认保存了 workspace 文件内容、容器状态或外部副作用状态。也就是说，workflow recovery 不是 execution environment recovery，更不是 workspace filesystem reconstruction。

两者相关，但不能混成一个恢复指标。

---

## 五、最容易被误写成定论的问题

当前阶段尤其要避免以下误解：

- Docker sandbox 足以保证 AI agent 安全
- environment 等于 execution container
- workspace 等于容器中的某个目录路径
- rollback 只有一种合理实现路径
- 恢复连接 / 会话恢复自动等于 workspace 文件系统可独立重建
- 只要官方文档列出 session / sandbox / tracing / state 模块，就自动等于 runtime/job resume 或 workspace recovery 已被证实
- 环境调度慢主要是 Python 语言性能问题
- 更强隔离一定意味着更优系统设计

这些问题都更适合先进入 `conflict.md` 或专题文档，而不是写成总论定论。

---

## 六、建议的目录理解方式

可以把 `05-environments/` 理解为一个三层相互耦合的系统：

```text
permission layer
  → execution sandbox
  → observability / recovery
```

不同子目录可理解为对这三层中某些部分的展开：

- `code-execution-environments/`：偏执行层与 workspace
- `sandboxing-and-safety/`：偏权限层与隔离层
- `browser-environments/`：偏特定交互环境
- `simulated-environments/`：偏受控任务世界
- `evaluation-environments/`：偏环境如何被评测和复现

---

## 七、与其他领域的概念关系

环境层定义的是 agent 能看到什么、能做什么、在哪里执行以及如何被观察和恢复，因此会持续影响单智能体、多智能体、框架工具与评估等主题：例如执行环境会改变工具调用与状态保持方式，共享环境会抬高协调成本，不同框架工具建立在不同环境假设之上，而评估是否可信也依赖环境是否可复现、可审计和可控。

具体目录分工与导航入口由 `agentic/README.md` 和各目录 `README.md` 承担。

---

## 八、当前最值得补齐的专题

最近一轮已经优先补上了 `workspace` 相关主线：

- `code-execution-environments/workspace-structure.md`：澄清 workspace 是 task-specific working context，而不只是 sandbox 内目录路径
- `code-execution-environments/workspace-checkpoint.md`：展开 snapshot、checkpoint、rollback 与 recovery 的状态粒度问题，并明确“会话恢复 / 连接恢复”不自动等于“文件系统可独立重建”
- `code-execution-environments/workspace-traceability.md`：展开日志、轨迹、artifact 归因与 auditability 边界

因此，`workspace / checkpoint / traceability` 这一组主题已从“待补齐”推进为“已有主干骨架，后续继续补证”。

当前更值得继续补齐的专题包括：

- permission layer 与 execution layer 的边界
- sandbox layers 的系统拆分
- headless autonomy vs interactive safety
- rollback / recovery 设计路径比较
- workspace lifecycle binding、shared vs overlay workspace、traceability object model 等后续补证问题

这些更适合按“已形成主干专题”与“仍需继续补证的问题”分开推进：前者继续完善正文，后者优先进入 `conflict.md`、`backlog.md` 与后续专题补证清单，再逐步反哺主干。
