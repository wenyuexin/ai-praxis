# 构造辅助层家族设计（初稿 · 待讨论）

> **状态**：设计透镜 / rationale（初稿）。本稿只解释这几层为何相关、为何不物理嵌套，**不再充当运行态落位模型**：运行态落位序见 [`../contributing/rules/documentation-workflow.md`](../contributing/rules/documentation-workflow.md) §9「通用落位序」，能力侧分流见 [`../capabilities/place.md`](../capabilities/place.md)。已按两轮外部 CONDITIONAL 评审整改（分类模型 + 边界）；已注册进 `design/README.md`；notes 二态原则与研究辅助层上下位关系已沉淀进 `intent/research-artifacts.md §9、§9.1`；`repo-map.md §3` 已加指向本稿的横切视角说明。
> **已收口**：是否下沉为强制规则已决——有序落位序落规则层（`documentation-workflow.md` §9），本稿降为 rationale；三个角色各自的现行机制、文件位置与 `.gitignore` 不变。
> **触发**：2026-09-21 本会话关于 `notes/` 与 `temp/` 定位的讨论。
> **证据**：模型部分是设计提案（Inferred）；现状陈述（`temp/` 被 gitignore、现存 4 个 `temp/`、`backlog/candidates/conflict` 已递归多层存在、`research-artifacts.md §5.1` 的约束）为本会话直接观察（Observed）。

## 1. 这份文档解决什么

把散在多处的三个问题收敛成一个统一视角：

- `temp/` 和 `notes/` 到底什么关系，要不要删 `temp/`、要不要把 `temp/` 嵌进 `notes/`。
- 多层级出现的 `temp/`（根 / 子领域 / 对象）是不是"碎片化"。
- `notes/` 内部该写哪些文档、类型是否封闭。

## 2. 定位：一层横切视角，不替代 repo-map 的功能分类

`repo-map.md §2` 按**组织功能**把仓库文档分成五类，回答"这份文档是哪一类"。本稿不改这套分类，而是在其中"研究辅助 / 元信息 / 临时输入"三类之上加一层**横切视角**：把它们看作**构造辅助层**——服务于构建与维护知识正文、自身不是给读者的稳定正文——回答"这些层在构建正文时如何配合、一份材料该落到哪一层"。

- **家族准入**：承载"构建 / 维护正文过程中的中转、支撑或规划"材料。
- **家族排除**：面向读者的稳定正文；以及稳定的导航 / 说明结构（`README.md`、`index.md`）——它们是元信息，但不是构建正文的中转或缓冲。
- 据此，`roadmap.md`（规划路径）属于家族；`README/index` 不属于。

## 3. 一个治理角色轴 + 若干描述属性

成员之间真正稳定的区别是**治理角色**，不是内容形态：

- **临时输入暂存** → `temp/`
- **持久研究辅助**（按作用域承接研究**过程**材料） → 对象 `notes/`、C 的结构研究辅助材料（及 `research-artifacts.md §5.1` 极窄的跨子目录辅助材料）
- **持久认知 / 规划元信息**（缺口 / 候选 / 冲突 / 路径） → `backlog` / `candidates` / `conflict` / `roadmap`

角色是**治理功能**，作用域是**独立的描述属性**：元信息角色可出现在任意层级——对象目录也有 `backlog` / `conflict`，所以“单对象”并不把材料推向 `notes/`（见 §6）。

下列是**描述属性**，用来刻画一份材料、但单独都不足以决定落位：

- **保存策略**：临时不提交 ↔ 持久提交
- **语义归属**：未归属 / 单对象 / 跨对象
- **容器范围**：对象目录 / 知识目录子树 / 仓库根（可递归）
- **结构程度**：仅描述内容形态，**不是决策轴**（`temp/` 可装结构完整的调研稿，`notes/evidence.md` 可高度结构化，`backlog` 可只是散 bullet）。

## 4. 三个角色的边界与判据归属

