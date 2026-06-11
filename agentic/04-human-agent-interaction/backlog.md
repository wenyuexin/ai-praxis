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
- **当前状态**：`human-in-the-loop/human-in-the-loop-patterns.md` 已形成第一版专题，初步拆分了介入时机与 approval / correction / review / validation / takeover / fallback 等控制语义，并已回填 OpenHands、Copilot Edits 与 Handoff Debt 的保守证据；后续重点应转向补更多产品证据、真实工作流组合和边界对照。
- **建议产物**：继续迭代 `human-in-the-loop-patterns.md`，优先补不同产品中的介入点组合、任务风险等级映射，以及 review / validation / takeover / fallback 的边界案例。

### 2.2 Delegation Granularity and Control Surface

- **关联目录**：`delegation-and-control/`
- **为什么重要**：用户到底是在委托目标、步骤、工具权限，还是整个工作流，是理解 Agent 控制面的关键问题。
- **当前状态**：`delegation-and-control/delegation-granularity.md` 已形成第一版专题，初步拆分了委托对象、自主厚度、权限边界、stop condition、恢复路径与 HITL 介入点的关系；`delegation-and-control/control-surface-design.md` 已形成第一版专题，并回填 OpenHands、Copilot Edits、Cursor cloud agents / SDK、Claude Code permission / subagent 官方文档、Hedwig 与 authenticated delegation 的保守案例。
- **建议产物**：继续迭代 `delegation-granularity.md` 与 `control-surface-design.md`，补真实产品中的 delegation 组合方式、control surface 组件、动态 autonomy budget、permission boundary 与 stop condition 的边界案例；下一轮优先核验 OpenAI Codex CLI 的 approval modes、sandbox、rollback / checkpoint 与 control surface 官方证据，随后再整理跨产品对照。

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
- **为什么值得关注**：vibe coding 把人类对 Agent 的外部支撑暴露得更明显：人类不仅给 prompt，还会提供目标拆分、上下文压缩、handoff、review、validation、权限边界与接管机制。
- **当前状态**：`human-in-the-loop/vibe-coding-human-harness.md` 已形成案例型入口，可作为 coding-agent 场景下的经验汇总和问题引子；但它仍不应被直接写成通用理论定论。
- **后续问题**：需要继续区分哪些做法只适用于 vibe coding，哪些可以抽象为一般 agentic workflow 的 harness；同时补充任务尺度与 harness 强度、动态引导、handoff / review / validation 组合方式、人类控制 / 支撑 / 评估的边界，以及多会话 / 多 agent 委托的交互体验。
- **Evidence need**：需要继续比较 coding-agent 产品、经验帖、案例复盘与团队工作流，区分哪些技巧已经形成可重复工作法，哪些仍主要停留在个体经验；同时继续观察多 agent 何时只是技巧容器，何时真的带来净收益。
- **建议产物**：在保留 `human-in-the-loop/vibe-coding-human-harness.md` 案例性质的基础上，后续可拆出更窄的专题，例如 human harness 抽象边界、任务尺度与动态引导、AI handoff 模式、review / validation 分层、以及多模型协作的 delegation UX。

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
- `human-in-the-loop/vibe-coding-human-harness.md` 已提供 coding-agent 场景下的人类外部支架案例，backlog 后续主要追踪其可泛化边界与仍未沉淀的专题。
- `evidence-registry.md` 汇总 HAI 主线已回填和待核验的外部证据，避免从 `temp/` 调研材料直接写成正文定论。
- 当前目录中的新增 overview 与正文专题仍以结构化归纳为主，后续需要按 `docs/contributing/evidence-rules.md` 和 `docs/contributing/traceability-rules.md` 继续补充 Evidence 状态与必要的 Trace 字段。
- 后续若某个问题获得更稳定证据，应优先拆成专题文档，而不是长期停留在 backlog。
