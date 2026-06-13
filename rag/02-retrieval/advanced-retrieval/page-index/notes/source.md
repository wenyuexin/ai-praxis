# Source Notes：PageIndex 源码核验

- Base Source: `https://github.com/VectifyAI/PageIndex`
- Version Basis: `branch main`, `commit 5a18553284ed`
- Observed At: `2026-06-13`
- Scope: 本笔记仅覆盖本地核验到的 Python 仓库、examples 和 cookbook 截面
- Drift Risk: `medium`

## 关键源码入口文件

所有路径相对于上游仓库根目录 `https://github.com/VectifyAI/PageIndex`。

| 文件 | 行数 | 核心职责 | 优先级 |
|---|---|---|---|
| `pageindex/page_index.py` | 1154 | PDF 索引构建 | P0 |
| `pageindex/page_index_md.py` | 342 | MD 索引构建 | P0 |
| `pageindex/client.py` | 235 | PageIndexClient 封装 | P1 |
| `pageindex/retrieve.py` | 138 | 检索工具函数 | P1 |
| `pageindex/utils.py` | 711 | 工具函数 | P1 |
| `pageindex/config.yaml` | 10 | 默认配置 | P2 |
| `pageindex/__init__.py` | 5 | 包导出 | P2 |
| `run_pageindex.py` | 134 | CLI 入口 | P2 |
| `examples/agentic_vectorless_rag_demo.py` | 189 | Agentic 检索 demo | P0（示例层核心） |
| `cookbook/agentic_retrieval.ipynb` | 900 | Cloud API 检索 notebook | P1（Cloud API 路径） |
| `cookbook/pageindex_RAG_simple.ipynb` | 610 | Cloud API 简单检索 | P2 |
| `cookbook/vision_RAG_pageindex.ipynb` | - | Vision RAG notebook | P3（未读） |

## 主要调用链

### PDF 索引
```
run_pageindex.py:run() → _index_pdf_or_md(path, opt) [run_pageindex.py]
  → ConfigLoader.load() [utils.py:670-685]（加载 config.yaml 默认值 + 覆盖）
  → page_index_main(path, opt) [run_pageindex.py:68]
    → page_index(doc, model, ...) [page_index.py: 核心入口]
      → get_pdf_title() [utils.py:230-234]
      → get_pdf_name() [utils.py:271-280]
      → get_page_tokens() [utils.py:387-410]（按页解析+token 计数）
      → 多轮 LLM 调用提取 TOC 和章节结构
      → check_title_appearance() [page_index.py:13-45]（LLM 验证每个章节标题的物理位置）
      → post_processing() [utils.py:433-452]（flat list → tree）
      → add_node_text() / add_node_text_with_labels() [utils.py:552-575]
      → (可选) generate_summaries_for_structure() [utils.py:589-596]（LLM 批量摘要）
      → format_structure() [utils.py:640-651]（字段排序和清理）
      → clean_structure_post() [utils.py:454-464]
```

### Markdown 索引
```
run_pageindex.py → asyncio.run(md_to_tree(path, ...)) [page_index_md.py:243-300]
  → extract_nodes_from_markdown() [page_index_md.py:32-59]（正则提取标题，跳过代码块）
  → extract_node_text_content() [page_index_md.py:62-87]（每个标题+内容区间）
  → 可选: update_node_list_with_text_token_count() + tree_thinning_for_index() [page_index_md.py:89-187]
  → build_tree_from_nodes() [page_index_md.py:190-221]（栈式层级树构建，与 PDF 索引共享）
  → 可选: generate_summaries_for_structure_md() [page_index_md.py:19-29]
```

### 检索（外部调用层）
```
LLM/Agent → get_document_structure(doc_id)  [client.py:224] → retrieve.py:100-107
  → 返回树结构 JSON（无文本）
LLM/Agent → get_page_content(doc_id, "5-7")  [client.py:230] → retrieve.py:110-137
  → 返回页面内容 JSON
```

## 重要函数 / 类 / 模块

| 名称 | 位置 | 说明 |
|---|---|---|
| `page_index()` | `page_index.py` | PDF 索引顶层入口，含完整 TOC 解析→结构验证→树构建→摘要 |
| `md_to_tree()` | `page_index_md.py:243` | MD 索引顶层入口 |
| `build_tree_from_nodes()` | `page_index_md.py:190` | 两个管线共享的树构建函数 |
| `PageIndexClient` | `client.py:28` | 高级客户端封装 |
| `ConfigLoader` | `utils.py:654` | YAML 配置加载器 |
| `_parse_pages()` | `retrieve.py:12` | 页面范围解析器 |
| `check_title_appearance()` | `page_index.py:13` | LLM 标题页验证 |
| `single_toc_item_index_fixer()` | `page_index.py:740` | 在局部页面窗口内重猜错误章节的起始物理页 |
| `fix_incorrect_toc()` | `page_index.py:760` | 用前后正确锚点裁剪窗口并修复错误 TOC 条目 |
| `fix_incorrect_toc_with_retries()` | `page_index.py:878` | 以最多 3 轮重试驱动局部错页修复 |
| `verify_toc()` | `page_index.py:900` | 验证标题是否真实落在预测页上 |
| `meta_processor()` | `page_index.py:959` | 串联 TOC 三路径、验证、修复和降级 |
| `get_page_tokens()` | `utils.py:387` | 按页 PDF 解析 + token 计数 |
| `remove_fields()` | `utils.py:466` | 树结构去文本 |
| `create_node_mapping()` | `utils.py:687` | node_id → node 映射 |
| `_normalize_retrieve_model()` | `client.py:18` | LiteLLM 路由前缀处理 |

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
2. **在 `client.py` 中搜索向量/embed 调用** → 0 结果。确认 vectorless 在源码层面成立。
3. **假设 `retrieve_model` 在核心库检索中使用** → 不成立。`client.py:48` 只存为属性，`client.py:220-234` 的检索函数不调用 LLM。
4. **假设 Cloud API 方法与本地 SDK 共享代码** → 不成立。`submit_document`、`chat_completions` 等不在本地任何文件中。
5. **假设 workspace 支持跨文档检索** → 部分不成立。支持多文档索引存储，但检索签名是单文档粒度。

## 待继续深挖

1. **`chat_completions()` 的实现位置** — 不在本地 `client.py` 或 `__init__.py` 导出中。可能存在于 pip 安装的单独版本中。
2. **Cloud API 端树搜索的具体实现** — Notebook 只展示了调用接口，服务端的树搜索算法和推理策略不可见。
3. **多文档检索在 OpenKB 侧的实现** — OpenKB 可能在应用层实现跨文档检索，而不依赖 PageIndex 索引层。这需要读 OpenKB 源码验证。