本节只给三个角色**装什么、不装什么**，以及**判据由谁裁决**；具体准入条件、门槛与 stop-line 一律回到对应 owner，本稿不复述。

- **`temp/`** — 临时输入暂存。装外部调研稿、网页抓取、arXiv 包、聊天 / handoff、未归属输入。它是**输入层**：材料的对象归属或证据状态尚未定型时停在这里。**是否已可迁出**见 [`documentation-workflow.md §7`](../contributing/rules/documentation-workflow.md) 与 [`evidence-and-traceability.md §5`](../contributing/rules/evidence-and-traceability.md)；**其路径不作为正式 Sources / Trace 端点**由 [`traceability-rules.md`](../contributing/rules/traceability-rules.md) 承接；**不提交**由 `.gitignore` 承接。这三件事分属不同 owner，拆分见 §5.3。
- **研究辅助材料** — 承接研究**过程**材料。对象研究时特化为 `notes/`；文档体系结构研究时，仅在 C 条件满足时形成独立材料；缺口 / 冲突等治理语义不进这里，走元信息文件，见 §6。对象 `notes/` 的二态、首文件命名与内部组织以 [`research-artifacts.md §3.9、§5`](../contributing/rules/research-artifacts.md) 为准；C 的准入与 stop-line 以 [`research-artifacts.md §5.2`](../contributing/rules/research-artifacts.md) 为准。本稿不复述运行规则。
- **`backlog` / `candidates` / `conflict` / `roadmap`** — 持久认知 / 规划元信息（缺口 / 待研究对象 / 口径冲突 / 有先后的路径）。它们承接**规划 · 认知**信息而非研究过程材料，这是与 `notes/` 的家族分界；**各自的出现条件与作用域属性以 [`metadata-files.md §2`](../contributing/rules/metadata-files.md) 为准**。本稿只借它们的一个 Observed 事实说明作用域不是落位判据：它们在多个层级递归存在（对象目录如 `codex/` 也有 `backlog` / `conflict`，子领域、根层亦然），所以“单对象”并不把材料推向 `notes/`（见 §3、§6）。

## 5. 关键设计决定

- **5.1 需求引出容器**：容器由“是否已出现独立于正文的过程材料”这一需求引出，而不是由材料体量引出。理由是：体量分档没有可执行阈值，会让同一份材料在平铺与目录之间反复摇摆；“有无独立辅助层”则是一次布尔判断，路径与文件名从第一天就稳定。另需分开的是：**“是否创建容器”与“容器内拆几个文件”是两个决定**，只有后者与材料类型、机制边界和维护频率有关。执行条件见 [`research-artifacts.md §5`](../contributing/rules/research-artifacts.md)，本稿只给理由。
- **5.2 按作用域递归**：三个角色都可多层存在，每层管子树；跨层材料上移到更靠近根的层。多层 `temp/` 不是碎片化，是这条的自然结果。
- **5.3 `temp/` 的“可撤销”是对共享知识可撤销，不等于本地随手可删、也不等于自动清理**：三件事分属不同 owner——① `.gitignore` 让未跟踪的 `temp/` 默认不进常规 Git 提交；② Traceability 保证正式 Sources / Trace 不以 `temp/` 路径为端点；③ 删除前仍须按 `AGENTS.md` 确认内容已迁移或不再需要。所以清理 `temp/` 不破坏已提交 / 共享的知识，但它可能仍含尚未回流的本地工作，**不是随手即删**；gitignore 也**不**保证 inbox 被清空（本地滞留会发生）。离开 `temp/` 的门槛与回退见 `research-ingestion-design.md`，本稿不重复生命周期。
- **5.4 位置不携带 committed 语义**：`temp/` 的层级只表达本地作用域；"原始 → 知识"的链由回流 **Trace** 承载（指向真实上游，不留 `temp/` 路径）。
- **5.5 归属边界**：`temp/`（归属前 / 跨对象）与 `notes/`（归属后 / 对象内）以"对象归属判断"为界。
- **5.6 聚拢靠概念，不靠物理嵌套**：跨对象 / 根层的未归属草稿没有对象 `notes/` 可挂；要让 `notes/` 上移到根 / 子领域又会撞上已占该位的元信息文件（`research-artifacts.md §5.1` 已把"notes 上移"卡到极严）。所以"辅助层是一族"靠**概念归类**表达，不靠把临时塞进持久。
- **5.7 对象级物理靠拢只用平级 temp**：真想靠拢，对象级原始材料放 `<obj>/temp/`（与 `<obj>/notes/` **平级**）；**不要** `<obj>/notes/temp/`——那把可删 / 被忽略的原始层塞进"持久"的 notes 树，与 `notes/` 持久定义及 `research-artifacts.md §3.9`"notes 不是 temp 镜像"都打架。根层保留一个不挂靠任何对象的 `temp/` 接未归属材料。

