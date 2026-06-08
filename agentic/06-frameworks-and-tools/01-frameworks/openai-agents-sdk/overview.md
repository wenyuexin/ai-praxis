# OpenAI Agents SDK / Responses API Overview

> 本文件是 `OpenAI Agents SDK / Responses API` 的对象总览，先建立研究地图，不替代更深 SDK 文档与源码核验。
> Evidence 状态：当前混合 `Observed` / `Inferred` / `Unverified`；稳定部分主要限于官方 docs 已公开的模块入口与 tracing / handoff 文档，恢复边界仍待继续核验。

## 一、定位

`OpenAI Agents SDK / Responses API` 更像一组围绕 agent 运行、委托、控制、结果、工具与可观测性组织起来的开发接口，而不是单一的 graph runtime 或单一的 coding-agent system。

在当前仓库里，它的重要性主要不在于“是否已经形成完整统一契约”，而在于它把几组常被混写的能力同时公开到了同一对象中：handoffs、guardrails / human review、results and state、sessions、tools / MCP、integrations and observability、sandbox agents。

因此，本目录优先研究它如何组织 agent orchestration 的控制面与结果面，而不是先把它归纳成某种固定层级架构。

## 二、当前可直接确认的能力面

当前对象总览只保留第一层判断，具体机制已分别下沉到专题：

- orchestration surface 已公开：官方 docs 明确列出 `Orchestration`、`Guardrails`、`Results and state`、`Integrations and observability`、`Sandbox agents` 等模块入口。
- delegation surface 已公开：`handoffs` 是显式机制，并以 tool 形式暴露；详见 `handoffs-and-delegation.md`。
- result / persistence surface 已公开：`results/state` 与 `sessions` 都有独立入口，但两者边界不同；详见 `results-and-state.md` 与 `sessions-and-persistence.md`。
- integration surface 已公开：docs 中明确存在 `MCP and Connectors`，tools 文档也出现 `HostedMCPTool`；详见 `tools-and-mcp.md`。
- observability surface 已公开：tracing 文档直接定义了 trace / span / `group_id` 与默认 span 类型；详见 `tracing-and-observability.md`。

这些点当前可以支持“对象能力面存在且已分层公开”的判断，但还不能支持“这些能力已经组成稳定统一运行时契约”的更强结论。

## 三、当前最值得保留的 stop-line

现阶段更重要的是 stop-line，而不是扩写成强综述：

- docs 导航中出现某个模块，不等于它的底层语义已经被本仓库完全核验。
- `results and state`、`sessions`、`sandbox agents` 的入口存在，不自动等于 runtime/job resume、workspace recovery 或 external side-effect recovery 已被证实。
- handoff、tool、guardrail、session、trace 已可确认同属一个对象的公开能力面，但不应因此被压平成同一抽象层。
- “七层架构”若不是 OpenAI 官方文档或 SDK 文档中的原生命名，就不应写成对象定论。

## 四、当前可用的第一轮对象判断

在保守口径下，当前可先保留几条对象级判断：

1. `OpenAI Agents SDK / Responses API` 的公开文档把 orchestration、guardrails、results/state、observability、sandbox 等能力面显式并列出来，说明它不是单一 tool-calling 封装，而是试图覆盖更完整的 agent application surface。
2. 当前证据最清楚的机制层是 handoff 与 tracing；其中前者直接暴露 delegation，后者直接暴露 workflow-level observability。
3. `results/state`、`sessions`、`server-managed continuation`、`tools/MCP` 已经可以稳定分层讨论，但它们与完整恢复、workspace persistence、统一 adapter/runtime 契约之间仍有明显 stop-line。
4. `sandbox agents` 与 `RunState` 现在已经可以写成保守机制专题，但当前只足以支持 run-level resume surface、sandbox runtime/session/snapshot surface 这一层；更深的 run error handlers、MCP server lifecycle、workspace recovery 仍需继续补证。

## 五、与其他框架的关系

当前适合做的只是保守对照，不宜直接排位：

- 与 `LangGraph` 相比，`OpenAI Agents SDK / Responses API` 当前更容易直接确认 orchestration surface、handoff、guardrail、trace 等模块入口；但 graph checkpoint / thread / replay 这类 workflow state 语义，当前仍是 `LangGraph` 更清楚。
- 与 `AutoGen` 相比，OpenAI SDK 当前同样公开了 agent-to-agent delegation、tools 与 tracing，但两者的 state persistence、MCP adapter、workflow abstraction 不能直接并表。
- 与 `Microsoft Agent Framework` 相比，当前更不能把 OpenAI 的 `results/state` 或 `sessions` 自动等同于 MAF 的 workflow checkpointing。
- 与 `OpenHands`、`SWE-agent` 这类 coding agents 相比，OpenAI SDK 当前更像 orchestration/application framework，对 workspace/runtime/file artifact 的语义公开度没有那么直接。

## 六、当前仍需继续核验的问题

以下问题当前不应写成稳定结论：

- `RunState` 是否是官方 SDK 中稳定公开的一等状态对象，以及它的序列化 / 恢复边界。
- `results and state` 中的 resumable state 到底处于 conversation、run，还是更高层 workflow 粒度。
- `sessions` 保存的是 message history、run context，还是更强的 job-level continuation。
- `sandbox agents` 是否公开了 workspace persistence、snapshot、resume/recovery 的正式语义。
- run error handlers、MCP servers / manager、tool context 与 tracing 之间是否存在更强的统一控制契约。

这些问题的更细 claim-source 对照、stop-line 与下一轮源码级 checklist，现统一收敛到 `notes/evidence.md`，对象总览只保留第一层判断与阅读指路。

## 七、与环境层研究的关系

这个对象对 `agentic/05-environments/` 的最大价值，是帮助拆开几种容易被误写成同义词的恢复与控制语义：

- session history persistence
- resumable run state
- sandbox identity / runtime isolation
- workspace / filesystem recovery
- trace / observability lineage

当前更稳妥的做法，是把它作为环境层的补证对象，而不是把它当作已经给出完整恢复答案的主证据。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources:
  - `https://developers.openai.com/api/docs/guides/agents`
- `https://openai.github.io/openai-agents-python/handoffs`
- `https://openai.github.io/openai-agents-python/results`
- `https://openai.github.io/openai-agents-python/sessions`
- `https://github.com/openai/openai-agents-python/blob/main/docs/tracing.md`

- Trace: 由 `agentic/06-frameworks-and-tools/backlog.md` 中的 `OpenAI Agents SDK / Responses API` 候选对象条目，以及 `agentic/06-frameworks-and-tools/conflict.md` 中关于 `RunState` / `Session` / `SandboxAgent` / pause-resume / workspace recovery 的待核验问题回流而来；当前仅把能直接追到官方 docs / SDK 文档的对象边界写入总览。
- Needs:
  - `Results and state` 正文页
  - `Sessions` / session implementations
  - `Sandbox agents` 正文页
  - run error handlers / lifecycle / results 相关 SDK 文档
  - MCP servers / manager 与 tool context 相关文档
  - 更细 SDK 源码中的状态对象与恢复逻辑
