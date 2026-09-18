# DeepSeek Harness 研究路线图

> 整体研究计划：阶段划分、每阶段产出与进入下一阶段的判断。问题级研究推进见 `notes/deep-research-question-tree.md`，两文件不重复维护同一信息：本文件只在阶段切换时更新，问题树随每轮核验持续更新。

## 阶段总览

### 阶段一：官方资料建档

- 目的：建立官方口径下的完整认知，让后续源码核验有可追溯的对照基线。
- 动作：逐篇通读并摘录上游 `docs/` 下官方文档，优先顺序为 `architecture.md` → `capability-seams.md` → `cordis-primer.md`，随后 `agent-lifecycle.md`、`tool-execution-pipeline.md`、`persistence-catalog.md`、`config-catalog.md`、`glossary.md`、`api-gateway.md`。
- 产出：`notes/source.md`（逐篇摘录 + 来源路径 + 待核验标记）。
- 完成标准：overview.md 中"核心设计主张"的每一条主张都能在 `notes/source.md` 找到逐条来源；阶段二可随时引用而不必回查上游。

### 阶段二：机制核验（问题树驱动）

- 目的：按机制链逐层完成源码核验，把"上游自述"变成"源码可证实的机制结论"。
- 顺序：按问题树的优先级走——插件层 → 组合层 → 生命周期层 → 能力边界层 → 接口层；演进层（版本差异）贯穿全程。
- 产出：`notes/evidence.md`（Claim-Source 对照、核验状态）；每压实一条机制主线，提炼一篇机制专题正文（如 `plugin-system.md`、`capability-seam.md`、`preset.md`、`session-and-compaction.md`、`self-modification.md`、`acp.md`）。
- 完成标准：问题树"待确认"项逐轮闭合；机制专题正文可从脚手架提炼，不再依赖"待确认/下一步"组织内容（见 `research-artifacts.md` §3.6 回流判断）。

### 阶段三：版本演进跟踪

- 目的：developer preview 期的破坏性变更不污染既有结论。
- 动作：每次研究前确认本地 checkout 版本锚点；通过 release notes / git history 梳理版本差异，判断旧结论是否失效。
- 产出：版本差异记录；新版本锚点出现时建立对应版本化正文（如 `dsh-0.1.0-rc7.md` 之后的新版本文件），并把 `Deprecated` 结论从旧正文迁出。
- 完成标准：任一版本正文的结论都带版本锚点；版本切换时能快速列出受影响结论。

### 阶段四：横向对比与回填

- 目的：把对象研究放回 agentic 主线，避免案例研究孤立。
- 动作：与 OpenClaw、OpenHands 等同类 agent 运行时对比，进入 `05-comparisons/`；插件 / skill / 能力注册机制证据摘要回填 `04-skill-and-tool-systems/`；agent loop、tool-use、sandbox 等机制按需回填 `02-single-agent/`、`05-environments/`。
- 产出：对比文档 + 主线回填引用。
- 完成标准：本目录结论可被主线文档摘要引用，主线不依赖本目录全文。

## 当前阶段

