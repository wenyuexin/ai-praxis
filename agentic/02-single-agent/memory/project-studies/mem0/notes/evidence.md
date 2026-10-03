# Mem0 — Claim-Source 对照

> 本文件是从 `notes/` 过程材料中整理出的 **claim-source 对照层**（[`research-artifacts.md`](../../../../../../docs/contributing/rules/research-artifacts.md) §3.8），不是杂项 notes。
> 来源摘录与版本语义见 [`source.md`](./source.md)；问题级递进见 [`deep-research-question-tree.md`](./deep-research-question-tree.md)。
> 状态词表见 [`evidence-assessment-rules.md`](../../../../../../docs/contributing/rules/evidence-assessment-rules.md) §3。
> **Algorithm Basis**：官方 OSS `v2 → v3` migration 文档（机制代次）
> **Package Basis**：本地 `main` 分支 `pyproject.toml` 为 `mem0ai 2.2.1`（`Observed`）
> **Core Release / Commit**：`v2.2.1` → `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`（`Verified`）
> **Observed At**：2026-10-01 初轮；2026-10-02 Python 增量；2026-10-03 TypeScript / Dream 回写与边界复核，其他网页未全量复查 · **Drift Risk**：`high`
> 同 commit 另有 `ts-v3.3.1` 标签；Python 结论不自动外推 TypeScript，已单独核对的 TS 范围见 M3e / `source.md` §1.3.7。

## 1. 对照总表

