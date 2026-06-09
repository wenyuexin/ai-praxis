# OpenCode Evidence Notes

> 本文件整理 OpenCode 研究中的 claim-source 对照。它服务于正文复核，不替代 `overview.md` 或机制专题。

## Claim Evidence Table

| Claim | Status | Sources | Notes | Needs |
|---|---|---|---|---|
| OpenCode 是开源 AI coding agent。 | Observed | <https://github.com/anomalyco/opencode>; <https://opencode.ai/docs/zh-cn> | 官方 README 使用 “The open source AI coding agent” 定位。 | 后续可补 release/version 基线。 |
| OpenCode 提供多种入口，包括 TUI、serve、web、desktop、attach。 | Observed | <https://opencode.ai/docs/zh-cn/cli>; `packages/opencode/src/index.ts`; `packages/opencode/src/cli/cmd/tui.ts`; `packages/opencode/src/cli/cmd/serve.ts`; `packages/opencode/src/cli/cmd/web.ts`; `packages/opencode/src/cli/cmd/attach.ts`; `packages/desktop/src/main/sidecar.ts` | 主 CLI 注册 TUI / attach / serve / web；desktop sidecar 启动本地 server。 | 运行验证各入口事件流一致性。 |
| 主 TUI、serve、web、desktop sidecar 围绕 `packages/opencode` server backend 工作。 | Observed / Inferred | `packages/opencode/src/cli/tui/worker.ts`; `packages/opencode/src/server/server.ts`; `packages/opencode/src/server/routes/instance/httpapi/server.ts`; `packages/desktop/src/main/sidecar.ts`; `packages/app/src/entry.tsx` | TUI worker 可走 `Server.Default().app.fetch()` 或 `Server.listen()`；web/desktop 通过 server connection / sidecar 访问后端。 | 运行验证 internal fetch 与 HTTP server 的 session 状态一致性。 |
| `packages/server` 不等价于全部 server 实现；主实现仍在 `packages/opencode/src/server/`。 | Observed | `packages/opencode/src/server/server.ts`; `packages/opencode/src/server/routes/instance/httpapi/server.ts`; `packages/server/src/api.ts`; `packages/cli/src/commands/handlers/serve.ts` | 存在 legacy/main server 与 preview/v2 API 并行路径。 | 跟踪后续迁移状态。 |
| OpenCode 支持 Build / Plan 两个内置 primary agent。 | Observed | <https://opencode.ai/docs/zh-cn/agents>; `packages/opencode/src/agent/agent.ts` | Build 是默认开发 agent；Plan 是受限计划 agent。 | 可补一次 UI 切换行为记录。 |
| Plan / Build 更像 agent 配置与权限边界，而不是两个独立执行引擎。 | Inferred | `packages/opencode/src/agent/agent.ts`; `packages/opencode/src/session/reminders.ts`; `packages/opencode/src/session/prompt/plan.txt`; `packages/opencode/src/session/prompt/build-switch.txt`; `packages/opencode/src/tool/plan.ts` | 源码显示 Plan/Build 通过 agent、permission、reminder、可选 `plan_exit` 组合实现。 | 运行验证 experimental plan mode。 |
| OpenCode 支持 primary agent 与 subagent；subagent 可通过 `task` 或 `@agent` 调用。 | Observed | <https://opencode.ai/docs/zh-cn/agents>; `packages/opencode/src/agent/agent.ts`; `packages/opencode/src/tool/task.ts`; `packages/opencode/src/tool/task.txt` | 内置 subagents 包含 General、Explore、Scout 等；task tool 是主要启动路径。 | 补 Scout 实现路径与调用样例。 |
| Skills 是可加载能力材料，不等同于 agent。 | Observed / Inferred | `packages/opencode/src/skill/index.ts`; `packages/opencode/src/tool/skill.ts`; `packages/opencode/src/session/system.ts` | 系统上下文列出可用 skills；正文通过 `skill` tool 加载。 | 补官方 skills 文档或示例。 |
| OpenCode 权限系统覆盖工具和安全防护项，支持 allow / ask / deny。 | Observed | <https://opencode.ai/docs/zh-cn/permissions>; `packages/opencode/src/permission/index.ts`; `packages/opencode/src/permission/evaluate.ts`; `packages/opencode/src/tool/registry.ts` | 工具执行通过 `ctx.ask(...)` 汇聚到 permission service。 | 补一次 permission request / reply API 行为记录。 |
| Session 是可继续、fork、share、summarize、diff、revert 的持久对象。 | Observed | <https://opencode.ai/docs/zh-cn/cli>; <https://opencode.ai/docs/zh-cn/server>; `packages/opencode/src/server/routes/instance/httpapi/groups/session.ts`; `packages/opencode/src/server/routes/instance/httpapi/handlers/session.ts`; `packages/opencode/src/session/session.ts`; `packages/opencode/src/session/revert.ts`; `packages/opencode/src/share/session.ts` | Legacy session API 覆盖 session lifecycle 与协作恢复动作。 | 运行验证默认 CLI 标志到 API 的映射。 |
| OpenCode 存在 legacy session/message storage 与 V2 durable input/projected message/event log 并行结构。 | Observed / Inferred | `packages/core/src/session/sql.ts`; `packages/core/src/session.ts`; `packages/core/src/session/projector.ts`; `packages/core/src/event/sql.ts`; `packages/server/src/groups/session.ts`; `packages/server/src/groups/message.ts` | V2 结构显示更明确的 durable prompt admission 与 projected message 方向。 | 跟踪默认 runtime 是否迁移到 V2。 |
| `/init` 是 guided `AGENTS.md` setup 入口。 | Observed | <https://opencode.ai/docs/zh-cn>; `packages/opencode/src/command/index.ts` | `/init` 注入初始化 prompt，引导生成或更新 `AGENTS.md`。 | 补一次实际 `/init` 输出样例。 |
| Project instructions 通过 `AGENTS.md`、可选 `CLAUDE.md`、deprecated `CONTEXT.md` 和 config `instructions` 进入上下文。 | Observed | `packages/opencode/src/session/instruction.ts`; `packages/opencode/src/session/system.ts`; `packages/opencode/src/tool/read.ts` | read tool 会发现嵌套 instructions 并追加 system reminder。 | 补 scoped instruction 行为样例。 |
| OpenCode 配置层级覆盖远程、全局、项目级、显式路径和显式内容。 | Observed | <https://opencode.ai/docs/zh-cn/config>; `packages/opencode/src/config/config.ts`; `packages/opencode/src/config/paths.ts`; `packages/core/src/v1/config/config.ts` | 配置覆盖 agent、permission、MCP、plugin、watcher、compaction、instructions 等。 | 补配置冲突和 merge precedence 示例。 |
| OpenCode 有独立 Provider runtime，合并模型目录、配置、环境变量、auth store、plugin hooks 和动态 provider loader。 | Observed / Inferred | `packages/opencode/src/provider/provider.ts`; `packages/opencode/src/auth/index.ts`; `packages/opencode/src/plugin/index.ts`; `model-and-provider-runtime.md` | 模型接入不是单纯 SDK 调用，而有 provider catalog、auth、status、default selection 和 custom loader。 | 补具体 Provider 请求例子。 |
| Provider transform 是兼容层，会影响消息、参数、schema、variants 和 token 默认值。 | Observed / Inferred | `packages/opencode/src/provider/transform.ts`; `packages/opencode/src/session/llm/request.ts`; `packages/opencode/src/session/llm.ts`; `model-and-provider-runtime.md` | transform 位于 LLM request prep 和 streaming 前，对不同 Provider 行为有实质影响。 | 补 transform 前后对照。 |
| OpenCode 同时存在 AI SDK streaming 和 experimental native LLM runtime。 | Observed | `packages/opencode/src/session/llm.ts`; `packages/opencode/src/session/llm/native-runtime.ts`; `packages/opencode/src/session/llm/native-request.ts`; `model-and-provider-runtime.md` | native runtime 是 opt-in/fallback 且 Provider 支持有限。 | 运行验证 native 分支。 |
| MCP、ACP、plugins、custom tools、SDK 是不同扩展边界，不是同一个插件系统。 | Observed / Inferred | `packages/opencode/src/mcp/index.ts`; `packages/opencode/src/acp/agent.ts`; `packages/opencode/src/acp/service.ts`; `packages/opencode/src/plugin/index.ts`; `packages/plugin/src/index.ts`; `packages/opencode/src/tool/registry.ts`; `packages/sdk/js/src/v2/client.ts`; `ecosystem-and-extensions.md` | MCP 扩展工具协议，ACP 扩展 IDE 接入，plugin 扩展 host，custom tool 扩展单个动作，SDK 扩展 HTTP API 自动化。 | 用真实 plugin/MCP 样例验证生命周期。 |
| OpenCode 有多产品表面：TUI、Web、Desktop、IDE/ACP、GitHub Action、share web、notifications、install/upgrade/release。 | Observed | `packages/tui/src/app.tsx`; `packages/opencode/src/cli/cmd/web.ts`; `packages/app/src/app.tsx`; `packages/desktop/src/main/index.ts`; `sdks/vscode/package.json`; `github/action.yml`; `github/index.ts`; `packages/web/src/pages/s/[id].astro`; `.github/workflows/publish.yml`; `product-surfaces.md` | 产品面远超 CLI/TUI，覆盖平台自动化和分发。 | 补实际 UI/Action 运行观察。 |
| OpenCode 具备长期维护项目的工程运营能力。 | Observed / Inferred | `package.json`; `turbo.json`; `.github/workflows/test.yml`; `.github/workflows/typecheck.yml`; `.github/workflows/publish.yml`; `packages/opencode/test/`; `perf/test-suite.md`; `packages/core/src/database/migration.ts`; `packages/core/src/observability/otlp.ts`; `infra/monitoring.ts`; `engineering-and-operations.md` | 覆盖测试、CI、benchmark、migration、observability、stats、release 等。 | 实际运行测试、typecheck、benchmark。 |

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - <https://opencode.ai/docs/zh-cn>
  - <https://opencode.ai/docs/zh-cn/agents>
  - <https://opencode.ai/docs/zh-cn/permissions>
  - <https://opencode.ai/docs/zh-cn/cli>
  - <https://opencode.ai/docs/zh-cn/config>
  - <https://opencode.ai/docs/zh-cn/server>
  - <https://opencode.ai/docs/zh-cn/ecosystem>
  - <https://github.com/anomalyco/opencode>
  - `../overview.md`
  - `../architecture.md`
  - `../agents-and-permissions.md`
  - `../session-and-server.md`
  - `../context-and-configuration.md`
  - `../model-and-provider-runtime.md`
  - `../ecosystem-and-extensions.md`
  - `../product-surfaces.md`
  - `../engineering-and-operations.md`
- Trace: 本文件从 OpenCode 初始研究 backlog、源码核验 notes 与后续遗漏自检中整理 claim-source 对照，支撑对象概览与机制专题的 Evidence。
- Needs:
  - 补运行实证，验证默认 CLI/TUI/server/product surface 路径。
  - 补跨工具横向比较。
  - 跟踪 legacy/main runtime 与 preview/v2 API 的演进关系。
