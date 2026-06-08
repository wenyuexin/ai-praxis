# Guardrails and Human Review

`OpenAI Agents SDK / Responses API` 当前对 guardrails 的公开程度已经足够支持一篇保守机制专题，因为官方文档直接定义了 input/output guardrails、tool guardrails、tripwire 与执行时机。

## 一、当前可直接确认的 guardrail 分类

官方 guardrails 文档明确区分了至少三类 guardrail：

- input guardrails
- output guardrails
- tool guardrails

当前可以直接确认：

- input guardrails 作用在 agent 执行前后的输入校验流程上。
- output guardrails 作用在最终 agent output 上。
- tool guardrails 绑定在 tool 本身上，会在每次 tool 调用前后执行。

这足以支持“guardrail 不是单一开关，而是按工作流边界分层配置”的判断。

## 二、tripwire 机制当前能证明什么

官方文档明确说明：

- 如果 input 或 output guardrail 触发 tripwire，会立即抛出 `InputGuardrailTripwireTriggered` 或 `OutputGuardrailTripwireTriggered`，并停止 agent execution。
- tool guardrails 可以在 tool 调用前后验证或阻断工具执行。
- 对于阻塞式 guardrail，若在 agent 启动前就触发 tripwire，可以避免 token 消耗与 tool side effects。

因此，当前已经可以稳定写下：guardrails 不只是“提示模型更安全”，而是显式的 workflow control / blocking surface。

## 三、执行时机的边界

当前文档最有价值的部分之一，是它没有把所有 guardrails 混成一层：

- output guardrails 只在 agent 真正成为最后输出者时运行。
- tool guardrails 每次 tool 调用都会运行。
- 如果 workflow 里有 managers、handoffs 或 delegated specialists，而你希望每次 function-tool call 都被检查，应使用 tool guardrails，而不只依赖 agent-level input/output guardrails。

这说明 guardrails 的粒度至少分为：

- agent 输入边界
- agent 最终输出边界
- tool 调用边界

## 四、human review 当前能如何写

在当前对象研究里，`human review` 最稳妥的写法不是扩成完整审批系统，而是把它视为与 guardrails / interruptions 相邻的一组控制面：

- 官方 docs 导航中明确存在 `Guardrails and human review` 模块。
- `results` 文档中明确出现 `interruptions` 与 pending approvals。
- `new_items` 中也直接出现 `ToolApprovalItem`。

这足以支持一个保守判断：OpenAI SDK 公开了 agent run 中与 approval / interruption 相关的控制面，并把它们与结果表面连接起来。

但当前仍不宜把它写成完整“审批状态机”或通用 HITL 契约。

## 五、当前不能直接外推的内容

当前还不能把 guardrails / human review 写成以下更强结论：

- 不能自动等同完整合规/风控系统。
- 不能自动等同 workflow checkpointing。
- 不能仅凭 `ToolApprovalItem`、`interruptions` 和 tripwire 就推出完整 human-in-the-loop 状态机。
- 不能把 guardrail、approval、handoff、tool 全部压平为一种统一控制原语。

当前能直接证实的是：这些控制面在 SDK 中被正式公开，并且 guardrails 至少有清楚的执行边界和阻断语义。

## 六、与其他机制的关系

当前更稳妥的对象分层是：

- handoff：agent-to-agent delegation
- tool：能力执行表面
- guardrail：在 agent input/output 或 tool 调用边界做校验与阻断
- approval / interruptions：run 暂停与继续的控制面
- tracing：记录这些控制面在 workflow 中实际发生了什么

这种分层有助于后续和 `LangGraph` 的 interrupt/HITL、`CrewAI` 的 human input、`Microsoft Agent Framework` 的 workflow control 继续对照。

## Evidence

- Status: Verified / Observed / Inferred
- Sources:
  - `https://openai.github.io/openai-agents-python/guardrails`
  - `https://openai.github.io/openai-agents-python/results`
  - `https://developers.openai.com/api/docs/guides/agents`
- Trace: 当前 guardrails 文档能直接支撑分类、tripwire、执行边界，因此先把它从总览中下沉为机制专题；其中 human review 仍只保留到 approval / interruption surface，不扩写成完整 HITL 契约。更细 approval、`ToolApprovalItem`、error handler 与 resumable state 的边界继续收敛到 `notes/evidence.md`。
- Needs:
  - `Guardrails and human review` 官方总览页的更细正文
  - approval / interruption 与 `to_state()` 的联动示例
  - tool guardrails 与 hosted tools / MCP tools 的具体边界
  - `notes/evidence.md` 中关于 terminal fallback、tool timeout、partial output 的后续核验
