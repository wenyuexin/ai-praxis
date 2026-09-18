# DeepSeek Harness 深度研究问题树

本文件不是面向普通读者的对象总览，而是 `deepseek-harness` 深度研究的**递进式问题树脚手架**。

它的目标不是重复 `overview.md` 已覆盖的"它是什么、有哪些能力"，而是把后续研究组织成一条可持续推进的主线：

> dsh 的 "everything is a plugin" 架构如何落地——插件如何定义、加载与组合，如何构成 agent 生命周期与 capability seam，这些机制又如何支撑 preset、self-modification、session 持久化等产品能力？

使用方式：

- 当 `overview.md` 已足够支撑初步理解，但不足以支撑机制级研究时，进入本文件（触发判断见 `research-artifacts.md` §3.4）。
- 每个问题都区分：**当前已知**、**待确认**、**下一步研究动作**。
- 后续研究不必重新发明问题，而是沿这棵树逐层回答、补证、收口；每轮核验结论追加到对应节点下方，不另起文件。
- 研究顺序与阶段划分见 `roadmap.md`，本文件只负责问题级递进。
- **版本范围**：本文「当前已知」均基于本地 checkout 版本锚点 `0.1.0-rc.7`（commit `99f6f02fecdb`，2026-08-17）；版本变化的影响见第 6 层与 `roadmap.md` 阶段三，不要混入新版本观察。
- **自述与结构冲突的处理**：上游文档/根 `AGENTS.md` 的自述与 checkout 实际结构冲突时，以 checkout 为准并标 `Conflicting` 待核验（实例：`self-modification` 包缺失，见 §4.1；`packages/` 实际包列表多于根 AGENTS.md 布局，见 §7）。
- **核验纪律**：源码核验结论应带锚点（`文件:行号` 或 commit）；源码直接可证为 `Observed`，从布局/文档推断为 `Inferred`，两者分开写，不要混标。
- **建档状态**：`notes/source.md` 已建档第一批三篇（architecture / capability-seams / cordis-primer，`Status: Verified`）；下文「官方口径」条目均可在其中复核。

---

## 0. 主线问题

### 0.1 主问题

dsh 的 "everything is a plugin" 架构如何落地——插件如何定义、加载与组合，如何构成 agent 生命周期与 capability seam，这些机制又如何支撑 preset、self-modification、session 持久化等产品能力？

### 0.2 为什么这条主线值得优先研究

当前 `overview.md` 已经能告诉读者：

- dsh 是 DeepSeek AI 开源的 agent harness，核心主张是 everything is a plugin；
- 运行时基于 vendored Cordis，仓库布局含 core / 各 capability 包 / preset / self-modification / acp / sdk 等；
- 它处于 developer preview，存在破坏性变更，必须按版本研究。

但如果想深入理解，还缺少下面这些机制级答案：

- "插件"在这个系统里到底是什么对象，注册与组合的生命周期如何；
- capability seam 三角色（Service Definition / Service Provider / Consumer）在代码里的实现边界；
- preset（cordis.yml）与 bundle 如何把插件组装成 per-session agent；
- agent-loop、session 持久化、compaction 如何串成一次任务的完整生命周期；
- self-modification 如何让 agent 检查/挂载自身插件，能力边界与安全约束是什么；
- ACP / hooks / SDK 如何对外投影这个 loop，交互与审批如何介入。

因此，这条主线可以作为后续深度研究的第一优先级。

---

## 1. 插件层（everything-is-a-plugin 如何落地）

### 1.1 "插件"到底是什么对象

#### 当前已知

- 上游自述：架构主张是 everything is a plugin，运行时基于 vendored Cordis（本地 checkout 根 `AGENTS.md` / `README.md`，`Status: Observed`，待源码核验）。
- 仓库约定：所有注册行为通过 `ctx.effect()` / `ctx.on()` 完成，registry 的 `register()` 返回 disposer（根 `AGENTS.md`，`Status: Observed`）。
- 配置组合入口是 `cordis.yml`，允许 `!!js`（非 `!js`）出现在 plugin `config` 与 entry `disabled` 下，其余元数据保持字面量（根 `AGENTS.md`，`Status: Observed`）。
- 官方口径（`docs/cordis-primer.md`，`Status: Verified`，已建档见 `notes/source.md` 二）：插件是实现 Service 的对象——带可选 `inject` / `apply(ctx)` 字段的函数，或 Cordis 把生命周期挂进 context 的 `Service` 子类；context 是服务仓库，服务声明 `ctx.<key>` 供按 key 发现（不 import 实现）；依赖经 `inject` 声明，服务存在后才运行；注册经 `ctx.effect()` / `ctx.on()` 可逆安装。
- 源码核验（`Status: Observed`，锚点见 `notes/evidence.md` C1-C5）：插件三形态（函数/类/`{apply}`）经 `ctx.plugin()` 归一化为 callback（registry.ts:222-228）；生命周期 = Fiber 状态机 PENDING→LOADING→ACTIVE，inject 门控经 epoch 机制（fiber.ts:597-639），服务出现/消失触发依赖方 unload+reload；`ctx.<key>` 解析走 Proxy → `internal/get` waterfall → 沿 fiber 父链、受 isolate 边界约束（reflect.ts:135-171）；服务注册 `provide` 即 effect（reflect.ts:277-305）；事件注册 `ctx.on` 即 effect（events.ts:254-260）。

