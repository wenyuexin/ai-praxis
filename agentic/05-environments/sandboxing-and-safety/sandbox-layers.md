# Sandbox Layers

> 适用范围：Agent 环境中 sandbox 不同层次的作用边界、相互关系与常见混淆
> 阶段状态：研究主干，持续补充实现案例与恢复机制证据
> 使用说明：本文件回答“为什么 sandbox 不能只理解为 container / VM、环境层至少应拆成哪些正交维度、这些层如何共同支撑 agent 安全与自治”，不直接等同于某一种具体隔离技术选型

## 一、定位

在 Agent 语境里，sandbox 不应被简化为“把代码放进容器里跑”。

更准确地说，sandbox 是一组**共同限制 agent 行动范围、执行位置与副作用传播方式**的环境层机制。

如果只把 sandbox 理解为 execution container，就会遗漏至少三类关键问题：

- Agent 被允许做什么
- 动作产生的状态如何追踪与恢复
- 自治执行与人工确认如何协调

因此，`sandbox` 更适合作为一个多层系统理解，而不是单一 runtime 组件。

---

## 二、为什么需要“分层”而不是“单点定义”

### 2.1 传统隔离讨论只覆盖了一部分问题

container、microVM、remote runtime 主要回答：

- 代码在哪里运行
- 本地副作用会扩散到哪里
- 资源消耗如何被约束

但 agent 系统还必须额外回答：

- 哪些动作在当前任务中被授权
- 哪些风险动作需要确认、拦截或限域
- 执行后出了问题如何回滚与审计

这说明单靠 execution isolation 不能完整描述 agent sandbox。

### 2.2 Agent 风险是多轴的，不是单轴的

一个系统即使运行在很强的隔离环境里，仍可能：

- 执行本不该执行的命令
- 访问不该访问的路径
- 在误解任务时持续放大副作用
- 在长链任务里留下难以回溯的状态污染

因此 sandbox 既要讨论 containment，也要讨论 capability、recovery 与 governance。

### 2.3 分层后才能避免术语混乱

如果不拆层，以下概念很容易被混写：

- sandbox
- permission policy
- workspace
- checkpoint
- confirmation gate
- audit trail

一旦混写，目录结构和后续专题都很难稳定。

---

## 三、建议采用的四层理解方式

这里给出的不是唯一理论标准，而是当前更适合 `05-environments/` 组织知识的工作性拆分。

### 3.1 Capability / Permission Layer

这一层回答：**Agent 被允许做什么**。

典型对象包括：

- tool allowlist / denylist
- command restrictions
- filesystem scope
- network scope
- confirmation / escalation gate
- risk-tiered authorization

这一层决定的是 capability boundary，而不是运行位置。

它主要防止的是：

- 越权动作
- 不必要的高风险操作
- 工具组合带来的能力放大

这一层对应 `permission-policy.md` 的主干内容。

### 3.2 Execution Isolation Layer

这一层回答：**Agent 实际在哪里执行，破坏半径被限制到哪里**。

典型对象包括：

- local subprocess
- venv
- container
- microVM
- remote runtime
- browser sandbox

这一层主要关注：

- 隔离强度
- 启动延迟
- 资源模型
- 文件和进程副作用边界

它解决的是 runtime containment，而不是 capability authorization。

### 3.3 State / Recovery Layer

这一层回答：**Agent 执行后留下的状态如何被保存、观察与恢复**。

典型对象包括：

- checkpoint
- snapshot
- rollback
- redo / replay log
- workspace artifact tracking
- traceability

没有这一层，系统即使“拦得住一部分风险”，也难以支持：

这里还需要补一条边界提醒：State / Recovery layer 可以稳定讨论 conversation restore、runtime resume、sandbox reuse、traceability 与 recovery hooks，但不能在证据不足时把这些能力直接写成“workspace 文件系统可独立 checkpoint / replay / restore”。恢复连接、恢复执行身份与恢复文件状态是相关但不同的三层语义。

- 长时任务
- 自动修改
- 失败恢复
- 责任归因

它把 sandbox 从“静态隔离”推进到“可治理执行”。

### 3.4 Autonomy Governance Layer

这一层回答：**系统默认是持续自治，还是在关键节点停下来请求人类确认**。

典型对象包括：

- headless autonomy
- interactive confirmation
- pre-authorized capability window
- mode switching
- rollback-backed autonomy

这一层不是单纯 UX，而是决定 agent loop 是否连续、风险由谁承担、系统如何在安全与自治间折中。

它与 permission layer 有交叉，但不完全相同：

- permission layer 定义“可不可以做”
- autonomy governance 定义“何时自动做、何时停下来”

---

## 四、四层之间是什么关系

### 4.1 它们不是串行替代关系，而是叠加关系

更合适的理解是：

