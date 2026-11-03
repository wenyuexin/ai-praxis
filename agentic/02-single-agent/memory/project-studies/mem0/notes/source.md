# Mem0 — 一手来源建档与版本语义

> 本文件承接 `mem0` 研究的上游来源摘录、核验路径与版本边界（[`research-artifacts.md`](../../../../../../docs/contributing/rules/research-artifacts.md) §3.9）。
> Claim-Source 对照与状态判定见 [`evidence.md`](./evidence.md)；问题级递进见 [`deep-research-question-tree.md`](./deep-research-question-tree.md)。
> 来源只记可还原形式（arXiv 编号、官方文档 URL、`owner/repo` + issue 号），不写本机路径（[`traceability-rules.md`](../../../../../../docs/contributing/rules/traceability-rules.md) §3.1）。

## 0. 版本锚点（Version Basis / Observed At）

> **Observed At**：2026-10-01（初轮）；2026-10-02 Python 增量；2026-10-03 TypeScript / Dream 回写与复核见 §1.3.7–§1.3.8；2026-10-10 写入入口与后端覆盖补记见 §1.3.1.2、§1.3.6；2026-10-30 Wave 1 复核版本谱系与过期过滤，未重查全部网页
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
- 2026-10-30 复查：远端 `main` 为 `26cdefb4`（2026-10-26），领先固定 `94c3fe9f` 9 个提交；`mem0/memory/main.py`、`configs/prompts.py`、`utils/scoring.py` 未变，仅 `vertex_ai_vector_search.py` 与文档 / 安全检查变动。机制结论继续按 `v2.2.1` 锚点。
- 初轮外部索引曾把 `2.2.1` 与 `3.1.0` 分别声称为最新；后续已核对 PyPI `2.2.1` JSON 的版本与上传时间，支持该指定版本的高置信关联，但不由此断言所有渠道的“最新版本”，也未完成工件级构建核验。
- 因此当前采用分层锚点：机制先锚到官方迁移文档的 `v2 → v3` 代次；当前核心源码行为锚到本地 `v2.2.1` commit；包安装行为仍需补可还原的 PyPI release。`v2.1.0` 仅作为前一轮远程源码对照点保留。

### 0.2 版本谱系：ADD-only 的分叉点（2026-10-30 核对）

- 旧算法：`v1.0.11`（2026-04-06，`144627c4`）的 `Memory._add_to_vector_store` 仍包含 `UPDATE` / `DELETE` / `NOOP` 分支，并调用 `_update_memory` / `_delete_memory`。
- 分叉提交：`a488e190`（2026-04-14，PR #4805 "port v3 pipeline with hybrid search, entity extraction, and additive scoring"）移除自动抽取中的这些分支；其父提交 `57f944e1` 仍保留旧分支。
- 首个包含该提交的 tag：`v2.0.0b1` / `ts-v3.0.0-beta.1`（2026-04-14）；首个稳定版 `v2.0.0` / `ts-v3.0.0`（2026-04-16）。
- 当前研究锚点 `v2.2.1` / `94c3fe9f`（2026-09-25）沿用 ADD-only；显式 `update()` / `delete()` API 仍在，但不在普通自动抽取路径内。
- 包版本 2.x 与算法代次 v3 不是同一套命名：迁移文档的 `v2 → v3` 指算法代次，不能读成 Python 包从 2.2.1 升到 3.x。

**Status**：`Observed`（本地 git 历史、tag containment 与源码分支核对）；官方迁移文档对机制变化的描述为 `Verified`。

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

### 1.3.1.1 Python 后端覆盖面

