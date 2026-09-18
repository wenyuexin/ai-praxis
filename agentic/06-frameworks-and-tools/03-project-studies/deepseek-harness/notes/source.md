# DeepSeek Harness 官方资料建档摘录

> 阶段一建档产物（roadmap 阶段一，第一批三篇）。版本锚点：`0.1.0-rc.7`（commit `99f6f02fecdb`，2026-08-17）。本文件承接上游 `docs/` 官方文档的逐篇摘录，供问题树与版本化正文引用；所有内容均可在对应上游路径复核。
>
> 状态约定：`已建档` = 摘录自官方文档原文，可作为上游口径引用；`待源码核验` = 官方口径存在，但机制级实现尚未在源码中确认。

## 一、docs/architecture.md

来源：`/Users/wenyuexin/github/deepseek-harness/docs/architecture.md`（129 行，已建档）

### 1.1 Cordis 与"无特权内核"主张

- dsh 之下是 Cordis：插件向共享 context 贡献 services、typed events、可逆 effects。
- **产品每一部分都是插件**：model adapter、tool registry、session log、agent loop 本身都是插件，因此每一部分都可从配置替换。
- **没有可打补丁的特权内核**：扩展 dsh = 在旁边挂载插件；注册是 effect，插件卸载时自动回卷。
- 关联：§1.1 待确认"插件加载/组合/生命周期机制"的官方口径起点；组合机制在 `packages/boot/app-boot/README.md#profiles`。

### 1.2 Profile 与 bundle（组合模型，直接回答 §2.1 待确认）

- 运行中的 dsh = **启动时按有序层组成的插件树**（plugin tree composed at boot from ordered layers）。
- **profile** = Harness home 中的命名组合：列出它堆叠的 bundles、持有安装的 out-of-tree 插件、保留用户自己的 `cordis.patch.yml`。`web` 和 `headless` 随附为模板。
- **bundle** = Cordis 配置行 + 其挂载代码的分发格式；bundle 插入的任何内容都可被上层继续 patch。
- 声明方式：各自 `package.json` 的 `dsh` 字段——`dsh.profile` 列出 profile 的 bundles；`dsh.bundle` 指向 bundle 的 patch 文件。
- 内置 bundle：`dsh-base`（每个 profile 的第一层：model adapters、tools、persistence、sandbox + approval policy、settings、credentials、telemetry）；`dsh-web-app`（浏览器应用）；`dsh-headless`（无 server 的一次性 runner）。
- **层的应用顺序**：profile 列出的每个 bundle（按序）→ profile 的 `cordis.patch.yml` → home 级 patch → 任何 `--patch` overlay。patch 按 id 定位一行并整体替换其 config，或插入新行。
- 查看本机实际启动的树：`dsh --profile web --dump-config`。
- 回答：§1.2 待确认"bundle 是 preset 的补丁层还是独立组合单元"——**bundle 是组合单元（配置行+代码），profile 是组合（命名堆叠）；patch 层是覆盖机制**。§2.1 待确认"preset 决定/用户如何覆盖"——profile 层序 + patch 覆盖。
- 关联：`ctx.agentPresets`（见第三节图谱）："discovers preset directories over trusted and user-authored roots and mounts one preset cordis.yml under an agent scope during creation, rejecting a row that never activates or that publishes into the root service realm"——**agent preset 是 per-session 的 cordis.yml 挂载，与 boot 级 profile 分层**。

### 1.3 核心包与 ctx key（部分回答 §3.1）

| 包（packages/ 内路径） | 职责 | ctx key |
|---|---|---|
| `core/session` | append-only `SessionEvent` 日志 + 内存 store | `ctx.sessions` |
| `core/system-prompt` | prompt 段与 tool schema 组装 | `ctx.systemPrompt` |
| `core/tools` | 作用域 tool registry + guarded 执行管线 | `ctx.tools` |
| `core/agent` | `Agent` 接口、live registry、`agent/*` 事件 | `ctx.agents` |
| `core/agent-loop` | 实现该接口的默认 driver | `ctx.agentLoop` |
| `core/scope` | per-agent 作用域注册原语 | 库，无 key |
| `llm/llm` | 消息/流词汇 + adapter seam | `ctx.llm` |

