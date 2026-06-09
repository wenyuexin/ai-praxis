# OpenCode Product Surfaces

> 本文整理 OpenCode 的产品入口与用户界面层：TUI、Web、Desktop、IDE/ACP、GitHub Action、分享页、通知、安装与升级。它补足仅从 runtime 视角无法解释的产品化部分。

## 核心判断

OpenCode 不是只有 CLI/TUI 一个产品面。它围绕同一套 agent/runtime 能力构建了多种表面：

- 终端 TUI：高频本地开发入口。
- Web app：浏览器 UI 与远程 server 入口。
- Desktop app：Electron shell + local sidecar server。
- IDE/ACP/VS Code：编辑器内 agent 入口。
- GitHub Action：PR/Issue 平台自动化入口。
- Share web：会话分享与展示入口。
- Notifications / settings：跨 web/desktop 的任务完成和错误反馈。
- Install / upgrade / release：跨平台分发与更新路径。

这些产品面说明 OpenCode 的目标不是只做“终端里的 agent”，而是把 agent runtime 放到多个人类工作入口中。

## TUI

主 CLI 默认进入 TUI。TUI command 接收 project、model、session、prompt、agent 等参数，创建 worker-backed fetch/event transport，校验 session 后运行 `@opencode-ai/tui`。

TUI 内部还有 session list/new、model/agent/MCP/provider/status/theme/help/workspace/debug/console 等命令面板能力。这说明 TUI 不只是 prompt 输入框，而是 runtime 控制台。

## Web App

`opencode web` 会启动 server，提示 password 安全风险，打印 local/network/mDNS URL，并打开浏览器。Web app 是 Solid/Vite 应用，提供 app shell、routing、platform abstraction、command/layout/session providers。

Web app 把 OpenCode 从本地终端扩展为浏览器可访问界面，也为 desktop renderer 复用同一 app 层打基础。

## Desktop

Desktop 是 Electron 应用。主进程负责 app name/id、userData、single-instance lock、deep links、sidecar server、protocol handler、updater、IPC 和窗口管理；renderer 则构造 desktop platform，提供文件/目录选择、storage、updater、WSL servers、通知、链接、重启、缩放和菜单动作。

Desktop 的关键不是重写 agent runtime，而是通过 sidecar server 和 app renderer 将同一 runtime 包装成桌面应用。

## IDE / ACP / VS Code

OpenCode 的 IDE 面包括 ACP command 与 VS Code extension。ACP 把编辑器交互映射到 OpenCode session/config/permission API；VS Code package 提供打开 terminal、打开新 terminal、把 filepath 加到 terminal 等命令和 keybindings。

这说明 OpenCode 的 IDE 集成当前更像“把 OpenCode 放进编辑器工作流”，而不是完全独立的 IDE runtime。

## GitHub Action 与平台自动化

OpenCode 提供 GitHub Action 和 `opencode github install/run` 命令。GitHub action 可以安装/缓存 OpenCode，并用 model、agent、share、prompt、mentions、variant、OIDC 等输入运行。

GitHub handler 支持用户事件和仓库事件，能响应 mentions，创建或更新评论/PR，推送分支，分享 session。另有 action runtime 会检查 `/opencode` 或 `/oc` 触发，启动 `opencode serve`，创建 session，订阅事件并回写 GitHub 评论。

这部分说明 OpenCode 具备从本地 agent 走向 PR/Issue 自动化的产品路径。相对地，当前核验没有发现等价 GitLab issue/MR workflow；GitLab 主要出现在 provider/auth 集成中。

## Share Web

`packages/web` 承载 marketing/docs/share web。分享页会读取 share data，渲染 session share，并设置 noindex 与 social metadata。

这说明 session share 不只是 API 字段，也有面向外部读者的展示面。

## Notifications

App 层有通知模型，覆盖 turn-complete 和 error，并按 session/project 建索引、剪枝历史、跟踪 unseen/error 状态。Web platform 使用浏览器 Notification；Desktop renderer 使用系统通知和窗口聚焦/导航；settings 中提供 agent response、permission、error 等通知开关。

通知机制体现了 long-running agent task 的产品需求：人类不一定一直盯着 agent，但需要在完成、错误或权限请求时被拉回。

## Install / Upgrade / Release

OpenCode 的 README 覆盖 curl installer、npm/bun/pnpm/yarn、Scoop、Chocolatey、Homebrew、pacman/AUR、mise、Nix，以及 desktop release assets。安装脚本会检测 OS/arch、Rosetta、musl、CPU baseline、版本、binary 和安装目录。

CLI upgrade 支持 curl、npm、pnpm、bun、brew、choco、scoop 等安装方式。Release pipeline 包含 CLI build、Windows signing、signature verification、Electron matrix build 等。

这说明 OpenCode 的产品成熟度不只在代码架构，还体现在分发、签名、升级和多平台安装路径。

## 设计意义

OpenCode 的产品面展示了 coding agent 平台化的方向：

- Runtime 可以共享，但用户入口必须贴近不同工作场景。
- 本地终端、浏览器、桌面、IDE、GitHub 平台都需要不同交互语义。
- Session、permission、event、notification 是多入口体验的一致性基础。
- 安装、升级、签名、Action 分发是开源 agent 工具走向主流使用的必要条件。

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - `README.md`
  - `install`
  - `packages/opencode/src/cli/cmd/tui.ts`
  - `packages/tui/src/index.tsx`
  - `packages/tui/src/app.tsx`
  - `packages/opencode/src/cli/cmd/web.ts`
  - `packages/app/src/app.tsx`
  - `packages/app/src/index.ts`
  - `packages/app/src/context/notification.tsx`
  - `packages/app/src/entry.tsx`
  - `packages/desktop/src/main/index.ts`
  - `packages/desktop/src/main/ipc.ts`
  - `packages/desktop/src/renderer/index.tsx`
  - `packages/desktop/src/preload/index.ts`
  - `sdks/vscode/package.json`
  - `github/action.yml`
  - `github/index.ts`
  - `packages/opencode/src/cli/cmd/github.ts`
  - `packages/opencode/src/cli/cmd/github.handler.ts`
  - `packages/web/src/pages/s/[id].astro`
  - `.github/workflows/publish.yml`
- Trace: 本文补足前一版 OpenCode 案例中遗漏的产品表面与分发路径，避免只从 agent runtime 解释该项目。
- Needs: 后续可补截图/运行观察，验证 TUI、Web、Desktop、GitHub Action 的实际用户流程。
