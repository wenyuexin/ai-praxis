# Mem0 — 一手来源建档与版本语义

> 本文件承接 `mem0` 研究的上游来源摘录、核验路径与版本边界（[`research-artifacts.md`](../../../../../../docs/contributing/rules/research-artifacts.md) §3.9）。
> Claim-Source 对照与状态判定见 [`evidence.md`](./evidence.md)；问题级递进见 [`deep-research-question-tree.md`](./deep-research-question-tree.md)。
> 来源只记可还原形式（arXiv 编号、官方文档 URL、`owner/repo` + issue 号），不写本机路径（[`traceability-rules.md`](../../../../../../docs/contributing/rules/traceability-rules.md) §3.1）。

## 0. 版本锚点（Version Basis / Observed At）

> **Observed At**：2026-10-01（初轮）；2026-10-02 Python 增量；2026-10-03 TypeScript / Dream 回写与复核见 §1.3.7–§1.3.8，未重查全部网页
> **Drift Risk**：`high`——2026 年 4 月与 5 月两次算法更新，OSS 有 v2→v3 破坏性变更

| 项 | 值 | 状态 |
|---|---|---|
| OSS 算法代次 | v2 → **v3**（官方迁移指南以此命名） | `Verified` |
| 本地 `main` 分支 `pyproject.toml` 的 Python 包版本 | `mem0ai 2.2.1` | `Observed` |
| 本地核心库 release tag / commit | `v2.2.1` → `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd` | `Verified` |
| 同 commit 的并行发布标签 | Python `v2.2.1` 与 TypeScript `ts-v3.3.1`；下文 `mem0/memory/main.py` 结论仅指 Python OSS | `Observed`（2026-10-02） |
| PyPI `mem0ai==2.2.1` 工件 | wheel / sdist 上传时间约在该 release commit 后两分钟；PyPI JSON 无 Git SHA，未下载工件 | `Inferred`（高置信，2026-10-02） |
| 仓库最新 release | `deepseek-plugin-v0.3.2`（2026-09-23） | `Verified` |
| 规模 | 65.5k–65.7k star / 7.7k fork（抓取期间数值漂移） | `Observed` |

**Stop-line 状态**：本地源码已固定到核心库 `v2.2.1` commit，且已完成普通 `add()` / `search()` 主路径及主要特殊入口的静态核验；仍不能把迁移文档的算法代次直接等同于所有部署形态，也未做运行时实验。此前 `v2.1.0` 是远程源码核验坐标，当前本地研究以 `v2.2.1` 为主；PyPI 版本、时间与 release subject 已核对，但因工件未下载且 JSON 无 Git SHA，仍不把构建对应关系写成 `Verified`。

### 0.1 算法代次与包版本暂不能直接等同

- 官方 OSS 迁移文档把 `v2 → v3` 作为**迁移 / 算法代次**命名，文档明确描述 ADD-only、hybrid retrieval、entity-aware retrieval 等变化。
- 此前远程 `main` 快照的 `pyproject.toml` 为 `2.1.0`；当前本地 `main` 已前进到 `2.2.1`。两者都只能说明源码快照的包版本字段，不能单独证明 v3 算法代次或某个 PyPI 文件的全部行为。
- 官方 GitHub tags / commit API 已将核心库 `v2.1.0` 对应到 commit `19f713408273fb1d657daa38d7b82ccf496d36d5`；该 commit 的 release bump 将 `pyproject.toml` 从 `2.0.20` 改为 `2.1.0`。这闭合了核心源码锚点，但不等于已经闭合 PyPI 文件与该 commit 的发布对应关系。
- 本地 Git 仓库当前为 `v2.2.1` / `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`，与 `v2.1.0` 的核心机制 diff 主要是：部分写入失败时只把实际持久化记录写入 history、实体链接和返回结果；另有上下文管理器与 procedural-memory 注释。未发现自动抽取或 hybrid scoring 机制改写。
- 初轮外部索引曾把 `2.2.1` 与 `3.1.0` 分别声称为最新；后续已核对 PyPI `2.2.1` JSON 的版本与上传时间，支持该指定版本的高置信关联，但不由此断言所有渠道的“最新版本”，也未完成工件级构建核验。
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
- 普通 Python OSS 路径未见 Temporal Reasoning / Memory Decay 执行逻辑；`expiration_date` 过滤不能等同于它们。`add(timestamp=...)` 与 `search(reference_date=...)` 在 OSS 中显式拒绝。特殊入口已在 §1.3.6 单独核验；TypeScript、后台与 Platform 服务端仍不外推。

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

