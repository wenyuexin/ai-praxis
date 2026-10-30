# Mem0

> 最后更新：2026-10-30
> 版本锚点：论文 `arXiv:2504.19413v1`；Python `v2.2.1`、TypeScript `ts-v3.3.1`，commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`

Mem0 是一个跨会话记忆层，核心接口是 `add()` 与 `search()`。它值得单独研究，不是因为名字统一代表一套固定算法，而是因为论文、开源 SDK 和托管 Platform 对“新事实如何处理旧事实”给出了不同答案。

## 版本与部署形态

Mem0 是版本敏感对象：不同代次和部署形态会改变能力边界与实现判断，因此本目录按“目录级总览 + 机制专题正文 + 研究辅助材料”组织，而不是压成一篇静态对象文。

- **论文旧版**：`arXiv:2504.19413v1`，正式管线为 extraction + update，LLM 在 ADD / UPDATE / DELETE / NOOP 中选择。
- **固定 OSS**：Python `v2.2.1` 与同 commit 的 TypeScript `ts-v3.3.1`；自 `v2.0.0`（2026-04）起普通自动抽取为 ADD-only，另有 `infer=False`、procedural 等直接追加入口；检索为后端条件性 hybrid；过期记忆由调用方设置的 `expiration_date` 在读取期过滤。
- **Platform**：托管平台，公开 Dream 契约包含 add-time supersede / merge 与 read-time `latest_only` / `include_merged`；含 OSS 不具备的专有优化与 Platform-only 时间能力。

## 本目录的正文

- [`consistency-model.md`](./consistency-model.md)：机制专题正文，回答论文、OSS、Platform 三层一致性机制如何分层，以及各自处理到哪里为止。
- [`retrieval.md`](./retrieval.md)：机制专题正文，回答一次 `search()` 如何构造候选、融合打分，以及 hybrid 为何是后端条件能力。
- [`evaluation.md`](./evaluation.md)：评测边界正文，回答 LoCoMo / LongMemEval 能观测什么、不能观测什么，以及验证旧事实复活还缺什么协议。

## 研究边界

- 只覆盖公开可核验的材料；不调用真实 API、不运行 benchmark、不改上游仓库。
- Platform 公开契约不等于服务端实现；客户端参数转发不等于服务端行为。
- 论文旧版设计不外推到当前 OSS 或 Platform。

以下内容仍不能写成定论：OSS 检索是否因果地补偿了旧版 update 的移除、Platform 服务端的实际建链与失败语义、`92.5 / 94.4` 的完整运行配置，以及真实持续交互中的写入污染和旧事实复活。未裁决的评测口径冲突见 [`conflict.md`](./conflict.md)，缺口与关闭条件见 [`notes/deep-research-question-tree.md`](./notes/deep-research-question-tree.md)。

## Evidence

- **Status**：论文机制、Platform 公开契约、版本锚点为 `Verified`；固定源码路径、客户端行为与评测争议主张为 `Observed`；跨层因果、`92.5/94.4` 的 run provenance 与 Platform 服务端实现为 `Unverified` 或有边界的 `Inferred`。
- **Sources**：`arXiv:2504.19413v1`；固定 Mem0 commit `94c3fe9f…` 的 Python/TypeScript 源码、Dream 与评测文档；评测子模块 `4b61c5d…` / `edcd6f1` 的结果 artifacts。
- **Trace**：Claim 级来源、版本锚点和未闭合项见 [`notes/evidence.md`](./notes/evidence.md)；源码、契约和评测摘录见 [`notes/source.md`](./notes/source.md)。
- **Needs**：Platform 服务端实现、`92.5/94.4` 的完整 manifest，以及同条件评测复现仍未闭合。