固定 commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd` 的 Python `VectorStoreBase.keyword_search()` 默认返回 `None`。`Memory` 初始化时如果发现后端没有覆盖该方法，会记录 warning，并说明 hybrid scoring 关闭、搜索退回 semantic similarity。搜索阶段若 `keyword_search()` 返回 `None`，就不会填充 `bm25_scores`；候选集仍只来自 semantic results。

已覆盖该方法的后端包括 Qdrant、Milvus、Pinecone、Weaviate、Baidu、Redis、Elasticsearch、OpenSearch、MongoDB、pgvector、Azure MySQL、Azure AI Search、Databricks、Upstash Vector 和 Vertex AI Vector Search。未覆盖并因此走基类 `None` 的已观察后端包括 Chroma、FAISS、Cassandra、Supabase、LangChain、S3 Vectors、Turbopuffer、OracleDB、Neptune 和 Valkey。

覆盖方法也不等于统一 BM25：

- Weaviate 调用原生 `query.bm25`；
- Qdrant 依赖 `bm25` sparse slot 和 fastembed；
- Pinecone 只有 `hybrid_search=True` 且 sparse encoder 可用时才执行关键词检索，否则返回 `None`；
- pgvector 使用 PostgreSQL `to_tsvector` / `ts_rank_cd` 全文检索，不是 BM25；
- Vertex AI Vector Search 覆盖了方法，但固定源码直接返回 `None`，注释说明 sparse hybrid 尚未实现。

因此，hybrid retrieval 是后端和配置条件，不是固定 Python SDK 的统一能力。调用方还会把不同后端返回的正分送进同一套 BM25 归一化，但本轮没有验证这些分数量纲是否等价。TypeScript 侧见 §1.3.1.2。

### 1.3.1.2 TypeScript 后端覆盖面

同一固定 commit 的 TypeScript OSS 中，`VectorStore.keywordSearch` 是可选方法，`Memory.search()` 在调用前用 `typeof` 检查：方法缺失、返回 `null` 或抛错时都归并为 `null`，`bm25Scores` 保持为空，`scoreAndRank` 的 `maxPossible` 相应缩小，最终退化为纯 semantic（+entity）排序，且不报错、不提示。

- 有真实实现的后端中，真正 BM25 的是 Qdrant、Milvus、Baidu、Weaviate、OpenSearch 和 Azure AI Search；pgvector、Azure MySQL、MongoDB、Databricks 是普通全文检索；`memory.ts` 是进程内 BM25（k1=1.5、b=0.75）；Upstash Vector 走服务端 keyword 检索。
- 其余后端返回 `null` 或未实现，包括 Pinecone、Vertex AI、Chroma、Redis、Valkey、Cassandra、Supabase、LangChain、S3 Vectors、Turbopuffer、Neptune、Vectorize；Elasticsearch 与 OracleDB 没有 `keywordSearch` 方法，由调用方的 `typeof` 检查跳过。
- `utils/bm25.ts` 有一个独立 `BM25` 类，但在 `mem0-ts/src` 内无引用，是未使用的死代码。

因此 TypeScript 的 hybrid 同样是后端条件能力，且与 Python 一样静默降级；哪些后端真正跑 BM25 必须逐个核对，不能按 SDK 版本号假定。

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

**标题与数值存在版本标注滞后（本轮发现）**：README 标题写 "April 2026"，早期提交中的对应数字为 91.6 / 93.4；后续文档更新将当前结果写为 92.5 / 94.4，并与 v3 + temporal reasoning 的算法代际关联（见 §1.5、§3.3.1）。这能说明文档演进线索，不能把标题或同步时间直接当作新版评测运行时间。

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
- **写入入口全量盘点（2026-10-10 补充）**：`Memory.add` / `AsyncMemory.add` 及其 `_add_to_vector_store`、`_create_memory`、`_create_procedural_memory` 对 **memory 记录**只做 INSERT + ADD history，从不调用 `_update_memory` / `_delete_memory`；`update()` / `delete()` / `delete_all()` / `reset()` 是独立的显式 CRUD。三点补充：
  - `add()` 并非完全不 UPDATE：Phase 7 会对已存在实体执行 `entity_store.update(...)` 扩展 `linked_memory_ids`。因此“add 只 INSERT”应限定为“对 memory 记录”。
  - 存在一个后台/延迟写入路径：`mem0/proxy/main.py` 的 OpenAI 兼容代理在 `Completions.create()` 中把 `mem0_client.add(...)` 放进 daemon 线程 fire-and-forget（无 join、无重试）。写入仍是 ADD-only，但时序上延迟。
  - SQLite `messages` 表在 `save_messages` 时会删除该 session_scope 下超过最近 10 条的历史消息；`infer=False` 不做去重，重复消息会重复 INSERT。
- `Memory.add` 的 docstring 仍写着 infer 会让 LLM 决定 add/update/delete，与实现不符，属陈旧注释；`__init__` 创建的 `_telemetry_vector_store`（collection `mem0migrations`）全包无引用，疑似废弃残留（`Inferred`）。

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

> Δ 为官方博客自报值；91.6/93.4 artifacts 可复核，92.5/94.4 的 run provenance 仍未闭合（M19）。

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

`mem0ai/mem0#4956` · opened 2026-04-24 · label `bug` · **状态 Open**（2026-10-30 复查仍 Open，23 条评论）· `Status: Observed`

标题："ADD-only extraction in v3 may surface stale/contradictory facts for time-sensitive attributes"

报告者场景逐字要点：

- 升级 OSS v3 后 `add()` 不再发出 UPDATE / DELETE 事件
- 对可变状态类事实（当前雇主、当前城市、感情状态），矛盾记忆**随时间累积**而非新事实取代旧事实
- 场景：3 个月前 "I work at Company A" → 今天 "I now work at Company B" → **两条共存**
- 查询 "Where does the user work?" 时，检索（semantic + BM25 + entity matching）**可能把 Company A 排在前列，因为打分信号不含 recency**

这是报告者关于「旧事实复活」的一手报告，不是本研究的复现或维护者确认。上述 label / Open 是 2026-10-30 复查状态（仍为 open、label `bug`、23 条评论）；不能由 issue 状态推断固定 `v2.2.1` 或 Platform 是否已修复。

### 1.8 仓库 issue：条目质量审计

`mem0ai/mem0#4573` · opened 2026-03-27 · **2026-06-05 以 completed 关闭** · `Status: Observed`

2026-10-30 复查：关闭说明称主要架构缺口已由已发布特性覆盖；固定 `v2.2.1` 的 `_add_to_vector_store` 仍未观察到显式 grounding / confidence 校验。关闭是否覆盖该具体缺口保持 `Unverified`，见 `evidence.md` M10。

标题："What we found after auditing 10,134 mem0 entries: 97.8% were junk"

- 分批审计表：6,264 条中 keeps 195，junk rate **96.9%**（抽取模型 gemma2:2b → 后期 Sonnet 时降至 89.6%）
- 报告者自我限定：**"This is a gemma-era problem, solved by upgrading the model"**
- 但同时指出架构缺口："it reveals the missing validation: **there's nothing between extraction and storage that checks whether a fact is grounded** in the actual conversation"
- 以及置信度问题："The pipeline stored each with **the same confidence as a real fact**"
- 另一例："808 copies of a hallucination"，报告者称"this one matters more, because it's architectural"

**引用纪律**：junk rate 高度依赖抽取模型，不可去掉 gemma2:2b 这个前提单独引用该数字。

### 1.9 仓库讨论与其他 issue

- `mem0ai/mem0` Discussion **#4289**「mem0 memory storage working too indiscriminately in OpenClaw」· `Status: Observed`（2026-10-30 复查仍可访问，未见维护者结论）
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

