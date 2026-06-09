# Orchestration Implementations and Agent Contracts

> 本文件是 `Agent Adapter / Orchestrator` 契约研究的第一版横向专题，只使用已核验的 `MCP` 与 `LangGraph` 作为样本；当前结论以 `Observed / Inferred` 为主，不把两者外推成统一行业标准。

## 一、问题定位

当一个 Agent 系统需要接入外部工具、内部 Agent、长任务 workflow 或人类审批时，真正需要统一的不是“某个 provider 怎么调用”，而是调用边界如何被表达：任务输入、能力发现、执行状态、结果结构、错误与取消、trace、workspace、checkpoint 和副作用责任分别落在哪一层。

当前两个样本提供了互补边界：

- `MCP` 标准化 LLM application 与外部 context / tool 的 protocol surface。
- `LangGraph` 标准化 graph workflow 的 state、checkpoint、interrupt、resume、replay 与 state edit 主路径。

这说明 `Agent Adapter / Orchestrator` 契约至少需要分成协议接入层、workflow runtime 层和 execution environment 层；不能用其中任一层替代完整系统契约。

---

## 二、样本边界

### 2.1 MCP：工具 / 上下文协议层样本

当前 MCP 核验可以稳定支持：

- server capability discovery，包括 tools、resources、prompts 等能力声明。
- `tools/list` 与 `tools/call` 形式的工具发现与调用。
- tool `inputSchema`、可选 `outputSchema` 与 `structuredContent`。
- `notifications/cancelled`、`notifications/progress`、`ping` 等协议级控制 / 可观测信号。
- `roots`、`sampling`、`elicitation` 等 client-side features。
- HTTP transport 的 authorization 框架。

但 MCP 当前不能直接支持以下完整契约：

- workflow state、checkpoint、time travel 或 replay。
- 强制 timeout、retry、backoff、idempotency 策略。
- workspace filesystem snapshot / rollback。
- external side-effect recovery。
- 跨工具、跨 server 的统一 business result model。
- 完整 tool governance、approval policy、audit 与权限隔离。

因此，MCP 适合作 `AdapterSpec` 与部分 `TaskSpec / ResultSpec / ErrorSpec` 的协议层样本，不适合作完整 `OrchestratorSpec` 样本。

### 2.2 LangGraph：workflow state / checkpoint 样本

当前 LangGraph 核验可以稳定支持：

- graph state 由 channels、reducers、`channel_values`、`channel_versions`、`versions_seen` 等结构推进。
- `thread_id` 是 checkpoint 保存与检索的 workflow identity / persistence key。
- checkpointer 通过 `put`、`put_writes`、`get_tuple`、`list`、`delete_thread` 等接口管理 checkpoint 与 pending writes。
- interrupt / resume 依赖 checkpointer；`Command(resume=...)` 不是进程级原地恢复。
- replay / fork / `update_state` 都围绕 graph state checkpoint 展开。
- 普通 `update_state` 应用 node writers / reducers，写入 `source=update` checkpoint；显式 copy / time-travel 路径可写入 `source=fork` checkpoint。

但 LangGraph 当前不能直接支持以下完整契约：

- workspace filesystem checkpoint、container resume 或 process snapshot。
- 外部 API、数据库写入、文件系统变更等副作用回滚。
- 跨实例并发操作同一 `thread_id` 的完整一致性保证。
- server 私有实现中 interrupted thread、branch UI、metadata propagation 的完整语义。
- 跨工具 / 跨 provider 的统一 permission、cost、audit、result normalization。

因此，LangGraph 适合作 `ExecutionSpec`、`StateSpec` 与部分 `ErrorSpec` 的 workflow runtime 样本，不应被写成完整 execution environment 或 tool governance 样本。

---

## 三、契约维度初稿

### 3.1 `TaskSpec`

`TaskSpec` 描述“要做什么”和“允许用什么能力”。第一版最小字段可以包括：

- `objective`：任务目标或用户请求。
- `inputs`：结构化输入、上下文片段、资源引用或 prompt 模板。
- `capabilities`：可用工具、server、agent、workflow node 或 human input。
- `schemas`：输入 schema、输出 schema、约束和验证要求。
- `scope`：filesystem roots、workspace boundary、allowed resources 或任务风险边界。

MCP 对 `capabilities`、tool schema 和 roots 有直接协议表达；LangGraph 更强调 graph state schema 与 node / reducer 规则。跨对象抽象目前只能标为 `Inferred`：`TaskSpec` 不应只等同于 tool call JSON，也不应只等同于 graph input state。

### 3.2 `ExecutionSpec`

`ExecutionSpec` 描述任务如何运行、暂停、恢复和分支。第一版最小字段可以包括：

- `runtime`：tool protocol、graph runtime、agent runtime、sandbox 或 remote worker。
- `session_identity`：thread、run、conversation、workspace 或 external session 的身份边界。
- `state_model`：graph state、tool request state、memory、workspace artifacts 或 external store。
- `durability`：checkpoint、pending writes、replay、resume、fork 或无持久化。
- `control_signals`：cancel、progress、interrupt、approval、timeout、retry。

MCP 只提供 cancellation / progress / ping 等协议信号，不提供 workflow checkpoint；LangGraph 提供 workflow state checkpoint、pending writes、interrupt / resume 与 time travel，但不提供 workspace / process 恢复。因此 `ExecutionSpec` 必须分层表达：control signal、workflow durability 和 environment recovery 是三类不同能力。

