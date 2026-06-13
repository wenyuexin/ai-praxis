---
title: "llm-wiki-compiler 研究"
category: source
source: atomicmemory/llm-wiki-compiler
url: https://github.com/atomicmemory/llm-wiki-compiler
tags:
  - llm-wiki
  - knowledge-compiler
  - typescript
  - cli
  - mcp
  - karpathy-pattern
created: 2026-05-03
updated: 2026-05-03
status: complete
---

# llm-wiki-compiler 研究

> 一个基于 Karpathy LLM Wiki pattern 的 TypeScript CLI 知识编译器。

## 基本信息

| 属性 | 值 |
|------|-----|
| **作者** | atomicmemory |
| **Stars** | 960 |
| **Forks** | 98 |
| **主要语言** | TypeScript |
| **许可证** | MIT |
| **安装** | `npm install -g llm-wiki-compiler` |
| **版本** | v0.6.0（2026-05-02） |

**一句话定位**：Raw sources in, interlinked wiki out. 自动化知识编译管线。

---

## 核心架构

```
Raw Sources ──► ingest ──► sources/ ──► compile ──► wiki/
                              │              │
                              ▼              ▼
                        多模态提取      概念提取 → 页面生成
                        (Web/PDF/图片)   语义搜索 + BM25
                        审查队列         索引 + MOC
                        溯源标注         MCP Server
```

### 三阶段管线

| 阶段 | 命令 | 职责 |
|------|------|------|
| **摄入** | `llmwiki ingest <url>` | 将原始来源（URL/文件/图片/PDF）写入 `sources/` |
| **编译** | `llmwiki compile` | 两阶段编译：提取概念 → 生成互联页面 |
| **查询** | `llmwiki query "..."` | 索引选择 + LLM 回答，支持 `--save` 写回 wiki |

### 增量机制

- **Hash-based 变更检测**：重新编译时跳过未变更的 source
- **原子写入**：编译过程使用 `.llmwiki/lock` 文件锁保护
- **孤儿标记**：source 被删除时，对应 wiki 页标记为孤儿而非直接删除

---

## 源码结构

`src/` 下 85 个文件，10 个模块：

| 模块 | 文件数 | 职责 |
|------|--------|------|
| `ingest/` | — | URL/本地文件/图片/PDF/转录文本/YouTube 摄入 |
| `compiler/` | — | 两阶段编译：概念提取 + 页面生成 |
| `providers/` | — | 多 LLM 后端（Anthropic/OpenAI/Ollama/MiniMax） |
| `export/` | — | 输出格式处理 |
| `mcp/` | — | Model Context Protocol 服务端 |
| `schema/` | — | 类型页面种类定义（concept/entity/comparison/overview） |
| `linter/` | — | 规则化检查（断链、孤立页、空页等） |
| `commands/` | — | CLI 命令实现 |
| `adapters/` | — | Session export 适配器（Claude/Codex/Cursor） |
| `utils/` | — | 共享工具函数 |

---

## 关键特性

### 1. 多模态摄入（v0.5.0+）

支持来源类型：`web` | `file` | `image` | `pdf` | `transcript`

- **图片**：通过 LLM vision 能力提取内容
- **PDF**：`pdf-parse` 动态懒加载提取文本 + 元数据
- **转录**：`.vtt` / `.srt` / 内容嗅探的 `.txt`
- **YouTube**：URL 自动路由到 transcript 提取
- **Session export**（v0.6.0）：Claude `.jsonl` / Codex `.json` / Cursor `.json`

### 2. 语义搜索与检索

- **Chunk-level 嵌入**（v0.5.0）：页面按段落 + 标题边界切分，单独嵌入
- **BM25 重排序**：0.5x 余弦相似度 + BM25 分数混合
- **Query 路由**：优先 chunk 命中，回退到页面级检索和全索引选择
- **冷启动**：空 store 自动全量嵌入

### 3. Claim-level 溯源（v0.4.0）

引用格式支持精确到行号范围：

```markdown
^[paper.md:42-58]     # 行号范围
^[paper.md:7]         # 单行
^[paper.md#L42-L58]   # GitHub anchor 形式
^[a.md, b.md:1-3]     # 多源混合
```

- `extractClaimCitations(body)` 返回结构化 `{raw, spans: [{file, lines?}]}`
- `inspectProvenance(body)` 按源文件分组
- Lint 规则检查越界 span、畸形引用

### 4. 审查队列（v0.3.0）

`llmwiki compile --review`：
- 生成页面写入 `.llmwiki/candidates/` 而非直接修改 `wiki/`
- `review list|show|approve|reject` 子命令人工审核
- `approve` 写入页面并刷新索引/MOC/嵌入
- `reject` 归档到 `.llmwiki/candidates/archive/`
- MCP `wiki_status` 暴露 `pendingCandidates`

审查候选携带：
- `schemaViolations` — schema 层违规
- `provenanceViolations` — 引用格式违规

### 5. Schema 层（v0.4.0）

项目可声明 `.llmwiki/schema.json|yaml|yml` 定义：
- 页面种类（`concept`, `entity`, `comparison`, `overview`）
- 每类 `minWikilinks`
- Seed 页面（schema 驱动自动生成）

