# Knowledge Graph Index

> 如果你是第一次进入本目录，先读 [`README.md`](./README.md) 建立方向感；如果你已经知道想找什么，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `overview.md` 的整体理解主线。

## 目录结构

```
knowledge-graph/
├── 01-foundations/                  # 基础
│   ├── symbolic-representation/     # 符号表示
│   │   ├── rdf-and-owl/
│   │   └── property-graphs/
│   ├── embedding-based-representation/ # 嵌入表示
│   │   ├── translational-models/
│   │   ├── bilinear-models/
│   │   └── neural-models/
│   └── ontology-and-schema/         # 本体与Schema
│
├── 02-construction/                 # 知识构建
│   ├── named-entity-recognition/    # 实体识别
│   ├── relation-extraction/         # 关系抽取
│   ├── entity-linking/              # 实体链接
│   ├── event-extraction/            # 事件抽取
│   └── knowledge-fusion/            # 知识融合
│       ├── entity-alignment/
│       └── conflict-resolution/
│
├── 03-storage-and-query/            # 存储与查询
│   ├── graph-databases/
│   ├── query-languages/
│   └── distributed-storage/
│
├── 04-reasoning/                    # 知识推理
│   ├── rule-based-reasoning/
│   ├── embedding-based-reasoning/
│   ├── neuro-symbolic-reasoning/
│   └── temporal-and-spatial-reasoning/
│
├── 05-applications/                 # 应用
│   ├── question-answering/
│   ├── recommender-systems/
│   ├── dialogue-systems/
│   ├── domain-specific-kg/
│   ├── graphify/                    # 代码项目知识图谱工具
│   └── synthetic-data/              # 合成数据生成
│
└── 06-quality/                      # 质量与演化
    ├── knowledge-completion/
    ├── error-detection/
    └── kg-versioning/
```

## 学习路径

**入门阶段**
- `01-foundations/` — 了解知识图谱的基本概念与表示方法
- `02-construction/` — 掌握实体识别和关系抽取

**构建阶段**
- `03-storage-and-query/` — 图数据库使用
- `04-reasoning/` — 知识推理方法

**应用阶段**
- `05-applications/` — LLM 与知识图谱结合
- `06-quality/` — 知识图谱质量与演化

## 开源仓库与工具存放指南

| 仓库功能 | 放入目录 | 示例 |
|---------|---------|------|
| 知识表示框架/嵌入库 | `01-foundations/` | RDFlib, PyKEEN |
| 抽取/融合/构建工具 | `02-construction/` | OpenIE, DeepKE |
| 代码项目知识图谱工具 | `05-applications/graphify/` | Graphify |
| 图数据库/查询引擎 | `03-storage-and-query/` | Neo4j, NebulaGraph |
| 推理引擎 | `04-reasoning/` | RLvLR, KGEM |
| 应用框架 | `05-applications/` | KG4LLM |
| 质量评估/补全工具 | `06-quality/` | 知识图谱质量评估套件 |

## 按子目录定位

- `01-foundations/`：知识表示方法（符号表示、嵌入表示）与本体建模
- `02-construction/`：从非结构化数据中抽取实体和关系
- `03-storage-and-query/`：图数据库存储与查询语言
- `04-reasoning/`：基于已有知识推导新知识
- `05-applications/`：KG 在下游任务中的应用
- `06-quality/`：知识图谱的维护、补全与评估
