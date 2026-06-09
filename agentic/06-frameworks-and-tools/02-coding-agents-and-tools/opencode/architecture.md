# OpenCode Architecture

> 本文解释 OpenCode 的多入口架构与共享后端边界。重点不是列包目录，而是说明不同入口如何汇聚到 server、session、tool、permission 等 agent runtime 能力。

## 总体判断

OpenCode 当前架构呈现“两条线并行”：

- **主实现线**：`packages/opencode` 承载成熟 CLI、TUI、serve、web、server routes、session、tool、permission、agent runtime。
- **preview / v2 线**：`packages/cli` 和 `packages/server` 提供较新的 CLI/API 路径，依赖 core 层和独立 server groups / handlers。

因此，研究 OpenCode 时不能只看包名。`packages/server` 并不等价于全部 server 实现；主产品入口的大量 server 能力仍在 `packages/opencode/src/server/` 中。

## 入口层

### 主 CLI

`packages/opencode/src/index.ts` 是主 CLI dispatcher，注册默认 TUI、`attach`、`serve`、`web` 等命令。默认 `$0` 进入 TUI，`serve` 启动无界面 HTTP server，`web` 启动 server 后打开浏览器，`attach` 连接到已有 server。

### 默认 TUI

默认 TUI 会启动 worker。worker 有两种后端访问方式：

- 内部模式：不打开 TCP 端口，直接调用 `Server.Default().app.fetch()`。
- 网络模式：调用 `Server.listen()`，再通过 URL 访问后端。

这说明默认 TUI 即使看起来是“本地终端应用”，也不是独立内存循环，而是通过同一 server app 访问 session/tool/permission 等能力。

### Serve / Web / Attach

- `serve` 直接启动 `packages/opencode` 的 server。
- `web` 也启动同一 server，再打开浏览器界面。
- `attach` 不启动本地后端，而是校验并连接外部 server URL。

这组入口使 OpenCode 同时支持本地交互、无头服务、浏览器 UI 和远程 TUI attach。

### Desktop 与 Web App

Desktop 是 Electron shell。它启动本地 sidecar server，然后渲染 app；web app 则通过 server connection / SDK context 访问 HTTP server。

因此 desktop 和 web 更像共享 backend 的客户端，而不是独立 agent 实现。

## 后端层

`packages/opencode/src/server/server.ts` 负责组装主后端；HTTP API routes 会注入 session、prompt、permission、tool registry、provider、LLM 等服务。

这个后端承担的职责包括：

- session 创建、查询、继续、fork、share、summarize、diff、revert。
- permission request 列表与 reply。
- provider/model/config/agent/tool/skill 等 runtime 查询。
- UI 与事件流支撑。

主入口共享后端的关键证据是：TUI、serve、web 和 desktop sidecar 都调用 `Server.listen()` 或 `Server.Default().app.fetch()` 这一组后端能力。

## Preview / V2 API 线

`packages/cli` 是一个更小的 preview CLI 包，命令处理器使用 lazy handlers。它的 `serve` 路径使用 `@opencode-ai/server/routes`。

`packages/server` 则定义独立 API groups / handlers，例如 session、message、permission、agent、skill、question、model、provider 等。主 `packages/opencode` server 也会挂载 v2 API 路径。

这意味着 OpenCode 正在从主实现线向 core/server 分层迁移，但当前两者仍需区分：

- 不能把 `packages/server` 看成唯一 server。
- 不能把 v2 session API 的设计直接等同于 legacy session loop 的全部行为。
- 需要按入口确认它走的是主 `packages/opencode` server，还是 preview `packages/server` routes。

## 架构意义

OpenCode 的架构价值在于，它把 coding agent 产品常见的几类形态合并到同一个 runtime 附近：

- CLI/TUI：本地高频交互。
- Serve/API：自动化和外部客户端入口。
- Web/Desktop：图形界面与团队协作入口。
- SDK/API：可编程扩展入口。
- Attach：远程连接与共享后端入口。

这使它适合研究“coding agent 如何从单进程 CLI 演化为多客户端 agent runtime”。

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - <https://opencode.ai/docs/zh-cn/cli>
  - `packages/opencode/src/index.ts`
  - `packages/opencode/src/cli/cmd/tui.ts`
  - `packages/opencode/src/cli/tui/worker.ts`
  - `packages/opencode/src/cli/cmd/serve.ts`
  - `packages/opencode/src/cli/cmd/web.ts`
  - `packages/opencode/src/cli/cmd/attach.ts`
  - `packages/opencode/src/server/server.ts`
  - `packages/opencode/src/server/routes/instance/httpapi/server.ts`
  - `packages/desktop/src/main/sidecar.ts`
  - `packages/app/src/entry.tsx`
  - `packages/cli/src/index.ts`
  - `packages/server/src/api.ts`
- Trace: 本文将 `backlog.md` 中“多入口架构与共享后端”缺口升级为机制专题；源码核验过程保留在 `notes/source.md`。
- Needs: 后续运行 `opencode serve`、TUI 内部模式和 desktop sidecar，确认事件流与 session 状态是否完全一致。
