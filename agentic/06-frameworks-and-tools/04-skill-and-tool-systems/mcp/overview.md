# MCP 协议边界核验

本文只核验 `Model Context Protocol`（MCP）官方规范能直接支持的协议层结论，用于拆清“工具协议层”与“完整编排运行时层”的边界；claim-source 对照见 `notes/evidence.md`。

---

## 一、MCP 的对象定位

MCP 是面向 LLM application 与外部 context / tool 集成的开放协议。官方规范把通信关系拆成：

- `Host`：发起连接的 LLM application。
- `Client`：host 内部维护的 MCP 连接器。
- `Server`：提供 context 与 capabilities 的服务。

协议底座是 JSON-RPC 2.0，并包含 lifecycle、capability negotiation、server features、client features 与 utilities。这个定位足以支持一个保守判断：MCP 是工具与上下文接入协议，不是 workflow / graph runtime，也不是完整 Agent orchestrator。

---

## 二、协议能直接确认的能力面

### 2.1 Server Features

MCP server 可以向 client 暴露三类主要能力：

- `resources`：供用户或模型使用的上下文与数据。
- `prompts`：模板化消息或 workflow 入口。
- `tools`：可由模型调用的函数或外部能力。

其中 tools 的最小协议面包括：

- server 必须声明 `tools` capability 才表示支持 tools。
- client 通过 `tools/list` 发现可用工具；该请求支持 pagination / cursor。
- client 通过 `tools/call` 调用工具。
- tool definition 包含 `name`、可选 `title`、`description`、`inputSchema`、可选 `outputSchema` 与可选 `annotations`。
- tool result 可以返回 `content`，也可以返回 `structuredContent`；提供 `outputSchema` 时，server 必须返回符合 schema 的 structured result，client 应验证。
- tools list 变化时，声明 `listChanged` 的 server 应发送 `notifications/tools/list_changed`。

这说明 MCP 对 tool discovery、tool invocation、schema 与结构化结果有明确协议表达。

### 2.2 Client Features

MCP 也允许 client 向 server 提供能力：

- `roots`：client 暴露 filesystem roots，server 可通过 `roots/list` 获取边界。
- `sampling`：server 可请求 client 发起 LLM sampling，由 client 保持模型访问、选择与权限控制。
- `elicitation`：server 可向用户请求额外信息。

这些能力说明 MCP 不是单向“工具列表协议”，而是 client / server 双向能力协商协议。但它们仍然是协议能力面，不等于完整系统运行时。

---

## 三、控制与治理信号的真实边界

### 3.1 Cancellation

MCP 支持可选 cancellation。任一方可发送 `notifications/cancelled`，引用同方向上此前发出的、仍被认为进行中的 request。

边界需要写窄：

- receiver 应停止处理、释放资源，并不再发送该 request 的 response。
- receiver 可以在 request unknown、已完成或不可取消时忽略 cancellation。
- sender 应忽略随后到达的 response。
- `initialize` request 不允许由 client 取消。

因此，MCP cancellation 是协议级“取消请求通知”，不是强制中断保证，也不是完整 abort / rollback / side-effect compensation 契约。

### 3.2 Progress

MCP 支持可选 progress notification。请求方在 request metadata 中提供唯一 `progressToken` 后，接收方可以发送 `notifications/progress`，包含 progress、可选 total 与可选 message。

边界需要写窄：

- progress token 只引用 active request。
- receiver 可以不发送 progress。
- progress 值必须递增，completion 后必须停止。

因此，progress 是长任务可观测信号，不等于 checkpoint、resume 或任务调度系统。

### 3.3 Ping 与连接健康

MCP 提供可选 `ping` request / response，用于确认对端是否仍响应、连接是否存活。未在合理 timeout 内收到响应时，sender 可以认为连接 stale、终止连接或尝试重连。

这只能支持 liveness / health-check 语义，不能直接推出 conversation recovery、workflow state recovery 或 workspace recovery。

### 3.4 Authorization

MCP authorization 规范定义的是 HTTP-based transports 的 transport-level authorization。受保护 MCP server 作为 OAuth 2.1 resource server，client 使用 access token 请求受限资源。

这说明 MCP 有 transport-level authorization 框架，但它不等于 tool permission policy、per-tool approval、side-effect governance 或组织级审计策略；这些仍需要 host / implementation 层补足。

---

## 四、Transport 与 streaming 边界

MCP 当前定义两种标准 transport：

- `stdio`：client 以 subprocess 启动 MCP server，通过 stdin / stdout 交换 JSON-RPC message。
- `Streamable HTTP`：server 作为独立进程，通过 HTTP POST / GET 与可选 SSE 传输 JSON-RPC message。

Streamable HTTP 可以使用 SSE 发送多个 server messages，支持 server-to-client notifications / requests。这个事实只能说明 MCP transport 能承载流式消息通道，不能自动推出任意 tool 的 large output 分片、partial result 语义、workflow streaming checkpoint 或 resumable execution。

---

## 五、不能外推成完整编排契约的部分

基于当前官方规范，MCP 不应被写成已经解决以下完整契约：

- **完整 tool governance**：协议要求 host 获得用户同意并建议 human-in-the-loop，但具体 UI、approval policy、risk classification、审计和权限隔离由实现负责。
- **强制 timeout / retry 策略**：规范有 ping、cancellation 与 progress，但没有定义统一 retry、backoff、timeout budget 或 idempotency contract。
- **workflow state / checkpoint**：协议不定义 graph state、thread state、checkpoint store、time travel 或 replay。
- **workspace recovery**：roots 描述 filesystem boundary，不等于 workspace snapshot、filesystem rollback 或 artifact recovery。
- **result normalization**：tool result 有 content / structuredContent / outputSchema，但跨工具、跨 server 的统一 business result model 仍需 host / orchestrator 设计。
- **external side-effect recovery**：协议不提供事务、补偿动作或外部副作用回滚语义。

更稳妥的表述是：MCP 标准化了 LLM application 与外部能力之间的 protocol surface；完整 orchestrator contract 仍需要在 host、framework 或上层 runtime 中定义。

---

## 六、对 Agent Adapter / Orchestrator 契约研究的意义

MCP 适合作为 `Agent Adapter / Orchestrator` 研究中的“协议层对照样本”：

- 它清楚覆盖 `capability discovery`、`tool invocation`、`schema`、`structured output`、`cancellation notification`、`progress notification`、`roots` 与 `sampling` 等协议面。
- 它也清楚暴露 stop-line：协议面存在，并不意味着完整 orchestration runtime 已存在。
- 因此，后续比较 LangGraph、OpenAI Agents SDK、AutoGen、CrewAI 等对象时，应避免用 MCP 的 tool surface 代替 workflow / runtime / state / recovery 语义。

---

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
- Trace: 本文从 `agentic/06-frameworks-and-tools/backlog.md` 的 MCP 核验队列进入，先按官方 `2025-06-18` specification 建立协议层 stop-line；正文中的对象定位和能力面为 `Observed`，对 Agent Adapter / Orchestrator 研究意义的归纳为 `Inferred`；更细 claim-source 对照下沉到 `notes/evidence.md`，正文只保留收敛后的边界结论。
- Needs:
  - 核验具体 MCP host / SDK 如何实现 tool approval、timeout、retry、streaming output、large output 与 audit。
  - 对照 LangGraph 的 checkpoint / workflow state，避免把 MCP cancellation / progress 误写成恢复机制。
  - 对照 OpenAI Agents SDK / Claude Code 等产品中的 MCP lifecycle，实现层结论不得回写为 MCP 协议本体结论。

*最后更新: 2026-06-09*
