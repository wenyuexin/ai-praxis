# Mem0 一致性模型

> 最后更新：2026-10-30
> 版本锚点：论文 `arXiv:2504.19413v1`；固定 Python `v2.2.1`、TypeScript `ts-v3.3.1`，commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`

本文回答 Mem0 研究的机制主线：当新事实与旧事实冲突、重复或只是随时间变化时，论文旧版、开源 SDK 和托管 Platform 分别在哪一层处理，哪些责任落在写入期、哪些落在读取期，哪些仍无法从公开材料确认。

## 为什么必须分层

“Mem0 如何处理旧事实”没有单一答案。论文、固定 OSS 和 Platform 对同一问题给出了不同机制，且分属不同部署形态。把它们压成“Mem0 会更新旧记忆”或“Mem0 只追加不更新”都会失真。

## 版本谱系

OSS 的 ADD-only 不是从论文旧版逐步演化出来的默认路径，而是一次明确的算法代次切换：

- `v1.0.11`（2026-04-06）及更早：普通自动抽取仍包含 UPDATE / DELETE / NOOP 分支，并调用 `_update_memory` / `_delete_memory`。
- `a488e190`（2026-04-14，PR #4805）：移植 v3 pipeline，普通自动抽取改为单趟 ADD-only + hybrid retrieval + entity extraction；父提交仍保留旧分支。
- 首个包含它的发布：`v2.0.0b1`（2026-04-14），稳定版 `v2.0.0`（2026-04-16）；当前锚点 `v2.2.1`（2026-09-25）。
- 包版本 2.x 与算法代次 v3 不是同一命名，不能把 `v2.2.1` 读成旧算法。

细节与 tag 证据见 [`notes/source.md`](./notes/source.md) §0.2 与 [`notes/evidence.md`](./notes/evidence.md) M22。

## 论文旧版：写入期变更

论文 `arXiv:2504.19413v1` 把基础 Mem0 的正式管线命名为 extraction + update。更新阶段先召回语义相似的旧记忆，再由 LLM 在 ADD、UPDATE、DELETE、NOOP 四值中选择。基础 Mem0 用 DELETE 移除被新信息矛盾的记忆；图方案 Mem0g 把过时关系标为 `invalid` 而不是物理删除，以保留时间语义。

这是论文设计，不能直接外推到当前 OSS 或 Platform 服务端；迁移文档里的 `extract + merge` 是概述用语，不替换论文的阶段名称。

## 固定 OSS：写入期对 memory 记录只追加

固定 Python `v2.2.1` 与同 commit 的 TypeScript `ts-v3.3.1` 中，普通自动抽取路径是单次 LLM 抽取后对 memory 记录只持久化 ADD（实体索引仍会更新）。它仍保留三类处理，但没有依据 LLM 输出执行 UPDATE / DELETE：

- prompt 要求做语义筛选、保留变化语境并关联矛盾旧记忆，但这是指令，不是运行效果保证；
- 原始文本 MD5 只对本次最多 10 条召回候选和当前 batch 去重，不是全库或语义去重；
- entity store 维护实体到记忆集合的索引，用于检索增强，不等于事实替代链。

普通路径也没有消费 LLM 返回的 `linked_memory_ids` 来建立 supersession 链。显式 `update()` / `delete()` API 仍然存在。

入口差异不能忽略：Python 的 `infer=False` 和 procedural memory 是直接追加，vision 只是写入前预处理；TypeScript 普通路径与 Python 相似，但未观察到 procedural 入口。因此“普通自动抽取是 ADD-only”不代表整个 SDK。写入也可能被延迟：`mem0/proxy` 的 OpenAI 兼容代理会在 daemon 线程后台 fire-and-forget 调用 `add()`，写入内容仍是 ADD。

## OSS 读取：相关性重排，不是矛盾裁决

OSS 检索先由语义搜索形成候选集，再在候选内融合 BM25 和实体信号。threshold 在融合前过滤语义分数，评分函数不读取 `created_at`。它提高的是相关性排序能力，不是矛盾事实裁决能力。

hybrid 还是后端条件能力：`keyword_search()` 在基类默认返回 `None`，未覆盖它的后端会关闭 BM25，退回 semantic-only；覆盖它的后端也不都实现 BM25（pgvector 是全文检索，Pinecone 需显式开启，Vertex AI Vector Search 覆盖后仍返回 `None`）。TypeScript 侧同样是可选方法、缺失或返回 `null` 时静默降级。因此不能假定所有部署都启用了三种信号。

读取还会按 payload 的 `expiration_date` 过滤过期记录：`search()` / `get_all()` 默认隐藏过期项，`show_expired=True` 才返回。该字段由调用方写入，不是 vector store 的 TTL，也没有默认过期时间；它不会自动判断哪条事实更新。

## Platform：写入期状态标记 + 读取期过滤

Platform 公开 Dream 契约把状态标记前移到新增阶段：矛盾的新事实可把旧记忆标为 `superseded` 并链接替代记忆，重复或更丰富的新事实可形成 `merged` 关系。默认读取返回 active 与 superseded、隐藏 merged；`latest_only=true` 只返回 active，`include_merged=true` 可纳入 merged。

这些是公开产品与 API 契约。客户端能转发相关参数，但服务端如何判定状态、建立关系、遍历删除链，以及事务和失败语义，尚未由公开材料确认。Temporal Reasoning 与 Memory Decay 也是 Platform-only；固定 Python OSS 明确拒绝 `reference_date`，不能把这两项能力赋予 OSS。

## 稳定结论与边界

可以把以下当作稳定理解：

- 一致性机制必须按论文、OSS、Platform 分层，不存在单一答案；
- 固定 OSS 的普通自动抽取减少了写入期显式变更，但仍保留 prompt 筛选、局部去重和实体索引；
- Platform 公开契约把状态标记前移到新增阶段，并用读取参数控制可见性；
- 读取期过滤还包含调用方设置的 `expiration_date`；它不是 store TTL，也不做新旧事实裁决；
- 检索排序、公开 benchmark 和社区争议都不能单独证明长期事实管理已经可靠。

仍不能写成定论：OSS 检索是否因果地补偿了旧版 update 的移除、Platform 服务端的实际建链与失败语义，以及真实持续交互中的写入污染和旧事实复活。这些缺口的关闭条件见 [`notes/deep-research-question-tree.md`](./notes/deep-research-question-tree.md)。

## Evidence

- **Status**：论文机制、Platform 公开契约为 `Verified`；固定源码路径、客户端行为与 hybrid 后端覆盖为 `Observed`；跨层因果与 Platform 服务端实现为 `Unverified` 或有边界的 `Inferred`。
- **Sources**：`arXiv:2504.19413v1`；固定 Mem0 commit `94c3fe9f…` 的 Python/TypeScript 源码、Dream 与评测文档。
- **Trace**：Claim 级来源、版本锚点和未闭合项见 [`notes/evidence.md`](./notes/evidence.md)；源码与契约摘录见 [`notes/source.md`](./notes/source.md)。
- **Needs**：Platform 服务端建链与失败语义、OSS 端到端冲突行为仍未闭合。
