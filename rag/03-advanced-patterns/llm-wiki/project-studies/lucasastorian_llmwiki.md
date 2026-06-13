---
skill: synthesize-repo
domain: github
theme: ""
purpose: "基于 GitHub 仓库信息生成结构化分析笔记"
sources:
  - "https://github.com/lucasastorian/llmwiki"
tags:
  - github
  - "Python"
  - repo-analysis
created: "2026-05-03"
updated: "2026-05-03"
status: "stable"
---

# lucasastorian/llmwiki

> **一句话总结**：Open Source Implementation of Karpathy's LLM Wiki. Upload documents, connect your Claude account via MCP, and have it write your wiki ! 

## 基本信息

- **仓库**：lucasastorian/llmwiki
- **描述**：Open Source Implementation of Karpathy's LLM Wiki. Upload documents, connect your Claude account via MCP, and have it write your wiki ! 
- **Stars**：786 | **Forks**：122
- **Watchers**：-
- **最近更新**：2026-04-27
- **Open Issues**：6 | **Open PRs**：-
- **创建时间**：2026-04-04
- **主要语言**：Python
- **许可证**：Apache License 2.0（可自由商用，需保留版权声明）
- **链接**：https://github.com/lucasastorian/llmwiki

## 项目定位

- **解决的问题**：个人研究文件夹积累资料的速度远快于手动维护摘要、链接和引用的速度。LLM Wiki 将编辑工作（更新交叉引用、保持摘要最新、标记矛盾）卸载给 Claude，用户只需专注于选择来源和指导分析方向。
- **目标用户**：研究人员、知识工作者、需要维护个人知识库的用户。适合已有大量分散文档（PDF、笔记、表格）但缺乏系统化整理的人群。
- **在生态中的角色**：Karpathy LLM Wiki pattern 的完整可运行实现，填补了"有 pattern 描述但无现成工具"的空白。与静态笔记工具（Obsidian、Notion）互补，核心差异在于 AI 持续维护而非人工编辑。

## 上手成本

- **安装方式**：Git 克隆 + Python 虚拟环境 + npm install。提供 `llmwiki` CLI 脚本简化操作。
- **前置依赖**：Python 3.11+、Node.js 20+
- **首次配置**：本地模式无需 API key；可选配置 `MISTRAL_API_KEY` 获得更高质量 PDF OCR。托管模式需要 Supabase + S3。
- **学习曲线**：README 包含完整的 Quick Start、架构说明、自托管指南。一键启动命令 `./llmwiki open ~/research` 降低了初始门槛。

## 技术栈分析

| 层级 | 技术 | 说明 |
|------|------|------|


> **技术栈推断**：未识别到明确的依赖配置文件。主要语言为 Python，具体技术栈需进一步分析源码。

## 核心功能

- **本地文档索引**：支持 PDF、Markdown、HTML、Excel/CSV、图片、Word/PowerPoint 等格式，提取文本并构建 SQLite FTS5 搜索索引
- **MCP 驱动的 AI 维护**：Claude 通过 MCP 工具（guide/search/read/write/delete）主动读写 wiki 页面、维护交叉引用和脚注
- **文件系统真相来源**：所有 wiki 页面以 Markdown 形式存储在 `wiki/` 目录，可直接用任何编辑器修改，SQLite 索引可重建
- **双模式架构**：本地模式（零配置、SQLite）和托管模式（Postgres + Supabase auth + S3）
- **一键启动**：`./llmwiki open ~/research` 自动完成初始化、启动服务、打开浏览器
- **后台文件监听**：外部编辑器修改文件后自动更新索引

## 架构设计

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Next.js    │────▶│   FastAPI    │────▶│   SQLite     │
│   Frontend   │     │   Backend    │     │   (local)    │
└──────────────┘     └──────┬───────┘     └──────────────┘
                            │
                     ┌──────┴───────┐
                     │  MCP Server  │◀──── Claude Desktop / Code
                     │   (stdio)    │
                     └──────────────┘
                            │
                     ┌──────┴───────┐
                     │  Filesystem  │  ← source of truth
                     └──────────────┘
