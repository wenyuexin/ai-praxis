# Mem0 深度研究问题树

本文件不是面向普通读者的对象总览，而是 `mem0` 深度研究的**递进式问题树脚手架**（触发判断见 [`research-artifacts.md`](../../../../../../docs/contributing/rules/research-artifacts.md) §3.4，结构要求见 §3.5）。

它的目标不是重复"它是什么、有哪些能力"——那部分已在 [`evidence.md`](./evidence.md) 的 Claim-Source 对照里达到 `Verified` / `Observed`；`general.md` 保留第一轮来源线索和过程摘要——而是把后续研究组织成一条可持续推进的主线：

> 工作假设：Mem0 不同版本与部署形态如何组合**写入期变更、状态标记与读取期过滤**来处理一致性？各层的代价分别落在哪里？

使用方式：

> 2026-10-03 研究回写更新：当前事实状态以 `evidence.md` M2 / M2a–M2f / M3a–M3f / M4 / M4a / M4b / M17–M21 和 `source.md` §1.3–§1.3.8、§3.1–§3.6 为准。论文旧版、OSS 普通自动抽取、Platform Dream 和 Platform Synthesis 分开处理。普通 Python / TypeScript 自动抽取的 ADD-only 不等于所有写入处理都消失：仍有 prompt 级筛选、候选局部 hash 去重和实体索引更新；Python 的 `infer=False` 与 procedural 是直接追加，vision 只是预处理；这些结论不适用于 Platform 服务端或未穷尽入口。同步 OSS 检索的候选集来自语义召回，BM25 / 实体参与打分；不能读成三路并行召回取并集。BM25 还受 vector store、collection slot 与可选依赖影响；评分函数不读取 `created_at`。Platform Dream 公开契约包含 add-time supersede / merge 与 read-time `latest_only` / `include_merged`；Temporal Reasoning / Memory Decay 仍不能外推到 OSS commit。LOCOMO 与 LongMemEval 的任务覆盖、适用边界、第三方审计入口及 Platform/OSS 分数口径已单独核验。

- 每个问题区分：**当前已知**、**待确认**、**下一步研究动作**。
- 后续研究沿这棵树逐层回答、补证、收口；每轮核验结论追加到对应节点下方，不另起文件。
- 一手来源摘录与核验路径进 `notes/source.md`；Claim-Source 对照与状态进 `notes/evidence.md`；本文件只负责问题级递进。
- **本对象暂无 `overview.md`**：按 §3.4 Stop-line，问题已进入机制级递进时不应继续补总览式正文；待主线压实后再按 §3.1.1 出目录级 `overview.md` + 带版本标识的对象正文。
- **暂无 `roadmap.md`**：推进顺序按"最能改变后续结构的问题优先"，写在各节的「下一步研究动作」里，不单独建 `roadmap.md`（[`metadata-files.md`](../../../../../../docs/contributing/rules/metadata-files.md) §5.3：仅在存在稳定阅读或建设顺序时创建）。

核验纪律：

- **版本锚点已部分闭合**：官方 OSS `v2 → v3` migration 文档作为算法代次、本地核心库 `v2.2.1` → `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd` 作为源码锚点、`pyproject.toml` 的 `mem0ai 2.2.1` 作为包字段观察；PyPI 同版本工件的上传时间与 release commit 高度吻合，但没有工件级 Git SHA 闭环，保持高置信 `Inferred`。按 [`traceability-rules.md`](../../../../../../docs/contributing/rules/traceability-rules.md) §3.5，源码观察可恢复到具体 commit；包构建对应关系不升级为 `Verified`。Mem0 迭代快，继续标 `Drift Risk: high`。
- 状态词只用 `Verified` / `Observed` / `Inferred` / `Unverified` / `Conflicting` / `Deprecated`（[`evidence-assessment-rules.md`](../../../../../../docs/contributing/rules/evidence-assessment-rules.md) §3）。第一轮 `general.md` 中的「需重限定」与「疑为错 / 版本混淆」已在 2026-10-02 改为词表状态；当前状态表以 `evidence.md` 为准。
- 论文原文与官方文档直接可证为 `Verified`；单一仓库或单版本观察为 `Observed`；从架构推断为 `Inferred`；三者分开写，不混标。
- 来源只写可还原形式（`mem0ai/mem0` + tag/commit、arXiv 编号、官方文档 URL），不写本机绝对路径（`traceability-rules.md` §3.1）。

