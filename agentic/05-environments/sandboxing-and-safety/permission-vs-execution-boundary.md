# Permission Layer vs Execution Layer

> 适用范围：Agent 环境中 permission policy 与 execution sandbox 的职责边界、耦合点与常见混淆
> 阶段状态：研究主干，持续补充产品案例与工程证据
> 使用说明：本文件回答“为什么 permission layer 与 execution layer 不能互相替代、它们分别解决什么问题、常见边界混淆来自哪里、核心 trade-off 是什么”，不直接等同于某一种具体产品的授权或隔离实现

## 一、定位

在 Agent 环境里，`permission layer` 与 `execution layer` 经常被同时讨论，但它们回答的其实不是同一个问题。

更准确地说：

- **permission layer** 回答：Agent **被允许做什么**
- **execution layer** 回答：Agent **实际在哪里做，以及副作用能扩散到哪里**

这意味着，前者主要是行动授权与能力边界，后者主要是运行位置、隔离强度与破坏半径。

如果把这两层混成一个概念，环境设计就会迅速退化成两种常见误解：

- 把 sandbox 技术强度误当成完整的安全能力
- 把 command allowlist 或确认弹窗误当成足够的隔离策略

因此，`permission vs execution` 不是一个术语细节，而是 Agent 环境层最基础的职责划分之一。

---

## 二、为什么它必须单列成主题

### 2.1 “能不能做” 与 “做坏了会坏到哪里” 是两类问题

Agent 被允许执行 `rm`、联网下载、写入工作区、访问凭证文件，这些都属于 permission 问题。

即使这些动作被允许，它们仍然需要回答：

- 在本地宿主机执行，还是在容器里执行
- 能写入哪些挂载路径
- 进程与网络副作用会扩散到哪里
- 一旦出错，能否被限域、回滚或审计

这就是 execution layer 的问题。

因此，permission 与 execution 并不是上下位关系，而是两个正交维度。

### 2.2 传统程序安全的直觉会误导 Agent 环境设计

在传统系统里，隔离常被优先理解为：

- 容器化
- VM
- 进程权限
- 系统资源限制

但 Agent 系统的额外风险在于，它不是被动运行固定程序，而是在持续生成动作。

所以除了 containment，还必须回答：

- 这个动作是否应该被发起
- 它是否超出当前任务上下文
- 是否需要确认、升级授权或直接拒绝

如果只谈 execution，不谈 permission，就会遗漏 agent-specific capability control。

### 2.3 两层混写会让后续专题边界持续模糊

一旦不拆开，很多问题都会被混写：

- command allowlist 到底算 sandbox 还是 policy
- 路径白名单属于文件系统隔离还是任务授权
- 网络访问限制是 capability gate 还是 runtime firewall
- 用户确认是 UX 设计还是环境控制

单列这个主题的价值，就是把这些混淆先拆清，再分别回到 `permission-policy.md`、`sandbox-layers.md`、`workspace-*` 和自治专题中。

---

## 三、两层分别回答什么

### 3.1 Permission Layer：定义行动授权

这一层主要回答：

- 哪些工具可用
- 哪些命令可执行
- 哪些路径可读写
- 是否允许联网
- 哪些动作需要确认
- 哪些能力需要升级授权

它更关注：

- capability boundary
- task-scoped authorization
- risk-tiered control
- human approval gate

如果把 Agent 比作一个持续行动的系统，这一层更接近“行动许可”。

### 3.2 Execution Layer：定义运行位置与破坏半径

这一层主要回答：

- 动作在本地进程、subprocess、venv、container、microVM 还是 remote runtime 执行
- 进程之间如何隔离
- 文件系统如何挂载
- 资源如何限制
- 网络、宿主机、子进程和系统调用边界在哪里

它更关注：

- runtime containment
- isolation strength
- side-effect boundary
- startup / performance cost

如果沿用上面的比喻，这一层更接近“允许行动后，行动实际发生的场地和护栏”。

---

## 四、它们为什么不能互相替代

### 4.1 只有 execution，没有 permission，不等于安全

即使系统运行在容器或 microVM 中，Agent 仍可能：

- 高频执行没有必要的破坏性命令
- 在错误理解任务时持续写坏 workspace
- 调用本不该开放的工具组合
- 访问虽在 sandbox 内但不属于当前任务范围的数据

也就是说，execution layer 可以限制破坏半径，但不自动决定动作是否合理。

### 4.2 只有 permission，没有 execution，也不等于安全

即使所有能力都经过 allowlist、risk tier 或确认 gate，只要动作最终在弱隔离环境中执行，仍可能出现：

- 允许的写入覆盖到宿主机敏感路径
- 允许的联网触达过宽范围
- 允许的子进程产生预期外副作用
- 依赖安装、服务启动或代码执行泄露到任务边界之外

也就是说，permission layer 可以约束“该不该做”，但不自动解决“做了以后会扩散到哪里”。

### 4.3 真实系统通常必须两层叠加

更合理的设计通常是：

- 用 permission layer 控制 capability access
- 用 execution layer 限制 side-effect radius
- 再用 recovery / traceability 补足失败后的可解释性与可恢复性

