# 检索协议、树导航模式与示例层 Reasoning Pattern

## 检索工具层

检索工具层由 `client.py` + `local_api.py` + `agent_tools.py` 组成（原 `retrieve.py` 已在新版 SDK 中移除）。核心原子操作：

| 函数 | 参数 | 返回 | Token 消耗 |
|---|---|---|---|
| `get_document(doc_id)` | doc_id | 元数据 JSON（doc_name, page_count, status） | 无（从 DocStore JSON 读取） |
| `get_tree(doc_id, node_summary, include_text)` | doc_id | 树结构 JSON（可含/不含文本与摘要） | 无（从 DocStore JSON 读取） |
| `get_document_structure(doc_id)` | doc_id | 树结构 JSON（不含文本字段） | 无（从 DocStore JSON 读取） |
| `get_ocr(doc_id, format)` | doc_id + `"page" / "node" / "raw"` | 页/节点/拼接文本 | 无（从 DocStore JSON 读取） |
| `get_page_content(doc_id, pages)` | doc_id + 页码范围 `"5-7,10,12"` | 页面内容 JSON | 无（从 DocStore JSON 读取） |

实现位置：`client.py:732`（get_document）、`client.py:320`（get_tree）、`client.py:343`（get_document_structure）、`client.py:278`（get_ocr，路由到 `local_api.py:249`）、`client.py:297`（get_page_content，基于 get_ocr 过滤）。

**这些检索函数都不调用 LLM**，只操作本地 DocStore 中缓存的 JSON 文件。`get_document_structure()` 通过 `get_tree(doc_id, node_summary=True, include_text=False)` 显式去掉文本字段，只返回树骨架以节约 token。

文档级管理工具由 `agent_tools.py` 提供：`browse_documents`（`agent_tools.py:664`，列举 + 分页，仅时间排序；`folder_id` 与语义排序在本地模式直接报错）、`get_document` / `get_document_structure` / `get_page_content`（按 doc_name 解析）等，并支持 `doc_ids` allowlist 限定作用域（`agent_tools.py:1140-1174`）。

## 页面范围解析

`client.py:20` 的 `_parse_pages()` 实现页面范围字符串解析器：`"5-7,10,12-15"` → `[5,6,7,10,12,13,14,15]`。`get_page_content` 用它展开页码范围，再从 `get_ocr(format="page")` 结果中过滤目标页（`client.py:297-311`）。

## 检索模式：两阶段树导航

核心库本身不包含完整的树导航推理逻辑。它提供的两个阶段是：

```
阶段 1: get_document_structure(doc_id)  → 获取树结构（无文本）
阶段 2: get_page_content(doc_id, "5-7") → 按推理结果获取页面内容
```

阶段 1 和阶段 2 之间的"LLM 根据树结构推理应读取哪些页"这一关键步骤，不在核心库中，由外部调用层负责（示例 demo 或 Agent SDK）。

## 示例层 Reasoning Pattern

核心库外有两个独立实现的推理检索模式：

### 路径 A：OpenAI Agents SDK（`examples/agentic_vectorless_rag_demo.py`，169 行）

新版 demo 直接使用 `PageIndexLocalClient`（`client.py:1153`，本地模式）或 `PageIndexCloudClient`（`client.py:1141`，cloud 模式）组装 agent：

- 工具契约：`browse_documents` / `get_document` / `get_document_structure` / `get_page_content`（`examples/agentic_vectorless_rag_demo.py:10-11`）
- 检索 playbook：`client.agent_instructions()` 提供（同一行），agent 通过 `Agent(**client.openai_agent_config(doc_id=doc_id))` 自动获得工具与指令（`ex:52`）
- 流程：索引 PDF → `get_tree` 查看树结构（`ex:153`）→ `get_document` 查看元数据（`ex:160`）→ agent 提问，由 Agents SDK 驱动工具调用
- 旧的 `AGENT_SYSTEM_PROMPT` 与 `retrieve_model` 消费点已在重构中移除

