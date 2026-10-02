# Mem0 — 一手来源建档与版本语义

> 本文件承接 `mem0` 研究的上游来源摘录、核验路径与版本边界（[`research-artifacts.md`](../../../../../../docs/contributing/rules/research-artifacts.md) §3.9）。
> Claim-Source 对照与状态判定见 [`evidence.md`](./evidence.md)；问题级递进见 [`deep-research-question-tree.md`](./deep-research-question-tree.md)。
> 来源只记可还原形式（arXiv 编号、官方文档 URL、`owner/repo` + issue 号），不写本机路径（[`traceability-rules.md`](../../../../../../docs/contributing/rules/traceability-rules.md) §3.1）。

## 0. 版本锚点（Version Basis / Observed At）

> **Observed At**：2026-10-01（本轮核验日期，非文件修改时间）
> **Drift Risk**：`high`——2026 年 4 月与 5 月两次算法更新，OSS 有 v2→v3 破坏性变更

| 项 | 值 | 状态 |
|---|---|---|
| OSS 算法代次 | v2 → **v3**（官方迁移指南以此命名） | `Verified` |
| 本地 `main` 分支 `pyproject.toml` 的 Python 包版本 | `mem0ai 2.2.1` | `Observed` |
| 本地核心库 release tag / commit | `v2.2.1` → `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd` | `Verified` |
| 仓库最新 release | `deepseek-plugin-v0.3.2`（2026-09-23） | `Verified` |
| 规模 | 65.5k–65.7k star / 7.7k fork（抓取期间数值漂移） | `Observed` |

**Stop-line 状态**：本地源码已固定到核心库 `v2.2.1` commit，且已完成 `add()` / `search()` 主路径的静态核验；仍不能把迁移文档的算法代次直接等同于所有部署形态，也未做运行时实验。此前 `v2.1.0` 是远程源码核验坐标，当前本地研究以 `v2.2.1` 为主；PyPI 发布文件对应关系仍未单独读取官方 JSON。

### 0.1 算法代次与包版本暂不能直接等同

- 官方 OSS 迁移文档把 `v2 → v3` 作为**迁移 / 算法代次**命名，文档明确描述 ADD-only、hybrid retrieval、entity-aware retrieval 等变化。
- 此前远程 `main` 快照的 `pyproject.toml` 为 `2.1.0`；当前本地 `main` 已前进到 `2.2.1`。两者都只能说明源码快照的包版本字段，不能单独证明 v3 算法代次或某个 PyPI 文件的全部行为。
- 官方 GitHub tags / commit API 已将核心库 `v2.1.0` 对应到 commit `19f713408273fb1d657daa38d7b82ccf496d36d5`；该 commit 的 release bump 将 `pyproject.toml` 从 `2.0.20` 改为 `2.1.0`。这闭合了核心源码锚点，但不等于已经闭合 PyPI 文件与该 commit 的发布对应关系。
- 本地 Git 仓库当前为 `v2.2.1` / `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`，与 `v2.1.0` 的核心机制 diff 主要是：部分写入失败时只把实际持久化记录写入 history、实体链接和返回结果；另有上下文管理器与 procedural-memory 注释。未发现自动抽取或 hybrid scoring 机制改写。
- 外部 PyPI 索引本轮返回了互相不一致的版本信息（`2.2.1` 与 `3.1.0` 均被不同索引结果声称为最新），尚未直接用 PyPI JSON / release 文件闭合，暂不选其一作为核心库最终版本。
- 因此当前采用分层锚点：机制先锚到官方迁移文档的 `v2 → v3` 代次；当前核心源码行为锚到本地 `v2.2.1` commit；包安装行为仍需补可还原的 PyPI release。`v2.1.0` 仅作为前一轮远程源码对照点保留。

## 1. 一手来源

### 1.1 官方迁移指南（Platform）

`docs.mem0.ai/migration/platform-v2-to-v3` · `Status: Verified`

官方 What Changed 对照表，逐字摘录：

