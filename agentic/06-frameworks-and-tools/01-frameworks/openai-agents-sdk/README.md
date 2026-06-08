# OpenAI Agents SDK / Responses API

本目录用于沉淀 `OpenAI Agents SDK / Responses API` 这一对象的研究材料，重点关注它如何组织 agent、handoff、guardrails / human review、tools / MCP、results / state、sessions、tracing 与 sandbox agents。

## 研究边界

本目录优先回答：

- `OpenAI Agents SDK` 官方公开了哪些 agent orchestration 能力面。
- handoff、tool、guardrail、session、result/state、trace 在官方文档中分别处于什么层。
- `Responses API` 与 `OpenAI Agents SDK` 在对象研究里应如何共同讨论，哪些判断只适用于 SDK。
- `sandbox agents`、session、resumable state 与 runtime / job resume、workspace recovery 的边界是什么。
- 它与 `LangGraph`、`AutoGen`、`CrewAI`、`Microsoft Agent Framework` 在状态与编排语义上有什么可比与不可比之处。

不在本目录：

- 把官方 docs 导航页出现的模块入口直接写成完整运行时契约。
- 把外部整理出的“七层架构”写成官方命名。
- 把 session / result state / sandbox agents 直接外推成 filesystem recovery 或完整 side-effect recovery。
- OpenAI 全部平台工具的逐项使用教程；这里只保留与 agent orchestration 边界直接相关的部分。

## 当前阶段

- **阶段一：对象入口建立**：已完成。当前先建立对象 README 与 `overview.md`，把第一轮可稳定确认的模块入口、边界与待核验问题落位。
- **阶段二：第一轮官方文档补证**：已完成入口级补证。当前可直接确认 handoffs、guardrails / human review、results and state、integrations and observability、sandbox agents 等模块在官方 docs 中存在；tracing 的 trace/span/group_id 与默认 span 类型在 SDK 文档中可直接确认。
- **阶段三：第一批稳定机制专题**：已完成。当前已把 handoff、guardrails、results/state、sessions、tools/MCP、tracing 分别下沉成独立专题，避免对象总览承担过多机制细节。
- **阶段四：第二批恢复专题**：已完成第一轮保守落位。当前已新增 `run-state-and-resume.md`、`server-managed-continuation.md`、`sandbox-agents.md`，但只写到官方 release notes、API reference 与 HITL 文档可直接支撑的边界。
- **阶段五：深水区源码补证**：已形成明确 checklist，待开始。后续优先核验 `RunState`、`Session`、resumable state、sandbox/workspace recovery、MCP lifecycle 与 run error handler 的精确边界。
- **阶段六：跨框架对照**：待开始。后续再与 `LangGraph`、`AutoGen`、`CrewAI`、`Microsoft Agent Framework` 做更细状态/控制语义对照。

## 目录结构

```text
openai-agents-sdk/
├── notes/
├── README.md
├── overview.md
├── handoffs-and-delegation.md
├── guardrails-and-human-review.md
├── results-and-state.md
├── run-state-and-resume.md
├── sandbox-agents.md
├── server-managed-continuation.md
├── sessions-and-persistence.md
├── tools-and-mcp.md
└── tracing-and-observability.md
```

## 阅读入口

推荐阅读顺序：

1. `overview.md`：先看对象定位、当前可稳定确认的能力面与 stop-line。
2. `handoffs-and-delegation.md`：先拆清 handoff、tool、`Agent.as_tool()` 的 delegation 边界。
3. `guardrails-and-human-review.md`：理解 input/output/tool guardrails、tripwire 与 approval / interruption surface。
4. `results-and-state.md`：区分 final output、new items、interruptions、`to_state()` 等不同结果表面。
5. `run-state-and-resume.md`：把 run interruption / resumable state 从一般结果表面中单独拆出。
6. `server-managed-continuation.md`：单独理解 `conversation_id` / `previous_response_id` 的 OpenAI-managed 续接层。
7. `sessions-and-persistence.md`：区分 session history persistence 与其他 continuation / resume surface。
8. `sandbox-agents.md`：只按 release notes 与 API reference 写 sandbox runtime/session/snapshot surface。
9. `tools-and-mcp.md`：把 hosted tools、function tools、agent-as-tool 与 hosted MCP 的能力接入面拆开。
10. `tracing-and-observability.md`：建立 trace / span / group_id 与默认 span 类型的稳定理解。

`notes/` 作为研究辅助材料层保留，但不是主要阅读入口；只有在需要追溯 claim-source 对照、深水区 checklist、失败搜索或更细源码补证线索时再进入。

## 与 05-environments 的对照价值

`OpenAI Agents SDK / Responses API` 对环境层研究的价值，不在于立即给出稳定的 workspace / checkpoint 结论，而在于帮助拆清几组容易混写的语义：

- session history / results state 与 runtime/job resume 的关系
- tracing / observability 与 checkpoint / recovery 的关系
- sandbox agents 与 workspace/filesystem recovery 的关系
- handoff / tool / guardrail 作为 orchestration control surface 的不同层次

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: `https://developers.openai.com/api/docs/guides/agents`、`https://openai.github.io/openai-agents-python/handoffs`、`https://openai.github.io/openai-agents-python/results`、`https://openai.github.io/openai-agents-python/sessions`、`https://github.com/openai/openai-agents-python/blob/main/docs/tracing.md`
- Trace: 先由 `agentic/06-frameworks-and-tools/backlog.md` 与 `conflict.md` 中的候选对象 / 待核验条目推进；在 `overview.md` 中先沉淀可直接追到官方入口的对象边界，再等待更细 SDK 文档或源码回流机制级专题。
- Needs: `Results and state`、`Sessions`、`Sandbox agents`、run error handlers、MCP servers / manager、SDK 源码中的 `RunState` / session 实现与恢复边界。