#### 待确认

- `vendor/loader/src/config/entry.ts`：cordis.yml entry 行 → `ctx.plugin()` 调用的映射细节（entry 的 `inject` 声明、`disabled` 求值时机）；
- 循环依赖/依赖缺失时的行为（epoch 机制如何避免 reload 死循环；`await()` 的错误传播）；
- `isolate` realm 在 dsh preset 中的实际用法（agent preset 挂载时是否创建 isolate 子作用域）。

#### 下一步研究动作

- 建档：已完成（`notes/source.md` 二）。源码核验第一轮已完成（`notes/evidence.md` C1-C5）。
- 源码：核验 `vendor/loader/src/config/entry.ts` 的 entry → plugin 映射；对照 `packages/preset/agent-presets` 看 isolate 用法。

### 1.2 插件如何加载与组合

#### 当前已知

- 上游自述：preset 是"per-session agent composition from preset cordis.yml files"（根 `AGENTS.md`，`Status: Observed`）。
- 条件组合使用 overlay（根 `AGENTS.md` 提到 conditional composition 用 overlays，`Status: Observed`）。
- 官方口径（`docs/architecture.md` §Profiles and bundles + `docs/cordis-primer.md` §Loader Configuration，`Status: Verified`）：运行中的 dsh = 启动时按有序层组成的插件树；**bundle** = 配置行 + 代码的分发单元，**profile** = 命名堆叠（bundles + out-of-tree 插件 + 用户 `cordis.patch.yml`）；层序 = profile 列出的各 bundle 按序 → profile `cordis.patch.yml` → home 级 patch → `--patch` overlay；patch 按 id 定位一行并整体替换 config，或插入新行；`!!js` 由 `@deepseek-ai/cordis-plugin-include` 解析为表达式节点，`config` 在注入激活后插值、`disabled` 在每次挂载决策时插值，其余元数据保持字面量。
- 源码核验（`Status: Observed`，`notes/evidence.md` C8）：`vendor/` 是 vendored 拷贝且带 **18 条本地修改**——loader/include 语义被改写：patch 不跨 include 边界（dsh 把 bundle patch 层、profile/home `cordis.patch.yml`、`--patch` overlay 作为同级 patch list 组合）；`disabled` 是唯一可插值元数据字段；config 表达式在注入激活后懒解析。引用上游 Cordis 文档需谨慎，以本 checkout 为准。
- 源码核验（`Status: Observed`，`notes/evidence.md` C9）：loader 完整桥接链——cordis.yml 行 → `EntryOptions` → `EntryTree` 节点（每 entry 独立子 context）；组合层 = entry context 原型链（`setPrototypeOf(this.ctx, parent.ctx)`）；启动桥接 `Entry._start` → `ctx.registry.plugin(plugin, config)`；**entry 级 `inject` 经 `internal/plugin` listener 合并进 fiber.inject**（index.ts:117-123）；`disabled` 懒求值 + 祖先链；config 插值仅对 entry root（tree carrier 保持字面量）；更新事务带回滚、自毁持久化 `disabled: true`。

#### 待确认

- `packages/boot/app-boot` 的 dsh 侧组合实现（profile 层序如何变成 patch list 喂给 include）；
- overlay 的合并语义与优先级（源码级）；
- agent preset（`ctx.agentPresets`：agent 创建时挂载一个 preset cordis.yml 于 agent scope，拒绝永不激活或发布进 root service realm 的行）与 boot 级 profile 的交互边界。

#### 下一步研究动作

- 建档：`docs/config-catalog.md` 未建档。
- 源码：核验 `packages/boot/app-boot`（组合机制）、`packages/preset/agent-presets`、`packages/bundle/` 的入口与组合逻辑；loader entry 映射已完成（`notes/evidence.md` C9）。