| ID | Claim | Status | 来源 |
|---|---|---|---|
| M1 | 框架无关记忆层，`add()` / `search()` 两个主 API，跨会话记住用户 | `Verified` | README + 多篇二手一致 |
| M2 | 固定 Python `v2.2.1` commit 的普通同步/异步 `infer=True` 自动抽取只生成 ADD history 事件；显式 update/delete API 仍存在，实体索引也会更新 | `Observed` | `source.md` §1.3、§1.3.5 |
| M2a | 普通自动抽取以原始文本 MD5 对本次最多 10 条召回记录的已有 hash 与当前 batch 去重，不是全库或语义去重 | `Observed` | `source.md` §1.3.5，事实 2 |
| M2b | additive prompt 要求语义去重、保留变化语境和关联矛盾旧记忆；这些是指令，不是运行效果保证 | `Observed` | `source.md` §1.3.5，事实 1 |
| M2c | 普通同步/异步 OSS 写入未消费 LLM 输出的 `linked_memory_ids`；entity store 的同名字段保存实体到记忆集合的关联，不是该 LLM 记忆间关系 | `Observed` | `source.md` §1.3.5，事实 3–4 |
| M2d | Platform 官方 Dream 文档契约称矛盾新事实会使旧记忆 `superseded` 并链接替代记忆，重复或更丰富的新事实可触发 `merged`；默认读取仍可返回 superseded、隐藏 merged，`latest_only=true` 只保留 active，`include_merged=true` 可纳入 merged；OpenAPI 声明 `delete_linked=true` 删除 linked memories 并可返回 `cascade_count`。SDK 参数转发另为客户端观察；服务端内部建链与执行仍未公开 | `Verified`（公开契约）；`Observed`（客户端） | `source.md` §1.3.8；对照 §1.3.5 事实 5；固定 commit `docs/platform/features/dream.mdx`、`docs/openapi.json`、`docs/changelog/sdk.mdx` |
| M2e | Platform 官方文档声明 Temporal Reasoning 与 Memory Decay 为 Platform-only；固定 Python OSS 源码拒绝 `reference_date` 且未见对应执行逻辑；真实 API 运行效果尚未验证 | `Verified`（Platform 契约）；`Observed`（OSS 源码）；`Unverified`（运行效果） | 固定 commit `docs/platform/features/*`；`mem0/memory/main.py`；官方博客 |
| M2f | PyPI `mem0ai==2.2.1` 工件与固定 release commit 的版本、release subject 和上传时间高度吻合，但 PyPI JSON 无 Git SHA，未下载工件核验 | `Inferred`（高置信） | PyPI JSON；固定 commit tags / release subject |
| M3a | 论文 v1 §2.1 / Figure 2 将旧版基础 Mem0 正式管线命名为 **extraction + update**；迁移文档另用 `extract + merge` 概述旧机制，不能替换论文阶段术语 | `Verified` | `arXiv:2504.19413v1` §2.1、Figure 2；Platform v2→v3 migration 表作旁证 |
| M3b | 论文旧版 update 阶段先检索 top-*s* 语义相似记忆，再由 LLM function-calling / tool call 在 **ADD / UPDATE / DELETE / NOOP** 四值中选择；Appendix B Algorithm 1 交叉确认 | `Verified` | `arXiv:2504.19413v1` §2.1、Appendix B Algorithm 1 |
| M3c | 论文旧版基础 Mem0 用 `DELETE` 移除被新信息矛盾的记忆；Mem0g 图 update 则将过时关系标为 `invalid` 而非物理删除 | `Verified` | `arXiv:2504.19413v1` §2.1–§2.2、Figure 3；这是论文设计描述 |
| M3d | 当前 Python OSS 的 `infer=False` 与 procedural 是直接追加路径，vision 只是预处理；普通 `infer=True` 的 ADD-only 结论不能直接扩大到这些入口 | `Observed` | `source.md` §1.3.6；固定 commit `mem0/memory/main.py` |
| M3e | 固定 commit 的 TypeScript OSS 普通 `infer=true` 在已核对机制上与 Python 普通路径相似：ADD-only、候选局部 MD5 去重、实体索引更新；TypeScript 没有观察到 Python procedural 入口 | `Observed` | `source.md` §1.3.7；固定 commit 的 TS memory、prompts、scoring 源码 |
| M3f | 不同 OSS 写入入口的操作集合和去重/实体索引行为不同，因此普通 `infer=True` 观察不能代表整个 Python OSS | `Inferred` | M3d；`source.md` §1.3.6 |
| M4 | 固定 `v2.2.1` commit 同步检索依次获取语义、BM25、实体信号；仅以语义结果构造候选集，再融合打分，并非三路并行召回取并集 | `Observed` | `source.md` §1.3 固定源码核验 |
| M4a | `v2.2.1` 的 threshold 在 hybrid 融合前门控 semantic score；BM25 / entity 不能把语义分数低于门槛的候选重新纳入 | `Observed` | `source.md` §1.3.2；`mem0/utils/scoring.py` |
| M4b | `v2.2.1` `score_and_rank` 不读取 `created_at`；只交换候选时间字段不会改变排序；信号融合只作用于已进入 semantic 候选集的记录 | `Observed` | `source.md` §1.3.3 源码函数级受控验证 |
| M5 | 图记忆已从 OSS **移除**，成为 Platform 内建常开特性；OSS 无替代、`relations` 字段不再返回 | `Verified` | OSS 迁移指南 |
| M6 | README 的 LoCoMo 92.5 等分数来自**托管平台**，含 OSS 不可用的专有优化 | `Verified` | README 逐字免责声明 |
| M7 | 论文 "91% lower p95 latency" / ">90% token 节省" 的比较对象是 **full-context**，非同类记忆系统 | `Verified` | arXiv:2504.19413 摘要 |
| M8 | 退役未区分版本、自动抽取、显式删除与衰减的「Mem0 遗忘仅按时间」概括 | `Deprecated` | M2 的显式 CRUD 边界、M3a 的旧版变更操作、M15 的后续能力声明；不作所有版本穷举结论 |
| M9 | `#4956` 报告 v3 OSS 可变事实共存、旧值可能排前，并归因于缺少 recency 信号；不是本研究的复现或所有版本结论 | `Observed` | `mem0ai/mem0#4956`（2026-10-01 抓取为 label `bug`、Open） |
| M10 | 抽取与存储之间缺少 grounding 校验；条目以同等置信度存储 | `Observed` | `mem0ai/mem0#4573` |
| M11 | 过度提取的主因在**写入阶段**而非检索 | `Observed` | `mem0ai/mem0` Discussion #4289 |
| M12 | Node / TS SDK “同步阻塞 30–40s”说法未找到可信一手支持；已核源码显示 Platform v3 `add()` 是异步请求，服务端处理/索引可见性也按异步流程处理，但不能据此否定所有历史版本或自托管配置的延迟问题 | `Unverified` | `source.md` §3.6；固定 TS client 与 integration test |
| M13 | LOCOMO 分数之争的主对手是 **Zep**；Zep 原博客已将其结果修正为 75.14% ± 0.17，Mem0 在 issue #5 中主张其复核为 58.44% ± 0.20，而 Mem0 论文报告的 Zep 数字为 65.99% ± 0.16；实现、计分和题类口径仍存在争议 | `Observed` | Zep 原博客；`getzep/zep-papers#5`；Mem0 论文/评测材料 |
| M14 | Letta 的质疑是另一条独立争议：Mem0 论文中的 MemGPT baseline 难以原生复现；Letta 原博客报告其 Filesystem 方案在 GPT-4o mini、受限工具规则下取得 74.0% LoCoMo | `Observed` | Letta 原博客；Letta CEO 公开表态；不等于同配置复现或普遍优越 |
| M15 | 5 月版新增 Temporal Reasoning 与 Memory Decay，temporal 类 +3.8、multi-session +1.5 | `Verified` | 官方博客 |
| M16 | Mem0 的一致性机制必须按版本/部署形态分层：论文旧版是 update-time LLM mutation；固定 Python/TypeScript OSS 普通自动抽取是 ADD-only + 候选局部去重 + entity index；Platform Dream 公开契约包含 add-time supersede/merge 与 read-time `latest_only` / `include_merged` 过滤；不能概括为一致性责任单向转移到读取期 | `Inferred`（机制分层由多条证据支持，跨层因果仍未证实） | M2–M4b、M3a–M3f、M2d–M2e；`source.md` §1.3.5–§1.4 |
| M17 | LOCOMO 原论文明确是长、多 session 对话上的离线 QA / summarization / multimodal benchmark：覆盖 single-hop、multi-hop、temporal、open-domain、adversarial；不提供事实覆盖、旧值复活、矛盾版本共存或持续写入污染的专门受控指标 | `Verified`（任务定义与数据结构）；`Inferred`（由缺少专门指标推出的适用边界） | `source.md` §3.1；`arXiv:2402.17753v1` §3–§4、§8 |
| M18 | LongMemEval 原论文明确包含 knowledge-update、temporal-reasoning、multi-session 和 abstention；它比 LOCOMO 更直接测试更新到后来的用户状态，但其历史构造避免无关冲突，不能单独作为旧事实复活或长期矛盾共存基准 | `Verified`（任务定义与构造）；`Inferred`（适用边界） | `source.md` §3.2；`arXiv:2410.10813v2` §3、Appendix A |
| M19 | 当前文档声明的 92.5 LoCoMo / 94.4 LongMemEval 属于 managed Platform；固定评测子模块快照另有可复核的 Platform 91.6 / 93.4 artifacts，二者不是简单四舍五入差异；固定快照的 OSS 公开结果只覆盖 LongMemEval，未形成与 Platform 92.5 LoCoMo 对等的同配置闭环 | `Verified`（结果归属、固定 artifact 与框架能力）；`Observed`（当前文档数字）；`Unverified`（92.5/94.4 的具体 run provenance）；`Inferred`（两组数字属于不同 run 或服务端版本） | `source.md` §3.3–§3.3.1；固定评测子模块 commit `4b61c5d…` 的 `results/platform/*`、`results/oss/*`、README 与评测文档 |
| M20 | Zep ↔ Mem0 与 Letta ↔ Mem0 是两条独立的评测争议：前者围绕 LoCoMo 实现/计分与 58.44、65.99、75.14 等相互冲突数字，后者围绕 MemGPT baseline 可复现性与 Letta Filesystem 74.0 结果；一手材料支持“争议主张存在”，不支持合并成单一胜负结论 | `Observed` | `source.md` §3.4；`getzep/zep-papers#5`；Zep 原博客；Letta 原博客 |
| M21 | `dial481/locomo-audit@9493fb4…` 固定快照包含原始数据 SHA 声明与校验脚本、逐题审计标注、审计报告、结果重算脚本及统计分析；审计者将 1,540 个非 adversarial 问题中的 156 项标为问题，其中 99 项归类为 score-corrupting、57 项为 citation-only。上述是仓库内容和审计者分类的观察，不是对数据集事实错误数的独立确认；审计者关于 judge 宽松性、调整后分数及“公开高分失效”的结论仍未独立抽查或复算 | `Observed`（固定材料、作者分类和脚本配置）；`Unverified`（分类正确性、统计结论与对具体系统分数的影响） | `source.md` §3.5；固定 commit 的 `AUDIT_REPORT.md`、`errors.json`、`scripts/verify_sha256.py`、`results-audit/audit_results.py` |