- 注意包路径语义：`packages/<group>/<pkg>`（如 `packages/core/session`、`packages/shell/bash-local`）；根 AGENTS.md 布局把 group 与 package 名混列，以实际目录与本文档路径为准。

### 1.4 事件三分域（部分回答 §3.1）

- **Session events**：持久事实，追加进日志并通过 `session/event` 广播；需要跨 reload 存活的事实用此类。
- **Agent events**（`agent/*`）：携带 live `Agent`（inbox、step、status、request、validation、continuation）；用于观察/拦截进行中的工作。
- **Capability events**：给 seam 挂策略与适配器（`fs/*`、`tools/*`、`telemetry/*`），不 import loop。
- 完整事件表：`docs/event-producer-consumer.md`。

### 1.5 Turn / step 流程（直接回答 §3.1 待确认的"agent-loop 迭代结构"）

- **step** = 一次 model request + 它调用的 tools；**turn** = 零或多个 step。
- 流程（官方序图，原文代码块）：`turn/start` → claim 输入 → 组装 prompt 段 + tool schema → `agent/pre-step`（reject | enter）→ `step/start` → 追加 entered messages → 从日志派生 model history → `agent/request` → `llm/stream` → `assistant/chunk*` → `assistant/message` → `tool/call*` → `tools/pre-execute` → `tools/execute` → `tools/post-execute` → `tool/result*` → `step/end` → `agent/turn-stopping` → `turn/end`。
- 持久 session 事件：`turn/*`、`step/*`、`user/message`、`assistant/*`、`tool/*`；其余是跨三域的 live 扩展点。
- **Waterfall 事件**（listener 必须 `next()` 委托）：`agent/pre-step`、`agent/request`、`llm/stream`、三个 `tools/*`；`agent/turn-stopping` 是 serial，无 `next()`。
- 输入经一个 inbox 到达 driver；injected context 在 inbox 等待直到其他消息唤醒。
- `agent/pre-step` 决定模型看到什么：可改写或 reject；rejected/empty 的首 claim 仍会关闭一个未花 step 的持久 turn（日志记录尝试）。
- 关联：§3.1 待确认"waterfall 语义在代码中的实现"——官方口径已明（见 cordis-primer §二.3），源码实现待核验。

### 1.6 Session log（回答 §3.1/§3.2 的日志相关待确认）

- session log 是模型所见上下文的来源：`deriveMessages()` 从日志投影 model history；原始 `assistant/chunk` 事件保留 replay 与 UI 保真度。
- Fork、resume、transcripts、telemetry、persistence 都从这条流派生。
- **Model-visible ⟺ logged**：到达 model request 的任何内容必须能从日志重建，且有 runtime invariant 断言；新 model-visible 输入必须新增 session event（扩展 `SessionEventMap` 并从日志渲染）。

### 1.7 Capability seam（回答 §2.2 官方口径）

- **seam** = 可替换能力，三角色：**Service Definition**（声明接口）、**Service Provider**（实现）、**Consumer**（使用方，通常是 model-facing tool）。包可合并角色，但**单一角色不构成 seam**；新增能力 = 设计全部三角色（图谱见 `capability-seams.md`）。
- seam 的意义：一个 provider 替换改变整个产品。Filesystem 与 subprocess provider 共享同一执行世界——把它们指向远端 sandbox 时，Bash、PTY、LSP 一起迁移，无 provider fork。Subagent providers 在单一接口后变化同样大（从全新 child agent 到另一产品内的 delegated turn）。
- 回答：§2.2 待确认"三角色在代码中的接口与注册点"——官方口径已明，代码级注册点待核验（图谱给出 owner/impl/consumer 包级映射）。

### 1.8 新行为去哪（扩展点速查，回答 §5 接口层部分待确认）

