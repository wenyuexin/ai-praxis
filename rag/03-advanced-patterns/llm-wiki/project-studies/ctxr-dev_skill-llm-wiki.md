---
skill: synthesize-repo
domain: github
theme: "knowledge_base"
purpose: "基于 GitHub 仓库信息生成结构化分析笔记"
sources:
  - "https://github.com/ctxr-dev/skill-llm-wiki"
tags:
  - github
  - "JavaScript"
  - repo-analysis
---


<!-- AI 分析引导（主题模板） -->
<!-- 主题: knowledge_base -->
<!-- "知识库 / 文档系统 / 个人 Wiki 类项目" -->
<!-- 对比维度: 架构分层, 摄入方式, 存储格式, 查询机制, 协作模式 -->
<!-- 分析要点:
  - 知识是如何被组织成层级或图谱的？
  - 支持哪些输入源（手动、剪藏、API、同步）？
  - 存储层是纯文本、数据库还是混合方案？
  - 查询能力是否支持语义搜索、全文检索、向量检索？
  - 是否支持多人协作、权限控制、版本历史？
-->
<!-- /AI 分析引导 -->
# ctxr-dev/skill-llm-wiki

> **一句话总结**：一个面向 AI 重度工作流的 LLM 知识库构建工具，以「Skill」形态嵌入 Claude Code，通过隔离私有 Git 提供版本化、可回滚的 Wiki 管理，并采用三级 AI 策略（TF-IDF → MiniLM → Claude）实现从自动化到智能决策的渐进式处理。

## 基本信息

- **仓库**：ctxr-dev/skill-llm-wiki
- **描述**：Build, extend, validate, repair, rebuild, or merge an LLM-optimized knowledge wiki from markdown notes, documentation, source code, or any text corpus.
- **Stars**：0 | **Forks**：0
- **主要语言**：JavaScript
- **许可证**：MIT License
- **链接**：https://github.com/ctxr-dev/skill-llm-wiki

## 项目定位

- **解决的问题**：AI 重度工作流中，随着与 LLM 交互的文档和知识不断积累，需要一种结构化、可版本化、可验证的方式来组织和维护这些知识资产。传统 Wiki 工具（如 Obsidian、Notion）虽然能存储笔记，但缺乏与 AI 工作流紧密集成的自动化构建、验证和修复能力，也没有为 LLM 消费场景优化的结构（如 frontmatter 元数据、层级索引、token 最小化的规范格式）。
- **目标用户**：使用 Claude Code 或类似 AI 编程助手的开发者、技术写作者、知识库维护者，以及需要将零散 Markdown 文档转化为结构化 LLM 知识库的工程团队。
- **在生态中的角色**：作为 Claude Code 的「Skill」插件存在，与 AI 助手形成紧密耦合。它不替代传统笔记工具，而是提供从原始文档到 LLM 消费级 Wiki 的转化层——类似 Obsidian 的「知识库」+ Git 的版本控制 + AI 驱动的自动化运维三者的结合体。与纯静态站点生成器（如 MkDocs）互补，后者侧重展示，前者侧重 AI 消费和持续维护。

## 技术栈分析

| 层级 | 技术 | 说明 |
|------|------|------|
| 运行时 | Node.js ≥ 20 | 核心依赖，提供脚本执行环境 |
| CLI 框架 | 自研 `scripts/cli.mjs` | 单文件 CLI，包含全部操作逻辑 |
| AI 策略 | TF-IDF / MiniLM / Claude | 三级策略：免费本地模型 → 轻量 embedding → 强 LLM |
| 版本控制 | Git（隔离私有仓库） | 每个 Wiki 内部维护独立的 git 历史 |
| 配置格式 | YAML | layout contract、frontmatter schema 使用 YAML |
| 测试 | 自研 testkit | 提供 fixtures 和 CI gate 工具 |

## 核心功能

