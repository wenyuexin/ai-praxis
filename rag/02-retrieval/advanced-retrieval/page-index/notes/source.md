# Source Notes：PageIndex 源码核验

- Base Source: `https://github.com/VectifyAI/PageIndex`
- Version Basis: 复核截面 `branch main`, `commit ae2a5b4`（2026-08-17）；原研究截面 `commit 5a18553284ed`（2026-06-13）
- Observed At: `2026-08-19`（复核，本地克隆 `/Volumes/ZGY93B6F/github/PageIndex`）；原截面 `2026-06-13`
- Scope: 本笔记覆盖复核截面的 Python 仓库、examples 和 cookbook
- Drift Risk: `medium`

## 关键源码入口文件

所有路径相对于上游仓库根目录 `https://github.com/VectifyAI/PageIndex`。

| 文件 | 行数 | 核心职责 | 优先级 |
|---|---|---|---|
| `pageindex/page_index_classic.py` | 1323 | PDF 索引构建（原 `page_index.py`） | P0 |
| `pageindex/flash/` | — | flash 快速索引管线（pypdfium2，本地默认模式） | P0 |
| `pageindex/page_index_md.py` | 344 | MD 索引构建（仅 CLI 接入） | P0 |
| `pageindex/client.py` | 1171 | PageIndexClient 本地/云双模式封装 | P1 |
| `pageindex/local_api.py` | 370 | LocalAPI 本地索引与检索 | P1 |
| `pageindex/local_store.py` | 164 | DocStore 文档持久化 | P1 |
| `pageindex/cloud_api.py` | 435 | CloudAPI 服务端接口客户端 | P1 |
| `pageindex/agent_tools.py` | 1633 | agent 工具契约与指令 | P1 |
| `pageindex/utils.py` | 1104 | 工具函数 | P1 |
| `pageindex/config.yaml` | 13 | 默认配置 | P2 |
| `run_pageindex.py` | — | CLI 入口（PDF + MD） | P2 |
| `examples/agentic_vectorless_rag_demo.py` | 169 | Agentic 检索 demo（Agents SDK） | P0（示例层核心） |
| `cookbook/agentic_retrieval.ipynb` | — | Cloud API 检索 notebook | P1（Cloud API 路径） |
| `cookbook/pageindex_RAG_simple.ipynb` | — | Cloud API 简单检索 | P2 |
| `cookbook/vision_RAG_pageindex.ipynb` | — | Vision RAG notebook | P3（未读） |

## 主要调用链

### PDF 索引
```
run_pageindex.py:run() → page_index_main(path, opt) [run_pageindex.py]
  → ConfigLoader.load() [utils.py:1029]（加载 config.yaml 默认值 + 覆盖）
  → page_index_main(path, opt) [page_index_classic.py]
    → page_index(doc, model, ...) [page_index_classic.py: 核心入口]
      → get_pdf_title() [utils.py]
      → get_pdf_name() [utils.py]
      → get_page_tokens() [utils.py:501]（按页解析+token 计数）
      → 多轮 LLM 调用提取 TOC 和章节结构
      → check_title_appearance() [page_index_classic.py]（LLM 验证每个章节标题的物理位置）
      → post_processing() [utils.py:553]（flat list → tree）
      → add_node_text() / add_node_text_with_labels() [utils.py]
      → (可选) generate_summaries_for_structure() [utils.py:711]（LLM 批量摘要）
      → format_structure() [utils.py]（字段排序和清理）
      → clean_structure_post() [utils.py]
```

### Markdown 索引
```
run_pageindex.py --md_path → md_to_tree(path, ...) [page_index_md.py:245]
  → extract_nodes_from_markdown() [page_index_md.py]（正则提取标题，跳过代码块）
  → extract_node_text_content() [page_index_md.py]（每个标题+内容区间）
  → 可选: update_node_list_with_text_token_count() + tree_thinning_for_index() [page_index_md.py]
  → build_tree_from_nodes() [page_index_md.py:192]（栈式层级树构建）
  → 可选: generate_summaries_for_structure_md() [page_index_md.py]
```

### 检索（外部调用层）
```
LLM/Agent → get_document_structure(doc_id)  [client.py:343]
  → 返回树结构 JSON（无文本）
LLM/Agent → get_page_content(doc_id, "5-7")  [client.py:297]
  → 返回页面内容 JSON
LLM/Agent → get_ocr(doc_id, format)          [client.py:278 / local_api.py:249]
  → 返回页/节点/拼接文本（format: page / node / raw）
```

## 重要函数 / 类 / 模块

