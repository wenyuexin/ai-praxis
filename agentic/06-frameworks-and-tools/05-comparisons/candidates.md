# Comparison Candidates

本文记录 `05-comparisons/` 下一步值得继续跟踪的候选研究对象。这里的“对象”主要是论文、产品机制、官方文档或高信号方法样本；它们本身还不是正文定论，也不替代 `backlog.md` 中的问题缺口。

这份文件当前按**动态研究队列**使用，不是普通研究文档：

- 你应能直接从队列里挑选一个对象，然后开始研究。
- 候选对象一旦进入正式研究，并在本层或对象目录中形成核心文档落位，就应从这里移出，而不是长期滞留。
- 因此，每个对象条目都应尽量就地提供最小研究抓手，而不是把关键入口集中藏在文末。

当前这份队列主要服务两条正在形成的比较主线：

- 纯文本 skill agent 的规则遵守与输出治理
- rules / skills / hooks / approvals / sandbox 如何真正进入执行面

---

## 一、纯文本 skill 治理优先对象

### 1.1 Instruction Hierarchy 的三种失效模式

- **对象类型**：论文
- **当前状态**：已通过一轮外部调研筛选，当前保留为高优先级候选对象
- **为什么值得继续研究**：它为 instruction layering 提供了更细的失效拆解视角，能帮助区分“规则没生效”到底是识别、冲突解析还是响应实现的问题。
- **当前最多可支撑的问题**：纯文本 skill agent 中，规则分层失效是否可以被稳定拆成几个阶段；哪些自监控机制可能把 prompt-level guidance 推向更强流程约束。
- **优先回流位置**：`text-skill-governance.md`、`execution-governance-layers.md`
- **研究入口**：Paper — https://browse-export.arxiv.org/abs/2606.07808
- **下一步补证方向**：核对一手论文版本，补不同模型、不同场景下的适用范围与限制。

### 1.2 Instruction Boosting

- **对象类型**：论文
- **当前状态**：已通过一轮外部调研筛选，当前保留为高优先级候选对象
- **为什么值得继续研究**：它把生成后修订作为提升多指令遵守率的机制，适合补 `self-check / critique loop` 的外部支撑。
- **当前最多可支撑的问题**：生成后修订是否能作为纯文本 skill agent 的稳定补充机制；指令冲突是否是遵守率退化的重要因素。
- **优先回流位置**：`text-skill-governance.md`
- **研究入口**：Paper — https://arxiv.org/html/2510.14842v1
- **下一步补证方向**：核对 benchmark、模型范围与增益边界，避免把提升幅度写成通用收益。

### 1.3 PR-CoT

- **对象类型**：论文
- **当前状态**：已通过一轮外部调研筛选，当前保留为高优先级候选对象
- **为什么值得继续研究**：它把 self-check 从单一反思推进到多维度结构化反思，适合补“结构化自检”这一类治理机制。
- **当前最多可支撑的问题**：多维度、结构化自检是否比单一自我反思更值得作为纯文本治理机制候选。
- **优先回流位置**：`text-skill-governance.md`
- **研究入口**：Paper — https://arxiv.org/html/2601.07780v1
- **下一步补证方向**：补不同任务类型与模型家族上的适用边界。

### 1.4 Self-Refine

- **对象类型**：论文
- **当前状态**：已通过一轮外部调研筛选，当前保留为高优先级候选对象
- **为什么值得继续研究**：它是纯文本自我评判与修订的经典对象，能帮助比较 self-check 究竟提升了什么、失效又发生在哪。
- **当前最多可支撑的问题**：不依赖外部工具时，自我批判与迭代修订是否真能提升执行质量。
- **优先回流位置**：`text-skill-governance.md`
- **研究入口**：Paper — https://arxiv.org/abs/2303.17651
- **下一步补证方向**：补 confirmation bias、错误强化与收益递减曲线的限制。

### 1.5 Constitutional AI

- **对象类型**：论文
- **当前状态**：已通过一轮外部调研筛选，当前保留为高优先级候选对象
- **为什么值得继续研究**：它提供了“自然语言规则集如何持续进入生成过程”的代表样本，适合支撑 rules 作为生成约束而非事后检查的讨论。
- **当前最多可支撑的问题**：规则集何时只是建议，何时已经成为生成过程中的持续约束。
- **优先回流位置**：`text-skill-governance.md`、`execution-governance-layers.md`
- **研究入口**：Paper — https://arxiv.org/abs/2212.08073
- **下一步补证方向**：区分 safety / harmlessness 场景与通用 task execution 的迁移边界。

### 1.6 Lost in the Middle

- **对象类型**：论文
- **当前状态**：已通过一轮外部调研筛选，当前保留为高优先级候选对象
- **为什么值得继续研究**：它是长上下文中规则位置效应与注意力衰减的高信号对象，适合支撑 context shaping 相关段落。
- **当前最多可支撑的问题**：长上下文中规则保持为什么不稳定；规则放置与重复策略为何会影响遵守率。
- **优先回流位置**：`text-skill-governance.md`
- **研究入口**：Paper — https://arxiv.org/abs/2307.03172
- **下一步补证方向**：补不同模型与更长上下文窗口下的变化趋势。