- 加 model provider → `ctx.llm` 注册 adapter；加 model-facing capability → `ctx.tools`；会话差异化能力集 → compose agent preset（service row 需要 `isolate` realm）；shell → `ctx.shell` backend（local 经 `ctx.subprocess` 派生）；持久终端 → `ctx.terminals` + `dsh-tool-terminal`；人类命令 → `ctx.commands`（不经 model turn 分发）；后台任务 → `ctx.jobs`；fs → `ctx.fs` provider 或 `fs/*` 事件；进程限制 → `ctx.sandbox` backend（consumer 在 spawn 前 wrap argv）；拦截 → `agent/*` / `tools/*` 事件；注入模型上下文 → `agent.inject()`；UI/编辑器集成 → `ctx.agents` + 从 `session/event` 渲染；持久会话状态 → 扩展 `SessionEventMap`；会话标题 → 唯一 `ctx.sessionTitle` provider；同会话目标 → `ctx.goals`；fork 会话 → `ctx.sessions.fork(source, boundary?, childSessionId?)`；作用域注册 → 该 agent 的 `agent.ctx`。

## 二、docs/cordis-primer.md

来源：`/Users/wenyuexin/github/deepseek-harness/docs/cordis-primer.md`（45 行，已建档）

### 2.1 Cordis 五观念（回答 §1.1/§1.2 官方口径）

1. **插件 = 实现 Service 的对象**：可以是带可选 `inject` 与 `apply(ctx)` 字段的函数，或 Cordis 把生命周期挂进当前 context 的 `Service` 子类。
2. **context = 服务的仓库**：服务声明稳定的 `ctx.<key>`（`ctx.tools`、`ctx.llm`、`ctx.sessions`）；其他插件按 key 找服务，不 import 具体实现。
3. **用 `inject` 声明服务依赖**：命名所需服务的插件会等到服务存在后才运行——加载顺序通过服务需求表达，而非手动启动顺序。
4. **Typed Events 通信**：服务用 TS 声明合并声明事件名，按 `emit` / `waterfall` / `parallel` / `serial` 分发。
5. **注册是可逆 effect**：prompt 段、tool schema、adapter、provider、listener 都经 `ctx.effect()` / `ctx.on()` 安装，reload 与 teardown 可预测回卷。

### 2.2 分发模式表（回答 §1.1"事件语义"待确认）

| 模式 | 等待? | 分发顺序 | 有返回值? |
|---|---|---|---|
| `emit` | 否 | 按注册序观察 | 否 |
| `waterfall` | 否 | 按注册序观察 | 是 |
| `parallel` | 是 | 全部并行观察 | 否 |
| `serial` | 是 | 按注册序观察 | 是 |

- 分发模式是事件公共契约的一部分；新 harness 事件用 `@mode` tag 记录，生成的 catalog 可校验声明与分发点。

### 2.3 Waterfall 语义（回答 §3.1/§1.1 waterfall 待确认）

- `ctx.waterfall` 是 around-middleware：listener 收到 `(...args, next)`；调 `next()` 委托（可能被 wrap 的）结果给下一个服务；不调 `next()` 直接 return = short-circuit；值经 `next()` 返回值传播。
- 协作式 listener 通常改写共享 request/decision 对象后委托；也可整体替换结果，下游只看到替换后结果。`prepend: true` 仅用于必须早于普通注册执行时。
- 单决策事件中 short-circuit 是设计意图：policy listener 拥有决策时可不调 `next()`；只注释/观察的 listener 必须委托。

### 2.4 Loader 配置（回答 §1.2 的 cordis.yml 语义）

- `@deepseek-ai/cordis-plugin-include` 把 `!!js` 解析为表达式节点。
- Loader 在注入激活后（针对该插件 context——`ctx.serviceName`）插值 entry 的 `config`；在每次挂载决策时（针对 loader context）插值 `disabled` 字段；Include 保留嵌套行表达式直到目标激活。
- **其他 entry 元数据保持字面量**；环境选择插件时用 overlays。

### 2.5 实用规则

- 行为封装进插件：tool pipeline 事件归 `ctx.tools`、模型流归 `ctx.llm`、live agent 协调归 `ctx.agents`；拦截/策略用事件，直接能力调用用 service methods。
- 每个注册都要有 disposer（`ctx.effect()` 返回或 Cordis helper）；teardown 顺序重要时，把相关工作放同一 effect 内按序回卷。

