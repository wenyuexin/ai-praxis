# Workspace Lifecycle and Sharing Model

> 适用范围：Agent 代码执行环境中的 workspace 生命周期绑定关系、共享/隔离模型与典型实现路径
> 阶段状态：研究主干，持续补充产品案例与多 agent 场景证据
> 使用说明：本文件回答“workspace 生命周期是否必须绑定 runtime / sandbox、多 subtask / 多 agent 如何共享 workspace、overlay / worktree / per-task isolation 各自解决什么问题”，不直接把某一种实现写成唯一标准答案

## 一、定位

如果说 `workspace-structure.md` 讨论的是 workspace 是什么，`workspace-checkpoint.md` 讨论的是 workspace 状态如何保存与恢复，那么本文件进一步讨论的是：**workspace 在任务执行过程中如何存活、与 runtime / sandbox / session 的关系如何组织，以及多个任务或 agent 如何围绕它共享或隔离状态**。

这里真正的问题不是“工作目录放哪”，而是：

- workspace 是否必须跟着某个 runtime 一起创建和销毁
- 多轮任务、多个 subtask 或多个 agent 是否应共享同一 workspace
- 状态复用、隔离强度、启动成本与污染风险如何一起权衡

因此，workspace lifecycle / sharing model 讨论的不是单纯的文件系统布局，而是 code execution environment 中的**状态边界设计**。

---

## 二、先区分几个容易混淆的对象

### 2.1 Workspace

workspace 更适合被理解为 **task-specific working context**：围绕某个任务持续存在的文件、产物、上下文与局部状态集合。

它回答的是：当前任务围绕什么工作域推进。

### 2.2 Runtime

runtime 是 workspace 具体执行时所依附的技术载体，例如本地进程、容器、remote runtime 或其他隔离执行单元。

它回答的是：当前动作实际在哪里执行。

### 2.3 Sandbox

sandbox 更强调执行隔离与副作用边界。

它回答的是：动作一旦发生，破坏半径被限制到哪里。

### 2.4 Session

session 更接近一次可寻址的执行会话或交互连接期。

它回答的是：这一轮交互或执行上下文从何时开始、在什么条件下结束。

### 2.5 为什么必须拆开

如果不区分这四者，就很容易把几种不同关系混成一句话：

- workspace 被挂载进 sandbox
- runtime 为 workspace 提供执行能力
- session 复用已有 workspace
- sandbox 销毁时 workspace 也可能一起销毁

这些关系在实现上可能耦合，但在概念上并不等价。

---

## 三、Workspace 生命周期是否必须绑定 runtime / sandbox

### 3.1 强绑定是一种常见实现，但不是唯一答案

很多系统会采用“runtime 创建时同时建立 workspace，runtime 销毁时同时清理 workspace”的模式。

这种方式的优点是：

- 生命周期简单
- 边界清晰
- 便于高隔离执行
- 更容易把副作用限制在单任务范围内

但它也意味着：

- workspace 状态难以跨 runtime 复用
- 多轮任务切换成本更高
- 一旦任务需要暂停、恢复或分支尝试，就更依赖额外的 checkpoint / snapshot / artifact 保存机制

### 3.2 部分解耦更适合长链任务与恢复场景

当系统需要支持多轮执行、暂停恢复、分支尝试或多 agent 协作时，workspace 往往不会与单个 runtime 严格同寿命。

更常见的解耦路径包括：

- runtime 变化，但 workspace 通过持久卷、远端工作域或逻辑状态描述延续
- workspace 基线保持稳定，但不同 runtime 在其上生成局部可写层
- 会话结束后销毁执行进程，但保留 workspace 元数据、artifact 或 checkpoint 锚点

### 3.3 更稳妥的工作性结论

因此，更稳妥的结论不是“workspace 必须绑定 runtime”或“workspace 必须独立于 runtime”，而是：

- **workspace 可以与 runtime 强绑定**，适合高隔离、短生命周期、一次性交付任务
- **workspace 也可以与 runtime 部分解耦**，适合长任务、恢复、并行尝试与多 agent 协作场景

关键在于系统需要回答：自己优先优化的是隔离、复用、恢复，还是交互连续性。

---

## 四、多 subtask / 多 agent 的三类典型 sharing model

### 4.1 Per-Task Isolated Workspace

每个 task 或 subtask 拥有独立 workspace。

典型特征：

- 状态边界最清楚
- 子任务之间默认不共享可写状态
- 常与 per-task runtime / per-task sandbox 一起出现

适用场景：

- 不可信 agent
- 多租户或高风险执行环境
- 希望每次尝试都能独立回滚、独立归档的任务

主要代价：

- 状态复用弱
- 启动成本高
- 子任务之间共享上下文更依赖显式传递

### 4.2 Shared Baseline + Isolated Overlay

多个 task 共享同一个只读基线，但各自拥有独立的可写层。

典型特征：

