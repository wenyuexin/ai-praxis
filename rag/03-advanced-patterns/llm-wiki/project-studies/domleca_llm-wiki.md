---
skill: synthesize-repo
domain: github
theme: knowledge_base
purpose: "分析 Obsidian 插件形态的 LLM Wiki 实现——与 nashsu 桌面端方案的对比研究"
sources:
  - "https://github.com/domleca/llm-wiki"
tags:
  - github
  - TypeScript
  - obsidian-plugin
  - knowledge-base
  - llm
  - rag
  - local-first
  - karpathy-pattern
created: "2026-05-04"
updated: "2026-05-04"
status: "active"
---

# domleca/llm-wiki

> **一句话总结**：LLM Wiki 的 Obsidian 插件实现——将 Karpathy 的 LLM Wiki 设计模式打包为 Obsidian 插件，零切换成本地在现有笔记生态中构建结构化知识库，默认全本地运行（Ollama），通过三路混合检索（关键词 + 语义 + 目录结构）+ Reciprocal Rank Fusion 实现精准问答。

## 基本信息

- **仓库**：domleca/llm-wiki
- **描述**：Obsidian plugin that turns your vault into a structured, queryable knowledge base using LLMs
- **Stars**：123 | **Forks**：7
- **最近更新**：2026-04-30
- **Open Issues**：4
- **创建时间**：2026-04-08
- **主要语言**：TypeScript
- **许可证**：MIT License（可自由商用）
- **链接**：https://github.com/domleca/llm-wiki

## 项目定位

- **解决的问题**：Obsidian 用户积累了大量笔记，但笔记间的关联依赖手动创建 `[[双向链接]]`，且无法对笔记内容进行自然语言问答。传统 RAG 方案需要将笔记导出到外部系统，破坏了 Obsidian 的工作流。LLM Wiki 让用户在 Obsidian 内部直接对笔记进行结构化提取和自然语言问答，零切换成本。
- **目标用户**：已有 Obsidian 使用习惯的知识工作者和研究者，希望在现有 vault 上叠加 LLM 能力，而非迁移到新工具。对隐私敏感、偏好本地处理的用户。
- **在生态中的角色**：与 nashsu/llm_wiki（独立桌面应用）和 SamurAIGPT/llm-wiki-agent（CLI Agent）形成三足鼎立，分别占据"插件集成"、"独立全栈"、"开发者 CLI"三个生态位。domleca 的核心优势是零切换成本——不离开 Obsidian。

## 上手成本

- **安装方式**：Obsidian 社区插件市场（等待审核通过）、手动安装（下载 3 个文件到 `.obsidian/plugins/llm-wiki/`）、从源码编译
- **前置依赖**：Ollama（本地 LLM 运行时）+ qwen2.5:7b 模型（~4.7 GB）+ nomic-embed-text 模型（~275 MB）。首次提取 600 笔记在 M2 MacBook Air 上约 4 小时。
- **首次配置**：安装 Ollama → 拉取模型 → 安装插件 → 运行 `LLM Wiki: Run extraction now` 命令。可选配置云端 LLM provider（OpenAI/Anthropic/Google）。
- **学习曲线**：Quick Start 文档清晰。核心操作只需两个命令（提取 + 提问），学习成本极低。后续增量提取自动触发，无需额外操作。

## 技术栈分析

| 层级 | 技术 | 说明 |
|------|------|------|
| 宿主平台 | **Obsidian Plugin API** | 作为 Obsidian 插件运行，复用 vault 和 UI 框架 |
| 语言 | TypeScript（严格模式） | 476 个测试，类型检查 + lint |
| 构建 | esbuild | 快速打包，`node esbuild.config.mjs production` |
| 测试 | Vitest | 476 个测试用例 |
| LLM 运行时 | Ollama（默认） | 本地免费，qwen2.5:7b 用于提取和问答 |
| Embedding | nomic-embed-text（Ollama） | 语义搜索，本地运行 |
| 云端 LLM | OpenAI / Anthropic / Google | 可选，需配置 API key |

### 关键设计决策

**为什么选择 Obsidian 插件而非独立应用？**
这是 domleca 与 nashsu 最根本的架构差异。Obsidian 插件的优势是零切换成本——用户不需要导出笔记、不需要学习新工具、不需要维护两套系统。代价是功能受限于 Obsidian 的插件 API，无法实现自定义布局、独立窗口、原生菜单等桌面应用能力。