## 三、docs/capability-seams.md（生成图谱）

来源：`/Users/wenyuexin/github/deepseek-harness/docs/capability-seams.md`（472 行，已建档）。**由 `scripts/gen-doc-graphs.ts` 生成（勿手改）**：services 从 Cordis 声明发现，接口/实现/消费三角色由脚本分类并带 completeness guard——即官方"权威包级图谱"，版本锚点内可作 Verified 来源。

### 3.1 图谱形态

- Mermaid 图：每个服务 = `ctx.<key>` + 职责一句话；边 = 声明包 / 实现包 / 直接消费包。
- 表：每服务一行——ctx key、角色（`seam` / `core` / `bundle`）、owner、implementations、direct consumers、companion plugins、note。

### 3.2 关键服务（按问题树分层摘录）

**核心主干（core）**：`ctx.sessions`（append-only Session 实例 + 持久 session event feed）、`ctx.agents`（live Agent handles、create/resume factory seam、process-local initiator 传播）、`ctx.agentLoop`（**唯一具体 loop 插件**，bundle 角色；扩展包依赖 dsh-agent 事件与服务，不依赖本包）、`ctx.systemPrompt`、`ctx.tools`（注册能力、Code Mode transport、pre-policy → monotonic guards → around dispatch → post-policy → final observation 路由）、`ctx.agentPresets`（见 1.2）、`ctx.goals`、`ctx.commands`、`ctx.invariants`（package-owned invariant registry）、`ctx.typert`（runtime type registry）+ `ctx.typertGateway`（Typert Host invocation gateway，经 Connection RPC carrier 暴露 unary 调用）、`ctx.sandboxPolicy`（deployment 默认 mode + workspace root 的唯一归属；bash 与 fs 读同一服务避免不同 root）、`ctx.permissionPresets`（`workspace-write` / `danger-full-access` 预设表，一次写一个 `permission/preset` 事件打通两个 knob 事件）、`ctx.planMode`、`ctx.sessionProjections` / `ctx.sessionProjectionCache`、`ctx.workspaceRegistry`、`ctx.messageFeedback`、`ctx.apiProxy`、`ctx.webServer`、`ctx.clientModules`、`ctx.dynamicCordisRunner` + `ctx.cordisInspect`（见 3.3）。

**能力 seam**（三角色齐备）：`ctx.llm`（impl: llm-deepseek / llm-pi-ai / llm-replay）、`ctx.fs`（fs-local / fs-sandbox / fs-e2b）、`ctx.shell`（bash-local / bash-sandbox / pwsh-local）、`ctx.subprocess`（subprocess-local / subprocess-e2b）、`ctx.subagents`（spawn/fork-in-process、acp、codex、claude-code、dsh-sdk 六实现）、`ctx.sandbox`（sandbox-local）、`ctx.approval`（**一次性权限决策，经 `approval/request` waterfall 分发；answerer 是 listener（ACP bridge 为自己的 agents）；无应答 fail-closed 到 `unavailable`**）、`ctx.settings`（settings-file）、`ctx.credentials`（credentials-local；按操作解析，轮换的凭据下一请求即生效）、`ctx.sessionPersistence`（jsonl / sqlite 后端，同一 SessionEvent 词汇）、`ctx.sessionQuery`、`ctx.sessionTitle`、`ctx.sessionTelemetry`、`ctx.storage`、`ctx.skills`（skill-badge / skill-filesystem）、`ctx.userQuestions`、`ctx.compaction`（impl: compaction-basic，**消费 post-step pressure 与 request-error recovery 事件；无 model-facing compact tool**）、`ctx.tokenMeter`、`ctx.toolResultPruner`（compaction 前重写超大 tool result）、`ctx.web`（search/fetch 四 provider）、`ctx.spillStore`、`ctx.directoryPicker`、`ctx.workflowEngine`、`ctx.lsp`、`ctx.codeRuntime`、`ctx.attachments`、`ctx.jobs`、`ctx.terminals`、`ctx.shellEnv`。

