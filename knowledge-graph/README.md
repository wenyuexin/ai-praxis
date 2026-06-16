# 知识图谱 (Knowledge Graph)

知识图谱的表示、构建、存储、推理、应用与质量管理。

> **入口说明**：第一次进入时，先读本页的分类依据与边界说明建立整体方向；如需查找目录结构、子目录定位或学习路径，转向 [`index.md`](./index.md)。

## 分类依据

Knowledge Graph 目录按"基础 → 构建 → 存储 → 推理 → 应用 → 质量"的全生命周期组织。

## 边界说明

| 内容 | 适合放 KG | 不适合放 KG |
|------|----------|------------|
| 知识表示方法（RDF、OWL、嵌入模型） | 01-foundations | — |
| 本体建模与 Schema 设计 | 01-foundations/ontology-and-schema | — |
| 实体识别、关系抽取等构建技术 | 02-construction | 抽取工具在 LLM 场景下的应用放 `llm/` |
| 图数据库选型与查询 | 03-storage-and-query | 向量数据库放 `rag/02-retrieval/vector-databases/` |
| 知识推理方法（规则推理、神经符号推理） | 04-reasoning | LLM 自身推理能力放 `llm/` |
| LLM + KG 应用 | 05-applications (概念层面) | 具体 RAG 项目（GraphRAG）放 `rag/03-advanced-patterns/graph-rag/` |
| 端到端 KG 工具（Graphify） | 05-applications/graphify/ | — |
| 知识图谱质量与补全 | 06-quality | — |

## 与其他目录的关系

| 本目录内容 | 关联目录 | 说明 |
|-----------|---------|------|
| GraphRAG 检索架构 | [../rag/03-advanced-patterns/graph-rag/](../rag/03-advanced-patterns/graph-rag/) | RAG 视角的图增强检索 |
| LLM 推理 | [../llm/04-serving/](../llm/04-serving/) | LLM 作为抽取/推理工具 |
| 可解释性 | [../llm/06-explainability/](../llm/06-explainability/) | KG 驱动的模型解释 |

---

*最后更新: 2026-05-11*
