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
├── index-rules.md
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
- **我现在要判断 `README.md` / `index.md` / `overview.md` / `landscape.md` / `backlog.md` / `candidates.md` 等元信息文件该怎么分工**：读 [`metadata-files.md`](./metadata-files.md)
- **我现在要修改 README 本身的写法、出现条件、入口说明或导航边界**：读 [`readme-rules.md`](./readme-rules.md)
- **我现在要修改 `index.md` 本身的结构、粒度、父子层导航方式或 stop-line**：读 [`index-rules.md`](./index-rules.md)
- **我现在要修改目录树、查找导航，或判断结构展示是否应从 README 迁到 `index.md`**：先读 [`metadata-files.md`](./metadata-files.md)；涉及 `index.md` 的专项细则时继续读 [`index-rules.md`](./index-rules.md)；README 写法问题再回到 [`readme-rules.md`](./readme-rules.md)
- **我现在要优化搜索 / 导航体验，或判断目录导航与定向搜索应如何配合**：先读 [`metadata-files.md`](./metadata-files.md) 与 [`index-rules.md`](./index-rules.md)；需要能力层说明时继续读 [`../capabilities/navigate.md`](../capabilities/navigate.md)
- **我现在要讨论结构是否长期合理、分类轴是否该调整**：读 [`organization-principles.md`](./organization-principles.md)
- **我现在已经决定做目录迁移、结构收敛或重构执行**：读 [`structure-refactoring-rules.md`](./structure-refactoring-rules.md)
- **我现在要标注 Evidence / Traceability，或判断一条说法该用什么证据状态**：先读 [`evidence-and-traceability.md`](./evidence-and-traceability.md)
- **我现在要处理研究对象、案例研究、辅助材料如何组织**：读 [`research-artifacts.md`](./research-artifacts.md)
- **我现在要判断规则本身该不该改、怎么最小改动**：读 [`meta-rules.md`](./meta-rules.md)

## 何时进入 `intent/` 与 `cases/`

- **主规则已经足够判断大方向，但你想理解为什么这样定、最容易被误解成什么**：进入 `intent/`
- **问题不在抽象原理，而在复杂真实样本上反复误判**：进入 `cases/`
- **如果你已经怀疑某个案例不只是规则复盘，而可能同时涉及 capability / design 边界**：仍先进入 `cases/` 看现有案例；长期升级判断再读 [`../design/cases-layer-design.md`](../design/cases-layer-design.md)

它们都不是默认入口。

`intent/` 按设计问题与复发误读组织，不要求与顶层规则文件一一对应；只有当某组解释值得独立长期保存时，才单独建文件。

### `intent/` 文件列表

- [`intent/metadata-files.md`](./intent/metadata-files.md)：元信息文件总模型，为什么按问题触发而不是按槽位补齐
- [`intent/navigation-files.md`](./intent/navigation-files.md)：`README.md` / `index.md` / `roadmap.md` 的拆分原理与导航入口误读
- [`intent/overview-landscape.md`](./intent/overview-landscape.md)：`overview.md` / `landscape.md` 的原意与常见误读
- [`intent/meta-rules.md`](./intent/meta-rules.md)：规则修改原则、`intent/` 分工与解释层边界
- [`intent/research-artifacts.md`](./intent/research-artifacts.md)：研究产物组织总原则与混合型研究单元
- [`intent/deep-research.md`](./intent/deep-research.md)：研究方法簇中的深度研究脚手架专项
- [`intent/reflections.md`](./intent/reflections.md)：研究方法簇中的反思层专项
- [`intent/traceability-rules.md`](./intent/traceability-rules.md)：Traceability 规则的设计意图
- [`intent/repo-entry-files.md`](./intent/repo-entry-files.md)：仓库入口文件设计原理

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
- [`index-rules.md`](./index-rules.md)：`index.md` 的触发条件、结构模型与导航细则
- [`organization-principles.md`](./organization-principles.md)：知识树设计与长期组织原则
- [`structure-refactoring-rules.md`](./structure-refactoring-rules.md)：目录重构与结构演进规则
- [`evidence-and-traceability.md`](./evidence-and-traceability.md)：Evidence + Traceability 组合入口
- [`evidence-rules.md`](./evidence-rules.md)：Evidence 规则
- [`traceability-rules.md`](./traceability-rules.md)：Traceability 规则

## 最小分工

- `README.md`：回答规则层是什么、为什么存在、与相邻层如何分工
- `index.md`：回答进入规则层后，如果我已经知道自己要处理哪类问题，该去哪篇规则
- `intent/`：解释规则背后的设计原理与常见误读
- `cases/`：沉淀高价值边界案例、误判复盘与对象级样本；若个别案例已出现 cross-layer signal，先在案例文件内显式标出，不急着立即迁出规则层

## 边界说明

- 本文件不重复规则层专项设计；为什么要拆成主规则、`intent/`、`cases/`，读 [`../design/contributing-design.md`](../design/contributing-design.md)
- 本文件不展开单条规则细节；更细的执行边界继续留在对应专题规则文件
- 如果规则层继续增长，本文件应优先继续补任务分流，而不是退化成另一份 README
