# 仓库组织与写作规范

本文件定义仓库的组织规则与笔记写作规范。更细的仓库建设、文档治理、证据分级与可追溯性规则，沉淀在 [`docs/contributing/`](./docs/contributing/)；本文件保留仓库级强约束和入口说明。AI 协作者入口见 [`AGENTS.md`](./AGENTS.md)。Evidence 与 Traceability 的统一工作流见 [`docs/contributing/evidence-and-traceability.md`](./docs/contributing/evidence-and-traceability.md)，细则见 [`evidence-rules.md`](./docs/contributing/evidence-rules.md) 和 [`traceability-rules.md`](./docs/contributing/traceability-rules.md)。

## 快速导读

如果你带着具体任务阅读，优先按任务入口跳转：

| 你要做什么 | 优先阅读 | 目的 |
|---|---|---|
| 新增一篇笔记 | [文档增改流程](./docs/contributing/documentation-workflow.md) 第 1、2、4 节 + 第二章；证据规则见 [Evidence 规则](./docs/contributing/evidence-rules.md)，链路规则见 [Traceability 规则](./docs/contributing/traceability-rules.md) | 先判断材料是否适合写成正文，再确定目录、命名和证据状态 |
| 新增一个子目录 | [文档增改流程](./docs/contributing/documentation-workflow.md) 第 5 节 + 第一章、第三章；README 细则见 [README 规则](./docs/contributing/readme-rules.md) | 先确定目录定位、层级和 README 要求 |
| 处理外部 AI 调研、网页搜索或临时笔记 | [文档增改流程](./docs/contributing/documentation-workflow.md) 第 3、8、10 节；统一流程见 [Evidence 与 Traceability 工作流](./docs/contributing/evidence-and-traceability.md) | 先做证据分级和材料分流，不直接写正文 |
| 记录主题缺口或待研究对象 | 第四章 L4/L5 + [文档增改流程](./docs/contributing/documentation-workflow.md) 第 2、9 节；文件职责见 [元信息文件模型](./docs/contributing/metadata-files.md) | 区分问题缺口与研究对象 |
| 处理术语、事实或结论冲突 | 第四章 L7 + [文档增改流程](./docs/contributing/documentation-workflow.md) 第 7 节；冲突文件职责见 [元信息文件模型](./docs/contributing/metadata-files.md) | 先记录冲突，再决定是否修改正文 |
| 查某个元信息文件的职责 | 第四章；细则见 [元信息文件模型](./docs/contributing/metadata-files.md) | 理解 `README.md`、`overview.md`、`backlog.md` 等文件边界 |
| 查仓库建设规则的承接位置 | [`docs/contributing/`](./docs/contributing/) | 区分知识正文与贡献、治理、证据、可追溯性约束 |
| 优化单篇文档表达 | 第五章 | 调整结构、公式、表格或论文笔记写法 |

如果只记一条默认策略：**拿不准时，先写元信息文件，不要着急写正文。**

---

## 一、层级设计

仓库按纵向技术栈分层，辅以横向跨学科层：

| 层级 | 定位 | 目录 |
|---|---|---|
| 基础方法层 | 经典算法与理论基础 | traditional-ml、deep-learning、reinforce-learning |
| 核心领域层 | 仓库主线 | llm、cv、knowledge-graph |
| 应用与集成层 | 技术落地与组合 | rag、agentic、world-models、embodied-intelligence |
| 基础设施层 | 训练与工程支撑 | training-infra |
| 跨学科层 | 横向交叉视角 | interdisciplinarity |
| 支撑资源 | 辅助资料、仓库建设规则与文档模板 | learning-materials、docs |
| 仓库建设规则 | 贡献、文档治理、证据分级与可追溯性约束 | [`docs/contributing`](./docs/contributing/) |

## 二、目录与文件命名

- 层级称呼：仓库根目录不计入编号层级；`agentic/`、`llm/`、`rag/` 这类仓库根目录下的直接子目录称为一级目录，也可称顶层领域目录；`agentic/01-foundations/`、`agentic/06-frameworks-and-tools/` 这类一级目录下的子目录称为二级目录；`agentic/06-frameworks-and-tools/02-coding-tools/` 这类二级目录下的子目录称为三级目录，依此类推。
- 一级目录（顶层领域目录）：英文短名，语义自解释，不使用数字前缀；一级目录表达领域分区，不表达线性阅读顺序。
- 二级及以下目录：只有当同一组目录存在明确阅读顺序、阶段顺序、流程顺序或稳定分类顺序时，才使用 `编号-英文短名` 格式（如 `01-foundations/`）；数字前缀只表达稳定顺序，不表达临时优先级或任务进度。
- 不需要编号的目录：并列领域、具体对象、产品、框架、论文、项目、工具名称目录原则上不使用数字前缀，例如 `autogen/`、`codex/`、`claude-code/`。
- 目录名不使用中文。
- 笔记文件：英文短名，语义自解释（如 `lora.md`、`rlhf-overview.md`）。
- 有明确阅读顺序、阶段顺序或流程顺序的一组正文文件，可以使用 `01-xxx-yyy.md`、`02-xxx-yyy.md`；数字前缀只表达稳定顺序，不表达临时优先级或任务进度。
- 独立专题、元信息文件、规则文件、论文笔记原则上不使用数字前缀；不要为了目录整齐批量重命名已有无编号文件。
- 论文笔记：`论文简称_arXiv编号.md`（如 `ReAct_2210.03629.md`）或 `论文简称_会议年份.md`（如 `BeingH05_ICLR2026.md`）。

