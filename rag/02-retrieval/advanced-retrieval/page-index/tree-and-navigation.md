# PageIndex 的树结构与导航机制

`PageIndex` 的核心不只是把长文档切成可检索片段，而是先把文档恢复为可导航的层级结构，再让 agent 或调用方基于这棵树逐步定位正文。对 `PageIndex` 的深入理解，不能只停留在“它支持树导航”这类高层描述，还需要进一步回答：这里的树到底是什么，两条构树路径如何收敛，以及这棵树在本地检索链里究竟扮演什么角色。

## 1. `PageIndex` 的树到底是什么

从源码看，`PageIndex` 的统一树抽象不是某个 PDF 专属数据结构，而是一个共享骨架明确、路径特有载荷不同的层级章节树。

这棵树的共享骨架至少包括：

- `title`：节点标题
- `nodes`：子节点列表

在这个基础上，不同输入路径会携带不同的节点信息：

- PDF 路径最终更接近“章节树 + 页范围”，核心定位字段是 `start_index` 和 `end_index`
- Markdown 路径最终更接近“章节树 + 原始文本块”，核心内容字段是 `text` 和 `line_num`

因此，`PageIndex` 的树不是单一静态 schema，而是一个**共享树骨架 + 路径特有载荷**的树模型。`summary`、`prefix_summary`、`node_id` 等字段可以附加在节点上，但它们更像索引增强信息，而不是树存在的前提。

## 2. 两条构树路径如何收敛到同一抽象

### 2.1 Markdown：直接从标题层级构树

Markdown 路径最清楚地暴露了 `PageIndex` 的标准树节点形态。`build_tree_from_nodes()` 依赖标题级别 `level`，按栈式层级归并的方式构树：当遇到同级或更高层标题时，就持续出栈，再把当前节点挂到最近的更高层父节点下。

最终得到的节点至少包含：

- `title`
- `node_id`
- `text`
- `line_num`
- `nodes`

这说明在 Markdown 路径里，树首先是一个章节层级结构，其次才是与正文文本和行号绑定的索引结果。

### 2.2 PDF：先恢复章节条目，再树化

PDF 路径不直接从天然层级结构出发，而是先从分页文本里恢复一组扁平章节条目，再把这些条目收敛成树。

树化前的关键条目字段包括：

- `structure`
- `title`
- `physical_index`
- `appear_start`

随后，`post_processing()` 会补出章节的 `start_index` / `end_index`，再由 `list_to_tree()` 按 `structure` 编号建立父子关系。例如：

- `1` 是顶层节点
- `1.2` 会挂到 `1` 下
- `1.2.1` 会挂到 `1.2` 下

因此，PDF 路径与 Markdown 路径共享的不是同一个建树算法，而是同一个最终树形输出：**带 `nodes` 的层级章节树**。

## 3. PDF 树为什么不是一次性生成

PDF 树的一个关键特点是：它不是“一次生成完毕”的静态结果，而是一个逐步逼近的结构恢复过程。

`tree_parser()` 的主链大致如下：

1. 检查是否存在可用 TOC
2. 由 `meta_processor()` 生成或修正扁平章节条目
3. 通过标题页首校验进一步清理条目
4. 用 `post_processing()` 补齐页范围并树化
5. 对超大节点进入 `process_large_node_recursively()` 做局部再次细分

这条链意味着：PDF 树的第一版结构只是初始树，超大章节还可能在其内部再次生长子树。

### 3.1 递归细化不会引入第二套树 schema

`process_large_node_recursively()` 不会产生一套与初始 PDF 树完全不同的数据结构。它对子节点重新调用 `meta_processor(..., mode='process_no_toc')`，之后仍然会经过标题页首校验和 `post_processing()`，因此新长出来的子树在输出字段上仍然是：

- `title`
- `start_index`
- `end_index`
- `nodes`

也就是说，递归细化节点与初始 PDF 节点在结构骨架上是基本同构的。

### 3.2 递归细化节点又不与初始节点完全等价

虽然结构骨架同构，但递归细化节点和初始 PDF 节点在三个方面并不完全等价：

