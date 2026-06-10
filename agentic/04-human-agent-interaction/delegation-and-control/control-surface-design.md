# Control Surface Design

> 适用范围：Agent 系统中人类可见、可操作、可收回控制权的控制面设计
> 阶段状态：研究主干，持续补充产品案例与控制面证据
> 使用说明：本文件回答“Agent 系统应该把哪些控制点暴露给人类、这些控制点分别解决什么问题、与 UI 展示和权限系统如何区分”，不直接把某一种产品交互写成通用标准

## 一、定位

在 Agent 系统里，`control surface` 不应被简单理解为“界面上有几个按钮”。

更准确地说，它描述的是：**系统把哪些可控点、停止点、授权点、接管点和恢复路径显式暴露给人类**。

这里最关键的不是控制项越多越好，而是：

- 人类能否理解 Agent 当前被委托了什么
- 哪些动作需要审批，哪些动作可以自动继续
- Agent 什么时候必须停下来询问或上交
- 人类如何纠偏、暂停、接管、回退或要求重新验证
- 控制点是否与权限、风险、任务阶段和完成标准一致

因此，control surface 是 delegation granularity 与 human-in-the-loop 的连接层：delegation 定义控制权交出去多厚，human-in-the-loop 定义控制权如何回到人类，control surface 则定义这些动作如何被系统显式表达和操作。

---

## 二、为什么它必须单列成主题

### 2.1 没有控制面，委托粒度很难被用户理解

用户即使知道自己“委托了任务”，也未必知道 Agent 可以自主做到哪一步。

如果系统没有把计划、权限、预算、停止条件和风险动作表达清楚，用户很容易把 goal-level delegation 误解成全权限授权，或者把一次 approval 误解成后续所有动作都安全。

### 2.2 没有控制面，human-in-the-loop 会退化成临时打断

人工介入如果只表现为临时弹窗，用户很难判断：

- 这次确认是在批准计划，还是批准某个副作用动作
- 这次 review 是检查过程，还是做最终 validation
- 这次上交是请求补充上下文，还是要求人类 takeover

control surface 的作用，是把这些介入语义从临时对话中抽出来，变成稳定可理解的操作面。

### 2.3 控制面不是纯 UI，也不是纯权限系统

UI 负责展示和操作，权限系统负责限制动作能否发生；control surface 关注的是二者之间的控制语义：用户看到什么、能决定什么、什么时候必须做决定、决定后系统如何继续。

因此它应放在 `delegation-and-control/` 下，而不是只放入 `interaction-surfaces/` 或 `05-environments/`。

---

## 三、常见 control surface 组件

control surface 可以先按“授权、约束、观察、介入、恢复”五类拆开：

| 控制组件 | 主要问题 | 常见形态 |
|---|---|---|
| approval point | 哪些节点需要人类批准 | 计划确认、高风险动作确认、阶段切换确认 |
| autonomy budget | Agent 可自主探索多远 | 时间、步骤数、工具次数、成本、文件范围 |
| permission boundary | Agent 被允许做什么 | 读写权限、命令权限、网络权限、发布权限 |
| stop condition | 什么时候必须停下 | 不确定度过高、连续失败、越界、预算耗尽 |
| takeover entry | 人类如何接管 | 暂停、回滚、转人工执行、改写下一步 |
| fallback path | 系统如何转保守路径 | 降级为只读、请求澄清、退回计划阶段 |
| validation gate | 如何证明完成 | 测试、diff、验收清单、人工签收 |

这些组件不一定都需要同时出现。任务越开放、风险越高、执行链越长，对 control surface 的要求通常越高。

### 3.1 Approval Point

approval point 是人类明确允许系统继续的节点。

典型场景：

- 计划执行前确认
- 写文件、删文件、运行命令前确认
- 从分析阶段进入修改阶段前确认
- 提交、发布、外部请求前确认

主要风险：

- approval 语义不清，用户不知道批准的是计划、动作还是结果
- 确认过多导致疲劳
- 一次确认被系统错误扩展为长期授权

### 3.2 Autonomy Budget

autonomy budget 定义 Agent 在不再次询问人类的情况下，可以自主探索多远。

典型维度：

- 可执行步骤数
- 可读取或修改的文件范围
- 可调用工具次数
- 可消耗的时间、token 或费用
- 可尝试的修复次数

主要风险：

- 预算没有被显式表达，用户只能事后发现范围扩大
- 预算只限制成本，不限制副作用
- 系统不知道预算耗尽后该停止、上交还是降级

### 3.3 Permission Boundary

permission boundary 定义 Agent 能做什么、不能做什么。

它和 autonomy budget 不同：budget 关注“做多远”，permission boundary 关注“能不能做”。

典型边界：

- 只读 / 可写
- 本地 / 联网
- 草稿 / 提交
- 低风险命令 / 高风险命令
- 当前目录 / 跨目录