**为什么默认用 Ollama 而非云端 API？**
隐私优先。Obsidian 用户往往选择 Obsidian 就是因为本地存储，如果 LLM 处理需要上传笔记到云端，就破坏了这一核心价值。Ollama 让一切在本地完成。云端 API 作为可选项提供，且在设置中明确标注隐私风险。

**为什么用 kb.json 而非纯 Markdown 存储知识库？**
`wiki/kb.json` 是结构化知识库（实体、概念、关系的 JSON 数据），而 `wiki/` 目录下的 Markdown 页面是面向用户浏览的渲染产物。这种分离让检索引擎可以直接查询结构化数据，而不需要解析 Markdown。Markdown 页面则利用 Obsidian 的 Bases 功能实现过滤和排序。

**为什么用 Reciprocal Rank Fusion 而非加权求和？**
RRF 不需要调权重，对三种检索策略（关键词、语义、目录结构）的分数尺度不敏感。每种策略只提供排序，RRF 基于排名融合，避免了"语义分数远大于关键词分数"之类的尺度问题。这是一个工程上更鲁棒的选择。

## 核心功能

1. **知识提取（Extraction）** — LLM 从笔记中提取实体、概念和 9 种关系类型，合并为 `kb.json`
2. **Wiki 页面生成** — 从知识库自动生成 entities/concepts/sources 三类 Markdown 页面
3. **三路混合检索** — 关键词 + 语义 + 目录结构，RRF 融合
4. **自然语言问答** — 基于检索结果的流式回答，附带可点击源链接
5. **增量更新** — 保存笔记时后台重新提取该文件，可选夜间全量刷新
6. **多轮对话** — 对话持久化，可恢复历史会话

> 以上为 README 提取的功能概要。以下对每个核心功能进行深度剖析：

### 1. 知识提取（Extraction）

**设计动机**：Obsidian 笔记中蕴含大量隐含知识（人物关系、概念层级、事件因果），但手动整理这些关联不现实。需要 LLM 自动提取结构化信息。

**实现策略**：逐条笔记发送给 LLM，提示词为"what entities, concepts, and connections are in this text?"。LLM 返回结构化数据（名称、类型、描述、关系），合并到 `wiki/kb.json`。支持 9 种关系类型。首次提取为全量（600 笔记 ~4 小时），后续仅提取变更笔记。

**工程价值**：单步提取（而非 nashsu 的两步 CoT），简单但缺乏可调试性。增量提取策略（仅变更笔记）大幅降低了更新成本，这是实用的工程折衷。

### 2. 三路混合检索

**设计动机**：单一检索策略覆盖不足——关键词搜索无法处理同义不同词，语义搜索可能遗漏精确匹配，目录结构提供了用户隐含的分类信号。需要多策略互补。

**实现策略**：三种策略并行执行——关键词匹配（找包含相同术语的笔记）、语义相似度（embedding 模型找语义相近的笔记，即使用词不同）、vault 目录结构（优先考虑用户指定范围内的笔记）。结果使用 Reciprocal Rank Fusion 合并——每种策略独立排序，RRF 基于排名位置（而非原始分数）融合，避免分数尺度问题。

**工程价值**：RRF 是一个实用的工程选择，不需要手动调权重。相比 nashsu 的四信号模型（Source Overlap ×4.0, Direct Link ×3.0 等），domleca 的 RRF 更简单但也更难针对特定场景优化。vault 目录结构作为检索信号是 Obsidian 插件的独特优势——nashsu 的独立应用没有这种隐含分类信息。

### 3. 增量更新

**设计动机**：全量提取耗时过长（4 小时/600 笔记），用户不可能每次修改笔记后等数小时。

**实现策略**：保存笔记时自动触发该文件的重新提取（后台执行）。可选夜间全量刷新（处理跨笔记的关系更新）。`wiki/` 文件夹默认从搜索、Quick Switcher 和图谱视图中隐藏，不干扰用户正常工作流。

**工程价值**：增量更新是知识库系统的基本需求。domleca 的实现简洁——单文件重新提取 + 可选全量刷新，与 nashsu 的持久化摄入队列 + SHA-256 缓存相比更简单，但在并发控制和崩溃恢复方面不够健壮。

## 架构设计

### 核心数据流