### 路径 B：手动 Prompt 检索（`cookbook/agentic_retrieval.ipynb`）

使用 Cloud 模式客户端：`submit_document(pdf_url)` 提交文档（cell 8），`chat_completions(messages, doc_id=doc_id, stream=True)` 流式对话（cell 12），并用手动构造的 `retrieval_prompt` 让模型返回 JSON 格式的页-内容列表（cell 14）：

```python
retrieval_prompt = f"""
Your job is to retrieve the raw relevant content from the document based on the user's query.

Query: {query}

Return in JSON format:
```json
[
  {{"page": <number>, "content": "<raw text>"}},
  ...
]
```
"""
```

旧版 notebook 的"thinking / node_list 树搜索 prompt"模式已随重构消失，当前以 `chat_completions` 端到端对话为主。

### 两条路径的差异

| 维度 | 路径 A：Agents SDK demo | 路径 B：Notebook |
|---|---|---|
| 依赖框架 | OpenAI Agents SDK | 客户端内置 chat_completions |
| 推理循环 | SDK 内置 agent 循环 + agent_instructions playbook | 手动构造检索 prompt |
| 使用的客户端 | `PageIndexLocalClient` / `PageIndexCloudClient` | Cloud 模式（`submit_document` + `chat_completions`） |
| 工具接入 | 自动（openai_agent_config / as_openai_tools / as_anthropic_tools / as_claude_mcp） | 无工具，单次对话 |
| 封装程度 | 完整 agent 组装 | 单 prompt 调用 |

两条路径实现了同一核心模式（获取树结构 → LLM 推理定位 → 精确获取页面内容），但不共享代码，不存在独立的封装抽象。

> **当前没有将示例层推理模式单独拆成专题。** 内容体量还不足以独立成文，且该模式的本质是检索协议的两条调用方实现路径，不是独立的机制专题。如果后续出现第三条实现路径，或该模式被核心库吸收封装，再评估拆分。

## 当前最稳的判断

- 核心库提供的是 index + 结构树 + 页内容三个原子能力，不是端到端推理系统。`Verified`
- 树结构去文本（`get_document_structure` / `include_text=False`）是一个省 token 的关键设计点——让 LLM 在不暴露完整文本的前提下推理定位。`Verified`
- 示例层已经形成了概念模式并通过 `agent_tools` / `agent_instructions` 封装成工具契约，但推理循环仍由 Agents SDK / 调用方驱动。`Verified / Observed`

## Evidence

- Status: `Verified`（检索接口签名与实现、`_parse_pages`、get_ocr format 取值、demo 与 notebook 结构）、`Observed`（cloud 端对话质量与检索行为不可核验）
- Sources: 基准来源：`https://github.com/VectifyAI/PageIndex`；后续引用：`pageindex/client.py`、`pageindex/local_api.py`、`pageindex/agent_tools.py`、`examples/agentic_vectorless_rag_demo.py`、`cookbook/agentic_retrieval.ipynb`
- Version Basis: 复核截面 `branch main`, `commit ae2a5b4`（2026-08-17）；原研究截面 `commit 5a18553284ed`（2026-06-13，本文旧版内容，含 `retrieve.py` 与 AGENT_SYSTEM_PROMPT 描述）
- Observed At: `2026-08-19`（复核，本地克隆 `/Volumes/ZGY93B6F/github/PageIndex`）；原截面 `2026-06-13`
- Scope: 核心库检索协议结论适用于本地 Python 仓库；示例层结论适用于本次观察到的 demo 与 notebook
- Drift Risk: `medium`
- Trace: 原截面：源码研究第二轮（`temp/chat/round9-feedback.md`）补读示例层与 notebook；2026-08-19 复核：`retrieve.py` 移除、检索接口重构、demo 与 notebook 重写后全文更新
- Needs: Cloud API 端的检索实现不可见，无法确认服务端推理模式与本地模式的差异