2026-10-01 曾在不安装依赖、不修改 Mem0 仓库的前提下，直接加载本地 `mem0/utils/scoring.py` 执行合成输入；结果只验证评分函数，不代表完整向量库、embedding 或 LLM 的端到端行为。以下保留历史结果；2026-10-02 增量研究仅静态读取，没有重跑或新增实验。

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

### 1.3.5 写入去重、变化叙述与两种关联

- **Status**：源码观察为 `Observed`；机制解释为 `Inferred`。
- **Version Basis / Observed At**：Python SDK `v2.2.1`，commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`；2026-10-02。
- **Scope**：普通 OSS `Memory` / `AsyncMemory` 的 `infer=True` 写入；不外推 TypeScript、Platform 服务端、procedural 或后台任务。
- **Sources**：以下源码路径均相对于固定上游 [commit tree](https://github.com/mem0ai/mem0/tree/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd)。`mem0/configs/prompts.py:468–701,918–943,1016–1062`；`mem0/memory/main.py:918–1220,2600–2901`；Platform 对照为 `mem0/client/types.py:61,85` 与 `mem0/client/main.py:416–460,504–532`。
- **Trace**：沿问题树 §1 / §2 的“去重是否消解矛盾”补读 prompt、解析、payload 与实体索引；结论留在对象 notes，回写 `evidence.md` M2a–M2d，不升级为全系统保证。
- **Needs**：实际 LLM 输出和检索结果未验证；其他写入入口、后台任务与 Platform 服务端仍待独立核验。

**事实（`Observed`）**

1. **写入并非只有 hash 去重。** additive prompt 要求：新信息若与召回旧记忆语义等价且无新语境，则不再抽取；同一响应不应重复。它也要求描述偏好变化时保留“从什么变成什么”的过渡语境，并把 contradiction 列为 `linked_memory_ids` 的关联条件。这是写入期的 LLM 指令，不是已测得的去重效果。
2. **确定性去重范围很窄。** 代码计算 `hashlib.md5(text.encode()).hexdigest()`，与最多 10 条本次召回记录中已有的 `hash` 和本 batch 的 `seen_hashes` 比较。它不是全库扫描，也不按语义、属性或日期判定等价；没有 `hash` 的旧 payload 不贡献该比较集合。hash 在 lemmatization 前计算，不能称为归一化文本去重。
3. **变化叙述不等于旧记录失效。** 普通同步/异步路径为保留的抽取文本分配新 UUID、insert，并记录 ADD；没有按 LLM 输出的 event 执行 UPDATE / DELETE。解析读取 JSON 的 `memory` 数组，随后取 `text` 和可选 `attributed_to`。
4. **两种 `linked_memory_ids` 不能混用。** prompt 的字段表达“新记忆关联旧记忆”；但上述 payload 构造没有消费 LLM 输出的这个字段。召回旧 ID 被映射为 `"0"`、`"1"` 等临时编号，`uuid_mapping` 建好后未在该路径使用。相反，entity store 确实保存同名字段，内容是“该实体关联的记忆 ID 集合”：对现有集合取并集，供检索 entity boost 使用。该代码没有标记哪个旧属性被新属性替代。调用方 metadata 被复制进 payload，因此这里不是断言任何 payload 都不可能含同名字段，而是说 **LLM 生成的关联未被该路径接入**。
5. **Platform 另有公开契约。** 固定 commit 的官方 Dream 文档称：新增矛盾事实会把旧记忆标为 `superseded` 并链接到替代记忆；默认读取仍可返回 superseded，`latest_only=true` 只返回 active 记忆。OpenAPI 还声明 `delete_linked=true` 删除 linked memories 并可返回 `cascade_count`；SDK changelog / client 将其说明为沿 v3 `linked_memory_ids` 的旧 superseded 链递归删除。这里的产品/API 契约可记为 `Verified`，客户端转发为 `Observed`；但 Platform 服务端的建链字段、判断规则、精确遍历和执行实现仍为 `Unverified`。公开 `replaced_by` 与 SDK / changelog 使用的 `linked_memory_ids` 是否一一对应，也不能由已读材料确认。

**解释与边界（`Inferred`）**

更准确的分层是“prompt 级语义筛选与变化叙述 → 候选局部 hash 去重 → 新记录追加 → 实体索引关联”。这组证据不支持“写入完全不处理一致性”，也不支持“实体链接已经消解矛盾”。在已读普通 OSS 路径中，没有看到基于新旧事实裁决并使旧记录失效的实现；但 LLM 是否会把变化写得足够清楚、下游能否据此回答当前状态，必须与静态控制流分开判断。M16 所涉及的跨层因果关系仍不成立为事实。

### 1.3.6 Python OSS 写入入口边界

- **Status**：路径观察为 `Observed`；“不能将普通 `infer=True` 结论扩大到整个 OSS”是 `Inferred`。
- **Version Basis / Observed At**：Python SDK `v2.2.1`，commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`；2026-10-02。
- **Scope**：`Memory` / `AsyncMemory` 的已读 `add()` 分支；不外推 Platform、TypeScript 或后台服务。
- `infer=False` 在 `_add_to_vector_store` 开头逐条处理非 system 消息，直接 embed → `_create_memory` → insert + `ADD` history；不执行普通分支的 top-10 召回、LLM additive extraction、局部 hash 检查或 entity linking。`_create_memory` 会计算并写入 hash，但写入 hash 不等于执行重复检查。
- `agent_id + memory_type="procedural_memory"` 进入独立 procedural 路径：LLM 生成一条 procedural 总结后直接写入，带 `memory_type` metadata；不使用普通 additive prompt、top-10 既有记忆比较、局部 hash 去重或 Phase 7 entity linking。
- vision 发生在普通写入前的消息预处理；之后仍由 `infer=True/False` 决定进入哪条写入路径，不是第三种持久化语义。

