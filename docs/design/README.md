# Design

本目录是 `docs/design/` 的入口，只回答：**这里有哪些设计侧文档，它们分别解决什么问题。**

当前设计层文件不多，所以这份 README 保持最小，不展开系统设计细节，也不替代具体设计文档本身。

## 当前文件

- `system-design.md`：系统级设计主文，解释仓库为什么被组织成一个 knowledge agent system。
- `capability-design.md`：capability layer 的专项设计文档，说明当前核心能力、承接文档与补强状态。
- `research-ingestion-design.md`：研究型输入吸收链的专项设计文档，解释为什么它更接近一条 skill-like workflow，以及它与 rules / capabilities / research artifacts 的分层关系。
- `cases-layer-design.md`：case layer 的专项设计文档，分析 `docs/contributing/cases/`、潜在 `docs/capabilities/cases/` 与潜在独立 `docs/cases/` 的长期边界。
- `../capabilities/meta.md`：capability layer 自我约束文档，说明能力层的边界、增长方式与联动更新面。
- `contributing-design.md`：`docs/contributing/` 这一层的专项设计文档，解释 rules / methods layer 为什么这样分层。
- `navigation-design.md`：导航 / 自导航子系统的专项设计文档，解释索引与查找为何应作为一套受治理的系统，以及如何搭建、走通并校验所有链路。
- `evolution-design.md`：知识与方法论协同进化的专项设计文档，解释为什么积累知识与沉淀方法论是一对协同进化过程，以及其节奏、评价尺度与实践/理论两种来源。
- `runtime-development-design.md`：运行态 / 开发态双模式的专项设计文档，解释两态为何必须显式分离、与控制面 / 数据面的关系，以及为什么开发态应在 `docs/` 层有独立入口。
- `auxiliary-layers-design.md`（初稿）：构造辅助层家族的**视角与设计理由**——把 `temp/`、对象 `notes/` 与 `backlog/candidates/conflict/roadmap` 元信息放到一层横切视角下，解释它们为何相关、为何不物理嵌套。**运行态落位序**见 [`../contributing/rules/documentation-workflow.md`](../contributing/rules/documentation-workflow.md) §9「通用落位序」，**能力侧分流**见 [`../capabilities/place.md`](../capabilities/place.md)。

以上为**系统架构文档**（描述系统怎么运作）。一次性**设计决策**（某时点的具体选择 + 备选 + 理由）单独收在子目录：

- `decisions/`：一次性设计决策记录（ADR 式），当前含 `repo-naming/`（为本仓库选名）。详见 [`decisions/README.md`](./decisions/README.md)。

## 阅读建议

- 想理解这个仓库为什么这样分层：读 [`system-design.md`](./system-design.md)
- 想横向看 capability layer 当前有哪些能力位、由什么承接：读 [`capability-design.md`](./capability-design.md)
- 想理解为什么“从外部临时材料到稳定知识层”的处理已经不只是规则，而更接近一条 skill-like workflow：读 [`research-ingestion-design.md`](./research-ingestion-design.md)
- 想判断复杂案例长期应留在 `docs/contributing/cases/`、进入潜在 `docs/capabilities/cases/`，还是升级为独立 `docs/cases/`：读 [`cases-layer-design.md`](./cases-layer-design.md)
- 想理解 capability layer 自己如何守边界、如何增长、变动后要同步哪些面：读 [`../capabilities/meta.md`](../capabilities/meta.md)
- 想理解为什么需要 `docs/contributing/` 这一层，以及它为什么继续拆成主规则、`intent/`、`cases/`：读 [`contributing-design.md`](./contributing-design.md)
- 想理解索引与查找为何应体系化、以及仓库如何只靠一个常驻根（`AGENTS.md`）+ 其余全靠 pull 实现自导航：读 [`navigation-design.md`](./navigation-design.md)
- 想理解为什么积累知识与沉淀方法论是一对协同进化过程、以及这个循环怎么跑：读 [`evolution-design.md`](./evolution-design.md)
- 想理解运行态与开发态为何要分开、以及开发态为什么需要 `docs/DEVELOPING.md` 这样的独立入口：读 [`runtime-development-design.md`](./runtime-development-design.md)
- 想理解 `temp/` / `notes/` / 元信息文件作为『构造辅助层』为何相关、为何不物理嵌套：读 [`auxiliary-layers-design.md`](./auxiliary-layers-design.md)（初稿）；运行态落位序见 [`../contributing/rules/documentation-workflow.md`](../contributing/rules/documentation-workflow.md) §9「通用落位序」

## 边界说明

- 本 README 只做设计层入口，不重复系统设计正文。
- 本层顶层的 `*.md` 只承载**系统架构**（描述系统怎么运作）；**一次性设计决策**（如仓库命名）放 [`decisions/`](./decisions/)，不与架构文档平级混放。判据：描述系统怎么运作 → 架构；记录一次选择 → 决策。
- 若未来设计层继续增长，再按实际需要决定是否补 `index.md` 或更多专项设计文档。
