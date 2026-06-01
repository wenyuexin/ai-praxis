# Permission Policy

> 适用范围：Agent 在环境中可被授予、限制、确认与审计的行动权限
> 阶段状态：研究主干，持续补充案例与证据
> 使用说明：本文件回答“权限策略为什么是环境层的一等主题、它与执行沙箱有什么边界、常见策略轴和核心矛盾是什么”，不直接等同于某个具体产品的授权实现

## 一、定位

在 Agent 环境里，permission policy 回答的不是“代码在哪里运行”，而是：**Agent 被允许做什么、在什么条件下做、做完后如何追溯**。

因此它不能被简化为：

- 一个命令白名单
- 一次用户确认弹窗
- 容器或 VM 自带的系统权限

更准确地说，permission policy 是环境层里的**行动授权系统**。它负责把模型能力翻译成受控行动边界，使 Agent 的“可执行能力”不直接等于“理论上能生成的操作”。

---

## 二、为什么它必须单列成主题

### 2.1 执行隔离不能替代行动授权

执行沙箱主要解决：

- 进程在哪里运行
- 文件和网络副作用能扩散到哪里
- 资源消耗如何被限制

permission policy 额外解决：

- 哪类工具、命令、路径、网络目标可被调用
- 哪些操作需要确认、升级授权或直接拒绝
- 高风险动作是否可在当前任务上下文中被解释为合理

如果只写 sandbox，而不单列 permission policy，环境文档很容易退化成 container / VM / remote runtime 的对比，而忽略 agent-specific capability control。

### 2.2 Agent 的风险来自“会行动”，不只是“会运行代码”

传统程序安全讨论常从运行时隔离切入；Agent 系统则额外面临：

- 被诱导执行越权操作
- 在错误理解任务时继续高影响动作
- 工具组合后形成原本未显式暴露的高风险能力
- 在长链任务中把一次误判放大为持续副作用

所以权限策略不是附属配置，而是环境层约束自治行为的核心部件。

---

## 三、它与执行沙箱的边界

### 3.1 一个负责“允许什么”，一个负责“隔离到哪”

可以用下面的方式区分：

- **permission policy**：定义行动授权与能力边界
- **execution sandbox**：定义运行位置、隔离强度与破坏半径

前者偏 capability-level control，后者偏 runtime-level isolation。

两者互相增强，但不能互相替代：

- 只有 sandbox，没有 policy：Agent 仍可能在隔离环境中高频执行不必要或危险操作
- 只有 policy，没有 sandbox：即使授权严格，允许执行的动作一旦出错，副作用仍可能外溢

### 3.2 权限策略通常位于工具调用与环境执行之间

在实际系统里，它常出现在以下边界层：

- 模型决定调用某个工具之后
- 工具真正触发命令、文件写入、网络访问之前
- 执行结果返回模型并进入下一轮循环之前

因此，permission policy 既不是纯提示词规则，也不是单纯 OS 权限，而是介于 agent loop 与 runtime 之间的治理层。

---

## 四、常见权限控制轴

### 4.1 Tool 权限

回答哪些工具可调用、哪些必须禁用、哪些只能在特定任务中开放。

典型形式：

- allowlist / denylist
- task-scoped tool profile
- read-only vs mutating tool split

### 4.2 Command 权限

回答哪些命令可执行、哪些 shell 能力需要拦截、是否允许链式执行或子进程扩散。

这类控制常直接决定：

- 是否允许包管理、测试、构建
- 是否允许删除、移动、批量修改
- 是否允许下载、联网、启动服务

### 4.3 Filesystem 权限

回答 Agent 能读写哪些路径、能否跨 workspace、能否访问敏感目录或凭证文件。

关键不是“是否可写”，而是：

- 写到哪里
- 改动是否局限于 task workspace
- 是否允许覆盖现有文件
- 是否允许访问隐式敏感路径

### 4.4 Network 权限

回答是否允许联网、能访问哪些域名或协议、出站能力是否受限。

这直接影响：

- 外部检索与依赖下载
- 数据外传风险
- 浏览器环境中的跨站操作能力

