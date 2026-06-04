# Environments Conflict

> 适用范围：`05-environments/` 范围内的术语冲突、事实分歧、结论矛盾与待核验问题
> 使用说明：本文件只记录会影响 `code-execution-environments / sandboxing-and-safety / browser-environments / simulated-environments / benchmarking-frameworks` 知识组织的问题；它是研究输入，不是主干定论

## 一、待核验问题

### 1.1 Docker Sandbox 是否足以保证 Agent 安全

- **冲突类型**：结论
- **当前较稳定结论**：Docker / VM / remote runtime 这类 execution isolation 只能覆盖 agent 安全的一部分；它们可以限制动作发生的位置与破坏半径，但不能替代 permission policy、capability control、rollback / recovery、traceability、audit 与 governance。`safety-composition.md` 已将该问题收敛为 permission / execution isolation / recovery-traceability / governance 的最小安全组合，而不是“是否使用 Docker”的单点问题。
- **仍待核验部分**：不同任务形态下四层安全组合如何调参；哪些能力必须由 permission / governance 解决，哪些属于 sandbox 自身职责；不同隔离实现（local process、container、microVM、remote runtime）在成本、恢复能力和风险边界上的真实差异。
- **涉及范围**：`sandboxing-and-safety/safety-composition.md`、`sandboxing-and-safety/permission-policy.md`、`sandboxing-and-safety/permission-vs-execution-boundary.md`、`sandboxing-and-safety/sandbox-layers.md`、`overview.md`
- **为什么重要**：会决定 `05-environments/` 是否只把 environment 理解为执行容器，还是扩展为权限层、执行层、可观测/恢复层与治理层的多层结构。
- **建议处理方向**：不再继续讨论“Docker 是否足够”这个二选一问题；后续重点比较不同安全层的职责切分、组合方式和适用场景。

### 1.2 Environment 是否等于 Execution Container

- **冲突类型**：术语 / 概念
- **当前较稳定结论**：在 agent 系统中，environment 不应被简化为 execution container、VM 或浏览器实例；更合适的最小工作定义应至少覆盖 permission、execution、observability / recovery，并在 sandbox 语境下进一步看到 autonomy governance 的存在。
- **仍待核验部分**：environment 的外延应在什么位置停止，哪些内容仍应归入 tool executor、policy engine、evaluation layer 或交互层，而不是无限扩张成一切运行相关概念。
- **涉及范围**：`overview.md`、`sandboxing-and-safety/sandbox-layers.md`、`sandboxing-and-safety/permission-vs-execution-boundary.md`、`code-execution-environments/`
- **为什么重要**：会直接影响 `05-environments/` 的组织框架，以及该目录是否能准确体现 agent environment 与传统 runtime 的差别。
- **建议处理方向**：主干已可把“environment 不等于 execution container”写成稳定认识；`overview.md` 已补入 boundary stop-line 的工作性定义。后续研究重点应转向 environment 与 tool executor、policy system、evaluation 的边界停在哪里。

### 1.3 Workspace 是否等于 Sandbox 内文件系统路径

- **冲突类型**：术语 / 概念
- **当前较稳定结论**：`workspace` 更适合被定义为 task-specific working context，而不是单纯的 sandbox 内目录路径；目录挂载只是常见实现方式，不应反过来充当概念本体。
- **仍待核验部分**：workspace 生命周期在什么条件下应与 runtime / sandbox / session 强绑定、在什么条件下应通过 checkpoint、overlay、persistent volume、remote runtime 或 git worktree 部分解耦；shared / isolated / hybrid workspace 在多任务、多 subtask、多 agent 场景下的取舍边界仍需跨系统比较。
- **涉及范围**：`code-execution-environments/workspace-structure.md`、`code-execution-environments/workspace-lifecycle.md`、`code-execution-environments/workspace-checkpoint.md`、`code-execution-environments/workspace-traceability.md`、`overview.md`
- **为什么重要**：这会直接影响 workspace、checkpoint、traceability 和 artifact 的组织方式；若概念混淆，恢复与归因边界会持续混乱。
- **Source / Trace**：`workspace-lifecycle.md` 已吸收 `agentic/temp/web-search/4.md` 中关于 per-task isolation、shared baseline + overlay、git worktree / branch isolation 的可用线索；本冲突条目从“workspace 是否等于路径”收窄为 lifecycle 绑定与 sharing model 取舍问题。
- **建议处理方向**：主干已可把“workspace 不等于 sandbox 内路径”写成稳定认识；后续不再讨论二选一概念冲突，重点比较生命周期绑定、共享模型、恢复成本、traceability 归属和多 agent 并发边界。

### 1.4 Rollback 的合理路径是否唯一