## 6. 落位分工的 why（非吸收执行协议）

> 本节**不**判断 Claim / Evidence / 来源恢复 / Trace，也不判断材料是否已满足离开 `temp/` 的条件——这些由 [`ingest.md`](../capabilities/ingest.md)、`documentation-workflow.md` 和 Evidence / Trace owner 负责。**强制落位序见 [`documentation-workflow.md §9 通用落位序`](../contributing/rules/documentation-workflow.md)，本节只给 why，不复述顺序。**

为什么这些层要按治理角色（临时暂存 / 对象内研究过程材料 / 缺口·候选·冲突·路径元信息）分工，而不是按内容形态或“单对象 / 跨对象”来分：

- **角色是治理功能，作用域是独立的描述属性**：元信息角色可出现在任意层级——对象目录也有 `backlog` / `conflict`，所以“单对象”并不把材料推向 `notes/`（这条作用域属性的 owner 是 [`metadata-files.md §2`](../contributing/rules/metadata-files.md)）。
- **`temp/` 与 `notes/` 的边界由“对象归属判断”划开**：归属前 / 跨对象留在 `temp/`，归属后 / 对象内才进对象辅助材料；因此 `notes/` 不是 `temp/` 的镜像或备份。
- **跨子目录共享的证据边界是极窄例外**：只在没有更轻量替代手段时使用，细则见 [`research-artifacts.md §5.1`](../contributing/rules/research-artifacts.md)。
- **聚拢靠概念，不靠物理嵌套**：把临时层塞进持久层会同时破坏两边的定义。

## 7. 本稿不改什么

- 不改三个角色各自的现有机制与既有规则条文。
- 不移动现有文件，不改 `.gitignore`。
- 只把已存在、但没被正着说出来的家族关系与落位协议讲清。

## 8. 待讨论 / 未决

- 这份视角的落点：独立成篇（现状），还是并入 `research-ingestion-design.md`。
- 家族准入边界的灰区（如历史上的 `overview.md` / `landscape.md` 既像元信息又像正文）是否需单列判据。`landscape.md` 固定文件类型已退役；当前结构研究辅助材料的准入与 stop-line 见 [`research-artifacts.md §5.2`](../contributing/rules/research-artifacts.md)。
- **是否下沉为强制规则：已决（2026-09-23）**——有序落位序落规则层（[`documentation-workflow.md §9 通用落位序`](../contributing/rules/documentation-workflow.md)），本稿降为 rationale，不再充当运行态模型。

## 9. 与现有文档的关系

- `research-ingestion-design.md`：讲输入的分流 / 暂存 / 回流**过程**；本稿讲这些层的**分类与落位**。
- `research-artifacts.md`（§3.3–3.9、§5、§5.1、§5.2、§7）：定义一般研究辅助层、对象 `notes/`、结构研究辅助材料与跨子目录例外；本稿把它们放进更大的家族视角。
- `metadata-files.md`：定义 `backlog/candidates/conflict/roadmap`；本稿指出它们是家族"跨层 · 持久"角色。
- `repo-map.md`：现有五类功能分类；本稿是其上的横切视角，不替代它（见 §2）。
