# Human-in-the-Loop Patterns

> 适用范围：Agent 系统中人类在规划、执行、审核、纠偏与接管环节的介入模式
> 阶段状态：研究主干，持续补充产品案例与模式比较证据
> 使用说明：本文件回答“human-in-the-loop 到底有哪些常见模式、它们分别解决什么问题、核心 trade-off 是什么”，不直接把任何一种介入方式写成唯一正确答案

## 一、定位

当 Agent 进入真实任务执行时，`human-in-the-loop` 不应被简单理解为“中间有人点一下确认按钮”。

更准确地说，它描述的是：**人在 Agent 执行闭环中的介入位置、决策角色与控制语义**。

这里最关键的不是“有没有人类参与”，而是：

- 人类在什么时刻进入回路
- 人类是做批准、纠偏、审查、选择，还是直接接管
- 人类介入后，任务如何继续推进或回退
- 介入点是安全结构的一部分，还是系统不成熟时的补丁式兜底

因此，human-in-the-loop 是人机协作中的结构性设计问题，而不是单一交互细节。

---

## 二、为什么它必须单列成主题

### 2.1 “有人参与”并不能说明介入模式已经清楚

很多系统都说自己支持 human-in-the-loop，但这句话通常没有说明：

- 是前置审批，还是关键节点确认
- 是失败后接管，还是执行中纠偏
- 是每一步都允许人工改写，还是只在高风险点介入

如果不把介入模式拆清，human-in-the-loop 很容易变成空泛标签。

### 2.2 它会直接塑造 autonomy 与 safety 的边界

人类介入点越靠前，系统越像半自动工具；介入点越靠后，系统越像高自治 agent。

这意味着 human-in-the-loop 不是附属功能，而是系统自治哲学的一部分。

### 2.3 它与 trust、delegation、UI 都高度耦合

人工介入不是孤立事件，而会反过来影响：

- 用户是否愿意委托任务
- 用户如何理解 Agent 当前状态
- 用户是否相信系统知道自己何时该停下
- 系统能否在不中断体验的前提下表达风险

因此它必须在 `04-human-agent-interaction/` 中单列，而不能只散落在 UI 或安全主题里。

---

## 三、常见 human-in-the-loop 模式

human-in-the-loop 可以先按两个维度拆开：

| 维度 | 关注问题 | 常见取值 |
|---|---|---|
| 介入时机 | 人类什么时候进入回路 | 执行前、执行中、执行后、失败后 |
| 介入语义 | 人类进入后实际做什么 | 审批、选择、纠偏、review、validation、接管、fallback |

同一个产品动作可能同时包含多个语义。例如“提交前确认”可能只是审批，也可能包含结果 review；“失败后交给用户”可能是 takeover，也可能只是请求补充上下文。因此下面的模式不是互斥类别，而是分析 human-in-the-loop 设计时常见的组合单元。

### 3.1 Pre-Execution Approval

在 Agent 执行高风险操作前，先请求人类批准。这个模式可以是全量确认，也可以是风险分级确认：例如 OpenHands SDK 将 action control 拆成 confirmation policy 与 security analyzer，支持 `AlwaysConfirm`、`NeverConfirm` 和 `ConfirmRisky`，其中 `ConfirmRisky` 会在动作被评估为高风险时要求用户批准。

典型场景：

- 写文件、删文件、发请求、执行命令前确认
- 发起外部副作用前的权限闸门
- 初次委托高风险任务时的总授权确认

优点：

- 边界明确
- 安全感更强
- 易于和 permission model 结合

代价：

- 容易打断流畅执行
- 确认过多会让用户疲劳
- 对长链任务扩展性较差

### 3.2 Step-Level Confirmation

在关键步骤、阶段切换或高不确定节点请求确认，而不是每次动作都确认。

典型场景：

- 计划确认后再执行
- 修复方案确定后再改代码
- 结果摘要与下一步策略切换时确认

优点：

- 比逐动作确认更流畅
- 更贴近任务语义
- 便于让用户保持高层控制权

代价：

- 关键步骤怎么定义并不总是清楚
- 阶段内部的局部风险可能被忽略
- 若状态展示不足，用户很难做有效判断

