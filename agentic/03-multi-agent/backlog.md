# Multi-Agent Backlog

> 适用范围：`03-multi-agent/` 的内容缺口、前沿问题与待补专题
> 使用原则：本文件记录多智能体主题中已经识别但尚未充分沉淀的高价值问题，不是任务计划，也不表示结论已进入主干定论

## 一、收录规则

适合进入本文件的内容通常满足以下条件之一：

- 能帮助解释 multi-agent 的收益边界、协调成本和失败模式
- 在当前目录结构中尚无稳定落点，但被高质量调研明确指出重要
- 属于新兴方向、冲突问题或需要系统比较的主题
- 不宜直接写进 `overview.md` 的主结论，但值得长期跟踪

不适合进入本文件的内容：

- 具体框架或项目案例（应放 `../06-frameworks-and-tools/`）
- 只有热度而无机制价值的话题
- 已经充分覆盖、暂时没有明显分歧的基础内容

---

## 二、P0：高优先级缺口

### 2.1 Coordination Topology

- **关联目录**：`coordination/`、`organizational/`
- **为什么重要**：`centralized / decentralized / hybrid / independent aggregation` 是 multi-agent 最关键的分类轴之一，比“是不是 multi-agent”本身更能决定收益与失败模式。
- **现状**：已形成专题文档 `coordination-topology.md`，后续需要继续补定量证据、术语对照和失败模式映射。
- **建议产物**：后续可拆分子专题，如 `topology-vs-task-type.md` 或与 `failure-modes.md` 联动。

### 2.2 Failure Modes Catalog

- **关联目录**：`coordination/`、`shared-state-and-context/`、`communication-protocols/`
- **为什么重要**：如果没有系统性失败分类，`03-multi-agent/` 容易被成功案例主导。错误放大、状态漂移、重复劳动、死锁、语义失配等都应被显式建模。
- **现状**：已形成第一版专题 `coordination/failure-modes.md`，后续需要继续补 taxonomy、案例与定量证据。
- **建议产物**：后续可拆成更细的失败模式专题。

### 2.3 Multi-Agent 何时优于 Single-Agent

- **关联目录**：`overview.md`
- **为什么重要**：这是整个 multi-agent 目录最关键的判断问题之一。当前已有大量正反证据，但没有统一决策框架。
- **现状**：已形成第一版专题 `coordination/when-multi-agent-helps.md`，后续需要继续补任务分类、基线能力与量化阈值。
- **建议产物**：后续可进一步拆为任务类型判断框架或案例专题。

### 2.4 Hidden Cost Modeling

- **关联目录**：`coordination/`、`shared-state-and-context/`
- **为什么重要**：token redundancy、重复消息传播、隐性 loop、状态同步成本往往在工程实践中被低估。
- **现状**：已有案例和定量线索，但主干中尚无位置系统承接。
- **建议产物**：`hidden-costs.md`

---

## 三、P1：值得持续跟踪的专题

### 3.1 Structured Protocols vs Emergent Coordination

- **关联目录**：`communication-protocols/`
- **为什么值得关注**：标准化协议（如 A2A / MCP 的不同层次）与 gossip / swarm / emergent coordination 在设计原则上有本质差异。
- **当前状态**：适合持续比较，但目前不宜写成单一正确路线。

### 3.2 Specialization Granularity

- **关联目录**：`collaboration/`、`organizational/`
- **为什么值得关注**：agent 应专业化到什么程度，直接影响协作收益与通信冗余。
- **当前状态**：共识不足，但对目录建设意义很大。

### 3.3 Multi-Agent Benchmark Interpretation

- **关联目录**：`../07-evaluation/`
- **为什么值得关注**：不同 benchmark 对 multi-agent 的正负结论高度依赖任务类型与评测设定。
- **当前状态**：更适合先放 backlog，后续跨目录联动。

### 3.4 Human Agent as Supervisor / Harness in Coding Workflows

- **关联目录**：`coordination/`、`organizational/`、`../04-human-agent-interaction/`
- **为什么值得关注**：在 coding workflow 中，人类可能不是外部审批者，而是作为特殊 agent 参与规划、切换、验收与 delegation policy 制定；这会改变“single-agent vs multi-agent”比较的前提，因为系统收益不再只取决于 agent 数量，还取决于人类 harness 如何参与协调。
- **当前状态**：当前目录已覆盖 topology、hidden cost、specialization granularity 与收益边界，但尚未单独讨论“人类 agent + 多 agent”组合何时真正优于“人类 harness + 单 agent”或“纯多 agent”结构。
- **Evidence need**：需要比较 planner / executor / reviewer / test-runner 等角色拆分在有人类监督时的净收益，尤其关注 coordination overhead、handoff 频率、批准成本与 artifact 可见性；避免把概念上的角色拆分直接写成工程上可落地的最优结构。
- **建议产物**：可先作为 `when-multi-agent-helps.md` 的补充问题，后续再决定是否拆成独立专题。

---

## 四、P2：保留观察线索

### 4.1 Gossip-Based Coordination

- **关联目录**：`communication-protocols/`
- **说明**：作为新兴方向值得观察，但当前仍偏研究议程，不宜直接进入主干中心叙事。

### 4.2 Budget-Aware Multi-Agent Coordination

- **关联目录**：`coordination/`
- **说明**：把预算控制作为一等设计目标的工作值得跟踪，尤其适合后续与 hidden cost 模型结合。

---

## 五、与现有材料的关系

- `overview.md` 已建立多智能体的高层组织框架。
- `temp/web-search/2.md` 提供了本 backlog 的大部分输入。
- `temp/conflict.md` 已承接“multi-agent 是否天然更强”“MCP 与 A2A 的边界”等关键冲突。
- 后续一旦某一问题获得更稳定证据，应拆成专题文档而不是长期停留在 backlog。
