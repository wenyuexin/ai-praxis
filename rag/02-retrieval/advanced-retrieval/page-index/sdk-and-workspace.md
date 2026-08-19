# 本地 SDK 架构、存储与本地模式边界

## 本地 SDK 架构

PageIndex 的 Python SDK（`ae2a5b4`，2026-08-17，约 8700 行）由以下模块组成：

| 模块 | 文件 | 行数 | 核心职责 |
|---|---|---|---|
| PDF 索引（经典） | `page_index_classic.py` | 1323 | PDF 树索引构建（原 `page_index.py`） |
| PDF 索引（快速） | `flash/` | — | flash 模式快速树结构生成（pypdfium2 解析） |
| Markdown 索引 | `page_index_md.py` | 344 | MD 树索引构建（仅 CLI 接入） |
| 本地 API | `local_api.py` | 370 | LocalAPI：索引、树/页/元数据检索、文档管理 |
| 本地存储 | `local_store.py` | 164 | DocStore：文档 JSON 持久化 + manifest 注册 |
| Cloud API | `cloud_api.py` | 435 | CloudAPI：闭源服务端接口客户端（见 `capability-boundary.md`） |
| 客户端封装 | `client.py` | 1171 | PageIndexClient：本地/云双模式统一入口 |
| Agent 工具 | `agent_tools.py` | 1633 | 工具契约、指令构建、工具调用路由 |
| 本地对话 | `local_chat.py` | 977 | 本地 chat 封装 |
| 树优化 | `tree_optimize.py` | 947 | 树优化（thinning 等） |
| MCP bridge | `mcp_bridge.py` | 210 | MCP 服务桥接 |
| 工具函数 | `utils.py` | 1104 | LLM 调用、token 计数、树操作、配置 |

`client.py:54` 的 `PageIndexClient` 是统一入口，通过 `api_key` 有无切换模式（`client.py:101-102`：传 `api_key` 走 cloud，不传走 local）；`PageIndexCloudClient`（`client.py:1141`）与 `PageIndexLocalClient`（`client.py:1153`）用于显式固定模式。开源/闭源两套形态的能力差异见 [`capability-boundary.md`](./capability-boundary.md)，本文只讲本地模式。

## 本地模式：DocStore 多文档存储，单文档检索

本地模式的文档存储由 `local_store.py` 的 `DocStore` 承担：

1. **索引阶段**：`LocalAPI.submit_document(file_path)`（`local_api.py:59`）将文档索引结果持久化为 `docs/{doc_id}/tree.json + pages.json + doc.json`，并写入 `manifest.json` 注册表（`local_store.py:80-90`）。
2. **注册表加载**：`list_metas()`（`local_store.py:126-145`）扫描 `docs/` 目录并对照 manifest 返回文档元数据列表。
3. **按需加载**：检索方法按 `doc_id` 懒加载对应目录文件。

**但检索接口是单文档粒度的**。公共检索方法全部接受 `doc_id` 参数，没有跨文档签名：

```python
def get_document(self, doc_id: str) -> dict:            # client.py:732
def get_tree(self, doc_id, ...) -> dict:                # client.py:320
def get_document_structure(self, doc_id) -> list:       # client.py:343（不含文本）
def get_ocr(self, doc_id, format="page") -> dict:       # client.py:278 / local_api.py:249
def get_page_content(self, doc_id, pages) -> list:      # client.py:297
```

**核心判断**：本地 SDK 支持多文档索引存储，但不支持真正的多文档检索闭环。所有检索调用必须显式传入 `doc_id`。如需跨文档查询，需要调用方自行决定先查哪个文档。

## 本地模式的三个硬边界

- **仅接受 PDF**：`submit_document` 对非 `.pdf` 文件直接报错（`local_api.py:96`："only PDF files are supported in local mode"）。Markdown 索引（`md_to_tree`）只经 `run_pageindex.py --md_path` CLI 接入，客户端 SDK 路径不可用。
- **无文件夹组织**：`create_folder` / `list_folders` 是 cloud-only（`client.py:1106-1131`）；`browse_documents` 传 `folder_id` 报 "Folders are not supported in local mode yet"（`agent_tools.py:490`）。
- **无语义文档排序**：`browse_documents` 的 `sort="relevance"` / `query` 参数在本地模式直接报错（`agent_tools.py:506-517`），错误提示明确要求调用方 "match the returned names and descriptions against the intent yourself"——文档选择由调用方负责。

## Evidence

- Status: `Verified`（本地 SDK 模块结构、`submit_document` PDF-only、`DocStore` 存储布局、单文档检索签名、folders/语义排序 cloud-only）
- Sources: 基准来源：`https://github.com/VectifyAI/PageIndex`；后续引用：`pageindex/client.py`、`pageindex/local_api.py`、`pageindex/local_store.py`、`pageindex/agent_tools.py`、`run_pageindex.py`
- Version Basis: 复核截面 `branch main`, `commit ae2a5b4`（2026-08-17）；原研究截面 `commit 5a18553284ed`（2026-06-13，本文旧版内容）
- Observed At: `2026-08-19`（复核，本地克隆 `/Volumes/ZGY93B6F/github/PageIndex`）；原截面 `2026-06-13`
- Scope: 本文结论适用于 2026-08-19 复核的 `ae2a5b4` 截面；Cloud 侧能力与开源/闭源边界见 `capability-boundary.md`
- Drift Risk: `high`
- Trace: 原截面：源码研究第三轮（`temp/chat/round10-feedback.md`）聚焦 SDK 边界与 Cloud API 差异；2026-08-19 复核：SDK 重构后模块表、DocStore、PDF-only、cloud-only 能力边界全部按新代码更新；同日重组：Cloud 对照部分拆分至 `capability-boundary.md`
- Needs: Cloud API 端服务实现需独立验证；File System cloud 版可用性无独立证据（见 `capability-boundary.md`）