---

## 2. 组合层（preset / bundle / 能力如何组装成 agent）

### 2.1 per-session agent 如何从 preset 组装出来

#### 当前已知

- 上游自述：preset 按会话组合 agent（`packages/preset/`），`packages/examples/` 提供 demo bundles（agent-spine + CLI / ACP / JSON-RPC bins）（根 `AGENTS.md`，`Status: Observed`）。
- 官方口径（`docs/architecture.md`，`Status: Verified`）：profile 是 boot 级命名组合；`web` / `headless` 随附为模板；`dsh-base` 是每个 profile 的第一层（model adapters / tools / persistence / sandbox + approval policy / settings / credentials / telemetry），`dsh-web-app` 加浏览器应用、`dsh-headless` 加一次性 runner。
- 官方口径（图谱，`Status: Verified`）：`ctx.agentPresets`（`packages/preset/agent-presets`）在**agent 创建时**挂载一个 preset cordis.yml 于 agent scope——per-session 组合与 boot 级 profile 分层。
- 源码核验（`Status: Observed`，`notes/evidence.md` C18）：**preset 挂载机制**——`PresetTree extends Include`（复用 loader include 机制），agent 创建时（`agent/created` → factory `setup` hook）挂载于 agent scope context，随 agent 卸载回卷；两道安全 guard（inactive rows 拒绝 + **root realm 泄漏拒绝**——preset 服务必须 `isolate` realm，否则进程全局冲突）；与 boot profile 的边界（boot = host 组合有 boot 审计 + 可写回；preset = per-session 子树自审计 + `write()` 空实现防截断）；bare specifier 从 harness base 解析；host 经 `serviceForAgent` 读 preset 内服务。

#### 待确认

- `packages/boot/app-boot` 的 dsh 侧组合实现（profile 层序 → patch list 的 dsh 侧接线，loader 侧已闭合 C9）。

#### 下一步研究动作

- 建档：`docs/config-catalog.md` 未建档。
- 源码：preset 交互边界已闭合（`notes/evidence.md` C18）；如需续读，核验 `packages/boot/app-boot` 的组合接线。

### 2.2 能力包如何通过 capability seam 接入

#### 当前已知

- 上游自述：capability seam 由 Service Definition / Service Provider / Consumer 三角色构成，且"完整"指三者齐备，不是单一角色（根 `AGENTS.md`，`Status: Observed`）。
- 官方口径（`docs/architecture.md`，`Status: Verified`）：seam = 可替换能力，包可合并角色，单角色不构成 seam；新增能力 = 设计全部三角色。Filesystem 与 subprocess provider 共享同一执行世界——指向远端 sandbox 时 Bash、PTY、LSP 一起迁移，无 provider fork。
- 官方口径（`docs/capability-seams.md` 生成图谱，`Status: Verified`）：图谱按包级列出每服务的 owner / implementations / direct consumers / companion plugins，三角色映射完备（60+ 服务；由 `scripts/gen-doc-graphs.ts` 生成，带 completeness guard）。
- 源码核验（`Status: Observed`，`notes/evidence.md` C6-C7）：Service Definition 代码形态 = `class SessionStore extends Service` + `declare module` 合并 Context 类型（session/src/index.ts:792,37）；Consumer 代码形态 = loop 的 `ctx.inject(['sessionPersistence'])` + `agent.ctx.sessions.enter/announce` + `ctx.tools[TOOL_RUNTIME_SCHEDULER].dispatch`（agent-loop/src/index.ts:371、tool-calls.ts:169-173）。
- 源码核验（`Status: Observed`，`notes/evidence.md` C12）：三角色完整代码范式已以 `ctx.shell` 样板闭合——Service Definition = 抽象类 extends `Service` + `super(ctx, 'shell')` + declare module 合并 + 语义契约 JSDoc + settings namespace 归 seam（shell/src/index.ts:65-101）；Provider = 子类化 Service Definition + `static inject`（跨 seam 依赖）+ `static Config` + **`resolve(request): Spec` 显式默认化（run/start 不收 raw request，正是"Explicit > implicit"模板）**（bash-local/src/index.ts:102-171）；Consumer 经 `ctx.shell` 消费（tool-bash / hooks）；duplicate provider fail loud。

#### 待确认

- `tool-bash` 作为 Consumer 的完整调用链（schema → pre-execute → `resolve` → `run`/`start` 的接线细节，可作样板续读）；
- 三角色拆分独立演进的触发条件在代码里如何体现（何时 provider 独立成包、何时合并角色）。

