# DeepSeek Harness agent-loop：turn/step 流程

> 版本锚点：`0.1.0-rc.7`（commit `99f6f02fecdb`，2026-08-17）。本文基于本地 checkout 源码核验（`Status: Observed`，锚点见 `notes/evidence.md` C11）；官方口径序图见 `notes/source.md` 1.5。

`ReactLoopAgent` 是 dsh 的唯一具体 agent driver（`ctx.agentLoop`，bundle 角色）。**step** 是一次 model request 加它调用的 tools；**turn** 是零或多个 step——它在第一个输入被 claim 时打开，在无所欠时关闭。所有请求都从 session log 派生（`deriveMessages()` 投影模型历史）。

## 1. driver 状态机

driver 有三个相位：`idle`（空闲）、`running`（驱动 turn 循环）、`maintenance`（非模型工作，如注入/清理）。唤醒（wake）在 idle 时开启 running 相位并经 `ctx.agents.withInitiator` 执行驱动循环 `while (await this.turn()) {}`；运行中被唤醒的输入会 latch，待当前 turn 收敛后重放。

## 2. turn 边界

```
turn/start
  preStep: inbox.claim + systemPrompt.assemble + runtimeContext.project
    → agent/pre-step waterfall（可 reject 或改写消息）
  step/start + user/message*（追加进日志）
  step()  ← 见下
  step/end（finally 保证）
  若 turn 结束且无 next-step 输入 → agent/turn-stopping（serial）
turn/end（带 reason，必达）
```

- `turn/end` 的 reason 枚举：`blocked`（pre-step 拒绝）、`completed`、`max-tokens`（粘性：一旦某 step 触顶，后续正常完成的 step 不得降级）、`aborted`（取消）、`error`（结构化：`LlmError` 保留事实，其余折叠为 `UNKNOWN`）。
- 被移除的唤醒消息或被改写为空的 enter 决策仍拥有初始 turn 边界，但不花任何模型调用（日志记录这次尝试）。
- turn 结束时若 inbox 仍有 pending 输入，换新 AbortController 开启下一 turn。

## 3. step 边界

```
buildRequest（见下）
stream: preparedCall.stream ?? ctx.llm.stream
  assistant/chunk* 逐块追加（带 seq）
assembler 收束 → finish
  error/aborted → agent/request-error waterfall（可决策 retry）→ 不重试则抛 LlmError
assistant/message 追加（surfaceOp append + sourceEventSeqs）
无 tool-call → completed
有 tool-call → executeToolCalls（经 ctx.tools 的 TOOL_RUNTIME_SCHEDULER 管线）
  concluded → completed；否则继续下一轮请求（tools 欠一次请求）
```

## 4. 请求组装（buildRequest）

1. 从 session 恢复持久化的 `request/header`（首个请求来自 `AgentOptions.provider/model`）。
2. **`agent/request` waterfall** 提出 config（插件可改写 provider/model 等）；缺 provider/model 抛错。
3. `ctx.llm.prepareCall` 解析适配器与模型默认（`NO_ADAPTER` 时 middleware 可自服务）。
4. `canonicalHeader` 组装 config + adapterDefaults + system + tools，作为 **`request/header` session 事件** 追加（reason：initial/resume/change——header 变化才追加）。
5. provider/model/contextWindow 变化时追加 **`request/context` session 事件**。
6. 冻结最终 request（`markAgentLoopRequest`），messages 来自 `session.deriveMessages()`。

## 5. 输入通道

- `send`（用户输入，可唤醒）、`followup`（next-turn + 唤醒）、`steer`（next-step + 唤醒）、`inject`（next-step + **不唤醒**——注入的上下文在 inbox 等待直到其他消息唤醒，与"model-visible ⟺ logged"配合：注入内容也要走 session 事件）。
- 取消（`cancel`）经 AbortController 传导：可保留或清空 inbox；`agent/status` 事件发布状态迁移。

## 6. 扩展点（waterfall 事件清单）

| 事件 | 模式 | 作用 |
|---|---|---|
| `agent/pre-step` | waterfall | 决定模型看到什么：改写/拒绝进入 |
| `agent/request` | waterfall | 提出/改写请求 config |
| `agent/request-error` | waterfall | 流失败/中止时的重试决策 |
| `agent/turn-stopping` | serial | turn 收尾拦截 |
| `tools/pre-execute` / `tools/execute` / `tools/post-execute` | waterfall | 工具执行管线（超时包装、审批、重复提醒挂这里） |
| `llm/stream` | waterfall | 模型流（"Model-visible ⟺ logged" invariant 在此校验） |

## 7. 与官方序图的两处差异（文档未覆盖的细化）

1. **`agent/request-error` waterfall**：architecture.md 序图未画出，源码中流失败/中止时经它决策重试。
2. **`request/header` / `request/context` session 事件**：请求配置与上下文的持久化日志（initial/resume/change 语义），序图未列出。

## Evidence

- **Status**: 本文件全部结论为 `Observed`（本地 checkout `0.1.0-rc.7` 源码直接核验，锚点见 `notes/evidence.md` C11；事件实现 C4/C7）。
- **Sources**: `packages/core/agent-loop/src/agent.ts`（全文）；`packages/core/agent-loop/src/{tool-calls,runtime-context,invariant}.ts`。
- **Trace**: 2026-08-21 由 ai-praxis 侧深研脚手架回流提炼（生命周期层压实后）；版本锚点 `0.1.0-rc.7`。
- **Needs**: `agent/request-error` 与 compaction（request-error recovery 事件）的联动未核验；若切换版本锚点，复查 agent-presets/acp 相关改动（`notes/source.md` 5.2）。