| What Changed | Before | After |
|---|---|---|
| Extraction | Two LLM passes (extract + merge) | Single-pass ADD-only (one LLM call) |
| Memory mutations | ADD, UPDATE, DELETE | **ADD only: nothing is overwritten or deleted** |
| Agent-generated facts | Often ignored | First-class, stored with equal weight |
| Graph memory | External graph store (Neo4j, etc.) + manual setup | Built-in and automatic; entities extracted and linked natively |
| Retrieval | Semantic (vector) only | Hybrid retrieval combining multiple signals |

文中另有小节标题：**"Memories accumulate instead of being overwritten"**。

检索兼容性口径：`score` 与 `results[]` 形状不变，但"scoring method behind the number"改为多信号融合，故绝对值漂移；**时间信号在检索内部应用，不改变客户端响应形状**。

### 1.2 官方迁移指南（OSS）

`docs.mem0.ai/migration/oss-v2-to-v3` · `Status: Verified`

- Extraction：Single-pass ADD-only（one LLM call, no UPDATE/DELETE）
- Retrieval：Multi-signal hybrid search（semantic + BM25 keyword + entity matching）
- Entity matching：自动实体抽取作为**第三路打分信号**，提升与查询共享实体的记忆
- **Graph memory moved to Platform**：外部图存储集成**从 OSS 移除**；图记忆成为 Platform 内建常开特性
- 迁移动作：移除 `enable_graph` / `graph_store` 配置；卸载 neo4j / memgraph 等外部驱动
- **`relations` 字段在 OSS 不再返回**，且"OSS has no graph memory replacement"——需要图记忆只能上 Platform

指南自述："Breaking changes ahead... a fundamentally different extraction model."

### 1.3 固定源码核验（本地 `v2.2.1`）

