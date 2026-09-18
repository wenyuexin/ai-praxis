# DeepSeek Harness 系列总览

> 目录级总览：承接 dsh 的定位、版本关系与研究边界。具体版本的对象研究在带版本标识的正文（如 [`dsh-0.1.0-rc7.md`](./dsh-0.1.0-rc7.md)）中承接，本文件不沉淀单个版本的机制细节。

## 定位

DeepSeek Harness（`dsh`）是 DeepSeek AI 开源的 agent harness（MIT 许可），核心设计主张是 **everything is a plugin**：系统由可组合插件构成，运行时基于 vendored Cordis（其设计描述见上游论文 [_A Programming Paradigm for Spatiotemporal Composability_](https://github.com/cordiverse/paper)）。`dsh` 提供 Web UI（默认 `http://127.0.0.1:3080`），可通过 `npx @deepseek-ai/dsh web` 或源码 `pnpm dsh web` 启动。

名字中的 "harness" 指 agent 运行时/外壳，不是 evaluation harness（`Status: Observed`，来自本地 checkout README 自述）。

## 核心设计主张（源码核验版，`Status: Observed`，证据见 `notes/evidence.md`）

以下主张均已源码核验（锚点版本 `0.1.0-rc.7`），机制级细节见各机制专题：

- **everything-is-a-plugin**：插件三形态经 `ctx.plugin()` 归一化，Fiber 生命周期状态机 + inject epoch 门控，注册即 effect、ctx 解析走 Proxy + isolate 边界（C1-C5）。
- **配置驱动组合**：boot 级 profile/bundle/patch 层序（loader entry → `ctx.plugin()` 桥接链，C9）+ per-session agent preset 挂载（`PresetTree extends Include` + 两道安全 guard，C18）。
- **capability seam**：Service Definition / Service Provider / Consumer 三角色齐备才构成，`resolve(request): Spec` 显式默认化（C6/C7/C12）。
- **session 持久化与 compaction**：session log 为 append-only 事件流 + surface 派生历史（model-visible ⟺ logged）；持久化在 seam 不在 core（C15）；compaction 双触发（post-step pressure / context-overflow recovery）+ model-free prune 先行 + KV cache 复用摘要 + surface replace 保可重放（C19）。
- **ACP / hooks / SDK**：ACP server 只输出已提交消息、hooks 把 CC/Codex 事件映射到 dsh 扩展点（C16）。
- **self-modification**：`self-modification/` 组已于 2026-08-11 改名 `extensions/`（根 AGENTS.md 未同步，`Deprecated`）；`tool-cordis` 即其工具面（agent 经 inspect/define/run/stop/undefine 检查与挂载插件）（C10）。

早期版本（骨架期）的"上游自述索引"口径已被本版取代；上游自述仅作为来源登记保留在 `notes/source.md`。

## 版本关系与研究边界

dsh 处于 **developer preview**，迭代快且**存在兼容性破坏变更**（上游 README 明确提示）。因此：

- 本目录不写单一静态总览；`overview.md` 只承接系列总览与版本关系。
- 具体版本对象研究进入带版本标识的正文，如 `dsh-0.1.0-rc7.md`。
- 已记录版本锚点：`0.1.0-rc.7` / commit `99f6f02fecdb7dff40c3fbc9470f5907c29f74ca` / 2026-08-17（本地 checkout 当前状态）。
- 后续版本研究应优先确认与上一版本的能力边界差异，避免把新版本观察写进旧版本正文。

## 上游文档入口

以下为上游仓库 `docs/` 下的官方文档，作为研究来源使用（`Status: Verified`，未逐篇核验内容）：

- [`docs/architecture.md`](https://github.com/deepseek-ai/deepseek-harness/blob/main/docs/architecture.md) — 架构总览
- [`docs/capability-seams.md`](https://github.com/deepseek-ai/deepseek-harness/blob/main/docs/capability-seams.md) — capability seam 模型
- [`docs/cordis-primer.md`](https://github.com/deepseek-ai/deepseek-harness/blob/main/docs/cordis-primer.md) — Cordis 基础
- `docs/agent-lifecycle.md`、`docs/tool-execution-pipeline.md`、`docs/persistence-catalog.md`、`docs/config-catalog.md`、`docs/glossary.md`、`docs/api-gateway.md`

## 与子领域文档的关系

- dsh 的插件 / skill / 能力注册机制证据，在 `agentic/04-skill-and-tool-systems/` 主线归纳需要时摘要回填。
- agent loop、tool-use、sandbox 等机制证据，按需回填 `agentic/02-single-agent/`、`agentic/05-environments/` 对应专题。
- 与 OpenClaw、OpenHands 等同类 agent 运行时的横向比较，进入 `agentic/06-frameworks-and-tools/05-comparisons/`。

## Evidence

- **Status**: 本文件各段均已标注；「核心设计主张」为源码核验版（`Observed`，C1-C18）；版本关系为 `Observed`（`notes/source.md` 五）。
- **Sources**: 本地 checkout `/Users/wenyuexin/github/deepseek-harness`（`vendor/` + `packages/` + `.agents/notes/`）；上游 `https://github.com/deepseek-ai/deepseek-harness`；Cordis paper 仓库 `https://github.com/cordiverse/paper`。
- **Trace**: 2026-08-21 由 ai-praxis 侧建立对象目录时创建；2026-08-21 设计主张升级为源码核验版；版本锚点 `0.1.0-rc.7`（`99f6f02fecdb`）。上游仓库本身在快速演进，本地 checkout 与上游 main 可能存在差异。
- **Needs**: 版本切换后按 `notes/source.md` 5.4 复查本文件受影响主张；compaction 策略细节核验后更新对应条目（§3.2）。

## 研究推进入口

- 按什么顺序研究（阶段划分与产出）：见 [`roadmap.md`](./roadmap.md)。
- 下一个问题是什么（机制级问题树与每轮研究动作）：见 `notes/deep-research-question-tree.md`。
