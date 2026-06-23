# docs Index

本文件是 `docs/` 的查找导航页，回答：**进入 `docs/` 之后，如果我已经知道自己要找的是哪类问题，接下来该去哪。**

它不代替 `docs/README.md` 的目录说明职责，也不代替 `docs/contributing/README.md` 与 `docs/capabilities/README.md` 各自目录内的细分索引职责。

## 目录骨架

```text
docs/
├── capabilities/   # 公共能力文档：回答系统提供什么能力、能力之间如何衔接
├── contributing/   # 规则目录：回答维护仓库时应遵守什么规则、按什么流程执行
├── design/         # 专项设计文档：解释系统为什么这样分层、当前设计如何演化
├── templates/      # 文档模板
└── test/           # Markdown / Mermaid 渲染测试材料
```

## 按问题找入口

- **我还拿不准仓库里有哪些文档类型、它们分别干什么**：先读 [`repo-map.md`](./repo-map.md)
- **我想理解 `docs/` 这个目录本身是什么、为什么这样分层**：先读 [`README.md`](./README.md)
- **我想知道这套仓库作为 knowledge agent system 提供了哪些公共能力**：先读 [`capabilities/README.md`](./capabilities/README.md)
- **我想理解 capability layer 自己的边界、增长方式与联动更新面**：读 [`capabilities/meta.md`](./capabilities/meta.md)
- **我已经进入能力层，想按能力问题继续找到具体文档**：读 [`capabilities/index.md`](./capabilities/index.md)
- **我想知道维护仓库时应该遵守什么规则、该按什么流程执行**：先读 [`contributing/README.md`](./contributing/README.md)
- **我已经进入规则层，想按任务继续找到具体规则文件**：读 [`contributing/index.md`](./contributing/index.md)
- **我想理解为什么会拆出 `capabilities/`，以及规则层和能力层如何协同**：读 [`design/system-design.md`](./design/system-design.md)
- **我想理解为什么“外部临时材料进入稳定知识层”更像一条 skill-like workflow**：读 [`design/research-ingestion-design.md`](./design/research-ingestion-design.md)
- **我想判断复杂案例长期应放在 contributing、capabilities 还是独立 case layer**：读 [`design/cases-layer-design.md`](./design/cases-layer-design.md)
- **我想理解为什么需要 `docs/contributing/` 这一层，以及它内部为什么还要继续分层**：读 [`design/contributing-design.md`](./design/contributing-design.md)
- **我想找文档模板**：进入 `templates/`
- **我想看 Markdown / Mermaid 等渲染测试材料**：进入 `test/`

## 按任务找入口

- **我现在要找的是规则、约束、执行流程**：进入 [`contributing/README.md`](./contributing/README.md)
- **我现在要找的是公共能力、判断框架、能力之间如何衔接**：进入 [`capabilities/README.md`](./capabilities/README.md)
- **我现在要判断 capability layer 自己该怎么长、边界怎么守**：读 [`capabilities/meta.md`](./capabilities/meta.md)
- **我现在要理解 `docs/` 为什么这样分层，以及 rules / capability layer 如何协同**：读 [`design/system-design.md`](./design/system-design.md)
- **我现在要找文档模板或渲染测试材料**：分别进入 `templates/` 或 `test/`

## 最小分工

- `repo-map.md`：回答仓库里有哪些文档类型、它们分别承担什么组织功能
- `README.md`：回答 `docs/` 是什么、为什么存在、下面有哪些稳定层
- `index.md`：回答进入 `docs/` 后，如果我已经知道自己要找的是规则、能力、设计还是支撑材料，该去哪一层
- `contributing/README.md`：回答规则目录内部怎么继续分流
- `capabilities/README.md`：回答能力目录内部怎么继续分流

## 边界说明

- 本文件不展开单条规则细节；更细的执行边界继续留在 `docs/contributing/`
- 本文件不展开单项能力细分入口；能力目录内部继续由 `docs/capabilities/README.md` 承接
- 本文件不解释每项能力的完整设计理由；能力层设计背景继续留在 `design/system-design.md`
- 如果 `docs/` 后续继续增长，本文件应优先继续补层级分流，而不是退化成另一份 `README.md` 或子目录索引页