前置源码核验（2026-10-01，`Observed`）：本轮直接读取本地固定 commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`，没有执行运行时测试。

- 来源：`mem0/memory/main.py`。`Memory._add_to_vector_store` 第 881–1220 行：`infer=False` 直接写入；普通 `infer=True` 路径先召回最多 10 条既有记忆，再使用 additive prompt 单次调用 LLM，批量 embedding、hash 去重、插入记录、写 ADD history，最后维护实体链接。
- hash 去重范围是本次召回的既有记录和当前 batch，不是全库去重。实体索引本身会更新，不能把 ADD-only 解释为所有存储层均不可变。
- 第 1829 行以后仍有显式 `update()` / `delete()` API。ADD-only 限定自动抽取的记忆事件，并不取消公共 CRUD。
- `_search_vector_store` 第 1642–1745 行：同步依次获取 semantic、keyword/BM25、entity boost；候选集仅由 semantic results 构造，关键词和实体信号参与已有候选打分，不能描述成三路召回取并集。
- semantic over-fetch 为 `max(limit * 4, 60)`；默认过滤过期 payload 后才调用 `score_and_rank`。`score_and_rank` 使用 `semantic + bm25 + entity_boost` 的加法归一化，并在融合前用 semantic threshold 淘汰候选。
- 本地迁移文档还明确记录退化条件：缺少 spaCy 时没有实体抽取和 BM25 lemmatization；Qdrant 缺少 fastembed 时没有 BM25 keyword search；实体存储不可用时没有 entity boost。三种情况下 semantic search 仍可工作。
- 该 OSS 路径未见 Temporal Reasoning / Memory Decay 执行逻辑；`expiration_date` 过滤不能等同于它们。`add(timestamp=...)` 与 `search(reference_date=...)` 在 OSS 中显式拒绝。尚未穷尽异步、procedural、vision 等路径。

### 1.3.1 Qdrant 后端的 hybrid 依赖边界

本地 `v2.2.1` 的 `mem0/vector_stores/qdrant.py` 进一步确认：

- 新建 collection 时配置 dense vector 和名为 `bm25` 的 sparse vector slot；既有旧 collection 若没有该 slot，会记录 warning 并关闭 BM25，但 semantic search 继续工作。
- BM25 encoder 通过懒加载 `fastembed` 的 `Qdrant/bm25` 模型获得；没有 `fastembed`、模型加载失败或编码失败时，`keyword_search()` 返回 `None`。
- 插入记忆时，Qdrant 后端根据 `text_lemmatized` 或 `data` 生成 sparse vector；没有可用 BM25 encoder 时仍可插入 dense vector。
- `keyword_search()` 使用 Qdrant 的 `bm25` named vector 查询；它不是 Python 侧重新扫描文本。
- `search_batch()` 优先使用 Qdrant batch API，失败时退化为逐条 `search()`。

因此“hybrid retrieval”在代码里是**条件能力**：新 collection + 可用 fastembed + 后端支持时才包含 BM25；否则仍是可工作的 semantic-only 或 semantic + entity 路径。不能仅根据 SDK 版本号假定所有部署都启用了三种信号。

### 1.3.2 评分公式与研究含义

`Observed`：`v2.2.1` 的 `mem0/utils/scoring.py:60` 中，先用原始 semantic score 与 threshold 比较，未通过即丢弃。通过后计算 `min((semantic + bm25 + entity_boost) / D, 1.0)`；`D = 1 + bool(bm25_scores) + 0.5 * bool(entity_boosts)`。分母由本次调用的两个分数字典是否非空决定，不是逐候选判断匹配信号，也不只是判断依赖是否安装。

`Inferred`：关键词或实体信号不能挽救语义候选池之外、或语义分数低于门槛的事实。当前评分函数没有读取记忆的新旧时间或执行矛盾判断，因此“更相关”不能直接推成“当前事实正确”。需用受控新旧事实样本继续检验，不能据此声称实际检索必然失败。

### 1.3.3 评分函数受控验证（源码函数级）

本轮在不安装依赖、不修改 Mem0 仓库的前提下，直接加载本地 `mem0/utils/scoring.py` 执行合成输入；结果只验证评分函数，不代表完整向量库、embedding 或 LLM 的端到端行为。

- **语义门槛优先**：候选 semantic score 为 `0.05`、BM25 为 `1.0`、entity boost 为 `0.5`，threshold 为 `0.1` 时，结果仍为空。BM25 / entity 不能挽救门槛前被淘汰的候选。
- **同一候选池内可重排**：old semantic `0.80`、new semantic `0.60` 时，给 new BM25 `1.0` 后，new 排在 old 前面；给 new entity boost `0.5` 也可得到同样的排序反转。
- **候选池外不可挽回**：只把 boost 字典提供给候选池之外的 ID，不会新增结果；候选集由调用方传入的 semantic results 决定。
- **时间字段不参与评分**：仅交换两个候选 payload 的 `created_at`，输出排序和分数不变；这是函数级事实，不等价于完整 search 一定忽略所有时间相关逻辑。
- **绝对分数受信号配置影响**：当一次查询的 BM25 字典非空时，所有候选共享 `max_possible = 2.0`，即使某候选自身没有 BM25 命中也会被同一分母归一化。该规则可能改变绝对分数，但在同一查询内不会单独改变排序。

**当前可写成的窄结论**：本地 OSS 评分函数负责“进入 semantic 候选后的相关性重排”，不负责根据 `created_at` 判断新旧，也不负责从 semantic 候选池之外找回事实。新旧事实冲突是否在更上游通过抽取、实体链接或其他路径被处理，仍需端到端实验与源码继续核验。

### 1.3.4 仓库 README

`github.com/mem0ai/mem0` · `Status: Verified`

"New Memory Algorithm (April 2026)" 基准表：

| Benchmark | Old | New | Tokens | Latency p50 |
|---|---|---|---|---|
| LoCoMo | 71.4 | 92.5 | 7.0K | 0.88s |
| LongMemEval | 67.8 | 94.4 | 6.8K | 1.09s |
| BEAM (1M) | — | 64.1 | 6.7K | 1.00s |
| BEAM (10M) | — | 48.6 | 6.9K | 1.05s |

**关键免责声明（逐字）**："All benchmarks run on the same production-representative model stack. Single-pass retrieval (one call, no agentic loops) at a top_200 retrieval budget. **Scores reflect Mem0's managed platform, which includes proprietary optimizations not available in the open-source SDK; open-source users should expect directionally similar gains but not identical numbers.**"

另称评测框架已开源，可复现。

**标题与数值不一致（本轮发现）**：README 标题写 "April 2026"，但 92.5 / 94.4 两个数字来自 **5 月**更新后的算法（见 §1.5），不是 4 月版。

### 1.4 论文 arXiv:2504.19413

`arxiv.org/abs/2504.19413` · Submitted **28 Apr 2025** · ECAI 2025 · `Status: Verified`（限摘要，全文未读）

- 架构：dynamically **extracting, consolidating, and retrieving** salient information
- 增强变体：graph-based memory representations（Mem0g）
- LOCOMO 对比**六类基线**：(i) 既有记忆增强系统 (ii) 不同 chunk/k 的 RAG (iii) full-context (iv) 一个开源记忆方案 (v) 一个专有模型系统 (vi) 一个专用记忆管理平台
- 结果：LLM-as-a-Judge 相对 OpenAI **+26%**；Mem0g 比基础配置高约 **2%**
- **"91% lower p95 latency" 与 ">90% token 节省" 的比较对象是 full-context method**，不是同类记忆系统

### 1.5 官方博客：5 月加入时间推理与记忆衰减

`mem0.ai/blog/the-token-efficient-memory-algorithm-now-has-temporal-reasoning` · `Status: Verified`

| Benchmark | April 算法 | Updated 算法 | Δ |
|---|---|---|---|
| LoCoMo | 91.6% | 92.5% | +0.9 |
| LongMemEval | 93.4% | 94.4% | +1.0 |

- 5 月版新增两项能力：**Temporal Reasoning** 与 **Memory Decay**
- 写入时额外抽取时间元数据：事件何时发生、是否进行中或已完成、时间精度、记忆类型
- 分类目标增益：temporal reasoning **+3.8**、multi-session reasoning **+1.5**
- 4 月版自述为 "single-pass extraction and hierarchical retrieval"

**对主线的影响（`Inferred`）**：后续 Memory Decay 描述要求研究区分版本与部署范围；不能把早期 ADD-only 描述当成终态。它是否补偿冲突消解、在哪个阶段执行及是否适用于 OSS，仍需核验。

### 1.6 官方博客：State of AI Agent Memory 2026

`mem0.ai/blog/state-of-ai-agent-memory-2026` · `Status: Verified`

- 两项架构变更驱动结果：single-pass ADD-only extraction（**agent 生成的事实升为一等公民**，与用户陈述事实同权重）+ multi-signal retrieval（三路打分**并行**后融合）
- 评测框架开源地址：`github.com/mem0ai/memory-benchmarks`
- 自述论文是"first broad head-to-head comparison of ten memory approaches"

### 1.7 仓库 issue：ADD-only 的过期事实问题

`mem0ai/mem0#4956` · opened 2026-04-24 · label `bug` · **状态 Open** · `Status: Observed`

