# Evidence：插件层源码核验（第一批）

> 版本锚点：`0.1.0-rc.7`（commit `99f6f02fecdb`，2026-08-17）。核验方式：本地 checkout 源码直接观察。
> 状态约定：`Observed` = 源码直接可证（带 `文件:行号` 锚点）；`Inferred` = 从源码/文档推断；`Verified` = 官方文档自述（见 `source.md`）。
> 重要前提：`vendor/` 是 vendored 拷贝且带 18 条本地修改，**研究 vendor/ 必须以本 checkout 为准，不能默认等于上游**（见 C8）。

## Claim-Source 对照

### C1. 插件是函数/类/对象三种形态，经 `ctx.plugin()` 归一化为 callback

- 锚点：`vendor/cordis/src/registry.ts:92-133`（Plugin 类型）、`:222-228`（resolve）、`:316-335`（plugin()）
- 结论：函数插件直接作为 callback；对象插件取其 `apply` 方法（`isApplicable` 判断 `typeof object.apply === 'function'`）；类插件在 fiber 执行时 `new`。callback 是注册表身份键（`_internal: Map<Function, Runtime>`）。
- 对应官方口径：primer "插件是实现 Service 的对象"（函数带 inject/apply 或 Service 子类）。
- `Status: Observed`

### C2. 插件生命周期 = Fiber 状态机；inject 门控经 epoch 机制实现

- 锚点：`vendor/cordis/src/fiber.ts:147-154`（FiberState）、`:597-623`（`_checkImpl` / `_refresh`）、`:625-639`（`_setEpoch`）、`:646-696`（`_reload` / `_unload`）
- 结论：`PENDING`（等依赖）→ `LOADING`（callback 运行）→ `ACTIVE`（提供中）；`FAILED` / `UNLOADING` / `DISPOSED`。服务出现/消失时 `reflect.notify()` 对每个注入该服务的 fiber 调 `_checkImpl`（含 `impl.check` 可用性谓词）+ `_refresh`（按注入服务 uid 链算 epoch）；epoch 变化 → `_unload` + `_reload`（插件 body 在依赖齐备后才运行）。
- 对应官方口径：primer "inject 声明依赖，服务存在后才运行"、"回调在依赖服务变化时卸载并重跑"。
- `Status: Observed`

### C3. 服务注册是 effect；`ctx.<key>` 解析走 Proxy + fiber 链 + isolate 边界

- 锚点：`vendor/cordis/src/reflect.ts:277-305`（provide 即 `fiber.effect`）、`:135-171`（proxy get 服务解析）、`:314-336`（notify）；`vendor/cordis/src/service.ts:42-59`（Service 构造即 `ctx.reflect.provide`）
- 结论：`Service` 子类构造时调用 `ctx.reflect.provide(name, self, check)` → 注册进 `fiber.effect`（卸载时自动移除）；实现存于 `store[isolateLabel]`。`ctx.<key>` 读取经 Proxy → `internal/get` waterfall → 沿 fiber 父链向上找 `store`，`fiber.parent[symbols.isolate][prop] !== key` 即停止（isolate 边界）；`ctx.isolate(name, label)` 创建独立服务作用域——architecture.md 的 `isolate` realm 即此原语。
- 对应官方口径：primer "context 是服务仓库，按 key 发现"；根 AGENTS.md "registrations are effects"。
- `Status: Observed`

### C4. 事件注册即 effect；分发模式与 waterfall 语义

- 锚点：`vendor/cordis/src/events.ts:254-260`（register 即 effect，disposer 移除 listener）、`:234-243`（waterfall 组合）、`:165-175`（dispatch + context filter）；`vendor/cordis/src/fiber.ts:415-561`（effect 机制：执行即注册、disposers 逆序回卷、double-call no-op）
- 结论：`ctx.on` 注册进当前 fiber 的 disposables，fiber unload 自动移除（与"listener 随 owner 卸载"一致）；waterfall 把 listeners 环绕 inner `next` 组合，不调 `next()` 即 veto 整链；`bail` / `serial` / `parallel` / `emit` 语义与 primer 分发模式表一致（emit 不等待、parallel 并行等待、serial 按序等待至 bail、bail 同步首值短路）。
- 对应官方口径：primer §Dispatch Modes、§Waterfall Semantics。
- `Status: Observed`

### C5. 插件 config 经 standard-schema 校验，且在注入激活后解析