### 1.7 SCOPE

- **对象类型**：论文
- **当前状态**：已通过一轮外部调研筛选，当前保留为高优先级候选对象
- **为什么值得继续研究**：它把 prompt / guideline 的持续演化当作治理机制，适合补“局部目标压过全局规则”与动态 context shaping。
- **当前最多可支撑的问题**：静态 prompt 是否足够；把执行经验回写成持续演化的 guidelines 是否更稳。
- **优先回流位置**：`text-skill-governance.md`
- **研究入口**：Paper — https://browse-export.arxiv.org/pdf/2512.15374
- **下一步补证方向**：补自动演化带来的新不稳定因素与适用边界。

### 1.8 PRISM

- **对象类型**：论文 / 平台实践样本
- **当前状态**：已通过一轮外部调研筛选，当前保留为高优先级候选对象
- **为什么值得继续研究**：它把 prompt engineering 推向持续可靠性工程，适合支撑“科学机制”不止是一次性写 prompt。
- **当前最多可支撑的问题**：纯文本规则遵守是否应通过持续测试、模拟、修复与监控来维持，而不是一次性优化完成。
- **优先回流位置**：`text-skill-governance.md`
- **研究入口**：Paper — https://browse-export.arxiv.org/abs/2605.15665
- **下一步补证方向**：补 LLM-as-Judge 与平台特定设置带来的局限。

---

## 二、执行约束边界样本

### 2.1 OctoBench

- **对象类型**：论文 / benchmark
- **当前状态**：已通过一轮外部调研筛选，当前保留为高优先级候选对象
- **为什么值得继续研究**：它提醒我们“完成任务”和“遵守 scaffold”是两个不同维度，适合作为执行约束研究的旁证对象。
- **当前最多可支撑的问题**：规则遵守能力不能被任务完成率替代；治理机制需要单独评估。
- **优先回流位置**：`execution-governance-layers.md`、`text-skill-governance.md`
- **研究入口**：Paper — https://export.arxiv.org/abs/2601.10343
- **下一步补证方向**：补它对纯文本 agent 的可迁移边界，避免直接把 coding benchmark 结论外推到所有文本场景。

### 2.2 Claude Skills Progressive Disclosure

- **对象类型**：产品机制样本 / 技术解读线索
- **当前状态**：暂列候选，不进入正文主证据
- **为什么值得继续研究**：它可能为“skill 分层加载”和“按需注入”提供对象级样本，但当前拿到的材料仍偏二手。
- **当前最多可支撑的问题**：大量 skill 指令是否可以从一次性全量注入，转向按需加载和渐进披露。
- **优先回流位置**：若补到官方一手材料，可支撑 `text-skill-governance.md`
- **研究入口**：Current lead — https://www.kdnuggets.com/anthropics-complete-guide-to-claude-skills-building
- **下一步补证方向**：优先找 Anthropic 官方一手 skill 文档或产品机制说明，再决定是否升级为正式对象证据。

### 2.3 Structured Outputs / Schema-Constrained Output

- **对象类型**：官方文档 / API 机制样本
- **当前状态**：保留为边界样本，不直接作为纯文本 skill 主证据
- **为什么值得继续研究**：它可以帮助界定“纯文本输出约束”和“API / infra 级格式硬约束”的边界。
- **当前最多可支撑的问题**：什么算 output constraint 的最强形态；从 prompt-level guidance 到系统硬约束的分界在哪里。
- **优先回流位置**：`execution-governance-layers.md`、`text-skill-governance.md` 的边界提醒段落
- **研究入口**：Official doc — https://developers.openai.com/api/docs/guides/structured-outputs
- **下一步补证方向**：不要继续用它回答“纯文本 skill 如何工作”，而应专门研究它为什么已经越过纯文本边界。

---

## 三、当前不建议直接回流正文的材料

以下对象或材料目前更适合继续保留在 `temp/` 或候选队列中，不建议直接写成主线结论：

- 一批来源混写、表述过满、尚未重新核到一手来源的外部调研卡片
- `LangGraph`：更像框架级 flow constraint 样本，已开始越过纯文本边界
- `AutoGen`：更适合后续 multi-agent execution governance，而不是当前纯文本 skill 主线

## Evidence

- Status: `Observed / Inferred`
- Sources:
  - `../../../backlog.md`
  - `./text-skill-governance.md`
  - `./execution-governance-layers.md`
  - 一轮外部调研筛出的候选论文、官方文档与产品机制线索
- Trace: 本文件当前按动态研究队列使用：高优先级对象已把最小研究入口内联到对应条目中，方便直接选题并启动研究；文末 `Evidence` 只保留全局来源状态与队列形成过程，不再承担单条对象入口索引。对象一旦进入正式研究并在本层或对象目录中形成核心文档落位，就应从本队列移出。
- Needs:
  - 为仍未补齐一手入口的对象继续恢复真实来源
  - 补 Anthropic / Claude Skills 的官方一手材料
  - 判断后续是否需要把部分对象升级为独立案例研究入口或主题级对照

*最后更新: 2026-06-18*
