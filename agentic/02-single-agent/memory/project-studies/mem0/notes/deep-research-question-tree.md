# Mem0 深度研究问题树

本文件不是面向普通读者的对象总览，而是 `mem0` 深度研究的**递进式问题树脚手架**（触发判断见 [`research-artifacts.md`](../../../../../../docs/contributing/rules/research-artifacts.md) §3.4，结构要求见 §3.5）。

它的目标不是重复"它是什么、有哪些能力"——那部分已在 [`general.md`](./general.md) 的 Claim-Source 对照里达到 `Verified` / `Observed`——而是把后续研究组织成一条可持续推进的主线：

> Mem0 把"记忆一致性"的责任从**写入期**移到了**读取期**——这个转移解决了什么，代价落在哪里？

使用方式：

> 2026-10-01 源码核验更新：当前事实状态以 `evidence.md` M2 / M4 / M4a / M4b 和 `source.md` §1.3–§1.3.3 本地 `v2.2.1` 核验为准。下文尚未逐节点回写的“只累积不覆盖”仅适用于普通自动抽取事件，不适用于公共 CRUD 或实体索引。同步检索的候选集来自语义召回，BM25 / 实体参与打分；不能读成三路并行召回取并集。BM25 还受 vector store、collection slot 与可选依赖影响；评分函数不读取 `created_at`。时间能力不能由 Platform 博客直接外推到该 OSS commit。

- 每个问题区分：**当前已知**、**待确认**、**下一步研究动作**。
- 后续研究沿这棵树逐层回答、补证、收口；每轮核验结论追加到对应节点下方，不另起文件。
- 一手来源摘录与核验路径进 `notes/source.md`；Claim-Source 对照与状态进 `notes/evidence.md`；本文件只负责问题级递进。
- **本对象暂无 `overview.md`**：按 §3.4 Stop-line，问题已进入机制级递进时不应继续补总览式正文；待主线压实后再按 §3.1.1 出目录级 `overview.md` + 带版本标识的对象正文。
- **暂无 `roadmap.md`**：推进顺序按"最能改变后续结构的问题优先"，写在各节的「下一步研究动作」里，不单独建 `roadmap.md`（[`metadata-files.md`](../../../../../../docs/contributing/rules/metadata-files.md) §5.3：仅在存在稳定阅读或建设顺序时创建）。

核验纪律：

- **版本锚点已部分闭合，但 PyPI 对应关系仍未单独闭合**：本轮已有三层信息——官方 OSS `v2 → v3` migration 文档作为算法代次、本地核心库 `v2.2.1` → `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd` 作为源码锚点、`pyproject.toml` 的 `mem0ai 2.2.1` 作为包字段观察。仍需确认 PyPI 安装包是否对应该 commit。按 [`traceability-rules.md`](../../../../../../docs/contributing/rules/traceability-rules.md) §3.5，源码观察已可恢复到具体 commit；包发布行为仍保留不确定性。Mem0 迭代快，继续标 `Drift Risk: high`。
- 状态词只用 `Verified` / `Observed` / `Inferred` / `Unverified` / `Conflicting` / `Deprecated`（[`evidence-assessment-rules.md`](../../../../../../docs/contributing/rules/evidence-assessment-rules.md) §3）。`general.md:20`「需重限定」与 `:22`「疑为错 / 版本混淆」不在词表内，待随 `evidence.md` 结晶时改正。
- 论文原文与官方文档直接可证为 `Verified`；单一仓库或单版本观察为 `Observed`；从架构推断为 `Inferred`；三者分开写，不混标。
- 来源只写可还原形式（`mem0ai/mem0` + tag/commit、arXiv 编号、官方文档 URL），不写本机绝对路径（`traceability-rules.md` §3.1）。

---

## 0. 主线问题

### 0.1 主问题

Mem0 在自动抽取路径中减少了写入期的显式变更，并把一部分排序与过滤压力交给读取期——这个变化解决了什么，代价落在哪里？

### 0.2 依据与状态

这条主线基于 `source.md` / `evidence.md` 已记录的一组带版本边界的事实：

- **旧版**（arXiv 2504.19413）：论文摘要确认两阶段的抽取与整合；ADD / UPDATE / DELETE / NOOP 四值的完整原文位置仍待全文建档，不能在此提前当作已闭合事实。
- **新版自动抽取路径**（本地 `v2.2.1` OSS commit）：单趟 **ADD-only**（一次 LLM 调用，抽取结果只持久化 ADD）+ 实体链接 + 语义候选上的 BM25 / entity 打分；显式 `update()` / `delete()` API 仍保留。
- **官方迁移文档 / 后续博客**：描述 v2 → v3 的 ADD-only、hybrid retrieval，以及后续 Temporal Reasoning / Memory Decay；这些能力是否对应当前 OSS commit、在何处执行，继续单独核验。

**把这组事实读成“自动抽取路径减少写入期显式变更、读取侧承担更多排序与过滤”仍是 `Inferred`，不是官方表述。** `evidence.md` M16 与本问题树保持同一状态；显式 CRUD、后端退化和时间机制必须分开处理。