- 基线代码、依赖或数据可以复用
- 子任务写入不会直接污染共享基线
- 每个 task 的局部修改可以被丢弃、比较或独立合并

Overlay / copy-on-write 在这里很重要，因为它提供了一条介于“完全共享”和“完全复制”之间的路径：

- read 优先读取基线
- write 落到自己的 delta / upper layer
- 放弃这次尝试时可直接丢弃局部可写层

适用场景：

- 重依赖安装成本较高
- 需要多次试探执行
- 既要复用环境，又要控制局部污染的任务

主要代价：

- 实现复杂度更高
- 并发语义与跨层可见性更难解释
- 对外部副作用和非文件状态的保护并不自动成立

### 4.3 Git Worktree / Branch Isolation

多个 task 或 agent 共享同一个仓库对象基础，但通过独立 worktree / branch 获得局部工作域。

典型特征：

- 文件系统层面不是完全复制
- 任务语义边界通过 branch / directory 切开
- 更适合代码修改任务而非任意环境状态共享

适用场景：

- 多 agent 并行改代码
- 同一仓库内的分支尝试
- headless delegation 或可异步交付的子任务

主要代价：

- 更依赖 Git 作为状态与合并基础设施
- 对运行时依赖、进程态、外部服务态的隔离较弱
- 更适合“代码任务隔离”，未必天然等于“执行环境隔离”

---

## 五、Workspace sharing 与 isolation 的核心权衡

### 5.1 安全隔离 vs 状态复用

隔离越强，越能降低污染、误改和副作用扩散；共享越多，越能降低状态传递与环境准备成本。

这是一条没有免费解的主轴。

### 5.2 启动成本 vs 污染风险

每次都新建独立 workspace，启动和准备成本更高；持续复用 workspace，则更容易累积历史脏状态、隐藏依赖和难解释的副作用。

### 5.3 交互式协作 vs Headless Delegation

交互式协作往往更重视连续上下文和即时观察，因而更容易倾向共享或半共享 workspace；而 headless delegation 更适合把任务切给相对独立的工作域，以降低相互干扰。

### 5.4 文件系统隔离 vs 任务语义隔离

有些系统在文件系统层面做了隔离，但任务语义仍然混在一起；另一些系统虽然共享底层对象存储，却通过 branch、attempt、checkpoint 或 task ownership 建立了清晰的任务边界。

因此，workspace model 不只是“目录怎么分”，还涉及**任务语义是否能被清楚切开**。

---

## 六、Overlay / Copy-on-Write 在这里的意义

### 6.1 它首先是一条共享与隔离之间的中间路径

overlay / copy-on-write 的价值，不在于它“更先进”，而在于它提供了一条重要中间路线：

- 既不要求每个任务完整复制一份基线
- 也不要求所有任务直接写同一个 workspace

### 6.2 它更接近局部 revert，而不是完整 checkpoint

overlay revert 更适合回答的是：

- 当前尝试写入了哪些局部改动
- 如果放弃这次尝试，能否快速丢弃这些改动

而不是：

- 是否已经完整保存了任务状态
- 是否已经记录了足够支持恢复的 checkpoint metadata
- 是否已经具备完整 traceability

所以，overlay / copy-on-write 很重要，但它更像 **workspace sharing model 的一种关键实现路径**，而不是 `workspace-checkpoint.md` 中意义上的完整 checkpoint / snapshot 替代品。

### 6.3 为什么不能把它写成唯一标准

overlay 对文件工作域很有吸引力，但并不自动覆盖：

- 外部服务状态
- 长时间运行进程状态
- 权限上下文变化
- 人工接管或多 agent 协同中的语义边界

因此它应被写成重要路径，而不是普适标准答案。

---

## 七、与已有专题的边界

### 7.1 与 `workspace-structure.md`

前者回答 workspace 是什么；本文件回答 workspace 在执行过程中如何存活、如何共享、如何隔离。

### 7.2 与 `workspace-checkpoint.md`

前者回答 checkpoint / snapshot / logical checkpoint 的差异；本文件只在需要时讨论这些机制如何支撑 lifecycle 解耦，不替代 checkpoint 专题。

### 7.3 与 `workspace-traceability.md`

前者回答 workspace 中的动作与产物如何被归因；本文件只讨论不同 sharing model 会如何影响 trace 的边界与归属难度。

### 7.4 与 `traceability-object-model.md`

前者回答 traceability 至少应把哪些对象显式建模；本文件不重复对象建模，而是关注对象在不同 workspace 生命周期模型下如何被切分和复用。

### 7.5 与 `rollback-recovery-design-paths.md`

前者回答 rollback / recovery 可以走哪些路径；本文件只在需要时指出不同 workspace model 会塑造恢复成本与恢复边界。

---

## 八、最重要的结构性矛盾

### 8.1 Lifecycle Coupling vs Recoverable Continuity