- permission 决定能力边界
- execution 决定运行位置与破坏半径
- recovery 决定失败后是否可追溯与可恢复
- autonomy governance 决定执行链是否连续以及安全责任如何分配

任何一层缺失，系统都会出现明显盲区。

### 4.2 只强化某一层并不能替代其他层

典型误区包括：

- 隔离够强，就不需要 permission policy
- 有用户确认，就不需要 rollback
- 有 rollback，就可以默认完全自治
- 有 command allowlist，就不需要 filesystem / network scope

这些都把本应正交的层错误地当成替代品。

### 4.3 实际系统通常是多层混合，而不是纯粹单层

例如一个看似“只是 Docker sandbox”的系统，实际往往还隐含：

- 路径级写入限制
- 联网限制
- 高风险命令拦截
- 操作日志与轨迹
- 人工确认节点

因此，文档组织不能沿用“容器类型 = sandbox 全部内容”的写法。

---

## 五、每层分别解决什么，不解决什么

### 5.1 Permission Layer

解决：

- 哪些动作被授权
- 哪些风险必须拦截或确认
- 权限是否按任务、路径、工具、网络分层

不解决：

- 代码逃逸后落在哪个系统边界
- 状态恢复如何落地

### 5.2 Execution Isolation Layer

解决：

- 运行位置
- 资源隔离
- 文件、进程、网络副作用的物理 / 系统边界

不解决：

- 当前动作在语义上是否合理
- 哪些动作该由人确认

### 5.3 State / Recovery Layer

解决：

- 执行历史是否可追踪
- 状态能否回滚
- 长链任务失败后是否能恢复

不解决：

- 本次动作本来是否应该被允许
- 系统默认应不应该停下来等人

### 5.4 Autonomy Governance Layer

解决：

- 默认自治还是默认确认
- 风险动作在什么时候切断 agent loop
- 哪些模式适合后台运行，哪些适合交互协作

不解决：

- 底层隔离强度本身
- checkpoint / snapshot 的具体实现路径

---

## 六、最重要的结构性矛盾

### 6.1 “在哪里执行” vs “允许做什么”

这是 execution layer 与 permission layer 的经典边界。

一类系统过度强调 runtime isolation，另一类系统过度强调 capability control；但二者分别防的是不同类型的失败。

### 6.2 “先拦住风险” vs “先执行再恢复”

这是 autonomy governance 与 recovery layer 的耦合矛盾。

系统越偏向 headless autonomy，就越需要恢复与审计来兜底；系统越偏向逐次确认，就越依赖人工作为第一道风险闸门。

同时还要避免把“可恢复”写得过粗：有些系统实际支持的是会话 / 连接 / runtime identity 的恢复，而不是文件系统状态可独立重建。如果不区分这两类恢复，恢复层就会被误写成比实际证据更强的能力承诺。

### 6.3 隔离强度 vs 执行效率

更强的 execution isolation 往往带来更高启动成本、更复杂资源管理和更重工程负担。

这解释了为什么 sandbox 不是“越强越好”，而是 workload-sensitive trade-off。

### 6.4 策略精细度 vs 系统摩擦

permission 与 governance 层越细，安全越可能提升；但开发、研究和日常使用中的摩擦也会更强。

这要求系统在默认安全与可用性之间做持续平衡。

---

## 七、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- sandbox 就是 container / VM
- environment isolation 足够强就不需要 policy layer
- confirmation gate 属于产品交互，不属于环境设计
- rollback 只是附加能力，不影响自治设计
- 只要支持 conversation restore / runtime resume，就等于完整 workspace 状态可恢复
- sandbox 层次越多，系统就一定越安全

这些判断都会导致 `05-environments/` 再次退化为 runtime 技术清单，而失去 agent-specific environment 的组织价值。

---

## 八、与已有专题的关系

- 与 `../overview.md`：本文件把其中的三层框架进一步展开为更适合 sandbox 语境的四层工作性拆分。
- 与 `permission-policy.md`：本文件把它放入 capability / permission layer，强调其不是执行层替代品。
- 与 `autonomy-vs-confirmation.md`：本文件把它放入 autonomy governance layer，作为横向设计轴而非孤立交互细节。
- 与 `../conflict.md`：Docker safety、environment 定义、rollback 路径和 autonomy tension 都是本文件的上游冲突来源。
- 与后续 workspace / checkpoint / traceability 专题：这些将进一步细化 state / recovery layer。

---

## 九、当前最值得继续补证的方向

- capability control 与 execution isolation 在主流 agent 系统中的职责切分
- transactional snapshot、checkpoint、redo / replay 三类恢复路径如何映射到 recovery layer
- confirmation gate、capability caps 与 mode switching 是否能统一到同一治理框架
- workspace 是否应视为 recovery layer 与 execution layer 的桥梁概念
- 浏览器环境中的 sandbox layers 是否需要与代码执行环境分开建模
