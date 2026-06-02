# Safety Composition

> 适用范围：Agent 环境中 permission、execution isolation、recovery、governance 如何组合形成最小安全结构
> 阶段状态：研究主干，持续补充产品案例与安全组合证据
> 使用说明：本文件回答“为什么 Agent 安全不能只靠 Docker sandbox 或单一策略层、最小安全组合通常包含哪些维度、这些维度如何互补、核心 trade-off 是什么”，不直接预设某一种具体产品组合为标准答案

## 一、定位

如果说 `permission-policy.md` 讨论的是行动授权，`permission-vs-execution-boundary.md` 讨论的是 permission 与 execution 的职责划分，`sandbox-layers.md` 讨论的是多层 sandbox 结构，那么本文件进一步讨论的是：**这些层在真实系统里应如何组合，才能形成一个对 Agent 有意义的最小安全结构**。

在 Agent 环境里，安全不应被狭义理解为“用了 Docker”或“做了 allowlist”。

更准确地说，Agent 安全通常来自多个维度的叠加：

- 哪些动作被允许发生
- 动作实际在哪里执行、破坏半径到哪里
- 出了问题后是否可追溯、可恢复
- 系统默认是持续自治，还是在关键节点停下来请求人类确认

因此，safety composition 讨论的不是某个单点机制强不强，而是：**不同层如何互补，才能避免系统在某一维过强、另一维完全空缺**。

---

## 二、为什么它必须单列成主题

### 2.1 单点安全机制很容易制造“已经足够安全”的错觉

真实系统里最常见的误判之一是：

- 有 container / VM，就觉得安全问题已经基本解决
- 有 command allowlist，就觉得高风险动作已经被充分约束
- 有人工确认，就觉得系统不再需要 recovery 与 audit

但这些都只覆盖了安全结构中的单一维度。

如果不把“安全组合”单列出来，文档很容易反复在单点机制之间摇摆，而看不到它们为什么必须一起工作。

### 2.2 Agent 风险天然是组合风险，而不是单点风险

Agent 的风险往往不是一个动作孤立造成，而是多个环节共同放大的，例如：

- 权限放得过宽 + 隔离过弱
- 默认自治 + 缺少回滚
- 有回滚 + 但缺少 traceability，导致恢复后仍无法判断安全基线
- 有隔离 + 但缺少治理层，导致高风险动作在错误上下文中持续执行

这说明 Agent 安全更像一个组合系统，而不是安全功能清单。

### 2.3 一旦缺少组合视角，环境层会重新碎片化

当前 `05-environments/` 已经拆出了 permission、execution、workspace、checkpoint、traceability、rollback、autonomy 等专题。

如果没有一个“安全组合”主题把它们重新拉回同一框架，后续就很容易再次碎片化成：

- 一篇讲权限
- 一篇讲容器
- 一篇讲回滚
- 一篇讲确认

但不知道这些部件为什么必须协同。

---

## 三、最小安全组合通常包含哪些层

这里给出的不是唯一标准，而是当前更适合 Agent 环境分析的工作性最小组合。

### 3.1 Permission / Capability Control

这一层回答：**哪些动作被允许发生**。

它通常包括：

- tool allowlist / denylist
- command restrictions
- filesystem scope
- network scope
- escalation / confirmation gate

如果没有这一层，系统很难阻止“本不应该发生”的动作。

### 3.2 Execution Isolation

这一层回答：**即使动作发生了，破坏半径能被限制到哪里**。

它通常包括：

- subprocess / venv / container / microVM / remote runtime
- 文件系统挂载边界
- 资源限制
- 网络出口控制
- 宿主机与子进程边界

如果没有这一层，允许动作一旦出错，副作用就可能扩散到任务边界之外。

### 3.3 Recovery / Traceability

这一层回答：**失败后能否解释发生了什么，并恢复到可继续状态**。

它通常包括：

- checkpoint / snapshot
- rollback / replay / reconstruction
- workspace traceability
- artifact attribution
- audit trail

如果没有这一层，系统即使前面两层都存在，仍可能在出错后无法复盘、无法恢复、无法建立可信自治。

### 3.4 Governance / Mode Control

这一层回答：**系统默认何时自动做、何时停下来请求人类确认**。

它通常包括：

- headless autonomy
- interactive confirmation
- staged authorization
- mode switching
- rollback-backed autonomy

如果没有这一层，permission 和 execution 即使都存在，也未必能解释系统在真实任务里为什么会在某个时刻停下来或继续执行。

---

## 四、这些层为什么互补而不是可替代

### 4.1 Permission 解决“该不该做”

它主要防止：

- 越权动作
- 与当前任务无关的高风险操作
- 工具组合放大能力边界

但它不自动解决：动作一旦出错，副作用会扩散到哪里。

### 4.2 Execution 解决“做坏了会坏到哪里”

它主要防止：

- 写入、进程、联网、副作用扩散到宿主机或任务边界之外
- 单次失败演化成宿主环境层面的事故

但它不自动判断：当前动作是否本来就不该被允许。

### 4.3 Recovery / Traceability 解决“出了问题之后还能不能收回来、讲清楚”

它主要防止：

- 状态污染后无法恢复
- 多次尝试之后无法区分安全基线
- 失败发生后无法解释来源与责任

但它不自动阻止错误动作最初发生。

### 4.4 Governance 解决“默认工作方式是什么”

它主要防止：

- 高风险动作在错误时机被无确认自动执行
- 交互式系统与后台自治系统采用同一风险假设