因此，普通同步/异步 `infer=True` 的 ADD-only、候选局部 hash 去重和实体索引三者应继续作为窄范围 `Observed`；不能写成“整个 Python OSS 都有同一套去重或实体关联”。`infer=False` 与 procedural 仍是 ADD 结果，但其 ADD 语义是直接追加，不应与普通自动抽取流水线混称。

### 1.3.7 TypeScript OSS 普通路径对照

- **Status / Version Basis / Observed At**：`Observed`；同一固定 commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`，标签 `ts-v3.3.1`，`mem0-ts/package.json` 为 `3.3.1`；2026-10-03 回写。
- **Sources**：固定上游 commit 的 `mem0-ts/src/oss/src/memory/index.ts`（`add`、`addToVectorStore`、`search`）、`mem0-ts/src/oss/src/prompts/index.ts` 和 `mem0-ts/src/oss/src/utils/scoring.ts`。
- **Trace / Scope**：在 Python 普通路径闭合后做跨 SDK 对照，只回写已读机制，不把局部同构解释为所有入口完全等价。
- 普通 `infer=true` 路径先召回 topK=10，再单次 LLM 抽取 `memory[]`；原始文本 MD5 与召回记录已有 hash 和 batch `seenHashes` 比较；保留项使用新 UUID insert，history 为 `ADD`，随后维护实体索引。
- prompt / 解析类型中的 `linked_memory_ids` 没有进入普通 memory payload；entity store 的 `linkedMemoryIds` 保存实体到 memory ID 集合的索引，用于 entity boost，不是已经确认的事实 supersession 链。
- search 先形成 semantic 候选池，BM25 / entity 对池内候选融合打分；threshold 在融合前门控 semantic score；固定评分函数不读取 `createdAt` / `updatedAt`。这不是所有后端效果或端到端矛盾处理的验证。
- `infer=false` 是直接追加，vision 是写入前预处理；没有观察到 Python 的 `agent_id + procedural_memory` 分支。显式 update / delete 仍是独立 API。
- **Needs**：未覆盖入口、依赖退化、实际抽取质量与运行结果仍未验证；不由此推断 Platform 服务端。

### 1.3.8 Platform Dream 的公开契约与实现边界

- **Status / Version Basis / Observed At**：产品/API 文档契约为 `Verified`，SDK 参数转发为 `Observed`，服务端内部实现为 `Unverified`；固定上游 commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`；2026-10-03 回读。该 commit 固定文档快照，不固定线上服务部署版本。
- **Sources**：`docs/platform/features/dream.mdx`（The three actions、Supersede、Merge、How reads change、How often Dream runs）；`docs/openapi.json`（`replaced_by`、`delete_linked`、`cascade_count`）；`docs/changelog/sdk.mdx` 与 Python / TypeScript Platform client。
- **Trace**：用于修正 M16 的“一致性责任向读取期转移”概括；保留在对象 notes，公开契约不升级为实测保证。
- **Supersede / Merge 的时机**：文档明确是 memory-addition pipeline 的一部分，随 add 处理评估，没有另一个定时周期。虽然页面概述称 Dream 为 background layer，不能据此把这两项读成定时后台任务。
- **Supersede**：旧矛盾记忆标为 `superseded` 并链接新事实，保留而不删除；默认读取包含 active + superseded。
- **Merge**：多数 exact / near-duplicate 在新增时不生成第二条；已有事实被更完整版本合并时可保留 `merged` 记录并链接 canonical memory。默认读取隐藏 merged，`include_merged=true` 可纳入。
- **读取模式**：文档表中 `latest_only=true` 只保留 active，排除 superseded 和 merged；`include_merged=true` 纳入三类。该表不证明每个 SDK 的单条 `get()` 均支持这些参数，也不说明两个 flag 同时使用时的优先级。
- **Synthesis**：按用户定时生成新的 pattern memories，源记忆保留；是与 add-time supersede / merge 不同的调度机制。
- **删除契约**：OpenAPI 声明 `delete_linked=true` 删除关联记忆、响应可有 `cascade_count`；SDK changelog 将其描述为沿 v3 `linked_memory_ids` 的旧 superseded 链传递删除。Dream 自身保留记录，不等于显式 delete API 也非破坏性。
- **Needs**：`replaced_by` 与 `linked_memory_ids` 的映射、supersede / merge 判断规则、递归/环处理、原子性与失败语义仍未知；客户端和 mock 测试只确认请求形状，未验证真实服务端状态。

