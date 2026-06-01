# Environments Conflict

> 适用范围：`05-environments/` 范围内的术语冲突、事实分歧、结论矛盾与待核验问题
> 使用说明：本文件只记录会影响 `code-execution-environments / sandboxing-and-safety / browser-environments / simulated-environments / benchmarking-frameworks` 知识组织的问题；它是研究输入，不是主干定论

## 一、待核验问题

### 1.1 Docker Sandbox 是否足以保证 Agent 安全

- **冲突类型**：结论
- **冲突描述**：一类说法把 container isolation 视为 agent 安全的主要保障；另一类则强调 capability control、permission policy、anti-jailbreak sanitation、rollback 与 audit 同样属于 agent 环境安全的关键部分。
- **涉及范围**：`sandboxing-and-safety/`、`overview.md`
- **为什么重要**：会决定 `05-environments/` 是否只把 environment 理解为执行容器，还是扩展为权限层、执行层、可观测/恢复层的多层结构。
- **待核验问题**：system isolation 与 capability safety 之间的边界如何定义？哪些安全要求是 sandbox 自身能解决的，哪些必须由 policy layer 补足？
- **建议处理方向**：优先查看安全框架论文、官方安全设计文档与高质量技术分析，区分不同安全维度。

### 1.2 Environment 是否等于 Execution Container

- **冲突类型**：术语 / 概念
- **冲突描述**：传统运行时讨论容易把 environment 简化为容器、VM 或浏览器实例；而 agent 系统中，environment 还可能包含 permission policy、observability、checkpoint 与 recovery。
- **涉及范围**：`overview.md`、`code-execution-environments/`、`sandboxing-and-safety/`
- **为什么重要**：会直接影响 `05-environments/` 的组织框架，以及该目录是否能准确体现 agent 环境与传统 runtime 的差别。
- **待核验问题**：环境概念应覆盖哪些层？哪些应归入 tool executor、policy system 或 evaluation layer？
- **建议处理方向**：通过对比论文、框架实现和产品文档，建立 environment 的最小必要范围。

### 1.3 Workspace 是否等于 Sandbox 内文件系统路径

- **冲突类型**：术语 / 概念
- **冲突描述**：部分实现把 workspace 与容器挂载目录绑定；另一类观点认为 workspace 应理解为 task-specific working context，与 runtime / sandbox 正交。
- **涉及范围**：`code-execution-environments/`、`overview.md`
- **为什么重要**：会影响 workspace、checkpoint、traceability 和 artifact 组织方式；若概念混淆，后续目录很难拆清。
- **待核验问题**：workspace 的核心定义究竟是文件位置、任务上下文，还是二者组合？
- **建议处理方向**：查看 OpenHands、LangGraph 等系统的 workspace / checkpoint 设计，区分实现耦合与概念本体。

### 1.4 Rollback 的合理路径是否唯一

- **冲突类型**：设计选择 / 事实
- **冲突描述**：checkpoint-based、transactional snapshot、redo/replay log 等方案都声称可支持恢复，但它们在粒度、性能开销和可审计性上差异明显。
- **涉及范围**：`code-execution-environments/`、`sandboxing-and-safety/`
- **为什么重要**：rollback / recovery 是 agent 环境的重要组成部分，若不记录冲突，容易过早把某一路径当成标准答案。
- **待核验问题**：不同恢复路径分别适合哪些任务形态？它们对长任务、代码修改和自治执行的影响有何差异？
- **建议处理方向**：建立对照表，比较 checkpoint、transaction 和 replay 三类恢复机制。

### 1.5 Headless Autonomy 与 Interactive Safety 是否可兼得

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