- 阶段一进行中（2026-08-21）：第一批三篇优先文档（architecture / capability-seams / cordis-primer）已建档至 `notes/source.md`（`Status: Verified`）；剩余 6 篇（agent-lifecycle / tool-execution-pipeline / persistence-catalog / config-catalog / glossary / api-gateway）未建档。
- 阶段二进行中：**第一轮（插件层源码核验）已完成**——Fiber 生命周期/inject 门控/ctx 解析/事件分发/配置校验 5 条机制结论 + dsh 侧 Service Definition / Consumer 形态 2 条 + vendored 本地修改语义 1 条，全部带源码锚点（`notes/evidence.md` C1-C8，`Status: Observed`）。
- **第二轮（组合层 + self-modification 悬案）已完成**——loader entry → `ctx.plugin()` 完整桥接链（C9）；`self-modification/` 组改名 `extensions/` 的悬案解决（2026-08-11 改名台账，C10）。
- **第三轮（生命周期层 + capability seam 样板）已完成**——`ReactLoopAgent` 完整 turn/step 流程与官方序图对应（含 `agent/request-error`、`request/header`/`request/context` 两处文档未覆盖细化，C11）；`ctx.shell` 三角色完整代码范式（含 `resolve(request): Spec` 显式默认化，C12）。
- **核心机制主线已闭合**：插件层 → 组合层 → 生命周期层 → capability seam 四层均达源码级（`notes/evidence.md` C1-C12）。剩余深度项（guard/approval、session log 内部、boot/app-boot、preset isolate）与广度项（版本差异表、剩余建档）见问题树 §7。
- **阶段三启动：版本差异表骨架完成**（`notes/source.md` 五）——0.0.1-rc.1~rc.5 → 0.1.0-rc.1~rc.7 全谱系（规模/日期/哈希）、rc.6→rc.7 机制变更采样（PTC Mode、ACP 图片桥接、agent-presets subagent）、命名演化两段改名（cordis/→self-modification/→extensions/）、对 C1-C12 的影响评估（高风险：C9/C10/C11）。逐版本穷举待续。
- **第四轮（安全治理）已完成**——guard 组谜底（两个纯 listener 插件：timeout-policy + repeat-tool-reminder，C13）；approval 决策链（scope-filtered waterfall + 三层 fail-closed + 审计对 + 持久策略事件，C14）。**§4.2 安全治理闭合**。
- **正文提炼（第一批）已完成**——按 research-artifacts §3.6 回流纪律，已把压实主线提炼为机制专题正文：`plugin-system.md`（插件层+组合层，C1-C5/C8/C9）、`agent-loop.md`（生命周期层 turn/step，C11）、`capability-seam.md`（seam 三角色，C6/C7/C12）、`guard.md`（执行卫生，C13）、`approval.md`（决策闸门，C14）。**五篇机制专题覆盖当前全部压实主线**；脚手架（问题树/evidence）继续保留未闭环问题。
- **第五轮（session log 内部结构）已完成**——append（不可变事件/seq/JSON 快照/surface 校验）、deriveMessages（surface 为派生唯一来源 + 增量缓存 + compaction replace 联动）、fork（boundary 校验 + open-turn 约束 + 前缀复制）、持久化在 seam 不在 core（C15）。**§3.1 生命周期层全部闭合**。
- **第六轮（接口层）已完成**——ACP server（agents 依赖 + session/event 只输出已提交消息 + approval 一次性 answerer）；hooks-claude-code（CC 事件 → dsh 扩展点 6 映射 + `ctx.shell` 执行 + 审计对 + 已知限制清单）（C16）。
- **第七轮（system-prompt 组装）已完成**——四类注册面（section/context/tools/variable）+ order 语义（persona=0、complete 唯一节）+ tool schemas 规范排序 + assemble waterfall（C17）。**核心机制主线全部源码级闭合（C1-C17）**：插件 → 组合 → 生命周期 → seam → 安全 → 接口。
- **第八轮（agent preset 交互边界）已完成**——`PresetTree extends Include` 挂载机制、两道安全 guard（inactive rows + root realm 泄漏拒绝）、isolate realm 产品约束落地、与 boot profile 边界（C18）。**组合层全部闭合（C1-C18）**。
- **第九轮（compaction 策略 + 文档整合）已完成**——compaction 双触发/定价/prune 先行/KV cache 复用/surface replace（C19）；`dsh-0.1.0-rc7.md` 从骨架整合为版本化正文（架构整合视图 + 机制专题索引 + 版本特有信息）；overview 设计主张升级为源码核验版。**核心机制主线全部源码级闭合（C1-C19）**。剩余项：boot/app-boot、jsonl/sqlite、凭据、Code Mode、协议面等细节；阶段四横向对比。
- 建档期间产出：capability 服务图谱（60+ 服务）可作为版本差异快速对照物。

## 前置依赖与注意事项

- 本地 checkout 版本锚点：`0.1.0-rc.7`（commit `99f6f02fecdb`，2026-08-17）；上游 main 快速演进，所有研究结论必须带版本。
- 阶段二开始前至少完成 `architecture.md`、`capability-seams.md`、`cordis-primer.md` 三篇建档——它们分别是插件系统、capability seam、Cordis 基底的官方口径。
- 每轮研究结束时只更新问题树的"下一步研究动作"，不更新本文件；本文件只在阶段完成或阶段内顺序调整时修改。
