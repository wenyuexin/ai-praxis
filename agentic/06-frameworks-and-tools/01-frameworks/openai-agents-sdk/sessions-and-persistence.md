# Sessions and Persistence

`OpenAI Agents SDK / Responses API` 中的 session 语义，当前最清楚的部分不是“完整持久化契约”，而是它明确负责 **conversation history persistence**，并提供多种可替换实现。

## 一、当前可直接确认的 session 角色

官方 `Sessions` 文档明确说明：session memory 用于跨多次 agent runs 自动维护 conversation history，避免开发者在每一轮之间手动处理 `.to_input_list()`。

当前可以直接确认：

- session 的核心职责是存 conversation history，而不是只做单轮结果缓存。
- session 针对特定 `session_id` 保存历史，使多轮对话可以自动延续上下文。
- SDK 提供多种 session 实现，包括 `SQLiteSession`、`AsyncSQLiteSession`、`RedisSession`、`SQLAlchemySession`、`MongoDBSession`、`DaprSession`、`OpenAIConversationsSession`、`OpenAIResponsesCompactionSession`、`AdvancedSQLiteSession`、`EncryptedSession`。
- `Running agents` 文档还把 session 与 `to_input_list()`、`conversation_id` / `previous_response_id` 并列，明确它们是不同 continuation 路径。
- `Sessions` 文档与 `Running agents` 文档都明确说明：session 不能与 `conversation_id`、`previous_response_id`、`auto_previous_response_id` 在同一个 run 中组合使用。

这足以支持“session 是 SDK 级的 conversation memory/persistence 抽象”这一结论。

## 二、session 解决的问题

当前文档能直接支持的，是以下较窄但稳定的判断：

- session 解决的是多轮对话历史的自动加载与保存。
- session 适合 chat applications 或 multi-turn conversations，这一点官方文档直接说明。
- session 让调用者不必在每轮都手动传 `result.to_input_list()`。
- 不同 `session_id` 代表彼此隔离的 conversation history。

因此，session 的“持久化”首先是 conversation persistence，而不是更广义的 workflow persistence。

## 三、当前不能直接外推的内容

当前还不能把 session 写成以下更强结论：

- 不能自动等同 run interruption state persistence。
- 不能自动等同 job-level continuation。
- 不能自动等同 workspace/filesystem persistence。
- 不能自动等同 tool side effects 或 external artifacts 的持久化。

目前抓到的文档重点都在 history/items/session store，而不是这些更强的运行时语义。

## 四、与其他 continuation 机制的边界

根据 `Running agents` 与 `Quickstart` 文档，当前至少存在三类 continuation 路径：

1. `result.to_input_list()`：本地手动续接历史。
2. `session=...`：SDK 自动管理 history。
3. `previous_response_id` / `conversation_id`：OpenAI server-managed continuation。

在此之外，`results` 文档中还出现了 `interruptions` / `to_state()` 这类与 pause/resume 更接近的状态表面。

因此更稳妥的结构是：

- session = conversation history persistence
- server-managed state = OpenAI 管理的 continuation surface
- interruption state = run-level resume surface

三者相关，但当前不应混写成同一种 persistence。

## 五、当前仍需继续核验的问题

- `OpenAIConversationsSession` 与 `conversation_id` 的边界和职责分工。
- `OpenAIResponsesCompactionSession` 如何改变 history surface，以及它对 traceability / replay 的影响。
- session 是否只保存 items/history，还是还保存 usage、branching、approval 等额外状态。
- session 与 `to_state()` 是否有正式组合路径，还是互相独立。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources:
  - `https://openai.github.io/openai-agents-python/sessions`
  - `https://openai.github.io/openai-agents-python/running_agents`
  - `https://openai.github.io/openai-agents-python/quickstart`
- Trace: 从对象总览中把 `sessions` 从泛化的“state”讨论里拆出来，单独确认为 conversation history persistence 层，避免与 interruption state、server-managed continuation、workspace persistence 混写；更细互斥关系、组合边界与 stop-line 继续收敛到 `notes/evidence.md`。
- Needs:
  - `OpenAIConversationsSession` / `OpenAIResponsesCompactionSession` 细页
  - session item schema 与 compaction 行为
  - session 与 approvals / interruptions / usage tracking 的关系
  - `conversation_id` 与 session 的官方对照说明
  - `notes/evidence.md` 中关于 `session_input_callback`、`OpenAIConversationsSession` 的后续核验