主要风险：

- 用户授权任务目标，却没有授权对应工具动作
- 系统开放工具能力，却没有解释副作用半径
- 权限边界和任务语义不一致，导致 Agent 看似能做但不该做

### 3.4 Stop Condition

stop condition 定义 Agent 什么时候必须停下来。

典型条件：

- 连续失败
- 关键上下文缺失
- 任务范围开始扩大
- 验证无法完成
- 需要修改高风险文件或规则
- 结果依赖未验证推断

主要风险：

- 只设计开始授权，不设计停止条件
- 把“模型自己判断何时该停”当作默认能力
- 停止后缺少 handoff，导致人类接手成本过高

### 3.5 Takeover and Fallback Entry

takeover entry 让人类接回控制权，fallback path 让系统切换到更保守路径。

典型方式：

- 暂停执行并输出当前状态
- 回到计划阶段重新确认
- 降级为只读分析
- 要求人类提供缺失上下文
- 把后续动作拆成更小授权单元

主要风险：

- 接管入口存在，但状态不可理解
- fallback 只是重试，没有改变风险边界
- 人类接管后，系统不知道如何恢复协作

---

## 四、如何比较 control surface 设计

### 4.1 控制点是否对应真实风险

好的控制面不是把所有动作都拦住，而是把控制点放在风险真正变化的位置：权限扩大、任务阶段切换、外部副作用、验证失败和不确定性上升。

### 4.2 控制语义是否清楚

同样是“确认”，语义可能完全不同：

| 确认对象 | 实际语义 |
|---|---|
| 确认计划 | 允许按这个方向执行 |
| 确认工具动作 | 允许产生某个副作用 |
| 确认结果 | 接受当前交付物 |
| 确认风险 | 人类知道风险并选择继续 |

如果系统只展示“确认 / 取消”，但不说明确认对象，就会削弱控制面的可靠性。

### 4.3 控制面是否支持恢复

控制面不仅要能阻止错误，也要能支持错误发生后的恢复：当前状态、已做动作、失败原因、可回退点和推荐下一步都需要被表达出来。

### 4.4 控制面是否增加过多负担

控制面越强，用户越容易掌控系统；但控制项过多也会让用户无法判断重点。

因此控制面需要按任务风险分层：低风险任务可以轻量授权，高风险长任务需要更强的计划、权限、停止和接管结构。

---

## 五、最重要的结构性矛盾

### 5.1 Explicit Control vs Interaction Friction

控制点越显式，边界越清楚；但用户需要处理的确认和判断也越多。

### 5.2 Safety Boundary vs Autonomy Experience

安全边界越强，Agent 越不容易越界；但过强边界也可能让自治体验退化成逐步工具调用。

### 5.3 Recoverability vs Logging Overhead

为了支持接管和恢复，系统需要记录更多状态和轨迹；但记录过多会增加展示噪声和维护成本。

### 5.4 Permission Semantics vs Task Semantics

权限系统通常按动作和资源组织，用户任务通常按目标和阶段组织；二者不一致时，control surface 需要承担翻译成本。

---

## 六、最容易被误写成定论的问题

当前阶段尤其要避免以下误解：

- control surface 就是 UI 按钮设计
- 有 approval popup 就说明控制面完整
- 权限系统可以替代所有人类控制
- stop condition 可以完全交给模型自行判断
- takeover 只需要一个暂停按钮，不需要状态交接
- 控制点越多一定越安全
- validation gate 可以替代执行中的 review 和纠偏

这些说法都会把本来关于控制权表达、风险分层和恢复路径的问题，误写成单点交互功能。

---

## 七、与其他专题的关系

- 与 `delegation-granularity.md`：delegation granularity 定义控制权交出去多厚，本文件讨论这些控制权如何被表达、限制和收回。
- 与 `../human-in-the-loop/human-in-the-loop-patterns.md`：HITL 定义人类介入语义，本文件讨论这些介入语义需要哪些控制点支撑。
- 与 `../interaction-surfaces/`：interaction surface 关注状态与 artifact 如何呈现，本文件关注呈现背后的控制语义。
- 与 `../trust-and-alignment/`：清楚的控制面有助于用户形成校准后的信任，而不是误以为系统比实际更可控。
- 与 `../../05-environments/`：permission boundary、sandbox、traceability 与 recovery 是 control surface 能否落地的环境基础。

---

## 八、当前最值得继续补证的方向

- 不同 coding-agent 产品如何组合 approval point、permission boundary、stop condition 与 takeover entry
- autonomy budget 是否能形成稳定设计语言，而不是只停留在产品内部实现
- control surface 如何随任务风险等级动态增强或降级
- validation gate、review point 与 permission gate 的边界如何区分
- control surface 与 interaction surface 是否应进一步拆成控制语义层和界面呈现层