### 1.4 论文 arXiv:2504.19413

`arxiv.org/abs/2504.19413` · Submitted **28 Apr 2025** · ECAI 2025 · `Status: Verified`（v1 HTML 全文核验：§2.1、§2.2、Figure 2/3、Appendix B）

- 架构：dynamically **extracting, consolidating, and retrieving** salient information
- §2.1 / Figure 2 将基础 Mem0 的正式管线命名为两个阶段：**extraction + update**；摘要中的 `consolidating` 是架构概述，不是论文另列的 `merge` 阶段名。
- update 阶段先检索 top-*s* 语义相似记忆，再由 LLM 通过 function-calling / tool call 在 **ADD / UPDATE / DELETE / NOOP** 间选择。论文 Appendix B Algorithm 1 以 `SemanticallySimilar`、`Contradicts`、`Augments` 的伪代码交叉描述这四路决策。
- 基础 Mem0 中，`DELETE` 用于移除被新信息矛盾的记忆；这只能作为论文旧版设计的 `Verified` 事实，不能外推当前 OSS `v2.2.1` 或 Platform 实际服务端。
- 增强变体：graph-based memory representations（Mem0g）
- Mem0g 的图 update 阶段也做冲突检测与解析；论文描述将过时关系标记为 `invalid`，而非物理删除，以保留时间推理能力。该图论文机制与当前 OSS 实体索引不是同一实现。
- LOCOMO 对比**六类基线**：(i) 既有记忆增强系统 (ii) 不同 chunk/k 的 RAG (iii) full-context (iv) 一个开源记忆方案 (v) 一个专有模型系统 (vi) 一个专用记忆管理平台
- 结果：LLM-as-a-Judge 相对 OpenAI **+26%**；Mem0g 比基础配置高约 **2%**
- **"91% lower p95 latency" 与 ">90% token 节省" 的比较对象是 full-context method**，不是同类记忆系统

### 1.5 官方博客：5 月加入时间推理与记忆衰减

`mem0.ai/blog/the-token-efficient-memory-algorithm-now-has-temporal-reasoning` · `Status: Verified`

