# DeepSeek Harness capability seam：三角色代码范式

> 版本锚点：`0.1.0-rc.7`（commit `99f6f02fecdb`，2026-08-17）。本文基于本地 checkout 源码核验（`Status: Observed`，锚点见 `notes/evidence.md` C6/C7/C12）；官方口径与包级图谱见 `notes/source.md` 一/三。

**capability seam** 是 dsh 的可替换能力单元：一个 **Service Definition**（声明接口）、一个或多个 **Service Provider**（实现接口）、以及 **Consumer**（使用接口，通常是 model-facing tool）——**三者齐备才构成 seam，单一角色不是**。包可以合并角色，只有当角色独立演进时才拆分。

seam 的意义在于：**一个 provider 替换改变整个产品**。Filesystem 与 subprocess provider 共享同一执行世界——把它们指向远端 sandbox 时，Bash、PTY、LSP 一起迁移，无 provider fork；Subagent providers 在单一接口后变化同样大（从进程内 child agent 到另一产品内的 delegated turn）。

## 1. 三角色的代码形态（`ctx.shell` 样板）

以 `ctx.shell`（bash 执行 seam）为样板，三角色在代码中的范式：

### 1.1 Service Definition

- **抽象类 extends `Service`**，构造时 `super(ctx, 'shell')`——注册为 `ctx.shell`，fiber 卸载自动移除。
- `declare module '@deepseek-ai/cordis'` 把 `shell: ShellExecutor` 并入 Context 类型面（TS 声明合并）。
- **契约方法 + 语义 JSDoc**：`resolve(request)`（显式默认化）、`run(spec)`（前台）、`start(spec)`（后台）——JSDoc 写明每个方法的行为承诺（如"run 只对基础设施失败 reject，非零退出/超时/中止 resolve 带描述性结果"）。
- **settings namespace 归 seam 所有**，不归任一实现（`settingsNamespace('shell')`）——因为它命名的是能力而非实现；多 provider 共享一个 namespace，跨平台 settings 文档保持可解析。
- 服务包 **default-export 服务类**（packages/AGENTS.md 插件导出约定）。
- 重复注册：同一 context 加载第二个实现抛错（Cordis 标准 duplicate-service 行为）——fail loud。

### 1.2 Service Provider

- **子类化 Service Definition**：`class LocalBashExecutor extends ShellExecutor`，default-export 类作为插件加载。
- **`static inject` 声明跨 seam 依赖**：`LocalBashExecutor.inject = ['subprocess']`——bash-local 依赖 subprocess seam 派生进程。
- **`static Config`（schemastery schema）** 提供默认与校验。
- **`resolve(request): Spec` 显式默认化**：`run`/`start` 只收 fully-specified spec，绝不内部 `?? default`（根 AGENTS.md "Explicit > implicit" 约定的代码模板）；默认与上限（timeout、输出字节）在 resolve 时 clamp。
- 配置双源：settings section（可热更新）或 composition entry，经 `installSettingsSection` 接入。

### 1.3 Consumer

- **经 `ctx.inject` 声明依赖**（生命周期门控）或 `ctx.get`（可选服务，绕过门控）。
- 通过 `ctx.<key>` 消费服务方法/事件，不 import 实现。
- 例：`tool-bash` / `tool-pwsh` / hooks（Claude Code、Codex）消费 `ctx.shell`；agent-loop 消费 `ctx.sessions`/`ctx.tools`/`ctx.llm`。

## 2. 包级图谱：seam 的权威映射

`docs/capability-seams.md` 是**由源码生成的权威图谱**（`scripts/gen-doc-graphs.ts`，带 completeness guard，勿手改）：60+ 服务按 `ctx key` 列出 owner（Service Definition 所在包）、implementations（Provider 包）、direct consumers（Consumer 包）、companion plugins、角色（`seam`/`core`/`bundle`）。核验 seam 三角色时以图谱为包级索引，再下钻代码。

三类角色的样例（图谱）：

| ctx key | 角色 | Definition | Implementations | Consumers |
|---|---|---|---|---|
| `ctx.shell` | seam | `shell` | `bash-local` / `bash-sandbox` / `pwsh-local` | `tool-bash` / `tool-pwsh` / hooks |
| `ctx.sessions` | core | `session` | —（核心主干） | `agent-loop` / `agent` / `session-persistence` 等 |
| `ctx.approval` | seam | `user-approval` | `acp`（bridge answerer） | `tools` / `tool-bash` |
| `ctx.subagents` | seam | `subagent` | 6 实现（in-process/acp/codex/claude-code/dsh-sdk） | `tool-subagent` 等 |

注意图谱 owner 字段存在轻微文档漂移（如 `ctx.approval` owner 写 `approval` 无链接，实际包为 `user-approval`）——以源码路径为准。

## 3. 新增能力的检查清单（从范式推导）

1. 设计三角色，不只定义接口：新增能力 = Service Definition + Provider + Consumer 齐备。
2. Definition 契约写明行为承诺（失败语义、超时语义、增量读取等），settings namespace 归 seam。
3. Provider 显式默认化：`resolve(request): Spec`，`run`/`start` 不收 raw request。
4. Consumer 经 `ctx.inject`/`ctx.get` 接入，不 import 实现。
5. 变更 provider 时 consumer 零改动（换实现不碰消费方）。

## Evidence

- **Status**: 本文件全部结论为 `Observed`（本地 checkout `0.1.0-rc.7` 源码直接核验，锚点见 `notes/evidence.md` C6/C7/C12）；图谱包级映射为 `Verified`（生成物，`notes/source.md` 三）。
- **Sources**: `packages/shell/shell/src/index.ts`（ShellExecutor）；`packages/shell/bash-local/src/index.ts`（LocalBashExecutor）；`packages/core/session/src/index.ts`（SessionStore）；`docs/capability-seams.md`。
- **Trace**: 2026-08-21 由 ai-praxis 侧深研脚手架回流提炼（capability seam 主线压实后）；版本锚点 `0.1.0-rc.7`。
- **Needs**: `tool-bash` 作为 Consumer 的完整调用链接线细节未核验（可选深度，见问题树 §2.2）。