- **冲突类型**：设计选择 / 事实
- **当前较稳定结论**：rollback / recovery 不应被预设为单一路径；`workspace-checkpoint.md` 与 `rollback-recovery-design-paths.md` 已将 checkpoint resume、snapshot recovery、overlay revert、replay / logical reconstruction 等路径区分开来。它们服务的状态粒度和恢复语义不同，不能用“是否能精确回到过去”作为唯一评价标准。
- **仍待核验部分**：不同恢复路径分别适合哪些任务形态；full snapshot、diff checkpoint、logical checkpoint、overlay revert 在代码任务中的恢复成本、审计能力与实现复杂度差异；哪些场景需要保留失败后的中间产物，哪些场景应直接回到干净基线。
- **涉及范围**：`code-execution-environments/workspace-checkpoint.md`、`code-execution-environments/rollback-recovery-design-paths.md`、`code-execution-environments/workspace-traceability.md`、`sandboxing-and-safety/`
- **为什么重要**：rollback / recovery 是 agent 环境的重要组成部分，若不记录冲突，容易过早把某一路径当成标准答案。
- **建议处理方向**：不再继续讨论“是否只有一种合理路径”；后续重点转向恢复语义、任务形态、审计要求与实现成本之间的映射关系。

### 1.5 恢复连接能力是否等于文件系统可重建

- **冲突类型**：术语 / 事实 / 结论力度
- **当前较稳定结论**：当前更稳妥的主干口径应是把“恢复连接 / 恢复会话 / 恢复 sandbox 身份”和“恢复 workspace 文件系统状态”明确拆开。OpenHands 与 `software-agent-sdk` 的交叉源码已经能稳定支持前者：conversation-level persistence / restore、workspace backend 的 `pause()` / `resume()`、agent-server 的持久化 conversation 恢复、以及 `pause` / `interrupt` / `resume` 的不同暂停语义都已成立；但这些证据并不足以直接推出“完整工作目录可脱离原 runtime / volume / working_dir 连续性独立重建”。
- **仍待核验部分**：在 DockerWorkspace、APIRemoteWorkspace、OpenHandsCloudWorkspace 与 agent-server 的具体恢复链路里，`working_dir` 与文件系统内容到底是被同一执行实体继续持有、被卷/挂载延续，还是能由 persisted events / trajectory / logical metadata 独立重建；不同系统里 replay engine 与 artifact traceability 是否足以承担文件系统恢复职责。
- **涉及范围**：`code-execution-environments/workspace-lifecycle.md`、`code-execution-environments/workspace-checkpoint.md`、`code-execution-environments/rollback-recovery-design-paths.md`、`../06-frameworks-and-tools/03-project-studies/openhands/official-docs.md`、`../06-frameworks-and-tools/03-project-studies/openhands/runtime-and-sandbox.md`
- **为什么重要**：如果把 conversation restore 直接写成 workspace checkpoint / filesystem restore，就会把恢复语义、traceability、checkpoint 粒度和 runtime 连续性全部混成一件事，导致环境层关于 recovery、checkpoint 与 sharing model 的边界持续失真。
- **Source / Trace**：该问题最初在 `official-docs.md` 中表现为官方文档对 `persistence_dir`、`base_state.json` 与 `events/` 的恢复表述容易被过度外推；随后在 `runtime-and-sandbox.md` 的 OpenHands + `software-agent-sdk` 交叉源码核验中进一步收敛为：当前证据更支持“先恢复 conversation / runtime identity，再尽量沿用既有 runtime / volume / working_dir 连续性”，而不是“已经证明可单靠 replay 或逻辑状态重建完整工作域”。
- **建议处理方向**：主干已可把“恢复连接能力 ≠ 文件系统可重建”写成稳定边界；后续不再争论是否存在恢复，而是重点比较不同系统把恢复能力落在哪一层，以及这些层之间的耦合强度。

### 1.6 Workspace Traceability 应以什么为核心组织单位

- **冲突类型**：术语 / 设计选择
- **当前较稳定结论**：workspace traceability 不应只以 command / event log 为核心；更稳妥的主干口径是同时显式建模 task / step、action、workspace state / checkpoint、artifact、actor / executor 与 decision / authorization context。`workspace-traceability.md` 已区分 event-first、artifact-centric、task-centric 与 hybrid 四类组织视角，`traceability-object-model.md` 已进一步把问题下沉为最小对象集与关系集。
- **仍待核验部分**：主流 agent 系统是否真的显式建模这些对象；不同系统更偏 event-first、artifact-first、task-first 还是 hybrid；causality、lineage、validity、containment 等关系中哪些最值得优先建模。
- **涉及范围**：`code-execution-environments/workspace-traceability.md`、`code-execution-environments/traceability-object-model.md`、`code-execution-environments/workspace-checkpoint.md`、`overview.md`
- **为什么重要**：会直接影响 workspace 中日志、文件变化、artifact lineage、task trace 与 auditability 的组织方式；若核心单位不清晰，trace 体系容易停留在原始日志堆积。
- **建议处理方向**：不再继续讨论“是否需要对象模型”；后续重点转向跨系统对象映射、关系强度比较，以及不同任务形态下最小对象集是否需要扩展。

### 1.7 Headless Autonomy 与 Interactive Safety 是否可兼得

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