---

## 0. 主线问题

### 0.1 主问题

论文旧版、OSS 普通自动抽取和 Platform Dream 分别在哪里处理事实变化、重复与矛盾？哪些能力发生在 add-time，哪些只影响 read-time，哪些仍无法从公开材料确认？

### 0.2 依据与状态

这条主线基于 `source.md` / `evidence.md` 已记录的一组带版本边界的事实：

- **旧版**：论文 v1 §2.1 / Figure 2 正式命名为 extraction + update；Appendix B 及 §2.1 确认 ADD / UPDATE / DELETE / NOOP。基础 Mem0 以 DELETE 处理矛盾，Mem0g 以 invalid 关系保留过时关系；论文设计与当前 OSS / Platform 实现分开。
- **OSS 普通自动抽取**（Python `v2.2.1` 与同 commit TypeScript `ts-v3.3.1`）：单趟 **ADD-only**（一次 LLM 调用，抽取结果只持久化 ADD）+ 候选局部 hash 去重 + 实体链接 + 语义候选上的 BM25 / entity 打分；显式 `update()` / `delete()` API 仍保留。Python `infer=False` 与 procedural 是直接追加；TypeScript 未观察到 procedural 入口。
- **写入期仍有局部处理**：prompt 要求语义去重并保留变化语境；普通同步/异步自动抽取还存在候选局部 hash 去重。prompt 要求输出 `linked_memory_ids`，但 OSS 普通 `add()` 没有消费这一 LLM 字段，实体索引的同名字段不等于记忆版本链。
- **特殊入口**：`infer=False` 逐条直接追加；procedural 由 LLM 生成单条 procedural memory 后直接追加；vision 只是进入上述路径前的消息预处理（`Observed`，固定 Python commit）。
- **Platform 公开契约**：Dream 文档称 supersede / merge 在新增记忆时评估；读取支持 `latest_only` / `include_merged`；Synthesis 是后台定时综合。`delete_linked`、`replaced_by` 与 `linked_memory_ids` 的服务端建链和执行细节仍待核验。
- **官方迁移文档 / 后续博客**：描述 v2 → v3 的 ADD-only、hybrid retrieval，以及 Platform-only 的 Temporal Reasoning / Memory Decay；后两者不能外推到 OSS commit。

**把这组事实压成“自动抽取路径减少写入期显式变更、读取侧承担更多排序与过滤”已经过度概括。** `evidence.md` M16 与本问题树保持同一状态：论文旧版是 update-time mutation，OSS 普通路径是 ADD-only + 局部去重 + entity index，Platform Dream 是 add-time supersede / merge + read-time filters；局部 hash 去重、实体索引更新、显式 CRUD、Platform Synthesis、后端退化和时间机制必须分开处理。

研究边界与后续反证方向：

- 当前没有证据建立 hybrid retrieval 与移除自动 UPDATE / DELETE 之间的冲突处理因果关系；不能从并列出现推出补偿动机，也不能反向断言二者必然无因果。
- 如果固定 commit 的普通 OSS 路径存在漏读的旧事实失效逻辑，需要修正当前静态观察；其他入口的不同机制只改变对应入口的范围，不自动推翻普通路径结论。
- 如果新的 Platform 一手材料修订 add-time 契约或读取状态表，应按文档版本更新分层；不能把当前文档快照当作所有线上版本的保证。

### 0.3 为什么这条主线值得优先

