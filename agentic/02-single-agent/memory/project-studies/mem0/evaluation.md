# Mem0 评测边界

> 最后更新：2026-10-30
> 版本锚点：固定 Python `v2.2.1`、TypeScript `ts-v3.3.1`，commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`；公开分数归属于 managed Platform

本文回答：LoCoMo、LongMemEval 与公开评测材料能证明 Mem0 的哪些一致性机制，不能证明哪些；要验证“旧事实复活 / 矛盾共存”还需要什么协议。

## 评测观测的是端到端问答，不是机制

两套 benchmark 都给定一段已经构造好的历史，再问系统能否从历史中检索、组合并回答问题。它们能观测最终答案质量，但不能直接观测写入期发生了什么、状态如何标记、读取时过滤了什么。分数差异同时受抽取模型、embedding、检索深度、answerer / judge 和计分口径影响，不能在没有消融的情况下归因到某一个一致性机制。

## 能力矩阵

“可观测”指任务定义本身包含能区分该行为的信号；“部分”指只有间接或弱信号；“不可观测”指任务构造移除了该行为，或没有相应指标。

| 观测点 | LoCoMo | LongMemEval | 受控协议 |
|---|---|---|---|
| 写入期 UPDATE / DELETE 是否发生 | 不可观测 | 不可观测 | 可观测 |
| add-time supersede / merge 是否建链 | 不可观测 | 不可观测 | 可观测 |
| read-time 状态过滤（`latest_only` / `include_merged`） | 不可观测 | 不可观测 | 可观测 |
| read-time 过期过滤（`expiration_date`） | 不可观测 | 不可观测 | 可观测 |
| 检索是否按 recency 排序 | 部分（时序题，不是受控新旧排序） | 部分（temporal-reasoning） | 可观测 |
| knowledge update 的结果（是否用新状态回答） | 部分 | 可观测 | 可观测 |
| 新旧事实是否共存 | 不可观测 | 不可观测 | 可观测 |
| 旧事实复活率 | 不可观测 | 不可观测 | 可观测 |
| 端到端问答质量 | 可观测 | 可观测 | 可观测（成本高） |

## 为什么两套 benchmark 都测不到“旧事实复活”

- LoCoMo 是长、多 session 对话上的离线 QA / summarization / multimodal benchmark，覆盖 single-hop、multi-hop、temporal、open-domain 和 adversarial；历史由 persona 与 temporal event graph 预先构造，不包含“同一可变属性在持续交互中被改写”的写入序列（M17）。
- LongMemEval 明确包含 knowledge-update、temporal-reasoning、multi-session 和 abstention，能测“系统是否使用后来的状态回答”；但其历史构造避免无关冲突，不包含“新旧事实同时存在、旧事实仍可能被召回”的场景（M18）。
- 两者都不暴露 `superseded` / `merged` / `expiration_date` 这类状态或过滤字段，也不记录逐轮写入后的记忆集合变化，因此无法区分“写入期没有失效”“读取期过滤了”“检索恰好没召回”这三种情况。

## 公开数字的归属

- 当前文档声明的 92.5 LoCoMo / 94.4 LongMemEval 属于 managed Platform，含 OSS 不具备的专有优化（M6、M19）。
- 固定评测子模块另有可复核的 Platform 91.6 / 93.4 artifacts；两组数字不是简单四舍五入差异，新版数字缺少完整 run manifest（M19）。
- 固定快照中的 OSS 公开结果只覆盖 LongMemEval，不能据此计算与 Platform 92.5 的 LoCoMo 差值。
- 因此，公开分数既不能直接代表 OSS，也不能在没有消融的情况下证明某个一致性机制有效。

## 受控协议需要什么

最小协议应包含：可控的逐轮写入序列（直接矛盾、重复、时间变化、同实体多值、长会话累积）、固定抽取与 embedding、固定 `top_k` / `threshold`、可选的 `infer=False` 对照，以及 `stale_recall@k`、`current_fact_rank`、`coexistence_rate` 和写入增长等指标。具体设计与判定条件见 [`notes/deep-research-question-tree.md`](./notes/deep-research-question-tree.md) §4.2；当前只读范围只做设计，不执行实验。

## 当前结论

- 高 LoCoMo / LongMemEval 分数不能证明写入期一致性，也不能证明旧事实不会复活。
- 分数差异不能在没有消融的情况下归因到 ADD-only、hybrid retrieval 或 Platform 状态标记。
- 要回答“旧事实复活 / 矛盾共存”，需要连续写入的受控协议；现有公开材料不足。
- 在此之前，一致性结论停在源码观察、公开契约和任务定义层，不升级为端到端效果结论。

## Evidence

- **Status**：LoCoMo / LongMemEval 的任务定义与覆盖为 `Verified`（M17 / M18）；写入期与读取期机制边界为 `Observed`（M2d、M4）；能力矩阵为 `Inferred`（M24，由任务定义与机制边界推出）；Platform 分数归属与 artifacts 为 `Verified` / `Unverified`（M19）。
- **Sources**：`arXiv:2402.17753v1`；`arXiv:2410.10813v2`；固定 commit `94c3fe9f…` 的源码与 Dream 文档；`mem0ai/memory-benchmarks@4b61c5d…` 的 `results/platform/*` 与 README。
- **Trace**：Claim 级来源见 [`notes/evidence.md`](./notes/evidence.md) M17–M19、M24；机制细节见 [`consistency-model.md`](./consistency-model.md) 与 [`retrieval.md`](./retrieval.md)。
- **Needs**：连续写入的受控协议；Platform 92.5 / 94.4 的完整 run manifest；同配置 OSS / Platform 对照。
