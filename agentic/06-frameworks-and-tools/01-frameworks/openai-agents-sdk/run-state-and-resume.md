# Run State and Resume

`OpenAI Agents SDK / Responses API` 当前已经公开了一条相对清晰的 **run-level interruption / resume** 路径，但这条路径不应被直接写成完整 workflow checkpoint 或 workspace recovery。

## 一、当前可直接确认的入口

根据 `Runner` API reference、`Results` 与 `Human-in-the-loop` 文档，当前可以直接确认：

- `Runner.run(...)` 的 `input` 参数可以直接接收 `RunState`。
- `RunResultBase` 暴露 `to_state()`，用于把当前运行结果转成可恢复状态。
- `interruptions` 用于暴露待处理的 approval / interruption。
- `Human-in-the-loop` 文档明确说明：工具可因 approval 暂停执行，随后通过 `RunState` 序列化、持久化并在后续恢复。
- `RunState` 提供 `to_json()` / `to_string()` 与 `from_json(...)` / `from_string(...)` 这类持久化与恢复入口。

这足以支持“SDK 已公开 run-level pause/resume surface”这一结论。

## 二、当前能确认的恢复语义

当前官方资料能直接支持的恢复语义主要包括：

- approval interruption 发生后，run 可以暂停。
- 暂停后的状态可以被序列化并在之后恢复。
- 恢复时可以继续同一个外层 run，而不仅是重新发起一轮新对话。
- `Human-in-the-loop` 明确说明 serialized run state 不只包含 app context，还包含 approvals、usage、serialized `tool_input`、nested `Agent.as_tool()` resumptions、trace metadata，以及 server-managed conversation settings。

因此，当前更稳妥的说法是：`RunState` 是一个 **run-wide resumable state surface**，尤其适合 approval / HITL 场景。

## 三、当前不能直接外推的内容

当前还不能把 `RunState` 写成以下更强结论：

- 不能自动等同完整 durable workflow checkpoint。
- 不能自动等同 workspace filesystem snapshot。
- 不能自动等同外部副作用（数据库写入、API 调用、邮件发送等）的可回滚恢复。
- 不能仅凭 `to_state()` 的存在，就断言 SDK 已统一定义 retry / cancel / rollback contract。

当前能直接证明的是 run interruption 后的可恢复状态，而不是所有运行时层面的全面恢复。

## 四、与其他 continuation 机制的边界

`RunState` 最容易和其他 continuation surface 混写，当前应明确区分：

- `to_input_list()`：本地 next-turn history surface。
- `session=...`：SDK-managed conversation history persistence。
- `conversation_id` / `previous_response_id`：OpenAI-managed server-side continuation。
- `RunState`：run interruption / resumable state。

官方 `Running agents` 文档还明确说明：如果 run 因 approval 暂停并从 `RunState` 恢复，SDK 会保留保存下来的 `conversation_id` / `previous_response_id` / `auto_previous_response_id` 设置，使恢复后的 turn 继续在同一个 server-managed conversation 中运行。

这说明 `RunState` 可以与 server-managed continuation 产生组合，但两者不是同一层机制。

## 五、当前最值得保留的 stop-line

- `RunState` 回答的是“如何继续一个被中断的 run”。
- 它不是当前已证实的 workspace recovery 机制。
- 它也不是当前已证实的 external side-effect recovery 机制。
- 即使官方说它是 durable，也应先理解为 **durable run state for interruption/resume**，而不是自动外推到更广义的 workflow/runtime/filesystem durability。

## Evidence

- Status: Verified / Observed / Inferred
- Sources:
  - `https://openai.github.io/openai-agents-python/ref/run`
  - `https://openai.github.io/openai-agents-python/results`
  - `https://openai.github.io/openai-agents-python/human_in_the_loop`
- Trace: 由对象总览与 `results-and-state.md` 中对 `interruptions` / `to_state()` 的保守表述继续下沉，单独把 run-level resume surface 抽成专题，避免与 session、server-managed continuation、sandbox recovery 混写。
- Needs:
  - `RunState` 独立 API reference / 源码字段级说明
  - interruption 恢复与 partial tool execution 的边界
  - `RunState` 与 sandbox session state 的组合关系
  - retry / cancel / rollback 是否另有正式控制面