Copilot Edits 这类多文件编辑体验提供了另一种 step-level / post-hoc 组合：系统先生成建议变更，再通过文件级、代码块级和多文件摘要 diff 让用户接受、拒绝或重置。这说明 step confirmation 不一定只发生在执行前，也可以发生在建议变更被应用或保留的过程中。

### 3.3 In-Flight Correction

任务执行中允许人类对 Agent 当前方向、约束、优先级或局部动作做纠偏，而不完全中止任务。

典型场景：

- 修改目标范围
- 修正误读需求
- 阻止某个偏离方向的尝试
- 临时增加/删除限制条件

优点：

- 更贴近真实协作过程
- 能避免错误路径继续扩大
- 让用户感觉“还能把方向盘拿回来”

代价：

- 对状态可见性要求高
- 需要系统支持局部更新而不是全量重来
- 若控制语义不清，可能造成上下文混乱

### 3.4 Post-Hoc Review and Acceptance

Agent 先完成一轮执行，再由人类审核结果、决定接受、退回或追加修改。这里需要区分两种动作：**review** 主要判断过程、范围和风险是否偏离预期；**validation** 主要判断结果是否满足完成标准。二者经常连续发生，但不应混写成同一种“审核”。

典型场景：

- 代码 patch review
- 结果报告验收
- 多方案比较后选择
- PR、文档、分析结论的人工签收

优点：

- 不打断中间执行流
- 适合批处理或异步协作
- 更接近交付物导向的工作流

代价：

- 错误可能已经积累很深
- 审核成本可能转移到末端集中爆发
- 对可追溯性要求更高

### 3.5 Failure Recovery Handoff

当 Agent 遇到失败、超时、冲突、不确定状态时，把任务交给人类接管或共同恢复。它不等同于 full takeover：handoff 的重点是把现场状态、已尝试路径、失败原因和可选下一步交出来，使任务仍可恢复；full takeover 则意味着控制权主要转回人类。`Handoff Debt` 论文把 agent 接手中断任务时的重新理解成本称为 handoff debt，并在受控 coding-agent 任务中比较 repo state、raw trace、summary notes 和 structured notes 等交接视图，说明结构化上下文会影响后继 agent 的恢复成本。

典型场景：

- 连续尝试失败后请求人类判断
- 工具或环境异常时请求人工修复
- 模型不确定度过高时中止并上交

优点：

- 能把 human-in-the-loop 作为真正的 recovery 结构
- 避免系统在低把握状态下继续犯错
- 更符合高风险场景中的责任分配逻辑

代价：

- 需要系统知道自己什么时候该上交
- 接管信息若不完整，人类仍需从头理解现场
- 可能暴露出上游 traceability 和 state handoff 的缺陷

### 3.6 Full Takeover

人类在某一时刻完全接管任务，Agent 暂停或转为辅助角色。

典型场景：

- 任务目标发生根本变化
- 高风险操作必须由人亲自完成
- 系统连续偏航，继续自治已无意义

优点：

- 控制权最清晰
- 适合高风险或复杂判断场景
- 责任边界明确

代价：

- 任务流会被中断
- 如果没有良好的 handoff，接管成本很高
- 容易让 agent 退化成低价值辅助工具

---

## 四、如何比较这些模式

一个实用的比较方式，是先把“人类介入”拆成更细的控制语义：

| 控制语义 | 主要问题 | 更适合的阶段 | 主要风险 |
|---|---|---|---|
| approval | 是否允许继续 | 执行前 / 高风险动作前 | 过多确认导致疲劳 |
| correction | 是否需要改方向 | 执行中 | 状态语义变乱 |
| review | 过程和改动是否可接受 | 执行中 / 执行后 | 发现问题太晚 |
| validation | 是否满足完成标准 | 执行后 | 只看结果而忽略过程风险 |
| takeover | 是否由人类接回控制权 | 失败后 / 高风险节点 | 接管信息不足 |
| fallback | 是否切换到更保守路径 | 失败后 / 不确定状态 | 触发条件不清 |

这张表的作用不是定义标准 taxonomy，而是避免把 approval、review、validation 和 takeover 都压缩成“人工确认”。

### 4.1 介入时机

第一条分界，不是 UI 长什么样，而是人类何时进入：

- 执行前
- 执行中
- 执行后
- 失败后

介入时机不同，对安全、流畅性和认知负担的影响完全不同。

### 4.2 介入语义

人类的作用可以很不一样：

- 批准
- 选择
- 修改
- 审查
- 接管