workspace 越和单个 runtime 紧绑定，生命周期越简单；但任务暂停、恢复、分支尝试与跨 runtime 延续就越困难。

### 8.2 Shared Context vs Contamination Control

共享上下文越多，协作与复用越顺；但污染控制、责任归因和并发安全也越难。

### 8.3 Fast Delegation vs Strong Isolation

把子任务快速分发给共享或半共享 workspace 可以降低切换成本；但越强调无缝 delegation，越需要额外机制控制副作用与任务边界。

### 8.4 Filesystem Efficiency vs Semantic Clarity

底层实现越高效，未必意味着任务语义越清晰；反之，语义边界越清楚，往往越需要额外对象、元数据或流程约束来维持。

---

## 九、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- workspace 生命周期必须与 runtime / sandbox 完全绑定
- shared baseline + overlay 一定优于 per-task isolated workspace
- git worktree 足以替代所有 workspace sharing 设计
- overlay revert 就等于完整 checkpoint / snapshot
- 共享 workspace 的问题只属于文件系统效率，不涉及任务语义与治理边界

这些说法都会把本来高度情境化的生命周期与共享模型问题，误写成单一实现偏好。

---

## 十、与其他文档的关系

- 与 `workspace-structure.md`：本文件在其“workspace 是 task-specific working context”的基础上，继续展开 workspace 生命周期和共享/隔离模型。
- 与 `workspace-checkpoint.md`：本文件只讨论 lifecycle 解耦与 overlay / revert 的关系；checkpoint 的状态粒度问题仍以该文为主。
- 与 `workspace-traceability.md`：workspace model 会直接影响 trace 的局部归因边界，但不替代 traceability 主题。
- 与 `rollback-recovery-design-paths.md`：不同 workspace model 会塑造恢复成本与恢复路径选择。
- 与 `traceability-object-model.md`：多 task / 多 agent 的 workspace sharing 会影响 task、attempt、artifact、checkpoint 等对象的归属边界。
- 与 `../conflict.md`：本文件进一步收窄 `Workspace 是否等于 Sandbox 内文件系统路径` 的剩余问题，把焦点转向 lifecycle 绑定与 shared / isolated / hybrid 的取舍。
- 与 `../backlog.md`：本文件对应 `Workspace Lifecycle Binding / Sharing Model` 这一 P0 缺口的主干化沉淀。

---

## 十一、Evidence

- Status: Inferred / Observed
- Sources: `agentic/temp/web-search/4.md`, `workspace-structure.md`, `workspace-checkpoint.md`, `workspace-traceability.md`, `rollback-recovery-design-paths.md`, `traceability-object-model.md`
- Trace: 从 `web-search/4.md` 中关于 workspace 生命周期绑定、共享基线、overlay / copy-on-write、git worktree 与 per-task isolation 的材料回流，结合现有 workspace / checkpoint / traceability 专题整理成主干专题。
- Needs: 继续补充官方文档、开源实现源码和跨产品对照，特别是多 agent 共享边界、overlay 并发语义、remote runtime 与 worktree 成本比较。

关键 Claim：

| Claim | Status | Sources | Notes |
|---|---|---|---|
| Workspace 不应被等同为 sandbox 内路径，而应作为 task-specific working context 理解。 | Inferred | `workspace-structure.md`, `agentic/temp/web-search/4.md` | 已进入主干口径，但仍需更多产品定义对照。 |
| Workspace 生命周期可以与 runtime / sandbox 强绑定，也可以通过持久化状态、checkpoint、overlay 或 worktree 部分解耦。 | Observed / Inferred | `agentic/temp/web-search/4.md`, `workspace-checkpoint.md`, `rollback-recovery-design-paths.md` | 具体系统的绑定程度仍需逐一核验。 |
| Per-task isolation、shared baseline + isolated overlay、git worktree / branch isolation 是多 subtask / 多 agent 场景中值得比较的三类 workspace sharing model。 | Inferred | `agentic/temp/web-search/4.md`, `workspace-structure.md` | 作为工作性分类，不写成行业标准。 |
| Overlay / copy-on-write 更适合作为共享与隔离之间的实现路径，不应等同于完整 checkpoint / snapshot。 | Inferred | `agentic/temp/web-search/4.md`, `workspace-checkpoint.md`, `rollback-recovery-design-paths.md` | 需要继续补充具体实现的并发语义与非文件状态边界。 |

---

## 十二、当前最值得继续补证的方向

- 主流 agent 产品中 workspace 与 runtime / sandbox / session 的绑定关系，是否存在稳定分型
- per-task isolation、shared baseline + overlay、git worktree 三类模型在真实代码任务中的长期维护成本比较
- 多 agent 场景下 shared workspace 的最小可接受共享边界应该如何定义
- overlay、remote runtime、persistent volume、branch-based isolation 在恢复、追踪与治理上的组合方式
- workspace sharing model 是否应进一步拆成独立的 lifecycle 与 sharing 两个专题
