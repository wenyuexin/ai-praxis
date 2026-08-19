# Evidence Notes：PageIndex Claim-Source 对照

- Base Source: `https://github.com/VectifyAI/PageIndex`
- Version Basis: 复核截面 `branch main`, `commit ae2a5b4`（2026-08-17）；原研究截面 `commit 5a18553284ed`（2026-06-13）
- Observed At: `2026-08-19`（复核，本地克隆 `/Volumes/ZGY93B6F/github/PageIndex`）；原截面 `2026-06-13`
- Scope: 本文中的源码 Claim 适用于复核截面 `ae2a5b4`；原截面结论已按复核截面更新，差异逐条标注
- Drift Risk: `medium`

## 核心 Claim 状态

| Claim | Status | 源码/来源证据 |
|---|---|---|
| PageIndex 是 VectifyAI 的具体开源系统/框架名 | Verified | `github.com/VectifyAI/PageIndex` 官方 README + `pageindex/` 源码 |
| "vectorless" 在源码层面成立 | Verified | `client.py` / `local_api.py` / `cloud_api.py` 无 embedding 调用（grep 0 结果） |
| 核心库不包含推理循环 | Verified | 检索函数不调用 LLM；推理由 Agents SDK / 调用方驱动 |
| `retrieve_model` 不参与核心检索 | Verified | 检索函数均不调用 LLM；示例 demo 重构后不再消费 `retrieve_model` |
| 本地 SDK 检索接口全是单文档粒度 | Verified | `client.py:732/320/343/278/297` 全部要求 `doc_id` |
| DocStore 支持多文档索引存储 | Verified | `local_store.py:29` `DocStore`，`docs/{doc_id}/`（tree.json/pages.json/doc.json）+ `manifest.json` |
| 本地客户端路径仅接受 PDF | Verified | `local_api.py:96` `submit_document` 对非 `.pdf` 报错；MD 仅 CLI |
| 文件夹 / 语义文档排序为 cloud-only | Verified | `client.py:1106` `create_folder` cloud-only；`agent_tools.py:490/506` 本地报错 |
| PDF 索引使用 LLM 进行标题页验证 | Verified | `page_index_classic.py` `check_title_appearance()` 使用 LLM prompt |
| PDF 和 MD 索引共享树构建函数（输出形态） | Verified | `page_index_md.py:192` `build_tree_from_nodes()` |
| Cloud API 方法已进入 SDK（cloud 模式） | Verified | `cloud_api.py`：`submit_document`/`chat_completions`/`submit_query`/`get_retrieval`/folders 等（原截面为"不在本地 SDK"，已反转） |
| Cloud API 服务端实现 | Unverified | `cloud_api.py` 仅客户端调用面；服务端多文档、增强 OCR、流式检索真实支持范围不可核验 |
| 语料级索引 "PageIndex File System" 为企业/云特性 | Observed | 官方 blog（2026-07）：Enterprise GA、cloud "coming soon"；OSS 仓库 grep 无实现代码 |
| File System 机制为 query-time 虚拟节点树 | Observed | 官方 blog 原文 "Virtual-node synthesis and query-dependent index construction"（"What's available today" 列表） |
| FinanceBench 98.7% | Observed | 官方博客页面声明，第三方独立复现未验证 |
| 官方增长数据（26k+ stars、GitHub Trending #1、Secure Open Source Fund、23k+ cloud users） | Observed | 官方 blog（2026-07）写作时点数据；与 2026-08-19 观察的 35.2k+ stars 均为时点值 |
| OpenKB 使用 PageIndex 作为长文档引擎 | Observed | 官方 OpenKB README 声明，OpenKB 源码不在本次研究范围 |
| 示例层已形成工具契约但推理循环在框架侧 | Verified / Observed | `agent_tools.py` + `agent_instructions()`；Agents SDK 驱动循环 |

## 仍不确定的点

1. **Cloud API 服务端实现**：`cloud_api.py` 已提供完整客户端调用面（`submit_document`、`chat_completions`、`submit_query`、`get_retrieval`、folders），但服务端实现不可核验；多文档、增强 OCR、流式检索的真实支持范围仍不可复现。**`Unverified`**（服务端）/ **`Verified`**（客户端调用面）。

2. **File System cloud 版可用性**：官方 blog（2026-07）称 Enterprise GA、cloud "rolling out later this month"；截至 2026-08-19 无独立证据确认 cloud 版是否上线，机制细节（virtual nodes、query-dependent construction、dynamic flattening）仅来自官方 blog 原文。**`Observed`**。

3. **多文档检索的真实实现**：本地 SDK 明确不支持。Cloud API 仅在产品宣称中涉及（File System）；OpenKB 可能在应用层实现跨文档检索，但这不是 PageIndex 研究范围。**`Unverified`**。

4. **PDF 树索引的 LLM prompt 具体设计**：`page_index_classic.py` 中多轮 LLM 调用的 prompt 模板细节未逐条记录。这些 prompt 本身可能是高价值的工程模式。**`Unverified`**。

## 与目录内其他专题的交叉引用

| 专题 | 依赖的证据 | 已标记 |
|---|---|---|
| `overview.md` | 核心 Claim 1-3、13-14 | ✅ Noted |
| `pdf-indexing.md` | 核心 Claim 9-10 | ✅ Noted |
| `retrieval-protocol.md` | 核心 Claim 3-5、18 | ✅ Noted |
| `sdk-and-workspace.md` | 核心 Claim 5-8（本地架构侧） | ✅ Noted |
| `capability-boundary.md` | 核心 Claim 8、11-14（开源/闭源边界侧） | ✅ Noted |