## 2. 需要展开的条目

### 2.1 M16 主线：改为版本/部署形态分层

M16 仍保持 `Inferred`，但主线不再是“写入期责任单向转移到读取期”。目前更稳妥的分层是：

- **论文旧版**：`extraction + update`，由 LLM 在 ADD / UPDATE / DELETE / NOOP 中做 update-time mutation；基础 Mem0 的 DELETE 处理矛盾，Mem0g 以 `invalid` 关系保留时间语义。
- **固定 Python / TypeScript OSS 普通自动抽取**：单趟 ADD-only，配合 prompt 级筛选、候选局部 MD5 去重和 entity index；普通路径没有从 LLM `linked_memory_ids` 建立已确认的事实 supersession 链。
- **Platform Dream**：公开契约称新增记忆时评估 supersede / merge；旧记忆可标为 `superseded` 或 `merged`，读取时再用 `latest_only` / `include_merged` 控制可见性。这里同时存在写入期状态标记和读取期过滤。
- **Platform Synthesis**：公开文档描述为后台定时综合，不应与 add-time supersede / merge 混为同一机制。

因此，当前证据支持“不同代次和部署形态组合了不同的写入变更、状态标记与读取过滤”，不支持“新版普遍把一致性责任转移到读取期”，也不支持把 OSS 的 ADD-only 直接外推到 Platform 服务端。