## 三、README 规范

README 的详细规则见 [`docs/contributing/readme-rules.md`](./docs/contributing/readme-rules.md)。本文件只保留仓库级强约束：

- 每个一级目录（顶层领域目录）下必须有 `README.md`。
- 二级及更深层目录仅在存在或未来可能容纳子目录时必须写 README；纯文件目录不需要。
- README 必须包含目录结构，但按所在层级控制展开深度：根 README 按内容活跃度展开；一级目录（顶层领域目录）README 允许展开到三级子目录；二级及更深层 README 默认只记录直接子目录。
- 目录树原则上只展示目录，不展示内容文件；README 导航不主动列出 `overview.md`、`backlog.md`、`candidates.md`、`roadmap.md`、`conflict.md` 等元信息文件。
- 如需标注 `最后更新`、来源、适用范围、状态等元信息，统一放在标题下方、正文开始之前。

## 四、元信息文件模型

元信息文件的详细模型见 [`docs/contributing/metadata-files.md`](./docs/contributing/metadata-files.md)。本文件只保留七层职责摘要：

| 层级 | 认知任务 | 文件名 | 核心职责 | 出现条件 |
|:---|:---|:---|:---|:---|
| L1 定向 | Orient | `README.md` | “这是什么地方？跟我有什么关系？” | 目录存在或未来可能容纳子目录时必选 |
| L2 发现 | Discover | `index.md` | “我要的文件/主题在哪？” | 可选；文件/子目录较多，或主题交叉关系复杂时创建 |
| L3 理解 | Understand | `overview.md` | “这个领域的全貌和现状是什么？” | 该目录代表一个需要建立整体认知的领域 |
| L4 推进 | Track | `backlog.md` | “这个领域还有什么没被覆盖？” | 目录处于活跃建设期，内容有明显缺口 |
| L5 选题 | Queue | `candidates.md` | “接下来值得研究哪些对象？” | 可选；需要持续跟踪论文、仓库、官方文档、产品、协议、benchmark 等候选研究对象时创建 |
| L6 规划 | Plan | `roadmap.md` | “从哪开始、按什么顺序阅读/建设？” | 可选；有明确的内容组织顺序或学习路径时创建 |
| L7 校验 | Reconcile | `conflict.md` | “这里是否存在定义冲突或口径不一致？” | 仅当发现冲突时创建 |

命名约定：L3 的标准文件名为 `overview.md`，不再使用 `summary.md` / `SUMMARY.md` 承接领域全貌。

## 五、文档写作与表达建议

本章回答“内容已经决定写成一篇文档后，怎样写得更清楚、更易读”。

- 它主要关注**表达形式**，例如结构、公式、表格、论文笔记写法。
- 它不负责判断材料该进入 `overview.md`、`backlog.md`、`conflict.md` 还是正文；这属于第六章。
- 它也不负责定义元信息文件职责；那属于第四章。

以下为推荐做法，不强制：

- 知识性笔记：可包含数学公式（LaTeX）、代码块、表格等
- 论文笔记：建议包含核心贡献、方法描述、关键结果，不必逐段翻译
- README 中如有流程或阶段关系，可用代码块/ASCII 图展示

## 六、材料处理与文档增改流程

详细流程见 [`docs/contributing/documentation-workflow.md`](./docs/contributing/documentation-workflow.md)。本文件只保留仓库级执行原则：

- **先判断，再落位，最后写正文**：新材料进入主线前，先判断材料类型、Evidence 状态、目标落位和临时落点。
- **正文不承接未稳定材料**：正文负责表达经过整理和判断后的知识；`overview.md`、`backlog.md`、`candidates.md`、`conflict.md` 等元信息文件负责承接尚未稳定、尚待分流、尚待核验的材料。
- **外部材料先分级分流**：`temp/`、网页搜索、外部 AI 调研、博客摘录、issue/PR 讨论等默认只是上游输入或候选证据，不直接写成主线结论。
- **证据状态必须显式判断**：从外部材料回流主线时，先区分 `Verified`、`Observed`、`Inferred`、`Unverified`、`Conflicting`、`Deprecated`；完整规则见 [`docs/contributing/evidence-rules.md`](./docs/contributing/evidence-rules.md)。
- **链路字段按需保留**：涉及迁移、回流、冲突处理或外部来源时，应能说明 `Source / Decision / Placement / Gap`；完整规则见 [`docs/contributing/traceability-rules.md`](./docs/contributing/traceability-rules.md)。

默认策略：**拿不准时，先写元信息文件或保留在 `temp/`，不要抢写正文。**

## 七、其他规则

- **空目录**（README 出现条件详见 [README 规则](./docs/contributing/readme-rules.md)）：
  - 如果该目录存在或未来可能容纳子目录：优先放入 `README.md`，用一句话说明该主题的定位和研究计划。既保持结构完整，又为未来贡献预留语义接口
  - 文件非常少的底层目录，或快速调整目录结构时的临时状态，可用 `.gitkeep` 占位
  - `.gitkeep` 仅作为中间状态，目录稳定后应替换为 `README.md`
  - 纯文件目录（不会容纳子目录）不需要 README 或 .gitkeep，直接等文件填充即可
