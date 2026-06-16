# PageIndex：对象总览

- **对象名称**：PageIndex
- **对象类型**：开源检索框架 / 索引库
- **开发者/机构**：VectifyAI（Mingtian Zhang, Yu Tang）
- **上游地址**：https://github.com/VectifyAI/PageIndex（MIT License, 32.9k+ stars）
- **官方文档**：https://docs.pageindex.ai
- **研究范围**：长文档层级树索引构建、vectorless 检索协议、单文档树导航

---

## 它是什么

PageIndex 是 VectifyAI 于 2025 年 9 月发布的开源检索索引库。它的核心主张不是把文档切成 chunks 后送入向量数据库，而是先把长文档恢复成一个可导航的层级树，再让 LLM 基于这棵树定位应读取的章节或页范围，最后只取局部正文完成回答。

因此，更准确的理解不是“另一个 RAG 项目”，而是：**一个以层级树索引和结构化导航为核心的 vectorless 检索框架**。官方将其描述为 "vectorless, reasoning-based RAG system / framework"；从本地 SDK 可见实现来看，这一定位在索引构建与检索协议层面基本成立。

> "PageIndex — a vectorless, reasoning-based RAG system that builds a hierarchical tree index from long documents and uses LLMs to reason over that index for agentic, context-aware retrieval."

核心 Python SDK 位于本地 `pageindex/` 目录，约 2600 行，包含两个独立索引管线（PDF / Markdown）、一个本地客户端封装和一个检索工具函数层。（另有 TypeScript SDK、MCP server 和云 API 作为独立的外部扩展层，不在本地仓库范围内。）

**证据等级**：`Observed`（官方自述）；`Verified`（本地 SDK 核心库结构与该定位中的索引/检索部分一致）。

## 它怎么工作

从单篇文档的角度看，PageIndex 的最小工作闭环可以概括为五步：

1. **先把文档构造成层级树**：PDF 路径会从分页文本中恢复章节结构，再生成带页范围的树；Markdown 路径直接从标题层级构树。
2. **先暴露结构，不先暴露全文**：调用方先获取文档树结构，而不是直接把完整正文交给 LLM。
3. **由 LLM 或上层 agent 做结构化粗定位**：基于树结构判断相关章节、候选节点或页范围。
4. **只读取局部正文**：再按推理结果获取紧凑页段或局部文本，而不是整篇文档全文读取。
5. **由调用方完成最终回答**：本地核心库负责结构与内容提取，不直接内建端到端问答代理。

因此，PageIndex 在本地 SDK 中更像“结构优先的检索底座”：树负责缩小范围，页内容负责支撑答案，真正把这条链串起来的是上层 agent 或调用方逻辑。

## 为什么值得研究

PageIndex 不只是另一个开源检索项目。它在源码层面实现了几个当前仓库中没有第二份同等深度研究的关键机制：

- **PDF 树索引构建**：基于 LLM 的目录页识别、章节-页码映射验证、non-chunking 层级树结构
- **vectorless 检索协议**：`get_document_structure`（获取树结构）→ LLM 推理定位 → `get_page_content`（精确获取页面内容）——完全不依赖向量嵌入
- **结构优先导航**：先看结构、再缩小范围、再读取局部正文，而不是先做全文语义召回
- **双重标题验证链路**：`appear`（标题是否出现在页上）与 `appear_start`（标题是否在页开头）——后者用于树构建时页码的精确切分

这些机制不只属于 PageIndex 本身。它们共同指向一个更宽的想法：**page-level / structure-aware indexing**，即「不把文档切碎，而是保留其层级结构，让 LLM 在上面推理导航」。PageIndex 是这个想法当前最高信号的工程载体。

## 核心机制骨架

从本地源码可见实现看，PageIndex 的主体可以收敛为“两条索引管线 + 一个检索协议层”：

1. **PDF 索引管线**（`page_index.py`，1154 行）：从 PDF 提取页文本 → LLM 辅助目录页识别 → 章节结构提取 → 标题-页面验证 → 树构建 → 可选 LLM 摘要。
2. **Markdown 索引管线**（`page_index_md.py`，342 行）：正则提取标题（跳过代码块） → 按层级建树 → 可选 tree thinning → 可选 LLM 摘要。
3. **检索协议层**（`retrieve.py`、`client.py`）：`get_document()` → 元数据；`get_document_structure()` → 树结构（不含文本，省 token）；`get_page_content(doc_id, "5-7")` → 特定页面内容。

这里最关键的不是“有树”，而是**树在检索中承担了结构化粗定位职责**：调用方先利用树判断应该去哪几页，再按页段取内容。也正因为如此，PageIndex 的本地实现更像“树导航协议”而不是传统意义上的“向量召回器”。

**证据等级**：`Verified` — 源码中的独立函数、类与公开检索接口。

## 与传统 RAG 的主要区别（源码维度）

| 维度 | 传统 RAG | PageIndex |
|---|---|---|
| 文档表示 | 分块，固定或语义分割 | 层级树，页面级或章节级节点 |
| 索引存储 | 向量嵌入到向量数据库 | `_meta.json` + 单个 doc JSON 文件（本地 workspace） |
| 检索方式 | 语义相似度匹配 | LLM 先读树结构，推理定位页码范围，再读目标页 |
| 分块问题 | chunking 切碎文档逻辑结构 | 无 chunking，页级精度 |
| 向量依赖 | 必需 | 零依赖 |
| 检索的可解释性 | 相似度分数，不透明 | 树路径可追踪 |