```
Obsidian Vault（用户笔记）
    ↓
Extraction（逐条笔记 → LLM 提取实体/概念/关系）
    ↓
wiki/kb.json（结构化知识库）
    ↓
Page Generation（kb.json → entities/concepts/sources Markdown 页面）
    ↓
User Query → 三路检索（关键词 + 语义 + 目录结构）→ RRF 融合
    ↓
Top-ranked Context + Question → LLM → 流式回答 + 源链接
```

### 数据流和模块关系

**笔记提取流程**：
1. 用户运行 `Run extraction now` 或保存笔记触发增量提取
2. 插件读取笔记内容 → 发送到 Ollama/云端 LLM → 获取结构化数据
3. 结构化数据合并到 `wiki/kb.json`（实体、概念、关系）
4. 基于知识库生成 `wiki/entities/`、`wiki/concepts/`、`wiki/sources/` 目录下的 Markdown 页面

**用户查询流程**：
1. 用户输入问题 → 三路检索并行执行
2. 关键词检索：匹配笔记中的术语
3. 语义检索：nomic-embed-text embedding → cosine similarity
4. 目录结构检索：优先指定范围内的笔记
5. RRF 融合三路结果 → Top-ranked context
6. Context + Question + Chat History → LLM → 流式回答
7. 回答与检索源交叉引用 → 生成可点击源链接

**增量更新流程**：
1. 用户保存笔记 → 触发该文件后台重新提取
2. 更新 kb.json 中的相关条目
3. 可选：夜间全量刷新（处理跨笔记关系变更）

### 与上游/理论基础对比

| 维度 | Karpathy 原始设计 | domleca 实现 | 差异说明 |
|------|------------------|-------------|---------|
| 架构形态 | 抽象设计模式文档 | Obsidian 插件 | 从"复制粘贴给 Agent"到"一键安装的插件" |
| 知识存储 | raw/wiki/schema 三层 | notes + wiki/kb.json + wiki 页面 | 简化了层级，kb.json 作为结构化核心 |
| 提取方式 | 概念性单步 | 单步 LLM 提取（实体+概念+9种关系） | 保持单步，未采用 nashsu 的两步 CoT |
| 查询机制 | 未明确 | 三路混合 + RRF | 实现了具体检索管线 |
| 知识关联 | `[[wikilink]]` | 9 种关系类型 + 目录结构信号 | 关系类型更丰富，利用了 vault 隐含信息 |
| 方向性 | 无 | 无 purpose.md | 与 nashsu 不同，未引入方向性控制 |
| 隐私 | 未强调 | 默认 Ollama 本地，零数据外传 | 隐私优先作为核心卖点 |

## 数据安全与隐私

- **数据存储位置**：完全本地（Obsidian vault 内 `wiki/` 目录），无云端同步。kb.json 和生成页面都在本地文件系统。
- **API Key 管理**：Obsidian 插件设置中配置，存储在本地 Obsidian 配置目录。云端 provider 为可选项。
- **数据传输**：默认 Ollama 模式下，所有处理在本地完成，零数据外传。云端 provider 会将笔记内容发送到对应 API，设置中明确标注隐私风险。
- **隐私设计**：无遥测、无分析、无追踪。这是项目的核心卖点之一——"Your notes never leave your computer."

## 横向对比（与 nashsu/llm_wiki）

| 维度 | nashsu/llm_wiki | domleca/llm-wiki | 差异说明 |
|------|----------------|-----------------|---------|
| 架构分层 | Tauri v2 独立桌面应用 | Obsidian 插件 | nashsu 功能自由度最高，domleca 零切换成本 |
| 摄入方式 | Two-Step CoT + 增量缓存 + 持久队列 | 单步提取 + 保存时增量 | nashsu 质量更高、更健壮，domleca 更简单 |
| 存储格式 | Markdown + frontmatter + 图谱索引 | kb.json（结构化核心）+ Markdown 页面 | domleca 用 JSON 存储知识结构，检索效率更高 |
| 查询机制 | 四阶段管线（分词+向量+图谱+预算控制） | 三路混合 + RRF | nashsu 更精细，domleca 更简洁 |
| 协作模式 | 单用户 | 单用户（继承 Obsidian） | 两者均无多人协作 |
| 知识图谱 | 四信号模型 + Louvain 社区检测 + 图谱洞察 | 无可视化图谱 | nashsu 独有，domleca 依赖 Obsidian 自带图谱 |
| 多模态 | Caption-First Hybrid | 不支持 | nashsu 独有 |
| 异步审核 | 审核队列 + 级联删除 | 无 | nashsu 独有 |
| 场景模板 | 5 种预配置 purpose.md + schema.md | 无 | nashsu 独有 |

