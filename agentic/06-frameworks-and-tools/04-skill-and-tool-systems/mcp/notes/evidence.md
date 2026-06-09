# MCP Evidence Notes

本文件整理第一轮官方规范核验中的 claim-source 对照。它是研究辅助材料，不是面向读者的正文综述。

## 1. 对象定位

### Claim 1.1：MCP 是 LLM application 与外部 context / tool 集成协议

- Status: Observed
- Sources:
  - `https://modelcontextprotocol.io/specification/2025-06-18`
- Notes:
  - 官方 specification 将 MCP 描述为连接 LLM applications 与 external data sources / tools 的开放协议。
  - 官方结构区分 `Host`、`Client`、`Server`，并说明 protocol 由 base protocol、lifecycle、authorization、server features、client features 与 utilities 组成。
- Boundary:
  - 只能写成协议层定位；不能写成完整 workflow runtime 或 Agent orchestrator。

### Claim 1.2：MCP 使用 JSON-RPC 2.0 与 capability negotiation

- Status: Observed
- Sources:
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic`
- Notes:
  - Base protocol 要求 messages 遵循 JSON-RPC 2.0。
  - Requests / responses / notifications 有明确 ID、result、error 与 notification 边界。
- Boundary:
  - JSON-RPC message 边界不等于任务调度、workflow state 或恢复语义。

## 2. Tools 协议面

### Claim 2.1：MCP tools 支持 discovery 与 invocation

- Status: Observed
- Sources:
  - `https://modelcontextprotocol.io/docs/concepts/tools`
  - `https://modelcontextprotocol.io/specification/2025-06-18/server/tools`
- Notes:
  - Server 支持 tools 时必须声明 `tools` capability。
  - Client 通过 `tools/list` 获取工具列表，且 operation 支持 pagination / cursor。
  - Client 通过 `tools/call` 调用工具。
  - Tool definition 包含 `name`、可选 `title`、`description`、`inputSchema`、可选 `outputSchema`、可选 `annotations`。
  - Tools list 变化时，声明 `listChanged` 的 server 应发送 `notifications/tools/list_changed`。
- Boundary:
  - 这能支撑 tool discovery / invocation 结论。
  - 不能直接支撑完整 permission、retry、timeout、audit、side-effect governance 或 result normalization 结论。

### Claim 2.2：MCP tools 支持 structured output 但不定义跨工具业务结果模型

- Status: Observed / Inferred
- Sources:
  - `https://modelcontextprotocol.io/specification/2025-06-18/server/tools`
- Notes:
  - Tool result 可以包含 `content` 与 `structuredContent`。
  - Tool 可提供 `outputSchema`，server 必须返回符合 schema 的 structured result，client 应验证。
- Boundary:
  - `structuredContent` / `outputSchema` 是协议层结构化输出能力。
  - “不定义跨工具业务结果模型”是基于协议内容缺口的 Inferred stop-line，不是官方原文命名。

## 3. Client features

### Claim 3.1：Roots 暴露 filesystem boundary，但不等于 workspace recovery

- Status: Observed / Inferred
- Sources:
  - `https://modelcontextprotocol.io/specification/2025-06-18/client/roots`
- Notes:
  - Roots 允许 client 向 server 暴露 filesystem roots，server 可通过 `roots/list` 获取 root 列表。
  - Root 当前包含 `uri` 和可选 `name`，`uri` 必须是 `file://` URI。
  - Security considerations 要求 client 只暴露有权限的 roots，server 尊重 root boundaries。
- Boundary:
  - filesystem boundary 是 Observed。
  - “不等于 workspace snapshot / rollback / recovery”是 Inferred stop-line。

### Claim 3.2：Sampling 是 server 通过 client 请求 LLM sampling 的协议面

- Status: Observed
- Sources:
  - `https://modelcontextprotocol.io/specification/2025-06-18/client/sampling`
- Notes:
  - Sampling 允许 server 请求 client 发起 LLM sampling。
  - 该机制让 client 保持模型访问、选择与权限控制。
  - 官方 trust & safety 建议 human-in-the-loop，允许用户查看和编辑 prompt、review response。