#### 下一步研究动作

- 建档：已完成（`notes/source.md` 三）。
- 源码：以 `ctx.shell` 或 `ctx.fs` 为样板，从声明包 → provider 包 → consumer 包下钻核验三角色实现。

---

## 3. 生命周期层（一次任务如何跑起来）

### 3.1 session / system-prompt / tools / agent-loop 如何串成任务

#### 当前已知

- 上游自述：`packages/core/` 是产品 API 主干：session、system-prompt、tools、agent、agent-loop（根 `AGENTS.md`，`Status: Observed`）。
- 上游自述：`packages/session/` 负责 durable session data：persistence、projection、titles、telemetry；`SESSION_FORMAT_VERSION` 保持 0 且无兼容承诺（根 `AGENTS.md`，`Status: Observed`）。
- 上游自述："Model-visible ⟺ logged"：任何到达模型请求的内容都必须能从 session log 重建（根 `AGENTS.md`，`Status: Observed`）。
- 官方口径（`docs/architecture.md`，`Status: Verified`，已建档见 `notes/source.md` 1.5）：**step** = 一次 model request + 其调用的 tools；**turn** = 零或多个 step。完整流程：`turn/start` → claim 输入 → 组装 prompt 段 + tool schema → `agent/pre-step`（reject | enter）→ `step/start` → 追加 entered messages → 从日志派生 model history → `agent/request` → `llm/stream` → `assistant/chunk*` → `assistant/message` → `tool/call*` → `tools/pre-execute` → `tools/execute` → `tools/post-execute` → `tool/result*` → `step/end` → `agent/turn-stopping` → `turn/end`。
- 官方口径：事件三分域——session（持久，`session/event` 广播）、agent（`agent/*` 携带 live Agent）、capability（`fs/*`、`tools/*`、`telemetry/*` 挂策略/适配器）；持久事件 = `turn/*`、`step/*`、`user/message`、`assistant/*`、`tool/*`。
- 官方口径：waterfall 事件 = `agent/pre-step`、`agent/request`、`llm/stream`、三个 `tools/*`（listener 必须 `next()`）；`agent/turn-stopping` 是 serial 无 `next()`。
- 官方口径（图谱，`Status: Verified`）：`ctx.agentLoop` 是唯一具体 loop 插件（bundle 角色）；扩展包依赖 dsh-agent 事件与服务，不依赖该包。
- 源码核验（`Status: Observed`，`notes/evidence.md` C4/C7）：waterfall 组合实现 = listeners 环绕 inner `next`（events.ts:234-243），不调 `next()` 即 veto；`agent/pre-step`、`llm/stream` 在 loop 中的真实调用点（agent.ts:234-235、invariant.ts:21-25）；"Model-visible ⟺ logged" 的 invariant 实现于 `llm/stream` listener（invariant.ts:21-25，校验 `ctx.sessions.get(options.sessionId)`）。
- 源码核验（`Status: Observed`，`notes/evidence.md` C11）：`ReactLoopAgent` 完整 turn 流程与官方序图逐行对应——turn/step 边界、`preStep`（claim → systemPrompt.assemble → runtimeContext.project → `agent/pre-step` waterfall）、`buildRequest`（`agent/request` waterfall → `prepareCall` → `request/header` / `request/context` session 事件 → `session.deriveMessages()` 作模型历史）、`step`（chunk 逐块 append → assistant/message → `executeToolCalls` → 欠请求循环）、`turn/end` 带 reason 必达（blocked/completed/max-tokens/aborted/error）、inject 不唤醒语义。**文档未覆盖的两处细化**：`agent/request-error` waterfall（流失败重试决策）、`request/header` / `request/context` 持久事件。
- 源码核验（`Status: Observed`，`notes/evidence.md` C15）：session log 内部结构——append（不可变事件 + seq + JSON 快照 + surface 校验 + observer containment）、`deriveMessages`（**surface 是派生历史唯一来源**：带 surfaceOp marker 的消息产生型事件才进派生；增量缓存 O(new nodes)；compaction replace 触发重建；空内容 assistant/message 不入转录）、header/context 增量 fold、fork（boundary 校验 + 不得落在 open turn 内 + 前缀复制）、**持久化不在 core/session**（`ctx.sessionPersistence` seam 订阅 `session/event`）。
- 源码核验（`Status: Observed`，`notes/evidence.md` C17）：system-prompt 组装——四类注册面（section/context/tools/variable，全部经作用域层 + effect disposer）；order 语义（升序、`PERSONA_ORDER=0` persona 第一节、`complete: true` 唯一节强制为唯一 prompt）；tool schemas（provider per-assembly 评估 + parameters structuredClone + knownNames 校验 + orderTools 规范排序）；`system-prompt/assemble` scope-filtered waterfall 供插件改写；组装时机 = `preStep` 每 step 调用。

