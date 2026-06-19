# Delegation Granularity

> 适用范围：Agent 系统中任务委托的粒度、控制边界与 autonomy budget 设计
> 阶段状态：研究主干，持续补充产品案例与控制面证据
> 使用说明：本文件回答“用户到底是在委托目标、步骤、工具权限，还是整个工作流、不同委托粒度分别解决什么问题、核心 trade-off 是什么”，不直接把某一种 delegation 方式写成标准答案

## 一、定位

在 Agent 系统里，`delegation` 不应被简单理解为“把一个任务交给模型去做”。

更准确地说，它描述的是：**用户把多大范围的决策权、执行权和副作用控制权交给 Agent**。

这里最关键的不是“有没有委托”，而是：

- 委托的是结果目标、执行计划、局部动作，还是整条工作流
- Agent 在哪些边界内可以自主决定，哪些边界上必须停下来询问
- 用户授予的是能力集合、预算、时间窗口，还是某种更高层的自治许可
- 当委托失败或偏航时，控制权如何被收回、修正或重新分配

因此，delegation granularity 是 control surface 设计中的基础问题，而不是 prompt 表达风格问题。

---

## 二、为什么它必须单列成主题

### 2.1 “把任务交给 Agent”常常掩盖了真正的控制差异

两个系统都可能说“支持任务委托”，但它们实际委托的可能完全不同：

- 一个系统只接受高层目标，但执行细节全由 Agent 自主决定
- 另一个系统只允许 Agent 在人类预设步骤内推进
- 还有系统允许任务自治，但不允许高风险工具调用自动执行

如果不把 delegation 粒度拆清，很多系统差异会被模糊成一句“更自动”或“更可控”。

### 2.2 它会直接决定 autonomy、safety 与 UX 的平衡点

委托粒度越粗，系统越接近高自治 agent；粒度越细，系统越接近带智能辅助的 workflow tool。

这意味着 delegation 不是附属配置，而是系统产品哲学的一部分。

### 2.3 它与 human-in-the-loop、permission、UI 都高度耦合

委托粒度一旦改变，会连带影响：

- 什么时候需要人工确认
- 哪些风险需要 permission gate 拦截
- 用户界面需要展示多深的计划与状态
- 出错时人类能否快速判断 Agent 是否越界

因此它必须在 `delegation-and-control/` 下单列，而不能只散落在 UI、权限或审批主题里。

---

## 三、常见 delegation 粒度

delegation granularity 可以先按三个维度拆开：

| 维度 | 关注问题 | 常见取值 |
|---|---|---|
| 委托对象 | 用户把什么交给 Agent | 目标、计划、步骤、动作、能力边界、工作流 |
| 自主厚度 | Agent 能独立决定到哪一层 | 路径、步骤、工具、参数、恢复策略 |
| 回收机制 | 人类如何重新进入控制回路 | approval、correction、review、validation、takeover、fallback |

同一个系统可能同时使用多种委托粒度。例如 coding agent 可以接受 goal-level 任务，但对写文件、运行命令、提交结果分别使用 capability-bounded、step-level 和 post-hoc review。因此下面的分类不是互斥架构，而是分析控制边界时的常见组合单元。

### 3.1 Goal-Level Delegation

用户只给出高层目标，Agent 自主决定路径、步骤和局部动作。

典型场景：

- “帮我定位并修复这个 bug”
- “整理这批资料并输出分析摘要”
- “研究这个项目并给出结论”

优点：

- 用户负担最低
- 最接近端到端自治体验
- 更能发挥 Agent 的规划与工具协同能力

代价：

- 偏航风险更高
- 对状态可见性和中途纠偏要求更高
- 容易让用户误判系统真实边界

### 3.2 Plan-Level Delegation

用户把目标交给 Agent，但要求先给出计划，再决定是否批准执行。

典型场景：

- 先看修复方案，再决定是否改代码
- 先看研究步骤，再决定是否展开执行
- 先确认任务分解，再允许进入下游动作

优点：

- 在高层保持人类控制权
- 更利于提前发现偏航
- 比逐动作干预更流畅

代价：

- 计划与实际执行仍可能脱节
- 用户需要具备一定判断计划质量的能力
- 对计划可读性要求更高

### 3.3 Step-Level Delegation

人类把任务拆成若干步骤，Agent 在每个步骤内部自主执行，阶段切换时再由人类确认。

典型场景：

- 检索、分析、执行、验证分步推进
- 代码任务中的“定位 → 修改 → 测试”逐段授权
- 审批流中的阶段性签收

优点：

- 结构清晰
- 更便于局部回退与纠偏
- 适合中高风险任务

