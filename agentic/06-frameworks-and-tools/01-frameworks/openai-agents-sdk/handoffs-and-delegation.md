# Handoffs and Delegation

在 `OpenAI Agents SDK / Responses API` 当前已经能稳定写下来的机制里，handoff 是最清楚的一层之一，因为官方文档直接把它定义为 **一个 agent 把任务委托给另一个 agent** 的机制。

## 一、handoff 当前能直接确认什么

根据官方 handoffs 文档，当前可以直接确认：

- handoff 允许一个 agent 把任务转交给另一个 agent。
- handoff 在 LLM 侧被表示为 tool。
- 如果目标 agent 名为 `Refund Agent`，默认会暴露成类似 `transfer_to_refund_agent` 的 tool。
- 所有 agents 都有 `handoffs` 参数，可以直接接收另一个 `Agent`，也可以接收自定义 `Handoff` 对象。
- `Handoff` 支持 `tool_name_override`、`tool_description_override`、`on_handoff`、`input_type` 等自定义项。

这几条已经足够支持“handoff 是 SDK 官方公开的一等 orchestration 机制”这一结论。

## 二、handoff 与 tool 的关系

当前最值得保留的不是“handoff 很强”，而是它与 tool 的边界：

- handoff **被表示为 tool**，说明在 LLM 看来，它和其他可调用能力共享同一种表面机制。
- 但 handoff 的目标不是执行普通函数或返回普通工具结果，而是把控制权交给另一个 agent。
- 文档还明确给出对照：如果你只是想让一个 specialist 以结构化输入被调用、但不转移对话控制，应优先使用 `Agent.as_tool(parameters=...)`，而不是 handoff。

因此，当前更稳妥的结构是：

- tool = 一般能力调用表面
- handoff = 以 tool 形式暴露的 agent-to-agent delegation
- `Agent.as_tool()` = 不转移控制权时的 agent reuse 方式

这三者相关，但不应合并成同一个抽象层。

## 三、handoff 当前暴露了哪些控制面

当前文档已经说明 handoff 不只是“目标 agent 名字”这么简单，还至少暴露了几类控制面：

- 目标 agent：把任务交给谁。
- tool 名称与描述：LLM 如何识别这个 handoff。
- `input_type`：handoff tool-call 参数的结构。
- `on_handoff`：handoff 被触发时的回调逻辑。
- history shaping：可以通过 `input_filter`、`RunConfig.nest_handoff_history`、`RunConfig.handoff_history_mapper` 控制接收方看到什么历史。

这意味着 handoff 已经不仅是“路由”，而是带有输入整形与历史边界控制的 delegation surface。

## 四、当前不能直接外推的内容

当前还不能把 handoff 写成以下更强结论：

- 不能自动等同完整 multi-agent workflow engine。
- 不能自动等同 persistent workflow checkpointing。
- 不能自动等同 manager/worker 拓扑下的全部控制语义。
- 不能因为 handoff 被表示为 tool，就把 handoff、普通 function tool、MCP tool、guardrail 视为同一层对象。

文档能直接证明的是“delegation exists and is first-class”，不是“所有 orchestration semantics 都已统一”。

## 五、与其他机制的边界

当前最稳妥的分层是：

- handoff：把任务和控制权交给另一个 agent。
- `Agent.as_tool()`：把另一个 agent 当工具使用，但不转移控制权。
- guardrail：在输入、输出或工具调用前后做校验与阻断。
- session / results / tracing：分别负责 history persistence、结果表面和 observability。

这也是为什么 handoff 适合单独成文，而不应该只在总览里一笔带过。

## Evidence

- Status: Verified / Observed / Inferred
- Sources:
  - `https://openai.github.io/openai-agents-python/handoffs`
  - `https://developers.openai.com/api/docs/guides/agents`
  - `https://openai.github.io/openai-agents-python/tools`
- Trace: 从对象总览中把 handoff 从“模块入口存在”继续下沉为机制专题，重点保留它与普通 tool、`Agent.as_tool()`、history shaping 的边界，而不是扩写成完整 multi-agent 框架结论。
- Needs:
  - handoff history nesting / filtering 的更细示例
  - handoff 与 sessions / results / tracing 的联合行为
  - handoff 在复杂多 specialist 路由中的稳定模式