| Benchmark | April 算法 | Updated 算法 | Δ |
|---|---|---|---|
| LoCoMo | 91.6% | 92.5% | +0.9 |
| LongMemEval | 93.4% | 94.4% | +1.0 |

- 5 月版新增两项能力：**Temporal Reasoning** 与 **Memory Decay**；官方文档和博客将二者限定为 Platform-only，不能外推到当前 OSS commit。
- 写入时额外抽取时间元数据：事件何时发生、是否进行中或已完成、时间精度、记忆类型
- 分类目标增益：temporal reasoning **+3.8**、multi-session reasoning **+1.5**
- 4 月版自述为 "single-pass extraction and hierarchical retrieval"

**对主线的影响（`Inferred`）**：后续 Memory Decay 要求研究区分版本与部署范围；不能把早期 ADD-only 描述当成终态，也不能把 Platform-only 时间能力解释为当前 OSS 已具备的冲突消解。它是否补偿冲突消解、在哪个阶段执行及其服务端边界，仍需核验。

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

这是报告者关于「旧事实复活」的一手报告，不是本研究的复现或维护者确认。上述 label / Open 是 2026-10-01 抓取状态；2026-10-02 未重新查询，也不能由 issue 状态推断固定 `v2.2.1` 或 Platform 是否已修复。

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

## 2. 补充来源与争议定位

- `essays.bloo-mind.ai/posts/2026-05-20-mem-eval`（The Benchmark Theatre）：对基准之争的第三方梳理；提供 Penfield Labs 对 LOCOMO 本身的审计线索（审计者将 1,540 个非 adversarial 问题中的 156 项标为问题，其中 99 项 score-corrupting、57 项 citation-only；另报告 62.81% judge 接受含糊错答）；审计仓库 `dial481/locomo-audit`，称 SHA256 可验。**非 vendor 来源；审计仓库已在 §3.5 固定 commit 并单独分层。**
- `atlan.com/know/zep-vs-mem0`：仅作争议入口导航；一手材料已在 §3.4 记录。
- `developersdigest.tech`、`dataaspirant.com`、`theaiengineer.substack.com`、`blog.devgenius.io`：LOCOMO 规模与争议数字的交叉印证（各方数字不一致，见 `evidence.md`）。
- `linkedin.com/posts/charles-packer_...`：Letta CEO 本人发言，作为当事方一手表态；核心争议与评测设置以 Letta 原博客 §3.4 为主。

## 3. 基准本体与评测边界

### 3.1 LOCOMO：长历史问答，不是完整的事实版本管理评测

`arXiv:2402.17753v1` · **Status: Verified**（论文 HTML 全文核验；2026-10-03）

- 数据集包含 **50** 段长对话；平均每段约 **19.3 sessions、304.9 turns、9,209.2 tokens**，最多 35 sessions，覆盖数月。
- QA benchmark 共 **7,512** 题，五类为 single-hop、multi-hop、temporal reasoning、open-domain knowledge、adversarial。
- 对话由 persona + temporal event graph 驱动生成，并经过人工修正长程不一致、图片和事件图对齐问题。
- 它明确测试跨 session 回忆、多跳组合、时间线索和不可回答性；QA 形态是对已完成历史的离线问答，不是让记忆系统在持续会话中逐轮写入、更新再读取的在线 protocol。
- 因此，LOCOMO 分数可以支持“在该配置下从长、多 session 历史中检索和回答这些题型”，不能直接证明事实覆盖、旧值复活、矛盾版本共存或长期写入污染已经被可靠处理。
- `adversarial` 题的目标是识别无法由上下文支持的问题，不等于测试互相矛盾事实的保留、选择或消解。
- 论文明确承认其数据主要由 hybrid human-machine pipeline 生成；合成数据与离线 QA 是解释分数时的边界。

### 3.2 LongMemEval：显式覆盖 knowledge update，但不等于冲突共存基准

`arXiv:2410.10813v2` / ICLR 2025 · **Status: Verified**（论文 HTML/PDF 章节核验；2026-10-03）