`evaluation/` 是独立 `mem0ai/memory-benchmarks` 子模块，固定 commit 为 `4b61c5d31b9c668a12b4f5e78064248a02c82d2b`；其中 Platform 结果文件由祖先提交 `edcd6f1` 首次加入，`4b61c5d` 后续只更新 README 数字。它不等同于外层 SDK `v2.2.1` / `ts-v3.3.1` 的发布版本。以下均为静态读取，未运行 benchmark。

- **固定子模块的 Platform artifacts**：`results/platform/locomo_results.json` 记录 `1410/1540 = 91.5584…`，即文档中的 91.6；`results/platform/longmemeval_results.json` 记录 `467/500 = 93.4`。LoCoMo artifact 显式记录 `answerer_model=gpt-5`、`judge_model=gpt-5`、`provider=azure`、`top_200`；LongMemEval artifact 记录 `model=gpt-5`、`generation_model=gpt-5`、`top_200`，answerer/judge 的具体映射依赖当前 adapter，不能把 artifact 字段全部当作原生语义。
- **当前文档的 Platform 声明**：`README` / `docs/core-concepts/memory-evaluation.mdx` 另行声明 LoCoMo 92.5（1425/1540）和 LongMemEval 94.4（472/500），同样使用 top_200、single-pass、无 agentic loops，并含 proprietary optimizations。当前固定子模块中没有能把 92.5/94.4 绑定到服务端 revision、数据 hash、完整模型 stack、随机性或逐题 output 的 run manifest。
- **提交链提供的版本语义**：`mem0ai/memory-benchmarks@edcd6f1` 首次加入 `results/platform/locomo_results.json` 与 `results/platform/longmemeval_results.json` 等结果文件；其 metadata 分别记录 `20260406_170942` / `20260406_101243`。后续 `4b61c5d31b9c668a12b4f5e78064248a02c82d2b` 的提交说明为 “update Mem0 Platform benchmark results with v3 + temporal reasoning”，但 diff 只修改 `README.md`，没有替换上述 Platform JSON。该提交将 README 结果说明更新到 LoCoMo 92.5 / LongMemEval 94.4，并把它们描述为 v3 + temporal reasoning 的更新结果。
- **Mem0 SDK 文档链**：`mem0ai/mem0@30469ae` 的 “New Memory Algorithm (April 2026)” 标题对应表格中的 91.6 / 93.4；之后的文档同步提交将当前算法说明更新到 92.5 / 94.4，但根 README 标题仍保留 April 标注。因此，标题滞后与数字更新可以解释为文档演进线索，不能单独证明新版数字的具体运行时间。
- **两组数字的关系**：91.6/93.4 与 92.5/94.4 不是简单四舍五入差异。现在可以将“官方材料把两组结果放在不同算法/配置代际”记为 `Observed`；“新版结果来自不同于旧 JSON 的评测执行或配置”是有提交链支持的 `Inferred`；但新版具体 run provenance 仍为 `Unverified`，不能绑定到某个服务端 revision。
- **固定快照的 OSS 结果**：`results/oss/` 只提供 LongMemEval 结果，公开标签为不同 memory extraction model 的 88.6–91.0；README 称这些 run 共用 Qwen 600M/SageMaker embedder、Qdrant、GPT-5 answerer/judge，部分 artifact 记录 `top_k=200`、`seed=42`。当前没有 OSS LoCoMo artifact，不能用这些结果计算 Platform/OSS 的 LoCoMo 差值。默认 OSS Docker 配置也不等于该对照实验配置。
- **可比性缺口**：评测 runner 使用浮动的 `main` 数据 URL，artifact 未统一保存数据 commit/hash；Platform extraction、embedding、向量/图存储、服务端 revision、重试/温度等也未完整写入 manifest。因此“固定子模块可运行”不等于“92.5/94.4 已被复现”。

### 3.4 评测争议的一手材料状态

- **Zep ↔ Mem0**：三组数字 `75.14% ± 0.17`、`58.44% ± 0.20` 和 `65.99% ± 0.16` 都是在比较 **Zep** 的 LoCoMo 结果：前者是 Zep 博客对“correct implementation”的自报，第二个是 Mem0 方在 `getzep/zep-papers#5` 对 Zep 的复核主张，第三个是 Mem0 论文 Table 2 报告的 Zep baseline。它们不是 Zep 与 Mem0 自身成绩的三方对比。当前材料支持“数字和双方主张存在”，不支持把三者合并成同一协议下的可比事实；实现、题类/分母、时间字段、检索并发性和 judge 聚合仍需逐项核验。
- **Zep 代码与博客的 C5 绑定缺口**：公开 Zep 代码提交曾将 Category 5 的 `adversarial_answer` fallback 移除，且修正路径使用 `category != 5` 与固定 `/1540` 分母；但部分代码快照仍可见 `gold_answer = qa.get("answer") or qa.get("adversarial_answer")`，并未提供与 `75.14% ± 0.17` 绑定的逐题 grades、运行次数或误差计算 artifact。因此只能确认代码演进和“排除 C5”的协议主张存在，不能由静态代码独立复算 75.14 或证明博客结果严格采用了该过滤。
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
- `Unverified`：上述 99 项分类是否都构成 score-corrupting error、judge 宽松比例以及“公开高分因此无效”等实质结论；本轮只完成 6 条静态抽查，未全量复核、未重算；
- `Inferred`：若不同实现使用不同模型、prompt、judge、题类筛选和分母，跨报告分数存在显著可比性风险。

本轮只做了有限静态抽查，未运行重算脚本，也未调用 LLM judge：