**差异解读**：两个项目的差异根因在于**产品形态决定功能边界**。nashsu 选择独立桌面应用，获得了最大的交互设计自由度（三栏布局、图谱可视化、自定义管线），但代价是用户需要迁移到新工具。domleca 选择 Obsidian 插件，获得了零切换成本和生态红利（vault 结构、Bases、自带图谱），但受限于插件 API 无法实现自定义窗口和复杂交互。这不是技术能力的差异，而是设计哲学的差异——nashsu 追求"功能最完整"，domleca 追求"最易上手"。

## 局限性

- **技术局限**：
  - 首次提取耗时过长（600 笔记 ~4 小时）。根因：串行处理 + 本地 LLM 速度限制，是架构约束（Ollama 本地运行）和 LLM 能力的双重限制。
  - 单步提取缺乏可调试性。根因：相比 nashsu 的两步 CoT（分析可单独审查），单步提取的分析和生成混合在一起，质量问题难以定位。这是设计选择——简单性优先。
  - 无知识图谱可视化。根因：作为 Obsidian 插件，依赖宿主的图谱功能，无法实现自定义图谱渲染。这是插件形态的架构约束。
  - 无 purpose.md 方向性控制。根因：项目选择了简洁路线，未引入 Karpathy 设计中的方向性组件。LLM 在提取时没有明确的"研究目标"作为上下文。
- **使用场景局限**：
  - 不适合非 Obsidian 用户
  - 不适合需要大规模并发处理的场景（无持久化队列）
  - 不适合需要多人协作的知识库
- **维护状态考量**：
  - 123 stars，7 forks，4 open issues，项目较新但活跃
  - 同样基于 Karpathy 设计模式，理论基础清晰
  - 正在申请 Obsidian 社区插件市场审核
  - 已有社区贡献（Mistral provider、多文件夹索引）

## 工程亮点

| 维度 | 工程实践 | 技术价值 |
|------|---------|---------|
| 检索融合 | Reciprocal Rank Fusion 合并三路结果 | 无需调权重，工程鲁棒性高 |
| 增量更新 | 保存笔记时自动触发单文件重新提取 | 更新成本从小时级降到秒级 |
| 隐私设计 | 默认 Ollama 本地运行，零数据外传 | 与 Obsidian 用户的隐私预期一致 |
| 非侵入式 | wiki/ 默认从搜索/图谱/Quick Switcher 隐藏 | 不干扰用户现有工作流 |
| 质量保障 | 476 个测试 + 严格 TypeScript + lint | 代码质量可控 |
| 多 provider | Ollama/OpenAI/Anthropic/Google 统一接口 | 灵活选择，本地/云端无缝切换 |

## 适用场景

- **最适合**：
  - 已有 Obsidian 使用习惯、希望叠加 LLM 能力的用户
  - 对隐私敏感、要求本地处理的用户
  - 笔记量中等（<1000 篇）且需要快速问答的场景
  - 想要最简单的 LLM Wiki 入门体验的用户
- **不适合**：
  - 不使用 Obsidian 的用户
  - 需要知识图谱可视化的深度研究者
  - 需要多模态图片处理的用户
  - 笔记量超大（>5000 篇）的用户（首次提取时间过长）
  - 需要团队协作的场景

## 关联项目

| 项目 | 关系 | 选择建议 |
|------|------|----------|
| [nashsu/llm_wiki](https://github.com/nashsu/llm_wiki) | 替代（独立桌面端） | 追求功能完整性和深度分析选 nashsu；追求零切换成本和简洁性选 domleca |
| [Karpathy's llm-wiki.md](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) | 上游/理论基础 | 两个实现都基于此设计模式，想理解核心思想先看原版 |
| [Obsidian](https://obsidian.md) | 宿主平台 | domleca 依赖 Obsidian 运行，不是独立产品 |

---

*笔记生成时间：2026-05-04*
*数据来源：GitHub API + README 分析*
*建议下次审查：2026-10-31*