### 3.3 self-modification 的 rc.7 实况（关键发现）

- **图谱中不存在 `self-modification` 包或服务**——与 §4.1 的 `Conflicting` 记录一致：rc.7 中该能力没有落地为独立包。
- **最接近的等价机制**：`tool-cordis` + `cordis-host-runner`（`packages/extensions/` 下）：
  - `ctx.dynamicCordisRunner`（core，owner: `cordis-host-runner`，consumer: `tool-cordis`）：in-memory definition registry、host half 的 vm sandbox、request-run round trip；浏览器页经 wire 的 remote namespace 访问同一服务。
  - `ctx.cordisInspect`（core，owner: `cordis-host-runner`，consumer: `tool-cordis`）：注册 host inspect providers、镜像 client provider manifest、经 dynamic Cordis transport 路由 client 查询。
- 推断（`Status: Inferred`）："agent 检查/挂载自己的插件"在 rc.7 由 tool-cordis（模型可用工具）经 cordis-host-runner 动态加载/inspect Cordis 包实现；与根 AGENTS.md 所述 `self-modification/` 包的关系（更名？未落地？）待 git history / release notes 核验。

### 3.4 图谱对问题树的其它回答

- §2.2：三角色的**包级映射**已完备（每服务 owner/impl/consumer 列全）；代码级接口与注册点仍待核验。
- §3.2：compaction 触发 = post-step pressure + request-error recovery 事件（官方口径）；策略细节待源码。
- §4.2：approval = waterfall 单决策 + fail-closed；permission presets = 两个 knob（sandbox-mode + approval-policy）的捆绑表。
- §5.1：Typert 角色明确（type registry + gateway，经 Connection RPC）；`acp` 包消费 `ctx.agents`（ACP server 接 live Agent）；hooks（hooks-claude-code / hooks-codex）消费 `ctx.sessionPersistence` 与 `ctx.shell`（桥接会话持久化与 shell 执行）；SDK（JSON-RPC）不在图谱（协议面）。

## 四、建档结论汇总

### 已闭合的问题树待确认（官方口径层，源码级仍待核验）

- §1.1 插件是什么对象：实现 Service 的对象（函数 inject/apply 或 Service 子类）；服务按 `ctx.<key>` 发现，不 import。
- §1.1 加载顺序：`inject` 声明依赖 → 服务存在后运行；非手动排序。
- §1.2 bundle vs preset：bundle = 配置行+代码的分发单元；profile = 命名堆叠；patch 层按序覆盖（bundle 序 → profile patch → home patch → `--patch`）。
- §2.1 preset：boot 级 profile（每会话可再经 agent preset cordis.yml 挂载于 agent scope）。
- §2.2 seam 三角色：官方定义 + 包级图谱（owner/impl/consumer）。
- §3.1 turn/step 流程、事件三分域、waterfall 清单、Model-visible ⟺ logged。
- §3.2 compaction：post-step pressure + request-error recovery 触发；无 compact tool。
- §4.2 approval/permission/sandbox 官方口径。
- §5.1 Typert 角色；ACP 消费 `ctx.agents`；hooks 消费 `sessionPersistence` + `shell`。

### 新产生的待核验点（转问题树）

- self-modification 的 rc.7 等价机制 = tool-cordis / cordis-host-runner？（源码 + release notes 核验）
- `guard` 包在图谱中无服务声明 → 其 loop-hygiene / tool-timeout 机制如何实现（纯插件？）。
- agent preset 的"reject a row that never activates / publishes into root service realm"具体行为。
- Code Mode transport（`ctx.tools` note 提及）是什么。
- approval 的 fail-closed `unavailable` 在 ACP bridge 之外的默认行为。
- session-persistence jsonl/sqlite 后端差异。
- `isolate` realm（session 差异化能力集的 service row）语义。

## 五、版本谱系与差异表（阶段三准备）

> 数据来源：`git log` release commits（main 分支）；tag 仅 `dsh-v0.1.0-rc.7`（早期版本无 tag）。版本锚点研究以 release commit 哈希为准。
> 规模 = 与上一 release commit 的 diff --shortstat。机制相关变更 = 该范围 commit message 采样，非穷举。