会推翻它的观察（按 `evidence-assessment-rules.md` §4.2 的护栏先写下来）：

- 新版仍在别处做写入期消解——异步 compaction、后台 merge，或托管平台独有逻辑；
- 多信号检索与时间推理的引入与 ADD-only 无因果关系，只是并行的检索增强；
- ADD-only 只是某个配置或模式下的默认值，而非新版唯一写入路径。

命中任一条，主线需重写，本文各层的组织方式也要跟着调整。

### 0.3 为什么这条主线值得优先

`general.md` 已能回答定位与 API 层面的问题，但缺的是机制级答案：矛盾事实在存储里如何共存、读取侧凭什么把它们消解、消解不掉的部分变成什么后果。

这条主线同时是三件事的共同上游：版本分叉该怎么组织正文（§6.2）、LOCOMO 争议有没有技术判据（§5）、OSS 与托管平台的能力差落在哪（§6.1）。

---

## 1. 写入层

### 1.1 各版本写入期到底做了什么

#### 当前已知

- 旧版论文摘要确认两阶段 = 抽取 + 整合；更新阶段的 ADD / UPDATE / DELETE / NOOP 四值完整依据仍待全文建档（`Status: Unverified`，arXiv 2504.19413）。
- 本地 `v2.2.1` OSS 的 `infer=True` 自动抽取为单趟 ADD-only，持久化 history 事件为 `ADD`；公共 `update()` / `delete()` API 仍存在（`Status: Observed`，源码 commit）。
- 「遗忘仅按时间」这一说法两版都不成立：旧版有 LLM 决策的 DELETE，新版只累积不覆盖（`Status: Deprecated`——该说法源于版本混淆，而非两个来源对立，故不标 `Conflicting`）。

#### 待确认

- 新旧版 pipeline 的一手确认：步骤顺序、LLM 调用次数、决策点位置。
- ADD-only 是否真无任何写入期消解，含异步与后台路径。
- 抽取阶段在新版是否也变了，还是只有更新阶段被替换。

#### 下一步研究动作

- 先补版本锚点（tag 或 commit + `Observed At`），再读 `mem0ai/mem0` 的写入路径。
- 建档 arXiv 2504.19413 的两阶段描述到 `notes/source.md`。

---

## 2. 存储层

### 2.1 去掉 UPDATE / DELETE 后，矛盾与过期事实以什么形式共存

#### 当前已知

- 论文 / v2 语境下有三层 scope（user / session / agent）以及向量与图（Mem0g）混合后端（`Status: Observed`，arXiv 2504.19413 / Zylos）。
- 官方 migration 将图记忆移出 OSS、转为 Platform 能力；本地 `v2.2.1` 的 graph 配置和源码范围仍需继续核对（`Status: Observed`）。

#### 待确认

- 同一事实的多个版本，是同一条记录的多版本，还是并列的独立记录。
- 图后端的实体链接是否承担了隐式消解：同一实体的矛盾属性如何表示。
- 是否存在 TTL、容量上限、去重或 dedup 阈值。

#### 下一步研究动作

- 读存储层 schema 与 `add()` 写入路径；图后端单独核验，不要用向量侧结论外推。

---

## 3. 读取层

### 3.1 多信号检索与时间推理各自补偿哪一类矛盾

#### 当前已知

- 本地 `v2.2.1` 检索候选由语义召回构造，BM25 / entity 参与融合；Qdrant 还受 sparse slot 与 fastembed 影响（`Status: Observed`，源码 commit）。

#### 待确认

- 三路信号如何融合：加权、重排，还是召回后过滤；时间推理在哪一步介入。
- 时间推理是简单"取最新"，还是更复杂的时序关系判断。
- 检索是否会同时返回相互矛盾的记忆条目，把取舍留给下游 LLM。

#### 下一步研究动作

- 读 `search()` 路径；重点找"同一实体存在多个矛盾值"时的行为——这是判断读取期补偿是否完备的关键样本。

---

## 4. 完备性层

### 4.1 哪些矛盾读取期解决不了

#### 当前已知

- 无。本层是主线的核心推论，目前全部待证。

#### 待确认

- 记忆膨胀：只累积在长会话下的增长曲线，以及对检索精度的影响。
- 污染与旧事实复活：过期事实是否会因语义相似度更高而被召回。
- 三条候选问题是否正落在本层：grounding / confidence（`Status: Observed`，#4573）、写入阶段过度提取（`Status: Observed`，Discussion #4289）、Node 同步阻塞 30–40s（`Status: Unverified`，尚无一手 issue）。

#### 下一步研究动作

- 核三条缺陷的原始 GitHub issue 与各自适用版本。按 `metadata-files.md` §6.2，它们目前属"尚未验证"，去向是 `backlog.md`，**不进** `conflict.md`。
- 若上游已有关于累积污染的 issue 讨论，优先作为本层一手证据。

---

## 5. 评测层

### 5.1 LOCOMO 这类评测能否暴露累积污染

#### 当前已知