- 锚点：`vendor/cordis/src/fiber.ts:50-62`（resolveConfig）、`:641-644`（`_resolveConfig` 经 `internal/config` waterfall）、`:646-673`（`_reload` 中 config 先于 body 解析）
- 结论：插件 body 运行前，raw config 经 runtime 的 `Config`（standard-schema）校验，`ValidationError` 聚合 issue 消息；config 解析走 `internal/config` waterfall，发生在注入激活后——对应 vendor/README 本地修改 #15（懒解析：config 表达式在注入激活后求值，仅作用于 entry root）。
- `Status: Observed`

### C6. dsh 侧 Service Definition 实例：`SessionStore extends Service` 声明 `ctx.sessions`

- 锚点：`packages/core/session/src/index.ts:792`（`class SessionStore extends Service`）、`:37`（`declare module '@deepseek-ai/cordis'` 合并 Context 类型）、`:798`（`ctx.inject(['typert'], ...)`）
- 结论：core/session 包承担 capability seam 的 **Service Definition 角色**（声明 `ctx.sessions`）；TS 声明合并把 `ctx.sessions` 加入 Context 类型面；该服务自身也 inject `typert`（跨包依赖）。
- 对应官方口径：architecture.md 核心包表（core/session → `ctx.sessions`）；capability-seams.md 图谱（owner: session）。
- `Status: Observed`

### C7. agent-loop 是 Consumer：inject 声明 + ctx 消费 + waterfall 事件

- 锚点：`packages/core/agent-loop/src/index.ts:371`（`ctx.inject(['sessionPersistence'], ...)`）；`agent.ts:234-235`（`waterfall('agent/pre-step', ...)`）；`tool-calls.ts:169-173`（`ctx.tools[TOOL_RUNTIME_SCHEDULER].dispatch(...)`）；`invariant.ts:21-25`（`ctx.on('llm/stream', (options, next) => ...)` 内 `ctx.sessions.get(...)` 校验）
- 结论：loop 经 `ctx.inject` 声明依赖（`sessionPersistence`）、消费 `ctx.sessions`（enter/announce/prepare）与 `ctx.tools`（TOOL_RUNTIME_SCHEDULER 管线）、经 waterfall 事件（`agent/pre-step`、`llm/stream`）做扩展点；"Model-visible ⟺ logged" 的 runtime invariant 实现在 `invariant.ts` 的 `llm/stream` listener 中（校验模型请求对应的 session 存在）。
- 对应官方口径：architecture.md turn 流程（`agent/pre-step`、`llm/stream` 为 waterfall）；图谱（agent-loop 消费 sessions/tools/llm）。
- `Status: Observed`

### C8. vendored 代码 ≠ 上游：18 条本地修改，loader/include 语义被改写

- 锚点：`vendor/README.md` "Local modifications" #1-18（manifest 表含每个包的上游 commit）
- 结论：`@deepseek-ai/cordis` 4.0.0-rc.7、`cordis-plugin-loader` 1.0.0-rc.5、`cordis-plugin-include` 1.0.4 均为 vendored 拷贝；本地修改直接改变组合语义：#15 懒 config 解析（注入激活后）、#11 `applyEntryPatches` 导出 + **patch 不跨 include 边界**（dsh 把 bundle patch 层、profile/home `cordis.patch.yml`、`--patch` overlay 作为同级 patch list 组合）、#18 `disabled` 是唯一可插值元数据字段。
- 影响：§1.2 组合模型的源码依据是"vendored+本地修改后"的语义，不是上游 Cordis 文档；引用上游文档需谨慎。
- `Status: Observed`（README 自述 + 代码锚点可复核）

### C9. cordis.yml entry → `ctx.plugin()`：loader 的完整桥接链

- 锚点：`vendor/loader/src/config/entry.ts:52-303`（Entry 类）、`vendor/loader/src/index.ts:92-157`（Loader 的 internal 事件接线）
- 结论：
  1. cordis.yml 行（id/name/config/disabled/group/inject）→ `EntryOptions`（entry.ts:9-22）→ `EntryTree` 中的 `Entry` 节点；每个 Entry 有独立子 context（`loader.ctx.extend({ [Entry.key]: this })`，entry.ts:66-69）。
  2. 组合层 = entry context 的原型链：`_patchContext` 中 `Object.setPrototypeOf(this.ctx, this.parent.ctx)`（entry.ts:116）——层序即原型链。
  3. 启动桥接：`Entry._start` → `this.ctx.registry.plugin(importedPlugin, this.options.config, ...)` + `fiber.await()`（entry.ts:291-302）；模块经 `tree.import(name)` 导入 + `unwrapExports` 处理 ESM/CJS default（index.ts:192-199）。
  4. **entry 级 `inject` 的落点**：Loader 的 `internal/plugin` listener 在 fiber 创建时 `Inject.resolve(fiber.entry.options.inject, fiber.inject)`（index.ts:117-123）——cordis.yml 的 inject 字段合并进 fiber 的依赖声明。
  5. `disabled` 求值：`!!js` 表达式经 `evaluate(this.ctx, expr)` 对 loader context 求值（entry.ts:104-108），且祖先 entry 的 disabled 链生效（entry.ts:88-98）；group 恒 enabled。
  6. config 插值：`internal/config` listener 仅对 entry root fiber 执行 `interpolate`，tree carrier（Group/Include，带 `EntryGroup.key`）保持字面量（index.ts:92-101）——本地修改 #15 的代码落点。
  7. 事务性更新：`update()` 带回滚（import → dispose → start → 失败回滚旧插件，entry.ts:142-246）；自毁（self-dispose）经 `internal/plugin` listener 的 7 种 case 判别后持久化 `disabled: true`（index.ts:117-157）。
