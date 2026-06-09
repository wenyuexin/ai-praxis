# OpenCode Model and Provider Runtime

> 本文解释 OpenCode 如何接入模型、Provider、认证、模型目录、兼容转换和 LLM streaming。它补足 `agents-and-permissions.md` 没有覆盖的模型运行层。

## 核心判断

OpenCode 的模型层不是简单地把用户 prompt 传给某个 SDK。它有一套完整 Provider runtime：从模型目录、配置扩展、环境变量认证、已保存认证、插件 provider hook、动态 npm loader，到 provider-specific transform 和 AI SDK / native runtime 两条执行路径。

这意味着 OpenCode 的“模型能力”由四个部分共同决定：

- Provider catalog 与模型状态过滤。
- 认证来源和配置覆盖。
- Provider transform 对消息、参数、schema、variant 的兼容处理。
- LLM streaming runtime 对 AI SDK 或 native client 的选择。

## Provider 生命周期

OpenCode 的 Provider 信息主要由以下来源合并：

- `models.dev` 类模型目录。
- 用户配置中的 provider / model 扩展。
- 环境变量中的 API key 或 Provider 配置。
- 保存到 auth store 的 API / OAuth / wellknown 认证。
- 插件提供的 provider model / auth hooks。
- 自定义 npm provider loader。

Provider runtime 会把这些来源合并成可用 provider/model 列表，再按模型状态、实验开关、deprecated 状态、白名单/黑名单等规则过滤。

## 认证模型

OpenCode 支持多种认证形态：

- `api`：常见 API key。
- `oauth`：OAuth 认证。
- `wellknown`：从 well-known 配置或远程组织配置中获得认证线索。

认证既可以来自本地文件，也可以由 `OPENCODE_AUTH_CONTENT` 覆盖。插件也可以参与认证加载。这个设计使 OpenCode 能同时服务个人本地使用、企业默认配置和插件集成 Provider。

## 模型选择与状态

模型状态包括 `alpha`、`beta`、`deprecated`、`active`。默认情况下，alpha / deprecated 模型不会无条件暴露给用户；实验开关和配置会影响可见性。

默认模型选择也不是固定常量，而是按配置、最近使用模型、可用 Provider、模型排序偏好等顺序决定。ACP / UI 中的模型选择还会把 provider/model/variant 组织成可展示选项。

这说明 OpenCode 将“模型选择”拆成两层：

- **用户界面层**：哪些模型可选、如何展示 variant。
- **执行层**：最终如何解析 provider、auth、params 并发起流式调用。

## Provider Transform

Provider transform 是 OpenCode 模型层的关键兼容层。它不是简单清洗，而会影响：

- 消息格式和不兼容内容清理。
- Provider options 的命名空间和参数映射。
- schema 修正，特别是不同模型对 JSON schema 的支持差异。
- model variant / effort / reasoning 等扩展参数。
- max tokens、temperature、topP、topK 等默认值和 Provider 差异。

这使 OpenCode 可以在上层保留统一 agent/tool/session 抽象，同时在下层适配不同 Provider 的实际限制。

## LLM Streaming 路径

默认执行路径大致是：

1. session processor 组织模型请求。
2. `LLM.Service` 解析语言、配置、Provider、认证。
3. request prep 应用插件 hook 和 provider transform。
4. AI SDK `streamText()` 发起流式调用。
5. 返回流被转换为 OpenCode 内部 `LLMEvent`。

另有 experimental native LLM runtime，当前只覆盖部分 Provider 和认证形态。Native path 仍会复用 provider transform，并把请求降低到 `@opencode-ai/llm` 的 canonical request。

## 与 Agent Runtime 的关系

Agent 决定“以什么身份、带什么权限、用什么提示”执行；Provider runtime 决定“这个执行请求最终如何落到具体模型”。两者的连接点在 session processor 和 LLM request。

因此，分析一次 OpenCode 生成行为时，至少要同时看：

- 当前 agent / mode。
- 当前 model / provider / auth。
- provider transform。
- tool schema 与 permission。
- AI SDK 或 native runtime 分支。

## 设计意义

OpenCode 的模型层说明成熟 coding agent 不只需要 agent loop，还需要 Provider compatibility layer。这个层次决定系统能否长期跟随模型生态变化，而不让上层 agent/runtime 频繁重写。

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - `packages/opencode/src/provider/provider.ts`
  - `packages/opencode/src/provider/auth.ts`
  - `packages/opencode/src/provider/model-status.ts`
  - `packages/opencode/src/provider/transform.ts`
  - `packages/opencode/src/auth/index.ts`
  - `packages/opencode/src/acp/config-option.ts`
  - `packages/opencode/src/acp/directory.ts`
  - `packages/opencode/src/session/processor.ts`
  - `packages/opencode/src/session/llm.ts`
  - `packages/opencode/src/session/llm/request.ts`
  - `packages/opencode/src/session/llm/native-runtime.ts`
  - `packages/opencode/src/session/llm/native-request.ts`
- Trace: 本文补足前一版 OpenCode 案例遗漏的模型/Provider runtime 层，把 Provider catalog、auth、transform、LLM streaming 从 agent/tool/session 专题中拆出。
- Needs: 后续可补一个具体 Provider 请求例子，展示 transform 前后差异和 AI SDK / native path 分流。