但它不自动替代 permission、execution 或 recovery 的具体机制。

因此更合理的理解是：

- permission 决定可不可以做
- execution 决定做了后破坏边界在哪里
- recovery 决定出了问题是否能解释和恢复
- governance 决定系统默认如何穿越这些边界

---

## 五、几种典型但不完整的安全组合

### 5.1 Isolation-Heavy, Policy-Light

特点：

- 强调 container / VM / remote runtime
- 对 capability control 与授权边界关注不足

风险：

- Agent 仍可能在隔离环境中高频执行不必要动作
- 误动作被 containment 限住，但不代表行为本身合理

### 5.2 Policy-Heavy, Isolation-Light

特点：

- 强调 allowlist、risk tier、确认 gate
- 底层执行环境较弱或接近宿主机

风险：

- 被允许的动作一旦出错，副作用可能直接外溢
- 对依赖安装、服务启动、代码执行这类动作更脆弱

### 5.3 Recovery-Backed Autonomy

特点：

- 倾向默认自治
- 依赖 rollback、traceability、audit 来兜底

风险：

- 如果 recovery 能力不够强，系统会把“可回滚”误写成“可以放心放权”
- 没有稳定 traceability 时，回滚后仍难判断哪条路径安全

### 5.4 Human-Gated Safety

特点：

- 通过确认 gate 把高风险动作交回给人类
- 更适合生产变更、凭证使用、数据外发等场景

风险：

- 连续自治能力被打断
- 如果没有稳定默认策略，确认次数会迅速膨胀成高摩擦系统

### 5.5 Layered Hybrid Model

特点：

- 用 permission 限制 capability
- 用 execution 限制 blast radius
- 用 recovery / traceability 提供失败后兜底
- 用 governance 决定默认自治还是默认确认

优势：

- 更符合真实 Agent 系统的多维风险结构
- 更利于针对不同任务切换安全姿态

代价：

- 设计与实现复杂度显著更高
- 更需要清晰边界，否则会迅速变成“所有东西都耦合在一起”的系统

---

## 六、如何判断一个系统的安全组合是否失衡

可以从几个问题快速判断：

- 它是否只能回答“在哪里执行”，却回答不了“为什么允许执行”
- 它是否只能回答“有没有确认”，却回答不了“默认失败后如何恢复”
- 它是否有 rollback，却没有稳定 traceability 去判断恢复点是否可信
- 它是否把大量治理逻辑塞进 tool wrapper，而没有更稳定的层次结构
- 它是否在低风险任务上也要求高摩擦确认，导致安全与可用性同时失衡

如果这些问题中有多个回答为“是”，系统往往说明某一层过强、某一层过弱，安全组合已经失衡。

---

## 七、最重要的结构性矛盾

### 7.1 Safe Defaults vs Workflow Fluidity

默认越安全，越可能增加确认、限域和隔离成本；但默认越开放，越容易把风险前移到用户和恢复机制身上。

### 7.2 Strong Containment vs Practical Throughput

更强隔离和更细控制通常意味着更高启动延迟、更重资源开销和更复杂的工程维护。

### 7.3 Preventive Control vs Recoverable Autonomy

系统越强调预防性拦截，越依赖 permission 与确认；系统越强调自治连续性，越依赖 recovery、traceability 和 audit 兜底。

### 7.4 Unified Safety Story vs Layer Coupling

越希望讲出一套统一安全故事，就越需要把不同层连接起来；但连接过度，又容易导致层次模糊、职责混写与实现耦合过深。

---

## 八、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- 有 Docker / VM 就已经具备主要安全性
- 只要有 allowlist，就不需要更强 execution isolation
- 只要有 rollback，就可以默认高自治
- 人工确认可以替代 recovery / traceability
- 所有 Agent 系统都需要同样厚的安全组合

这些说法都会把组合问题重新压扁成单点判断，掩盖不同任务、不同风险等级与不同产品模式下的真实差异。

---

## 九、与其他专题的关系

- 与 `permission-policy.md`：本文件把 permission 当成安全组合中的一层，而不是全部安全设计。
- 与 `permission-vs-execution-boundary.md`：本文件在其边界划分之上，继续讨论这些层如何与 recovery / governance 一起形成最小组合。
- 与 `sandbox-layers.md`：本文件可以视为其四层结构在“最小可用安全组合”视角下的专题展开。
- 与 `autonomy-vs-confirmation.md`：本文件把它放进 governance 维度，而不是把确认机制孤立为单纯交互问题。
- 与 `../code-execution-environments/rollback-recovery-design-paths.md`：当安全策略转向“默认自治 + 可恢复兜底”时，需要依赖 recovery 侧主干专题。
- 与 `../conflict.md`：对应 `Docker Sandbox 是否足以保证 Agent 安全` 这一条目，并为后续比较最小安全组合提供主干支撑。
- 与 `../overview.md`：本文件可作为后续回填 overview 中“最小安全组合”总结段的主要来源。

---

## 十、当前最值得继续补证的方向

- 主流 agent 产品是否存在相对稳定的最小安全组合模式，还是高度依赖产品定位与部署场景
- default-safe、human-gated、rollback-backed autonomy 三类安全姿态各自适合哪些任务形态
- permission profile、execution upgrade、recovery policy 与 governance mode 是否能形成统一配置模型
- 最小安全组合在 browser environments 与 code execution environments 中应如何分别调整
- 哪些安全层最适合在 tool wrapper、policy engine、runtime gateway、control plane 中分别落地