标题："ADD-only extraction in v3 may surface stale/contradictory facts for time-sensitive attributes"

报告者场景逐字要点：

- 升级 OSS v3 后 `add()` 不再发出 UPDATE / DELETE 事件
- 对可变状态类事实（当前雇主、当前城市、感情状态），矛盾记忆**随时间累积**而非新事实取代旧事实
- 场景：3 个月前 "I work at Company A" → 今天 "I now work at Company B" → **两条共存**
- 查询 "Where does the user work?" 时，检索（semantic + BM25 + entity matching）**可能把 Company A 排在前列，因为打分信号不含 recency**

这是「旧事实复活」的一手确认，且由官方仓库标记为 bug、尚未关闭。

### 1.8 仓库 issue：条目质量审计

`mem0ai/mem0#4573` · `Status: Observed`

标题："What we found after auditing 10,134 mem0 entries: 97.8% were junk"

- 分批审计表：6,264 条中 keeps 195，junk rate **96.9%**（抽取模型 gemma2:2b → 后期 Sonnet 时降至 89.6%）
- 报告者自我限定：**"This is a gemma-era problem, solved by upgrading the model"**
- 但同时指出架构缺口："it reveals the missing validation: **there's nothing between extraction and storage that checks whether a fact is grounded** in the actual conversation"
- 以及置信度问题："The pipeline stored each with **the same confidence as a real fact**"
- 另一例："808 copies of a hallucination"，报告者称"this one matters more, because it's architectural"

