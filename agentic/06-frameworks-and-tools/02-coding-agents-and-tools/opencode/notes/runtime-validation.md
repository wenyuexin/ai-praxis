# OpenCode Runtime Validation

> 本文件记录 OpenCode 运行实证的补证计划与观察模板。当前主案例文档主要基于官方文档和源码静态核验；这里为后续动态验证预留统一落点，避免把未执行的验证写成正文定论。

## 目标

本文件主要补三类动态证据：

- 默认用户路径是否与源码推断一致。
- 多入口产品面的实际交互与状态流。
- 关键 runtime 机制在运行时的边界行为。

它不直接替代 `product-surfaces.md`、`engineering-and-operations.md` 或其他机制专题；验证完成后，应把稳定结论回填到对应正文的 `Evidence` / `Needs`。

## 建议验证矩阵

### 入口与产品面

- `opencode` / TUI：确认默认启动路径、session continue/fork、agent 切换、permission 提示。
- `opencode serve`：确认 HTTP server、session API、share/diff/revert 相关路径。
- `opencode web`：确认浏览器入口、URL 输出、password 提示、session 连接方式。
- desktop：确认 sidecar server、renderer 交互、通知和 updater 基本行为。
- GitHub Action：确认 `/opencode` / `/oc` 触发、评论回写、session share。

### 关键机制

- Plan → Build 切换：确认 prompt reminder 与权限边界是否符合文档推断。
- permission ask：确认 tool-level request/reply 的触发点和返回路径。
- session diff / revert：确认用户表面与 server/session 结构的对应关系。
- provider transform：确认至少一个 provider 请求在 transform 前后的关键差异。
- telemetry / observability：确认 runtime 中可见的 logs、traces 或 error reporting 入口。

### 工程运行

- package-local tests：确认核心 package 的测试入口和约束。
- typecheck：确认 package-local typecheck 与 CI 定义是否一致。
- benchmark：确认 perf 脚本的输入/输出与静态文档描述一致。

## 记录模板

每次运行验证建议至少记录：

- Date:
- Scope:
- Environment:
- Command:
- Entry Surface:
- Expected From Static Analysis:
- Observed:
- Drift From Docs/Code:
- Evidence Status:
- Follow-up:

## 当前状态

- Status: 待执行。
- Covered: 尚无动态运行记录；当前仅建立验证矩阵与记录模板。
- Priority:
  - P0：TUI / `serve` / `web` / Plan-Build-permission
  - P1：provider transform / session diff-revert / package-local tests
  - P2：desktop / GitHub Action / telemetry / benchmark

## 与正文回填关系

- `architecture.md`：回填 TUI internal fetch、HTTP server、desktop sidecar 的实际状态流。
- `agents-and-permissions.md`：回填 Plan / Build / permission ask / subagent task 的运行记录。
- `session-and-server.md`：回填 diff / revert / share / legacy-v2 默认路径。
- `context-and-configuration.md`：回填 `/init`、scoped instruction、config merge 行为样例。
- `model-and-provider-runtime.md`：回填 provider transform 与 native runtime 观察。
- `product-surfaces.md`：回填 TUI / Web / Desktop / GitHub Action 的用户流程。
- `engineering-and-operations.md`：回填 tests / typecheck / benchmark 的动态结果。

## Evidence

- Status: `Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - `../backlog.md`
  - `../architecture.md`
  - `../agents-and-permissions.md`
  - `../session-and-server.md`
  - `../context-and-configuration.md`
  - `../model-and-provider-runtime.md`
  - `../product-surfaces.md`
  - `../engineering-and-operations.md`
  - `source.md`
  - `evidence.md`
- Trace: 本文件从 `backlog.md` 中“运行实证与版本漂移跟踪”的剩余缺口拆出，作为案例研究辅助材料层的动态验证入口。
- Needs: 后续需要真实运行命令、环境、日志、截图或输出记录，才能把这里的占位模板升级为 `Observed`。
