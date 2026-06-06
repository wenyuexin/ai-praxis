# Frameworks and Tools Backlog（候选对象与调研线索）

本文件用于记录 `06-frameworks-and-tools` 的候选对象、前沿观察项、证据不足项和后续调研入口。它不是任务列表，也不代表对象已经进入主干结构。

---

## 一、收录规则

### 1.1 主干收录信号

对象进入 `01-frameworks/`、`02-coding-agents-and-tools/`、`03-project-studies/`、`04-skill-and-tool-systems/` 等主干目录前，至少应满足若干信号：

- 有明确官方文档、GitHub 仓库、论文或技术报告
- 有稳定 API、版本路线或持续维护记录
- 在生态中具备代表性，而不是短期热点
- 有清晰类别归属，能解释它为什么属于框架、工具、项目案例或 Skill/Tool 系统
- 其价值不只是“热度高”，还应有明确架构特征或工程启发

### 1.2 前沿观察信号

即使对象不成熟，也可以进入 backlog 观察：

- 提出了新的系统设计范式
- 被学术综述、顶会论文、官方博客或主流 GitHub 项目明确讨论
- 代表协议、记忆、沙箱、多 Agent 协作、coding agent 等方向的新分岔
- 有可能在未来 1–3 年影响 Agent 工程生态

### 1.3 不直接进入主干的情况

以下对象默认先放 backlog：

- 论文原型，无稳定工程实现
- research prototype，非正式产品
- 协议早期竞品，生态尚未形成
- 商业或社区数据证据不足
- 只有第三方热度描述，缺乏官方或高质量来源验证

---

## 二、主干候选对象

### 2.1 Frameworks 候选

| 对象 | 候选归属 | 观察理由 | 状态 |
|------|----------|----------|------|
| `Semantic Kernel` | `01-frameworks/` | 微软企业级 AI SDK，模型无关，和 Agent Framework 生态有强关联 | 待补正式文档 |
| `Microsoft Agent Framework` | `01-frameworks/` | 整合 AutoGen 与 Semantic Kernel 的企业级 Agent 框架方向 | 待核验版本状态 |
| `DSPy` | `01-frameworks/` 或 `05-comparisons/` | 声明式 LM program / prompt optimization 代表，与 Agent orchestration 有交集 | 观察中 |
| `Google ADK` | `01-frameworks/` | 大厂 Agent Development Kit，可能影响协议与框架生态 | 证据不足，观察中 |
| `Agno` | `01-frameworks/` | 轻量 Agent framework / runtime，工程化方向明确 | 观察中 |
| `smolagents` | `01-frameworks/` | 轻量 code-as-action / ToolCallingAgent 范式 | 观察中 |

### 2.2 Coding Tools 候选

| 对象 | 候选归属 | 观察理由 | 状态 |
|------|----------|----------|------|
| `OpenHands / OpenDevin` | `02-coding-agents-and-tools/` | 开源 coding agent 代表，和 SWE-bench 生态关联强 | 候选主干 |
| `SWE-agent` | `02-coding-agents-and-tools/` | 软件工程任务 Agent 代表项目 | 待调研 |
| `Cline` | `02-coding-agents-and-tools/` | IDE 内编码 Agent 工具，产品形态有代表性 | 待调研 |
| `GitHub Next Ace` | `02-coding-agents-and-tools/` | GitHub Next research prototype，关注团队部署 coding agents 的 alignment bottleneck | research prototype，先观察 |
| `LocAgent` | `02-coding-agents-and-tools/` 或 `07-evaluation/swe-benchmarks/` 相关引用 | 图引导代码定位，可能成为 coding agent 标准组件 | 学术项目，先观察 |

### 2.3 Project Studies 候选

| 对象 | 候选归属 | 观察理由 | 状态 |
|------|----------|----------|------|
| `OpenClaw` | `03-project-studies/openclaw/` | 自托管 AI 自动化执行引擎，适合作完整系统案例 | 已创建目录 |
| `Hermes Agent` | `03-project-studies/hermes-agent/` | 记忆、技能、网关、上下文、自动化等完整模块拆解 | 已在主干 |
| `Ruflo` | `03-project-studies/` 或 `02-coding-agents-and-tools/` | Claude Code 编排平台，若资料可靠可做系统案例 | 证据不足，观察中 |
| `Agency` | `03-project-studies/` 或 `01-frameworks/` | 多 Agent 编排热度高，但需核验官方资料与差异性 | 证据不足，观察中 |

### 2.4 Skill / Tool Systems 候选

