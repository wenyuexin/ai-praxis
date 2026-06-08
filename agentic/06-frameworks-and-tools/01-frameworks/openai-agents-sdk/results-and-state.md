# Results and State

`OpenAI Agents SDK / Responses API` 的 `results / state` 语义，当前最值得研究的不是“是否有结果对象”，而是它把哪些表面暴露给调用者，以及这些表面分别回答什么问题。

## 一、当前可直接确认的结果表面

根据官方 `Results` 与 `Running agents` 文档，`Runner.run(...)` / `Runner.run_sync(...)` 返回 `RunResult`，`Runner.run_streamed(...)` 返回 `RunResultStreaming`；两者共享 `RunResultBase` 暴露的结果表面。

当前可以直接确认的表面包括：

- `final_output`：用于读取最终输出；若最后一个 agent 定义了 `output_type`，则类型跟随该 agent。
- `new_items`：用于查看 run 期间新增的丰富事件项，包括 message、tool call、tool output、tool approval、handoff call、handoff output 等。
- `last_agent`：用于知道最后由哪个 agent 结束本轮运行，通常也是下一轮默认应继续接手的 agent。
- `raw_responses`：用于读取 run 期间的原始模型响应。
- `input_guardrail_results` / `output_guardrail_results`：用于查看 agent-level guardrail 结果。
- `interruptions` 与 `to_state()`：用于在审批或中断场景下保留可恢复快照；`Human-in-the-loop` 文档进一步说明 serialized run state 会带上 approvals、usage、serialized `tool_input`、nested `Agent.as_tool()` resumptions、trace metadata，以及 server-managed conversation settings。
- `to_input_list()`：用于把本轮历史转成下一轮可继续传回模型的输入列表。

这些字段说明 SDK 已经把“最终输出”“丰富事件”“下一轮历史”“中断恢复快照”“守护结果”分成了不同表面，而不是只返回单一字符串。

## 二、各结果表面回答的不是同一个问题

当前文档已经很明确地把几种结果表面分开：

- `final_output` 回答“最终要展示给用户的结果是什么”。
- `to_input_list()` 回答“下一轮要继续对话时，模型输入历史应该长什么样”。
- `new_items` 回答“这一轮实际发生了什么，包括 agent、tool、handoff、approval 边界”。
- `last_agent` 回答“哪一个 agent 最后完成了这一轮”。
- `interruptions` / `to_state()` 回答“如果运行因为审批或中断停下，如何继续”。

因此，`results / state` 不能被简单等同为“conversation history”或“run resume”某一种东西；它更像是面向不同调用需求暴露的多视图结果面。

## 三、当前能确认的中断 / 恢复边界

当前可以直接确认两点：

- `final_output` 在 run 尚未真正完成时可能为 `None`，例如因为 approval interruption 暂停。
- `interruptions` 与 `to_state()` 被文档明确用于 pending approvals 与 resumable snapshot。

这足以支持“SDK 存在 run-level interruption / resumable snapshot 表面”的保守判断。

但当前仍不能直接下更强结论：

- 还不能仅凭 `to_state()` 就断言存在完整 workflow checkpointing。
- 还不能断言这个 state 自动覆盖 workspace / filesystem / external side effects。
- 还不能仅凭 `results and state` 一页就把 resumable state 的完整生命周期写成稳定契约。

## 四、与 session / conversation continuation 的边界

`results / state` 与 session、`previous_response_id`、`conversation_id` 相关，但不应混写：

- `to_input_list()` 偏本地 next-turn history surface。
- `session=...` 偏 SDK 帮你自动存取 conversation history。
- `previous_response_id` / `conversation_id` 偏 OpenAI server-managed continuation。
- `interruptions` / `to_state()` 则更接近 run interruption / resume surface。

因此，当前更稳妥的理解是：OpenAI SDK 在 continuation 上至少同时暴露了本地历史续接、SDK session、OpenAI server-managed state、以及 interruption state 这几种不同层次，而不是单一 continuation 机制。

## 五、当前仍需继续核验的问题

- `to_state()` 返回的对象边界到底是什么，是否等同 `RunState`。
- resumable snapshot 保存的是 agent loop state、pending approvals、history，还是更广义的 execution state。
- `last_agent`、`new_items`、`raw_responses` 与下一轮 continuation 之间的最小必需关系是什么。
- interruption state 与 `session`、`previous_response_id`、`conversation_id` 是否可组合，以及组合时哪一层是权威状态源。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources:
  - `https://openai.github.io/openai-agents-python/results`
  - `https://openai.github.io/openai-agents-python/running_agents`
  - `https://openai.github.io/openai-agents-python/ref/run`
- Trace: 从 `openai-agents-sdk/overview.md` 中对 `results/state` 的保守总览继续下沉，单独把结果表面与恢复表面拆开，避免在对象总览里把 `final_output`、history continuation、interruption state 混成一层；更细 claim-source 对照与 stop-line 继续收敛到 `notes/evidence.md`。
- Needs:
  - `RunState` API reference 与源码
  - interruption / approval 恢复路径示例
  - `to_state()` 的字段级说明
  - `previous_response_id` / `conversation_id` 与 resumable state 的组合边界
  - `notes/evidence.md` 中关于 `RunState`、`session`、server-managed continuation 的下一轮源码级核验
