# Frameworks and Tools Candidates（候选研究对象）

本文件只记录 `06-frameworks-and-tools/` 下一步值得持续跟踪的论文、仓库、产品、协议或机制对象。它不代表对象已经进入主干，也不承接一般内容缺口；问题和比较缺口继续记录在 `backlog.md`，事实或口径冲突继续记录在 `conflict.md`。

## 1. 候选对象及预期归属

### 1.1 Frameworks

| 对象 | 候选归属 | 观察理由 | 状态 |
|---|---|---|---|
| `Semantic Kernel` | `01-frameworks/` | 微软企业级 AI SDK，需继续核验 AgentGroupChat、plugin/kernel、observability 与 durable checkpoint / resume 的边界 | 待补正式文档 |
| `Microsoft Agent Framework` | `01-frameworks/` | 后继整合框架，需继续核验 agents / workflows、checkpointing、session、middleware 和 MCP 入口的版本状态 | 待核验版本状态 |
| `AutoGen` | `01-frameworks/` | 多 Agent 编排框架，需继续核验 group chat、tool、MCP、handoff / delegation、trace、cancel / retry 的对象内边界，并与 Microsoft Agent Framework 分开核验 | 待补对象研究 |
| `DSPy` | `01-frameworks/` 或 `05-comparisons/` | 声明式 LM program / prompt optimization 代表，与 Agent orchestration 有交集 | 观察中 |
| `Google ADK` | `01-frameworks/` | 大厂 Agent Development Kit，可能影响协议与框架生态 | 证据不足，观察中 |
| `Agno` | `01-frameworks/` | 轻量 Agent framework / runtime，工程化方向明确 | 观察中 |
| `smolagents` | `01-frameworks/` | 轻量 code-as-action / ToolCallingAgent 范式 | 观察中 |

### 1.2 Coding tools

| 对象 | 候选归属 | 观察理由 | 状态 |
|---|---|---|---|
| `OpenDevin` | `02-coding-agents-and-tools/` | OpenHands 相关的历史/前身对象，需核验其当前独立项目状态与研究价值 | 状态与归属待核验 |
| `Cline` | `02-coding-agents-and-tools/` | IDE 内编码 Agent 工具，产品形态有代表性 | 待调研 |
| `GitHub Next Ace` | `02-coding-agents-and-tools/` | GitHub Next research prototype，关注团队部署 coding agents 的 alignment bottleneck | 先观察 |
| `LocAgent` | `02-coding-agents-and-tools/` 或 `07-evaluation/swe-benchmarks/` 相关引用 | 图引导代码定位，可能成为 coding agent 标准组件 | 学术项目，先观察 |

### 1.3 Project studies

| 对象 | 候选归属 | 观察理由 | 状态 |
|---|---|---|---|
| `Ruflo` | `03-project-studies/` 或 `02-coding-agents-and-tools/` | Claude Code 编排平台，若资料可靠可做系统案例 | 证据不足，观察中 |
| `Agency` | `03-project-studies/` 或 `01-frameworks/` | 多 Agent 编排对象，需核验官方资料与差异性 | 证据不足，观察中 |

### 1.4 Skill / Tool systems

| 对象 | 候选归属 | 观察理由 | 状态 |
|---|---|---|---|
| `MCP-Zero` | `04-skill-and-tool-systems/` | 主动工具发现范式，可能改变 tool-use agent 设计 | 论文原型，先观察 |
| `Doc2Agent` | `04-skill-and-tool-systems/` | 从 API 文档自动生成工具和 tool-using agent | 论文原型，先观察 |
| `AutoTool` | `04-skill-and-tool-systems/` | 工具选择数据集与推理-工具统一框架 | 论文原型，先观察 |
| `Mem0` | 视内容归属到 `02-single-agent/memory/` 或具体框架实现案例 | 记忆基础设施，已有框架集成线索 | 需独立评估 |
| `MemOS` | 视内容归属到 `02-single-agent/memory/` 或项目案例 | memory as OS 抽象有观察价值 | 观察中 |

## 2. 前沿观察对象

| 对象 | 方向 | 为什么值得观察 | 主归属提醒 |
|---|---|---|---|
| `SAGE` | Memory / graph memory | 自演进图记忆，可能突破静态 GraphRAG / RAG 记忆模式 | 理论归 `02-single-agent/memory/`，工程案例再进 `06` |
| `Hindsight` | Memory architecture | 多网络结构化记忆，探索“记忆作为一等执行对象” | 理论归 `02-single-agent/memory/` |
| `ATOM` | Multi-agent collaboration | 将预算控制作为多 Agent 协作的一等设计目标 | 理论归 `03-multi-agent/coordination/` |
| `Fault-Tolerant Sandboxing` | Execution safety | 事务性沙箱，把回滚和安全拦截引入 Agent 执行环境 | 主归属 `05-environments/sandboxing-and-safety/` |
| `AG-UI` | Agent UI protocol | Agent 与 UI 的交互协议标准化方向 | 主归属 `04-human-agent-interaction/interaction-surfaces/` |
| `Code as Agent Harness` | Methodology | 将代码视为 Agent harness 的方法论框架 | 主归属 `01-foundations/agent-system-modeling/` |
| `MANGO` | Multi-agent optimization | 用 flow network / gradient optimization 优化协作关系 | `03-multi-agent/coordination/` |
| `Meta-Team / Collaborative Self-Evolution` | Multi-agent self-evolution | 多 Agent 证据交换与协同演进 | `03-multi-agent/collaboration/` |
| `AgentBay` | Hybrid runtime | 同一 session 支持 AI 程序化接口和 human takeover | `05-environments/` 与 `04-human-agent-interaction/` |
| `ceLLMate` | Browser sandbox | browser-level sandboxing，降低 prompt injection 攻击面 | `05-environments/browser-environments/` |
| `SWE-Gym` | SWE training environment | 从 benchmark measurement 走向 training environment | `07-evaluation/swe-benchmarks/` |
| `ScalingEval` | Evaluation infrastructure | 大规模自动化评估协议探索 | `07-evaluation/agent-benchmarks/` |
| `Cognitive Kernel-Pro` | Research-oriented agent framework | 信息不足，等待更多公开资料 | 待观察 |
| `MemEngine` | Memory library | 独立信息不足，仅作 citation 线索 | 待观察 |
| `NVIDIA NemoClaw` | Secure execution / agent runtime | 定位与成熟度需核验 | 待观察 |

## 3. 维护边界

- 候选对象进入本文件，不等于已经进入主干目录；
- 对象的事实核验、版本边界和来源链路进入对象研究或 `notes/`，不在本文件展开长篇证据；
- 仅是“还缺哪些主题、比较维度或证据”的问题，继续进入 `backlog.md`；
- 存在来源冲突、术语冲突或边界相反结论时，进入 `conflict.md`；
- 本文件中的对象和状态仍是研究输入，不能直接升级为 `overview.md` 的稳定结论。

---

*迁移来源：历史 `landscape.md` 与原 `backlog.md` 中尚未进入主干的候选对象和前沿观察条目。已进入主干的对象不在本队列重复登记；其事实核验和补证缺口回到对象目录、`backlog.md` 或 `conflict.md`。*