**引用纪律**：junk rate 高度依赖抽取模型，不可去掉 gemma2:2b 这个前提单独引用该数字。

### 1.9 仓库讨论与其他 issue

- `mem0ai/mem0` Discussion **#4289**「mem0 memory storage working too indiscriminately in OpenClaw」· `Status: Observed`
  参与者定位："the main problem is **at the write stage, not retrieval**. A retrieval budget could limit the damage, but it would not stop low-value memories from entering"；提出写入门控应默认保守（估计复用价值、低置信候选不进热集、保留时附理由或 TTL）。作者补充："there is no real distinction between information that has re-use value and information that can be forgotten immediately."
- `mem0ai/mem0#4926`「Type-Aware Memory Retrieval with Support for Deterministic (Persistent) Memory」· **Closed** · `Status: Observed`
  另一类问题：persona / policy 类记忆因 embedding 相似度不够而检索不到（"Policy may not be retrieved → incorrect responses possible"），诉求是脱离纯相似度的确定性检索。

## 2. 二手来源（仅用于定位争议存在，不作机制依据）

- `essays.bloo-mind.ai/posts/2026-05-20-mem-eval`（The Benchmark Theatre）：对基准之争的第三方梳理；提供 Penfield Labs 对 LOCOMO 本身的审计线索（1,540 题中 99 处错误；62.81% judge 接受含糊错答；审计仓库 `dial481/locomo-audit`，称 SHA256 可验）。**非 vendor 来源，值得单独拉一手。**
- `atlan.com/know/zep-vs-mem0`：给出 Zep 争议的一手记录位置 `getzep/zep-papers` issue **#5**。
- `developersdigest.tech`、`dataaspirant.com`、`theaiengineer.substack.com`、`blog.devgenius.io`：LOCOMO 规模与争议数字的交叉印证（各方数字不一致，见 `evidence.md`）。
- `linkedin.com/posts/charles-packer_...`：Letta CEO 本人发言，属当事方一手表态、但载体为社媒，按 `evidence-source-rules.md` 边界另行判定。

## 3. LOCOMO 基准本体（待拉一手）

- LOCOMO 论文：**arXiv:2402.17753**（未读）
- LongMemEval 论文：**arXiv:2410.10813**（未读）
- 二手描述的规模口径不一致：一处称"约 300 turns、up to 35 sessions"，另一处称"10 段长对话、各约 600 dialogues、约 26,000 tokens"。**规模口径本身需一手确认**，它决定 LOCOMO 能否暴露累积污染（见问题树 §5）。

## 4. 下一轮建档目标

1. **PyPI 发布对应关系**：补齐官方 `mem0ai` JSON / release 文件，确认 PyPI `2.2.1` 包与本地 `v2.2.1` commit 的对应关系（核心 tag / commit 已闭合）。
2. **arXiv:2504.19413 全文**：确认两阶段中 ADD / UPDATE / DELETE / NOOP 的原文表述（摘要只写 "extracting, consolidating"）。
3. **`getzep/zep-papers#5`** 与 Zep 原博客「Lies, Damn Lies, & Statistics」：Zep 争议双方一手。
4. **arXiv:2402.17753**：LOCOMO 任务形态与规模一手口径。
5. **`dial481/locomo-audit`**：非 vendor 的基准缺陷审计。
6. **Node 同步阻塞 30–40s**：本轮未核到任何一手 issue，需定向搜 `mem0-ts` / Node SDK 的 issue；若仍无，应降级或从缺陷清单移除。
