# OpenCode Ecosystem and Extensions

> 本文解释 OpenCode 的扩展生态：MCP、ACP、plugins、custom tools、SDK，以及它们各自扩展的是哪一层能力边界。

## 核心判断

OpenCode 的扩展生态不是单一插件机制，而是多种边界并存：

- **MCP**：把外部 tools/prompts/resources 接入模型可调用工具集合。
- **ACP**：把 OpenCode 暴露给编辑器/IDE 作为 agent protocol server。
- **Plugins**：在 OpenCode host 内部注册 hooks、tools、auth/provider 行为和 workspace adapter。
- **Custom tools**：以本地 tool 文件或 plugin tool 的方式扩展可调用动作。
- **SDK**：通过 HTTP API 自动化 OpenCode server，与外部应用集成。

这说明 OpenCode 的生态设计不是“所有能力都插件化”，而是按扩展对象分层：工具协议、IDE 协议、host hook、API client、UI/client integration 分别有不同边界。

## MCP：外部工具桥

MCP 配置支持 local stdio server 和 remote HTTP/SSE server，可配置 headers、OAuth、enable flag、timeout 等。运行时会连接 MCP server，发现 prompts、resources、tools，并将 MCP tools 暴露为模型可调用工具。

MCP tool 名称会带 server 前缀，进入 session tool assembly 后仍要经过 OpenCode 的 permission、truncation、plugin before/after hooks 等流程。

因此 MCP 的职责边界是：

- 扩展模型可调用工具集合。
- 通过协议接入外部能力。
- 不直接注册 OpenCode plugin hooks。
- 不直接改变 Provider runtime，除非通过其他机制配合。

## ACP：编辑器/IDE 协议适配

ACP 是面向 editor / IDE 的 agent protocol adapter。它实现 `@agentclientprotocol/sdk` 的 Agent 接口，把编辑器侧的 session、prompt、context、permission 等交互映射到 OpenCode SDK 和 server 能力。

ACP 也可以接收 client-provided MCP servers，并按 session 注册。这使 IDE 客户端可以把自身能力带入 OpenCode session。

ACP 的边界是协议适配：

- 它让 IDE/编辑器以标准协议驱动 OpenCode。
- 它代理 permission request 给 ACP client。
- 它不等同于 OpenCode plugin system。
- 它主要改变“客户端如何接入”，而不是“host 如何加载插件”。

## Plugin System：Host 级扩展

OpenCode plugin 是 host 内部扩展机制。公开 API 位于 `@opencode-ai/plugin`，支持 hooks、tools、auth/provider hooks、workspace adapter、TUI peers 等。

Runtime 会加载内置 auth/provider plugins，也会加载用户配置的外部 plugins。外部 plugin 可以来自 npm 或文件路径，并会经过 entrypoint、engine version 等校验。

Plugin 能扩展的边界明显比 custom tool 更宽：

- 修改 chat params、headers、messages、system prompt。
- 拦截 tool execution 和 permission。
- 添加 provider / auth 行为。
- 注册 custom tools。
- 注册 experimental workspace adapter。

## Custom Tools：窄扩展点

Custom tools 可以来自配置目录中的 `{tool,tools}/*.{js,ts}`，也可以由 plugin hooks 提供。它们最终会被包装成内部 `Tool.Def`，获得 worktree、directory、ask、truncation、telemetry 等上下文能力。

Custom tool 的边界比 plugin 窄：它主要提供一个可调用动作，包含 description、args schema 和 execute 逻辑；如果要影响消息、Provider、权限策略或生命周期，需要走 plugin hook。

## SDK：远程控制和自动化边界

`@opencode-ai/sdk` 是 HTTP API/client/server automation 边界。SDK 既可作为外部应用调用 OpenCode server 的方式，也被 ACP、plugins 或其他集成复用。

SDK 的职责不是定义 runtime，而是把 server API 变成可编程接口：

- 创建和调用 OpenCode client。
- 启动 OpenCode server。
- 封装生成自 OpenAPI 的 endpoint。
- 传递 workspace / directory 等上下文头或参数。

## 边界对照

| 扩展机制 | 主要扩展对象 | 能否加工具 | 能否影响 Provider/Auth | 能否影响客户端接入 | 典型用途 |
|---|---|---:|---:|---:|---|
| MCP | 外部工具协议 | 是 | 否 | 间接 | 接入外部工具、资源、prompt |
| ACP | IDE/编辑器协议 | 间接 | 否 | 是 | IDE agent client 集成 |
| Plugin | OpenCode host | 是 | 是 | 部分 | hooks、provider、auth、tools、workspace adapter |
| Custom tool | 单个工具动作 | 是 | 否 | 否 | 项目内工具或脚本动作 |
| SDK | HTTP API 自动化 | 否 | 否 | 是 | 外部系统控制 server/session |

## 设计意义

OpenCode 的扩展生态显示它不是把所有扩展都塞进一个 plugin abstraction，而是区分：

- 外部工具如何进入 agent loop。
- IDE 如何控制 agent。
- Host 如何被插件改写。
- 项目如何提供局部工具。
- 第三方应用如何通过 API 自动化。

这比单一“插件系统”更复杂，也更接近成熟 coding agent 平台的生态边界。

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - <https://opencode.ai/docs/tools>
  - <https://opencode.ai/docs/plugins>
  - <https://opencode.ai/docs/config>
  - <https://opencode.ai/docs/acp>
  - <https://opencode.ai/docs/agents>
  - `packages/core/src/v1/config/mcp.ts`
  - `packages/core/src/v1/config/config.ts`
  - `packages/opencode/src/mcp/index.ts`
  - `packages/opencode/src/session/tools.ts`
  - `packages/opencode/src/acp/agent.ts`
  - `packages/opencode/src/acp/service.ts`
  - `packages/opencode/src/acp/permission.ts`
  - `packages/opencode/src/plugin/index.ts`
  - `packages/opencode/src/plugin/loader.ts`
  - `packages/opencode/src/plugin/shared.ts`
  - `packages/plugin/src/index.ts`
  - `packages/plugin/src/tool.ts`
  - `packages/opencode/src/tool/registry.ts`
  - `packages/sdk/js/src/v2/client.ts`
  - `packages/sdk/js/src/v2/server.ts`
- Trace: 本文补足前一版 OpenCode 案例中被压缩到 context 文档里的扩展生态层，单独解释 MCP、ACP、plugins、custom tools、SDK 的职责边界。
- Needs: 后续可用一个真实 plugin 和一个 MCP server 例子验证扩展生命周期。
