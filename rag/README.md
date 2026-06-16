# RAG (检索增强生成)

检索增强生成：通过外部知识检索提升 LLM 的事实准确性和时效性。

> **入口说明**：第一次进入时，先读本页的分类依据与边界说明建立整体方向；如需查找目录结构、子目录定位或学习路径，转向 [`index.md`](./index.md)。

## 分类依据

RAG 目录按"基础 → 检索 → 高级范式 → 评估 → 生产"组织。检索是 RAG 的差异化核心；03 中的 llm-wiki 等方向虽暂列于此，但关注的是查询前的知识预编译和持久化，而非经典的查询时检索拼接。

## 边界说明

| 内容 | 归属 | 说明 |
|------|------|------|
| Prompt 工程 | [../llm/04-serving/prompt-engineering/](../llm/04-serving/prompt-engineering/) | 通用技术 |
| LLM 推理优化 | [../llm/04-serving/](../llm/04-serving/) | 通用推理优化 |
| 知识图谱通用构建方法 | [../knowledge-graph/02-construction/](../knowledge-graph/02-construction/) | 实体抽取、关系抽取等基础方法 |
| 知识图谱在 RAG 中的应用 | `03-advanced-patterns/graph-rag/` | GraphRAG 等方案集中在此 |
| Agent 体系知识 | [../agentic/](../agentic/) | Agent 驱动 RAG 在 `03-advanced-patterns/agentic-rag/` |

## 与其他目录的关系

- [LLM 推理](../llm/04-serving/) — 生成模块优化
- [知识图谱](../knowledge-graph/) — 结构化知识源
- [LLM 评估](../llm/05-evaluation/) — 通用评估方法
- [Agentic AI](../agentic/) — Agent 驱动的 RAG 范式

---

*最后更新: 2026-05-11*