#### 待确认

- `agent/request-error` 与 compaction（`request-error recovery` 事件）的联动（可选深度）。

#### 下一步研究动作

- 建档：`docs/agent-lifecycle.md`（时序图官方版）未建档，可作 C11 的对照复核。
- 源码：**§3.1 生命周期层已全部闭合**（C11/C15/C17）；如需续读，核验 `agent/request-error` 与 compaction 联动。

### 3.2 compaction 与上下文管理

#### 当前已知

- 上游自述：`packages/compaction/` 是 compaction capability + basic provider（根 `AGENTS.md`，`Status: Observed`）。
- 官方口径（图谱 + architecture.md，`Status: Verified`）：`ctx.compaction` 是 seam，实现 `compaction-basic` 消费 **post-step pressure 与 request-error recovery** 事件；**无 model-facing compact tool**；配套 `ctx.tokenMeter`（per-session replay folds、不可变修订版测量）与 `ctx.toolResultPruner`（压缩前重写超大 tool result 的 replayable 替换）。
- 源码核验（`Status: Observed`，`notes/evidence.md` C19）：**compaction 策略**——双自动触发（`agent/pre-step` → pressure：失败 warn 不阻断 turn；`agent/request-error` → context-overflow：`CONTEXT_WINDOW_EXCEEDED` 时压缩并 `{ kind: 'retry' }` 重试，maxOverflowRetries 上限）；定价 = `tokenMeter.measure` + 最新持久 `request/header` 路由；pressure 路径（thresholdRatio 定价 → **model-free prune 先行**（toolResultPruner）→ 仍超阈值才 LLM 摘要 → compactionRetries 重试）；overflow 路径（retain 0 强制一次缩减）；**摘要复用会话自己的 system/tools/messages 前缀——provider KV cache 不失效**；压缩 = surface `replace`（C15 机制）故仍满足 model-visible ⟺ logged；手动路径 `compactNow`（idle + runMaintenance + flush）。

#### 待确认

- `selectCompactableRange` 的具体区域选择算法（region.ts，可选深度）。

#### 下一步研究动作

- 建档：`docs/persistence-catalog.md` 与 `docs/glossary.md` 未建档。
- 源码：compaction 策略已核验（`notes/evidence.md` C19）；如需续读，region.ts 的区域选择算法。

---

## 4. 能力边界层（self-modification 与安全治理）

### 4.1 self-modification 的能力边界

#### 当前已知

- 上游自述：根 `AGENTS.md` 布局声称存在 `packages/self-modification/`（agent 检查并挂载自己的插件）。
- **悬案已解决（`notes/evidence.md` C10，`Status: Observed`）**：上游改名台账（`.agents/notes/implemented/architecture/2026-08-11-repository-naming-contract-and-rename-ledger.md:260`）记录 `packages/self-modification/` → `packages/extensions/`；`tool-cordis` 即"runtime self-modification toolset"（2026-07-29 package-regrouping note）。rc.7 中该能力由 `tool-cordis`（6 个模型工具：inspect_list / inspect_query / inspect_self / define / run / stop / undefine）+ `cordis-host-runner`（`ctx.dynamicCordisRunner` / `ctx.cordisInspect`）实现。根 AGENTS.md 布局未同步改名 → 标记 `Deprecated`（以改名台账为准），非功能缺失。
- 安全语义（工具面观察）：define 只校验参数并记录源码、不执行不切版本；run 对未授权 Client Package 走 approval（awaiting-approval），`currentPackageId` 仅完全成功后切换；插件按 session 归属；`agent/pre-step` 检测 `@pluginId` 引用并注入动态插件上下文。

#### 待确认

- `cordis-host-runner` 内部的挂载机制细节（vm sandbox 隔离、guard 校验、registry 生命周期——`cordis-host-runner/src/{registry,sandbox,guard}.ts` 未逐行核验）；
- 动态插件与 `guard/`、`interaction/`（approval、permission）的联动细节；
- `native/` 的 landlock-run 是否作为沙箱/隔离机制参与（`Status: Inferred`，landlock 的具体角色待核验）。

#### 下一步研究动作