- `data/locomo10.json` 的 SHA256 与固定 commit 中 `scripts/verify_sha256.py` 声明的 `79fa87e90f04081343b8c8debecb80a9a6842b76a7aa537dc9fdf651ea698ff4` 一致；
- 固定数据共 1,986 个 QA，其中 Category 1–4 合计 1,540，Category 5 为 446；
- `errors.json` 共 156 条、156 个唯一 `question_id`，没有独立的 `score_corrupting` 字段；`99/57` 来自审计者把非 `WRONG_CITATION` 的 99 项归为 score-corrupting、把 57 项 `WRONG_CITATION` 归为 citation-only；
- 抽查 6 条 score-corrupting 标注：`locomo_0_qa5`、`locomo_1_qa57`、`locomo_9_qa72`、`locomo_9_qa1` 与所引对话存在审计者指出的静态不一致；`locomo_0_qa2`、`locomo_0_qa32` 只能确认金标比原文更具体或可能偏窄，不能仅凭静态文本判定为计分级错误。

因此，抽查支持“审计标注不是空泛计数，至少部分条目能在原始对话中定位”，但不支持把 99 项整体升级为已确认的数据集事实错误，也不支持由此宣布公开分数失效。

因此它适合作为 benchmark protocol conflict 的研究入口，不足以单独推翻任何一方成绩；相关协议层冲突已登记在对象级 [`../conflict.md`](../conflict.md)。

### 3.6 Node / TS “同步阻塞 30–40s”说法

本轮定向搜索未找到支持该具体时长或“同步阻塞”语义的一手 issue、release note、性能测量或官方文档。固定 TypeScript client 的 `add()` 是 `async` HTTP 请求；v3 integration test 以 `PENDING`/eventId 描述服务端异步处理，并通过轮询等待索引可见性。

这些源码只能支持“该版本的请求与后处理是异步接口/流程”，不能证明所有历史版本、OSS 自托管或外部 provider 都没有延迟问题。因此该说法已于 2026-10-30 退役为 `Deprecated`，不再作为已确认缺陷或待溯源线索保留。

## 4. 下一轮建档目标

1. **PyPI 发布对应关系**：版本与 release 时间已高度吻合；PyPI JSON 无 Git SHA，未下载工件，故“由该 commit 构建”仍保持高置信 `Inferred`。
2. **评测配置可比性**：已恢复 Mem0 benchmark 框架的具体 commit、主要 Platform/OSS 结果分区和可见模型/embedding/judge/top-k 边界；后续只补缺失的 run manifest、数据版本与服务端 provenance，仍只读静态核验，不运行 benchmark。
3. **`dial481/locomo-audit`**：已固定第三方审计 commit，并完成数据 hash 与 6 条代表性 score-corrupting 标注的静态抽查；下一轮只补 judge 输入输出和统计重算，不把 6 条抽查推广到其余标注。
4. **Node 同步阻塞 30–40s**：未找到一手支持，2026-10-30 已退役为 `Deprecated`，不再保留为线索。
5. **Platform 版本链**：公开 Dream / supersede、`latest_only` 与 `delete_linked` 契约已核验；服务端建链字段、latest 判定、递归范围以及 `replaced_by` / `linked_memory_ids` 映射仍未核验。

## 5. 整体解释与检索补查（2026-10-30）

以下均恢复到 `mem0ai/mem0@94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`，不改上游、不调用真实 API、不执行 benchmark。Platform 文档也是该 commit 的快照，不表示线上当前版本的实现或效果。新增观察支持整体解释与检索正文，不重开已有评测争议。

### 5.1 应用闭环、组件与数据

