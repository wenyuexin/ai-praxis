# Tracing and Observability

在 `OpenAI Agents SDK / Responses API` 当前已公开的能力面中，tracing 是证据最强、边界最清楚的一层之一。

## 一、当前可直接确认的 tracing 语义

官方 tracing 文档明确说明：

- tracing 默认启用。
- trace 表示一次 end-to-end workflow。
- span 表示 workflow 内部有开始和结束时间的操作。
- trace 具有 `workflow_name`、`trace_id`、可选 `group_id`、`metadata` 等属性。
- span 具有 `trace_id`、`parent_id`、`span_data`、开始/结束时间等属性。

这足以支持“OpenAI SDK 已把 workflow-level observability 作为正式一等能力面公开”这一判断。

## 二、默认追踪哪些操作

当前文档还直接列出了默认追踪范围：

- `Runner.run()` / `Runner.run_sync()` / `Runner.run_streamed()` 外层 trace
- agent run
- LLM generation
- function tool call
- guardrail
- handoff
- transcription / speech / speech group（音频场景）

这说明 tracing 并不是只包模型调用，而是覆盖 agent orchestration 中多个控制面事件。

## 三、`group_id` 的价值与边界

当前可直接确认：

- `group_id` 是可选字段，用于把多个 traces 关联到同一 conversation 等更高层实体。
- 官方示例明确提到可以把 chat thread ID 作为 `group_id`。

但当前不能进一步外推：

- `group_id` 不自动等于 session ID、conversation ID、checkpoint lineage 或 workspace lineage。
- tracing 文档没有因此自动证明 pause/resume、filesystem snapshot 或 workflow checkpoint 之间存在统一 lineage 模型。

## 四、tracing 与结果 / 状态 / session 的边界

当前更稳妥的理解是：

- tracing 回答的是“这次 workflow 发生了什么”。
- `results` 回答的是“这次 run 返回了哪些结果表面”。
- `sessions` 回答的是“多轮 conversation history 如何自动保存与续接”。

三者有关联，但不是同一件事。

因此不能把 tracing 文档里出现的 workflow / group_id 直接写成完整 persistence/recovery 契约的一部分。

## 五、当前仍需继续核验的问题

- tracing metadata 是否与 `RunState`、interruptions、approvals 有正式字段级关联。
- traces dashboard / external processors 如何表达多轮 continuation。
- MCP servers、tool context、sandbox agents 是否会写入专门 span_data 类型。
- tracing 与 server-managed continuation、session persistence 的最小关联面是什么。

## Evidence

- Status: Verified / Observed / Inferred
- Sources:
  - `https://github.com/openai/openai-agents-python/blob/main/docs/tracing.md`
  - `https://openai.github.io/openai-agents-python/tracing`
  - `https://developers.openai.com/api/docs/guides/agents`
- Trace: tracing 是当前 OpenAI 对象里最清楚的一层，因此先从对象总览中单独拆出机制专题，作为后续对照 `LangGraph`、`CrewAI`、`Semantic Kernel` observability 能力的稳定基线。
- Needs:
  - trace processors / external tracing processors 细页
  - traces dashboard 与多轮 continuation 的关系
  - span_data 类型与各运行时对象的映射