- **来源不同**：递归细化节点不是从全局 TOC 或文档主干章节恢复出来，而是从父节点页范围内部重新走一遍无 TOC 生长路径
- **粒度不同**：它们天然是局部再切分结果，而不是文档的第一层主干章节
- **父节点边界会被回写调整**：如果细化后首个子节点与父标题重复，系统会跳过该重复节点，并用后续子节点的 `start_index` 回写父节点边界；否则直接用第一个子节点回写

因此，更稳的理解是：PDF 路径中的树既有统一骨架，也有“初始恢复节点”和“局部再生长节点”这类来源与粒度差异。

## 4. 树在本地检索链里扮演什么角色

如果只看本地可见 SDK，`PageIndex` 并没有暴露一个显式名为 search 或 navigate 的独立树上搜索器。可直接看到的工具接口只有三类：

- `get_document()`：取文档元信息
- `get_document_structure()`：取无正文文本的树结构
- `get_page_content()`：取指定页范围或 Markdown 节点对应文本

这说明在本地实现中，树首先承担的是**结构定位与导航**作用，而不是一个直接返回答案的检索器。

一个更准确的本地行为链是：

1. 先看文档元信息，确认状态与页数
2. 再看树结构，识别相关章节范围
3. 再按页码或行号提取正文内容
4. 最后由 agent 或调用方基于正文完成回答

`get_document_structure()` 会显式去掉 `text` 字段，只返回树骨架；这进一步说明它的主要用途是让 agent 或用户先看结构，而不是直接拿树节点正文作答。

## 5. 搜索和导航在这里如何区分

在 `PageIndex` 的本地可见实现里，“搜索”和“导航”不太像两个并列的独立模块，而更像同一行为链中的两个阶段：

- **搜索** 更接近基于树结构找到相关章节或页范围
- **导航** 更接近沿结构决定下一步该看哪一段正文

也就是说，本地实现里更像是：

- 树负责定位范围
- 页内容负责支撑答案
- agent 负责把“先看结构、再看正文”的过程串起来

因此，把 `PageIndex` 简单描述成“树搜索器”并不准确。更稳的说法是：**它提供了树结构索引，供 agent 先做结构导航与范围定位，再取页内容完成回答。**

## 6. 这棵树的边界与限制

这条树主线并不意味着 `PageIndex` 已经把所有输入都稳定映射成高质量结构。尤其在 PDF 路径中，树质量仍然高度依赖结构恢复质量。

当前可以较稳确认的限制包括：

- PDF 路径不是 OCR-first，文本抽取质量会直接影响后续结构判断
- TOC 错误、页码不准、标题不规范都会影响树恢复
- 系统虽然有无 TOC 路径、局部修复和递归细化机制，但这些都是在弥补输入与结构之间的不稳定关系

同时，当前这篇文只覆盖了本地可证实的树结构与导航机制。Cloud API、notebook、外围生态是否暴露了更强的树搜索封装，仍需单独核实，不能从本地 SDK 的行为直接外推。

## 7. 总结

`PageIndex` 的核心不是“把文档切块后做搜索”，而是**先把文档恢复为可导航的层级结构，再让调用方沿结构定位正文**。

Markdown 路径直接从标题层级得到这棵树，PDF 路径则先从分页文本恢复扁平章节条目，再逐步收敛成树，并在必要时对超大节点继续细化。到了本地检索链里，这棵树主要承担的是结构定位与导航职责，而真正支撑回答的仍是页内容文本。

---

## Evidence

- Claim type: Source-based implementation summary
- Status: Observed
- Sources:
  - `pageindex/page_index_md.py:190`
  - `pageindex/utils.py:324`
  - `pageindex/utils.py:433`
  - `pageindex/utils.py:132`
  - `pageindex/page_index.py:1000`
  - `pageindex/page_index.py:1029`
  - `pageindex/retrieve.py:100`
  - `pageindex/retrieve.py:110`
  - `examples/agentic_vectorless_rag_demo.py:44`
- Trace:
  - Consolidated from `notes/deep-research-question-tree.md` after three rounds of source verification focused on tree schema, construction paths, and local retrieval behavior.
- Version Basis: branch `main`, local checkout observed at `/Users/wenyuexin/github/PageIndex`
- Observed At: 2026-06-14
- Scope: local source paths for Markdown tree construction, PDF tree construction, recursive refinement, and local SDK retrieval behavior
- Drift Risk: medium — tree schema is relatively stable, but retrieval behavior and interface exposure may evolve across SDK / service layers