- 对应官方口径：architecture.md 组合模型；vendor/README 本地修改 #8/#11/#15/#18。
- `Status: Observed`

### C10. self-modification 悬案解决：组改名而非功能缺失

- 锚点：`.agents/notes/implemented/architecture/2026-08-11-repository-naming-contract-and-rename-ledger.md:260`（重命名台账：`packages/self-modification/` → `packages/extensions/`，"The group contains repository plugin inspection and mounting tools. `extensions` states the stable package role without asserting that the agent modifies itself"）；`.agents/notes/implemented/architecture/2026-07-29-package-regrouping.md:14,34,70`（`tool-cordis` is "the runtime self-modification toolset"；组名先拟 `self-evolve/`，后定 `extensions/`）；`packages/extensions/tool-cordis/src/index.ts:26-379`（工具面）
- 结论：
  1. rc.7 中 `tool-cordis` + `cordis-host-runner`（`packages/extensions/` 组）**就是**根 AGENTS.md 所述 `self-modification/` 的能力：agent 检查（inspect：list/query/self 三工具，只读）与挂载（define → run → stop → undefine，模型以纯 JS 函数体定义不可变 Package）。
  2. 根 AGENTS.md 布局过时：`self-modification/` 是 2026-08-11 之前的组名，改名台账为权威记录；Conflicting 项**解决**（文档未同步，非功能缺失）。
  3. 安全语义（工具面可见）：define 仅校验参数与记录源码，不执行；run 对未授权 Client Package 走 approval 请求（awaiting-approval），`currentPackageId` 仅在完全成功后切换；全部插件按 session 归属（`sessionId` 参与 define）。
  4. `agent/pre-step` listener 检测用户消息中的 `@pluginId` 引用并注入动态插件上下文（index.ts:381-398）。
- 状态变化：§4.1 由 `Conflicting` 转为 `Observed`（等价机制确认）+ 根 AGENTS.md 布局标记 `Deprecated`（以改名台账为准）。
- `Status: Observed`

### C11. ReactLoopAgent 完整 turn 流程（与 architecture.md 序图逐行对应）

- 锚点：`packages/core/agent-loop/src/agent.ts:210-223`（kick 驱动）、`:246-330`（turn 边界）、`:332-401`（step 边界）、`:407-495`（buildRequest）、`:225-243`（preStep）
- 结论：
  1. **driver 状态机**：phase = idle / running / maintenance（L38-46）；`wakeDriver`（L172-193）在 idle 时开 running 相位并经 `ctx.agents.withInitiator` 执行 `kick`（initiator 传播）；`kick` 循环 `while (await this.turn()) {}`。
  2. **turn 边界**（与序图一致）：`turn/start` append（L255）→ `preStep`：`inbox.claim(target, turn)` + `systemPrompt.assemble` + `runtimeContext.project` + `agent/pre-step` waterfall（L225-243）→ `step/start` + `user/message`* append（L279-284）→ `step()` → `step/end`（finally，L292）→ 结束时 `agent/turn-stopping` serial（L296）→ `turn/end` 带 reason 必达（L319）。turn 结束原因：blocked / completed / max-tokens / aborted / error。
  3. **step 边界**：`buildRequest` → `ctx.llm.prepareCall`（adapter 解析，NO_ADAPTER 回退）→ stream（preparedCall 或 `ctx.llm.stream`）→ `assistant/chunk`* 逐块 append（带 seq）→ `assistant/message`（surfaceOp append + sourceEventSeqs）→ 无 tool-call 则 completed；有则 `executeToolCalls`（TOOL_RUNTIME_SCHEDULER 管线）→ 未 concluded 则继续下一轮请求（tools 欠一次请求）。
  4. **buildRequest**：`agent/request` waterfall 提出 config（L438-441）→ `prepareCall` → `canonicalHeader` → **`request/header` session 事件**（initial/resume/change，L458-470）+ **`request/context` session 事件**（provider/model/contextWindow 变化时，L472-483）→ 冻结 request（messages 来自 `session.deriveMessages()`）。
  5. **文档未覆盖的两处细化**：`agent/request-error` waterfall（流失败/中止时的重试决策扩展点，L355-365）；`request/header` / `request/context` 作为持久 session 事件（序图未画出）。
  6. **inject 语义验证**：`inject()` = next-step 目标 + 不唤醒 driver（L130-132）——"injected context waits in the inbox until another message does" 与 architecture.md 一致。
