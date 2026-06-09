# Model Context Protocol

本目录用于沉淀 `Model Context Protocol`（MCP）作为工具协议与上下文接入协议的对象研究，重点关注它如何定义 client / server 间的能力发现、工具调用、上下文边界与协议级控制信号。

## 研究边界

本目录优先回答：

- MCP 官方协议如何定义 server features、client features 与 base protocol。
- `tools/list`、`tools/call`、tool schema、structured output 与 tool error 的协议边界是什么。
- cancellation、progress、ping、roots、sampling、authorization 等能力分别处在什么层。
- MCP 能支持哪些 tool integration / capability surface，哪些不能直接外推为完整 agent orchestration contract。

不在本目录：

- 把 MCP 写成完整 Agent runtime、workflow engine 或 orchestrator。
- 把 tool discovery / invocation 直接等同于 permission、retry、timeout、checkpoint、workspace recovery、result normalization 的完整契约。
- 把某个 MCP host / SDK / 产品实现的行为直接写成 MCP 协议本体能力。

## 当前阶段

- **阶段一：官方规范窄口径核验**：已完成第一轮。当前只基于 `2025-06-18` 官方 specification 与 tools concept docs，确认 MCP 的协议层能力与 stop-line。
- **阶段二：host / SDK 实现对照**：待开始。后续可对照 Claude Code、OpenAI Agents SDK、LangGraph 或其他 MCP host 的实现差异。
- **阶段三：跨对象契约比较**：待开始。后续再把 MCP 与 LangGraph 等 workflow runtime 放到 Agent Adapter / Orchestrator 契约维度中比较。

## 目录结构

本目录当前以对象正文为主，研究辅助材料放在 `notes/` 下；README 只保留阅读定向，不展开元信息文件树。

## 阅读入口

推荐先读 `overview.md`，重点看“协议能确认什么”和“不能外推什么”。研究辅助材料只用于追溯 claim-source 对照和后续补证，不作为主要阅读入口。

## Evidence

- Status: Observed / Inferred
- Sources:
  - `https://modelcontextprotocol.io/specification/2025-06-18`
  - `https://modelcontextprotocol.io/docs/concepts/tools`
  - `https://modelcontextprotocol.io/specification/2025-06-18/server/tools`
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic`
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic/transports`
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic/utilities/cancellation`
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic/utilities/progress`
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic/utilities/ping`
  - `https://modelcontextprotocol.io/specification/2025-06-18/client/roots`
  - `https://modelcontextprotocol.io/specification/2025-06-18/client/sampling`
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization`
- Trace: 从 `agentic/06-frameworks-and-tools/backlog.md` 的 `Agent Adapter / Orchestrator 契约核验队列` 中选择 `MCP` 做第一轮窄口径核验；本目录只沉淀协议层边界，不写 host / SDK 实现结论；claim-source 对照见 `notes/evidence.md`。
- Needs: 后续需要补 MCP host / SDK 的实现对照，尤其是 tool approval、timeout、retry、large output、streaming、side-effect governance 与 workspace policy 的对象内边界。

*最后更新: 2026-06-09*
