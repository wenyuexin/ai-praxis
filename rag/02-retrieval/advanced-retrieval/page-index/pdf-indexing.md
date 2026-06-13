# PDF 树索引构建机制

## 核心流程

PageIndex 的 PDF 索引管线负责将长 PDF 文档转化为层级树结构。`pageindex/page_index.py`（1154 行）实现了完整的 LLM-driven 管线。

源码入口：`page_index.py` 中的 `page_index()` 函数，被 `client.py:55-130` 的 `PageIndexClient.index()` 和 `run_pageindex.py` 的 `page_index_main()` 调用。

完整调用链：

```
run_pageindex.py → page_index_main(path, opt) [run_pageindex.py]
  → page_index(doc, model, ...) [page_index.py]
    → get_pdf_title() [utils.py]
    → get_page_tokens() [utils.py]（按页解析 + token 计数）
    → LLM 调用提取 TOC 和章节结构（多轮）
    → check_title_appearance()（LLM 验证每个章节标题的物理位置）
    → post_processing() [utils.py]（flat list → tree）
    → add_node_text() / add_node_text_with_labels() [utils.py]
    → (可选) generate_summaries_for_structure() [utils.py]（LLM 批量摘要）
    → format_structure() / clean_structure_post() [utils.py]
```

## 关键子步骤

### 1. 页面解析与 token 计数

`pageindex/utils.py:387-410` 的 `get_page_tokens()` 是 PDF 文本抽取的单一入口，返回 `[(text, token_count), ...]` 列表。

它内部有两条**互斥**路径：

- `PyPDF2` 路径（默认）：逐页调用 `PyPDF2.PdfReader().pages[page_num].extract_text()`。
- `PyMuPDF` 路径：逐页调用 `pymupdf.open(...).load_page(...).get_text()`。

当前源码层没有自动回退、没有双路组合，也没有 OCR 管线；调用方如果不显式指定 `pdf_parser`，默认走 `PyPDF2`。`PyMuPDF` 路径额外支持 `BytesIO` 输入，因此在内存字节流场景下，它不是“更好”的备选，而是唯一可行的解析路径。

从工程边界看，PageIndex 对 PDF 特殊要素的策略更接近“**先把文本抽出来，再让 LLM 在文本层补结构**”，而不是“在解析层重建 PDF 版面语义”：

- 表格不会保留行列结构，只会被压成纯文本流。
- 图片和扫描页没有 OCR 补充路径。
- 页眉、页脚、脚注不会在解析阶段被专门过滤。
- TOC、页码和标题层级主要通过后续 LLM 推理恢复，而不是依赖 PDF 原生大纲或布局标签。

### 2. TOC 识别与章节结构提取

`page_index.py` 里的 TOC 路线不是“一条 prompt 直接出树”，而是一组串联的检测、提取、校验和降级步骤。

主干调用关系现在可以更明确地写成：

1. `check_toc()` 先调用 `find_toc_pages()`，在前 `toc_check_page_num` 页内逐页做 TOC 检测；如果已经进入 TOC 区，会继续往后扫直到 TOC 结束。
2. `toc_extractor()` 把识别出的 TOC 页拼接成一段 TOC 文本，并先用 `detect_page_index()` 判断其中是否真的包含页码。
3. 如果 TOC 自带页码，`tree_parser()` 进入 `meta_processor(mode='process_toc_with_page_numbers')`。
4. 如果 TOC 不带页码，`tree_parser()` **不会**直接进入 `process_toc_no_page_numbers`，而是直接落到 `process_no_toc`。
5. `process_toc_no_page_numbers` 仍然存在，但它主要作为 `meta_processor()` 在“有页码 TOC 路径验证不佳”时的**降级分支**，不是 `check_toc()` 返回“无页码 TOC”后的首选入口。

其中最重要的工程点不是“能识别 TOC”，而是**它把目录页码和 PDF 物理页码当成两个不同坐标系处理**。PageIndex 会先提取目录中的章节页码，再通过 `extract_matching_page_pairs()`、`calculate_page_offset()`、`add_page_offset_to_toc_json()` 这组后处理逻辑，把目录中的逻辑页码平移到真实物理页码范围。

`check_toc()` 还有一个之前没写清的细节：如果第一次识别到的 TOC 文本里没有页码，它会从当前 TOC 结束页之后继续尝试寻找下一段 TOC 页；只要在 `toc_check_page_num` 限制内找到了带页码的 TOC，就会返回那一段作为正式入口。只有在持续找不到带页码 TOC 时，它才返回最初那段“无页码 TOC”结果。

### 3. 三种结构提取路径

PageIndex 的 PDF 管线最终有三种结构提取函数，但它们的触发位置并不对称：