- 建档：architecture.md 已完成（`notes/source.md` 一）；`docs/glossary.md` 未建档。
- 源码：如需深挖动态挂载机制，读 `cordis-host-runner/src/registry.ts`、`sandbox.ts`、`guard.ts`（本轮已确认工具面与安全语义，机制内核非当前主线）。

### 4.2 权限与审批如何介入执行

#### 当前已知

- 上游自述：`packages/interaction/` 承载 approval / interaction capabilities、permission、commands、ask-user（根 `AGENTS.md`，`Status: Observed`）。
- 上游自述：`packages/fs/` 是 filesystem capability + policy；`packages/settings/`、`packages/credentials/` 提供配置与凭据引用（根 `AGENTS.md`，`Status: Observed`）。
- 官方口径（图谱，`Status: Verified`）：`ctx.approval` = 一次性权限决策，经 `approval/request` waterfall 分发，answerer 是 listener（ACP bridge 为自己的 agents），**无应答 fail-closed 到 `unavailable`**；`ctx.permissionPresets` = `workspace-write` / `danger-full-access` 预设表（捆绑 sandbox-mode 与 approval-policy 两个 knob，一次写一个 `permission/preset` 事件）；`ctx.sandboxPolicy` = 默认 mode + workspace root 的唯一归属（bash 与 fs 读同一服务，避免不同 root）；`ctx.userQuestions` = 人类问答 seam（`tool-ask-user` 在 provider-neutral `ask()` promise 上暂停工具调用）。
- 源码核验（`Status: Observed`，`notes/evidence.md` C13）：**guard 组 = 两个纯 listener 插件**（图谱无服务声明的谜底）——`timeout-policy`（`tools/execute` waterfall 包装 deadline，结构化 `TOOL_TIMEOUT` 替换结果）+ `repeat-tool-reminder`（`tools/post-execute` 观察连续重复调用，被拒调用也计数，observe-and-enrich never veto）。
- 源码核验（`Status: Observed`，`notes/evidence.md` C14）：**approval 决策链**——`ApprovalService extends Service`（`user-approval` 包，图谱 owner 字段漂移）；`approval/request` scope-filtered waterfall；三层 fail-closed（aborted→cancelled / 无 answerer·throw·rogue 返回值→unavailable / 'never' 策略 dispatch 前确定性拒绝）；`approval/asked`+`approval/decided` 审计对（open-turn 前置、永不拆散）；`approval/policy` 持久事件（重放即状态）；模型可见性 = `approval:policy` systemPrompt section + setPolicy 时 `agent.inject` 通知。

#### 待确认

- 凭据引用（credential-reference）的解析与暴露边界（`ctx.credentials` 按操作解析，轮换凭据下一请求即生效——官方口径已明，代码待核验）；
- `permission-presets` 的"一次写一个 `permission/preset` 事件打通两个 knob"的代码实现（可选深度）。

#### 下一步研究动作

- 建档：`docs/config-catalog.md` 与 `docs/tool-execution-pipeline.md` 未建档。
- 源码：guard 与 approval 已核验（`notes/evidence.md` C13-C14）；如需续读，核验 `packages/credentials/credentials` 与 `packages/interaction/permission-presets`。

---

## 5. 接口层（loop 如何对外暴露）

### 5.1 ACP / hooks / SDK 如何投影 agent loop

#### 当前已知

- 上游自述：`packages/acp/` 是 automation-only Agent Client Protocol server；`packages/hooks/` 是 Claude Code / Codex hook bridges + wire-protocol；`packages/sdk/` 是 JSON-RPC protocol、server 与 TypeScript client（根 `AGENTS.md`，`Status: Observed`）。
- 上游自述：`packages/api/` 是 remote BFF assembly + Typert RPC gateway；`python/` 是 Python SDK 与 bundled runtime（根 `AGENTS.md`，`Status: Observed`）。
- 官方口径（图谱，`Status: Verified`）：`ctx.typert`（runtime type registry，owner `typert-registry`）与 `ctx.typertGateway`（Typert Host invocation gateway，把 Remote descriptors 与 live Cordis services 关联，经 Connection RPC carrier 暴露 unary 调用）；`acp` 包消费 `ctx.agents`；`hooks-claude-code` / `hooks-codex` 消费 `ctx.sessionPersistence` 与 `ctx.shell`；SDK（JSON-RPC）不在图谱（协议面）。
- 源码核验（`Status: Observed`，`notes/evidence.md` C16）：**ACP server**——`inject = ['agents']`，经 `session/event` 只输出已提交的 assistant/message（raw chunk/reasoning/tools 不上自动化线），`agent/inbox/claimed`+`agent/error` 追踪 inflight，是 `approval/request` 的 answerer（allow-once/reject-once 一次性选项，不推断持久授权）。**hooks-claude-code**——`inject = ['shell']`（其余 `ctx.get` 按需读），CC 事件 → dsh 扩展点 6 映射（SessionStart/UserPromptSubmit/PreToolUse/PostToolUse/Stop/SubagentStart-Stop），hook 命令经 `ctx.shell` 在 session workspace 运行，`hook/invoked`+`hook/result` 审计对，`transcript_path` 经 `sessionPersistence.locate()`；已知限制：`updatedInput` 不 honored、Stop 续跑无 loop guard、SessionStart 无 startup gate。