- 共 **500** 个问题，细分为 single-session-user、single-session-assistant、single-session-preference、multi-session、knowledge-update、temporal-reasoning、abstention 七类，覆盖五种长期记忆能力。
- 多数问题需要多个证据 session，最多 6 个；LongMemEvalS 约 50 sessions、115k tokens，另有约 1.5M tokens 的可扩展设置。
- `knowledge-update` 明确测试识别用户个人信息/生活状态的变化，并随时间更新用户知识；`temporal-reasoning` 同时使用 session timestamp 和对话中的时间表达。
- 历史由证据 session 与大量无关 session 拼装；构造时明确避免无关 session 引入会使问题失效的冲突信息。
- 因此它比 LOCOMO 更直接支持“更新到后来的用户状态”这一评测判断，但不能单独证明系统能处理旧事实复活、长期并列的矛盾版本或开放世界污染。
- 论文的长期性主要来自 session 数量、检索干扰和上下文规模；不能把 token 长度直接等同于真实用户交互持续了多少月/年。

### 3.3 Mem0 公开分数的 Platform / OSS 口径

- 当前 README 的 **92.5 LoCoMo / 94.4 LongMemEval** 明确属于 Mem0 managed Platform，带有 OSS SDK 不具备的 proprietary optimizations；不能当作固定 Python/TypeScript OSS 的同分结果。
- 开源评测框架同时支持 `oss` 与 `cloud` backend，默认 `oss`，但“框架支持两种后端”不等于“Platform 分数已由默认 OSS 配置复现”。
- 公开结果至少可确认 Platform 的 retrieval budget 为 `top_200`、single-pass、无 agentic loops；但 Platform 具体服务端版本、抽取/answerer/judge 模型、embedding、数据版本、随机性和完整 run artifacts 仍不完整。
- 框架明确提示 embedding、抽取模型、judge、top-k 等都会影响分数；不能把 Platform 与 OSS 的公开数字差异单独归因于存储或检索机制。
- 固定框架公开 OSS 结果没有提供与 Platform **92.5 LoCoMo** 完整对等的 OSS 成绩表；当前不能计算 Platform 与 OSS 的 LoCoMo 差值。

### 3.3.1 benchmark provenance：固定 artifact 与当前文档数字不是同一证据层

`evaluation/` 是独立 `mem0ai/memory-benchmarks` 子模块，固定 commit 为 `4b61c5d31b9c668a12b4f5e78064248a02c82d2b`；它不等同于外层 SDK `v2.2.1` / `ts-v3.3.1` 的发布版本。以下均为静态读取，未运行 benchmark。

- **固定子模块的 Platform artifacts**：`results/platform/locomo_results.json` 记录 `1410/1540 = 91.5584…`，即文档中的 91.6；`results/platform/longmemeval_results.json` 记录 `467/500 = 93.4`。LoCoMo artifact 显式记录 `answerer_model=gpt-5`、`judge_model=gpt-5`、`provider=azure`、`top_200`；LongMemEval artifact 记录 `model=gpt-5`、`generation_model=gpt-5`、`top_200`，answerer/judge 的具体映射依赖当前 adapter，不能把 artifact 字段全部当作原生语义。
- **当前文档的 Platform 声明**：`README` / `docs/core-concepts/memory-evaluation.mdx` 另行声明 LoCoMo 92.5（1425/1540）和 LongMemEval 94.4（472/500），同样使用 top_200、single-pass、无 agentic loops，并含 proprietary optimizations。当前固定子模块中没有能把 92.5/94.4 绑定到服务端 revision、数据 hash、完整模型 stack、随机性或逐题 output 的 run manifest。
- **两组数字的关系**：91.6/93.4 与 92.5/94.4 不是简单四舍五入差异。较稳妥的状态是：前者为固定 commit 中可复核的旧一组 Platform artifacts（`Verified`）；后者为当前官方文档声明的 Platform 结果（`Observed`）；后者具体 run provenance 为 `Unverified`；“来自后续 Platform run 或服务端版本”只能作 `Inferred`，不能绑定到某个 commit。
- **固定快照的 OSS 结果**：`results/oss/` 只提供 LongMemEval 结果，公开标签为不同 memory extraction model 的 88.6–91.0；README 称这些 run 共用 Qwen 600M/SageMaker embedder、Qdrant、GPT-5 answerer/judge，部分 artifact 记录 `top_k=200`、`seed=42`。当前没有 OSS LoCoMo artifact，不能用这些结果计算 Platform/OSS 的 LoCoMo 差值。默认 OSS Docker 配置也不等于该对照实验配置。
- **可比性缺口**：评测 runner 使用浮动的 `main` 数据 URL，artifact 未统一保存数据 commit/hash；Platform extraction、embedding、向量/图存储、服务端 revision、重试/温度等也未完整写入 manifest。因此“固定子模块可运行”不等于“92.5/94.4 已被复现”。