1. `process_toc_with_page_numbers()`：这是**首选路径**。前提是 `check_toc()` 找到了带页码的 TOC。
2. `process_toc_no_page_numbers()`：这是**降级路径**。它会先把无页码 TOC 转成章节树，再对分页文本分组，逐组补 `physical_index`。它主要由 `meta_processor()` 在“有页码 TOC 路径验证效果不够好”时触发。
3. `process_no_toc()`：这是**无 TOC / 无可用页码 TOC 时的直接路径**。`tree_parser()` 在没有可用带页码 TOC 时会直接走这里。

`meta_processor()` 还负责把这三条路径串成一个“先优先、再验证、再修正、最后降级”的流程：

- 先执行当前模式对应的结构提取函数。
- 过滤 `physical_index is None` 的条目，并用 `validate_and_truncate_physical_indices()` 去掉越界页码。
- 用 `verify_toc()` 抽样或全量检查“标题是否确实出现在对应页”。
- 如果准确率为 `1.0`，直接接受结果。
- 如果准确率 `> 0.6` 且存在错误条目，进入 `fix_incorrect_toc_with_retries()` 做局部修正。
- 如果准确率仍然太低，则按 `process_toc_with_page_numbers → process_toc_no_page_numbers → process_no_toc` 的顺序继续降级。

### 4. 无 TOC 文档的处理路径

无 TOC 不是异常分支，而是 PDF 管线中的一条正式路径。`process_no_toc` 的基本思路是：**先按 token 预算把页面分组，再让 LLM 分批从零生长出结构**。

其数据流可以概括为：

```text
page_list
  → page_list_to_group_text()      # 按 token 预算分组
  → generate_toc_init()            # 第一批页面生成初始结构
  → generate_toc_continue()        # 后续批次增量补全结构
  → convert_physical_index_to_int()
```

这意味着 PageIndex 并不要求 PDF 先天存在目录页；即使是实验报告、手册片段或无正式目录的长文档，也可以通过 LLM 对分页文本的增量阅读构建出一套章节骨架。与之前的模糊判断不同，现在可以更明确地说：**无页码 TOC 在顶层调度上也会落到这条路径，而不是直接把 `process_toc_no_page_numbers()` 作为首选主路由。**
### 5. 双重标题校验：是否出现 vs 是否从页首开始

当前 PDF 管线里有两条用途不同的标题校验链路，不应混成一个函数理解：

- `check_title_appearance()`：检查“某个标题是否出现在该物理页上”，主要用于 TOC 修正与局部核验。
- `check_title_appearance_in_start()` / `check_title_appearance_in_start_concurrent()`：检查“该标题是否从这一页开头开始”，主要用于后续页范围切分。

```python
async def check_title_appearance(item, page_list, start_index=1, model=None):
    title = item['title']
    page_number = item['physical_index']
    page_text = page_list[page_number-start_index][0]
    prompt = f"""...Your job is to check if the given section appears or starts in the given page_text..."""
    # 返回 {'answer': 'yes'/'no', ...}
```

这两条链路的差别很关键：前者只解决“目录抽出来的标题是否落在正确页面”，后者则决定“下一章是否从页首开始，从而当前章节应不应该在上一页结束”。后者直接参与树节点的 `end_index` 计算，所以它不是附属校验，而是页范围切分精度的一部分。

### 6. flat list → tree 的数据流

TOC 路线和无 TOC 路线在中段都会收敛到一种扁平结构，大致形态如下：

```text
{structure, title, physical_index, appear_start}
```

`utils.py:433-452` 的 `post_processing()` 会基于这组中间字段做两件事：

1. 计算每个条目的 `start_index` / `end_index`。
2. 调用树化逻辑，把点分层级编号（如 `1`、`1.1`、`1.1.1`）转成真正的父子树。

`appear_start` 在这里起的是切分页边界的作用：

- 如果下一章节在新页页首开始，当前章节的 `end_index = next.physical_index - 1`。
- 如果下一章节只是出现在同一页后半段，当前章节允许与下一章共享该页。

因此，“标题在页首开始吗”并不是可有可无的附加信息，而是扁平 TOC list 变成可检索树结构时的关键分界条件。

### 7. 局部错页修复

`meta_processor()` 在验证阶段不会只给出一个“准不准”的整体分数；如果 `verify_toc()` 发现部分标题没有真正落在预测页上，而且整体准确率仍高于阈值，PageIndex 会进入一套**局部修复**流程，而不是整棵树推倒重来。

这套流程由 `fix_incorrect_toc_with_retries()` 和 `fix_incorrect_toc()` 驱动，核心思路是：

