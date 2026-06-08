# Server-Managed Continuation

`OpenAI Agents SDK / Responses API` 当前已经明确公开了一组 **OpenAI-managed server-side continuation** 机制，但它们与 session、`RunState`、workspace recovery 都不是同一层概念。

## 一、当前可直接确认的 continuation 方式

根据 `Running agents`、`Results`、`Sessions` 与 `Quickstart` 文档，当前至少可以直接确认四类续接方式：

1. `result.to_input_list()`：调用方手动把上一轮历史重新传回下一轮。
2. `session=...`：由 SDK 自动保存和加载 conversation history。
3. `conversation_id`：使用 OpenAI Conversations API 持有一个命名会话资源。
4. `previous_response_id`：使用 Responses API 的 response chaining 继续下一轮。

其中只有后两者属于 **OpenAI-managed server-side continuation**。

## 二、`conversation_id` 与 `previous_response_id` 的边界

当前官方文档能直接支持：

- `conversation_id` 与 `previous_response_id` 都是 OpenAI-managed continuation。
- `conversation_id` 适合需要命名 conversation resource、可跨系统共享会话资源的场景。
- `previous_response_id` 更像最轻量的 Responses API chaining primitive，用于从上一轮 response 继续下一轮。
- 二者互斥，不能在同一个 run 中同时使用。
- `last_response_id` 是 run 中最新的 model response ID，可在下一轮作为 `previous_response_id` 传回。

这足以支持“OpenAI 至少公开了两种 server-managed continuation primitive”这一结论。

## 三、与 session 的边界

当前最重要的不是“server-managed 很方便”，而是它与 session 的边界：

- `session` 是 SDK-managed conversation history persistence。
- `conversation_id` / `previous_response_id` 是 OpenAI-managed continuation。
- `Running agents` 与 `Sessions` 文档都明确说明：session persistence 不能与 `conversation_id`、`previous_response_id`、`auto_previous_response_id` 在同一个 run 中组合使用。

因此，当前更稳妥的结构是：

- client-managed history：`to_input_list()` / `session`
- server-managed history：`conversation_id` / `previous_response_id`

两类模式相关，但不是一套统一 persistence surface。

## 四、与 `RunState` 的边界

当前文档还能直接支持一条更细的判断：

- 如果 run 因 approval 暂停并从 `RunState` 恢复，SDK 会保留保存下来的 `conversation_id` / `previous_response_id` / `auto_previous_response_id` 设置，使恢复后的 turn 继续在同一个 server-managed conversation 中运行。

这说明：

- `RunState` 与 server-managed continuation 可以联动。
- 但 `RunState` 负责的是 run interruption / resumable state。
- `conversation_id` / `previous_response_id` 负责的是 OpenAI server-side conversation continuation。

因此二者是可组合的不同层，而不是同一对象。

## 五、当前不能直接外推的内容

当前还不能把 server-managed continuation 写成：

- 完整 workflow checkpointing。
- workspace/filesystem persistence。
- tool side effects 的自动恢复。
- 与 session 完全等价的 persistence contract。

当前能直接证实的是“OpenAI 负责保留某一层会话续接状态”，而不是更广义的运行时恢复。

## Evidence

- Status: Verified / Observed / Inferred
- Sources:
  - `https://openai.github.io/openai-agents-python/running_agents`
  - `https://openai.github.io/openai-agents-python/results`
  - `https://openai.github.io/openai-agents-python/sessions`
  - `https://openai.github.io/openai-agents-python/quickstart`
- Trace: 基于 `sessions-and-persistence.md` 中已经确立的 continuation 分层，把 OpenAI-managed 这一支单独抽出，避免在 session 或 `RunState` 文档里混写 `conversation_id` / `previous_response_id`。
- Needs:
  - `auto_previous_response_id` 的更细行为说明
  - `conversation_locked` retry 的边界与一致性语义
  - server-managed continuation 在 handoff / nested agent-as-tool 场景下的精细行为
  - `OpenAIConversationsSession` 与 `conversation_id` 的对象关系