- 对应官方口径：architecture.md §Turn flow（主要一致，两处细化如上）。
- `Status: Observed`

### C12. capability seam 三角色代码范式（ctx.shell 样板）

- 锚点：Service Definition `packages/shell/shell/src/index.ts:40-44`（declare module 合并）、`:65-101`（`ShellExecutor extends Service` 抽象契约：resolve/run/start + 语义 JSDoc + "加载第二个实现会抛错"）；Provider `packages/shell/bash-local/src/index.ts:102-137`（`LocalBashExecutor extends ShellExecutor` + `static inject = ['subprocess']` + `static Config` schemastery schema）、`:146-171`（resolve 显式默认与上限）、`:211-240`（run → `ctx.subprocess.spawn`）、`:242-318`（start 后台进程句柄）
- 结论：
  1. **Service Definition 范式**：抽象类 extends `Service`，构造 `super(ctx, 'shell')` 注册；`declare module '@deepseek-ai/cordis'` 把 `ctx.shell` 并入 Context 类型面；契约方法 + 语义 JSDoc（run 只对基础设施失败 reject、start 立即返回、readOutput 增量等）；settings namespace 归 seam 所有（不归实现）；service 包 default-export 服务类（packages/AGENTS.md 约定）。
  2. **Provider 范式**：子类化 Service Definition；`static inject` 声明跨 seam 依赖（bash-local 依赖 subprocess）；`static Config` 提供默认与校验；**`resolve(request): Spec` 显式默认化——run/start 只收 fully-specified spec，绝不内部 `?? default`**（正是根 AGENTS.md "Explicit > implicit" 模板）；provider 也 default-export 类作为插件加载。
  3. **Consumer 范式**（C7 已核验）：tool-bash / hooks 消费 `ctx.shell`；"一个 provider 替换改变整个产品"——bash-local / bash-sandbox / pwsh-local 三实现共享同一 seam，换实现不碰 consumer（architecture.md 断言成立）。
  4. **"完整 = 三角色齐备"**：shell 包单独不构成 seam；duplicate provider 经 Cordis 标准 duplicate-service 行为 fail loud。
- 对应官方口径：architecture.md §Capability seams；根 AGENTS.md capability-seam 约定；packages/AGENTS.md 插件导出约定。
- `Status: Observed`

### C13. guard 组 = 两个纯 listener 插件（"图谱无服务声明"谜底）

- 锚点：`packages/guard/timeout-policy/src/index.ts:28-31`（`export const name/inject = ['tools']` 纯函数插件）、`:55-81`（`ctx.on('tools/execute', ...)` waterfall 包装）；`packages/guard/repeat-tool-reminder/src/index.ts:17`（`export const name`）、`:213-232`（`tools/post-execute` + `agent/pre-step` listeners）
- 结论：
  1. **guard 不注册任何服务**——timeout-policy 与 repeat-tool-reminder 都是纯 listener 插件（named export 函数插件，无 Service 类、无 `ctx.provide`），这解释了 capability 图谱中 guard 无服务声明的现象（图谱只收录服务声明，不收录纯事件监听插件）。
  2. **timeout-policy（tool-timeout）**：`ctx.on('tools/execute')` waterfall——工具声明 `timeoutMs` 字段（`ctx.tools.get(exec.name)?.timeoutMs`），无声明不设限；有声明则 `deadline(exec.signal, timeoutMs, TOOL_TIMEOUT)` 包装，临时替换 `exec.signal` 委托 `next()` 再恢复；仅当自己的 timer 触发（`timeoutOf` 按 code 作用域，避免嵌套外层 deadline 误读）时把结果替换为结构化 `TOOL_TIMEOUT` 错误（模型可见 `Error: tool call timed out after Nms`，`error.code = TOOL_TIMEOUT` 供重试/回放路由）。
  3. **repeat-tool-reminder（重复调用检测，loop-hygiene 核心）**：`tools/post-execute` waterfall 中 observe（per-agent 连续重复链 `WeakMap<Agent, Chain>`，参数 deep key-sort 后 canonicalize；**被拒的调用也计数**——"模型反复打被拒调用正是值得打破的循环"）→ 委托 `next()` → 把 reminder 折叠进 `additionalContexts`（**observe-and-enrich, never veto**）；阈值默认 `[3,5,8]` 升级（gentle → detailed，参数预览 cap 500 字符）；`agent/pre-step` 中用户插话重置链。
  4. 历史关联（对照 source.md 5.3）：`timeout/` 在 2026-08-09 并入 `guard/`；timeout-policy 有 FIXME 计划改名 `@deepseek-ai/dsh-timeout-guard`（改名待办）。
