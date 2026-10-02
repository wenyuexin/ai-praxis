# Mem0 — Claim-Source 对照

> 本文件是从 `notes/` 过程材料中整理出的 **claim-source 对照层**（[`research-artifacts.md`](../../../../../../docs/contributing/rules/research-artifacts.md) §3.8），不是杂项 notes。
> 来源摘录与版本语义见 [`source.md`](./source.md)；问题级递进见 [`deep-research-question-tree.md`](./deep-research-question-tree.md)。
> 状态词表见 [`evidence-assessment-rules.md`](../../../../../../docs/contributing/rules/evidence-assessment-rules.md) §3。
> **Algorithm Basis**：官方 OSS `v2 → v3` migration 文档（机制代次）
> **Package Basis**：本地 `main` 分支 `pyproject.toml` 为 `mem0ai 2.2.1`（`Observed`）
> **Core Release / Commit**：`v2.2.1` → `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`（`Verified`）
> **Observed At**：2026-10-01 · **Drift Risk**：`high`

## 1. 对照总表

| ID | Claim | Status | 来源 |
|---|---|---|---|
| M1 | 框架无关记忆层，`add()` / `search()` 两个主 API，跨会话记住用户 | `Verified` | README + 多篇二手一致 |
| M2 | 固定 `v2.2.1` commit 的普通同步 `infer=True` 自动抽取只持久化 ADD 事件；显式 update/delete API 仍存在，实体索引也会更新 | `Observed` | `source.md` §1.3 固定源码核验 |
| M3a | v2 写入期为两阶段 extract + merge，含 ADD / UPDATE / DELETE | `Verified` | Platform 迁移表 Before 列 |
| M3b | v2 更新阶段由 LLM function-call 在 ADD / UPDATE / DELETE / **NOOP** 四值间决策 | `Unverified` | 论文摘要只写 "extracting, consolidating"；四值表述待全文核 |
| M4 | 固定 `v2.2.1` commit 同步检索依次获取语义、BM25、实体信号；仅以语义结果构造候选集，再融合打分，并非三路并行召回取并集 | `Observed` | `source.md` §1.3 固定源码核验 |
| M4a | `v2.2.1` 的 threshold 在 hybrid 融合前门控 semantic score；BM25 / entity 不能把语义分数低于门槛的候选重新纳入 | `Observed` | `source.md` §1.3.2；`mem0/utils/scoring.py` |
| M4b | `v2.2.1` `score_and_rank` 不读取 `created_at`；只交换候选时间字段不会改变排序；信号融合只作用于已进入 semantic 候选集的记录 | `Observed` | `source.md` §1.3.3 源码函数级受控验证 |
| M5 | 图记忆已从 OSS **移除**，成为 Platform 内建常开特性；OSS 无替代、`relations` 字段不再返回 | `Verified` | OSS 迁移指南 |
| M6 | README 的 LoCoMo 92.5 等分数来自**托管平台**，含 OSS 不可用的专有优化 | `Verified` | README 逐字免责声明 |
| M7 | 论文 "91% lower p95 latency" / ">90% token 节省" 的比较对象是 **full-context**，非同类记忆系统 | `Verified` | arXiv:2504.19413 摘要 |
| M8 | 「Mem0 遗忘仅按时间」在任一版本都不成立 | `Deprecated` | v2 有 LLM 决策的 DELETE；v3 只累积不覆盖 |
| M9 | ADD-only 使可变状态类事实矛盾共存，且检索打分不含 recency，旧事实可能被排在前列 | `Observed` | `mem0ai/mem0#4956`（label `bug`，Open） |
| M10 | 抽取与存储之间缺少 grounding 校验；条目以同等置信度存储 | `Observed` | `mem0ai/mem0#4573` |
| M11 | 过度提取的主因在**写入阶段**而非检索 | `Observed` | `mem0ai/mem0` Discussion #4289 |
| M12 | Node / TS SDK 同步阻塞 30–40s | `Unverified` | 本轮未核到任何一手 issue |
| M13 | LOCOMO 分数之争的主对手是 **Zep**；Zep CEO 已承认算错并把 84% 修正为 75.14% ± 0.17 | `Observed` | 多篇二手一致 + 指向 `getzep/zep-papers#5`（一手未读） |
| M14 | Letta 的质疑是另一条独立争议：Mem0 论文中的 MemGPT 基线数字不可复现 | `Observed` | Letta CEO 本人社媒发言 |
| M15 | 5 月版新增 Temporal Reasoning 与 Memory Decay，temporal 类 +3.8、multi-session +1.5 | `Verified` | 官方博客 |
| M16 | 主线假设：自动抽取的一致性处理减少写入期显式变更，部分转由读取期排序与过滤承担 | `Inferred`（需版本限定，见 §2.1） | M2 + M4 + M5 + M15 |

## 2. 需要展开的条目

### 2.1 M16 主线：保持 `Inferred`，并明确版本限定

问题树 §0.2 将“自动抽取的一致性处理减少写入期显式变更、部分转由读取期排序与过滤承担”标为 `Inferred`。本地 `v2.2.1` 源码核验增强了这一较窄表述，但没有把它升级为 `Verified`：它仍是跨文档与源码的机制解释，不是官方原句；显式 CRUD 仍存在，且时间机制未在该 OSS 搜索路径中出现。

- **反驳条件 1（新版仍在别处做写入期消解）→ 尚未完全排除。** 本地 `v2.2.1` 的普通 `infer=True` 自动抽取路径未执行 UPDATE / DELETE，但类级显式 CRUD 仍存在；后台路径与 Platform 内部实现也未穷尽。
- **反驳条件 2（检索增强与 ADD-only 无因果）→ 尚未排除。** 官方博客并列两项架构变更，只能证明共同出现，不能证明检索增强承担了被移除的冲突消解责任。
- **反驳条件 3（ADD-only 只是某模式默认值）→ 文档提供反证，源码仍待核验。** 两份迁移指南将其描述为抽取模型变化，但配置分支和其他写入入口尚未检查。

