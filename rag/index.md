# RAG Index

> 如果你是第一次进入本目录，先读 [`README.md`](./README.md) 建立方向感；如果你已经知道想找什么，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `overview.md` 的整体理解主线。

## 目录结构

```
rag/
├── 01-foundations/                  # 基础
│   ├── what-is-rag/                 # RAG概念与动机
│   ├── naive-rag/                   # 朴素RAG
│   ├── rag-pipeline-overview/       # 管线总览
│   ├── context-integration/         # 上下文构建与注入
│   ├── generation-strategies/       # 生成策略
│   └── evaluation-of-generation/    # 生成质量评估
│
├── 02-retrieval/                    # 检索
│   ├── chunking-strategies/         # 分块策略
│   │   ├── fixed-size/
│   │   └── semantic-chunking/
│   ├── embedding-models/            # 嵌入模型
│   │   ├── dense-embeddings/
│   │   └── sparse-embeddings/
│   ├── vector-databases/            # 向量数据库
│   └── advanced-retrieval/          # 高级检索
│       ├── hybrid-retrieval/
│       ├── multi-hop-retrieval/
│       ├── recursive-retrieval/
│       ├── reranking/
│       └── page-index/
│
├── 03-advanced-patterns/            # 高级范式（含代表实现）
│   ├── graph-rag/                   # 图增强RAG（含Microsoft GraphRAG）
│   ├── self-reflective-rag/         # 自反思RAG（含Self-RAG、Corrective RAG）
│   ├── agentic-rag/                 # Agent驱动RAG
│   ├── modular-rag/                 # 模块化RAG
│   ├── multimodal-rag/              # 多模态RAG
│   └── llm-wiki/                    # LLM Wiki / 知识编译范式
│
├── 04-evaluation/                   # 评估
│   ├── end-to-end-metrics/
│   └── public-benchmarks/
│
└── 05-production/                   # 生产与生态
    ├── frameworks/
    ├── caching-and-scaling/
    └── security-and-privacy/
```

## 按问题找入口

- 想看 RAG 基础概念、朴素 RAG 管线或生成策略：先看 `01-foundations/`
- 想看分块策略、嵌入模型、向量数据库或高级检索方法：先看 `02-retrieval/`
- 想看 GraphRAG、Self-RAG、Agentic RAG、Modular RAG 或 LLM Wiki：先看 `03-advanced-patterns/`
- 想看端到端评估指标或公开基准：先看 `04-evaluation/`
- 想看 RAG 框架、缓存与扩展或安全与隐私：先看 `05-production/`

## 按阅读目标找入口

- 想了解 RAG 概念与动机：`01-foundations/what-is-rag/`
- 想对比高级检索方法（hybrid / multi-hop / reranking）：`02-retrieval/advanced-retrieval/`
- 想查具体框架选型（LlamaIndex, LangChain 等）：`05-production/frameworks/`
- 想查评估基准（RAGAS 等）：`04-evaluation/`

## 开源仓库与工具存放指南

| 仓库/工具 | 放入目录 | 说明 |
|-----------|---------|------|
| 具体 RAG 范式的代表实现 | `03-advanced-patterns/` 对应子目录 | GraphRAG、HippoRAG 等与范式放在一起 |
| 通用 RAG 框架（LlamaIndex, LangChain RAG 等） | `05-production/frameworks/` | 工程化框架 |
| RAG 评估套件（RAGAS, TruLens 等） | `04-evaluation/` | 评估工具 |

## 按子目录定位

- `01-foundations/`：RAG 概念、朴素 RAG、管线总览、上下文构建、生成策略、生成质量评估
- `02-retrieval/`：分块策略、嵌入模型、向量数据库、高级检索方法
- `03-advanced-patterns/`：图增强、自反思、Agent 驱动、模块化、多模态等高级范式，含代表实现
- `04-evaluation/`：端到端指标、公开基准
- `05-production/`：框架、缓存与扩展、安全与隐私