- `Status: Observed`

### C14. approval 决策链：waterfall 单决策 + 三层 fail-closed + 审计对

- 锚点：`packages/interaction/user-approval/src/index.ts:192-198`（`ApprovalService extends Service`，`super(ctx, 'approval')`）、`:22-31`（`approval/request` waterfall 事件声明，scope-filtered dispatch）、`:34-72`（session 事件：`approval/asked`/`approval/decided`/`approval/policy`）、`:257-276`（request：open-turn 前置 + 审计对）、`:304-344`（decide：fail-closed 归一化）、`:226-237`（setPolicy + `agent.inject` 通知）
- 结论：
  1. **决策链**：`request()` 要求 open turn（审计对必须 turn 内，裸事件在 reload 时是 crash-tail 垃圾）→ append `approval/asked` → `decide()` → append `approval/decided`（审计对永不拆散，append 失败即 reject）。
  2. **三层 fail-closed**：aborted → `'cancelled'`；无 answerer / answerer throw / 非词汇表返回值 → 全部归一化 `'unavailable'`（`Promise.resolve().then` 链保证同步 throw 也进 containment；`OUTCOMES.includes` 归一化 rogue 返回值）；'never' 策略在 dispatch 前决定（避免 prepend listener 绕过确定性承诺）。
  3. **approval/request 是 scope-filtered waterfall**：`scopeTarget(this, req.agent)` 限定 agent-scoped listeners 只收本 agent 的请求（answerer 是 listener：ACP bridge 为自己的 agents）。
  4. **模型可见性**：`approval:policy` systemPrompt context section（模型从 runtime-context 快照得知策略）；`setPolicy` 切换时 `agent.inject()` 通知（"policy changed from X to Y (changed by the user)"）；`approval/policy` 是持久 session 事件（"replaying the log IS the state"，resume 无需 catch-up）。
  5. **图谱 owner 字段漂移**：图谱 `ctx.approval` owner 写 `approval` 且无链接——实际包为 `packages/interaction/user-approval`（与 C10 同类但轻量的文档漂移）。
- 对应官方口径：图谱 note（waterfall 单决策 + fail-closed unavailable）完全成立并细化。
- `Status: Observed`

### C15. session log 内部结构：append / deriveMessages / fork（§3.1 剩余闭合）

- 锚点：`packages/core/session/src/index.ts:604-655`（append）、`:657-699`（header/context 增量 fold）、`:726-757`（deriveMessages + deriveEventMessage）、`:1081-1153`（fork）；`packages/core/session/src/surface.ts:83-106`（deriveEventMessage 的 case：user/message、assistant/message、tool/result 等）、`:185-205`（surfaceOp 校验 fail loud）
- 结论：
  1. **append**：事件 = `{ type, seq（= log 长度自增）, time, data（JSON 快照）, 可选 surfaceOp/sourceEventSeqs }`，deepFreeze 不可变；data 必须 JSON 可序列化（否则抛错）；`surfaceManager.validateNext` 校验 surface 顺序；广播 `session/event` 时 observer 失败被 containment（不影响 append 提交）；`appending` 标志防重入。
  2. **deriveMessages（派生历史）**：**surface 是派生历史的唯一来源**——消息产生型 append 必须带 `surfaceOp` marker（surface.ts:185-205 校验：surface-eligible 必须带、非 eligible 不能带，fail loud）；无 marker 的 raw 事件（chunk、turn 边界）正确地不进派生；compaction 的 `replace` 删除被遮蔽节点并触发缓存重建（`surface.replaceGeneration`）。**缓存**：每节点只投影一次（per-step 成本 O(new nodes)）；返回数组是新鲜快照，Message 对象共享且 deep-frozen（复用已冻结的日志数据，无二次深拷贝）；空内容 assistant/message（max-tokens step 仅含 usage）派生为 null 不入转录。派生规则为纯函数 `deriveEventMessage`（surface.ts:83+，case 覆盖 user/message、assistant/message、tool/result 等）。
  3. **requestHeader/requestContext 增量 fold**：`headerFoldSeq`/`contextFoldSeq` 使每事件只 fold 一次，per-step 读成本 O(new events)；结果 deepFreeze（防原地修改破坏与日志的一致性比对）。
  4. **fork**：`fork(source, boundary?, childSessionId?)`——boundary 必须是非负安全整数且等于连续事件 seq（INVALID_BOUNDARY）；**boundary 不得落在 open turn 内**（OPEN_TURN——turn/end 未闭合处不能 fork，与审计对/turn 边界语义一致）；seed = 事件前缀 `events.slice(0, boundary+1)`，create 时 meta 记录 `parentSession`/`seedLength`/`cwd`；fork 源必须是 live store 实例（SESSION_NOT_LIVE）。resume 同构：Session 构造的 seed 即完整存储日志（重放）。
  5. **持久化不在 core/session**：`SessionStore` 是纯内存 store（`Map<SessionId, SessionEntry>`）；持久化由插件订阅 `session/event` 并 flush（index.ts:786-790 注释）——这就是 `ctx.sessionPersistence` seam（jsonl/sqlite 后端）存在的原因。
