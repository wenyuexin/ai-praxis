# 本地 SDK 架构、Workspace 与 Cloud API 边界

## 本地 SDK 架构

PageIndex 的 Python SDK 由五个核心模块组成：

| 模块 | 文件 | 行数 | 核心职责 |
|---|---|---|---|
| PDF 索引 | `page_index.py` | 1154 | PDF 树索引构建 |
| Markdown 索引 | `page_index_md.py` | 342 | MD 树索引构建 |
| 检索工具 | `retrieve.py` | 138 | 文档元数据、树结构、页内容 |
| 客户端封装 | `client.py` | 235 | PageIndexClient：索引→持久化→按需加载 |
| 工具函数 | `utils.py` | 711 | LLM 调用、token 计数、树操作、配置 |

`client.py:28-34` 中的 `PageIndexClient` 是用户的主要入口：

```python
class PageIndexClient:
    """
    A client for indexing and retrieving document content.
    Flow: index() -> get_document() / get_document_structure() / get_page_content()
    """
```

初始化参数 `api_key`、`model`、`retrieve_model`、`workspace` 通过 `ConfigLoader`（`utils.py:654-685`）合并 `config.yaml` 默认值。

## Workspace 机制：多文档索引存储

`PageIndexClient` 的 workspace 支持多个文档的索引存储：

1. **索引阶段**：`index(file_path)`（`client.py:55-130`）将文档索引结果存为独立 JSON 文件 + 更新 `_meta.json` 元数据索引
2. **工作区加载**：`_load_workspace()`（`client.py:196-206`）在初始化时加载 `_meta.json`，显示 `f"Loaded {len(meta)} document(s) from workspace"`
3. **按需加载**：`_ensure_doc_loaded(doc_id)`（`client.py:208-218`）不在初始化时加载所有完整文档，而是首次访问具体 doc_id 时懒加载

**但检索接口是单文档粒度的**。三个公共检索方法全部接受 `doc_id: str` 参数，没有跨文档签名：

```python
def get_document(self, doc_id: str) -> str:       # client.py:220
def get_document_structure(self, doc_id: str) -> str:  # client.py:224
def get_page_content(self, doc_id: str, pages: str) -> str:  # client.py:230
```

**核心判断**：本地 SDK 支持多文档索引存储，但不支持真正的多文档检索闭环。所有检索调用必须显式传入 `doc_id`。如需跨文档查询，需要调用方自行决定先查哪个文档。

## Cloud API 与本地 SDK 的能力对比

### Cloud API 独有方法

两个 cookbook notebook 使用了以下不在本地 `client.py` 中的方法：
- `submit_document(file_path)` — 提交文档到服务端处理（不在本地 `client.py` 中）
- `is_retrieval_ready(doc_id)` — 检查处理状态（同上）
- `get_tree(doc_id, node_summary)` — 获取树结构（与 `get_document_structure` 功能相近但来自 Cloud API）
- `chat_completions(prompt, ...)` — 流式检索对话（本地 SDK 中没有）

### 能力对照

| 能力 | 本地 `PageIndexClient` | Cloud API | 证据 |
|---|---|---|---|
| PDF 索引（本地 PDF 解析） | ✅ `index()` | ❌ 不可用 | `client.py:55-130` |
| MD 索引（正则 + LLM） | ✅ `index()` | ❌ 不可用 | `client.py:97-123` |
| 树结构获取 | ✅ `get_document_structure()` | ✅ `get_tree()`（调用层可见） | `client.py:224` vs `notebook:323` |
| 文档提交（服务端处理） | ❌ | ✅ `submit_document()`（调用层可见） | `notebook:151` |
| 处理状态检查 | ❌ | ✅ `is_retrieval_ready()`（调用层可见） | `notebook:321` |
| 流式检索对话 | ❌ | ⚠️ `chat_completions()`（调用层可见，服务端实现不可见） | `notebook:342` |
| 多文档检索 | ❌ 单文档接口 | ⚠️ 仅产品级宣称 | `notebook:52`、`README.md:34` |
| 增强 OCR | ❌ PyPDF2 基础提取 | ⚠️ README 文案提及 | `README.md:211-223`（注释块） |
| Workspace 持久化 | ✅ JSON + `_meta.json` | ❌ 服务端管理 | `client.py:157-194` |
| 检索模型配置 | ✅ `retrieve_model` | ⚠️ 不可见 | `client.py:48` |

### 关键差异总结

1. **索引层不同**：本地 SDK 使用本地 PDF 解析（PyPDF2/PyMuPDF）构建索引；Cloud API 使用服务端引擎（含增强 OCR），质量更高但对本地侧不可见。
2. **检索路径不同**：本地 SDK 需要调用方手动构建 Agent 或逻辑来执行"取结构→推理→获取页面"三步流程；Cloud API 通过 `chat_completions()` 在服务端封装了推理+检索逻辑（仅调用层可见，`notebook:342`）。
3. **能力边界**：`submit_document`、`chat_completions`、`is_retrieval_ready` 等 Cloud API 方法在本地 SDK 的 `client.py`（235 行，15 个方法）中**完全不存在**。`Verified`。这些方法仅在 notebook 调用层可见（`Observed`）；服务端实现质量不可核验。
4. **多文档差距**：本地 SDK 明确不支持跨文档检索（检索方法签名全为单 `doc_id` 参数）；Cloud API 在 notebook 的产品描述行中宣称支持多文档（`notebook:52`），但不可复现，不应作为已验证能力引用。`Observed`。

## 对对象研究的影响

PageIndex 作为仓库中独立研究对象的核心价值在于其**本地 SDK（索引构建 + 单文档检索协议）**。Cloud API 的额外能力（多文档、流式对话、增强 OCR）在本地源码中不可见，不应纳入对象研究的核心判断。这强化了 PageIndex 最独特的贡献是其 PDF/MD 树索引构建管线和 vectorless 检索协议设计，而非端到端检索系统能力。

## Evidence

- Status: `Verified`（本地 SDK 所有公共方法签名、workspace 多文档索引存储机制、Cloud API 方法不在本地 SDK 中）、`Observed`（Cloud API 方法在 notebook 调用层可见，但服务端实现不可核验；多文档宣称不可复现）
- Sources: 基准来源：`https://github.com/VectifyAI/PageIndex`；后续引用：`pageindex/client.py`、`pageindex/__init__.py`、`cookbook/agentic_retrieval.ipynb`、`cookbook/pageindex_RAG_simple.ipynb`
- Version Basis: `branch main`, `commit 5a18553284ed`
- Observed At: `2026-06-13`
- Scope: 本文中的本地 SDK 结论适用于当前仓库截面；Cloud API 结论仅限 notebook 调用层可见能力，不代表服务端实现已核验
- Drift Risk: `high`
- Trace: 源码研究第三轮（`temp/chat/round10-feedback.md`）聚焦 SDK 边界与 Cloud API 差异
- Needs: Cloud API 端服务实现需独立验证；pip 安装版 pageindex 包的功能集与本地 repo 版本的关系待确认