- Mem0 自述 LOCOMO 分数来自**托管平台**，官方明示 OSS 会不同（`Status: Verified`，README 免责声明）。
- 91% 延迟优势是对 full-context 基线，不是对同类记忆系统（`Status: Verified`，arXiv 摘要）。
- LOCOMO 争议至少拆成 Zep ↔ Mem0 的分数重算争议与 Letta ↔ Mem0 的 MemGPT baseline 可复现性争议；两边的一手材料仍未完整拉齐（`Status: Observed`）。

#### 待确认

- LOCOMO 的任务形态：单轮问答，还是长程多轮累积。
- 若为单轮问答，它在结构上能否暴露 §4 的累积污染——这是把口径之争转成技术判据的关键（当前 `Status: Inferred`）。
- Mem0 开源 eval 框架跑的是 OSS 配置还是托管配置。

#### 下一步研究动作

- 拉 Letta 反驳原文与 Mem0 eval 框架仓库；读 LOCOMO 原始论文确认任务形态。
- 坐实后按 `metadata-files.md` §6.1 落 `memory/conflict.md`——该冲突跨 Mem0 与 Letta 两个对象，`mem0/` 这一层范围覆盖不了；与既有 `memory/memory-taxonomy-conflicts.md` 的职责关系待维护者确认。

---

## 6. 形态层

### 6.1 OSS 与托管平台的差异落在哪

#### 当前已知

- 官方明示 LOCOMO 分数来自托管平台、OSS 会不同（`Status: Observed`）。

#### 待确认

- 差异是模型与参数配置，还是写入或检索机制本身不同。
- 三条 OSS 缺陷是否即该差异的表现。

#### 下一步研究动作

- 对照官方文档的 OSS / Platform 功能矩阵；差异若落在写入或检索机制，回写 §1、§3。

### 6.2 版本演进如何影响正文结构

#### 当前已知

- Mem0 属版本敏感对象：版本差异会改变能力边界与实现判断（`Status: Observed`），故按 `research-artifacts.md` §3.1.1，未来形态是目录级 `overview.md` + 带版本标识的对象正文，不是一篇静态 Mem0。

#### 待确认

- 新旧版之间是否还有中间版本；ADD-only 的分叉点对应哪个 release。

#### 下一步研究动作

- 版本锚点补齐后建版本谱系。主线压实一层就提炼一层正文，不等全树闭合。

---

## 7. 当前研究状态面板

### 已相对清楚

- 对象定位与 `add()` / `search()` API 面（`Verified`）。
- 三层 scope + 向量与图混合后端（`Observed`）。
- 版本分叉的存在与大致形态（`Observed`；算法代次、本地核心 `v2.2.1` commit 与包字段已有锚点，PyPI 对应关系仍未单独闭合）。
- 「遗忘仅按时间」不成立——两版都不是按时间遗忘（`Deprecated`）。

### 仍然模糊

- 写入期各版本的确切 pipeline 与决策点位置。
- 矛盾事实在存储层的表示形式。
- 三路检索的融合方式、后端退化条件、semantic threshold 对冲突事实的影响，以及时间推理的介入点。
- 累积污染是否真实发生、以什么形式发生。
- LOCOMO 任务形态，以及争议双方的一手口径。
- OSS 与托管平台差异的性质。

### 当前优先补证点

按"最能改变后续结构"排序，不按容易程度排：

1. **版本锚点**——Stop-line 级要求，且决定其余所有结论能否复核。
2. **§1 写入层 + §2 存储层**——主线假设成立与否在此判定。
3. **§5 评测层**——等 §4 有了判据再动，否则只能罗列各方说法。

### 下一轮推荐研究任务

1. 补齐 PyPI release 文件与本地 `v2.2.1` commit 的对应关系，并保留源码观察与算法代次的分层。
2. 继续读代表性 vector store 的 `keyword_search()` 与依赖退化路径，判定 hybrid retrieval 的真实适用范围。
3. 设计包含真实 embedding / 本地 vector store 的新旧事实实验，验证 semantic candidate、threshold 和 boost 的端到端关系。
4. 再读新旧版写入路径和论文全文，判定主线假设（含 §0.2 的三条反驳条件）。
5. 核三条 OSS 缺陷的原始 issue 与适用版本。

---

## 8. 与现有文档的关系

- [`general.md`](./general.md)：第一轮一手核对与过程记录；其中的旧状态词和旧判断保留到阶段四统一回写，不作为当前权威状态表。
- [`source.md`](./source.md)：一手来源摘录、核验路径、版本语义。
- [`evidence.md`](./evidence.md)：Claim-Source 对照层，每轮核验结论先沉淀于此。
- [`../../../candidates.md`](../../../candidates.md)：Mem0 作为候选对象的队列条目与研究理由；本文件压实的结论应回写其状态。
- [`../../../memory-taxonomy-conflicts.md`](../../../memory-taxonomy-conflicts.md)：分类体系冲突专题。本文件不承接 taxonomy 议题（`general.md` 末尾的 CoALA 合流仍走该文件）。
- 未来的 `overview.md` 与带版本标识的对象正文：按 `research-artifacts.md` §3.1.1 在主线压实后提炼；本文件继续负责未闭环问题与后续推进。