- 对应官方口径：architecture.md §Session log（"Fork, resume, transcripts, telemetry, and persistence all derive from this stream"）完全成立并细化。
- `Status: Observed`

### C16. 对外投影：ACP server 与 hooks 桥接（§5.1 闭合）

- 锚点：`packages/acp/acp/src/index.ts:45`（`export const inject = ['agents']`）、`:222-252`（`session/event` 输出 assistant 消息）、`:254-266`（`agent/inbox/claimed` + `agent/error` 追踪 inflight）、`:271-279`（`approval/request` answerer）；`packages/hooks/hooks-claude-code/src/index.ts:39-42`（`inject = ['shell']`）、`:96-296`（6 个扩展点 listener + `runHook(ctx.shell, ...)`）、`:322-331`（`base()`：`transcript_path` 经 `ctx.get('sessionPersistence')?.locate()`）
- 结论：
  1. **ACP server（automation-only）**：`inject = ['agents']`——直接依赖 live Agent 服务；经 `ctx.agents.get()` 校验 session 归属（L369/382）；监听 `session/event` 只输出**已提交的 assistant/message**（raw chunk/reasoning/tools/plans/titles/retry markers 是展示或追踪数据，不上自动化线——"Emit only committed assistant text/images"）；`agent/inbox/claimed`/`agent/error` 追踪 inflight 请求；**是 `approval/request` 的 answerer**——经 `conn.requestPermission` 提供 allow-once/reject-once 一次性选项，"从不从未知客户端响应推断持久授权"。
  2. **hooks 桥接（Claude Code dialect）**：`inject = ['shell']`——**只注入 shell**，其余（sessionPersistence、agents）经 `ctx.get` 按需读（部署可加载 bridge 而不必每个扩展点都在）。CC 事件 → dsh 扩展点映射：`agent/session-start`→SessionStart（detached 注入上下文）、`agent/pre-step`→UserPromptSubmit（deny→reject）、`tools/pre-execute`→PreToolUse（matcher=工具名，deny/ask 映射）、`tools/post-execute`→PostToolUse（deny→block，否则 fold 上下文）、`agent/turn-stopping`→Stop（deny→`agent.steer` 强制续跑）、`subagent/start|end`→SubagentStart/Stop。
  3. **hooks 的执行与审计**：hook 命令经 `ctx.shell` 运行（图谱 hooks 消费 shell 的原因），工作目录 = agent 的 session workspace（`session.header.cwd`）；`hook/invoked`+`hook/result` session 事件对（turn 内审计）；payload 字段名匹配 CC 输入 schema（session_id/transcript_path/cwd/hook_event_name 等），`transcript_path` 经 `sessionPersistence.locate()`（图谱 hooks 消费 sessionPersistence 的原因）。
  4. **已知限制（源码 TODO）**：`updatedInput` 不 honored（logged+warn）；`systemMessage` 不 surfaced；Stop 强制续跑无 loop guard（`stop-loop-guard` TODO）；SessionStart 无 startup gate（慢 hook 可能错过首请求）。
  5. **SDK（JSON-RPC）**：协议面，不在服务图谱；TS client + JSON-RPC protocol/server（`packages/sdk/`）——与 hooks 的 wire-protocol 是否构成两套独立协议面仍未源码确认（`Status: Inferred` 保持）。