#### 待确认

- hooks 的 wire-protocol 与 SDK 的 JSON-RPC 是否构成两种相互独立的对外协议面（`Status: Inferred` 保持，`packages/sdk/` 协议面未逐行核验）；
- `hooks-codex` 与 `hooks-claude-code` 的 dialect 差异点（payload 字段名/事件映射）。

#### 下一步研究动作

- 建档：`docs/api-gateway.md` 与 `docs/cordis-api/` 未建档。
- 源码：ACP 与 hooks-claude-code 已核验（`notes/evidence.md` C16）；如需续读，对照 `packages/hooks/hooks-codex` 的 dialect 差异。

---

## 6. 演进层（developer preview 与版本差异）

### 6.1 破坏性变更如何影响既有结论

#### 当前已知

- 上游自述：developer preview，存在兼容性破坏变更；SQLite 用单调 `SCHEMA_VERSION`，`dsh-session` 的 `SESSION_FORMAT_VERSION` 保持 0 且无兼容承诺（根 `AGENTS.md`，`Status: Observed`）。
- 当前本地版本锚点：`0.1.0-rc.7`（commit `99f6f02fecdb`，2026-08-17）。
- 方法补充（`Status: Observed`）：`docs/capability-seams.md` 由 `scripts/gen-doc-graphs.ts` 从源码生成，可作为版本差异的快速对照物——每次版本切换后重跑/重读该图谱，即可发现服务级增删与角色变化。

#### 待确认

- 版本差异表骨架已建立（`notes/source.md` 五：版本谱系、rc.6→rc.7 变更采样、命名演化、影响评估）；逐版本机制级变更的穷举梳理待续（source.md 5.5）；
- 版本变化是否改变上述各层的机制结论——当前已识别高风险项：C9（PTC Mode 涉及 preset 概念）、C11（agent-presets/acp 相关改动）、C10（extensions 命名需跟改名台账）；
- 本地 checkout 与上游 main 的差异是否已影响已记录结论。

#### 下一步研究动作

- 完成：版本差异表骨架（`notes/source.md` 五）。
- 待续：逐版本机制级变更穷举（0.0.1-rc.5 与 0.1.0-rc.1 两次大重构的内部结构变化）；新版本锚点出现时更新差异表并检查 C1-C12 受影响项（source.md 5.5 联动约定）。

---

## 7. 当前研究状态面板

### 已相对清楚