- **仍未闭合的因果问题**：OSS 的 hybrid retrieval 是否承担了旧版 update 被移除后的冲突处理责任；当前只有并列架构描述，没有因果证据。
- **仍未闭合的服务端问题**：Platform 如何判断 supersede / merge、如何建立 `replaced_by` / `linked_memory_ids` 关系、`delete_linked` 如何遍历，以及失败和事务语义。
- **仍未闭合的端到端问题**：LLM 抽取质量、矛盾事实实际共存率，以及各类检索结果对当前事实的实际排序效果。

### 2.2 M9 是 §4「完备性层」的第一条一手证据

`#4956` 的报告场景与问题树 §4.1 相符；label / Open 状态只记录到 2026-10-01，不能视为维护者已确认根因：

- 矛盾共存：3 个月前 "I work at Company A"、今天 "I now work at Company B"，两条都在
- 旧事实复活：查 "Where does the user work?"，Company A 可能排前
- 机制归因（issue 原话）：**打分信号不含 recency**

M15 的时间能力是相关线索，但不能仅凭发布时间或名称认定用于修复本 issue。报告适用版本、Platform / OSS 边界与后续修复状态仍待核验。该报告既不能证明每次去重失败，也不能由局部去重机制反推矛盾必被消解。

### 2.3 M2e / M2f：Platform 与发布制品边界

- M2e 将 Temporal Reasoning / Memory Decay 限定为 Platform-only；OSS 不能因同一 release commit 或同一 SDK 仓库而继承这些能力。
- Dream / `latest_only` / `delete_linked` 已有公开产品/API 契约，见 M2d 与 `source.md` §1.3.8；客户端类型、注释与 HTTP 转发是另一个证据层。服务端如何构建 superseded chain、执行状态判定及递归删除，仍不能从这些材料确认。
- M2f 只能记录高置信版本关联，不把 PyPI 上传时间相近升级成“工件由该 commit 构建”的 `Verified` 事实。

### 2.4 M5 是版本敏感性的实例，直接改写既有记录

`general.md` 原记「三层 scope + 向量 + 图（Mem0g）混合后端」为 `Observed`。该判断基于 arXiv:2504.19413（2025 年 4 月，即 v2 时期）。**在 v3 OSS 中图后端已被移除**，因此：

- 描述 **v2 / 论文语境**时，"向量 + 图混合后端"成立
- 描述 **v3 OSS**时不成立；图记忆只在 Platform
- 这正是 `research-artifacts.md` §3.1.1 所说"版本边界本身就是对象语义的一部分"——未来正文必须按版本分开写，不能合成一句

### 2.5 M13 / M14：两条独立的评测争议，已读取到主张层

`general.md` 原记「LOCOMO 之争（Letta 质疑 Mem0 未公平跑对手）」为 `Conflicting`。本轮已读取 Zep 与 Mem0、Letta 与 Mem0 的一手争议材料和主要方法分歧，确认这是**两条独立争议**，且不能把任何一方的分数或方法解释直接升级为最终事实。

**争议一 · Zep ↔ Mem0**

- Zep 原博客（标题「Lies, Damn Lies, & Statistics」）曾报告 Zep 的 LOCOMO 分数及其与 Mem0 的比较
- Mem0 方在 GitHub issue 中质疑 user model、timestamp、parallel search 与计分口径，并指出 Category 5 分母处理可能影响结果
- **Zep CEO Daniel Chalef 后续承认原计算有误并修正文中的结果**，修正值为 `75.14% ± 0.17`
- Mem0 侧在 issue 中给出对 Zep 的另一组复核数字 `58.44% ± 0.20`，论文中还存在 `65.99% ± 0.16` 的历史报告
- 上述数字涉及实现、计分、题类、分母和时间字段等方法差异；当前尚未完成逐项核验和同条件复现

**争议二 · Letta ↔ Mem0**