- [固定 README](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/README.md#L202) 的 `chat_with_memories`（202–236）展示先 `search(..., filters={"user_id": ...}, top_k=3)`，取 `results[].memory` 拼入 system prompt，调用回答模型，再将本轮 user/assistant 消息送入 `add()`。检索结果进入上下文由应用显式编排，不能把 Mem0 描述成自动改变回答模型。
- `mem0/memory/main.py:488–605` 创建 LLM、embedder、vector store、SQLite history 和派生 entity collection；`920–1047` 抽取、embedding 并准备 memory payload；`1015–1047` 为每条记录保留文本、hash、时间和 scope。此路径是外部记录与推理调用，没有模型权重训练步骤（`Observed`；不推论外部模型服务自己的数据政策）。
- `mem0/memory/storage.py:102–150,257` 区分 memory 变更 history 与 messages 表；后者按 session_scope 保留最近 10 条经过管线处理的消息字段，不是完整原始会话归档。实体索引保存实体文本及关联 memory IDs，不替代主记录。
- `main.py:1393–1518` 的公开检索数量参数为 `top_k`，内部才赋值 `limit = top_k`。不能将内部变量名当作该固定 API 的公开参数。`add/search` 主要返回 `{"results": [...]}`；搜索结果包装见 `1704–1745`。

### 5.2 作用域：OSS 与 Platform 不能合并

- [固定 OSS metadata/filter helper](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/main.py#L314)（314–410）要求 `user_id/agent_id/run_id` 至少一个有效 ID；add 将提供的 IDs 同时放进 metadata。普通抽取逐条构造记录时 `deepcopy(metadata)`（1008–1047），`attributed_to` 只增加归因字段，不删除另一个 scope ID；异步对应循环同样如此（2696 起）。
- OSS search（1393–1477）要求 ID 放入 `filters`，拒绝顶层 ID，且没有自动补出的默认用户范围；未提供的维度不主动增加条件。默认 Qdrant 在 `mem0/vector_stores/qdrant.py:368–386` 将平铺字段放入 `must`，多 ID 是 AND；其他 provider 的过滤能力需分别核对。
- [Platform Entity-Scoped Memory](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/entity-scoped-memory.mdx#L34)（34–46、80–84）另有契约：默认抽取按说话方使记录带 user 或 agent，而非同时带两者；`app_id/run_id` 随记录保留。未指定维度不受限；AND user+agent 会漏掉按此规则产生的记录。Direct Import 是例外，不能将这一归属拆分外推到 OSS。
- 三种 OSS IDs 是 metadata/query filter，不能从这些路径证明是授权机制；Platform 客户端转发和服务端 ACL 也应区分。实际应用需要自行确认调用者身份与查询范围一致。scope IDs 与文本中抽取的图实体是两种概念。

### 5.3 检索信号、过滤与最终重排

- [固定 `_search_vector_store`](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/main.py#L1642)（1642–1701）先预处理查询和实体、embedding 原查询，semantic 与 keyword 搜索各请求 `max(limit*4,60)`，都收到完整 filters；entity boost 的索引检索只取平铺的非空 user/agent/run 字段（1770 起），不复制其余 metadata 条件。最终记录资格仍来自 semantic 候选；不能据此声称三路过滤实现完全相同。
- keyword 正分数经 `mem0/utils/scoring.py:16–54` sigmoid 变换：`1/(1+exp(-k*(raw-m)))`；m/k 随词形还原后查询长度分档，短查询（≤3 terms）为 5.0/0.7。非正 hit 不写入分数字典，缺失 ID 在融合时贡献 0。
- `main.py:1747–1827` 对查询前 8 个实体再按规范化文本去重，embedding 后查 entity store；相似度低于 0.5 的匹配跳过，boost 为 `similarity*0.5/(1+0.001*(linked_count-1)^2)`。同一 memory 多次命中取最大值，不累加；不将 0.5 写成独立硬 clamp。可选 spaCy full model 缺失时 `mem0/utils/entity_extraction.py:751` 返回空实体；lemmatization 缺模型时则回退原 query（`lemmatization.py:22–30`）。
- `main.py:1680–1701` 在构造 semantic candidates 时先排除过期 payload，`scoring.py:94–139` 再进行原 semantic threshold、字典决定分母、加法融合、降序与 top-k。`explain=True` 可返回三项贡献与分母；创建/更新时间不参与此函数。
- `main.py:1508–1518` 的可选 reranker 接收已经 hybrid top-k 截断的结果，再重排；外层失败回退原结果。它不能补回原 semantic 池外、threshold 以下或已被首轮 top-k 截掉的记录。标准 provider 可附 `rerank_score`，因此不能假设启用后顺序仍严格按原 hybrid score。

### 5.4 Platform 能力的概念与契约

- [Graph Memory](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/graph-memory.mdx#L25)（25–55）是实体与提及它的记忆相连，查询匹配实体参与排序；不是预定义 `manages` 等有类型关系边，也不单独返回可解析图 payload。旧外部图方案、当前 OSS entity index 和 Platform 原生图需分开。
- [Dream](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/dream.mdx#L19)（19–88、114–158）将 add-time supersede/merge 与 scheduled Synthesis 分开。Synthesis 新增带来源链接的 pattern memory，保留原记录；项目 opt-in、Pro+、只纳入开启后新增且仅带 user_id 的记录，不含 agent/app/run scope，至少 20 条后按套餐周期运行，非实时。默认读仍含 superseded，不因功能名“清理”就推成 active-only。
- [Temporal Reasoning](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/temporal-reasoning.mdx#L7)（7–71）按记忆事件日期与查询时间表达的匹配加排序 boost；`timestamp` 保留导入对话的原时间，`reference_date` 指定查询参考“现在”。它不是单纯按创建时间取最新，不能移植为固定 OSS 功能。
- [Memory Decay](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/memory-decay.mdx#L23)（19–46、138–146）项目 opt-in、默认关闭；依据检索访问历史给分数乘 0.3–1.5，异步强化返回记录，是软排序而非删除/过滤；无访问历史时有 event_date/updated_at 回退。公开文档的候选池和乘法公式属于 Platform，不能与 OSS 加法融合公式混用。个别“不会丢结果”文案只宜理解为不额外硬过滤，top-k 截断仍可能改变返回集合。

### 5.5 独立能力扫描与覆盖选择

以固定 `docs/docs.json`、`docs/platform/platform-vs-oss.mdx` 及能力页扫查，除原有主线外，影响整体认识的内容包括：filters/作用域、custom instructions、直接导入、过期、可选 reranker 和输入模态。`docs/platform/features/custom-instructions.mdx` 与 OSS 同名页分别定义项目/调用设置和 `MemoryConfig.custom_instructions`，它们控制抽取内容而非事后查询过滤。Platform multimodal 页与 OSS 页说明图片等输入需适用的抽取模型；不据此宣称独立的多模态向量检索。

`docs/platform/features/user-profiles.mdx` 的 Profiles 是 beta、按 JSON Schema 生成每用户结构化摘要，与 Dream pattern memory 不同。总览保留其旁支定位；各集成、管理面、部署调优和遗留 procedural 路径不作首轮机制级研究。Platform Direct Import 的精确去重契约也不外推到已有 M3d 所记 OSS 逐条追加。

## 6. 普通写入机制补查（2026-11-02）

**Version Basis**：Python OSS `v2.2.1`，`mem0ai/mem0@94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`。两路只读核查确认所读上游 origin、HEAD 与干净工作区。**Observed At**：2026-11-02；**Drift Risk**：high。未运行 API、模型、benchmark 或故障注入。

公开固定来源及复核定位：

- [main.py](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/main.py#L760)：scope helper L360–419；add L760–879；infer=False L882–916；普通路径 L920–1220；`_create_memory` L1975–2005；async 对照 L2564–2901。
- [utils.py](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/utils.py#L61)：消息渲染 L61–76，代码块处理 L115–158，vision L186–232。
- [prompts.py](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/configs/prompts.py#L468)：additive 提示 L468–701，上下文格式与截断 L918–991、L1016–1062。
- [storage.py](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/storage.py#L150)：history 单条／批量各自事务，messages L257–324 另有事务与 scope 裁剪。
- [OSS 自定义指令示例](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/open-source/features/custom-instructions.mdx#L31)：facts 格式与 main.py 的 memory 消费结构不一致。

**M41–M42**：全部有效 user/agent/run 组合决定 session key 和旧记忆召回 filters；只取最近十条短消息与后端 top10 旧候选。新消息渲染忽略其他角色；旧消息 content 最多300源字符另加省略号，提示限定只辅助新消息理解。`custom_instr = prompt or self.custom_instructions`。一次 JSON object LLM 调用后取 `.get("memory", [])`。归因 truthy 原样复制，不作值域校验；id/event/linked_memory_ids 不被用于生成操作或事实替代链。

**M43**：所有 truthy text 先 batch embedding，异常逐条 fallback，失败文本跳过；`dict(zip(...))` 不校验批结果长度，短结果可能漏文本。去重仅召回旧 hash 与本批 hash。每条生成 UUID4、复制 metadata、写 data、词形归一文本和 hash；created_at 可来自已有 metadata，updated_at 初始相同。batch insert 不抛错即认定全批 persisted；失败以原 ID 逐条重试，不读回验证、也不检查 insert 返回值。

**M44**：全部主 insert 调用失败，先 save_messages 再抛 VectorStoreError；messages 失败可覆盖预期异常。部分主 insert 未抛错，仅该子集进入 history／entity／results。history 批量失败逐条重试，单条再失败记日志继续；普通 history 不传 updated_at/actor_id/role，不是完整 payload。实体按 normalized text 合并 IDs；批量查询覆盖全部有效实体，选结果时 exact 优先，否则向量 top1、相似度至少0.95，不能将结果优先级读成按需执行向量查询。batch entity search/insert 没有逐条 fallback，相关错误被捕获。最后 save_messages 失败向外传播，其事务不回滚主存储、history 或实体。消息同批共用时间，按时间排序没有额外 tie-break，不能承诺同批稳定顺序。Async 主要阶段与失败边界相应，to_thread 不提供全局回滚。

infer=False 的 `_create_memory` 为逐条 embedding→vector insert→history；history 错误传播，无普通 recall、去重、实体、messages 保存；vision 预处理仍先于该分支。不能把普通路径的 history 容错或 Platform 导入去重契约套用到这里。

**M45**：合法 JSON 只含 facts 时，缺少 memory 得到 []，尝试 save_messages 后空返回；解析错误被捕获不等于结构完整校验，`{"memory":["fact"]}` 会在后续 m.get 失败。上述是指定输入下的静态分支推演，未观测真实模型按冲突指令会输出什么。

**Decision / Placement / Gap**：稳定机制提炼到 [`../write-pipeline.md`](../write-pipeline.md)，入口联动 README／overview；新增 Claim M41–M45。源码分支为 `Observed`，重试和残留后果为条件性 `Inferred`；模型效果、故障现场、并发及服务端实现保持 `Unverified`。

## 7. 派生记忆与 Profiles 补查（2026-11-02）

**Version Basis**：`mem0ai/mem0@94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`；本轮两路只读核对确认 origin／HEAD 与干净上游工作区。**Observed At**：2026-11-02；**Drift Risk**：high。固定文档代表公开契约，SDK只代表客户端行为，不把同commit当作托管服务端revision。

公开固定来源：

- [Dream](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/dream.mdx)：L29–55 模式追加、来源、evidence幂等与纯user范围；L76–99读取／关闭；L103–164资格、20 memories、cadence与约24h延迟。
- [OpenAPI Dream](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/openapi.json#L7211)：config L7211起（含最近开启时间reflection_enabled_at）；stats L7443；activity L7528；runs L7659；run memories L7808；sources L7918；preview L8007。配置修改要求organization owner及套餐资格，preview为capped sample/count而非LLM生成。
- [Profiles](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/user-profiles.mdx)：L19–34 beta/schema；L105–175读取及按需触发；L177–253 sample、schema延后生效、自动约10 messages、异步增量；L283–348预算、资格、关闭读取和字段裁剪。
- [Profiles OpenAPI](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/openapi.json#L8074)：profile read L8074、settings L8160、jobs L8336、job status L8467；实体删除另见L483–524，200仅表已排队。
- [Profiles notebook](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/examples/notebooks/user-profiles.ipynb)：L20 Pro+，L158与729–730 flush window，L375–376 whole rewrite，L659–662关闭后已有内容可读，L692 entity级联profile row，L734–735 insufficient_data涵盖queued。仅阅读notebook文件，未执行其中代码。
- [Python客户端](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/client/main.py)：Profiles L807–952（及async对应）；设置辅助L51–78，delete_linked注释L503–527；[TS客户端](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0-ts/src/client/mem0.ts)及其[utils](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0-ts/src/client/utils.ts)：设置嵌套、幂等键、envelope转换并保留用户schema键。

**M46–M47**：Dream模式加入普通记忆集合；synthesized与lifecycle_state分属来源与状态，replaced_by不是sources。来源接口返回source ID/text/lifecycle_state；runs列表next_cursor与运行内has_more/mem_cursor分层分页，run顶层sources仅对应内联第一页去重，runs及run-memories的source text明确truncated，翻页不保证取得完整来源文本。stats水位为项目级，非用户输入版本；activity不含合成。固定v3 get-all schema有synthesized，search schema未声明；Python kwargs可透传include_merged、TS类型未全面覆盖，不升级为服务端已验证。没有找到受支持的synthesized服务端过滤契约，故不据此编写调用示例。

**M48**：Profiles固定user与项目边界；REST需entities.user嵌套，客户端平铺入参由helper转换。schema/null可清配置，不能反推已有内容同步清除。主文档约10 messages阈值，具体计数未知；sample最多10真实实体、保存并计费，非Dream preview。作业幂等键8–128字符，同键重放同一请求返回同job，异请求同键或已有job运行可409、cooldown可429，客户端省略时新生键。作业状态与画像状态不同；updated_at/generation_count不是源memory水位或逐字段证据。notebook L404–414警示应先确认add event成功，否则generate可能只用旧memories；示例固定等待不是服务保证。全量rebuild/backfill未开放；schema移除字段在下次生成时裁剪。

**M49**：主文档L296与notebook L659–662对关闭后已有画像读取相反；主文档L247的无schedule描述约10 messages触发，notebook L157–158明确Below that等flush，可兼容而非直接冲突；L253增量与notebook L375–376整体重写可能分属输入与JSON输出层，不据此裁决算法；L155的insufficient_data与notebook L734–735涵盖queued有口径差异。另有主文档要求description与notebook不强制校验的差异，不将生成建议误写成服务端强校验。对象conflict §6登记并区分直接冲突和可兼容差异，不以notebook自动覆盖主文档。

**M50 / 搜索边界**：交叉检索固定docs、OpenAPI、Python/TS客户端及测试、server、cli、skills与integrations的Dream/Synthesis/reflection/Profiles/source/delete/update相关项，未找到托管worker实现，或源memory原地更新／单条和批量删除后向模式／画像传播的明确契约。通用delete_linked说明涉及superseded旧链，不能扩成合成来源边或profile删除；notebook称删entity级联profile，实体删除API仅排队，二者不等于delete_all memories。插件旧Dream consolidation也不能移植为当前Platform契约。此处“未找到”限定所查公开快照，不表示服务端不做传播。

**Decision / Placement / Gap**：公开机制与使用边界提炼到 [`../derived-memory.md`](../derived-memory.md)，联动README、overview、conflict与问题树；M46–M50区分契约、客户端观察、条件推演及冲突。未调用API、执行notebook、安装依赖或测生成质量；服务端传播、输入快照、重试／原子性、资格与刷新差异继续等待明确材料或获授权后的受控行为证据。

## 8. 访问强化与时间排序补查（2026-11-02）

**Version Basis**：`mem0ai/mem0@94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`；两路调查先核HEAD和干净上游工作区。**Observed At**：2026-11-02；**Drift Risk**：high。文档契约不等于托管revision或实测效果。

- [Decay](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/memory-decay.mdx)：L20项目默认关闭且只v3；L25–46乘数0.3–1.5、候选扩展、未clamp排序、最终返回项异步touch、最近20次及回退；L63/88配置；L109/129关闭可逆及历史保留；L146 threshold先于Decay；L157/173未来事件约1.0；L179细节不公开；L182 rerank先于Decay。
- [Temporal](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/temporal-reasoning.mdx)：L7–15 Platform v3自动事件匹配、通常数秒生效，L47–71 reference_date与timestamp。异步时间元数据另见[评测概念页](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/core-concepts/memory-evaluation.mdx#L31)；L47–54描述并行信号、融合及temporal additive，不能补齐完整顺序。
- [Expiration](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/memory-expiration.mdx)：L11–20 UTC日期包含当天、列表式读过滤、get仍读、坏日期fail open；L78–98 show_expired及候选不足；L138–149与Decay/delete区别。
- [OpenAPI](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/openapi.json)：L2862–2864 timestamp回填created_at；L3016 threshold语义、L3023–3036 reference_date；L3101起多信号score及score_breakdown（semantic/BM25/entity，不含Decay）；L5707、5832项目decay。
- [Python客户端](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/client/main.py)：get/get_all/search分离L343–460，async search L1527–1539只等待HTTP；配置在[project.py](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/client/project.py#L395)。反馈另见[feedback机制](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/feedback-mechanism.mdx#L44)。
- [OSS scoring](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/utils/scoring.py#L94)：L94–139仅semantic/keyword/entity；[main.py](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/main.py#L1642) search L1642–1701候选/过期/评分，可另rerank；get L1222–1267、get_all L1324–1391；history经[storage.py](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/storage.py#L227)按ID，无过期过滤。

**M51–M52**：明确threshold→Decay、rerank→Decay→截断及最终返回项异步touch，未公开完整融合/Temporal/threshold/reranker顺序及强化函数。0.3是倍率非score地板，公开clamp不能反推内部乘积。软调整不清零不等于top-k集合不变；Decay L135的exact top_k需与过期页承认短结果并读，不能保证必足条数。

**M53**：无历史回退event_date，再updated_at，非created_at；timestamp不直接等于event_date。Decay L46未来事件fresh/full activation与L157/173近1.0有措辞张力，不直接将fresh解释成1.5或断言数值矛盾。reference_date未据此控制其他状态时钟。

**M54**：强化最终search返回而非仅候选，不需点击／回答采用／反馈；get/get_all是否记访问未找到明确契约。客户端再筛选不撤回已发起touch，updated_at不是强化完成标志。score_breakdown无Decay项，不能把single-score概述扩成所有细分都不存在。重复query幂等、并发原子追加、失败重试、可见延迟和恰好一次未查到保证。

**M55 / 搜索边界**：检查固定decay/Temporal/expiration/feedback/Dream文档、OpenAPI、Python/TS客户端及配置/测试、Python OSS scoring/main/storage。未找到托管Decay实现；已核OSS无访问乘数，不外推到自定义provider/reranker。show_expired只撤过期排除，不取消其他过滤；Dream组合开关优先级未知。

**Decision / Placement / Gap**：稳定解释提炼到 [`../memory-decay.md`](../memory-decay.md)，联动README/overview/retrieval与问题树，M51–M55登记。连续查询和乘法例为条件推演；未运行API、模型或排名实验，实际偏置／效益、服务端时序和并发仍缺证据。

## 9. 直接更新来源重建（2026-11-03）

**Version Basis**：`mem0ai/mem0@94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`，Python OSS `v2.2.1`；**Observed At**：2026-11-03；**Drift Risk**：high。整体验收发现旧更新来源锚点失效、M32–M35登记缺失，本节为重新核对后的来源重建，不是恢复历史原文。下列相对路径均相对于[固定上游根目录](https://github.com/mem0ai/mem0/tree/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd)。

| Claim | 固定路径与定位 | 核对结果及边界 |
|---|---|---|
| M32 | `mem0/memory/main.py:143–162,1829–1881,2052–2112`；async `3507–3560,3770–3832` | 公共入口与内部函数连读；同ID主写，metadata-only仍embedding，身份键过滤，expiration独立参数优先。scope处理不是调用者授权证明 |
| M33 | `mem0/memory/storage.py:102–125,150–191`；`main.py:652–728,2361–2430` | history表及插入事务只保存部分字段；主写先于history，实体维护尽力、扫描最多10000。部分失败状态为控制流推演 |
| M34 | `main.py:1222,1393,1960–1973`；`mem0/vector_stores/qdrant.py:500–520`；`pyproject.toml:17` | get/search/history不同数据路径；同ID upsert，dense新值、sparse有条件重算；依赖只限定qdrant-client>=1.12.0 |
| M35 | `mem0/client/main.py:463–501,566–587`；`docs/openapi.json:3553,3852` | Platform单条更新PUT和历史GET；不能观察服务端事务、索引刷新或Dream联动 |

外部依赖另固定到 [qdrant/qdrant-client@v1.12.0 的 qdrant_client.py](https://github.com/qdrant/qdrant-client/blob/v1.12.0/qdrant_client/qdrant_client.py#L1472)。本轮重新获取原始源码确认 `upsert(..., wait: bool = True, ...)`，说明等待变更应用；不把省略wait说成只入队。实际安装版本、注入client、服务端和读一致性配置未知，不能承诺任意部署即时search命中。

**Decision / Placement / Gap**：为 [`../direct-update.md`](../direct-update.md) 修复来源终点与M32–M35登记；固定控制流为Observed，失败及读取后果为Inferred，部署行为和服务端实现仍Unverified。未运行API或故障注入。

## 10. 删除与重置来源重建（2026-11-03）

**Version Basis / Observed At / Drift Risk**：同§9。旧删除来源锚点失效、M36–M39登记缺失，本节按固定源码重新核对并重建，不声称找回原始登记。路径均相对于§9固定上游根目录。

| Claim | 固定路径与定位 | 核对结果及边界 |
|---|---|---|
| M36 | `mem0/memory/main.py:1883–1902,2114–2142`；async `3562–3581,3834–3863` | 先get，再vector→DELETE history→entity；不清messages，history失败无vector回滚 |
| M37 | `main.py:136,1904–1958,2339–2359,3583–3654`；`mem0/vector_stores/qdrant.py:401–410` | 每批1000，重复批次正常break；sync异常中止，async gather return_exceptions继续、批后实体最多10000；固定Qdrant平铺scope为AND |
| M38 | `main.py:2144–2174,3865–3896`；`mem0/memory/storage.py:326–339`；`docs/openapi.json:3761–3784`；`docs/changelog/sdk.mdx:330` | reset的SQLite表范围及sync/async顺序不同；Platform delete_linked、cascade_count和旧链说明仅支持公开契约／客户端说明，不能确认内部遍历原子性 |
| M39 | `main.py:920–954,2604,2636`；`mem0/configs/prompts.py:511,519–521` | 后续普通抽取读取同scope最近消息；提示词限定旧消息仅作理解上下文、新消息才是抽取来源。缓存影响有条件，不是旧事实复活实测 |

**Decision / Placement / Gap**：来源供 [`../deletion-lifecycle.md`](../deletion-lifecycle.md) 与M36–M39引用，普通写入另见§6。主链、清理对象和异常处理为Observed，残留与缓存风险为Inferred；真实provider、并发／故障及Platform级联执行仍Unverified。未执行删除、reset或其他真实API。

## 11. 对象整体验收补查（2026-11-03）

**Version Basis**：同一Mem0固定commit及§3已列论文版本；**Observed At**：2026-11-03。这里只登记本轮补查与纠偏，不表示重新访问全部外部网页。

- **M56，Platform新增受理与完成**：[固定 add-memories.mdx](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/api-reference/memory/add-memories.mdx#L63) L15、63–84声明异步新增返回 `event_id`、`status="PENDING"`，并要求用 `GET /v1/event/{event_id}/` 查询至 `SUCCEEDED` 或 `FAILED`。这是公开契约（Verified）；受理不等于完成，不由成功终态推导任意search命中、Synthesis／Profiles均已刷新，也不外推到OSS async或Platform update。
- **M24，评测可观测性纠偏**：§3.1–§3.2支持任务定义与缺少专门机制指标的边界，不支持“数据没有新旧状态”的排除断言。LongMemEval避免无关session冲突，不是排除knowledge-update中的状态变化。问答分数不能直接区分内部覆盖、共存、过滤或恰好未召回，是Inferred的指标边界；可在扩展协议增加逐轮快照与检索日志。
- **范围精度**：总览将payload过期过滤限定为已核Python路径；检索例子使用 `expiration_date < 查询时UTC日期`，日期当天不按此条件隐藏。Dream重复直接保留canonical与已有独立旧项标merged的区别，沿用§1.3.8及现有一致性正文，不新增服务端机制断言。

**Decision / Placement / Gap**：修改总览、评测与检索的对应解释，补齐Claim和来源断链；实际异步延迟、运行效果和未公开服务端实现仍需对应证据。