- 对象定位与核心主张（everything is a plugin + vendored Cordis，官方口径已建档）。
- 组合模型：profile / bundle / patch 层序（architecture.md，官方口径已闭合）。
- turn/step 流程与事件三分域、waterfall 清单（architecture.md，官方口径已闭合）。
- capability seam 三角色 + 服务图谱包级映射（capability-seams.md 生成图谱，官方口径已闭合）。
- approval / permission-presets / sandbox / compaction 官方口径（图谱）。
- **插件层源码机制**：Fiber 生命周期状态机、inject epoch 门控、provide/on 即 effect、ctx.<key> Proxy 解析 + isolate 边界（`notes/evidence.md` C1-C5，`Status: Observed`）。
- **组合层源码机制**：loader entry → `ctx.plugin()` 完整桥接链、entry 级 inject 合并、config 插值边界、事务性更新（`notes/evidence.md` C9）。
- **生命周期层源码机制**：`ReactLoopAgent` 完整 turn/step 流程与官方序图对应，含两处文档未覆盖细化（`agent/request-error` waterfall、`request/header`/`request/context` 持久事件）（`notes/evidence.md` C11）。
- **capability seam 三角色代码范式**：以 `ctx.shell` 样板闭合（Service Definition / Provider / Consumer 完整范式 + `resolve(request): Spec` 显式默认化）（`notes/evidence.md` C12）。
- **self-modification 悬案解决**：`self-modification/` 组已改名 `extensions/`（2026-08-11 改名台账），`tool-cordis` 即其工具面；根 AGENTS.md 布局 `Deprecated`（`notes/evidence.md` C10）。
- **vendored 语义前提**：vendor/ 带 18 条本地修改，以 checkout 为准（`notes/evidence.md` C8）。
- **guard 与 approval 源码机制**：guard = 两个纯 listener 插件（timeout-policy + repeat-tool-reminder，图谱无服务声明谜底）；approval = scope-filtered waterfall + 三层 fail-closed + 审计对 + 持久策略事件（`notes/evidence.md` C13-C14）。
- **session log 内部结构**：append/deriveMessages/fork 全部源码核验（surface 为派生唯一来源、增量缓存、持久化在 seam 不在 core）（`notes/evidence.md` C15）。
- **ACP 与 hooks 对外投影**：ACP server（agents 依赖 + session/event 输出 + approval answerer）；hooks-claude-code（CC 事件 → dsh 扩展点 6 映射 + `ctx.shell` 执行 + 审计对）（`notes/evidence.md` C16）。
- **system-prompt 组装**：四类注册面 + order 语义 + complete 节唯一性 + tool schemas 规范排序 + assemble waterfall（`notes/evidence.md` C17）。
- **agent preset 交互边界**：`PresetTree extends Include` 挂载于 agent scope、两道安全 guard（inactive rows + root realm 泄漏拒绝）、isolate realm 产品约束落地、与 boot profile 的边界（`notes/evidence.md` C18）。
- **compaction 策略**：双触发（pressure / context-overflow）、定价与 model-free prune 先行、KV cache 复用摘要、surface replace 保 model-visible ⟺ logged（`notes/evidence.md` C19）。
- **版本谱系与差异表骨架**：0.0.1-rc.1~rc.5 → 0.1.0-rc.1~rc.7 全谱系 + 规模 + 机制变更采样 + 命名演化（`notes/source.md` 五，`Status: Observed`）。
- 版本锚点与"必须按版本研究"的判断；图谱可作为版本差异快速对照物。

### 仍然模糊

- `packages/boot/app-boot` 的 dsh 侧组合实现（profile 层序 → patch list）。
- Code Mode transport（`ctx.tools` note 提及）。
- session-persistence jsonl/sqlite 后端差异。
- 凭据引用（credential-reference）解析与暴露边界。
- hooks 与 SDK 两套协议面关系；`hooks-codex` dialect 差异。
- 逐版本机制级变更穷举（差异表骨架已建，深度待续）。

### 当前优先补证点

核心机制 + 组合层 + compaction 已全部源码级闭合（C1-C19）。剩余项按价值排序：**(a) 文档整合收尾**（版本化正文已整合、overview 已升级——检查 README 与专题索引同步）；**(b) 其余细节**（boot/app-boot、jsonl/sqlite、凭据、Code Mode、协议面）；**(c) 阶段四横向对比**（把 dsh 放回 agentic 主线）。

### 下一轮推荐研究任务

1. 文档整合收尾：README 机制专题索引与版本化正文引用核对。
2. 细节项按需：`packages/boot/app-boot`、session-persistence jsonl/sqlite 差异、凭据解析。
3. 阶段四：与 OpenClaw / OpenHands 横向对比（进入 `05-comparisons/`）。

---

## 8. 与现有文档的关系

- `overview.md`：回答"它是什么、为什么值得研究、版本关系与研究边界"。
- `roadmap.md`：回答"按什么顺序研究"（阶段划分），本文件回答"下一个问题是什么"。
- `plugin-system.md` / `agent-loop.md` / `capability-seam.md` / `guard.md` / `approval.md`：**机制专题正文（已提炼）**——插件系统、turn/step 流程、seam 三角色、执行卫生、决策闸门的压实结论；本文件继续负责未闭环问题与后续推进。
- `dsh-0.1.0-rc7.md`：版本化对象正文骨架，承接 `0.1.0-rc.7` 锚点下的压实结论；本文件负责未闭环问题的递进。
- `notes/source.md`：官方资料建档摘录（第一批三篇已完成，`Status: Verified`）；本文件的"下一步研究动作"优先指向剩余建档与源码核验。
- `notes/evidence.md`：Claim-Source 对照层；每轮核验结论（带源码锚点）先沉淀于此，压实后再提炼进版本化正文与机制专题。
- 本文件：负责把分散问题重新组织成**主线驱动的研究递进结构**；压实一条主线后，提炼为机制专题正文（如 `plugin-system.md`、`capability-seam.md`），并同步收窄本文件对应节点。
