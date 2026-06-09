# OpenCode Overview

> OpenCode 是一个开源 AI coding agent。本文基于官方文档与本地源码核验，概括它作为多入口 coding agent runtime、产品系统和长期维护开源项目的对象定位、核心结构和研究价值。

## 对象定位

OpenCode 不是单一终端补全工具，而是围绕共享后端组织的 coding agent 系统：终端 TUI、`serve`、`web`、desktop sidecar、远程 `attach`、IDE/ACP、GitHub Action 等入口，最终围绕 server、session、tool、permission、agent、provider、extension 等能力工作。

它的核心价值在于把 coding agent 的几个关键问题放到同一个对象内观察：

- 多客户端入口如何共享 agent runtime。
- Plan / Build 如何通过 agent 权限和 prompt reminder 实现人类门控。
- 工具注册、权限请求、session diff / revert / share 如何构成执行边界。
- Provider runtime 如何适配快速变化的模型生态。
- `AGENTS.md`、配置层级、skills、MCP、plugins、SDK 如何把项目约束与外部能力注入 agent。
- TUI、Web、Desktop、IDE、GitHub Action 等产品表面如何承接同一 runtime。
- 测试、迁移、observability、release 等工程能力如何支撑长期维护。

## 核心结构

OpenCode 的当前实现可以分成八层：

1. **交互入口层**：CLI/TUI、`serve`、`web`、desktop、remote attach、IDE/ACP、GitHub Action。
2. **共享后端层**：server routes、session service、permission service、tool registry、provider/model service。
3. **agent runtime 层**：primary agent、subagent、tool execution、prompt loop、session reminders。
4. **session substrate 层**：session、message、part、event、share、diff、revert、permission API。
5. **context / configuration 层**：config hierarchy、`AGENTS.md` instructions、nested instructions、skills、compaction、LSP。
6. **model / provider 层**：provider catalog、auth、model status、provider transform、AI SDK / native streaming。
7. **extension ecosystem 层**：MCP、ACP、plugins、custom tools、SDK。
8. **product / operations 层**：TUI/Web/Desktop/IDE/GitHub surfaces、notifications、install/upgrade/release、tests、CI、observability、stats。

其中 `packages/opencode` 是当前最重要的核心实现集中区；`packages/cli` 和 `packages/server` 还承载 preview / v2 API 路径，和主实现之间存在并行演进关系。

## 关键机制

### 多入口共享后端

主 CLI 的 TUI、`serve`、`web` 和 desktop sidecar 都会走 `packages/opencode` 的 server 能力。默认 TUI 可以不打开 TCP 端口，但仍通过内部 fetch 调用同一 server app；`attach` 则连接到已有 server URL。

这意味着 OpenCode 更接近“带本地/远程客户端的 agent runtime”，而不是只在终端中运行的一次性 CLI。

### Agent 即模式

OpenCode 的 Plan / Build 不是完全独立的执行引擎，而是 primary agent 的不同配置：Build 是默认全权限主代理，Plan 是受限主代理。Plan 默认限制编辑和命令执行，并通过 prompt reminder 强化只读/规划行为。

Subagent 通过 `task` 工具或 `@agent` 调用进入，例如 General、Explore、Scout 等。它们把复杂搜索、代码探索和多步任务从主 agent 中分离出来，同时可以拥有独立权限边界。

### 工具、权限和 session 构成执行边界

OpenCode 的工具系统覆盖文件读写、patch、shell、LSP、web、skill、task、question、todo 等能力。权限系统支持 allow / ask / deny，并通过工具执行中的 `ctx.ask(...)` 汇聚到统一 permission request 流程。

Session 则把 agent 行为记录为可继续、可 fork、可分享、可 summarize、可 diff、可 revert 的对象。权限请求、文件变更和模型输出都可以通过 server/session 相关结构进入可审查、可恢复的 execution substrate。

### 项目规则与上下文注入

`/init` 的职责不是直接硬编码生成结论，而是注入 guided `AGENTS.md` setup prompt；真正的项目规则通过 `AGENTS.md`、可选 `CLAUDE.md`、配置中的 `instructions`、远程 URL、以及读取文件时发现的嵌套 instruction 共同进入上下文。

这使 OpenCode 的项目适配更像“外部规则与上下文源的组合”，而不是单一 repo map。

