# Mem0 研究缺口（backlog）

> 最后更新：2026-11-06
> 适用范围：已脱离当前研究主线、仍需跨轮维护的对象级缺口
> 维护约定：主线内的子问题与核验动作留在 [`notes/deep-research-question-tree.md`](./notes/deep-research-question-tree.md) §7；已有冲突不在此复制，见 [`conflict.md`](./conflict.md)。同一缺口只在一处维护主要状态。

本文件只承接三类**不影响对象机制解释、但影响外部数字或跨对象比较**的核验任务。它们都已脱离当前主线，且短时间内不会随正文推进而改变，因此从问题树摘要迁出，避免把长期缺口全部留在脚手架里。

## 1. Platform `92.5 / 94.4` 的 run provenance

固定子模块只有 `91.6 / 93.4` artifacts；新版数字缺独立 JSON、run ID、数据 hash 与服务端 revision，无法确认两次运行的条件与口径。

- 关闭条件：Mem0 公开对应的 run manifest 或 artifact。
- 关联：不改变 OSS／Platform 机制解释，只影响总览与评测正文引用该数字时的置信度表述。

## 2. Zep 三组 LoCoMo 数字的对齐

`75.14 / 58.44 / 65.99` 都指向 Zep，但 Category 5 过滤、答案 fallback、计分脚本 commit、`±` 运行次数与逐题 grades 尚未绑定到同一协议。

- 关闭条件：同协议 harness 与逐题输出。
- 关联：跨对象比较问题；已成立的争议见 [`conflict.md`](./conflict.md)，此处只记待补材料。

## 3. LoCoMo 第三方审计的实质结论

已完成数据 hash 与 6 条 score-corrupting 标注的静态抽查；99 项整体分类、judge 宽松性与统计重算未复算。

- 关闭条件：judge 输入输出与统计重算。
- 关联：跨对象评测问题；争议判定仍以 [`conflict.md`](./conflict.md) 为准。

## 复查记录

- 2026-10-30：上述三项的远端固定点均未前进（`memory-benchmarks@4b61c5d`、`locomo-audit@9493fb4`、`zep-papers#5` 无新裁决），继续等待材料（M25）。此条为观察时点记录，不是完成状态。

## 不属于本文件

- 仍属主线的缺口（Platform 服务端实现、OSS 写入污染实验、PyPI 工件闭环、其余 provider／TS 适配器、provider 后台 compaction）：见 [`notes/deep-research-question-tree.md`](./notes/deep-research-question-tree.md) §7。
- 已成立的冲突与口径差异：见 [`conflict.md`](./conflict.md)。
- 候选对象的跟踪状态：见 [`../../candidates.md`](../../candidates.md)。