代价：

- 用户仍需频繁关注任务状态
- 过细会损失自治效率
- 步骤划分不合理时仍会掩盖局部风险

### 3.4 Action-Level Delegation

用户只允许 Agent 对单个动作或单类动作进行委托，例如一次调用、一次写入、一次查询。

典型场景：

- 只允许执行一个命令
- 只允许生成 patch 草稿
- 只允许查找信息，不允许修改环境

优点：

- 控制最强
- 责任边界清楚
- 适合高风险或低信任阶段

代价：

- 交互负担高
- 很难支持长链任务
- Agent 容易退化为函数调用包装层

### 3.5 Capability-Bounded Delegation

用户不是按任务步骤授权，而是按能力边界授权，例如允许哪些工具、路径、预算、网络范围与副作用。授权研究中的 authenticated delegation 思路也强调：用户可以把权限委托给 agent，但需要通过 agent-specific credentials、delegation tokens、scope limitation 和 audit trail 维持身份、范围和责任链。

典型场景：

- 允许读文件但不允许写文件
- 允许本地分析但不允许联网
- 允许低风险命令但禁止删除或发布动作

优点：

- 更贴近权限模型
- 适合与 sandbox / permission layer 结合
- 比逐动作授权更可扩展

代价：

- 用户理解能力边界本身需要心智成本
- 能力集合未必与任务语义完全对齐
- 细粒度能力组合可能变得复杂难懂

### 3.6 Workflow-Level Delegation

用户把一整条固定工作流交给 Agent 执行，Agent 的自主性主要体现在局部参数、分支选择或恢复处理，而不是重写流程本身。Claude Code named subagents 提供了一个 workflow / role delegation 样本：官方文档允许为子 agent 配置独立 prompt、tools、model、permissionMode、background 与 worktree isolation 等属性，适合表达专门 reviewer、debugger 或 research agent；但这些配置不能直接等同于安全隔离，仍需要核验权限继承与隔离边界。

典型场景：

- 固定 CI/CD 诊断流程
- 固定研究模板或分析模板
- 标准化审批与执行管线

优点：

- 行为可预期
- 更易审计与复现
- 适合组织化落地

代价：

- 灵活性较弱
- 对开放任务适应性不足
- 可能把 Agent 限制成 workflow executor，而非更一般的自治体

---

## 四、如何比较不同 delegation 粒度

一个实用的比较方式，是把 delegation 拆成“交出什么、保留什么、如何收回”：

| 比较维度 | 需要回答的问题 | 容易混淆的点 |
|---|---|---|
| 委托对象 | 交出目标、计划、步骤、动作，还是能力边界 | 把目标委托误写成全权限委托 |
| autonomy budget | Agent 可以自主探索多远 | 把更长执行链误写成更强能力 |
| permission boundary | 哪些动作必须被门控 | 把能力开放误写成任务授权 |
| stop condition | 什么时候必须停下询问或上交 | 只写开始授权，不写终止条件 |
| recovery path | 偏航后如何纠偏、回退或接管 | 把失败重试误写成恢复机制 |

这张表的作用不是定义统一标准，而是避免把 delegation 简化成“任务写得粗一点还是细一点”。

### 4.1 委托对象是什么

第一条分界不是 UI 长什么样，而是用户到底在委托什么：

- 目标
- 计划
- 步骤
- 动作
- 能力边界
- 固定工作流

委托对象不同，控制语义就完全不同。

### 4.2 自主权的厚度

不同粒度意味着不同层级的 autonomy budget：

- goal-level delegation 最厚
- action-level delegation 最薄
- capability-bounded 与 workflow-level 常处于中间，但维度不同

因此比较 delegation 时，不能只说“更自动”或“更保守”，而要看它把自主权放在哪一层。`Hedwig` 这类 dynamic autonomy 原型进一步提示：autonomy budget 未必只能在任务开始前静态设定，也可能随着开发者反馈、任务熟悉度和历史交互逐步调整。

### 4.3 纠偏与恢复方式

有些 delegation 更依赖事前控制，例如 action-level；有些更依赖中途纠偏和失败接管，例如 goal-level。

这决定系统到底在强调 prevention，还是强调 recoverability。

### 4.4 对用户心智模型的要求

有的 delegation 对用户要求更高：

- capability-bounded delegation 要求用户理解权限模型
- plan-level delegation 要求用户理解计划质量
- workflow-level delegation 要求用户理解流程模板与边界

因此 delegation 设计不只是系统问题，也是用户认知负担问题。

### 4.5 与 human-in-the-loop 的对应关系