如果不区分这一层，就很难比较不同系统到底在提供什么样的人类控制。

### 4.3 恢复路径

有些模式更偏 prevention，例如 pre-execution approval；有些更偏 recovery，例如 failure handoff。

这决定它们到底是在减少错误发生，还是在错误发生后控制损失。

### 4.4 对系统能力的要求

不同模式对系统提出的要求不同：

- 审批模式更依赖 permission 和 risk classification
- 纠偏模式更依赖状态可见性与局部可更新性
- 接管模式更依赖 traceability、checkpoint 与 handoff quality

所以 human-in-the-loop 不能脱离系统整体能力单独讨论。

---

## 五、最重要的结构性矛盾

### 5.1 Safety Assurance vs Execution Flow

介入点越多，安全感越强；但执行流越容易被打断。

### 5.2 Human Oversight vs Human Bottleneck

人工监督能提升可控性，但也会让系统扩展性受限，特别是在长链任务和高频任务中。

### 5.3 Correctability vs State Coherence

允许执行中纠偏可以减少偏航，但如果状态管理不清晰，也容易让任务语义越来越混乱。

### 5.4 Takeover Readiness vs Interaction Overhead

为了让人类随时能接管，系统必须持续维护更强的可见性与 handoff 信息；但这也会带来额外的记录和展示成本。

---

## 六、最容易被误写成定论的问题

当前阶段尤其要避免以下误解：

- human-in-the-loop 就等于人工确认弹窗
- approval、review、validation 和 takeover 可以混写成同一种“审核”
- 每一步都确认一定更安全
- 后验 review 足以替代中间控制
- validation 只需要看最终结果，不需要看验收证据来源
- 只要支持人工接管，就说明系统已经可控
- fallback 只是失败后重试，不涉及控制权和责任边界
- 有人参与的问题主要是 UI 问题，不涉及系统架构

这些说法都会把本来关于控制关系与恢复结构的问题，误写成交互小功能。

---

## 七、与其他专题的关系

- 与 `../overview.md`：本文件把其中 human-in-the-loop 这一核心视角下沉为具体介入模式与比较框架。
- 与 `../backlog.md`：对应 `Human-in-the-Loop 模式分类` 这一 P0 缺口。
- 与 `../delegation-and-control/`：delegation 决定人类原本交出了什么控制权，本文件进一步讨论这些控制权如何在执行中被收回或介入。
- 与 `../interaction-surfaces/`：介入模式能否成立，很大程度取决于状态是否被清楚展示给用户。
- 与 `../trust-and-alignment/`：系统是否知道何时该停、何时该问、何时该上交，会直接影响用户信任结构。
- 与 `../../05-environments/`：接管、恢复与审批 often 依赖权限控制、traceability 与 recovery 机制支撑。

---

## Evidence

- Status: `Observed / Inferred`
- Sources: OpenHands `Security & Action Confirmation` 官方文档、Microsoft Learn `GitHub Copilot Edits` 官方文档、`Handoff Debt: The Rediscovery Cost When Coding Agents Take Over Interrupted Tasks`、`agentic/temp/web-search/16.md`、`agentic/temp/web-search/17.md`
- Trace: 本次仅回填已通过官方文档或 arXiv 页面初步核验的材料：OpenHands confirmation policy / security analyzer、Copilot Edits inline review / reset 机制，以及 Handoff Debt 对 agent-to-agent 交接恢复成本的定义；第三方在线调研中的其他博客和工具材料仍作为上游线索，不直接写成主线定论
- Needs: 继续核验不同 coding-agent 产品中的 approval、review、validation、handoff、takeover 与 fallback 组合方式；Handoff Debt 目前只支持 agent-to-agent handoff 成本，不直接支持 human takeover 结论

---

## 八、当前最值得继续补证的方向

- 不同产品中 pre-approval、step confirmation、handoff、takeover 的真实组合模式
- human-in-the-loop 介入点与任务类型、风险等级之间是否存在稳定映射
- approval、review、validation、takeover、fallback 在真实 coding-agent 工作流中的组合边界
- 执行中纠偏对状态一致性与任务恢复能力的要求
- full takeover 与 partial correction 在长任务中的维护成本差异
- human-in-the-loop 是否应进一步拆成 approval、correction、review、validation、handoff、fallback 等独立专题
