# Retrieval

RAG 的检索模块，覆盖文档分块、嵌入、索引、排序全链路。

## 子目录

| 目录 | 说明 |
|------|------|
| chunking-strategies/ | 文档分块策略：固定大小、语义分块、递归分块 |
| embedding-models/ | 嵌入模型：稠密嵌入、稀疏嵌入、多模态嵌入 |
| vector-databases/ | 向量数据库：选型对比、索引结构、性能调优 |
| advanced-retrieval/ | 高级检索：混合搜索、递归检索、多跳检索、重排序，以及 PageIndex 这类推理型检索框架 |

## 相关资源

- [评估指标](../04-evaluation/) — 检索质量的评估方法

---

*最后更新: 2026-05-11*