委托粒度决定了人类介入的默认位置：

| delegation 粒度 | 常见 HITL 介入点 | 主要控制语义 |
|---|---|---|
| goal-level | 计划前、阶段中、交付后 | correction / review / validation |
| plan-level | 计划完成后、执行前 | approval / correction |
| step-level | 阶段切换时 | approval / review |
| action-level | 单次高风险动作前 | approval |
| capability-bounded | 权限越界或预算耗尽时 | fallback / takeover |
| workflow-level | 分支选择、异常恢复、最终签收 | review / fallback / validation |

因此 delegation 与 human-in-the-loop 不是两个独立主题：前者定义控制权交出去的厚度，后者定义控制权如何在执行过程中被检查、修正或收回。

---

## 五、最重要的结构性矛盾

### 5.1 Autonomy Breadth vs User Predictability

委托范围越大，用户负担越小；但系统行为也越难完全预测。

### 5.2 Control Precision vs Interaction Overhead

控制越精细，边界越清楚；但用户干预频率和交互成本也越高。

### 5.3 Capability Governance vs Task Semantics

按能力边界委托更利于治理；但能力集合未必总能自然映射到任务语义，容易让用户“知道能做什么，却不知道该怎么委托”。

### 5.4 Reusability vs Flexibility

workflow-level delegation 更容易复用与标准化；goal-level delegation 更灵活，但也更难审计与复制。

---

## 六、最容易被误写成定论的问题

当前阶段尤其要避免以下误解：

- delegation granularity 只是 prompt 写得细不细
- goal-level delegation 一定更先进
- action-level control 一定更安全
- 授权一个目标就等于授权所有工具和副作用
- capability-bounded delegation 足以替代所有人工确认
- stop condition 可以靠模型自行判断，不需要外部表达
- workflow-level delegation 不算真正的 agent 委托

这些说法都会把本来关于控制边界和自治厚度的问题，误写成偏好判断或营销话术。

---

## 七、与其他专题的关系

- 与 `../overview.md`：本文件把 delegation and control 这一核心视角下沉为委托粒度与控制边界框架。
- 与 `../backlog.md`：对应 `Delegation Granularity and Control Surface` 这一 P0 缺口。
- 与 `../human-in-the-loop/human-in-the-loop-patterns.md`：delegation granularity 决定人类何时、以什么语义重新进入执行回路。
- 与 `../interaction-surfaces/`：不同委托粒度要求不同的任务可见性、计划呈现与中断操作面。
- 与 `../trust-and-alignment/`：用户是否愿意委托更粗粒度任务，与系统能否清楚表达边界和不确定性紧密相关。
- 与 `../../05-environments/`：capability-bounded delegation 与 permission、sandbox、recovery 机制有直接耦合。

---

## Evidence

- Status: `Observed / Inferred`
- Sources: OpenHands `Security & Action Confirmation` 官方文档、Microsoft Learn `GitHub Copilot Edits` 官方文档、Claude Code permission modes / settings / subagents 官方文档、`Authenticated Delegation and Authorized AI Agents`、`Hedwig: Dynamic Autonomy for Coding Agents Under Local Oversight`、Cursor SDK changelog
- Trace: 本次仅回填已初步核验的 delegation / permission 相关材料：OpenHands 作为 risk-based approval 与 capability gate 案例，Copilot Edits 作为建议变更 review / reset 案例，Claude Code 作为 permission modes、allow / ask / deny 规则和 named subagent delegation 案例，Authenticated Delegation 作为 agent-specific credentials、scope limitation 与 audit trail 的框架思路，Hedwig 作为 dynamic autonomy 的研究原型，Cursor SDK 作为嵌套子 agent 与持久化运行状态的产品案例；其他在线调研输入仍待核验
- Needs: 继续比较不同 coding-agent 产品如何组合 goal-level、plan-level、step-level 与 capability-bounded delegation；继续核验 Claude Code subagent 的权限继承和隔离边界；Hedwig 目前只支持研究原型层面的动态自治，不应写成生产最佳实践

---

## 八、当前最值得继续补证的方向

- 不同产品中 goal-level、plan-level、step-level 与 capability-bounded delegation 的真实组合方式
- autonomy budget 是否可以形成更稳定的设计语言，而不是各产品各说各话
- delegation granularity 与任务类型、风险等级之间是否存在稳定映射
- 委托对象、permission boundary、stop condition 与 HITL 介入点之间是否存在稳定组合
- workflow-level delegation 与开放任务委托之间的长期维护成本比较
- delegation granularity 是否应进一步拆成委托对象、权限厚度、纠偏机制、控制权回收四个独立专题