`general.md` 已能回答定位与 API 层面的问题，但缺的是机制级答案：不同部署形态如何表示矛盾与重复、哪些状态在 add-time 产生、读取模式如何解释这些状态，以及仍无法消解的部分变成什么后果。

这条主线同时是三件事的共同上游：版本分叉该怎么组织正文（§6.2）、LOCOMO 争议有没有技术判据（§5）、OSS 与托管平台的能力差落在哪（§6.1）。

---

## 1. 写入层

### 1.1 各版本写入期到底做了什么

#### 当前已知

- 论文全文已确认旧版 extraction + update 与四值操作（M3a–M3c）；这只闭合了论文旧版设计，不闭合当前 OSS、特殊入口或 Platform 服务端对应关系。
- 本地 `v2.2.1` OSS 的 `infer=True` 自动抽取为单趟 ADD-only，持久化 history 事件为 `ADD`；公共 `update()` / `delete()` API 仍存在（`Status: Observed`，源码 commit）。
- 普通自动抽取仍有候选局部 hash 去重与实体索引更新；prompt 要求的记忆间 `linked_memory_ids` 未在该 OSS 写入路径持久化（`Status: Observed`，源码 commit）。
- 退役未区分版本、自动抽取、显式 CRUD 与衰减的「遗忘仅按时间」概括（M8，`Status: Deprecated`）；不作所有版本穷举判断。

#### 待确认

- 旧版论文与当前实现之间的版本对应关系；OSS 普通 Python / TypeScript 路径已观察，其他入口、后台路径与 Platform 服务端仍分开核验。
- 候选外事实、后台路径、Platform 状态判定与实际 LLM 抽取效果；Python procedural、vision 和 `infer=False` 的路径边界已观察，但不外推为全系统语义。
- OSS 抽取阶段与旧版 update 阶段的变化是否存在因果关系；不能从同一 release 的并列说明推出。

#### 下一步研究动作

- 继续补未覆盖后台入口与 Platform 服务端边界；TypeScript 普通路径已核对，但不能把它扩大到所有 SDK 入口。
- 继续对照论文旧版设计与当前版本实现，不把论文四值操作外推到当前 OSS 或 Platform Dream。

---

## 2. 存储层

### 2.1 不同机制下，矛盾与过期事实以什么形式共存

#### 当前已知

- 论文 / v2 语境下有三层 scope（user / session / agent）以及向量与图（Mem0g）混合后端（`Status: Observed`，arXiv 2504.19413 / Zylos）；该论文设计不等于当前 OSS 后端。
- 官方 migration 将图记忆移出 OSS、转为 Platform 能力；本地 `v2.2.1` 的 graph 配置和源码范围仍需继续核对（`Status: Observed`）。
- 普通 Python / TypeScript OSS 自动抽取将每条新记忆作为独立记录插入，并以候选局部 hash 去重；实体索引保存实体到记忆 ID 的关联，但不是已证实的记忆版本/矛盾消解链（`Status: Observed` + `Inferred`，固定 commit）。
- Platform Dream 的公开契约补充了另一种状态模型：矛盾新事实可使旧记忆标为 `superseded`，重复或更丰富的新事实可使旧记忆标为 `merged`；旧记录不等于被物理删除。多数重复或近重复事实在添加时直接保持单条 canonical memory，较完整的新事实到来时才可能保留一条 `merged` 记录。`latest_only` 和 `include_merged` 是读取模式，服务端建链和状态判定仍未公开。

#### 待确认

- 其他写入入口与 Platform 如何表示事实版本；普通 OSS 已观察到新 UUID 独立插入，但“同一事实”的语义判定与实际抽取结果未验证。
- 论文 / 旧版图方案与 Platform 图关系是否承担冲突处理；不能与已读 OSS 的实体索引混为一种机制。
- 是否存在全库语义去重、TTL、容量上限，或对同实体不同值的时间/冲突裁决；当前只观察到候选局部 hash 去重。
- Platform 的 `replaced_by` 是否与 SDK / changelog 所称 `linked_memory_ids` 一一对应；`delete_linked` 的递归范围、环检测、原子性与失败语义；OSS prompt 输出的 `linked_memory_ids` 是否与 Platform 链接使用同一字段。