1. 先找出验证失败的 TOC 条目。
2. 对每个错误条目，向前回看最近一个“已知正确”的章节页码，向后回看下一个“已知正确”的章节页码。
3. 只在这两个锚点之间切一段局部页面窗口，而不是重新把整份 PDF 喂给 LLM。
4. 用 `single_toc_item_index_fixer()` 让 LLM 在这段局部窗口里重新判断该标题真正开始的 `physical_index`。
5. 再次调用 `check_title_appearance()` 复核新页码；只有复核通过，才把新的 `physical_index` 写回 TOC。
6. 如果仍有失败项，最多重试 3 轮；再不行才接受当前结果或继续走更低一级的降级路径。

这个修复设计有两个值得单独记住的特点：

- 它是**局部窗口修复**，不是全局重跑，因此成本和误伤范围都更小。
- 它把“LLM 负责重新猜页码”和“LLM 负责验证猜测是否落页”拆成了两个步骤，减少了一次调用里既生成又自证的风险。

### 8. 大节点递归细化

`process_large_node_recursively()` 会在初始树构建完成后，继续检查是否有章节过大。如果同一节点同时超过页数阈值和 token 阈值，管线会在该节点内部再次调用结构提取逻辑，为大章节补出更细的子节点。

这让 PageIndex 不必完全依赖原始目录粒度：即使 TOC 只给出一个非常粗的 30 页大章，后处理也可能在该章节内部继续长出更细的层级。

### 9. 摘要生成

`utils.py:578-596` 的 `generate_summaries_for_structure()` 使用 `asyncio.gather` 并行为每个树节点调用 LLM 生成摘要。这在索引构建阶段预计算了每个节点的摘要信息，使得后续检索时 LLM 可以直接读摘要而非全文，大幅节省 token。

## 控制参数

`config.yaml` 和运行时参数共同控制索引管线的关键行为：

- `toc_check_page_num: 20` — TOC 检测最多扫描的前置页数。
- `max_page_num_each_node: 10` — 大节点递归细化的页数阈值之一。
- `max_token_num_each_node: 20000` — 大节点递归细化的 token 阈值之一。
- `model: "gpt-4o-2024-11-20"` — 索引阶段的主要 LLM。
- `if_add_node_summary: "yes"` — 是否为节点预生成摘要。

这些参数在 `utils.py:654-685` 的 `ConfigLoader` 类中被加载，支持 `config.yaml` 默认值 + 运行时参数覆盖。

其中两个容易漏掉的关系是：

- `max_page_num_each_node` 与 `max_token_num_each_node` 不是二选一触发，而是共同决定“大节点是否需要递归细化”的条件。
- `if_add_node_summary` 与 `if_add_node_text` 之间存在实现层耦合：即使最终不想在索引产物中保留节点文本，管线也可能先临时补文本以支持摘要生成，再在后处理中收敛成目标输出。

## 与 Markdown 索引的差异

| 维度 | PDF 索引 | Markdown 索引 |
|---|---|---|
| 核心源码 | `page_index.py`（1154 行） | `page_index_md.py`（342 行） |
| 结构提取方式 | LLM 驱动（多轮调用） | 正则表达式（无 LLM） |
| 标题页验证 | LLM 验证标题是否出现在页面 | 不再需要（行号天然对应） |
| 章节映射 | 需要建立标题→页码映射 | 标题行号即位置 |
| 主依赖 | PyPDF2 / PyMuPDF | 纯文本处理 |
| 索引复杂度 | 高 | 低 |

## Evidence

- Status: `Verified`（`get_page_tokens()` 的双路径文本抽取、无 OCR、`check_toc()` 的 TOC 检测与再搜索逻辑、三种结构提取路径的真实触发关系、`meta_processor()` 的验证/修正/降级顺序、局部错页修复链路、双重标题校验、`post_processing()` 的树化与页范围切分、递归大节点细化）
- Sources: 基准来源：`https://github.com/VectifyAI/PageIndex`；后续引用：`pageindex/page_index.py`、`pageindex/page_index_md.py`、`pageindex/utils.py`、`pageindex/config.yaml`；研究输入：`temp/chat/round13-feedback.md`
- Version Basis: `branch main`, `commit 5a18553284ed`
- Observed At: `2026-06-13`
- Scope: 本文中的 PDF/Markdown 索引实现判断仅适用于本次观察到的本地 Python 仓库截面
- Drift Risk: `medium`
- Trace: 第一轮核心库核验 + 第四轮 PDF 管线补读（`temp/chat/round13-feedback.md`）+ 本轮直接补读 `pageindex/page_index.py` 中段源码
- Needs: 若后续要继续提升研究价值，可再单独整理 `fix_incorrect_toc()` / `single_toc_item_index_fixer()` 的 prompt 设计模式，但当前主路由已经足够闭环