### Provider runtime 适配模型生态

OpenCode 的模型层包含 provider catalog、认证、模型状态过滤、provider transform、AI SDK streaming 和 experimental native runtime。Provider transform 会影响消息格式、参数、schema、variant 和 token 默认值。

这说明成熟 coding agent 不只是 agent loop，还需要一个长期适配模型生态变化的 compatibility layer。

### 扩展生态不是单一插件系统

OpenCode 同时提供 MCP、ACP、plugins、custom tools 和 SDK。它们扩展的对象不同：MCP 接入外部工具协议，ACP 适配 IDE/编辑器协议，plugins 改写 host 行为，custom tools 提供局部动作，SDK 暴露 HTTP API 自动化边界。

这让 OpenCode 更像一个可扩展 agent platform，而不是只有内置工具集合的 CLI。

### 产品表面和工程运营支撑长期使用

TUI、Web、Desktop、IDE/ACP、GitHub Action、share web、notifications、install/upgrade/release 共同构成 OpenCode 的产品表面。测试、CI、benchmark、迁移、observability、stats、release signing 等则支撑它作为长期维护开源项目持续演进。

这部分很重要：OpenCode 的研究价值不只在 agent 机制，也在它如何把 agent runtime 产品化、分发化和运营化。

## 可迁移启发

- **把 mode 建模为 agent**：Plan / Build 的差异可以通过 agent 权限、prompt reminder、工具集合和 UI 切换组合出来。
- **把人类门控放进 tool/runtime 边界**：permission request 比“执行前统一确认”更细粒度，也更容易被 session 记录。
- **让多入口共享同一 execution substrate**：TUI、desktop、web、remote attach 如果共享 session/server，会减少状态割裂。
- **把项目知识外部化为 instruction artifact**：`AGENTS.md`、skills、config、custom command 都是可复用 harness 的落点。
- **把模型适配独立成 Provider compatibility layer**：避免每次模型生态变化都侵入 agent/runtime 主体。
- **区分扩展机制的职责边界**：MCP、ACP、plugins、custom tools、SDK 不应混成一个“插件”概念。
- **保留 case 内聚性**：OpenCode 同时涉及架构、权限、session、context、provider、生态、产品和工程运营，不适合只作为某个子领域的一条证据。

## 限制

- 当前源码中存在 legacy/main runtime 与 preview/v2 API 并行路径，不能把所有 server/session 结论无差别套到每个入口。
- 本文主要基于官方文档、本地源码结构与关键实现路径；尚未运行 OpenCode 或复现完整用户 workflow。
- 官方文档、源码和 AGENTS.md 中的 V2 Session Core 设计显示项目仍在快速演进，结论有较高 drift risk。

## Relation to Topics

OpenCode 可作为以下主题的案例证据：

- human-in-the-loop：Plan / Build、permission ask、question tool、revert。
- agent runtime：agent/subagent、tool registry、prompt loop、session execution。
- context engineering：`AGENTS.md`、instructions、skills、compaction、LSP、file read nested instructions。
- coding agent product architecture：多入口 client + shared server/session backend。
- model/runtime compatibility：Provider catalog、auth、transform、AI SDK/native streaming。
- extension ecosystem：MCP、ACP、plugins、SDK、custom tools。
- open-source product operations：tests、CI、benchmarks、migrations、observability、release。

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - <https://opencode.ai/docs/zh-cn>
  - <https://opencode.ai/docs/zh-cn/agents>
  - <https://opencode.ai/docs/zh-cn/permissions>
  - <https://opencode.ai/docs/zh-cn/cli>
  - <https://opencode.ai/docs/zh-cn/config>
  - <https://github.com/anomalyco/opencode>
  - `architecture.md`
  - `agents-and-permissions.md`
  - `session-and-server.md`
  - `context-and-configuration.md`
  - `model-and-provider-runtime.md`
  - `ecosystem-and-extensions.md`
  - `product-surfaces.md`
  - `engineering-and-operations.md`
  - `notes/evidence.md`
- Trace: 本文从 `backlog.md` 的研究缺口和 `notes/` 的源码核验材料回流为对象级概览，并在后续新增模型、生态、产品和工程运营专题后更新总框架。
- Needs: 后续可补一次实际运行路径验证，并跟踪 V2 Session Core 从设计说明到默认 runtime 的迁移状态。
