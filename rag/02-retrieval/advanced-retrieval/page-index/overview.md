# PageIndex：对象总览

- **对象名称**：PageIndex
- **对象类型**：开源检索框架 / 索引库
- **开发者/机构**：VectifyAI（Mingtian Zhang, Yu Tang）
- **上游地址**：https://github.com/VectifyAI/PageIndex（MIT License, 32.9k+ stars）
- **官方文档**：https://docs.pageindex.ai
- **研究范围**：长文档层级树索引构建、vectorless 检索协议、单文档树导航

---

## 它是什么

PageIndex 是 VectifyAI 于 2025 年 9 月发布的开源检索索引库。它的核心能力是：把长文档解析为一个层级树索引，然后让 LLM 在这个树结构上推理导航，而不是在向量数据库中做语义匹配。

核心 Python SDK 位于本地 `pageindex/` 目录，约 2600 行，包含两个独立索引管线（PDF / Markdown）、一个本地客户端封装和一个检索工具函数层。（另有 TypeScript SDK、MCP server 和云 API 作为独立的外部扩展层，不在本地仓库范围内。）

官方自称 "vectorless, reasoning-based RAG system / framework"，核心主张可以概括为：层级树索引 + LLM 推理导航，替代传统分块 + 向量检索的 RAG 路径。

> "PageIndex — a vectorless, reasoning-based RAG system that builds a hierarchical tree index from long documents and uses LLMs to reason over that index for agentic, context-aware retrieval."

**证据等级**：`Observed`（官方自述）；`Verified`（本地 SDK 核心库结构与该定位中的索引/检索部分一致）。

## 为什么值得研究

PageIndex 不只是另一个开源检索项目。它在源码层面实现了几个当前仓库中没有第二份同等深度研究的关键机制：

- **PDF 树索引构建**：基于 LLM 的目录页识别、章节-页码映射验证、non-chunking 层级树结构
- **vectorless 检索协议**：get_document_structure（获取树结构）→ LLM 推理定位 → get_page_content（精确获取页面内容）——完全不依赖向量嵌入
- **双重标题验证链路**：appear（标题是否出现在页上）与 appear_start（标题是否在页开头）——后者用于树构建时页码的精确切分

这些机制不只属于 PageIndex 本身。它们共同指向一个更宽的想法：**page-level / structure-aware indexing**，即「不把文档切碎，而是保留其层级结构，让 LLM 在上面推理导航」。PageIndex 是这个想法当前最高信号的工程载体。

## 它代表什么，不等于什么

PageIndex 能代表上述索引思路，**但不等于**该思路已在开源生态中形成成熟、丰富、可充分横向比较的对象簇。基于当前研究范围，几点观察：

- 在已完成源码核验的方向中，PageIndex 是 page-level / structure-aware indexing 这条路线上最完整的开源实现
- 类似的树索引方案（如 llama-index 的 tree summarization、Karpathy 模式中的 index.md）在机制上和工程深度上与 PageIndex 有本质差异
- 同一方向上目前还缺少足够多的可横向比较对象

因此，本目录把它作为**代表性对象入口**来研究——通过一个具体项目的深入研究来理解这个方向，而不是作为该方向的完整主题综述。

## 核心机制

两个独立索引管线 + 一个检索工具层：

1. **PDF 索引管线**（`page_index.py`，1154 行）：从 PDF 提取页文本 → LLM 辅助目录页识别 → 章节结构提取 → 标题-页面验证 → 树构建 → 可选 LLM 摘要。

2. **Markdown 索引管线**（`page_index_md.py`，342 行）：正则提取标题（跳过代码块） → 按层级建树 → 可选 tree thinning → 可选 LLM 摘要。

3. **检索工具层**（`retrieve.py`、`client.py`）：`get_document()` → 元数据；`get_document_structure()` → 树结构（不含文本，省 token）；`get_page_content(doc_id, "5-7")` → 特定页面内容。

**证据等级**：`Verified` — 源码中的独立函数和类。

## 与传统 RAG 的主要区别（源码维度）

| 维度 | 传统 RAG | PageIndex |
|---|---|---|
| 文档表示 | 分块，固定或语义分割 | 层级树，页面级或章节级节点 |
| 索引存储 | 向量嵌入到向量数据库 | `_meta.json` + 单个 doc JSON 文件（本地 workspace） |
| 检索方式 | 语义相似度匹配 | LLM 先读树结构，推理定位页码范围，再读目标页 |
| 分块问题 | chunking 切碎文档逻辑结构 | 无 chunking，页级精度 |
| 向量依赖 | 必需 | 零依赖 |
| 检索的可解释性 | 相似度分数，不透明 | 树路径可追踪 |

**证据等级**：`Verified` — `client.py:55-130` 索引流程无向量操作；`retrieve.py:100-107` 返回结构不返回文本。

## 与 OpenKB / LLM Wiki 的边界

`pageindex/` 代码中无 OpenKB 引用、无 wiki 编译或多文档知识库管理能力。PageIndex 是完全独立的索引库。OpenKB 是把它用作长文档检索引擎的一条 LLM Wiki 系统实现路线，两者是系统—组件关系，不是同一层级的范式归属关系。

**证据等级**：`Verified`（PageIndex 独立性基于源码）、`Observed`（OpenKB 依赖关系基于官方 README 声明）

## 仍不确定的点

1. **示例层 reasoning 模式是否值得封装为独立抽象**：核心库不包含 LLM 推理循环；两条示例路径（Agents SDK 路径 vs 手动 prompt 路径）不共享代码、不封装。是否值得抽象为一个可复用的 "TreeNavigationAgent" 模式是开放问题。`Inferred`
2. **多文档检索的真实模式**：本地 SDK workspace 支持多文档索引存储，但检索接口是单文档粒度。Cloud API 宣称支持多文档，但不可核验。`Unverified`
3. **Cloud API 具体能力断层**：`submit_document`、`chat_completions` 等方法不在本地 SDK 中，Cloud API 端的增强 OCR 和流式检索能力不可核验。`Unverified`
4. **PageIndex 之外的同思路实现还未充分出现**：当前缺乏一组可横向比较的 page-level indexing 开源对象。这一缺口是否会随时间自然填补，还是说明这个思路的实际适用面偏窄，仍不确定。`Inferred`

## Evidence

- Status: `Verified`（核心事实）、`Observed`（绩效数据与生态比较）、`Inferred`（抽象定位判断）
- Sources: 基准来源：`https://github.com/VectifyAI/PageIndex`；后续引用：`pageindex/page_index.py`、`pageindex/page_index_md.py`、`pageindex/client.py`、`pageindex/retrieve.py`；官方 GitHub README、官方 blog
- Version Basis: `branch main`, `commit 5a18553284ed`
- Observed At: `2026-06-13`
- Scope: 本页中所有源码结论仅适用于本次观察到的 `PageIndex` 默认分支当前截面
- Drift Risk: `medium`
- Trace: 基于本地源码核验（`pageindex/page_index.py`、`pageindex/client.py`、`pageindex/retrieve.py`、`pageindex/utils.py`、`pageindex/page_index_md.py`；示例层 `examples/agentic_vectorless_rag_demo.py`、`cookbook/agentic_retrieval.ipynb`）
- Needs: Cloud API 与本地 SDK 的能力差异需产品文档或代码级确认
