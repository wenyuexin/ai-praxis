# Workspace Structure

> 适用范围：Agent 代码执行环境中的 workspace 定义、组织方式与共享/隔离边界
> 阶段状态：研究主干，持续补充框架案例与状态管理证据
> 使用说明：本文件回答“workspace 到底是什么、它为什么不应等同于容器中的目录路径、常见组织方式与核心矛盾是什么”，不直接等同于某个框架的具体挂载实现

## 一、定位

在 Agent 代码执行环境里，workspace 更适合被理解为：**任务运行期间承载文件、产物、中间状态与工作上下文的工作域**。

它不只是一个目录名，也不只是一个挂载点。

如果把 workspace 简化为“容器里的 `/workspace` 路径”，会遗漏几个关键事实：

- workspace 服务的是任务上下文，不只是文件存放
- workspace 可以与 sandbox 实现强耦合，也可以与 runtime 正交
- workspace 既影响状态管理，也影响协作、恢复与审计

因此，workspace 应被当成 code execution environment 中的桥梁概念：它连接执行层、状态层与多任务协作边界。

---

## 二、为什么它必须单列成主题

### 2.1 它不等于执行容器中的某个目录

许多系统实现上会把 workspace 挂载到固定路径，但这只是实现方式，不是概念本体。

workspace 真正回答的是：

- 本次任务的文件上下文放在哪里
- 哪些中间产物属于当前任务
- 多轮执行如何复用已有状态
- 不同任务之间哪些内容共享，哪些内容隔离

所以“路径”只是载体，`task-specific working context` 才是核心定义。

### 2.2 它天然横跨执行、恢复与协作问题

workspace 不只是文件管理，它还决定：

- 执行时 agent 看到哪些代码与依赖
- checkpoint / rollback 针对哪一组状态生效
- 多个 subtask 是否共享同一上下文
- audit / trace 能否准确归属到某个任务

如果把这些问题混写在一起，又不单列 workspace，后续目录边界会持续模糊。

### 2.3 它是长期任务的基础设施，不只是临时目录

在简单脚本执行里，workspace 似乎只是“工作目录”；但在长链 agent 任务里，它往往承载：

- 多轮修改后的文件状态
- 测试输出与日志
- 下载的依赖或中间 artifact
- 人工介入后的修正痕迹
- 后续恢复或追责所依赖的上下文快照

这意味着 workspace 不是边角料，而是状态连续性的载体。

---

## 三、workspace 到底由什么构成

### 3.1 文件工作域

最直观的一层是文件本身，包括：

- 输入代码
- 新生成或修改的文件
- 配置文件
- 临时脚本
- 下载产物与缓存

但若只看到这一层，会低估 workspace 的系统角色。

### 3.2 任务上下文域

workspace 还隐含一层更重要的上下文：

- 当前任务关心哪些文件
- 当前任务是否延续上一步产物
- 哪些 artifact 被视为本次任务的一部分
- 哪些状态需要在多轮循环中保留

这说明 workspace 不是“任何文件都在里面”，而是**围绕任务边界组织的工作上下文**。

### 3.3 状态桥接域

workspace 还是执行层与恢复层之间的桥梁，因为：

- checkpoint 常以 workspace 状态为核心对象
- rollback 往往回到某个 workspace snapshot
- traceability 需要知道某条日志对应哪个 workspace 状态

所以它既不是纯 execution 概念，也不是纯 recovery 概念，而是二者之间的交叉点。

---

## 四、常见 workspace 组织方式

### 4.1 One Task, One Workspace

每个任务使用独立 workspace。

优点：

- 边界清晰
- 污染少
- 容易回滚与归档
- 适合高隔离要求

代价：

- 状态迁移成本高
- 多任务协作时共享信息不方便
- 重复准备环境的成本更高

### 4.2 Shared Workspace Across Steps

同一任务的多轮执行持续使用同一 workspace。

优点：

- 连续性强
- 文件与 artifact 不需要反复搬运
- 适合代码修改、测试、修复这种连续工作流

代价：

- 状态污染风险更高
- 出错后更难定位是哪一轮引入问题
- 长时间运行后上下文可能越来越脏

### 4.3 Shared Workspace Across Subtasks

多个 subtasks 共享同一 workspace。