- 对应官方口径：图谱（acp 消费 ctx.agents；hooks 消费 sessionPersistence + shell）完全成立；architecture.md "Add UI or editor integration: drive ctx.agents and render from session/event" 在 ACP 中落地。
- `Status: Observed`

### C17. system-prompt 组装：section/context/tools/variables 与 assemble 流程（§3.1 最后细节闭合）

- 锚点：`packages/core/system-prompt/src/index.ts:398-455`（context/tools/variable/suppressRuntimeContext 注册，全部 `layers.effect`）、`:467-542`（`assemble()` 完整流程）、`:53-131`（PromptSection/PERSONA_SECTION/PERSONA_ORDER=0）、`:103-118`（ToolProviderResult/PromptAssembly）
- 结论：
  1. **四类注册面**（全部经作用域层 + effect disposer，卸载自动移除）：`section()`（静态或函数化 text，可 `complete: true`）、`context()`（动态 runtime context，按 order 升序）、`tools()`（per-assembly 评估的 tool-schema provider，返回 schemas + knownNames）、`variable()`（prompt 变量，`[a-z][a-z0-9_]*` 命名，作用域遮蔽全局）。
  2. **order 语义**：sections 按 order 升序拼接；`PERSONA_ORDER = 0`（`deployment:persona` 是模型读的第一节）；约定 `-100` 级在 persona 之前。
  3. **assemble 流程**：scope 层链收集（scoped 遮蔽全局）→ tool providers 评估（schemas 的 parameters **structuredClone** 防共享突变；knownNames = pre-restriction 名称宇宙供配置校验）→ sections/contexts 按 order 排序 → **complete section 唯一性检查**（多个 complete 抛错；`complete: true` 的 section 在 waterfall 后被强制为唯一 prompt section——"complete prompt"语义）→ tools 经 `orderTools` 规范排序 → **`system-prompt/assemble` scope-filtered waterfall**（plugins 可改写 assembly）。
  4. **suppressRuntimeContext**：调用作用域内抑制所有动态 context 贡献（不改拥有方）。
  5. 组装时机（对照 C11）：`ReactLoopAgent.preStep` 中 `loopCtx.systemPrompt.assemble(assembleContextFor(this, signal))` 每次 step 调用；tool schemas 经 `assembly.tools` 进入 `buildRequest` 的 canonicalHeader。
- 对应官方口径：architecture.md 核心包表（core/system-prompt → `ctx.systemPrompt`：prompt-section 与 tool-schema 组装）完全成立。
- `Status: Observed`

### C18. agent preset 与 boot profile 的交互边界（§2.1 闭合）

- 锚点：`packages/preset/agent-presets/src/mount.ts:57-112`（`PresetTree extends Include` + import override + write 空实现）、`:332-381`（`mountPreset`：挂载 → inactive rows 审计 → leaked services 审计）、`:189-203`（`leakedServices`：root realm 判定）、`:283-301`（`inactiveRows`）、`:222-230`（standingMountFor）、`:256-272`（serviceForAgent）；`packages/preset/agent-presets/src/index.ts:18,28,154-164`（挂载点 = agent factory 的 `setup(agentCtx)` hook + `agent/created` 事件）
- 结论：
  1. **挂载机制**：agent 创建时（`agent/created` 事件 → factory `setup` hook），`ctx.agentPresets` 按 roster 解析 preset，`mountPreset(agentCtx, preset)` → `agentCtx.plugin(PresetTree, config)`——**PresetTree 是 Include 子类**（复用 loader 的 include 机制），子树挂载在 agent 的 scope context 下，由 agentCtx 的 fiber 拥有（随 agent 卸载回卷，调用方无 disposer）。
  2. **per-session 的本质**：scope context 是组合 per-session 的原因——entry contexts 链到子树插入的 context，preset 内每个 `ctx.tools`/`ctx.systemPrompt` 注册都 file 进该 agent 的 layer 并随 agent 卸载。
  3. **两道安全 guard**（图谱 note 的代码实现）：**(a) inactive rows 拒绝**——行未达可用状态（fiber 未启动 / `fiber.inject` 中服务缺失 waiting for X）→ 挂载失败，因为直接插拔的子树不在 `ctx.loader.entries()` 中、无 boot 审计覆盖；**(b) root realm 泄漏拒绝**——preset 行把服务发布进 ROOT realm（`rootIsolate[impl.name] === key`，即未包 isolate realm）→ 拒绝（"进程全局而非 per-session，第二个会话挂载同一 preset 冲突"；错误消息明确要求 "a preset service must sit behind an `isolate` realm or move to the host composition"）——**isolate realm 从 Cordis 原语（C3）到 dsh 产品约束（preset 安全）的完整链路闭合**。
  4. **与 boot profile 的边界**：boot profile 是 host 组合（loader entries，有 boot 审计、可持久化写回）；agent preset 是 per-session 子树（无 boot 审计所以挂载时自审计；`write()` 空实现——preset 是输入不是持久化目标，继承的 write 会在 agent 拆卸时把 preset 文件截断为 `[]`，必须覆盖）。
  5. **bare specifier 解析**：相对路径从 preset 目录解析；包名从 harness base（宿主组合 baseUrl）解析（本地 preset 在用户 home 下，node_modules 上溯找不到 harness 依赖）；绝对路径转 file URL（Windows 盘符）。
  6. **host 读 preset 内服务**：`serviceForAgent`——READ addressing（host 行不能 `inject` 访问 preset 服务，因为注入在会话存在前解析；经 `standingMountFor` 找 agent 加入的 standing mount 再按服务名查实例）。