#### 下一步研究动作

- 继续读其他写入入口与 Platform 服务端边界；图后端单独核验，不要用普通向量侧结论外推；不要把 Platform 公开契约写成已知的内部实现。

---

## 3. 读取层

### 3.1 多信号检索、时间能力与读取过滤对冲突可见性有什么影响

#### 当前已知

- 本地 `v2.2.1` 检索候选由语义召回构造，BM25 / entity 参与融合；Qdrant 还受 sparse slot 与 fastembed 影响（`Status: Observed`，源码 commit）。
- 同步路径先过滤过期记录，评分函数再以 semantic threshold 门控，随后对三种分数做加法归一化与排序；`score_and_rank` 不读取 `created_at`（M4–M4b）。这是候选内相关性重排，不是矛盾裁决。
- Platform Dream 公开读取契约默认包含 active + superseded、隐藏 merged；`latest_only=true` 只保留 active，`include_merged=true` 纳入 merged（M2d）。这依赖已标记的状态，不能与 OSS 的相关性评分或 Platform Temporal Reasoning 合称一种机制。

#### 待确认

- 其他后端、异步或未覆盖入口是否遵循同一候选与融合顺序；Platform 时间机制和 Dream 状态链的具体服务端执行位置。
- 时间推理是简单"取最新"，还是更复杂的时序关系判断。
- OSS 实际样本中是否会同时召回矛盾条目，以及下游 LLM 如何取舍；Platform 默认保留 superseded 的公开契约已知，具体查询结果仍未验证。

#### 下一步研究动作

- 读 `search()` 路径；重点找“同一实体存在多个矛盾值”时的返回与排序行为——这是判断读取侧是否会暴露或隐藏冲突的关键样本，不能预设它是在“补偿”写入期机制。

---

## 4. 完备性层

### 4.1 哪些冲突仍会暴露在读取结果中

#### 当前已知

- #4956 已有可变事实共存、旧值可能排前的报告（M9，`Observed`）；不是本研究复现，适用版本和修复状态仍待核验。M4a / M4b 已限定评分函数能重排哪些候选，但不证明端到端冲突发生率。

#### 待确认

- 记忆膨胀：只累积在长会话下的增长曲线，以及对检索精度的影响。
- 污染与旧事实复活：过期事实是否会因语义相似度更高而被召回。
- 三条候选问题是否正落在本层：grounding / confidence（`Status: Observed`，#4573）、写入阶段过度提取（`Status: Observed`，Discussion #4289）、Node 同步阻塞 30–40s（`Status: Unverified`，尚无一手 issue）。

#### 下一步研究动作

- 核三条缺陷的原始 GitHub issue 与各自适用版本。按 `metadata-files.md` §6.2，它们目前属"尚未验证"，去向是 `backlog.md`，**不进** `conflict.md`。
- 若上游已有关于累积污染的 issue 讨论，优先作为本层一手证据。

---

## 5. 评测层

### 5.1 LOCOMO / LongMemEval 能否暴露累积污染

#### 当前已知