```

The filesystem is the source of truth. SQLite is a derived index — it accelerates search and stores extracted page data, but it can always be rebuilt from the files. A background file watcher picks up changes you make outside the app.

### 关键设计决策

- **文件系统作为真相来源**：SQLite 索引可随时从文件重建，这消除了对数据库持久性的依赖，也允许用户用任何编辑器直接修改 wiki 文件。替代方案是将内容存入数据库，但那样会失去文件系统的可移植性和工具生态兼容性。
- **MCP 而非自定义 API**：通过 Model Context Protocol 让 Claude 接入，而非开发自定义 LLM 集成。这降低了对特定模型的依赖，MCP 是开放标准，未来可支持其他兼容 Agent。
- **SQLite FTS5 而非向量搜索**：本地模式使用 SQLite 的全文搜索而非嵌入向量，牺牲了语义搜索能力但获得了零配置和零依赖的优势。托管版才使用 PGroonga 提供 ranked search。

### 数据流和模块关系

典型工作流（Claude 通过 MCP 写入 wiki 页面）：
1. **用户指令** → Claude Desktop/Code 通过 MCP stdio 连接本地 MCP Server
2. **工具调用** → MCP Server 提供 `guide`/`search`/`read`/`write`/`delete` 工具
3. **写操作** → `write` 工具将 markdown 写入 `wiki/` 目录下的文件系统
4. **索引更新** → 后台文件监听器检测到新文件，更新 SQLite FTS5 索引
5. **前端展示** → Next.js 前端通过 FastAPI 读取索引内容，展示 wiki 页面

反向流（用户在前端浏览）：
1. Next.js 前端发起搜索请求 → FastAPI 查询 SQLite FTS5
2. 返回结果包含文件路径 → 前端可直接读取 `wiki/` 下的 markdown 渲染

### 扩展点与插件机制

- **文档解析器扩展**：通过配置不同的解析器支持新格式。当前支持 PDF（pdf-oxide）、HTML（webmd）、Excel/CSV（openpyxl）、Office（LibreOffice）。添加新格式只需实现提取 + 索引接口。
- **OCR 后端切换**：默认 pdf-oxide 免费，可选 Mistral OCR API 获得更高质量。通过环境变量 `MISTRAL_API_KEY` 切换。
- **托管模式扩展**：支持从本地 SQLite 切换到 Postgres + Supabase auth + S3，适合团队部署。

## 数据安全与隐私

- **数据存储位置**：本地文件系统（`wiki/` + `.llmwiki/`）。托管模式使用用户自有的 Postgres + S3。
- **API Key 管理**：本地模式无需 API key。可选 `MISTRAL_API_KEY` 环境变量用于 PDF OCR。
- **数据传输**：本地模式下数据不离开本机。托管模式下数据存储在用户控制的 Supabase/S3 中。
- **隐私设计**：文件系统级隔离——每个工作区是独立的 MCP server entry，Claude 只能访问该工作区内的文件。

## 横向对比

| 维度 | atomicmemory/llm-wiki-compiler | lucasastorian/llmwiki | Ar9av/obsidian-wiki |
|------|-------------------------------|----------------------|---------------------|
| **定位** | CLI 知识编译器 | 本地应用 + MCP 集成 | Skill-based Agent 框架 |
| **运行时** | Node.js CLI | Next.js + FastAPI + SQLite | 无（Agent IS runtime） |
| **AI 集成** | MCP Server（被动查询） | MCP Server（主动读写） | 多 Agent 兼容（15+） |
| **存储** | 文件系统（sources/ + wiki/） | 文件系统（真相来源）+ SQLite（索引） | Obsidian Vault |
| **多模态** | Web/图片/PDF/转录/YouTube | PDF/HTML/Excel/CSV/Office | 文本/Markdown |
| **搜索** | Chunk 嵌入 + BM25 | SQLite FTS5（本地）/ PGroonga（托管） | QMD 语义搜索（可选） |
| **溯源** | Claim-level 行号范围 | 脚注引用 | 来源文件名 |
| **审查机制** | `compile --review` 队列 | 无（直接写入） | 无（直接写入） |
| **托管能力** | 无 | 支持（Postgres + Supabase + S3） | 无 |
| **安装复杂度** | `npm install -g` | Git clone + Python venv + npm | `npx skills add` |

**解读**：lucasastorian/llmwiki 是三个实现中最接近"完整应用"的——有前端 UI、有后端 API、有文件系统真相来源。atomicmemory 的编译器更适合自动化管线（CI/CD 集成），obsidian-wiki 更适合多 Agent 生态。llmwiki 的独特价值在于本地优先 + 可选托管的架构，以及 Claude 通过 MCP 主动维护 wiki 的交互模式。

## 局限性

- **技术局限**：
  - PDF 表格提取质量差（pdf-oxide 提取散文可靠但表格混乱，需 Mistral OCR 改善）
  - 本地模式无向量搜索，仅支持关键词查询（FTS5 porter stemming）
  - LibreOffice 依赖增加 Office 文件处理门槛
  - 一个工作区对应一个 MCP server，多项目管理需配置多个入口
- **使用场景局限**：
  - 不适合需要语义搜索的复杂查询场景（本地模式）
  - 不适合多人实时协作（无冲突解决机制）
  - 不适合纯云端使用（核心体验依赖本地文件系统）
- **维护状态考量**：
  - 786 stars / 122 forks，创建约 1 个月（2026-04-04），非常新的项目
  - 仅 6 个 open issues，维护活跃（最后更新 2026-05-03）
  - 有 llmwiki.app 托管服务，说明作者有长期运营意图
  - Apache 2.0 许可证降低了社区贡献门槛

## 适用场景

- **最适合**：
  - 需要 AI 辅助维护个人知识库的研究人员（让 Claude 处理枯燥的交叉引用更新）
  - 已有大量 PDF/笔记/表格需要系统化整理的用户
  - 偏好本地优先 + 可选云端同步的隐私敏感用户
  - 熟悉 Claude Desktop/Code 且希望通过 MCP 扩展工作流的用户
- **不适合**：
  - 需要团队实时协作的知识管理（无冲突解决）
  - 完全依赖语义搜索的场景（本地模式只有 FTS5）
  - 不想维护本地服务器的技术成本敏感用户
  - 主要使用移动端或纯浏览器的用户（目前无移动端适配）

## 关联项目

| 项目 | 关系 | 选择建议 |
|------|------|----------|
| atomicmemory/llm-wiki-compiler | 同类实现（不同路线） | 需要自动化管线、多模态摄入、审查队列时选择 |
| Ar9av/obsidian-wiki | 同类实现（不同路线） | 使用多种 AI Agent、零依赖偏好时选择 |
| Karpathy's LLM Wiki pattern | 参考设计 | 理解原始设计理念的起点 |

---

*笔记生成时间：2026-05-03*
*数据来源：GitHub API + README 分析*
*建议下次审查：2026-10-30*
