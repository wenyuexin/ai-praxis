# 检索协议、树导航模式与示例层 Reasoning Pattern

## 检索工具层

`pageindex/retrieve.py`（138 行）和 `PageIndexClient`（`client.py:220-234`）提供了三个原子检索操作：

| 函数 | 参数 | 返回 | Token 消耗 |
|---|---|---|---|
| `get_document(doc_id)` | doc_id | 元数据 JSON（doc_name, page_count, status） | 无（从已加载的 JSON 读取） |
| `get_document_structure(doc_id)` | doc_id | 树结构 JSON（不含文本字段） | 无（从已加载的 JSON 读取） |
| `get_page_content(doc_id, pages)` | doc_id + 页码范围 "5-7,10,12" | 页面内容 JSON | 无（从已缓存的 pages 数据读取） |

**这三个函数都不调用 LLM**，只操作本地 workspace 中缓存的 JSON 文件。`client.py:226-228` 的 `get_document_structure` 在 `retrieve.py:100-107` 中通过 `remove_fields(structure, fields=['text'])` 移除文本字段以节约传输成本。

## 页面范围解析

`retrieve.py:12-24` 的 `_parse_pages()` 实现了一个页面范围字符串解析器：`"5-7,10,12-15"` → `[5,6,7,10,12,13,14,15]`。这个解析器同时支持 PDF（页码）和 Markdown（行号）的页面概念。

对于 PDF，`_get_pdf_page_content()`（`retrieve.py:36-53`）优先使用索引时预缓存的 pages 数据，否则从原始 PDF 中实时提取。

对于 Markdown，`_get_md_page_content()`（`retrieve.py:56-76`）将页码范围映射为树节点的行号范围，遍历树结构提取匹配节点的文本内容。

## 检索模式：两阶段树导航

核心库本身不包含完整的树导航推理逻辑。它提供的两个阶段是：

```
阶段 1: get_document_structure(doc_id)  → 获取树结构（无文本）
阶段 2: get_page_content(doc_id, "5-7") → 按推理结果获取页面内容
```

阶段 1 和阶段 2 之间的"LLM 根据树结构推理应读取哪些页"这一关键步骤，不在核心库中，由外部调用层负责（示例 demo 或 Agent SDK）。

## 示例层 Reasoning Pattern

核心库外有两个独立实现的推理检索模式：

### 路径 A：OpenAI Agents SDK（`examples/agentic_vectorless_rag_demo.py`）

将三个检索函数包装为 Agent tool，由 Agents SDK 的 agent 循环驱动 LLM 自主决策。关键控制信号是 `AGENT_SYSTEM_PROMPT`（`ex:44-52`）：

```
- Call get_document_structure() to identify relevant page ranges.
- Call get_page_content(pages="5-7") with tight ranges; never fetch the whole document.
```

Agent 使用 `model=client.retrieve_model`（`ex:85`），这是仓库中 `retrieve_model` 的唯一定义消费点。

### 路径 B：手动 Prompt 树搜索（`cookbook/agentic_retrieval.ipynb`）

使用 PageIndex Cloud API，推理由手动 prompt 实现（`notebook:359-377`）：

```python
search_prompt = f"""
You are given a question and a tree structure of a document.
Each node contains a node id, node title, and a corresponding summary.
Your task is to find all nodes that are likely to contain the answer to the question.
Please reply in the following JSON format:
{"thinking": "...", "node_list": ["node_id_1", "node_id_2", ...]}
"""
tree_search_result = await call_llm(search_prompt)
```

### 两条路径的差异

| 维度 | 路径 A：Agents SDK | 路径 B：手动 Prompt |
|---|---|---|
| 依赖框架 | OpenAI Agents SDK | 手动构造 LLM 调用 |
| 推理循环 | SDK 内置 agent 循环 | 手动实现 |
| 使用的客户端 | 本地 `PageIndexClient` | Cloud API（`submit_document` + `chat_completions`，见 `sdk-and-workspace.md` 能力对照） |
| 检索模型 | `retrieve_model`（`config.yaml`，被 Agent SDK 消费，不参与核心库检索调用） | Notebook 内手动指定 |
| 封装程度 | 函数级工具包装 | 单次 prompt 调用 |

两条路径实现了同一核心模式（获取树结构 → LLM 推理定位 → 精确获取页面内容），但不共享代码，不存在独立的封装抽象。

> **当前没有将示例层推理模式单独拆成专题。** 内容体量（约 40 行）还不足以独立成文，且该模式的本质是检索协议的两条调用方实现路径，不是独立的机制专题。如果后续出现第三条实现路径，或该模式被核心库吸收封装，再评估拆分。

## 当前最稳的判断

- 核心库提供的是 index + get_document_structure + get_page_content 三个原子能力，不是端到端推理系统。`Verified`
- 树结构去文本（remove_fields）是一个省 token 的关键设计点——让 LLM 在不暴露完整文本的前提下推理定位。`Verified`
- 示例层已经形成了概念模式但未封装——"TreeNavigationAgent"可作为机制专题记录但不等于核心库内建能力。`Observed`

## Evidence

- Status: `Verified`（核心库检索接口不调用 LLM）、`Verified`（三个检索函数签名和实现）、`Observed`（示例层 reasoning pattern 存在但未封装）
- Sources: 基准来源：`https://github.com/VectifyAI/PageIndex`；后续引用：`pageindex/retrieve.py`、`pageindex/client.py`、`examples/agentic_vectorless_rag_demo.py`、`cookbook/agentic_retrieval.ipynb`
- Version Basis: `branch main`, `commit 5a18553284ed`
- Observed At: `2026-06-13`
- Scope: 核心库检索协议结论适用于本地 Python 仓库；示例层 reasoning pattern 结论适用于本次观察到的示例与 notebook
- Drift Risk: `medium`
- Trace: 源码研究第二轮（`temp/chat/round9-feedback.md`）补读了示例层和 notebook
- Needs: Cloud API 端的检索实现不可见，无法确认服务端推理模式与本地模式的差异