- Mem0 自述 LOCOMO 分数来自**托管平台**，官方明示 OSS 会不同（`Status: Verified`，README 免责声明）。
- 91% 延迟优势是对 full-context 基线，不是对同类记忆系统（`Status: Verified`，arXiv 摘要）。
- LOCOMO 原论文是长、多 session 历史上的离线 QA / summarization / multimodal benchmark；多跳与时序覆盖已确认，但没有事实版本更新、旧值复活或冲突共存的专门受控指标（M17）。
- LongMemEval 明确包含 knowledge-update、temporal-reasoning、multi-session 和 abstention；其更新能力覆盖“识别变化并使用后来的状态”，但历史构造避免无关冲突，不能等同于矛盾共存基准（M18）。
- Mem0 评测框架同时支持 OSS 与 Cloud，但当前 92.5 / 94.4 公开数字归属于 managed Platform，缺少与 OSS 同配置、同模型、同数据版本的完整对等成绩闭环（M19）。
- 固定 benchmark 子模块 commit `4b61c5d…` 内可复核的是 Platform 91.6 / 93.4 artifacts；当前文档声明的 92.5 / 94.4 没有匹配的公开 run manifest，不能把两组数字当作同一轮结果或简单修订（`source.md` §3.3.1）。
- LOCOMO 争议已拆成 Zep ↔ Mem0 与 Letta ↔ Mem0 两条独立争议；一手材料支持争议主张存在，但没有形成统一胜负结论（M20）。
- 固定审计快照可确认逐题标注、SHA 声明、重算脚本和统计材料存在；审计者的 156/99/57 分类与分数影响论证尚未独立验证（M21）。

#### 待确认

- 91.6/93.4 与 92.5/94.4 分别对应哪些 Platform 服务端版本、数据快照和模型 stack；固定 artifacts 已知，但后一组当前没有完整 run manifest。
- LOCOMO / LongMemEval 的 Platform extraction、embedding、retrieval depth、重试/随机性和数据版本是否完全一致；OSS LongMemEval 的公开模型标签已知，但 embedding endpoint、版本和完整挂载配置仍不完整。
- LongMemEval knowledge-update 是否足以区分“正确更新到新状态”与“长期保留、条件化选择多个冲突版本”；论文任务定义支持前者，后者仍缺专门指标。
- LoCoMo / LongMemEval 的高分是否能预测开放世界持续运行中的写入污染、旧事实复活和冲突共存；需要额外受控评测或真实运行证据。

#### 下一步研究动作

- 已恢复 benchmark 子模块 commit 与主要结果分区；下一步只补读 run manifest / artifact 缺口，不运行 benchmark，不把“可运行”写成“已复现”。
- 若要升级第三方审计结论，先分层抽查：原始数据 hash → 有代表性的 score-corrupting 标注及其原始对话证据 → 统计/LLM-judge 输入输出；在此之前仅将其作为第三方协议审计主张。
- 对照 LongMemEval 的 knowledge-update 与 Mem0 OSS ADD-only / Platform Dream supersede 机制，明确 benchmark 能测到哪一层、测不到哪一层。
- Zep 与 Letta 的一手材料已拉齐到争议主张层；若需要记录跨对象口径冲突，再按 `metadata-files.md` §6.1 评估 `memory/conflict.md`，不把当前争议直接改写成胜负结论。

---

## 6. 形态层

### 6.1 OSS 与托管平台的差异落在哪

#### 当前已知

- 官方明示 LOCOMO 分数来自托管平台、OSS 会不同（`Status: Verified`，README 免责声明）。
- 固定 Python / TypeScript OSS 普通自动抽取与 Platform Dream 的公开机制不同：前者观察到 ADD-only + 局部去重 + entity index，后者公开描述 add-time supersede / merge 和 read-time `latest_only` / `include_merged`（`Status: Observed` / `Verified`，见 M2d、M3e；`source.md` §1.3.7–§1.3.8）。服务端内部判定与建链仍未闭合。

#### 待确认