### 5.1 版本谱系

| 版本 | 日期 | 规模 | 特征 |
|---|---|---|---|
| 0.0.1-rc.1 | 2026-08-11 | 首发 | `b64c3ac1ba` |
| 0.0.1-rc.2 | 2026-08-11 | 小 | `5ca7be5dcb` |
| 0.0.1-rc.3 | 2026-08-13 | 中 | `1e99f20963` |
| 0.0.1-rc.4 | 2026-08-13 | 小 | `a90d9af1b2` |
| 0.0.1-rc.5 | 2026-08-13 | **757 文件，-5521/+2923** | 0.0.1 系列收尾重构（`3e8a1cfa33`） |
| 0.1.0-rc.1 | 2026-08-13 | **817 文件，+6674/-2963** | 0.1.0 里程碑：大扩展（`22ab3beac1`；含 session-query 全文搜索 opt-in、bundle 排除 product subagents） |
| 0.1.0-rc.2 | 2026-08-13 | 小 | `60b04b6ef7` |
| 0.1.0-rc.3 | 2026-08-13 | 228 文件 | `8a954b2eca`（rc.4 无独立 release commit，rc.3 直接到 rc.5） |
| 0.1.0-rc.5 | 2026-08-13 | 223 文件（小） | `abe560f81e` |
| 0.1.0-rc.6 | 2026-08-13 | 223 文件（224/224 符号性） | `15148dbd9a`；含 `fix: SessionPersistenceCorruptionError` |
| **0.1.0-rc.7** | 2026-08-17 | **538 文件，+8181/-1623** | **当前版本锚点**（`bb4ca698d6`）；见 5.2 |

### 5.2 rc.6 → rc.7 机制相关变更（采样）

- `feat(agent-presets)`: enable background Codex and Claude Code subagent tasks——agent-presets 能力扩展（直接影响 §2.1 preset 结论的新版本风险点）。
- `feat(acp)` / `fix(acp)`: bridge durable image prompts and replies；isolate prompt admission from agent work——ACP 桥接面变化（§5.1）。
- `refactor/ui` + `fix(ui-agent-preset)`: rename code preset to **PTC Mode**——preset 命名/概念变化（§2.1 术语风险点）。
- 未变（rc.6→rc.7 无 diff）：`vendor/README.md`、`packages/AGENTS.md`。

### 5.3 命名演化关键点（对照 C10）

- **2026-08-09** `2a40cbf8ef`：`refactor(packages): merge timeout/ into guard/, rename cordis/ to self-modification/`——第一段改名 + `timeout/` 并入 `guard/`（guard 与 timeout 的历史关联）。
- **2026-08-11** 改名台账（`.agents/notes/.../2026-08-11-repository-naming-contract-and-rename-ledger.md`）：`self-modification/` → `extensions/`——第二段改名；根 AGENTS.md 布局未同步（仍显示 `self-modification/`），以台账为准（C10 的 `Deprecated` 判定成立）。

### 5.4 对已闭合结论的影响评估

- C1-C12 全部锚定 `0.1.0-rc.7`（2026-08-17）。rc.6→rc.7 规模（538 文件）预示 rc.8 可能继续大改；**每次切换版本锚点后需重跑核验**。
- 高风险受影响结论：C9（loader 组合——PTC Mode 涉及 preset 概念）、C11（turn 流程——agent-presets/acp 相关）、C10（extensions 命名——需持续跟改名台账）。
- 快速对照手段：`docs/capability-seams.md` 生成图谱（`scripts/gen-doc-graphs.ts`）——版本切换后重跑/重读图谱即可发现服务级增删与角色变化（§6.1 方法补充的落地）。

### 5.5 待补（下一轮可选）

- 逐版本机制级变更的穷举梳理（0.0.1-rc.5 与 0.1.0-rc.1 两次大重构的内部结构变化）。
- 版本差异表与问题树 §6.1 的联动：新版本锚点出现时更新本表 + 检查 C1-C12 受影响项。