更白话地说，传统 RAG 通常是“先把全文打散再召回”，而 PageIndex 更接近“先恢复结构，再缩小阅读范围”。

**证据等级**：`Verified` — `client.py:55-130` 索引流程无向量操作；`retrieve.py:100-107` 返回结构不返回文本。

## 单文档与多文档的真实边界

PageIndex 当前最稳的本地能力边界是：**单文档树导航检索闭环成立，多文档只解决存储，不解决跨文档检索。**

- 本地 `workspace` 支持同时保存多个文档索引：每篇文档各自一个 JSON 文件，并由 `_meta.json` 做文档级注册。
- 但三个公共检索接口 `get_document(doc_id)`、`get_document_structure(doc_id)`、`get_page_content(doc_id, pages)` 都要求显式传入 `doc_id`。
- 这意味着本地 SDK 默认前提是：调用方已经先选定要查哪一篇文档；PageIndex 负责的是这篇文档内部的结构化定位，而不是多篇文档之间的路由与聚合。

因此，多文档关系更像“文档集合 + 每篇文档各自一棵树”，而不是“跨文档总树 + 统一树导航”。如果需要跨文档问答，上层系统仍需先完成文档选择，再进入单文档树导航。

## 上层 agent 的角色与能力边界

PageIndex 本地核心库不内建完整的 tree-search agent，也不直接保证 LLM 一定会按树逐层探索。它做的是提供一种**更容易被编排成树导航代理的检索协议**：

- `get_document_structure()` 只返回结构树，不返回全文文本
- `get_page_content()` 支持按紧页范围读取局部内容
- 调用方天然更容易走“先看结构，再读局部正文”的路径

因此，PageIndex 能较强地**引导**上层 agent 采用按树粗定位 → 局部精读 → 汇总回答的流程，但真正的搜索策略、候选节点筛选、多跳推理和最终回答生成，仍属于调用方或外部 agent 框架的职责。

## 它代表什么，不等于什么

PageIndex 能代表上述索引思路，**但不等于**该思路已在开源生态中形成成熟、丰富、可充分横向比较的对象簇。基于当前研究范围，几点观察：

- 在已完成源码核验的方向中，PageIndex 是 page-level / structure-aware indexing 这条路线上最完整的开源实现
- 类似的树索引方案（如 llama-index 的 tree summarization、Karpathy 模式中的 `index.md`）在机制上和工程深度上与 PageIndex 有本质差异
- 同一方向上目前还缺少足够多的可横向比较对象

因此，本目录把它作为**代表性对象入口**来研究——通过一个具体项目的深入研究来理解这个方向，而不是作为该方向的完整主题综述。

## 与 OpenKB / LLM Wiki 的边界

`pageindex/` 代码中无 OpenKB 引用、无 wiki 编译或多文档知识库管理能力。PageIndex 是完全独立的索引库。OpenKB 是把它用作长文档检索引擎的一条 LLM Wiki 系统实现路线，两者是系统—组件关系，不是同一层级的范式归属关系。

这一区分很重要：研究 PageIndex 时，应优先抓住它作为“文档结构恢复 + 树导航检索协议”的对象本体，而不是把它与上层知识系统实现混写成同一件事。

**证据等级**：`Verified`（PageIndex 独立性基于源码）、`Observed`（OpenKB 依赖关系基于官方 README 声明）

## 主要限制与未确定点

当前最值得明确保留的限制和未定点包括：

1. **本地核心库不包含完整推理循环**：示例层已有“先树定位，再局部取页”的 pattern，但未被封装为统一抽象。`Verified / Observed`
2. **多文档检索闭环未在本地 SDK 中成立**：workspace 支持多文档存储，但跨文档检索仍需调用方自建。`Verified`
3. **Cloud API 具体能力仍不可充分核验**：`submit_document`、`chat_completions` 等方法不在本地 SDK 中，服务端对多文档、增强 OCR、流式检索的真实支持范围仍需独立验证。`Unverified`
4. **PageIndex 之外的同思路对象仍然稀少**：当前尚难把它直接上升为一个已成熟、可充分横向比较的开源对象簇。`Inferred`

## Evidence

- Status: `Verified`（核心事实）、`Observed`（绩效数据与生态比较）、`Inferred`（抽象定位判断）
- Sources: 基准来源：`https://github.com/VectifyAI/PageIndex`；后续引用：`pageindex/page_index.py`、`pageindex/page_index_md.py`、`pageindex/client.py`、`pageindex/retrieve.py`；官方 GitHub README、官方 blog
- Version Basis: `branch main`, `commit 5a18553284ed`
- Observed At: `2026-06-13`
- Scope: 本页中所有源码结论仅适用于本次观察到的 `PageIndex` 默认分支当前截面
- Drift Risk: `medium`
- Trace: 基于本地源码核验（`pageindex/page_index.py`、`pageindex/client.py`、`pageindex/retrieve.py`、`pageindex/utils.py`、`pageindex/page_index_md.py`；示例层 `examples/agentic_vectorless_rag_demo.py`、`cookbook/agentic_retrieval.ipynb`）
- Needs: Cloud API 与本地 SDK 的能力差异需产品文档或代码级确认