- **功能1**：**Git-backed history.** Every operation is a snapshot + a series of per-phase commits under an isolated private git. Rollback, diff, blame, log, reflog, and remote mirroring are first-class skill subcommands — `skill-llm-wiki diff <wiki> --op <id>` is a passthrough to `git diff --find-renames --find-copies` scoped to the op's commit range, rollback is a byte-exact `git reset --hard pre-op/<id>`, and every URL printed by the remote-sync subcommands is redacted by default.
- **功能2**：**Stable sibling layout.** `<source>.wiki/` is the one folder a wiki ever lives in. No more `.llmwiki.v1`/`.v2`/`.v3` directory proliferation — prior states are reachable as git tags (`pre-op/<id>`, `op/<id>`) in the private repo.
- **功能3**：**Three layout modes, never guessed.** `sibling` (default), `in-place` (source IS the wiki), and `hosted` (user-chosen path with a `.llmwiki.layout.yaml` contract). Ambiguous invocations refuse and prompt — see the "Ask, don't guess" rule.
- **功能4**：**User-repo coexistence.** An auto-generated `.gitignore` hides the private metadata from any ancestor user git. The skill's isolation env block (`GIT_DIR`, `GIT_CONFIG_NOSYSTEM`, `core.hooksPath=/dev/null`, …) keeps the two gits from leaking into each other.
- **功能5**：**Tiered AI strategy.** TF-IDF (free) → local MiniLM embeddings (required, ~23 MB one-time model download, zero-API) → Claude (only for mid-band ambiguity and decisions requiring natural-language judgment). `--quality-mode tiered-fast|claude-first|deterministic` selects the escalation policy.
- **功能6**：**Deterministic slug collisions.** NEST operator auto-resolves slug-vs-member-id collisions with a deterministic `-group` suffix before apply. Your convergence loop never needs manual retries for DUP-ID.
- **功能7**：**Optional interactive review.** `skill-llm-wiki rebuild <wiki> --review` prints the post-convergence diff and commit list, lets the user approve / abort / `drop:<sha>` specific iterations, and re-runs validation + index regen on the reverted tree.
- **功能8**：**Windows parity.** The CI matrix runs the smoke suite on both `ubuntu-latest` and `windows-latest`; the isolation env switches `/dev/null` to `NUL` and enables `core.longpaths=true` on Windows.

## 架构设计

### 整体架构概述

skill-llm-wiki 采用「极简运行时 + 丰富文档」的双层架构，核心设计原则是**Claude 只读 SKILL.md，只执行 scripts/ CLI**。

安装包结构：

```
skill-llm-wiki/
├── SKILL.md                # Claude 运行时唯一读取的文件（自包含）
├── README.md               # 面向人类用户的文档
├── scripts/
│   └── cli.mjs             # 单一 CLI 入口，包含全部 6 种操作
├── guide/                  # Claude 激活时按需加载的叶子文档
│   ├── hidden-git.md       # git 历史管理
│   ├── layout-modes.md     # 布局模式选择
│   ├── tiered-ai.md        # AI 策略配置
│   └── ...
└── testkit/                # 消费者测试工具
```

**模块关系**：

