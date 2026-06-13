# Advanced Patterns

RAG 高级范式及其代表实现，范式与实例放在一起便于对照理解。

## 定位

本目录收录 RAG 的通用增强模式与架构范式，每个范式目录下同时包含该范式的代表实现（如 GraphRAG 项目笔记放在 `graph-rag/` 下）。

## 子目录

| 目录 | 说明 |
|------|------|
| graph-rag/ | 图增强 RAG：用知识图谱增强检索（含 Microsoft GraphRAG） |
| self-reflective-rag/ | 自反思 RAG：检索-生成-修正闭环（含 Self-RAG、Corrective RAG） |
| agentic-rag/ | Agent 驱动的 RAG：自主规划与工具调用 |
| modular-rag/ | 模块化 RAG：组件解耦与灵活组合 |
| multimodal-rag/ | 多模态 RAG：跨模态检索与生成 |
| llm-wiki/ | LLM Wiki：在摄入时将资料编译为持久化、可维护的结构化知识库；相关支撑框架可被比较，但 `PageIndex` 这类检索框架主条目应回到 `02-retrieval/` |

---

*最后更新: 2026-05-11*
