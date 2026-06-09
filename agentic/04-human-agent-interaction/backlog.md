# Human-Agent Interaction Backlog

> 适用范围：`04-human-agent-interaction/` 的内容缺口、关键问题与待补专题
> 使用原则：本文件记录人机协作主题中已经识别但尚未充分沉淀的高价值问题，不是实现路线图，也不表示相关结论已成为主线定论

## 一、收录规则

适合进入本文件的内容通常满足以下条件之一：

- 能帮助解释人与 Agent 之间的控制边界、信任结构与接管机制
- 当前目录骨架已建立，但缺少足够稳定的正文专题来承接
- 属于不同产品和论文之间常见但尚未收敛的人机协作矛盾
- 不宜直接写进 `overview.md` 的稳定结论，但值得长期跟踪

不适合进入本文件的内容：

- 具体产品的 UI 细节截图或交互实现（应放 `../06-frameworks-and-tools/` 的对象案例）
- 单纯视觉设计偏好而不影响控制关系的内容
- 没有机制意义、只反映短期产品流行样式的话题

---

## 二、P0：高优先级缺口

### 2.1 Human-in-the-Loop 模式分类

- **关联目录**：`human-in-the-loop/`
- **为什么重要**：审批、纠偏、审核、接管、fallback 等介入方式经常被混写；如果没有稳定分类，human-in-the-loop 很容易退化成“有人工参与”这一空泛概念。
- **当前状态**：目录已建立，但主线中尚无系统专题承接。
- **建议产物**：`human-in-the-loop-patterns.md`

### 2.2 Delegation Granularity and Control Surface

- **关联目录**：`delegation-and-control/`
- **为什么重要**：用户到底是在委托目标、步骤、工具权限，还是整个工作流，是理解 Agent 控制面的关键问题。
- **当前状态**：目录已建立，但缺少把 delegation granularity、approval points、autonomy budget 与 stop conditions 联系起来的主线专题。
- **建议产物**：`delegation-granularity.md`、`control-surface-design.md`

### 2.3 Agent UI 的最小必要可见性

- **关联目录**：`interaction-surfaces/`
- **为什么重要**：如果没有清晰的状态呈现与步骤可见性，用户难以判断 Agent 是否偏离目标；但展示过多信息也会制造认知负担。
- **当前状态**：目录已建立，但尚无专题讨论任务面板、步骤可视化、artifact view、risk prompt 与 progress feedback 的最小组合。
- **建议产物**：`minimum-visible-state.md`

### 2.4 Trust Calibration and Boundary Disclosure

- **关联目录**：`trust-and-alignment/`
- **为什么重要**：Agent 系统最大的风险之一不是单次失败，而是用户在错误的能力预期下过度信任系统。
- **当前状态**：目录已建立，但尚无专题系统展开 uncertainty communication、boundary disclosure 与 calibrated trust。
- **建议产物**：`trust-calibration.md`

---

## 三、P1：值得持续跟踪的专题

### 3.1 Interruptibility and Takeover Design

- **关联目录**：`human-in-the-loop/`、`delegation-and-control/`
- **为什么值得关注**：用户是否能中断 Agent、何时中断、接管后如何恢复，是自治系统走向实用化的关键问题。
- **当前状态**：值得持续比较不同产品的接管模型，但目前不宜写成单一标准答案。

### 3.2 Conversation UI vs Task UI

- **关联目录**：`interaction-surfaces/`
- **为什么值得关注**：对话式 UI 与任务面板式 UI 各自强调不同的心智模型，会显著影响可控性、可见性与协作效率。
- **当前状态**：适合结合真实产品案例持续比较。

### 3.3 Preference Alignment in Long-Running Tasks

- **关联目录**：`trust-and-alignment/`
- **为什么值得关注**：长任务中的偏好漂移、上下文遗忘与误判，会让“对齐”从模型问题延伸到交互与控制问题。
- **当前状态**：尚需更多产品与论文证据支撑。

### 3.4 Human-as-Agent Harness in Vibe Coding

- **关联目录**：`delegation-and-control/`、`human-in-the-loop/`、`interaction-surfaces/`、`../01-foundations/agent-system-modeling/`、`../03-multi-agent/`
- **为什么值得关注**：把人类视作轻量 harness provider 可以作为一个研究引子，帮助系统整理 vibe coding 中与 plan、constraint、review、rule、takeover 和角色切换有关的实用技巧。
- **当前状态**：已在 `human-in-the-loop/vibe-coding-human-harness.md` 形成第一版小专题，但当前更像技巧收集框架，而不是已证明的理论专题；后续重点应转向补具体工作法和案例。
- **Evidence need**：需要继续比较 coding-agent 产品、经验帖、案例复盘与团队工作流，区分哪些技巧已经形成可重复工作法，哪些仍主要停留在个体经验；同时继续观察多 agent 何时只是技巧容器，何时真的带来净收益。
- **建议产物**：继续迭代 `human-in-the-loop/vibe-coding-human-harness.md`，优先沉淀 plan 审核、模型自审、rule 文档写法、设计/代码 review 技巧与接管时机，而不是先追大理论。

---

## 四、P2：保留观察线索

### 4.1 Multi-User Oversight

- **关联目录**：`human-in-the-loop/`、`trust-and-alignment/`
- **说明**：多用户共同监督、审批与签收 Agent 结果的模式值得长期观察，但当前仍偏产品化议题。

### 4.2 Delegation Memory and User Modeling

- **关联目录**：`delegation-and-control/`
- **说明**：系统是否应记住用户过往委托偏好、风险承受度与常见控制习惯，是高价值方向，但暂不宜提前写成稳定主线。

---

## 五、与现有材料的关系

- `README.md` 已建立 `04-human-agent-interaction/` 的目录骨架与主题边界。
- `overview.md` 已提供 delegation、human-in-the-loop、UI 与 trust 四个核心视角的总体框架。
- 当前目录中的新增 overview 与正文专题仍以结构化归纳为主，后续需要按 `docs/contributing/evidence-rules.md` 和 `docs/contributing/traceability-rules.md` 继续补充 Evidence 状态与必要的 Trace 字段。
- 后续若某个问题获得更稳定证据，应优先拆成专题文档，而不是长期停留在 backlog。