- 对应官方口径：图谱 `ctx.agentPresets` note 完全成立并细化（"rejecting a row that never activates or that publishes into the root service realm" 即 inactiveRows + leakedServices）。
- `Status: Observed`

### C19. compaction 策略：双触发 + 定价 + model-free prune 先行（§3.2 闭合）

- 锚点：`packages/compaction/compaction-basic/src/index.ts:104`（`static inject = ['llm', 'tokenMeter', 'sessions']`）、`:137-224`（`_registerAutomaticCompaction` 两个触发点）、`:258-332`（`compactIfNeeded`：pressure/overflow 双路径）、`:236-246`（`summarize` 复用会话 KV cache）、`:368-420`（`compactNow` 手动路径）
- 结论：
  1. **两个自动触发点**（图谱 note 的代码落地）：**(a) `agent/pre-step` → 'pressure'**（step 边界压力，失败 warn 并继续 turn 不阻断）；**(b) `agent/request-error` → 'context-overflow'**（`failure.code === CONTEXT_WINDOW_EXCEEDED_CODE` 时恢复——压缩成功返回 `{ kind: 'retry' }` 重试请求；`maxOverflowRetries` 上限，成功 assistant/message 或 agent idle 重置重试序列）。
  2. **定价**：`ctx.tokenMeter.measure(session)`（单例 replay fold 测量）+ `routedTarget`（最新持久 `request/header` 的 provider/model——压缩定价的是"最新持久路由请求"，不是内存态）。
  3. **pressure 路径**：`resolveModelInfo` 取 contextWindow → `resolveCompactSpec`（thresholdRatio → thresholdTokens；retainRatio/retainTokens → 保留尾部）→ 低于阈值不压缩；**先 model-free prune**（`ctx.get('toolResultPruner')` 可选兄弟服务，`pruneSession` 后重测——toolResultPruner 是"无模型工具结果修剪"，先做免费缩减再决定是否值得 LLM 摘要）→ 仍超阈值 → 循环 `selectCompactableRange` + `compactRegion`（`compactionRetries` 次）→ 仍超阈值抛错。
  4. **overflow 路径**：跳过阈值/保留尾策略，retain 0 强制一次有用缩减（provider 已确认溢出，无需再定价）。
  5. **摘要方式**：`summarize` 经 `ctx.llm.stream()` 直呼，**复用会话自己的 system prompt/tools/messages 前缀——provider 的 KV cache 不失效**（压缩后重试请求的成本显著降低）。
  6. **压缩后仍满足 model-visible ⟺ logged**：压缩 = surface 的 `replace` 操作（C15 已核验：replace 删除被遮蔽节点 + 派生缓存重建）；summary 成为新 surface 节点，日志经 sourceEventSeqs 关联、可重放。
  7. **手动路径**：`compactNow` 在 idle agent 的 `runMaintenance` 内执行（busy 抛 ManualCompactionError），可指定 sourceCommandId 并 flush 到持久层。
- 对应官方口径：图谱（post-step pressure + request-error recovery、无 model-facing compact tool）完全成立并细化。
- `Status: Observed`

## 待补证点（下一轮）

- `vendor/loader/src/config/entry.ts`：cordis.yml entry 行 → `ctx.plugin()` 调用的映射细节（entry 的 `inject` 声明、`disabled` 求值时机）。
- `packages/core/agent-loop/src/agent.ts` 全文逐行核验：完整 turn 流程（claim → pre-step → llm/stream → tools 管线 → turn-stopping）与 architecture.md 序图的对应。
- `packages/guard/`：loop-hygiene / tool-timeout 实现（图谱中无服务声明）。
- `packages/extensions/tool-cordis` + `cordis-host-runner`：self-modification 等价机制核验。
