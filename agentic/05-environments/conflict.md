# Environments Conflict

> 适用范围：`05-environments/` 范围内的术语冲突、事实分歧、结论矛盾与待核验问题
> 使用说明：本文件只记录会影响 `code-execution-environments / sandboxing-and-safety / browser-environments / simulated-environments / benchmarking-frameworks` 知识组织的问题；它是研究输入，不是主干定论

## 一、待核验问题

### 1.1 Docker Sandbox 是否足以保证 Agent 安全

- **冲突类型**：结论
- **当前较稳定结论**：container / VM / remote runtime 这类 execution isolation 只能覆盖 agent 安全的一部分；它们可以限制运行位置与破坏半径，但不能替代 permission policy、capability control、rollback / recovery、traceability 与 audit 这些环境层机制。
- **仍待核验部分**：system isolation、capability safety、anti-jailbreak sanitation、恢复机制与审计能力之间的最小必要组合是什么；哪些安全要求必须在 policy layer、governance layer 或 recovery layer 解决，哪些才是 sandbox 自身职责。
- **涉及范围**：`sandboxing-and-safety/permission-policy.md`、`sandboxing-and-safety/permission-vs-execution-boundary.md`、`sandboxing-and-safety/sandbox-layers.md`、`overview.md`
- **为什么重要**：会决定 `05-environments/` 是否只把 environment 理解为执行容器，还是扩展为权限层、执行层、可观测/恢复层与治理层的多层结构。
- **建议处理方向**：主干已可把“Docker sandbox 不足以单独定义 Agent 安全”写成稳定认识；后续重点转向不同安全维度的职责切分，而不是继续把它写成单轴对立。

### 1.2 Environment 是否等于 Execution Container

- **冲突类型**：术语 / 概念
- **当前较稳定结论**：在 agent 系统中，environment 不应被简化为 execution container、VM 或浏览器实例；更合适的最小工作定义应至少覆盖 permission、execution、observability / recovery，并在 sandbox 语境下进一步看到 autonomy governance 的存在。
- **仍待核验部分**：environment 的外延应在什么位置停止，哪些内容仍应归入 tool executor、policy engine、evaluation layer 或交互层，而不是无限扩张成一切运行相关概念。
- **涉及范围**：`overview.md`、`sandboxing-and-safety/sandbox-layers.md`、`sandboxing-and-safety/permission-vs-execution-boundary.md`、`code-execution-environments/`
- **为什么重要**：会直接影响 `05-environments/` 的组织框架，以及该目录是否能准确体现 agent environment 与传统 runtime 的差别。
- **建议处理方向**：主干已可把“environment 不等于 execution container”写成稳定认识；后续研究重点应转向 environment 与 tool executor、policy system、evaluation 的边界停在哪里。

### 1.3 Workspace 是否等于 Sandbox 内文件系统路径

- **冲突类型**：术语 / 概念
- **当前较稳定结论**：`workspace` 更适合被定义为 task-specific working context，而不是单纯的 sandbox 内目录路径；目录挂载只是常见实现方式，不应反过来充当概念本体。
- **仍待核验部分**：workspace 生命周期是否应与 runtime / sandbox 生命周期绑定，以及在多任务、多 subtask、overlay workspace 场景下应如何表达共享与隔离边界。
- **涉及范围**：`code-execution-environments/workspace-structure.md`、`code-execution-environments/workspace-checkpoint.md`、`code-execution-environments/workspace-traceability.md`、`overview.md`
- **为什么重要**：这会直接影响 workspace、checkpoint、traceability 和 artifact 的组织方式；若概念混淆，恢复与归因边界会持续混乱。
- **建议处理方向**：主干已可把“workspace 不等于 sandbox 内路径”写成稳定认识；后续调研重点转向生命周期绑定、共享模型与实现成本比较。

### 1.4 Rollback 的合理路径是否唯一

- **冲突类型**：设计选择 / 事实
- **当前较稳定结论**：rollback / recovery 不应被预设为单一路径；checkpoint-based、transactional snapshot、redo/replay log、logical reconstructability 等机制都可能成立，但服务的状态粒度与恢复语义不同。
- **仍待核验部分**：不同恢复路径分别适合哪些任务形态，以及 full snapshot、diff checkpoint、logical checkpoint 在代码任务中的恢复成本、审计能力与实现复杂度差异。
- **涉及范围**：`code-execution-environments/workspace-checkpoint.md`、`code-execution-environments/`、`sandboxing-and-safety/`
- **为什么重要**：rollback / recovery 是 agent 环境的重要组成部分，若不记录冲突，容易过早把某一路径当成标准答案。
- **建议处理方向**：保留该条目，但把研究重点从“是否只有一种路径”转向“不同路径如何映射不同任务语义与成本边界”。

### 1.5 Workspace Traceability 应以什么为核心组织单位

- **冲突类型**：术语 / 设计选择
- **冲突描述**：一类实现以 command / event log 为核心组织 trace；另一类更强调 artifact-centric attribution 或 step / task-centric traceability。不同视角都能提供可观测性，但未必都足以支持恢复判断、错误归因与审计。
- **涉及范围**：`code-execution-environments/workspace-traceability.md`、`code-execution-environments/workspace-checkpoint.md`、`overview.md`
- **为什么重要**：会直接影响 workspace 中日志、文件变化、artifact lineage、task trace 与 auditability 的组织方式；若核心单位不清晰，trace 体系容易停留在原始日志堆积。
- **待核验问题**：workspace traceability 的最小必要对象集应包含哪些实体？command trace、artifact attribution、checkpoint metadata 与 task trace 之间应如何关联？
- **建议处理方向**：优先把该问题作为 `workspace-traceability.md` 的后续补证主线，比较 event-first、artifact-centric、task-centric 与 hybrid 四类组织路径。

### 1.6 Headless Autonomy 与 Interactive Safety 是否可兼得

- **冲突类型**：结论
- **冲突描述**：一类系统强调危险操作前必须人工确认；另一类系统追求无人工干预的自主执行与可回滚恢复。两者在设计假设上存在根本张力。
- **涉及范围**：`sandboxing-and-safety/`、`browser-environments/`、`overview.md`
- **为什么重要**：这是 agent 环境区别于传统交互式软件的重要矛盾，会影响权限模型、恢复机制和产品化边界。
- **待核验问题**：是否存在可兼顾自治与安全的折中路径，还是必须按场景做明确取舍？
- **建议处理方向**：分别收集产品安全设计和研究型自治环境方案，区分“默认确认”与“可回滚自治”的不同哲学。

---

## 二、维护规则

- 若问题已获得较稳定证据并形成主干结论，应同步更新相关文档并移除或改写对应条目。
- 若问题明显属于单智能体或多智能体能力层，而非环境层，应在对应目录的 `conflict.md` 建立主条目，本文件只保留环境相关部分。
- 若问题暂时只影响临时调研判断，可继续保留在 `../temp/conflict.md` 作为上游入口。