优点：

- 协作成本低
- 状态传递快
- 适合需要围绕同一代码库快速协同的场景

代价：

- 干扰更强
- 写冲突更常见
- 副作用来源更难归因

### 4.4 Hybrid Workspace Model

共享只读基础上下文，同时隔离可写空间。

典型思路包括：

- 共享代码快照，只隔离写入层
- 共享依赖缓存，只隔离任务产物
- 共享基线 workspace，为每个分支任务附加 overlay

这类设计更复杂，但常能更好平衡连续性与隔离性。

---

## 五、它与 sandbox 的关系

### 5.1 workspace 不等于 sandbox

一个 sandbox 可以只服务一个 workspace，也可以服务多个 workspace；反过来，一个 workspace 也可能被不同 runtime 或不同 sandbox 周期复用。

因此两者更合适的关系是：

- sandbox 定义执行边界
- workspace 定义工作上下文边界

前者偏“在哪里跑”，后者偏“围绕什么状态工作”。

### 5.2 二者在实现上常耦合，但概念上应拆开

很多框架中二者之所以常被混同，是因为实现上常出现：

- container 启动时挂载 workspace
- sandbox 销毁时 workspace 一起清理
- runtime 生命周期与 workspace 生命周期绑定

但这只是实现耦合，不应反过来定义概念。

### 5.3 一旦不拆开，后续恢复与追踪会变乱

如果把 workspace 直接当作 sandbox 内路径，那么：

- checkpoint 可能被误解为容器快照，而不是任务状态快照
- traceability 可能只记录进程日志，而忽略文件上下文变化
- 多任务协作时难以区分“共享执行环境”与“共享工作上下文”

---

## 六、最重要的结构性矛盾

### 6.1 Workspace Sharing vs Workspace Isolation

共享 workspace 有利于连续执行和协作效率；隔离 workspace 有利于减少污染与副作用扩散。

这是一条没有免费解的结构性 trade-off。

### 6.2 Context Continuity vs State Cleanliness

连续性越强，任务越容易延续已有状态；但状态越容易累积污染、隐藏依赖和历史包袱。

所以“保留上下文”与“保持干净”天然相互拉扯。

### 6.3 Artifact Richness vs Management Complexity

workspace 容纳的 artifact 越丰富，后续恢复、审计和复用越方便；但结构越复杂，管理、清理和定位问题也越困难。

### 6.4 Runtime Coupling vs Conceptual Independence

实现上把 workspace 绑定到 runtime 往往更简单；但概念上过度绑定，会让系统失去对 checkpoint、traceability 和多执行形态复用的清晰表达。

---

## 七、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- workspace 就是容器里的工作目录
- 只要 sandbox 隔离得够强，workspace 组织不重要
- shared workspace 一定更高效
- isolated workspace 一定更安全
- workspace 问题只属于文件系统设计，不涉及恢复和审计

这些说法都会把 workspace 从环境层的桥梁概念，误写成单纯目录管理问题。

---

## 八、与后续专题的关系

- 与 `../overview.md`：本文件展开其中的 `workspace artifact management` 问题，强调 workspace 是 execution 与 observability/recovery 的桥梁。
- 与 `../conflict.md`：对应 `Workspace 是否等于 Sandbox 内文件系统路径` 这一核心冲突条目。
- 与后续 `workspace-checkpoint.md`：后者将进一步讨论 snapshot、rollback 与恢复路径，不再重复本文件中的结构定义。
- 与后续 `workspace-traceability.md`：后者将继续展开日志、轨迹、artifact 归因与审计边界。
- 与 `../sandboxing-and-safety/sandbox-layers.md`：本文件可视为其中 `state / recovery layer` 与 `execution layer` 的桥梁专题。

---

## 九、当前最值得继续补证的方向

- 主流 agent 框架中 workspace 生命周期是否与 runtime 生命周期绑定
- shared workspace、overlay workspace、isolated workspace 三类模型的长期维护成本比较
- workspace 应如何同时承载文件、artifact、checkpoint metadata 与 trace metadata
- 多 agent / 多 subtask 场景下 workspace sharing 的最小可接受隔离边界
- workspace 定义是否应作为 execution environment 的一级概念，而不是附属实现细节
