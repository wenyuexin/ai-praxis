# Evidence Notes：PageIndex Claim-Source 对照

- Base Source: `https://github.com/VectifyAI/PageIndex`
- Version Basis: `branch main`, `commit 5a18553284ed`
- Observed At: `2026-06-13`
- Scope: 本文中的源码 Claim 仅适用于本次观察到的本地 Python 仓库与示例层截面
- Drift Risk: `medium`

## 核心 Claim 状态

| Claim | Status | 源码/来源证据 |
|---|---|---|
| PageIndex 是 VectifyAI 的具体开源系统/框架名 | Verified | `github.com/VectifyAI/PageIndex` 官方 README + `pageindex/` 源码 |
| "vectorless" 在源码层面成立 | Verified | `client.py:55-130` 无向量操作；`client.py` 无 embedding 调用 |
| 核心库不包含推理循环 | Verified | `page_index.py`、`page_index_md.py`、`client.py`、`retrieve.py` grep "agent"/"reasoning"/"loop" 0 结果 |
| `retrieve_model` 不参与核心检索 | Verified | `client.py:48` 只存为属性；`client.py:220-234` 三个检索方法均不调用 LLM |
| `retrieve_model` 在示例 demo 中被消费 | Verified | `agentic_vectorless_rag_demo.py:85` `Agent(..., model=client.retrieve_model)` |
| 本地 SDK 检索接口全是单文档粒度 | Verified | `client.py:220` `get_document(doc_id)`、`client.py:224` `get_document_structure(doc_id)`、`client.py:230` `get_page_content(doc_id, pages)` |
| workspace 支持多文档索引存储 | Verified | `client.py:50-53` workspace 创建 + `client.py:196-206` `_load_workspace()` 多文档加载 |
| PDF 索引使用 LLM 进行标题页验证 | Verified | `page_index.py:13-45` `check_title_appearance()` 使用 LLM prompt |
| PDF 和 MD 索引共享树构建函数 | Verified | `page_index_md.py:190-221` `build_tree_from_nodes()` 被两个管线调用 |
| FinanceBench 98.7% | Observed | 官方博客页面声明，第三方独立复现未验证 |
| OpenKB 使用 PageIndex 作为长文档引擎 | Observed | 官方 OpenKB README 声明，OpenKB 源码不在本次研究范围 |
| 示例层已形成 concept pattern 但未封装 | Observed | 两个示例实现同一模式但不共享代码 |
| Cloud API 宣称支持多文档 | Observed | `agentic_retrieval.ipynb:52` 产品级宣称；不可复现 |
| Cloud API 方法不在本地 SDK | Verified | `client.py` 15 个方法；`__init__.py` 导出；`submit_document`/`chat_completions` 等完全不在此本地仓库中 |
| 多文档检索能力（非索引存储） | Unverified | 本地 SDK 不支持；Cloud API 不可核验 |
| Cloud API 端增强 OCR | Unverified | README 注释块提及，但不在本地源码中 |
| chat_completions 签名和实现 | Unverified | 不在本地仓库任何文件中 |

## 仍不确定的点

1. **Cloud API SDK 与本地 SDK 的关系**：`client.py`（235 行）不包含 `submit_document`、`chat_completions` 等方法。这些方法不在任何本地文件中；Cloud API 端的能力主要通过 notebook 调用层可见，服务端实现不可核验。**`Observed`**（调用层可见）/**`Unverified`**（服务端实现）。

2. **多文档检索的真实实现**：本地 SDK 明确不支持。Cloud API 在产品描述行中宣称支持（`notebook:52`），但不可复现。OpenKB 可能在应用层实现跨文档检索，但这不是 PageIndex 研究范围。**`Unverified`**。

3. **`retrieve_model` 与实际推理质量的关系**：`config.yaml` 设置 `model: "gpt-4o-2024-11-20"`（索引用）和 `retrieve_model: "gpt-5.4"`（检索推理用），说明团队认为检索推理需要更强的模型。但这一策略的实际效果、token 成本影响、模型选择依据等均不可知。**`Inferred`**。

4. **PDF 树索引的 LLM prompt 具体设计**：`page_index.py` 中多轮 LLM 调用的 prompt 模板细节未逐条记录。这些 prompt 本身可能是高价值的工程模式。**`Unverified`**。

## 与目录内其他专题的交叉引用

| 专题 | 依赖的证据 | 已标记 |
|---|---|---|
| `overview.md` | 核心 Claim 1-3 | ✅ Noted |
| `pdf-indexing.md` | 核心 Claim 7-9 | ✅ Noted |
| `retrieval-protocol.md` | 核心 Claim 3-6 | ✅ Noted |
| `sdk-and-workspace.md` | 核心 Claim 5-12 | ✅ Noted |