### 3.4 评测争议的一手材料状态

- **Zep ↔ Mem0**：Zep 原博客已明确将其 LoCoMo 结果修正为 **75.14% ± 0.17**，并提出 Mem0 评测中的 user model、timestamp handling、sequential search 等方法问题；Mem0 在 `getzep/zep-papers#5` 主张其复核为 **58.44% ± 0.20**，相对 Mem0 论文中报告的 65.99%。这些是双方一手主张，不应合并成单一事实；实现是否忠实、题类/分母与时间字段如何处理仍需逐项核验。
- **Letta ↔ Mem0**：Letta 原博客主张 Mem0 论文中的 MemGPT/Letta baseline 难以原生复现，且其 Filesystem 方案在 GPT-4o mini、受限工具规则下取得 74.0% LoCoMo；这证明争议存在，不等于已完成同配置复现或证明某方案普遍优越。

### 3.5 dial481/locomo-audit：第三方协议审计

公开仓库 `dial481/locomo-audit` 当前应固定到 commit `9493fb4b4af4256ed17a18e8fd0b3cfdeec29539`（2026-04-02，未发现 release/tag）。仓库自述包含：

- 对 `snap-research/locomo/data/locomo10.json` 的 SHA256 标识；
- 审计者将 1,540 个非 adversarial 问题中的 156 项标为问题，其中 99 项归类为 score-corrupting、57 项为 citation-only；
- 按每题多次 judge 结果进行多数投票重算的脚本与报告；
- 对 Mem0、Zep、EverMemOS 等实现的模型、judge、Category 5 处理和分母差异表；
- Wilson 置信区间等统计有效性分析。

证据分层：

- `Observed`：该审计仓库、固定 commit、源码/报告、数据 hash 和第三方复现入口确实存在；
- `Unverified`：上述 99 项分类是否都构成 score-corrupting error、judge 宽松比例以及“公开高分因此无效”等实质结论，本轮未逐题抽查、未重算；
- `Inferred`：若不同实现使用不同模型、prompt、judge、题类筛选和分母，跨报告分数存在显著可比性风险。

因此它适合作为 benchmark protocol conflict 的研究入口，不足以单独推翻任何一方成绩，也不足以触发跨对象 `memory/conflict.md`。

### 3.6 Node / TS “同步阻塞 30–40s”说法

本轮定向搜索未找到支持该具体时长或“同步阻塞”语义的一手 issue、release note、性能测量或官方文档。固定 TypeScript client 的 `add()` 是 `async` HTTP 请求；v3 integration test 以 `PENDING`/eventId 描述服务端异步处理，并通过轮询等待索引可见性。

这些源码只能支持“该版本的请求与后处理是异步接口/流程”，不能证明所有历史版本、OSS 自托管或外部 provider 都没有延迟问题。因此该说法继续保持 `Unverified`，并应从已确认缺陷清单中移除。

## 4. 下一轮建档目标

1. **PyPI 发布对应关系**：版本与 release 时间已高度吻合；PyPI JSON 无 Git SHA，未下载工件，故“由该 commit 构建”仍保持高置信 `Inferred`。
2. **评测配置可比性**：已恢复 Mem0 benchmark 框架的具体 commit、主要 Platform/OSS 结果分区和可见模型/embedding/judge/top-k 边界；后续只补缺失的 run manifest、数据版本与服务端 provenance，仍只读静态核验，不运行 benchmark。
3. **`dial481/locomo-audit`**：已固定第三方审计 commit；下一轮只做逐题抽查或复算价值评估，不把审计主张直接写成事实。
4. **Node 同步阻塞 30–40s**：未找到一手支持，已降为待溯源线索；若无新证据，从缺陷清单移除。
5. **Platform 版本链**：公开 Dream / supersede、`latest_only` 与 `delete_linked` 契约已核验；服务端建链字段、latest 判定、递归范围以及 `replaced_by` / `linked_memory_ids` 映射仍未核验。
