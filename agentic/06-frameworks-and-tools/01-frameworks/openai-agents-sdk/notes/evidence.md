# OpenAI Agents SDK Evidence Notes

> 本文件用于整理 `OpenAI Agents SDK / Responses API` framework study 中尚未适合直接写成主线结论的 Claim / Status / Sources / Needs 对照。
> 当前状态：已建立第一轮恢复语义与深水区 checklist，用于支撑 `run-state-and-resume.md`、`server-managed-continuation.md`、`sandbox-agents.md` 以及对象 README / overview 中的 Evidence 段落。

## Claim Tracker

### 1. `RunState` 提供 run-level interruption / resume surface

- Status: Observed
- Sources:
  - `https://openai.github.io/openai-agents-python/ref/run`
  - `https://openai.github.io/openai-agents-python/results`
  - `https://openai.github.io/openai-agents-python/human_in_the_loop`
- Notes:
  - `RunResultBase.to_state()`、`RunState.from_json(...)` / `from_string(...)`、`interruptions` 与 approval interruption 已有公开入口。
  - 当前足以支持“run 被中断后可序列化并恢复”的保守结论。
- Needs:
  - `RunState` 字段级结构
  - `_current_step` 与 partial tool execution 的恢复边界
  - schema version compatibility 与跨版本恢复策略

### 2. `RunState` 不应直接写成完整 durable workflow checkpoint

- Status: Inferred
- Sources:
  - `run-state-and-resume.md`
  - `https://openai.github.io/openai-agents-python/human_in_the_loop`
- Notes:
  - 当前公开资料重点覆盖 approval / HITL interruption resume。
  - 尚无足够公开证据把它上推成 workspace filesystem recovery 或 external side-effect recovery。
- Needs:
  - 已完成 tool call 在 resume 后是否重放
  - cross-process / cross-machine 恢复语义
  - rollback / cancel / retry 是否存在独立正式控制面

### 3. `conversation_id` / `previous_response_id` 属于 OpenAI-managed continuation

- Status: Observed
- Sources:
  - `https://openai.github.io/openai-agents-python/running_agents`
  - `https://openai.github.io/openai-agents-python/results`
  - `https://openai.github.io/openai-agents-python/quickstart`
- Notes:
  - 当前已可稳定区分 `to_input_list()`、`session`、`conversation_id`、`previous_response_id` 四类续接 surface。
  - `conversation_id` / `previous_response_id` 应单独放在 server-managed continuation 层讨论。
- Needs:
  - `auto_previous_response_id` 的更细行为
  - handoff / nested agent-as-tool 场景下的 continuation 细节
  - `conversation_locked` retry 与一致性边界

### 4. `session` 与 server-managed continuation 互斥

- Status: Observed
- Sources:
  - `https://openai.github.io/openai-agents-python/sessions`
  - `https://openai.github.io/openai-agents-python/running_agents`
- Notes:
  - 当前官方文档已明确 `session` 不与 `conversation_id`、`previous_response_id`、`auto_previous_response_id` 在同一个 run 中组合使用。
  - 这足以支持“client-managed history”与“server-managed continuation”分层讨论。
- Needs:
  - `session_input_callback` 与 `RunState` 组合时的输入合并顺序
  - `OpenAIConversationsSession` 与 `conversation_id` 的对象关系

### 5. `SandboxAgent`、`session_state`、`snapshot` 是正式公开对象面

- Status: Observed
- Sources:
  - `https://github.com/openai/openai-agents-python/releases/tag/v0.14.0`
  - `https://openai.github.io/openai-agents-python/ref/sandbox`
- Notes:
  - `SandboxRunConfig` 已公开 `client`、`session`、`session_state`、`manifest`、`snapshot` 等字段。
  - 当前足以支持 sandbox runtime / session / snapshot surface 已公开，但不足以闭合完整恢复语义。
- Needs:
  - `SandboxSessionState` / `SnapshotSpec` 字段级说明
  - snapshot 存储位置与生命周期
  - fresh session creation 与 resume 的精确分工

### 6. sandbox resume 不应直接写成完整 workspace filesystem recovery

- Status: Inferred
- Sources:
  - `sandbox-agents.md`
  - `https://github.com/openai/openai-agents-python/releases/tag/v0.14.0`
- Notes:
  - 当前只能稳定写到 manifest / session_state / snapshot 的存在性与保守边界。
  - snapshot 是否完整捕获 workspace changes、是否跨 provider 一致，公开资料仍不足。
- Needs:
  - snapshot 对 workspace files / mounts / remote storage 的捕获边界
  - local / docker / hosted sandbox client 的兼容性与差异
  - `session_state` 是否只记录 session metadata 或也覆盖文件改动

### 7. local MCP 与 hosted MCP 应分模式处理

- Status: Observed / Inferred
- Sources:
  - `tools-and-mcp.md`
  - `https://developers.openai.com/api/docs/guides/agents`
- Notes:
  - 当前能稳定确认 SDK 存在 tools / MCP surface，以及 `HostedMCPTool` 这类对象入口。
  - 但 local MCP、hosted MCP、tool lifecycle、resume、cancellation propagation 不宜压成单一恢复语义。
- Needs:
  - cancellation 是否传播到 MCP server / tool call
  - hosted MCP continuation / recovery 语义
  - MCP server reconnect 后的状态保持范围

### 8. run error handlers 不能写成统一 retry / recovery contract

- Status: Observed / Inferred
- Sources:
  - `guardrails-and-human-review.md`
  - `run-state-and-resume.md`
- Notes:
  - 当前主线正文只宜保留“存在结构化错误类型与有限 error handlers”的判断。
  - tool timeout、partial output、rollback、terminal fallback 与 retry policy 仍需继续拆分。
- Needs:
  - `ToolTimeoutError` 是否携带 partial result
  - `RunErrorDetails` 的字段边界
  - tool-level retry policy / cancel contract 的公开入口

## 恢复语义分层提醒

- conversation history persistence：`to_input_list()` / `session`
- server-managed continuation：`conversation_id` / `previous_response_id`
- run interruption / resumable state：`RunState`
- sandbox runtime / session resume：`SandboxRunConfig.session_state` / `snapshot`
- workspace / filesystem recovery：当前仍不能仅凭上述 surface 直接写成已证实结论
- external side-effect recovery：当前未见可直接支撑的公开结论

## 下一轮源码级 checklist

- `RunState` 的 `_current_step`、pending tool call、partial result 是否进入可恢复状态
- `RunState` schema version 升级时的兼容与迁移策略
- `Runner.run(...)` 在 `RunState` 恢复后是否重放已完成 tool calls
- `SandboxSessionState` 的序列化范围是否覆盖 workspace file changes
- snapshot 的存储位置、provider 兼容性与 remote storage 协同边界
- local MCP / hosted MCP 的 cancellation、resume、reconnect 语义
- `session_input_callback` 与 `RunState` / session history 的合并顺序
- run error handlers、tool timeout、partial output、retry / cancel 的正式控制面

## 使用说明

- 当某个 Claim 获得官方文档或源码直接支持时，可升级为 `Observed`，并同步回填对应专题正文。
- 当某个 Claim 仍主要依赖 stop-line 或边界判断时，保持 `Inferred`，不要直接抬升为对象总览定论。
- 当不同文档、release notes、reference 或源码之间出现口径冲突时，应改写为 `Conflicting` 并迁移到更细 notes，而不是硬写进正文。
- 正文中的 `## Evidence` 应优先摘要引用本文件中的稳定结论，而不是要求读者从 `agentic/temp/web-search/14.md` 还原论证过程。
