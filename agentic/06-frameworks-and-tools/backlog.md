# Frameworks and Tools Backlog（内容缺口与待核验问题）

本文件记录 `06-frameworks-and-tools/` 尚未覆盖的内容缺口、比较缺口和证据缺口。候选研究对象单独记录在 [`candidates.md`](./candidates.md)；事实冲突或口径不一致记录在 [`conflict.md`](./conflict.md)。本文件中的条目仍是研究输入，不代表已经进入主干结论。

---

## 1. Agent Adapter / Orchestrator 契约缺口

当前仍缺少一套经过逐对象核验的比较材料，用来区分 Agent adapter、orchestrator、workflow state、runtime environment 和最终产物之间的契约边界。现有条目先记录“还缺什么证据”，不把候选对象队列重新混入本文件。

| 研究对象 | 仍需补足的内容缺口 | 当前承接 |
|---|---|---|
| `MCP` | 区分 tool discovery / invocation 与完整 orchestration contract；补齐 `tools/list`、`tools/call`、capabilities、timeout、cancel、large output、streaming / partial output 的协议边界 | `04-skill-and-tool-systems/mcp/` |
| `LangGraph` | 补齐 checkpoint、thread、interrupt、retry、time travel、replay 的真实边界；区分 graph/workflow state 与 workspace filesystem recovery | `01-frameworks/langgraph/notes/evidence.md`；server 私有语义仍待补证 |
| `OpenAI Agents SDK / Responses API` | 继续核验 sessions、results/state、handoff、tracing、guardrails、sandbox agents，以及 `RunState`、server continuation、sandbox state、MCP lifecycle、error/recovery 边界 | 对象目录与 `conflict.md` |
| `CrewAI` | 补齐 Flow persistence、checkpoint、restore、fork lineage、retry / cancel / human input 的粒度和版本状态 | 待核验 |
| `OpenHands` | 补齐 workspace、runtime、event log、sandbox、recovery 与环境层对象研究之间的边界 | `03-project-studies/openhands/`；环境层补证见 `05-environments/candidates.md` |
| `SWE-agent` | 补齐 trajectory replay、checkpoint / resume、workspace / container / final artifact 的正式语义 | `03-project-studies/swe-agent/`；环境层补证见 `05-environments/candidates.md` |
| `AutoGen` | 补齐 group chat、tool、MCP、handoff / delegation、trace、cancel / retry 的对象内边界，不与 Microsoft Agent Framework 混写 | 候选对象见 `candidates.md` |
| `Microsoft Agent Framework` | 补齐 GA / RC / LTS 状态，以及 agents / workflows、AgentTool、MCP、checkpointing、session、middleware 和 AutoGen / Semantic Kernel 整合边界 | 候选对象见 `candidates.md` |
| `Semantic Kernel` | 补齐 AgentGroupChat、plugin/kernel、observability 与 checkpoint / resume / durable execution 的边界，区分 SK 本体与 Azure / Foundry 集成 | 候选对象见 `candidates.md` |

---

## 2. 其他证据与比较缺口

- `OpenClaw`、`HermesAgent`、`Ruflo`、`Agency` 等项目的 GitHub stars、增长速度、生态排名等热度数据，需要官方仓库或可信统计源核验。
- `CrewAI` 的 Fortune 500 使用比例、agent 月度运行量等数据，需要官方案例或可靠来源确认。
- `Microsoft Agent Framework` 的 GA / RC / LTS 状态，需要 Microsoft 官方文档确认。
- `AG-UI` 的标准状态、roadmap 和生态采纳度，需要官方协议文档确认。
- `SAGE`、`ATOM`、`MANGO`、`AgentBay`、`ceLLMate` 等论文原型，需要持续跟踪是否开源、是否被引用、是否形成真实工程实现。
- `Mem0`、`MemOS`、`Hindsight` 等 memory infrastructure，需要区分理论价值、工程可用性和生态采用度。

## 3. 尚未形成稳定比较主线的方向

以下方向已被识别，但尚未形成可以直接进入 overview 或比较专题的稳定材料：

1. Agent adapter / orchestrator / workflow / runtime 的契约边界；
2. checkpoint、resume、retry、cancel、human input 和 recovery 的跨对象粒度比较；
3. coding agent 的 workspace、sandbox、event log 与最终 artifact 的关系；
4. memory infrastructure 与完整 Agent project 的归属边界；
5. Skill、Tool、MCP、permission、approval 和 sandbox 的执行治理分层。

这些条目描述的是内容缺口，不是当前任务清单；具体研究计划仍放在 `docs/temp/`，稳定的跨轮次推进顺序才考虑提炼为 `roadmap.md`。

---

*迁移来源：历史 `landscape.md` 与原 `backlog.md` 的对象核验、证据不足和建设建议条目。候选对象已拆分至 `candidates.md`；本文件继续承接内容、比较和证据缺口，事实或口径冲突转入 `conflict.md`。*
