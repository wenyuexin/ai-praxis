# 开源版与闭源产品线的能力边界

## 三层产品形态

PageIndex 的能力不是单一产品形态，而是"开源 SDK + 托管服务 + 企业部署"三层，闭源部分位于后两层：

| 形态 | 代码可见性 | 获取方式 | 说明 |
|---|---|---|---|
| 开源仓库（OSS） | MIT，完全可见 | GitHub 自托管 | 本地索引（standard/flash）+ 本地检索 + Cloud API 客户端，全部在仓内 |
| Cloud Service | 仅客户端接口面（`cloud_api.py`） | API key 即用（`dash.pageindex.ai/api-keys`） | 服务端引擎含增强 OCR、树构建、检索、流式对话；Chat 平台 / MCP / API 均可接入（本地模式同样可经仓内 `mcp_bridge.py` / `as_claude_mcp` 接入 MCP） |
| Enterprise | 不可见 | 联系销售（VPC / on-prem 部署） | 含 PageIndex File System（语料级索引，见下文） |

代码证据：`client.py:101-102` 同一 `PageIndexClient` 传 `api_key` 走 cloud、不传走 local；`PageIndexCloudClient` / `PageIndexLocalClient` 显式固定模式（`client.py:1141/1153`）。

## 开源仓库内可见的"闭源侧"接口面

`cloud_api.py`（435 行）是闭源服务端的完整客户端调用面，已在开源仓库内（旧截面 `5a18553284ed` 时这些方法不在仓内，仅在 notebook 调用层可见；复核截面 `ae2a5b4` 已入仓）：

- 文档：`submit_document`（`:36`）、`get_document`（`:293`）、`delete_document`（`:318`）、`list_documents`（`:337`，支持 `folder_id`）
- 检索：`get_ocr`（`:86`）、`get_tree`（`:111`）、`submit_query`（`:135`）、`get_retrieval`（`:162`）
- 对话：`chat_completions`（`:183`，流式）；`client.py` 另提供 OpenAI/Anthropic 兼容的 `chat` / `responses` / `messages`
- 文件夹：`create_folder` / `list_folders`（`client.py:1106-1131`）

这些接口的**服务端实现不可核验**：多文档路由、增强 OCR、流式检索的真实支持范围均只能从调用层推断。`Unverified`（服务端）/ `Verified`（客户端调用面）

## 能力对照（本地模式 vs Cloud 模式）

| 能力 | 本地模式 | Cloud 模式 | 证据 |
|---|---|---|---|
| PDF 索引 | ✅ `submit_document`（本地解析，PyPDF2；flash/standard） | ✅ `submit_document`（服务端增强 OCR） | `local_api.py:59` vs `cloud_api.py:36` |
| MD 索引 | ⚠️ 仅 CLI（`run_pageindex.py`），客户端路径不可用 | 未核验 | `run_pageindex.py:146-148` |
| 树结构获取 | ✅ `get_tree` / `get_document_structure` | ✅ `get_tree` | `client.py:320/343` |
| 页内容获取 | ✅ `get_ocr` / `get_page_content` | ✅ `get_ocr` | `client.py:278/297` |
| 流式检索对话 | ❌ | ✅ `chat_completions`（调用层可见，服务端不可见） | `cloud_api.py:183` |
| 文件夹组织 | ❌ 报错 | ✅ `create_folder` / `list_folders` / `folder_id` | `client.py:1106`、`agent_tools.py:490` |
| 语义文档排序 | ❌ 报错 | ✅ `sort="relevance"` / `query` | `agent_tools.py:506-517` |
| 文档级列举 | ✅ `browse_documents` / `list_documents`（时间序分页） | ✅ 同上 + 语义排序 | `agent_tools.py:664`、`local_api.py:317` |
| 多文档检索 | ❌ 单文档接口 | ⚠️ 仅产品级宣称（File System，enterprise/cloud） | 官方 blog（`Observed`） |
| 增强 OCR | ❌ PyPDF2 基础提取 | ✅ 服务端引擎 | README 文案（`Observed`） |
| 持久化 | ✅ DocStore（`docs/{doc_id}/` + `manifest.json`） | ❌ 服务端管理 | `local_store.py` |

## 语料级索引：PageIndex File System（企业/云特性）

这是"开源版能不能做文档级索引"问题的答案核心。官方立项动机（blog 原文）：经典向量 RAG 在语料规模上有三个天花板——"accuracy falling short, recall gaps you can't audit, retrieval blind to context"（准确率不足、召回缺口不可审计、检索对上下文无感知），File System 是其给出的替代路径。`Observed`

- **开源仓库中无任何实现代码**：全仓 grep `file system` / `corpus` / `FileSystem` 仅命中 README.md 第 34 行指向官方 blog 的链接。`Verified`
- **可用性**：官方 blog（2026-07）称 "Available today for enterprise. Cloud rollout coming soon."——Enterprise GA（Dedicated or VPC deployment），cloud 版无独立证据确认是否已上线（截至 2026-08-19）。`Observed`
- **OSS 路线承诺**：官方明确 "Existing OSS users keep everything that's in the open-source repo, and the OSS roadmap continues unchanged."——File System 不会回流开源，OSS 路线保持不变。`Observed`
- **对用户场景的含义**：按目录树组织的文档库若想用 OSS 部署做文档级索引——不可行；OSS 只提供扁平文档集合 + 调用方自行匹配（`browse_documents` 报错原文："match the returned names and descriptions against the intent yourself"）。`Verified`

