# Tools and MCP

对 `OpenAI Agents SDK / Responses API` 来说，tool 与 MCP 当前可以写，但应该只写到“能力接入与执行表面”这一层，不能直接扩成完整 orchestrator 契约。

## 一、当前可直接确认的 tool 面

根据 tools 文档与官方 docs 导航，当前可以直接确认：

- SDK 明确支持 hosted tools、local runtime tools、function tools、agents as tools。
- hosted tools 包括 `WebSearchTool`、`FileSearchTool`、`CodeInterpreterTool`、`HostedMCPTool`、`ImageGenerationTool`、`ToolSearchTool`。
- function tools 可以由 Python 函数自动包装而成，SDK 会根据函数签名、docstring 和 schema 工具生成参数结构。
- `Agent.as_tool()` 提供了“让一个 agent 被另一个 agent 调用，但不发生 handoff”的路径。

这足以支持“tool 是 OpenAI SDK 中显式分层的能力接入面”这一判断。

## 二、MCP 当前能直接写到哪里

当前官方 docs 至少已经明确把 `MCP and Connectors` 放进同一知识体系中；工具文档也直接出现 `HostedMCPTool`，说明 MCP 在 OpenAI 体系中确实被当成正式工具接入路径之一。

这足以支持较保守的判断：

- MCP 在 OpenAI SDK 里不是外围话题，而是正式工具接入面的一部分。
- `HostedMCPTool` 表明 MCP 可以作为 hosted tool surface 接入。

但当前仍不宜在本对象正文里直接写更强结论，例如：

- OpenAI SDK 已经给出完整 MCP lifecycle 契约。
- MCP 在这里已经等同完整 agent adapter。
- MCP server / manager / tool context 的全部边界都已被当前轮次核验。

## 三、tool、handoff、agent-as-tool 的边界

当前这三者最需要分开写：

- 普通 tool：调用某种能力，拿回结果。
- handoff：以 tool 形式暴露的 delegation，把控制权交给另一个 agent。
- `Agent.as_tool()`：让另一个 agent 作为 tool 执行，但不转移控制权。

这说明 OpenAI SDK 并不是只提供“一个工具系统”，而是在同一调用表面下叠加了不同控制语义。

## 四、当前不能直接外推的内容

当前还不能把 tools / MCP 这一层写成：

- 完整的 tool governance contract
- 完整的 permission / trust / side-effect contract
- 完整的 MCP server lifecycle / cancellation / persistence contract
- 与 sessions / results / tracing 天然统一的总运行时模型

当前更稳妥的做法，是把 tools / MCP 保留在“capability surface”与“integration surface”这一层。

## 五、当前仍需继续核验的问题

- `HostedMCPTool` 与 docs 中 `MCP and Connectors`、SDK reference 中 `MCP servers / manager / util` 的关系。
- tool guardrails 在 hosted tools、function tools、agent-as-tool、MCP tools 之间是否完全一致。
- function tool、hosted tool、MCP tool 的错误处理与 schema 严格性边界。
- MCP tool context、server lifecycle、connect/cleanup、sampling 等更细控制面。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources:
  - `https://openai.github.io/openai-agents-python/tools`
  - `https://developers.openai.com/api/docs/guides/agents`
  - `https://openai.github.io/openai-agents-python/handoffs`
- Trace: 从对象总览中把 `tools / MCP` 从泛化的“能力面存在”下沉到一个独立机制专题，重点保留 tool、handoff、`Agent.as_tool()` 与 hosted MCP 的边界，而不提前扩写成完整 adapter/runtime 契约；更细 local MCP / hosted MCP stop-line 与恢复疑点继续收敛到 `notes/evidence.md`。
- Needs:
  - MCP servers / manager / util 参考文档
  - hosted MCP 与 local MCP 的边界
  - strict schema、tool errors、tool guardrails 的对象级对照
  - `notes/evidence.md` 中关于 cancellation、resume、reconnect 的后续核验
