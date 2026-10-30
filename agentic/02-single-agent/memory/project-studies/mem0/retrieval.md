# Mem0 检索机制

> 最后更新：2026-10-30
> 版本锚点：固定 Python `v2.2.1`、TypeScript `ts-v3.3.1`，commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`

本文回答 Mem0 研究的第二条机制主线：一次 `search()` 如何把查询变成排好序的记忆，这一层实际做了什么、没做什么。它讲的是固定 OSS 的读取路径，与 [`consistency-model.md`](./consistency-model.md) 的写入/状态一致性是两条不同的线。

## 候选集由语义搜索构造

固定 Python OSS 的 `_search_vector_store` 依次获取 semantic、keyword/BM25 和 entity 三类信号，但**候选集只由 semantic results 构造**：语义检索先做 `max(limit * 4, 60)` 的 over-fetch，关键词与实体信号只对已进入候选池的记录参与打分。因此它不是“三路并行召回取并集”。

这一点决定了整个检索层的边界：没有进入语义候选池的事实，后续任何信号都无法把它找回来。

## 融合评分

过滤过期 payload 后调用 `score_and_rank`。评分先做门控：原始 semantic score 低于 threshold 的候选在融合前就被丢弃，BM25 或 entity 不能把它拉回。通过的候选按 `min((semantic + bm25 + entity_boost) / D, 1.0)` 融合，其中 `D` 由本次调用的 BM25 / entity 分数字典是否非空决定。

评分函数不读取 `created_at` / `updatedAt`：只交换候选的时间字段不会改变排序和分数。因此“更相关”不能推成“当前事实正确”。

## hybrid 是后端条件能力

“hybrid retrieval”不是 SDK 的统一能力，而是后端和配置条件：

- 基类 `keyword_search()` 默认返回 `None`；未覆盖该方法的后端在初始化时记录 warning 并关闭 BM25，搜索退回 semantic-only。
- 覆盖该方法的后端也不都实现 BM25：Qdrant 依赖 `bm25` sparse slot 与 fastembed，Pinecone 需显式开启 hybrid，pgvector 是 PostgreSQL 全文检索，Vertex AI Vector Search 覆盖后仍返回 `None`。
- TypeScript 同样是可选方法 `keywordSearch`，缺失、返回 `null` 或抛错时都静默降级为纯 semantic（+entity）排序，不报错、不提示。
- Qdrant 还要求新建 collection 带 `bm25` sparse slot；旧 collection 缺该 slot 时只关 BM25，语义检索继续工作。

因此不能按 SDK 版本号假定所有部署都启用了三种信号；哪些后端真正跑 BM25 必须逐个核对。

## 这一层能做什么、不能做什么

可以把以下当作稳定理解：

- 它负责“进入 semantic 候选后的相关性重排”；
- 它按调用方设置的 `expiration_date` 隐藏过期记录（默认 `show_expired=False`），但不自动判断哪条事实更新；
- 它不负责把语义分数低于 threshold、或不在候选池内的记录找回；
- 它不按时间判断新旧，也不执行矛盾裁决。

因此，检索排序、公开 benchmark 和社区争议都不能单独证明长期事实管理已经可靠；新旧事实冲突是否在更上游通过抽取或实体链接被处理，见 [`consistency-model.md`](./consistency-model.md)。

## Evidence

- **Status**：固定源码路径与机制为 `Observed`；融合公式、时间字段，以及“候选池外 / 低于 threshold 的记录不能被其他信号找回”为函数级 `Observed`（端到端效果未验证）；“相关性不等于事实正确”为 `Inferred`。
- **Sources**：固定 commit `94c3fe9f…` 的 `mem0/memory/main.py`、`mem0/utils/scoring.py`、`mem0/vector_stores/*`；`mem0-ts/src/oss/src/memory/index.ts`、`.../utils/scoring.ts`、`.../vector_stores/*`。
- **Trace**：Claim 级来源与状态见 [`notes/evidence.md`](./notes/evidence.md)（M4、M4a、M4b）；源码摘录与后端覆盖见 [`notes/source.md`](./notes/source.md) §1.3–§1.3.3、§1.3.1.1–§1.3.1.2。
- **Needs**：真实样本下 semantic threshold 与各信号权重的端到端效果、以及同一实体多个矛盾值的实际返回行为仍未验证。
