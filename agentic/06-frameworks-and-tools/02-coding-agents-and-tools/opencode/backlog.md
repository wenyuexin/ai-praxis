# OpenCode Backlog

> 本文件记录 `anomalyco/opencode` 已识别但尚未完全覆盖的研究缺口。已升级为正文或机制专题的内容保留轻量 trace，避免重复维护完整分析。

## 已回流正文的研究缺口

### 多入口架构与共享后端

- Status: 已回流。
- Output: `architecture.md`
- Trace: 基于官方 CLI 文档与源码入口核验，确认主 TUI、`serve`、`web`、desktop sidecar、remote attach 等入口围绕 server/session/tool/permission 后端工作；同时保留 `packages/cli` / `packages/server` preview-v2 路径与主实现的边界。
- Remaining gap: 仍可补一次实际运行验证，确认 TUI internal fetch、HTTP server、desktop sidecar 的事件流与 session 状态一致性。

### Plan / Build、Agent / Subagent / Skill 与权限边界

- Status: 已回流。
- Output: `agents-and-permissions.md`
- Trace: 基于官方 agents / permissions / tools 文档与 `agent`、`session`、`tool`、`permission` 源码核验，确认 Plan / Build 主要由 primary agent、权限规则和 prompt reminder 组合实现；subagent 通过 task / mention 进入；skill 是可加载能力材料。
- Remaining gap: 仍可补一次实际 Plan → Build → permission ask → subagent task 的运行记录。

### 会话状态、diff、revert 与分享

- Status: 已回流。
- Output: `session-and-server.md`
- Trace: 基于官方 CLI/server 文档与 session/server/storage/share/revert 源码核验，确认 session 是可继续、fork、share、summarize、diff、revert 的持久对象，并存在 legacy 与 v2 API 并行路径。
- Remaining gap: 仍需确认默认用户路径中 legacy API 与 v2 API 的具体分工。

### `/init`、项目规则、配置层级与上下文注入

- Status: 已回流。
- Output: `context-and-configuration.md`
- Trace: 基于官方 config 文档与 config、instruction、command、session、skill 源码核验，确认 `/init` 是 guided `AGENTS.md` setup，项目规则通过 `AGENTS.md`、config instructions、嵌套 instruction discovery 等路径进入上下文。
- Remaining gap: 仍可补 scoped instruction 与 `/init` 实际运行样例。

### 模型与 Provider runtime

- Status: 已回流。
- Output: `model-and-provider-runtime.md`
- Trace: 基于 provider、auth、model-status、transform、LLM streaming 源码核验，补足模型目录、认证、Provider transform、AI SDK/native runtime 等模型运行层。
- Remaining gap: 仍可补一个具体 Provider 请求例子，展示 transform 前后差异。

### 扩展生态与协议边界

- Status: 已回流。
- Output: `ecosystem-and-extensions.md`
- Trace: 基于官方 tools/plugins/config/acp 文档与 mcp、acp、plugin、SDK、custom tool 源码核验，区分 MCP、ACP、plugins、custom tools、SDK 的职责边界。
- Remaining gap: 仍可用真实 plugin 和 MCP server 样例验证扩展生命周期。

### 产品表面与平台入口

- Status: 已回流。
- Output: `product-surfaces.md`
- Trace: 基于 TUI、Web、Desktop、VS Code、GitHub Action、share web、notifications、install/upgrade/release 源码和配置核验，补足 OpenCode 的产品化入口。
- Remaining gap: 仍可补截图或运行观察，验证实际用户流程。

### 工程成熟度与运营支撑

- Status: 已回流。
- Output: `engineering-and-operations.md`
- Trace: 基于 tests、CI、benchmark、migration、observability、stats、release、CONTRIBUTING 和 AGENTS 核验，补足 OpenCode 作为长期维护开源项目的工程治理层。
- Remaining gap: 仍可实际运行 package-local tests、typecheck 和 benchmark，补动态验证结果。

## 仍建议后续研究的缺口

### 与 Claude Code / Codex CLI / Aider 的横向比较

- Gap: OpenCode 自身机制已经有对象级分析，但还没有与 Claude Code、Codex CLI、Aider 对齐到同一组维度。
- Why it matters: 横向比较可以判断 OpenCode 的差异点是多入口架构、权限模型、agent/subagent、server API、生态插件、Provider runtime、产品表面，还是工程运营成熟度。
- Existing signals: 本目录已有 Claude Code、Codex 相关材料；Aider 在候选池中。
- Evidence needed: 对齐各工具在 task interface、context、tools、permissions、session、verification、model/provider、extension、product surfaces、operations 维度的证据。
- Possible output: 横向比较专题，或各案例文档中的 `Relation to Topics` 段落。

### 运行实证与版本漂移跟踪

- Gap: 当前 OpenCode 文档主要基于官方文档和源码静态核验，尚未运行 TUI、serve、web、desktop、GitHub Action 或 provider 请求。
- Why it matters: 运行实证可以验证默认路径、UI 行为、permission ask、session diff/revert、provider transform 与 telemetry 是否符合源码推断。
- Existing signals: 各机制专题的 `Needs` 已标出运行验证项；`notes/runtime-validation.md` 已建立统一记录入口。
- Evidence needed: 运行命令、环境、输出、截图或日志，配合版本基线。
- Possible output: 优先更新 `notes/runtime-validation.md`，再回填各机制专题 Evidence。

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - <https://opencode.ai/docs/zh-cn>
  - <https://opencode.ai/docs/zh-cn/config>
  - <https://opencode.ai/docs/zh-cn/tools>
  - <https://opencode.ai/docs/zh-cn/agents>
  - <https://opencode.ai/docs/zh-cn/cli>
  - <https://opencode.ai/docs/zh-cn/server>
  - <https://opencode.ai/docs/zh-cn/ecosystem>
  - `overview.md`
  - `architecture.md`
  - `agents-and-permissions.md`
  - `session-and-server.md`
  - `context-and-configuration.md`
  - `model-and-provider-runtime.md`
  - `ecosystem-and-extensions.md`
  - `product-surfaces.md`
  - `engineering-and-operations.md`
  - `notes/source.md`
  - `notes/evidence.md`
- Trace: 本文件最初承接官方“简介”页和文档目录树推断出的研究缺口；经多轮源码核验后，核心 runtime、模型、生态、产品和工程运营缺口已回流为对象级概览和机制专题，剩余缺口保留为后续研究入口。
- Needs: 继续补运行实证和跨工具横向比较；如新增动态证据，应回填对应机制专题的 Evidence。
