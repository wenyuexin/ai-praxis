# Sandbox Agents

`OpenAI Agents SDK / Responses API` 当前已经把 `SandboxAgent`、`SandboxRunConfig`、manifest、snapshot、session state、sandbox capabilities 公开为正式对象面，但这些公开对象不自动等于“已证实完整 workspace recovery”。

## 一、当前可直接确认的对象面

根据 `v0.14.0` release notes 与 sandbox API reference，当前可以直接确认：

- `SandboxAgent` 是一类带有 sandbox-specific configuration 的 `Agent`。
- runtime transport details（如 sandbox client、client options、live session）不存放在 agent 本体上，而是在运行时通过 `RunConfig(sandbox=...)` 提供。
- `SandboxRunConfig` 至少公开了 `client`、`options`、`session`、`session_state`、`manifest`、`snapshot` 等字段。
- `session_state` 被说明为：在不使用 `RunState` payload 时，用于 resume 的显式 sandbox session state。
- `snapshot` 被说明为：可选的 sandbox snapshot，用于 fresh session creation。
- `v0.14.0` release notes 明确把 sandbox agents、workspace manifests、sandbox-native capabilities、sandbox clients、snapshots、resume support 写成正式新增能力。

这足以支持“OpenAI SDK 已公开 sandbox runtime/session 相关对象面与恢复入口”这一结论。

## 二、manifest、session_state、snapshot 当前分别能写到哪里

当前更稳妥的理解是：

- `manifest` 更接近 fresh workspace / session creation surface。
- `session_state` 更接近 sandbox session resume surface。
- `snapshot` 更接近 fresh session creation 时可选加载的 sandbox snapshot surface。

但当前公开资料尚不足以证明：

- snapshot 一定等于完整 workspace filesystem snapshot。
- `session_state` 一定完整覆盖所有 workspace 改动。
- 不同 sandbox client / hosted provider 对 snapshot 与 session_state 的行为完全一致。

因此，这三个对象当前都应被写成 **正式公开的配置与状态入口**，而不是已经完全闭合的恢复语义。

## 三、capabilities 当前能直接证明什么

`SandboxAgent` 的 capabilities 当前可确认是正式对象面的一部分。release notes 明确提到 sandbox-native capabilities，并列出 shell access、filesystem editing、skills、memory、compaction 等方向。

这足以支持：

- sandbox agents 不是普通 tool-calling agent 的小变体，而是把 workspace-oriented capability surface 正式带入 SDK。

但当前不能直接推出：

- 每种 capability 在 resume 时都有一致恢复语义。
- memory / compaction 与 `RunState` / snapshot / session_state 的组合关系已被完整定义。

## 四、当前最值得保留的 stop-line

- `SandboxAgent` 的官方存在性已经足够稳定，可以进入对象正文。
- `SandboxRunConfig.session_state` 与 `snapshot` 的字段存在性也足够稳定。
- 但“字段存在”不等于“完整恢复语义已被公开证明”。
- 目前最稳妥的结论，是 SDK 已公开 sandbox runtime / session / snapshot surface，而完整 workspace filesystem recovery 仍应保留为待核验问题。

## 五、与其他恢复层的关系

当前应明确区分：

- `RunState`：run interruption / resumable state
- `session=...`：conversation history persistence
- `conversation_id` / `previous_response_id`：OpenAI-managed continuation
- `SandboxRunConfig.session_state` / `snapshot`：sandbox runtime/session 层的恢复入口

这几者相关，但不应压成一个统一 state surface。

## Evidence

- Status: Verified / Observed / Inferred
- Sources:
  - `https://github.com/openai/openai-agents-python/releases/tag/v0.14.0`
  - `https://openai.github.io/openai-agents-python/ref/sandbox`
  - `https://developers.openai.com/api/docs/guides/agents`
- Trace: 由对象总览中对 `sandbox agents` 的入口级判断继续下沉，并结合 `14.md` 的清洗后核验结果，只保留 release notes 与 API reference 可以直接支撑的对象面和 stop-line。
- Needs:
  - `SandboxSessionState` / `SnapshotSpec` 字段级说明
  - 各 sandbox client / hosted provider 的行为对照
  - snapshot 与 workspace file changes 的捕获边界
  - `session_state` 与 `RunState` 的组合恢复语义