### 3.3 `ResultSpec`

`ResultSpec` 描述执行结束或阶段性产物如何返回。第一版最小字段可以包括：

- `content`：面向模型或用户的自然语言 / 多模态结果。
- `structured_result`：可校验的结构化输出。
- `artifacts`：文件、patch、logs、workspace changes 或外部资源引用。
- `state_delta`：graph state update、tool result、memory write 或 checkpoint transition。
- `provenance`：result 来源、对应 run / checkpoint / tool call / node。

MCP 可以观察到 `content`、`structuredContent` 与 `outputSchema`；LangGraph 可以观察到 state delta、checkpoint metadata 和 pending writes。跨工具 business result normalization 仍是上层 orchestrator 责任，不能由 MCP 或 LangGraph 单独推出。

### 3.4 `ErrorSpec`

`ErrorSpec` 描述失败如何被分类、传播、恢复或终止。第一版最小字段可以包括：

- `error_type`：schema validation、tool failure、runtime failure、interrupt、timeout、permission denied、side-effect inconsistency。
- `retry_policy`：是否可重试、重试预算、backoff、幂等要求。
- `abort_policy`：取消请求、强制终止、资源释放和 response handling。
- `recovery_policy`：pending writes、checkpoint resume、manual state edit、compensation。
- `visibility`：错误是否进入 trace、state、result 或用户可见信息。

MCP 的 cancellation 是可选取消通知，不是强制 abort / rollback；progress 也不是 checkpoint。LangGraph 的 pending writes / replay 可以支持 workflow recovery，但不保证外部副作用恢复。因此 `ErrorSpec` 应明确区分“请求被取消”“workflow 可恢复”“外部世界已回滚”三种不同事实。

### 3.5 `AdapterSpec`

`AdapterSpec` 描述某个外部能力如何被 host / orchestrator 安全接入。第一版最小字段可以包括：

- `discovery`：能力列表、schema、版本与变更通知。
- `invocation`：调用协议、transport、streaming / progress 表达。
- `permission`：用户同意、风险等级、approval policy、token / auth scope。
- `state_bridge`：外部能力状态如何映射到 orchestrator state。
- `result_bridge`：tool result / agent result 如何归一到系统结果模型。
- `governance`：audit、cost、rate limit、cleanup、side-effect policy。

MCP 是当前最接近 `AdapterSpec` 的样本，但它只覆盖 protocol surface；LangGraph 可作为 adapter 之后的 workflow runtime 承接层。完整 `AdapterSpec` 仍需要继续对照 OpenAI Agents SDK、OpenHands、SWE-agent、CrewAI、AutoGen 等对象。

---

## 四、第一版停线

当前只能形成保守抽象：

- `TaskSpec` 不只是 prompt，也不只是 tool input；它需要同时描述目标、能力、schema 与边界。
- `ExecutionSpec` 不能把 progress / cancel 写成 checkpoint / resume；也不能把 graph checkpoint 写成 workspace recovery。
- `ResultSpec` 不能把 structured tool output 直接等同于跨系统 business result normalization。
- `ErrorSpec` 必须区分 cancellation、retry、checkpoint recovery 和 side-effect rollback。
- `AdapterSpec` 可以从 MCP 获得协议层证据，但 permission、audit、cost、cleanup 仍需要 host / runtime / product 实现补足。

这些维度是基于两个样本的 `Inferred` 研究框架，不是行业标准命名；后续每引入一个对象，都应先补对象内 evidence，再决定是否加强或修改契约维度。

---

## 五、后续核验顺序

优先补证对象：

1. `OpenAI Agents SDK / Responses API`：核验 session、run state、handoff、MCP、tracing、guardrails、sandbox agent 的边界。
2. `OpenHands`：核验 workspace、runtime、event log、sandbox、recovery 与 tool execution 的环境层契约。
3. `SWE-agent`：核验 trajectory、patch artifact、container / workspace 与 replay 边界。
4. `CrewAI` / `AutoGen`：核验 multi-agent coordination、flow persistence、handoff、retry / cancel 的实现边界。

每个对象进入横向比较前，必须保留对象内 `Observed` 证据；跨对象字段映射默认标为 `Inferred`，避免把单对象实现写成通用架构。

## Evidence

- Status: Observed / Inferred
- Sources:
  - `../04-skill-and-tool-systems/mcp/overview.md`
  - `../04-skill-and-tool-systems/mcp/notes/evidence.md`
  - `../01-frameworks/langgraph/checkpoint-and-persistence.md`
  - `../01-frameworks/langgraph/human-in-the-loop.md`
  - `../01-frameworks/langgraph/notes/evidence.md`
- Trace: 从 `agentic/backlog.md` 的 `2.5 Agent Adapter 与 Orchestrator 契约` 进入；`MCP` 和 `LangGraph` 分别提供协议层与 workflow runtime 层的已核验证据，本文件只做第一版契约维度抽象，后续对象细节仍留在各自对象目录。
- Needs:
  - 补 OpenAI Agents SDK / Responses API 的 session、run state、handoff、sandbox 与 tracing 证据。
  - 补 coding agent 项目中的 workspace、runtime、artifact、event log 与 recovery 契约。
  - 补 multi-agent framework 中 handoff、coordination、retry / cancel 与 result aggregation 边界。

*最后更新: 2026-06-09*