命令：`schema init`（写 starter schema）、`schema show`（查看 resolved schema）

Lint 规则：`schema-cross-link-minimum` 强制执行每类链接期望。

### 6. MCP Server（v0.2.0）

`llmwiki serve` 暴露 7 个 tools + 5 个 resources：

**Tools**：`ingest_url`, `compile_wiki`, `query_wiki`, `search_wiki`, `lint_wiki`, `read_page`, `review_candidate`

**Resources**：`wiki://index`, `wiki://moc`, `wiki://pages/{slug}`, `wiki://sources`, `wiki://status`

支持 Claude Desktop / Cursor 配置。

### 7. 置信度与矛盾元数据（v0.3.0）

可选 frontmatter 字段：
- `confidence` — 置信度（多源合并取 `min`）
- `provenanceState` — 溯源状态（合并时为 `'merged'`）
- `contradictedBy` — 矛盾来源 slug 列表（去重并集）
- `inferredParagraphs` — 推断段落数（v0.6.0 后改为 body 派生）

对应 Lint 规则：`low-confidence`, `contradicted-page`, `excess-inferred-paragraphs`

### 8. 多 Provider 支持

| Provider | 配置 |
|----------|------|
| Anthropic（默认） | `ANTHROPIC_API_KEY` / `ANTHROPIC_AUTH_TOKEN` |
| OpenAI | `LLMWIKI_PROVIDER=openai` + `OPENAI_API_KEY` |
| Ollama | `LLMWIKI_PROVIDER=ollama` |
| MiniMax | OpenAI-compatible endpoint |

超时配置：`LLMWIKI_REQUEST_TIMEOUT_MS`（通用）、`OLLAMA_TIMEOUT_MS`（Ollama 专用，默认 30 分钟）

### 9. 输出语言控制（v0.6.0）

`LLMWIKI_OUTPUT_LANG` 环境变量 + `--lang <code>` CLI flag：
- 所有 prompt builder（提取、页面生成、seed 页、查询回答）追加 `Write the output in <lang>.`
- 支持 `--lang Chinese`, `--lang Japanese` 等

### 10. 防御性预算控制（v0.6.0）

`LLMWIKI_PROMPT_BUDGET_CHARS`（默认 200,000）：
- 解决热门概念多源合并时上下文溢出问题
- 超预算时公平分配每源配额，附加截断标记

---

## 开发规范

代码质量要求严格：

- **文件大小限制**：< 400 行（不含注释），超限即重构
- **函数大小限制**：< 40 行（不含注释和 catch/finally）
- **Fallow 代码健康分析**：死代码、重复、复杂度检查（CI 必过）
- **测试覆盖**：632 个测试（v0.6.0），Vitest 框架
- **Git hooks**：pre-commit（fallow + tsc）、pre-push（build + test）

---

## 发布历程

| 版本 | 日期 | 核心更新 |
|------|------|----------|
| 0.1.0 | 2026-04-05 | 初始发布：ingest/compile/query/watch |
| 0.2.0 | 2026-04-16 | MCP Server、语义搜索、多 Provider、Lint |
| 0.3.0 | 2026-04-23 | 审查队列、置信度元数据、Husky hooks |
| 0.4.0 | 2026-04-25 | Claim-level 溯源、Schema 层、Slug wikilinks |
| 0.5.0 | 2026-04-27 | 多模态摄入、Chunk 嵌入、BM25、Node 24 最低版本 |
| 0.5.1 | 2026-04-27 | 修复 youtube-transcript 导入崩溃 |
| 0.6.0 | 2026-05-02 | Session 历史导入、输出语言、防御预算控制、CJK 修复 |

---

## 与 Karpathy 原始模式的差异

| 维度 | Karpathy 模式 | llm-wiki-compiler |
|------|--------------|-------------------|
| **实现方式** | 描述性 pattern | 完整 CLI 工具 |
| **自动化** | 手动操作 | `ingest` / `compile` / `query` 命令 |
| **增量更新** | 手动跟踪 | Hash-based 变更检测 |
| **溯源** | 段落级 `^[filename]` | Claim-level 行号范围 |
| **质量控制** | 人工审核 | 审查队列 + Lint 规则 |
| **多模态** | 文本 | Web/图片/PDF/转录/YouTube |
| **AI 集成** | 通用 LLM | MCP Server + 多 Provider |
| **Schema** | 无 | 类型页面种类 + Seed 页面 |

---

## 适用场景

- **自动化知识库构建**：CI/CD 集成，定时摄入新来源
- **多模态内容整理**：研究论文（PDF）、会议记录（转录）、网页、图片
- **团队协作**：审查队列确保 AI 生成内容经人工确认后入库
- **学术研究**：Claim-level 溯源精确到行号，适合引用验证
- **多语言知识库**：`--lang Chinese` 输出中文 wiki 页面
- **Agent 集成**：通过 MCP 让 Claude/Cursor 直接查询/操作 wiki

---

## 相关链接

- [GitHub 仓库](https://github.com/atomicmemory/llm-wiki-compiler)
- [Karpathy's LLM Wiki pattern](https://github.com/karpathy/llm-wiki)

---

*研究基于 2026-05-03 仓库快照，v0.6.0 版本。*