### 4.5 Escalation / Confirmation 权限

回答哪些动作可自动执行，哪些必须经用户确认或更高等级授权。

这类策略是 interactive safety 的主要抓手，但也最容易与 headless autonomy 发生冲突。

### 4.6 Audit / Trace 权限

回答授权决策是否可追溯、被拒动作是否记录、权限升级是否留下证据链。

没有审计能力的权限系统，很难支持后续复盘、治理和可信自治。

---

## 五、常见策略模式

### 5.1 Static Allowlist

预先定义允许能力，默认拒绝未声明动作。

优点：

- 简单直接
- 可解释性强
- 适合高风险场景

局限：

- 对开放任务适应性差
- 容易因为规则过粗导致能力受限

### 5.2 Risk-Tiered Policy

按风险等级区分动作：

- 低风险：自动放行
- 中风险：记录或限域放行
- 高风险：确认或拒绝

这比单一 allow/deny 更贴近真实 agent 场景，因为许多动作不是“本质危险”，而是“脱离上下文后危险”。

### 5.3 Context-Aware Authorization

根据任务目标、当前工作区状态、最近动作和工具链上下文动态调整权限。

这是更接近 agent-native 的方向，但难点也更明显：

- 策略解释更复杂
- 容易引入误判
- 审计和复现难度更高

### 5.4 Human-in-the-Loop Gate

把高风险决策交给用户确认。

它能降低误操作，但会牺牲自治连续性，因此更适合：

- 生产环境变更
- 凭证使用
- 对外发送或删除数据

---

## 六、最重要的结构性矛盾

### 6.1 Least Privilege vs Task Completion

权限越收紧，越安全；但限制过严会让 Agent 无法完成真实任务，最终迫使系统不断绕过原有边界。

所以“最小权限”不是静态最小，而是**对给定任务足够且不过量**。

### 6.2 Interactive Safety vs Headless Autonomy

高风险动作前确认能显著降低事故，但也会打断自治循环。

这意味着很多系统必须在两种哲学间选边或分层：

- 默认确认，优先避免误操作
- 默认自治，依赖回滚、审计和限域控制补救

### 6.3 Static Rules vs Context Sensitivity

静态规则更稳定、更可审计；上下文感知更灵活、更接近真实任务需求。

但一旦上下文策略过强，权限系统本身也会变成一个需要被验证的智能组件。

### 6.4 Safety Coverage vs Developer Friction

权限检查越细，系统越安全；但开发者、研究者或高级用户越容易感到阻塞。

因此权限策略不能只讨论“能否拦住风险”，还必须讨论：

- 是否让低风险路径保持流畅
- 是否为常见任务提供稳定默认配置
- 是否支持逐级放权而不是一次性全开

---

## 七、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- Docker / VM 隔离已经足够，不需要单独的 permission policy
- 用户确认就是完整的权限系统
- 权限控制只需要 command allowlist
- headless autonomy 与安全确认可以无代价兼得
- 更细粒度的权限策略一定更优

这些更适合继续放在 `../conflict.md` 或与恢复/自治相关专题联动核验。

---

## 八、与本目录其他主题的关系

- 与 `../overview.md`：本文件展开其中的 permission layer，不覆盖执行层与恢复层全貌。
- 与 `../conflict.md`：Docker safety、environment 定义和 autonomy vs safety 的冲突是本文件的重要上游输入。
- 与 `README.md`：本文件属于 sandboxing-and-safety 目录下的主干专题之一。
- 与后续 `autonomy-vs-confirmation.md`：后者可进一步展开确认机制与自治连续性的冲突。
- 与后续 `sandbox-layers.md`：后者应把 permission、execution、recovery 三层关系系统化。

---

## 九、当前最值得继续补证的方向

- permission policy 在研究型系统与产品型系统中的实现差异
- command / file / network / credential 四类权限是否需要统一抽象
- 动态 capability caps 是否已形成稳定工程模式
- 默认自治系统如何通过 rollback 与 audit 抵消部分确认需求
- policy interception 与 tool wrapper 的职责边界