- Letta 方质疑 Mem0 论文中的 MemGPT baseline 难以在原生 MemGPT agent 上复现，并认为公开材料不足以重建其运行方式
- Letta 报告 Filesystem + 文件检索方案取得 `74.0%` LOCOMO 结果
- 该争议的核心是**基线可复现性与配置可比性**，不能简化为“未公平跑对手”，也不能仅凭单方结果推出普遍优越

**落位影响**：该冲突跨 Mem0 / Zep / Letta 三个对象，`mem0/` 层范围覆盖不了，按 [`metadata-files.md`](../../../../../../docs/contributing/rules/metadata-files.md) §6.1 可评估是否落 `memory/conflict.md`。当前已读取到双方争议主张层，但实现、计分、配置和同条件复现仍未闭合，因此本轮仍不创建 `conflict.md`。

### 2.6 M12 应从缺陷清单降为待溯源线索

「Node 同步阻塞 30–40s」原出自二手转述，本轮仍未找到对应的一手 issue、release note 或性能测量。可核源码显示 Platform v3 TypeScript client 的 `add()` 是 `async` HTTP 请求，integration test 将服务端处理和索引可见性描述为异步，并采用轮询等待；这不足以否定历史版本或自托管配置可能存在延迟，但也不支持“同步阻塞 30–40s”的具体说法。

处置：保持 `Unverified`，不再把它作为已确认 OSS 缺陷；若后续仍无一手来源，应从缺陷清单移除，仅保留为待溯源线索。

### 2.7 M21：第三方 LoCoMo 审计可作为协议层争议入口

`dial481/locomo-audit` 的固定 commit `9493fb4b4af4256ed17a18e8fd0b3cfdeec29539` 提供了原始数据 SHA 声明与校验脚本、逐题审计标注、审计报告、统计脚本、结果重算脚本和不同实现的配置差异表。固定快照的 `AUDIT_REPORT.md` 将 1,540 个非 adversarial 问题中的 156 项标为问题：99 项 score-corrupting、57 项 citation-only；这是审计者的分类，不等于本研究已确认数据集存在 99 个事实错误。

它可以支持“存在独立的 LoCoMo 数据与评测协议审计”，并提醒不能把不同模型、prompt、judge、Category 5 处理和分母口径下的分数直接横比。`audit_results.py` 的结果重算路径默认依赖外部 published eval results、三次 temperature=0 的 LLM judge 调用和 API key；本轮未执行脚本、下载被审计系统输出或复现 judge。

但审计仓库中的“156/99/57 分类是否正确”“judge 接受含糊错误答案的比例”以及“公开高分因此失效”等结论仍属于审计者的主张；本轮未逐题抽查、未重算、未运行其脚本。因此不把它升级为 Mem0、Zep 或其他系统分数已被推翻的事实，也不据此创建 `conflict.md`。

## 3. 已回写与仍待回写的既有记录

2026-10-02 已将以下第一轮记录同步到 `general.md`、问题树与候选队列：

- 两处非法状态词：「需重限定」→ 拆为 M6（`Verified`，官方免责声明）+ M7（`Verified`，比较对象是 full-context）；「疑为错 / 版本混淆」→ M8（`Deprecated`）
- 争议对手修正：Letta → 以 Zep 为主争议，Letta 为独立第二条（M13 / M14）
- 「向量 + 图混合后端」需加版本限定（M5）
- 三条 OSS 缺陷拆分：过度提取 → M10 / M11（`Observed`）；置信度 → M10（`Observed`）；Node 阻塞 → M12（`Unverified`，候选移除）
- 补 `Version Basis` / `Observed At` / `Drift Risk`（本文件与 `source.md` 已有；`general.md` 保留历史笔记性质，不重复伪造版本表）

仍待回写或继续核验：

- `general.md` 中的第一轮来源列表仍包含二手材料，保留为研究过程记录；不作为当前权威来源表。
- M13 / M14 的 Zep issue、原博客与 Letta 原始材料已读取到争议主张层；若要创建 `memory/conflict.md`，还需继续核对实现、计分和同条件复现。
- M12 的 Node / TS 同步阻塞说法仍为 `Unverified`；当前只保留为待溯源线索，若继续找不到一手来源应从缺陷清单移除。
- M21 的第三方 LoCoMo 审计已固定 commit；156/99/57 是审计者的分类观察，逐题错误判断、judge 宽松性、调整后分数和统计结论尚未独立抽查或复算。
- 旧版论文已从“全文未读”更新为 M3a–M3c；未来若提炼正文，保留“论文设计”与“当前实现”两条版本边界。
