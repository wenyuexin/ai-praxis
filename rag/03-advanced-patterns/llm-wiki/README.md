# LLM Wiki

LLM Wiki：将外部资料在摄入时编译为可持续维护的结构化知识库的高级知识工程范式。

## 定位

本目录在 `rag/03-advanced-patterns/` 下，与 graph-rag、self-reflective-rag 等并列。
与那些范式一样，LLM Wiki 也回答"如何把外部知识转化为更可用的中间知识层"——但它走了一条不同的路：不再每次查询时临时检索与拼接，而是在摄入阶段就把资料编译成一个持续维护、可交叉引用的 wiki。

因此它放在 "高级范式" 下是合适的：它是一种完整的范式反转，而不只是某种检索增强。

## 边界

适合放入本目录的内容：

- LLM Wiki / knowledge compiler / persistent wiki 这一类高级知识编译范式
- Karpathy 模式及其直接演化实现
- 围绕 ingest / compile / query / lint / schema 展开的对象研究与机制分析
- 与该范式直接相关的代表项目、插件、CLI、Agent-native 实现、维护工具

不适合放入本目录的内容：

- 通用 Agent 框架、Skill 系统、Tool registry：主归属 `../../../agentic/`
- 通用个人知识管理工具或 Obsidian 生态综述：应放更贴近对应主题的位置
- 一般 RAG 基础、检索算法、向量数据库：主归属 `../../01-foundations/` 或 `../../02-retrieval/`

## 目录结构

```text
llm-wiki/
├── project-studies/    # 代表实现与对象研究
└── notes/              # 原始资料与研究辅助材料
```

## 如何阅读

- 想快速建立整体认知：从主文入口进入
- 想理解本目录的板块划分和边界判断：查阅结构研究文
- 想比较不同实现路线：进入 `project-studies/`
- 底层有 `notes/` 存放原始资料与参考材料

---

*最后更新: 2026-06-13*