1. **SKILL.md ↔ guide/**：SKILL.md 是 Claude 的「操作手册」，内部包含激活关键词路由规则。当 Claude 识别到用户意图时，从 guide/ 中按需加载对应叶子文档（如「history」关键词 → 加载 hidden-git.md）。这种设计避免了将大量文档塞进 prompt，只在需要时加载。

2. **cli.mjs ↔ 私有 Git**：CLI 不直接操作文件系统，而是通过隔离的 Git 环境（独立的 `GIT_DIR`、`GIT_CONFIG_NOSYSTEM` 等环境变量）与每个 Wiki 内部的私有仓库交互。所有变更都以 commit 形式存在，rollback 就是 `git reset --hard`。

3. **Tiered AI ↔ 重写操作符**：底层决策由四种重写操作符（DECOMPOSE、NEST、FLATTEN、PRUNE）驱动，每个操作符的触发由三级 AI 策略决定——TF-IDF 处理高频简单场景，MiniLM 处理语义相似度歧义，Claude 处理需要自然语言理解的复杂决策。

### 关键设计决策及其原因

| 决策 | 原因 | 替代方案及取舍 |
|------|------|----------------|
| **Skill 形态而非独立应用** | 直接嵌入 Claude Code 工作流，用户无需切换上下文即可管理知识库。这是「AI-native」设计——工具存在于 AI 助手的认知边界内。 | 独立桌面应用（如 Obsidian）需要用户离开对话环境操作，破坏了 AI 工作流的连续性。 |
| **隔离私有 Git** | 利用 Git 的成熟版本控制能力，同时通过环境变量隔离避免与用户自己的 Git 仓库冲突。一个 Wiki 一个私有 repo，天然支持 diff/blame/rollback。 | SQLite 或自定义版本格式：需要自己实现回滚和 diff 逻辑，Git 已有完整解决方案。 |
| **三级 AI 策略** | 大部分操作（如 slug 碰撞检测、结构验证）可以通过确定性算法或轻量模型完成，不需要每次调用 Claude API。这既节省成本又降低延迟。 | 所有决策都交给 Claude：API 成本高、延迟大，且对网络有依赖。TF-IDF/MiniLM 是零成本/低成本的本地方案。 |
| **Sibling layout 为默认** | 将 Wiki 输出与源代码物理隔离（`<source>.wiki/`），避免污染用户的工作目录。用户可以随时删除 .wiki 目录而不影响源文件。 | In-place 模式虽然省空间，但将 Wiki 结构和源文件混在一起，不适合需要保留原始文档结构的场景。 |
| **NEST 操作符的确定性后缀** | 自动解决 slug 冲突（`name-group`），避免收敛循环的无限重试。这是从实际运行中提炼出的工程经验。 | 随机命名或让用户手动解决：前者不可预测，后者打断自动化流程。 |

### 数据流和模块关系

以 **Build 操作**为例，一个典型的数据流如下：

```
用户输入 → Claude 解析意图 → SKILL.md 路由 → 调用 cli.mjs build
    ↓
Preflight（环境检查：Node.js / Git / 磁盘空间）
    ↓
Ingest（读取源目录 Markdown，解析 frontmatter）
    ↓
Schema 验证（22 条 hard invariants）
    ↓
Tiered AI 决策（TF-IDF → MiniLM → Claude，按需升级）
    ↓
重写操作符执行（DECOMPOSE / NEST / FLATTEN / PRUNE）
    ↓
收敛循环（反复应用操作符直到结构稳定）
    ↓
Index 重建（生成统一的 index.md）
    ↓
Validation（最终校验）
    ↓
Phase-commit 序列（每个阶段一个 git commit）
    ↓
输出：结构化的 <source>.wiki/ 目录 + 完整 git 历史
```

**关键设计**：每个操作都产生一个「操作 ID」（op ID），对应一组 git tag（`pre-op/<id>` 和 `op/<id>`）。这意味着用户可以精确地 diff 任意两个操作之间的变更，或者 rollback 到任意操作前的状态。

## 横向对比（与同类/竞品，可选）

| 维度 | 对比对象 | 本实现 | 差异说明 |
|------|----------|--------|----------|
| 架构分层 | Obsidian + Git 插件 | Skill（嵌入 AI 助手）+ 隔离 Git | skill-llm-wiki 没有独立 UI，完全存在于 AI 对话流中；Obsidian 是桌面应用，AI 集成依赖第三方插件 |
| 摄入方式 | MkDocs / Docusaurus | 命令行 + Claude 自然语言指令 | 传统工具需要手动写配置和导航；skill-llm-wiki 支持「Build a wiki from ./docs」这样的自然语言触发 |
| 存储格式 | Notion / 语雀 | 纯 Markdown + YAML frontmatter + 私有 Git | 传统 Wiki 使用专有格式或数据库存储；skill-llm-wiki 完全基于文本，无供应商锁定 |
| 查询机制 | Obsidian Graph / roam | 依赖外部 AI 模型消费 | skill-llm-wiki 本身不提供查询 UI，而是生成 AI 友好的结构化文档供 Claude 消费；Obsidian 提供可视化图谱 |
| 协作模式 | GitBook / 飞书文档 | 单用户 + 远程镜像（Git 推送） | 传统 Wiki 支持实时多人协作；skill-llm-wiki 目前面向个人/单用户，通过 git remote 实现跨设备同步 |

**核心差异解读**：

skill-llm-wiki 与传统 Wiki 工具的根本差异在于**设计目标**——它不是给人看的，而是给 AI 消费的。传统 Wiki（Obsidian、Notion、语雀）优先考虑人类阅读体验（可视化、搜索、协作），而 skill-llm-wiki 优先考虑 AI 消费效率（token 最小化的规范格式、结构化的 frontmatter、可验证的不变量）。这决定了它在架构上选择了「无 UI + Skill 嵌入」而非「独立应用」。

在协作方面，skill-llm-wiki 目前明显落后于商业 Wiki 工具。它依赖 Git 的异步协作模式，不支持实时多人编辑。但考虑到其目标场景是个人或小团队维护 LLM 知识库，这种取舍是合理的——实时协作对 AI 知识库场景的价值不如对人类知识库场景的价值高。

## 局限性

- **技术局限**：纯 Node.js 实现，依赖 Node.js ≥ 20 和 Git 环境。MiniLM embedding 模型需要约 23MB 的一次性下载。对于超大型文档库（多 GB），chunked iteration 虽然解决了内存问题，但处理时间可能较长。
- **使用场景局限**：目前仅支持 Markdown 源文件，不支持直接摄入 PDF、Word 等格式。没有内置的 Web 展示功能（需要配合静态站点生成器）。Claude-only 的 AI 策略意味着如果使用其他 LLM（如 GPT-4、Gemini），需要自行适配 SKILL.md。
- **维护状态考量**：仓库非常新（0 stars / 0 forks），作者 ctxr-dev 是个人开发者。文档极其完善（43 个 guide 文档），但社区验证不足。MIT 许可证降低了采用风险，但长期维护依赖作者投入。

## 适用场景

- **最适合**：已经在使用 Claude Code 的开发者，需要将项目文档、API 文档、技术笔记转化为 AI 可消费的结构化知识库；需要版本化 Wiki 管理（diff、rollback、blame）的技术团队；偏好纯文本、无供应商锁定方案的极客用户。
- **不适合**：需要多人实时协作的场景（飞书文档、Notion 更合适）；需要可视化图谱或美观 Web 展示的场景（Obsidian Publish、GitBook 更合适）；不使用 Claude Code 的用户（SKILL.md 是为 Claude 优化的，迁移成本高）；非技术用户（需要 Node.js 环境和命令行操作）。

## 关联项目

- **与 nashsu/llm_wiki 的关系**：nashsu/llm_wiki 是一个概念验证级的 LLM Wiki 实现，侧重个人笔记的 AI 增强。skill-llm-wiki 在工程化程度上远超前者——它提供了完整的 6 种操作、22 条验证不变量、三级 AI 策略和隔离 Git 版本控制。如果说 nashsu/llm_wiki 是「原型」，skill-llm-wiki 是「生产级工具」。
- **与 Obsidian + Copilot 的关系**：Obsidian 提供人类友好的笔记体验，Copilot 插件提供 AI 辅助。skill-llm-wiki 走了完全不同的路线——它不追求人类阅读体验，而是追求 AI 消费效率。两者可以互补：Obsidian 用于日常笔记，skill-llm-wiki 用于将笔记转化为 AI 知识库。
- **与 RAG 框架（如 LangChain）的关系**：RAG 框架侧重「检索-生成」pipeline，skill-llm-wiki 侧重「知识库构建与维护」。两者不在同一层——skill-llm-wiki 生成的结构化 Wiki 可以作为 RAG 的输入数据源。

---

*笔记生成时间：2026-05-03*
*数据来源：GitHub API + README 分析*