| 名称 | 位置 | 说明 |
|---|---|---|
| `page_index_main()` / `page_index()` | `page_index_classic.py` | PDF 索引顶层入口，含完整 TOC 解析→结构验证→树构建→摘要 |
| `md_to_tree()` | `page_index_md.py:245` | MD 索引顶层入口 |
| `build_tree_from_nodes()` | `page_index_md.py:192` | 树构建函数（与 PDF 路径共享输出形态） |
| `PageIndexClient` / `PageIndexLocalClient` / `PageIndexCloudClient` | `client.py:54/1153/1141` | 客户端封装（双模式） |
| `LocalAPI` / `DocStore` / `CloudAPI` | `local_api.py:34` / `local_store.py:29` / `cloud_api.py:15` | 本地 API / 存储 / 云客户端 |
| `ConfigLoader` | `utils.py:1029` | YAML 配置加载器 |
| `_parse_pages()` | `client.py:20` | 页面范围解析器 |
| `check_title_appearance()` | `page_index_classic.py` | LLM 标题页验证 |
| `fix_incorrect_toc_with_retries()` / `meta_processor()` 等 | `page_index_classic.py` | TOC 验证/修复/降级链 |
| `get_page_tokens()` | `utils.py:501` | 按页 PDF 解析 + token 计数 |
| `browse_documents` / `call_tool` | `agent_tools.py:664/1140` | 文档级列举 / 工具路由 |

## Prompt 设计模式

### TOC 局部错页修复

`single_toc_item_index_fixer()`、`fix_incorrect_toc()`、`verify_toc()` 这一组函数体现了一个很稳定的 prompt 设计模式：

1. **先缩小搜索窗口**：不是让模型在整份文档里重新找标题，而是用前后已知正确章节做锚点，只保留局部页面范围。
2. **先生成候选页码，再单独验证**：`single_toc_item_index_fixer()` 负责找页，`check_title_appearance()` 负责验页，避免一次调用里既生成又自证。
3. **把物理页码显式编码进文本**：局部窗口里的每页都包上 `<physical_index_X>` 标签，让模型输出页码时不需要额外做版面推理。
4. **只修失败条目**：`fix_incorrect_toc()` 只更新验证失败的章节，不回写其他已经通过验证的条目。
5. **允许有限重试，不无限自循环**：`fix_incorrect_toc_with_retries()` 最多重试 3 次，说明它把 LLM 修复视为局部补救手段，而不是无上限搜索过程。

这个模式对理解 PageIndex 很重要：它不是“LLM 一次性抽出完美 TOC”，而是“先粗抽，再用局部验证与局部修复把结构拉回可用精度”。

## 失败搜索或不成立假设

1. **在 `pageindex/` 中搜索 OpenKB 引用** → 0 结果。确认 PageIndex 核心库完全独立。
2. **在 `client.py` / `local_api.py` / `cloud_api.py` 中搜索向量/embed 调用** → 0 结果。确认 vectorless 在源码层面成立。
3. **假设 `retrieve_model` 在核心库检索中使用** → 不成立。检索函数不调用 LLM；示例 demo 重构后也不再消费 `retrieve_model`。
4. **假设 Cloud API 方法不在本地仓库** → 旧截面成立，复核截面**不成立**。`submit_document`、`chat_completions`、`submit_query` 等已随 `cloud_api.py` 进入 SDK（cloud 模式），服务端实现仍不可见。
5. **假设 workspace 支持跨文档检索** → 部分不成立。支持多文档索引存储（`DocStore`），但检索签名是单文档粒度。
6. **假设本地客户端支持 Markdown 索引** → 复核截面**不成立**。`submit_document` 仅接受 PDF；MD 只经 CLI。
7. **假设 "PageIndex File System" 有开源实现** → 不成立。全仓 grep 仅 README 一处博客链接，该能力为 Enterprise / Cloud 特性（`Observed`）。

## 待继续深挖

1. **Cloud API 端树搜索的具体实现** — `cloud_api.py` 只暴露调用面（`submit_query` / `get_retrieval` / `chat_completions`），服务端的树搜索算法和推理策略不可见。
2. **File System cloud 版可用性** — 官方 blog（2026-07）称 "Cloud rollout coming soon"，截至 2026-08-19 无独立证据确认是否上线。
3. **本地默认 flash 管线的机制** — `submit_document` 默认 `mode="flash"`（`local_api.py:85-88`），但文档机制研究集中在 classic 管线（`page_index_classic.py`）；`flash/`（embedded_toc / outline assembly / classification 等）与 `tree_optimize.py`（947 行）机制未研究。
4. **多文档检索在 OpenKB 侧的实现** — OpenKB 可能在应用层实现跨文档检索，而不依赖 PageIndex 索引层。这需要读 OpenKB 源码验证。
5. **生态观察** — 第三方文章提及 VectifyAI 另有 ChatIndex（树索引思路用于对话历史）与 TypeScript SDK（`pageindex-js-sdk`），均未核验（第三方转述，`Unverified`）。