### 官方博客描述的机制（仅 blog 原文，无代码可核验）

以下全部来自 `https://pageindex.ai/blog/pageindex-filesystem`（2026-07），`Observed`：

1. **统一树**：语料即树——"each document tree hangs off a leaf of the file system tree, and the whole corpus becomes one big tree"；同一套树导航策略从语料级目录导航一路下沉进单文档内部树（"descend, without changing tools, into a specific document's internal tree"）。
2. **虚拟节点合成**：语料无可用层级时主动构建——"Documents are clustered into topic nodes by topic models or LLM-driven grouping; each document also gets LLM-inferred metadata (category, summary, key entities)"；同一文档可挂多个虚拟祖先（如 vendor / region / year），扁平文件系统无法表达这种多归属。
3. **查询相关构建**：树按查询按需生成——"Given a question, it picks which metadata axes to use as internal nodes, which clusters to surface, and how deep to nest them"；不同查询对同一语料长出不同视图，无需重新摄取或重新嵌入；历史查询的 traversal patterns 反哺虚拟节点与元数据。
4. **dynamic flattening / 逐节点策略选择**：静态逐层遍历在规模上是错误默认——"Sometimes a node's children carry rich, query-relevant labels (`/contracts/2024/vendor_X/`) … Sometimes the labels are uninformative (`misc/`, `folder_1/`, an arbitrary user-uploaded directory) … walking the structure layer by layer just burns LLM calls … PageIndex picks the strategy per node, conditioned on the query"；子层标签有信息量时按标签剪枝（layer-wise），无信息量时折叠子树直达叶子、把判别推迟到实际内容（"the depth of the search shrinks to the depth that actually carries information for the question being asked"），这是百万文档规模下保持效率的关键宣称。

### 博客没有讲清楚的

- 无检索调用链：没有"查询 → 树上逐层决策 → 选中文档"的接口、伪代码或 prompt 设计
- 无工程参数：剪枝阈值、聚类规模、嵌套深度规则均未给出
- 无 benchmark：百万文档单索引是宣称，无 latency / throughput / 召回率数据
- 无开源实现：所有机制 "all live in the enterprise release"

## 证据等级与未确定点

| 断言 | 证据等级 |
|---|---|
| 开源仓库三层形态成立、本地/云双模式客户端 | `Verified` |
| 本地模式仅 PDF、无文件夹、无语义排序 | `Verified` |
| Cloud API 客户端调用面（方法签名） | `Verified` |
| Cloud API 服务端实现（多文档、增强 OCR、流式检索） | `Unverified` |
| File System 为 Enterprise/Cloud 特性、OSS 无代码 | `Verified`（代码侧）/ `Observed`（产品侧） |
| File System 机制细节（virtual nodes / query-dependent / dynamic flattening） | `Observed`（blog 原文） |
| File System cloud 版已上线 | `Unverified`（无独立证据） |

## 对对象研究的影响

Cloud / Enterprise 侧的额外能力（文件夹、语义排序、流式对话、File System）在开源仓库中或不可见、或仅以客户端接口存在，不应纳入对象研究的核心判断。PageIndex 最独特的贡献仍是其 PDF/MD 树索引构建管线和 vectorless 检索协议设计，而非端到端检索系统能力。本地 SDK 架构细节见 `sdk-and-workspace.md`。

## Evidence

- Status: `Verified`（三层形态、双模式客户端、本地硬边界、cloud_api.py 调用面、OSS 无 File System 代码）、`Observed`（File System 产品宣称与机制、增强 OCR、服务端能力）、`Unverified`（服务端实现、File System cloud 版）
- Sources: 基准来源：`https://github.com/VectifyAI/PageIndex`；后续引用：`pageindex/client.py`、`pageindex/cloud_api.py`、`pageindex/local_api.py`、`pageindex/local_store.py`、`pageindex/agent_tools.py`、`run_pageindex.py`；官方 blog（`https://pageindex.ai/blog/pageindex-filesystem`，2026-07，`Observed`）
- Version Basis: 复核截面 `branch main`, `commit ae2a5b4`（2026-08-17）；原研究截面 `commit 5a18553284ed`（2026-06-13，Cloud API 方法不在仓内，本文差异已标注）
- Observed At: `2026-08-19`（复核，本地克隆 `/Volumes/ZGY93B6F/github/PageIndex`）；原截面 `2026-06-13`
- Scope: 本文能力对照仅覆盖开源仓库内可核验的本地模式与 Cloud 客户端调用面；服务端实现不在范围内
- Drift Risk: `high`（产品形态与 File System 可用性变化快）
- Trace: 由 `sdk-and-workspace.md` 拆分而来（2026-08-19 重组：本地架构与开源/闭源边界分离）；Cloud 客户端入仓、本地硬边界、File System 企业特性均按复核截面核验；File System 机制引文与"博客没讲清楚"清单基于 2026-08-19 对官方 blog 的完整抓取
- Needs: Cloud API 端服务实现需独立验证；File System cloud 版上线时间与机制需官方文档或代码级确认
