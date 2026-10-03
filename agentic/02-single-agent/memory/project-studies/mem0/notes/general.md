# Mem0 — 研究笔记（对象研究 · 第一轮一手核对）

> 研究单元：**对象**（`research-artifacts.md` §2.2；"记忆层"这一想法有众多同类对象，故不按混合型处理）。
> 阶段：notes（研究过程 + claim-source 对照），**非正文**；结论压实后按 §3.6 提炼 overview / 版本对象正文。
> 目的：拉一手来源，逐条核对 [`../../../candidates.md`](../../../candidates.md) 中来自 DeepSeek 综述、默认 `Inferred` 的 Mem0 核验点。
> 更新说明（2026-10-03）：保留第一轮来源线索，校准后的 Claim 与待办如下；当前权威状态表见 [`evidence.md`](./evidence.md)。写入机制与部署边界见 [`source.md`](./source.md) §1.3.5–§1.3.8：Python `v2.2.1`、TypeScript `ts-v3.3.1`，commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`；各轮 Observed At 见对应小节，Drift Risk `high`。不能把实体索引或 prompt 关联要求当作已实现的矛盾消解；Platform Dream 公开契约不等于已验证的服务端实现。

## 本轮来源（Sources / Trace）

- [一手] Mem0 GitHub `github.com/mem0ai/mem0`（README；65.5k star / 7.7k fork；标注 "New Memory Algorithm, April 2026"）。
- [一手] Mem0 论文 arXiv 2504.19413（两阶段 extraction + update 架构，Mem0 / Mem0g）。
- [二手] memo.d.foundation Mem0 breakdown；Zylos research（架构与开放问题）；vectorize / dataaspirant / developersdigest（Mem0 vs Letta vs Zep 对比，2026-07）。
- [二手] Reddit r/LocalLLaMA "Letta vs Mem0"（仅证实"基准之争存在"）。

## Claim-Source 对照

| 核验点（来自 candidates） | 现状 | 依据 |
|---|---|---|
| 定位：框架无关记忆层，`add()` / `search()`，跨会话记住用户 | **Verified** | README + 多篇二手一致 |
| 论文 / v2 语境的 scope 与向量 / 图（Mem0g）方案 | **Observed** | arXiv 2504.19413 / Zylos；不外推当前 OSS，图能力迁移边界见 evidence.md M5 |
| README 分数来自托管平台；论文 91% 延迟优势的比较对象为 full-context | **Verified** | evidence.md M6 / M7；不支持“在 LOCOMO 击败所有对手”的无条件概括 |
| LOCOMO 之争（Zep 重算与 Letta baseline 可复现性） | **Observed** | 已读 Zep issue / 原博客与 Letta 原博客；只确认双方主张，未做同配置复现，见 M20 |
| "遗忘仅按时间" | **Deprecated** | 该概括未区分版本、自动抽取、显式 CRUD 与衰减；不以“所有版本均无时间遗忘”替代它，见 M8 |
| 抽取质量、grounding 与置信度问题的社区报告 | **Observed** | #4573 / Discussion #4289；见 M10 / M11，质量数字须保留 gemma2:2b 等报告条件，不是所有部署结论 |
| Node / TS 同步阻塞 30–40s | **Unverified** | M12 仍未找到对应一手 issue；不能当作已确认缺陷 |

## 关键发现：Mem0 版本敏感（`research-artifacts.md` §3.1.1）

不能压成单一静态总览：

- **旧版**：论文全文确认正式管线为 extraction + update；更新阶段由 LLM 在 **ADD / UPDATE / DELETE / NOOP** 四值中决策，并用 DELETE 处理被新信息矛盾的基础记忆；Mem0g 则将过时关系标为 invalid。以上是论文旧版设计，不外推当前 OSS 或 Platform 服务端。
- **新版 OSS**（固定 Python `v2.2.1` 与同 commit TypeScript `ts-v3.3.1`）：普通自动抽取单趟 **ADD-only**（一次 LLM 调用，不执行 UPDATE / DELETE）+ 候选局部 hash 去重 + 实体索引更新 + 多信号检索；Python 另有 `infer=False` 与 procedural 直接追加入口，TypeScript 未观察到 procedural 入口。
- **Platform Dream**：公开契约称 supersede / merge 在新增记忆时评估，旧记录保留状态；读取时可用 `latest_only` / `include_merged` 调整可见性。Synthesis 是后台定时综合。Temporal Reasoning / Memory Decay 仍不能外推到 OSS commit。
- **评测边界**：LOCOMO 覆盖长、多 session 历史上的离线问答与时序/多跳回忆；LongMemEval 更直接包含 knowledge-update，但两者都不能单独证明持续写入后的矛盾版本管理。当前文档的 92.5 / 94.4 属于 managed Platform；固定 benchmark 子模块另有可复核的 Platform 91.6 / 93.4 artifacts，二者不能当作同一轮结果（M17–M19）。

→ 后续正文按 §3.1.1：目录级 overview（记忆层定位 + 版本关系）+ 带版本标识的对象正文，别写成一篇静态 Mem0。

## 待确认 / 下一步

- Zep 与 Letta 一手主张已读；仍需逐项核对实现/计分细节与同配置可复现性，再判断是否需要 `memory/conflict.md`。
- 已有 #4573 / #4289 的报告线索；继续核适用版本与前提。Node / TS “同步阻塞 30–40s”未找到一手支持，暂只保留为待溯源线索。
- 普通自动抽取的 ADD-only 相对旧版更新阶段的取舍，是否带来新的膨胀 / 污染问题；不外推到显式删除 API。
- `dial481/locomo-audit@9493fb4…` 的固定快照已确认含逐题标注、数据 SHA 声明、校验/重算脚本与统计报告；它是评测协议争议入口。156 项问题、其中 99 项 score-corrupting 与 57 项 citation-only 是审计者分类，逐题判断、judge 宽松性和对具体系统分数的影响仍未独立抽查或复算，见 `evidence.md` M21。
- benchmark provenance 仍不完整：固定子模块可复核 91.6 / 93.4 Platform artifacts，当前文档声明 92.5 / 94.4 的对应 run manifest、数据 hash、服务端 revision 与完整模型配置未公开闭合。
- 分类法（CoALA 等）合流回 [`../../../memory-taxonomy-conflicts.md`](../../../memory-taxonomy-conflicts.md) 前，先核一手（CoALA 原论文）。
