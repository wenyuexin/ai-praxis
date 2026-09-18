# DeepSeek Harness guard：执行卫生（tool-timeout 与重复调用检测）

> 版本锚点：`0.1.0-rc.7`（commit `99f6f02fecdb`，2026-08-17）。本文基于本地 checkout 源码核验（`Status: Observed`，锚点见 `notes/evidence.md` C13）。

`packages/guard/` 组承载 dsh 的"执行卫生"：工具超时强制与重复调用检测。它由**两个纯 listener 插件**组成——不注册任何服务，只监听工具执行管线事件。这解释了 capability 图谱中 guard 无服务声明的现象：图谱只收录服务声明，不收录纯事件监听插件。

## 1. timeout-policy：工具超时强制

插件：`packages/guard/timeout-policy`（`@deepseek-ai/dsh-tool-call-timeout-policy`，有 FIXME 计划改名 `dsh-timeout-guard`）。

机制（`ctx.on('tools/execute')` waterfall）：

1. 工具声明 `timeoutMs` 字段（经 `ctx.tools.get(exec.name)?.timeoutMs` 读取）；未声明不设时限，直接委托。
2. 有声明则用 `deadline(exec.signal, timeoutMs, TOOL_TIMEOUT)` 包一层合并期限，**临时替换 `exec.signal`**，委托 `next()` 后恢复上游 signal——post-execute listener 永远看不到本插件的（可能已中止的）超时信号。
3. 仅当**本插件的** timer 触发时（`timeoutOf` 按 code 作用域，嵌套外层 deadline 先触发读作普通上游取消）才把结果替换为结构化错误：`Error: tool call timed out after Nms`，`error.code = 'TOOL_TIMEOUT'`——模型可见、可被重试/回放插件路由。

## 2. repeat-tool-reminder：重复调用检测

插件：`packages/guard/repeat-tool-reminder`（`@deepseek-ai/dsh-repeat-tool-reminder`）。

机制（`ctx.on('tools/post-execute')` waterfall + `ctx.on('agent/pre-step')`）：

1. **per-agent 连续重复链**：`WeakMap<Agent, Chain>`，链 key = `[toolName, canonicalizedArgs]`（参数 deep key-sort 后 stringify——属性顺序不同的等价参数归一化）。
2. **被拒的调用也计数**：denied calls 走同一执行管线——"模型反复打被拒调用正是值得打破的循环"。
3. 命中阈值（默认 `[3, 5, 8]`）时注入提醒：首个阈值是 gentle 提示，后续是 detailed（带工具名、连续次数、参数预览 cap 500 字符）。
4. **observe-and-enrich, never veto**：提醒折叠进 `additionalContexts`（block 决策也附带），从不拦截或改写调用；用户插话（`agent/pre-step` 中出现 user source 消息）重置链——跨插话的重复不算循环。
5. 配置 fail loud：空 thresholds、非整数、<2、重复值都在插件加载时抛错。

## 3. 与其它机制的关系

- 两者都挂在 `tools/*` 事件上——与 approval（`approval/request` waterfall）同为工具执行管线的治理层，但 guard 是"执行后"的卫生（超时替换、重复提醒），approval 是"执行前"的闸门（见 `approval.md`）。
- 历史：`timeout/` 组在 2026-08-09 并入 `guard/`（见 `notes/source.md` 5.3）。

## Evidence

- **Status**: 本文件全部结论为 `Observed`（本地 checkout `0.1.0-rc.7` 源码直接核验，锚点见 `notes/evidence.md` C13）。
- **Sources**: `packages/guard/timeout-policy/src/index.ts`；`packages/guard/repeat-tool-reminder/src/index.ts`。
- **Trace**: 2026-08-21 由 ai-praxis 侧深研脚手架回流提炼（安全治理主线压实后）；版本锚点 `0.1.0-rc.7`。
- **Needs**: guard 组是否有第三类卫生机制未穷举（当前组内仅两包，已确认）；`timeout-policy` 改名待办的落地版本需跟版本差异表。
