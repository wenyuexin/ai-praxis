# 贡献与仓库建设规则

本目录沉淀与仓库建设、文档维护、证据分级、可追溯性相关的规则。

它不保存具体机器学习或 Agentic 知识内容；知识内容应放在对应主题目录中，例如 `agentic/`、`llm/`、`training-infra/` 等。

## 职责边界

- [`CONTRIBUTING.md`](../../CONTRIBUTING.md)：仓库级贡献入口，保留必须遵守的核心规则。
- [`AGENTS.md`](../../AGENTS.md)：AI 协作者入口，提炼执行时必须优先遵守的规则。
- `docs/contributing/`：承接更细的文档治理、证据、可追溯性和维护流程规则。
- 主题目录：保存经过整理后的知识正文、综述、backlog、candidates 和 conflict 记录。
- `docs/templates/`：保存仓库文档模板。
- `docs/test/`：保存 Markdown、Mermaid 等文档渲染测试材料。

## 目录结构

```text
docs/contributing/
├── README.md                         # 本目录索引：按任务找到规则文件
├── meta-rules.md                     # 规则怎么改：适用时机、最小改动、落位边界
├── documentation-workflow.md         # 材料处理、分流、落位与回流流程
├── research-artifacts.md             # 子领域研究、案例研究与研究辅助材料组织
├── metadata-files.md                 # README / overview / landscape / backlog 等元信息文件模型
├── readme-rules.md                   # README 出现条件、展开深度与导航边界
├── organization-principles.md        # 知识树设计、分类轴与长期组织原则
├── structure-refactoring-rules.md    # 目录重构、导航收敛与结构演进规则
├── evidence-and-traceability.md      # Evidence + Traceability 组合执行入口
├── evidence-rules.md                 # Claim / Evidence / Status / 标注方式
├── traceability-rules.md             # Source / Decision / Placement / Gap 规则
└── intent/                           # 非默认入口：保存规则设计意图与长解释
```

## 当前规则入口

当前仓库级强约束仍以根目录 [`CONTRIBUTING.md`](../../CONTRIBUTING.md) 为准，AI 协作者入口见 [`AGENTS.md`](../../AGENTS.md)。本目录承接更细的规则设计；进入本目录后，可用本文件的职责索引定位对应专题文件；当 `CONTRIBUTING.md` 中某类规则继续增长时，应逐步迁移到本目录下的专题文件，并在 `CONTRIBUTING.md` 中保留摘要和链接。

本目录文件不使用数字编号，也不要求从上到下通读。推荐阅读方式是：先根据当前任务类型选择最相关的规则文件；如果不确定该读哪一个，再用本文件定位规则入口。只有需要 Evidence、Traceability、README 或元信息细则时，再进入对应细则文件。

## 先读哪篇

可按下面的职责边界快速定位：

- **为什么这样组织知识**：先读 [文档组织原则](./organization-principles.md)
- **已经决定重构后怎么低风险执行**：先读 [结构重构规则](./structure-refactoring-rules.md)
- **README 应该展开到几级、如何导航**：先读 [README 规则](./readme-rules.md)
- **`README.md` / `overview.md` / `landscape.md` / `backlog.md` 等文件各自做什么**：先读 [元信息文件模型](./metadata-files.md)；如果想看单篇文档的文档头信息与尾注信息如何摆放，也看该文附录
- **研究对象、案例研究、辅助材料如何组织**：先读 [研究产物组织规则](./research-artifacts.md)
- **判断某个对象是否应升格为独立目录（非迁移执行问题）**：先读 [研究产物组织规则](./research-artifacts.md) 和 [文档组织原则](./organization-principles.md)
- **新材料、外部调研、`temp/` 内容如何判断与分流**：先读 [材料处理与文档增改流程](./documentation-workflow.md)
- **证据状态和轻量链路如何一起使用**：先读 [Evidence 与 Traceability 工作流](./evidence-and-traceability.md)

如果主规则已经读过，但仍需要误判复盘、边界案例或更长解释，再看 [`cases/README.md`](./cases/README.md) 或 `intent/` 下的对应释义文档；这里都不是默认入口。

已拆分规则：

- [规则的规则](./meta-rules.md)（按需）：定义什么时候改规则、规则如何最小改动，以及专题规则与 meta-rule 的边界。
- [`intent/meta-rules.md`](./intent/meta-rules.md)（按需）：解释为什么仓库需要 meta-rules，以及为什么只有短规则条文还不足以守住规则原意。
- [研究产物组织规则](./research-artifacts.md)：定义子领域研究、具体案例研究、研究辅助材料的组织方式。
- [材料处理与文档增改流程](./documentation-workflow.md)：定义新材料、外部调研、`temp/` 内容和正文修改的判断、落位与回流流程。
- [Evidence 与 Traceability 工作流](./evidence-and-traceability.md)：组合执行入口，只说明正式 Evidence + 轻量 Traceability 如何一起使用。
- [Evidence 规则](./evidence-rules.md)：定义 Claim、Evidence 状态、来源类型、标注密度、正文标注方式、主题级 evidence registry 的启用条件，以及研究开源代码库时的版本基线要求。
- [Traceability 规则](./traceability-rules.md)：定义 Source / Decision / Placement / Gap 四个轻量链路字段，以及代码库研究中的 `Version Basis` / `Observed At` / `Scope` / `Drift Risk`。
- [`intent/traceability-rules.md`](./intent/traceability-rules.md)（按需）：解释为什么当前 Traceability 既不能做成重型 lineage 系统，也不能退化成一句模糊备注，以及为什么源码观察必须带版本链路字段。
- [README 规则](./readme-rules.md)：定义 README 出现条件、目录树展开深度、导航边界和类型化结构建议。
- [文档组织原则](./organization-principles.md)：定义知识树设计、分类轴一致性、长期主义、材料目录与知识目录分离等上位组织原则。
- [结构重构规则](./structure-refactoring-rules.md)：定义目录改名、结构迁移、README 导航收敛、材料目录治理与结构经验沉淀原则。
- [元信息文件模型](./metadata-files.md)：定义 `README.md`、`overview.md`、`landscape.md`、`backlog.md`、`candidates.md`、`roadmap.md`、`conflict.md` 等文件的职责边界。
- [`intent/metadata-files.md`](./intent/metadata-files.md)（按需）：解释为什么 `README.md`、`overview.md`、`landscape.md` 不能被当成一组对称槽位来理解，尤其解释 `overview` 的独立可读性与 `landscape` 的结构化文档体系构建职责。