因此两层不是二选一，而是互补关系。

---

## 五、最常见的边界混淆

### 5.1 Command Restriction

`rm -rf`、`git push`、`pip install` 这类限制常被写成 sandbox 能力，但更核心的问题通常是：是否授权执行。

因此它首先属于 permission layer；至于命令在什么环境里跑、是否能逃逸影响宿主机，才属于 execution layer。

### 5.2 Filesystem Scope

“只能写 workspace，不能写宿主机其他路径”看起来像单纯文件系统隔离，但其实包含两层：

- permission：哪些路径被授权访问
- execution：路径访问最终在什么挂载与隔离模型下落地

缺少任何一层，边界都不完整。

### 5.3 Network Access

是否允许联网、能访问哪些域名，首先是 permission 决策；

真正的网络出口如何被防火墙、容器网络或 remote runtime 限定，则属于 execution 决策。

### 5.4 Confirmation Gate

用户确认常被误认为只是交互体验细节，但它本质上是 permission / governance 层的一部分。

它并不回答代码最终在哪里运行，因此不能替代 execution isolation。

### 5.5 Tool Wrapper

某些系统把安全检查直接写进 tool wrapper，容易产生错觉：好像只要包装工具就完成了环境控制。

但 wrapper 最多解决部分 permission interception；真正的进程隔离、资源限制与宿主机边界仍属于 execution layer。

---

## 六、如何判断某个问题属于哪一层

一个实用判断方法是问两句话：

- **如果动作本身就不应该发生，这是谁来拦？**
- **如果动作已经发生并且出错，副作用由谁来限域？**
- **如果会话可以恢复，但文件系统状态未必能独立重建，这属于哪一层的能力？**

通常：

- 前一个问题更偏 permission layer
- 后一个问题更偏 execution layer
- 第三个问题更偏 recovery / traceability，但如果要判断文件状态是否真能独立恢复，还需要继续下沉到 workspace lifecycle / checkpoint 语义，而不能停留在“已经支持 resume”的表面描述

再细一点看：

- “是否允许” → permission
- “在哪里执行” → execution
- “出了事如何回退与解释” → recovery / traceability
- “恢复的是连接、执行身份，还是完整文件状态” → recovery / traceability 与 workspace lifecycle 的交叉边界
- “什么时候自动做、什么时候停下确认” → autonomy governance

这个判断方法有助于避免把所有安全问题都堆进一个“sandbox”词里。

---

## 七、最重要的结构性矛盾

### 7.1 Capability Control vs Execution Friction

权限控制越细，越可能防止越权与误动作；但策略系统本身也会增加使用摩擦、误拦截和维护成本。

### 7.2 Strong Isolation vs Performance Overhead

执行隔离越强，副作用半径越小；但启动延迟、资源消耗、实现复杂度和调试成本通常越高。

### 7.3 Static Policy vs Dynamic Runtime Context

静态权限更好审计，动态上下文更贴近真实任务；但一旦两者深度耦合，permission 与 execution 的边界也更容易变模糊。

### 7.4 Safe Defaults vs Flexible Workflows

默认安全配置通常要求 permission 更保守、execution 更限域；但开放工作流又要求系统在不同任务中支持逐步放权与执行升级。

---

## 八、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- container / VM 隔离强，就不需要 permission layer
- command allowlist 已经足够代表 execution control
- 用户确认就是完整安全边界
- network restriction 只属于 runtime 层
- 只要系统支持 resume / reconnect，就不必再区分 recovery 语义与文件系统恢复语义
- 这两层完全可以合并成一个统一实现而不损失解释力

这些说法都会把环境层重新压扁成单轴安全模型，失去 Agent-specific environment 的分析价值。

---

## 九、与其他专题的关系

- 与 `permission-policy.md`：本文件进一步细化 permission layer 与 execution layer 的边界，帮助避免把权限策略误写成运行时隔离。
- 与 `sandbox-layers.md`：本文件可以视为对其中 `capability / permission layer` 与 `execution isolation layer` 的专题展开。
- 与 `../overview.md`：对应其中“`在哪里执行` vs `允许做什么`”这一核心结构性矛盾。
- 与 `../conflict.md`：也服务于 `Docker Sandbox 是否足以保证 Agent 安全` 与 `Environment 是否等于 Execution Container` 这两条上游冲突。
- 与 `../code-execution-environments/`：当问题转向 workspace、副作用范围与恢复路径时，应继续下沉到代码执行环境专题，而不是停留在权限边界抽象层。
- 与 `autonomy-vs-confirmation.md`：当问题转向“什么时候自动做、什么时候停下确认”时，应进入 governance 维度，而不是继续混在 permission / execution 的二分里。

---

## 十、当前最值得继续补证的方向

- 主流 agent 产品如何在 policy interception 与 runtime isolation 之间切分职责
- command、filesystem、network 三类限制中，哪些更适合放在 permission，哪些必须落到 execution
- permission layer 与 execution layer 是否存在稳定的统一对象模型，还是应保持松耦合
- default-safe 模式下的逐级放权，如何同时映射 permission profile 与 execution upgrade
- tool wrapper、policy engine、remote runtime gateway 三者之间的边界该如何表达