| 对象 | 候选归属 | 观察理由 | 状态 |
|------|----------|----------|------|
| `MCP-Zero` | `04-skill-and-tool-systems/` | 主动工具发现范式，可能改变 tool-use agent 设计 | 论文原型，先观察 |
| `Doc2Agent` | `04-skill-and-tool-systems/` | 从 API 文档自动生成工具和 tool-using agent | 论文原型，先观察 |
| `AutoTool` | `04-skill-and-tool-systems/` | 工具选择数据集与推理-工具统一框架 | 论文原型，先观察 |
| `Mem0` | 视内容归属到 `02-single-agent/memory/` 或具体框架实现案例 | 记忆基础设施，已有框架集成线索 | 需独立评估 |
| `MemOS` | 视内容归属到 `02-single-agent/memory/` 或项目案例 | memory as OS 抽象有观察价值 | 观察中 |

---

## 三、前沿观察对象

### 3.1 P0：高优先级观察

| 对象 | 方向 | 为什么值得观察 | 主归属提醒 |
|------|------|----------------|------------|
| `SAGE` | Memory / graph memory | 自演进图记忆，可能突破静态 GraphRAG/RAG 记忆模式 | 理论归 `02-single-agent/memory/`，工程案例再进 `06` |
| `Hindsight` | Memory architecture | 多网络结构化记忆，探索“记忆作为一等执行对象” | 理论归 `02-single-agent/memory/` |
| `ATOM` | Multi-agent collaboration | 将预算控制作为多 Agent 协作一等设计目标 | 理论归 `03-multi-agent/coordination/` |
| `Fault-Tolerant Sandboxing` | Execution safety | 事务性沙箱，把回滚和安全拦截引入 Agent 执行环境 | 主归属 `05-environments/sandboxing-and-safety/` |
| `AG-UI` | Agent UI protocol | Agent 与 UI 的交互协议标准化方向 | 主归属 `04-human-agent-interaction/interaction-surfaces/` |
| `Code as Agent Harness` | Methodology | 将代码视为 Agent harness 的方法论框架；相关候选研究对象见 `../01-foundations/agent-system-modeling/candidates.md` | 主归属 `01-foundations/agent-system-modeling/` |

### 3.2 P1：一般观察

| 对象 | 方向 | 为什么值得观察 | 主归属提醒 |
|------|------|----------------|------------|
| `MANGO` | Multi-agent optimization | 用 flow network / gradient optimization 优化协作关系 | `03-multi-agent/coordination/` |
| `Meta-Team / Collaborative Self-Evolution` | Multi-agent self-evolution | 多 Agent 证据交换与协同演进 | `03-multi-agent/collaboration/` |
| `AgentBay` | Hybrid runtime | 同一 session 支持 AI 程序化接口和 human takeover | `05-environments/` 与 `04-human-agent-interaction/` |
| `ceLLMate` | Browser sandbox | browser-level sandboxing，降低 prompt injection 攻击面 | `05-environments/browser-environments/` |
| `SWE-Gym` | SWE training environment | 从 benchmark measurement 走向 training environment | `07-evaluation/swe-benchmarks/` |
| `ScalingEval` | Evaluation infrastructure | 大规模自动化评估协议探索 | `07-evaluation/agent-benchmarks/` |

### 3.3 P2：记录留存

| 对象 | 方向 | 状态 |
|------|------|------|
| `Cognitive Kernel-Pro` | Research-oriented agent framework | 信息不足，等待更多公开资料 |
| `MemEngine` | Memory library | 独立信息不足，仅作 citation 线索 |
| `NVIDIA NemoClaw` | Secure execution / agent runtime | 定位与成熟度需核验 |

---

## 四、证据不足与待核验项

以下内容不能直接写入主干结论，只能作为后续调研线索：

- `OpenClaw`、`HermesAgent`、`Ruflo`、`Agency` 等项目的 GitHub stars、增长速度、生态排名等热度数据，需要官方仓库或可信统计源核验。
- `CrewAI` 的 Fortune 500 使用比例、agent 月度运行量等数据，需要官方案例或可靠来源确认。
- `Microsoft Agent Framework` 的 GA / RC / LTS 状态，需要 Microsoft 官方文档确认。
- `AG-UI` 的标准状态、roadmap 和生态采纳度，需要官方文档确认。
- `SAGE`、`ATOM`、`MANGO`、`AgentBay`、`ceLLMate` 等论文原型，需要持续跟踪是否开源、是否被引用、是否形成真实工程实现。
- `Mem0`、`MemOS`、`Hindsight` 等 memory infrastructure，需要区分“理论价值”“工程可用性”和“生态采用度”。

---

## 五、后续建设建议

1. 优先补 `01-frameworks/` 中 LangGraph、AutoGen、CrewAI、Semantic Kernel、MAF 的正式对象文档。
2. 补 `02-coding-agents-and-tools/` 中 OpenHands / SWE-agent / Cline 的初步调研。
3. 把 `OpenClaw` 拆成 architecture、safety、ecosystem 等专题文档。
4. 在 `05-comparisons/` 中建立 `framework-selection.md` 和 `coding-agent-products.md`。
5. 将 memory、multi-agent、sandbox、evaluation 等前沿对象分别回流到 `02/03/05/07` 的 backlog 或专题文档中，避免全部堆在 `06`。

---

*最后更新: 2026-05-31*