**但主线要加一条限定**：M15 显示 5 月版又补了 **Memory Decay**。所以准确表述不是"责任被转移到读取期就结束了"，而是：

> 官方迁移文档描述抽取路径移除显式 UPDATE / DELETE，并增强检索；后续博客又描述时间推理与记忆衰减。将其解释为“一致性责任向读取侧转移”仍是待检验假设。后续能力的部署范围、执行位置及改动动机不能由发布时间先后推定。

后续补入写入期或衰减机制可作为继续调查的线索，但不能单凭新增功能断言读取期补偿不完备；还需设计说明、源码或实验支持。

### 2.2 M9 是 §4「完备性层」的第一条一手证据

`#4956` 的场景与问题树 §4.1 的推测**逐条吻合**，且由官方仓库标为 `bug` 并保持 Open：

- 矛盾共存：3 个月前 "I work at Company A"、今天 "I now work at Company B"，两条都在
- 旧事实复活：查 "Where does the user work?"，Company A 可能排前
- 机制归因（issue 原话）：**打分信号不含 recency**

注意与 M15 的张力：5 月版加了 Temporal Reasoning 与 Memory Decay，理论上正是针对该问题。**issue 开于 2026-04-24（4 月版时期）且仍 Open，是否已被 5 月版修复未知**——这是下一轮必须确认的点，不能假定已修。

### 2.3 M5 是版本敏感性的实例，直接改写既有记录

`general.md` 原记「三层 scope + 向量 + 图（Mem0g）混合后端」为 `Observed`。该判断基于 arXiv:2504.19413（2025 年 4 月，即 v2 时期）。**在 v3 OSS 中图后端已被移除**，因此：

- 描述 **v2 / 论文语境**时，"向量 + 图混合后端"成立
- 描述 **v3 OSS**时不成立；图记忆只在 Platform
- 这正是 `research-artifacts.md` §3.1.1 所说"版本边界本身就是对象语义的一部分"——未来正文必须按版本分开写，不能合成一句

### 2.4 M13 / M14 修正既有记录：争议对手记错了

`general.md` 原记「LOCOMO 之争（Letta 质疑 Mem0 未公平跑对手）」为 `Conflicting`。本轮发现这是**两条独立争议**，且主争议对手是 Zep：

**争议一 · Zep ↔ Mem0（主争议，已部分收敛）**

- Zep 原博客（标题「Lies, Damn Lies, & Statistics」）称 Zep 在 LOCOMO 得约 84%、领先 Mem0 约 24%
- Mem0 方在 GitHub issue 指出算术错误：Category 5 答案计入分子但排除在分母外，虚高约 25 个百分点
- **Zep CEO Daniel Chalef 承认并修改原文**："we erred in how we calculated Zep's LoCoMo score... corrected result is 75.14% ± 0.17"
- 一手记录位置：`getzep/zep-papers` issue **#5**（本轮未读）
- Mem0 侧报出的 Zep 分数在不同二手来源中为 65.99% 或 58.44%，**口径不一致，需一手澄清**

**争议二 · Letta ↔ Mem0（独立，未收敛）**

- Letta CEO 指 Mem0 论文中的 MemGPT 基线数字"baseless"：无法原生在 MemGPT agent 上跑 LOCOMO，询问 Mem0 团队未获回应
- Letta 用 Filesystem + 文件检索得 74%，高于其引用的 Mem0 68.5%
- 注意：这条质疑的靶子是**基线可复现性**，不是"未公平跑对手"这个笼统说法

**落位影响**：该冲突跨 Mem0 / Zep / Letta 三个对象，`mem0/` 层范围覆盖不了，按 [`metadata-files.md`](../../../../../../docs/contributing/rules/metadata-files.md) §6.1 应落 `memory/conflict.md`。但**一手材料尚未读到**（`getzep/zep-papers#5`、Zep 原博客、Mem0 的 issue 回应），现阶段仍留在本文件，不急着建 `conflict.md`。

### 2.5 M12 应考虑降级或移除

「Node 同步阻塞 30–40s」原出自二手转述的 GitHub issue。本轮定向搜索**未找到任何对应的一手 issue**；反而有二手教程称 Mem0 的抽取是异步、不阻塞 API 响应（但该教程用的是 cloud client，与自托管 OSS 不同，不足以反证）。

处置：保持 `Unverified`，并在下一轮定向搜 `mem0-ts` / Node SDK 的 issue。**若仍无一手来源，应从缺陷清单移除**，而不是长期挂着一条无源断言——按 `AGENTS.md` §4 自检，这正是"没有来源的断言读起来像定论"的风险点。

## 3. 待回写的既有记录

以下修正应在下一轮随 `general.md` 瘦身一并落地（本轮只在此登记，不改 `general.md`）：

- 两处非法状态词：「需重限定」→ 拆为 M6（`Verified`，官方免责声明）+ M7（`Verified`，比较对象是 full-context）；「疑为错 / 版本混淆」→ M8（`Deprecated`）
- 争议对手修正：Letta → 以 Zep 为主争议，Letta 为独立第二条（M13 / M14）
- 「向量 + 图混合后端」需加版本限定（M5）
- 三条 OSS 缺陷拆分：过度提取 → M10 / M11（`Observed`）；置信度 → M10（`Observed`）；Node 阻塞 → M12（`Unverified`，候选移除）
- 补 `Version Basis` / `Observed At` / `Drift Risk`（本文件头部已有，`general.md` 待同步）
