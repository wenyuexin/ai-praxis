# Contributing Index

本文件是 `docs/contributing/` 的查找导航页，回答：**进入规则层之后，如果我已经知道自己要处理的是哪类维护问题，接下来该去哪篇规则。**

它不代替 `README.md` 的目录说明职责，也不代替各专题规则文件本身。

## 目录骨架

```text
docs/contributing/
├── README.md
├── index.md
├── meta-rules.md
├── documentation-workflow.md
├── research-artifacts.md
├── metadata-files.md
├── readme-rules.md
├── organization-principles.md
├── structure-refactoring-rules.md
├── evidence-and-traceability.md
├── evidence-rules.md
├── traceability-rules.md
├── intent/
└── cases/
```

## 按任务找入口

- **我现在要处理新材料、外部调研、`temp/` 内容或正文回流**：读 [`documentation-workflow.md`](./documentation-workflow.md)
- **我现在要判断 `README.md` / `overview.md` / `landscape.md` / `backlog.md` / `candidates.md` 等元信息文件该怎么分工**：读 [`metadata-files.md`](./metadata-files.md)
- **我现在要修改 README、目录树或入口导航**：读 [`readme-rules.md`](./readme-rules.md)
- **我现在要讨论结构是否长期合理、分类轴是否该调整**：读 [`organization-principles.md`](./organization-principles.md)
- **我现在已经决定做目录迁移、结构收敛或重构执行**：读 [`structure-refactoring-rules.md`](./structure-refactoring-rules.md)
- **我现在要标注 Evidence / Traceability，或判断一条说法该用什么证据状态**：先读 [`evidence-and-traceability.md`](./evidence-and-traceability.md)
- **我现在要处理研究对象、案例研究、辅助材料如何组织**：读 [`research-artifacts.md`](./research-artifacts.md)
- **我现在要判断规则本身该不该改、怎么最小改动**：读 [`meta-rules.md`](./meta-rules.md)

## 何时进入 `intent/` 与 `cases/`

- **主规则已经足够判断大方向，但你想理解为什么这样定、最容易被误解成什么**：进入 `intent/`
- **问题不在抽象原理，而在复杂真实样本上反复误判**：进入 `cases/`

它们都不是默认入口。

## 代表样本

如果你是第一次进入 `cases/`，可优先看这些代表样本：

- [`cases/pageindex-hybrid-study-unit.md`](./cases/pageindex-hybrid-study-unit.md)
- [`cases/interdisciplinarity-readme-index-roadmap.md`](./cases/interdisciplinarity-readme-index-roadmap.md)
- [`cases/learning-materials-readme-backlog.md`](./cases/learning-materials-readme-backlog.md)
- [`cases/training-infra-landscape-before-overview.md`](./cases/training-infra-landscape-before-overview.md)
- [`cases/agent-system-modeling-readme-candidates.md`](./cases/agent-system-modeling-readme-candidates.md)

## 已稳定的专题规则

- [`meta-rules.md`](./meta-rules.md)：规则如何修改
- [`documentation-workflow.md`](./documentation-workflow.md)：材料处理、分流、落位与回流流程
- [`research-artifacts.md`](./research-artifacts.md)：研究产物组织方式
- [`metadata-files.md`](./metadata-files.md)：元信息文件职责边界
- [`readme-rules.md`](./readme-rules.md)：README 出现条件、展开深度与导航边界
- [`organization-principles.md`](./organization-principles.md)：知识树设计与长期组织原则
- [`structure-refactoring-rules.md`](./structure-refactoring-rules.md)：目录重构与结构演进规则
- [`evidence-and-traceability.md`](./evidence-and-traceability.md)：Evidence + Traceability 组合入口
- [`evidence-rules.md`](./evidence-rules.md)：Evidence 规则
- [`traceability-rules.md`](./traceability-rules.md)：Traceability 规则

## 最小分工

- `README.md`：回答规则层是什么、为什么存在、与相邻层如何分工
- `index.md`：回答进入规则层后，如果我已经知道自己要处理哪类问题，该去哪篇规则
- `intent/`：解释规则背后的设计原理与常见误读
- `cases/`：沉淀高价值边界案例、误判复盘与对象级样本

## 边界说明

- 本文件不重复规则层专项设计；为什么要拆成主规则、`intent/`、`cases/`，读 [`../design/contributing-design.md`](../design/contributing-design.md)
- 本文件不展开单条规则细节；更细的执行边界继续留在对应专题规则文件
- 如果规则层继续增长，本文件应优先继续补任务分流，而不是退化成另一份 README