- Platform Dream 服务端实际如何判定 supersede / merge、建立 `replaced_by` / `linked_memory_ids`，以及 flag 组合和失败语义。
- 除公开契约外，OSS 与 Platform 的模型、参数、写入或检索机制还有哪些差异。
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
- 论文 / v2 语境的 scope 与向量 / 图方案线索，以及 OSS / Platform 图能力迁移声明（M5）；不合并为当前 OSS 的统一后端结论。
- 版本分叉的存在与大致形态（`Observed`；算法代次、本地核心 `v2.2.1` commit 与包字段已有锚点，PyPI 对应为高置信 `Inferred`，仍无工件级 SHA 闭环）。
- 论文旧版的 extraction + update、四值操作与基础冲突处理设计（M3a–M3c，`Verified`）。
- 普通 Python / TypeScript OSS 同步写入的 prompt 指令、局部 hash 去重、新记录追加与实体索引边界（M2–M2c、M3e，`Observed`）。
- `infer=False`、Python procedural 和 vision 的入口边界（M3d，`Observed`）；不同入口不能统一外推到整个 Python OSS（M3f，`Inferred`）。
- Platform Dream 的 add-time supersede / merge 与 read-time `latest_only` / `include_merged` 公开契约（M2d，`Verified`）；Platform-only Temporal Reasoning / Memory Decay 的公开能力边界（M2e，`Verified`），其运行效果、执行阶段与服务端实现仍未闭合。
- 未区分版本和操作的「遗忘仅按时间」概括已退役（M8，`Deprecated`）。

### 仍然模糊

- 旧版论文设计与当前未覆盖写入入口之间的版本对应关系；Platform Dream 公开契约与服务端实现之间的落差。
- 普通路径实际抽取哪些矛盾事实，以及 Platform 如何构建 superseded / merged 版本关系。
- 其他后端 / 入口的退化条件、semantic threshold 对实际冲突样本的影响，以及 Platform 时间机制和 Dream 状态链的服务端介入点；固定同步路径融合公式已记录。
- 累积污染是否真实发生、以什么形式发生。
- LOCOMO / LongMemEval 的具体运行配置、judge、模型、embedding、retrieval depth 和数据版本；争议双方的实现与计分口径仍未统一。
- OSS 与托管平台在已知公开能力差异之外的实现差异及其效果。

### 当前优先补证点

按"最能改变后续结构"排序，不按容易程度排：

1. **§1 写入层 + §2 存储层**——已有固定源码锚点；继续对照未覆盖入口，并把 Platform 公开状态模型与 OSS 记录模型分开。
2. **发布制品与部署边界**——PyPI 关联为高置信 `Inferred`；时间能力的 Platform-only 公开可用性边界已核验，其运行效果与服务端实现、Dream 版本链实现仍未闭合。
3. **§5 评测层**——等 §4 有了判据再动，否则只能罗列各方说法。

### 下一轮推荐研究任务

1. 继续核对未覆盖的 OSS 写入入口与后台路径，判断哪些结论可以扩大到 SDK 层。
2. 保持 Platform 公开契约与服务端实现的边界；不调用真实 API，不把客户端参数当作服务端实现。
3. 继续读代表性 vector store 的 `keyword_search()` 与依赖退化路径，判定 hybrid retrieval 的真实适用范围。
4. （暂缓，当前只读范围不执行）设计包含真实 embedding / 本地 vector store 的新旧事实实验，验证 semantic candidate、threshold 和 boost 的端到端关系。
5. 核三条 OSS 缺陷的原始 issue 与适用版本；LOCOMO 论文本体与争议一手材料随后处理。

---

## 8. 与现有文档的关系

- [`general.md`](./general.md)：保留第一轮来源线索，2026-10-02 已校准状态词、版本边界与争议拆分；不作为当前权威状态表。
- [`source.md`](./source.md)：一手来源摘录、核验路径、版本语义。
- [`evidence.md`](./evidence.md)：Claim-Source 对照层，每轮核验结论先沉淀于此。
- [`../../../candidates.md`](../../../candidates.md)：Mem0 作为候选对象的队列条目与研究理由；本文件压实的结论应回写其状态。
- [`../../../memory-taxonomy-conflicts.md`](../../../memory-taxonomy-conflicts.md)：分类体系冲突专题。本文件不承接 taxonomy 议题（`general.md` 末尾的 CoALA 合流仍走该文件）。
- 未来的 `overview.md` 与带版本标识的对象正文：按 `research-artifacts.md` §3.1.1 在主线压实后提炼；本文件继续负责未闭环问题与后续推进。