- Boundary:
  - 只能支撑 nested LLM call / model access control surface。
  - 不支撑 server 自带模型权限、完整 agent loop 或 workflow runtime 结论。

## 4. Utilities 与控制信号

### Claim 4.1：Cancellation 是可选取消通知，不是强制 abort / rollback 保证

- Status: Observed / Inferred
- Sources:
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic/utilities/cancellation`
- Notes:
  - 任一方可发送 `notifications/cancelled`，引用此前同方向发出的、仍被认为 in-progress 的 request。
  - Receiver SHOULD stop processing、free resources、not send response。
  - Receiver MAY ignore unknown、completed 或 cannot be cancelled 的 request。
  - `initialize` request MUST NOT be cancelled by clients。
- Boundary:
  - cancellation notification 是 Observed。
  - “不是强制 abort / rollback / side-effect compensation”是 Inferred stop-line。

### Claim 4.2：Progress 是可选进度通知，不是 checkpoint / resume

- Status: Observed / Inferred
- Sources:
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic/utilities/progress`
- Notes:
  - Request metadata 可包含唯一 `progressToken`。
  - Receiver MAY 发送 `notifications/progress`，包含 progress、可选 total、可选 message。
  - Receiver MAY 不发送 progress；completion 后 progress notifications MUST stop。
- Boundary:
  - progress notification 是 Observed。
  - “不等于 checkpoint / resume / scheduler”是 Inferred stop-line。

### Claim 4.3：Ping 支持 liveness / connection health，不等于 recovery

- Status: Observed / Inferred
- Sources:
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic/utilities/ping`
- Notes:
  - 任一方可发送 `ping` request，对方必须及时返回空 response。
  - 未在合理 timeout 内收到响应时，sender 可认为 connection stale、terminate connection 或 attempt reconnection。
- Boundary:
  - liveness / stale connection handling 是 Observed。
  - conversation recovery、workflow state recovery、workspace recovery 都不能由 ping 直接推出。

## 5. Transport 与 authorization

### Claim 5.1：MCP 标准 transport 包括 stdio 与 Streamable HTTP

- Status: Observed
- Sources:
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic/transports`
- Notes:
  - stdio transport 中 client 将 server 作为 subprocess 启动，通过 stdin / stdout 交换 JSON-RPC messages。
  - Streamable HTTP 使用 HTTP POST / GET，可选 SSE，server 可发送 server-to-client notifications / requests。
- Boundary:
  - SSE / streamable transport 只能支撑消息通道层结论。
  - 不能自动推出 tool large output 分片、partial result semantics、workflow streaming checkpoint 或 resumable execution。

### Claim 5.2：Authorization 是 HTTP transport 层授权框架，不是完整 tool governance

- Status: Observed / Inferred
- Sources:
  - `https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization`
- Notes:
  - Authorization 规范定义 HTTP-based transports 的 transport-level authorization。
  - Protected MCP server 作为 OAuth 2.1 resource server，client 使用 access token 请求受限资源。
- Boundary:
  - transport-level authorization 是 Observed。
  - per-tool approval、risk classification、audit、permission policy、side-effect governance 仍属于 implementation / host 层，是 Inferred stop-line。

## 6. 当前正文 stop-line 自检

- Status: Inferred
- Sources:
  - `agentic/06-frameworks-and-tools/04-skill-and-tool-systems/mcp/overview.md`
- Notes:
  - 正文中对象定位、tools protocol、roots、sampling、cancellation、progress、ping、transport、authorization 的能力描述都有官方来源支撑。
  - 正文中“不是 orchestrator / runtime / recovery / full governance”的表述属于 stop-line 归纳，应保留为 Inferred，不应写成官方命名。
- Needs:
  - 后续对具体 MCP host / SDK 做实现对照。
  - 若正文继续增长，应把更细证据继续放在本文件，正文只保留收敛结论。

*最后更新: 2026-06-09*
