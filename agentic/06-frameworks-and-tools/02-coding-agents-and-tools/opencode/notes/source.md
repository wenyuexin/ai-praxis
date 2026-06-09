# OpenCode Source Notes

> 本文件记录 OpenCode 官方文档和本地源码的核验过程。它是研究辅助材料，允许保留搜索路径、临时观察、失败搜索和未闭合问题；不要把这里的观察直接写成对象级结论。

## 研究基线

- Upstream: <https://github.com/anomalyco/opencode>
- Official docs: <https://opencode.ai/docs/zh-cn>
- Local source: `D:\github\opencode`
- Current status: 核心静态源码核验已回流；运行实证仍待补充

## 初始核验范围

优先围绕 `backlog.md` 的 P0 缺口展开：

- 多入口架构与共享后端。
- `/init` 与项目规则注入。
- Plan / Build 模式与人类门控。
- 工具注册、权限与执行边界。
- 会话状态、diff、revert 与分享。

## 待查入口

- `packages/cli/`
- `packages/server/`
- `packages/tui/`
- `packages/app/`
- `packages/desktop/`
- `packages/opencode/`
- `packages/core/`

## 初步观察（2026-06-09）

### 官方文档线索

- 官方 README 将 OpenCode 定位为 `The open source AI coding agent`，并说明有桌面应用 beta。
- 官方中文简介页说明 OpenCode 提供终端界面、桌面应用和 IDE 扩展。
- 官方 CLI 文档显示 `opencode` 支持 `--continue`、`--session`、`--fork`、`--share`、`--agent`、`--file`、`--format json`、`--attach` 等标志；`serve` 提供无界面 HTTP server；`attach` 可把 TUI 连接到已启动的后端 server。
- 官方权限页显示权限键包括 `read`、`edit`、`glob`、`grep`、`bash`、`task`、`skill`、`lsp`、`webfetch`、`websearch`、`external_directory`、`doom_loop`；默认多数权限为 `allow`，`external_directory` 和 `doom_loop` 默认为 `ask`，`.env` 读取默认拒绝。
- 官方代理页显示 OpenCode 有两类代理：primary agent 和 subagent；内置 primary agents 为 Build 和 Plan，内置 subagents 为 General、Explore、Scout。
- 官方代理页说明 Plan 是受限代理，默认对 file edits 和 bash 设置为 `ask`；Build 是启用所有工具的默认主代理。
- 官方配置页显示配置覆盖远程组织默认、全局、项目级、自定义路径和自定义目录，并覆盖模型、代理、权限、MCP、插件、压缩、文件监视等。

### 本地源码结构线索

- 根 `package.json` 显示 monorepo 使用 Bun workspaces，核心包包括 `packages/opencode`、`packages/cli`、`packages/server`、`packages/tui`、`packages/app`、`packages/desktop`、`packages/sdk`、`packages/plugin` 等。
- `packages/opencode/src/` 是核心实现集中区，包含 `agent/`、`cli/`、`config/`、`mcp/`、`permission/`、`server/`、`session/`、`share/`、`skill/`、`storage/`、`tool/` 等目录。
- `packages/opencode/src/tool/` 包含 `bash/shell`、`edit`、`patch`、`lsp`、`skill`、`task`、`todo`、`webfetch`、`websearch`、`question`、`registry` 等文件，与官方工具/权限文档高度对应。
- `packages/opencode/src/server/routes/` 和独立 `packages/server/src/groups|handlers/` 都存在 server/API 相关结构，后续需要确认两者职责关系。
- `packages/cli/src/` 包含 `commands/`、`framework/`、`services/daemon.ts`、`index.ts`、`tui.ts`，适合追踪 CLI 到 TUI/server 的启动路径。
- `packages/opencode/test/` 覆盖 `agent`、`config`、`permission`、`server`、`session`、`tool`、`skill`、`mcp` 等模块，可作为实现行为的辅助证据。

## 下一步搜索路径

- 从 `packages/cli/src/index.ts`、`packages/cli/src/tui.ts`、`packages/opencode/src/cli/bootstrap.ts` 追踪 CLI 启动链路。
- 从 `packages/opencode/src/server/server.ts`、`packages/opencode/src/server/routes/`、`packages/server/src/routes.ts` 确认 HTTP API 与客户端入口关系。
- 从 `packages/opencode/src/agent/agent.ts`、`packages/opencode/src/agent/subagent-permissions.ts` 核验 Build / Plan / subagent 权限模型。
- 从 `packages/opencode/src/permission/evaluate.ts`、`packages/opencode/src/tool/registry.ts` 核验 allow / ask / deny、细粒度规则和工具注册方式。
- 从 `packages/opencode/src/session/`、`packages/opencode/src/share/`、`packages/opencode/src/storage/schema.ts` 核验 session、diff、revert、share 和持久化模型。

## 回流状态

- 多入口架构与共享后端已回流到 `../architecture.md`。
- Plan / Build、agent/subagent、tool registry 与 permission 已回流到 `../agents-and-permissions.md`。
- Session、server API、diff、revert、share 已回流到 `../session-and-server.md`。
- 配置、`AGENTS.md`、instructions、skills 与上下文注入已回流到 `../context-and-configuration.md`。
- Provider runtime 已回流到 `../model-and-provider-runtime.md`。
- MCP、ACP、plugins、custom tools、SDK 已回流到 `../ecosystem-and-extensions.md`。
- 产品表面已回流到 `../product-surfaces.md`。
- 工程运营成熟度已回流到 `../engineering-and-operations.md`。

## Notes

- 官方文档目录树提供了研究方向；核心静态机制已经回流正文，但默认路径、UI 行为、permission ask、provider transform 和运行时事件流仍需要运行实证。
- 本文件继续作为源码核验与失败搜索记录，不承担对象级结论叙事。
