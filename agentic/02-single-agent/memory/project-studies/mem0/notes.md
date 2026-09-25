# Mem0 — 研究笔记（对象研究 · 第一轮一手核对）

> 研究单元：**对象**（`research-artifacts.md` §2.2；"记忆层"这一想法有众多同类对象，故不按混合型处理）。
> 阶段：notes（研究过程 + claim-source 对照），**非正文**；结论压实后按 §3.6 提炼 overview / 版本对象正文。
> 目的：拉一手来源，逐条核对 [`../../candidates.md`](../../candidates.md) 中来自 DeepSeek 综述、默认 `Inferred` 的 Mem0 核验点。

## 本轮来源（Sources / Trace）

- [一手] Mem0 GitHub `github.com/mem0ai/mem0`（README；65.5k star / 7.7k fork；标注 "New Memory Algorithm, April 2026"）。
- [一手] Mem0 论文 arXiv 2504.19413（两阶段 extraction + update 架构，Mem0 / Mem0g）。
- [二手] memo.d.foundation Mem0 breakdown；Zylos research（架构与开放问题）；vectorize / dataaspirant / developersdigest（Mem0 vs Letta vs Zep 对比，2026-07）。
- [二手] Reddit r/LocalLLaMA "Letta vs Mem0"（仅证实"基准之争存在"）。

## Claim-Source 对照

| 核验点（来自 candidates） | 现状 | 依据 |
|---|---|---|
| 定位：框架无关记忆层，`add()` / `search()`，跨会话记住用户 | **Verified** | README + 多篇二手一致 |
| 三层 scope（user / session / agent）+ 向量 + 图（Mem0g）混合后端 | **Observed** | arXiv 2504.19413 / Zylos |
| "在 LOCOMO 击败所有对手" | **需重限定** | README 自述分数来自**托管平台**（明示 OSS 会不同）；Zylos：91% 延迟是对 full-context 基线、非对同类系统 |
| LOCOMO 之争（Letta 质疑 Mem0 未公平跑对手） | **Conflicting** | 争议真实（Reddit / 对比文）；Mem0 已开源 eval 框架回应；**Letta 原始反驳尚未拉到一手** |
| "遗忘仅按时间" | **疑为错 / 版本混淆** | 旧版有 LLM 决策的 ADD/UPDATE/DELETE/NOOP；新版单趟 ADD-only、只累积不覆盖——两版都不是"按时间遗忘" |
| OSS 过度提取无调参 / 无置信度追踪 / Node 同步阻塞 30–40s | **Unverified** | 均出自 DeepSeek 转述的 GitHub issue，本轮未核到原始 issue |

## 关键发现：Mem0 版本敏感（`research-artifacts.md` §3.1.1）

不能压成单一静态总览：

- **旧版**（arXiv 2504.19413）：两阶段 = 抽取 + 更新；更新阶段用 LLM function-call 决定 **ADD / UPDATE / DELETE / NOOP**（含冲突消解）。
- **新版**（README "April 2026"）：单趟 **ADD-only**（一次 LLM 调用、不 UPDATE / DELETE、只累积）+ 实体链接 + 多信号检索（语义 + BM25 + 实体）+ 时间推理。

→ 后续正文按 §3.1.1：目录级 overview（记忆层定位 + 版本关系）+ 带版本标识的对象正文，别写成一篇静态 Mem0。

## 待确认 / 下一步

- 拉 **Letta 对 Mem0 的原始反驳**（一手），核 LOCOMO 之争口径 → 大概率进 memory 的 `conflict.md`。
- 核 OSS 三条缺陷（过度提取 / 置信度 / 同步阻塞）的原始 issue 与适用版本。
- 新版 "只累积不删" 相对旧版 DELETE 的取舍，是否带来新的膨胀 / 污染问题。
- 分类法（CoALA 等）合流回 [`../../memory-taxonomy-conflicts.md`](../../memory-taxonomy-conflicts.md) 前，先核一手（CoALA 原论文）。
