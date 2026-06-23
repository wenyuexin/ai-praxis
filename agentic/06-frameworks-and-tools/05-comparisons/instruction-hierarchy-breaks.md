# Where Instruction Hierarchy Breaks

> 论文：*Where Instruction Hierarchy Breaks: Diagnosing and Repairing Failures in Reasoning Language Models*
> Authors：Sanjay Kariyappa, G. Edward Suh
> ArXiv：`2606.07808`
> 适用范围：instruction hierarchy、规则分层失效、self-monitoring 修复机制
> Evidence 状态：`Observed`（基于 arXiv 原文 HTML / ar5iv 可观察正文）；对纯文本 skill governance 的迁移判断单独标为 `Inferred`

## 背景知识和术语

这一节只放读后续内容所需的最少背景，不是这篇论文自己的工作。

### 什么是 instruction hierarchy

`instruction hierarchy` 指 LLM 在同一输入中面对来自不同来源、可能彼此冲突的指令时，应当按预定优先级选择哪一条指令生效、并据此生成的层级化约束机制。常见分层包括 system / developer / user 三级；高优先级指令被违反时，模型应优先遵守更高层级，必要时拒绝或改写更低层级请求。

`Where Instruction Hierarchy Breaks` 的工作正是在这一机制之上，把 non-compliance（不合规/非遵从）拆成 instruction identification（指令识别）、conflict resolution（冲突解析）、response realization（响应实现）三类。也就是说，它不是只问“最后有没有听话”，而是继续追问：模型究竟是没找到该听的高优先级指令、找到了但层级判断算错，还是层级判断已经做对、最终输出却没有照着实现。

### 论文里的符号

为了不卡在符号上，先把后面会反复出现的几个符号一次说清：

- `r`：模型自己产出的推理轨迹，也就是它在回答前如何理解规则、如何做层级判断
- `y`：模型最终给用户的输出，也就是最后真正交付出去的回答
- `C`：当前输入上下文，包含模型此刻能看到的全部相关指令与请求
- `A(C)`：论文定义的、在该上下文下“应当生效的指令集合”；可以先粗略理解为把不同层级的指令按优先级筛过一遍后，最后真正该遵守的那批指令

论文定义 non-compliant‌ 说的是：最终输出 `y` 没有遵守 `A(C)`，并把这类违反按“推理轨迹 `r` 第一个坏掉的阶段”拆成三段。

## 1. 这篇论文到底回答什么问题

这篇论文不只是问“模型最后有没有遵守 instruction hierarchy”，而是更细地问：

- 当最终输出不合规时，问题到底卡在了哪一层
- 是没有识别相关高优先级指令
- 还是识别了但冲突解析错了
- 还是推理里已经做对层级判断，但最终输出仍然违背了该判断

它把 instruction hierarchy 失效从单一合规率，拆成一个更可诊断的白盒过程。

论文摘要与正文最核心的贡献可以压成三点：

1. 提出一个三阶段失效分类，把 instruction hierarchy 失效拆成 instruction identification、conflict resolution、response realization。
2. 在长上下文版本的 IHEval 与 IHChallenge 上比较不同模型的失效分布，而不是只看最终是否合规。
3. 提出两种无需训练改动的 self-monitoring 机制：`PIM`（parallel input monitor）和 `SOM`（sequential output monitor），用于提高 IH compliance。

## 2. 论文里的三种失效模式

论文明确说这三类 failure 是**互斥的**，并按“第一个坏掉的阶段”分类。

| Failure mode | 用论文符号表达 | 白话理解 | 对应坏掉的阶段 |
| --- | --- | --- | --- |
| Instruction identification failure | 推理轨迹 `r` 把本该属于 `A(C)` 的相关指令“omits or misstates” | 该听哪条规则都还没找对，或者把规则内容说错了 | 规则识别 |
| Conflict resolution failure | `r` 已识别 relevant instructions，但最终“computes the wrong active instruction set” | 相关规则都看见了，但谁优先、谁该生效算错了 | 层级/冲突解析 |
| Response realization failure | `r` 已识别 relevant instructions 且算对 active set，但最终输出 `y` 仍 non-compliant | 脑子里已经判断对了，最后说出来或做出来时还是违背了规则 | 最终响应实现 |

### 2.1 Instruction identification failure

论文定义：

> “An instruction identification failure occurs when `y` is non-compliant and `r` omits or misstates an instruction that belongs to the ideal active set `A(C)`.”

也就是说，模型最终没遵守 hierarchy，不是因为它权衡错了，而是因为它在推理轨迹里就没把真正相关的高优先级 instruction 找出来，或者把它说错了。

对当前仓库最重要的一点是：这类 failure 发生在规则识别阶段，比“模型没听话”更早。

### 2.2 Conflict resolution failure

论文定义：

> “A conflict resolution failure occurs when `y` is non-compliant and `r` identifies the active-set-relevant instructions, but computes the wrong active instruction set.”

正文进一步解释，这通常包括：

- 没识别出会改变 `A(C)` 的冲突
- 错认 instruction 的来源或 privilege level
- 应用了错误的 precedence relation

换句话说，规则都看见了，但 hierarchy 决策错了。

### 2.3 Response realization failure

论文定义：

> “A response realization failure occurs when `y` is non-compliant even though `r` identifies the relevant instructions and computes the correct active instruction set.”

也就是说，推理轨迹里已经做出了正确的层级判断，但最终输出或 tool call 仍然没有实现这个决定。

这一步对当前主线尤其关键，因为它说明：即使规则识别和冲突解析都对，最终生成阶段仍可能把对的决定实现错。

## 3. 它的实验到底怎么做

### 3.1 基础任务来源

论文不是凭空构造任务，而是基于已有 instruction hierarchy benchmark 家族：

- `IHEval`
- `IHChallenge`

论文做的关键改造是：把它们变成长上下文版本，通过插入不同数量的 benign chat turns，控制相关 instruction 与冲突 instruction 之间的分离距离。

这一步很重要，因为它不只是测试“会不会 obey hierarchy”，而是在测试：

- 长上下文中是否还能找回相关规则
- 多轮历史中是否还能恢复 privilege relation
- separation length 增大时，failure mode 分布如何变化
### 3.2 任务类型

从可见正文与搜索片段看，至少包括两大类：

- `Rule following`
- `Safety`

其中 `Rule following` 子集关注输出约束，如：

- casing
- quoting
- bullet counts
- word counts
- required / forbidden tokens

`Safety` 子集则包括：

- prompt hijack
- system prompt extraction
- 与高优先级保护指令冲突的请求

### 3.3 模型设置

论文在失效诊断主实验中评估了三类模型：

- `Gemma-4-31B-IT`
- `Qwen3.6-35B-A3B`
- `Claude Sonnet 4.6`

在 self-monitoring intervention 实验中，还额外报告了：

- `GPT-5.3`

| 维度 | 论文中的具体对象 |
| --- | --- |
| 失效诊断主实验模型 | `Gemma-4-31B-IT`、`Qwen3.6-35B-A3B`、`Claude Sonnet 4.6` |
| Monitor 干预实验模型 | `Gemma-4-31B-IT`、`Claude Sonnet 4.6`、`GPT-5.3` |
| 基础 benchmark family | `IHEval`、`IHChallenge` |
| 主要任务族 | `Rule following`、`Safety` |
| 长上下文改造方式 | 插入 benign chat turns，拉开 relevant instruction 与冲突 instruction 的 separation length |

## 4. 这篇论文真正证明了什么

### 4.1 失效不是单一现象

这是最核心、也最适合回流到当前主线的结论。

论文明确表明：non-compliant response 不能笼统理解成一个单一失效桶，而应至少拆成：

- 规则识别失败
- 冲突解析失败
- 响应实现失败

而且主导失效模式会随着：

- 模型
- 任务族
- 上下文长度

发生变化。

这意味着：如果一个系统只盯最终合规率，往往会错过真正该修哪一层。

### 4.2 长上下文会把 instruction hierarchy 问题放大

论文的长上下文改造说明：相关规则和冲突请求被多轮 benign context 隔开后，IH compliance 会恶化，而且恶化不只是一种模式。

对当前仓库更稳的表述是：

- 长上下文不仅影响“能不能遵守规则”
- 还会改变 failure 出现在哪一层

这比只说“长上下文注意力衰减”更贴当前主线，因为它直接落在规则执行链上。
### 4.3 模型常常“能看出来”，但默认不会稳定执行

论文提出 PIM / SOM 的出发点，是一个很有价值的观察：

- 模型在聚焦提示下，往往能检测 context conflict 或 output violation
- 但在默认单线程生成路径里，这种能力并不会自动转化成稳定的 hierarchy compliance

这对纯文本 skill governance 很重要，因为它提示：

- “知道规则”
- “能在单独检查时说出哪里错了”
- “主线程里稳定按规则执行”

不是同一件事。
## 5. PIM 和 SOM 到底是什么

这一章最值得当作“解决方案”来读的，不是它们的名字本身，而是它们分别对应了**两种不同的修复插入点**：

- `PIM`：在主响应生成前，显式插入一次额外的输入侧指令冲突检查
- `SOM`：在草稿生成后，显式插入一次额外的输出侧复核与修订

这里两个 `monitor` 的共同点是：都不是“顺便想一下”，而是把检查动作单独拉出来，做成一条额外的处理路径。

它们的区别在于：

- `PIM` 里的 `monitor` 更像**生成前的监控动作**，重点是“先检查当前上下文里有没有高优先级规则冲突，再决定主响应如何继续”
- `SOM` 里的 `monitor` 更像**生成后的复核动作**，重点是“先看草稿已经写成什么样，再检查最终输出是否真的把前面的层级判断实现出来”

所以这篇论文给出的不是一个抽象口号式“再检查一次”，而是两个可以直接比较的治理方案：**前置监控** 和 **后置复核**。

| 方案 | `monitor` 在这里更接近什么 | 插入点 | 主要检查对象 | 更擅长修什么 | 主要代价 |
| --- | --- | --- | --- | --- | --- |
| `PIM` | 生成前的额外监控动作 | 主响应生成前 | 输入上下文中的 hierarchy 冲突 | 规则识别失败、部分冲突解析失败 | 覆盖范围较窄 |
| `SOM` | 生成后的额外复核动作 | 草稿生成后 | 最终输出是否违背前面的层级判断 | 响应实现失败，也能兜住部分上游失败 | 顺序延迟更高 |

### 5.1 PIM：Parallel Input Monitor

论文定义：

> “A parallel input monitor (PIM) checks the shared context for IH conflicts before the main response is produced...”

从正文与 Appendix G 可见，它的关键点是：

- monitor 看的是共享输入上下文
- 主响应与 monitor 并行运行
- 对齐 / 无害 case 下，如果 monitor 不触发，就直接使用 speculative main response
- 它主要针对输入侧的 conflict detection

换成更工程一点的话说，`PIM` 更像一个**生成前的冲突扫描器**：它的重点不是“必须由哪个模型来做”，而是把“生成前扫描 instruction hierarchy 冲突”这一步显式拉出来。如果上下文里没有高优先级规则与当前请求打架，就让主响应直接放行；如果有，再进入额外处理路径。

优点：

- 延迟开销低
- 更像生成前的 conflict scan
- 对“先把不该做的事拦在生成前”这类问题有直接参考价值

限制：

- 只能看输入侧
- 抓不到只在最终响应中暴露的 realization failure
- 如果问题不是“没看见冲突”，而是“最后没照着实现”，它帮不上太多

### 5.2 SOM：Sequential Output Monitor

论文定义：

> “A sequential output monitor (SOM) reviews and revises a drafted response..."

它的关键点是：

- 先生成草稿
- 再做 sequential review / repair
- 能覆盖任何在输出中显现出来的上游 failure

换成更工程一点的话说，`SOM` 更像一个**发布前复核器**：它的重点也不是“必须由另一个专门模型来做”，而是把“草稿完成后再显式检查一次最终输出”单独拉出来。先让模型把回答写出来，再问一次“这份回答有没有真正遵守前面的高优先级规则”，必要时再修订。

优点：

- 覆盖范围更广
- 特别适合抓 response realization failure
- 对“模型脑子里可能想对了，但最后交付错了”的问题很有参考价值

限制：

- 有额外顺序延迟
- 仍可能接受 monitor 自己认为没问题、但实际仍违背 brittle string-level 或 safety constraint 的输出
- 它更像兜底复核，不等于前面识别/解析阶段就已经稳定无误

### 5.3 作为解决方案时该怎么理解

如果把这章当成可复用的解决思路，而不是只当论文细节，那么最重要的 takeaway 是：

- 不是所有“规则不遵守”都该用同一种补丁
- 如果问题主要出在**没看见规则**或**没看见冲突**，更该优先考虑像 `PIM` 这种前置监控
- 如果问题主要出在**最后输出没照着执行**，更该优先考虑像 `SOM` 这种后置复核
- 如果还不知道失败发生在哪一层，就不要急着先选方案，而应先做 failure diagnosis

这也是为什么这篇论文的参考价值不只在于“提出了两个 monitor”，而在于它把**诊断**和**修复插入点**连起来了。
## 6. 关键结果

### 6.1 Monitor 的核心改进结果

论文给出的核心结果是：

- 在 rule-following 场景里，`SOM` 相对 baseline 将 non-compliance 降低 `81%–99%`
- 搜索片段还给出：`GPT-5.3` 在 static attacks 下 reduction 为 `86%`，在 adaptive attacks 下 reduction 为 `45%`

| 结果项 | 原文可直接支持的数值 |
| --- | --- |
| Strongest monitor on rule-following non-compliance | `81%–99%` reduction |
| GPT-5.3 static attacks | `86%` reduction |
| GPT-5.3 adaptive attacks | `45%` reduction |

下面这张 utility / ASR 表来自 paper appendix 中可见的 monitor 对照结果，适合快速把握“monitor 不是免费修复”的另一面：

| Model | Baseline Util. | Baseline ASR | PIM Util. | PIM ASR | SOM Util. | SOM ASR |
| --- | --- | --- | --- | --- | --- | --- |
| Gemma-4-31B-IT | `89.46%` | `3.69%` | `89.99%` | `0.00%` | `89.99%` | `0.00%` |
| GPT-5.3 | `85.25%` | `0.53%` | `84.30%` | `0.00%` | `84.19%` | `0.42%` |
| Claude Sonnet 4.6 | `86.51%` | `0.00%` | `87.25%` | `0.00%` | `86.72%` | `0.00%` |

更保守的写法应是：论文在其所研究的模型与 benchmark 设置中，观察到无需训练改动的 monitor，尤其是 SOM，对 rule-following non-compliance 有大幅改善。

### 6.2 PIM / SOM 的延迟权衡

正文给出的 aligned-control latency 数据：

- baseline median latency：`46.8s`
- `PIM`：`54.8s`，约 `+17%`
- `SOM`：`69.4s`，约 `+48%`

95th percentile 也同步上升：

- baseline：`116.5s`
- PIM：`125.9s`
- SOM：`154.5s`

| Method | Median latency | 95th percentile latency | 相对 baseline |
| --- | --- | --- | --- |
| Baseline | `46.8s` | `116.5s` | — |
| PIM | `54.8s` | `125.9s` | median `+17%` |
| SOM | `69.4s` | `154.5s` | median `+48%` |

所以它不是“免费修复”，而是很典型的：

- PIM 用更低延迟换更窄覆盖
- SOM 用更高延迟换更广覆盖

### 6.3 残余 failure 的结构变化

论文还指出，monitor 不只是减少 failures 总量，还会改变 residual failure composition：

- 当 diagnostic traces 可用时，剩余 failures 会从 identification / resolution errors，转向更多 response realization

这说明：把前两层修掉之后，系统会暴露出更后段的 failure surface。

补一张来自 appendix 的 monitor 质量表，有助于把 PIM / SOM 的“抓得准”和“误杀少”分开看：

| Model | Split | PIM TPR | PIM UWR | SOM TPR | SOM UWR |
| --- | --- | --- | --- | --- | --- |
| Gemma-4-31B-IT | Rule following | `539/541 (99.6%)` | `1/541 (0.2%)` | `527/540 (97.6%)` | `0/541 (0.0%)` |
| Claude Sonnet 4.6 | Rule following | `531/541 (98.2%)` | `66/541 (12.2%)` | `480/541 (88.7%)` | `75/541 (13.9%)` |
| GPT-5.3 | Rule following | `538/541 (99.4%)` | `13/541 (2.4%)` | `523/541 (96.7%)` | `0/541 (0.0%)` |
| Gemma-4-31B-IT | Safety | `1787/1860 (96.1%)` | `2/1590 (0.1%)` | `1748/1860 (94.0%)` | `0/1590 (0.0%)` |
| Claude Sonnet 4.6 | Safety | `1475/1860 (79.3%)` | `7/1590 (0.4%)` | `1368/1860 (73.5%)` | `330/1590 (20.8%)` |
| GPT-5.3 | Safety | `1730/1860 (93.0%)` | `2/1590 (0.1%)` | `1691/1860 (90.9%)` | `0/1590 (0.0%)` |

其中：

- `TPR` 可理解为 true positive rate
- `UWR` 是 appendix 中直接给出的缩写，这里先按原文保留，不在笔记里额外改写定义
## 7. 这篇论文不能被过度外推成什么

这部分很重要。

### 7.1 它不是通用的纯文本 skill procedure 理论

论文研究的是 `instruction hierarchy compliance`，不是通用 skill procedure 设计理论。

所以它能很好支撑：

- rule layering
- conflict precedence
- monitor placement

但不能直接证明：

- 纯文本 skill 的最小字段应该是什么
- 所有 self-check 都有效
- 所有 structured procedure 都会提升遵守率

### 7.2 它不是复杂 agent runtime hard boundary 研究

PIM / SOM 虽然是 runtime-intervention flavor 的机制，但它们仍然主要围绕文本 instruction hierarchy 和 response compliance。

所以不应把它直接写成：

- approval policy
- sandbox hard boundary
- tool call gate
- environment isolation

这些复杂 agent 治理的现成证据。

### 7.3 它的外推受 benchmark / model / attack setting 限制

论文自己明确写了限制：

- evaluation limited to the studied benchmarks and models
- stronger adaptive attacks may expose more failure modes
- different chat-template behavior may change outcomes
- deployment-specific tool interfaces may introduce additional failures
- SOM 仍可能放过 brittle string-level 或 safety constraint 问题

所以更稳的回流方式应该是：

- 把它作为高信号机制论文
- 不把其 improvement 幅度写成跨系统通用收益

## 8. 结论

对当前仓库主线来说，这篇论文最重要的价值不是“又证明了 instruction hierarchy 重要”，而是把纯文本规则治理问题进一步切开了。

它告诉我们：

- 规则不生效，未必是单一 failure
- 规则识别、层级决策、最终实现要分开看
- self-monitoring 可以按输入侧 / 输出侧两种插入点设计
- 当规则进入长上下文和多轮对话时，治理问题会明显复杂化
因此，它非常适合作为 `text-skill-governance.md` 的第一批正式对象级支撑材料。

## Evidence

- Status: `Observed` / `Inferred`
- Sources:
  - `https://arxiv.org/abs/2606.07808`
  - `https://arxiv.org/html/2606.07808v1`
  - `https://ar5iv.labs.arxiv.org/html/2606.07808`
- Trace: 从 `05-comparisons/candidates.md` 中“Instruction Hierarchy 的三种失效模式”候选对象出发，按原始论文正文精读，抽取失效分类、benchmark 设计、PIM / SOM 机制、核心结果和限制，再回流到 `text-skill-governance.md` 的 rule-layering / self-monitoring / failure pattern 主线。
- Needs:
  - 如后续需要更强可复核性，可继续补 PDF 中表格与 Appendix H monitor prompts 的更细摘录
  - 继续核验论文中的 `Qwen3.6-35B-A3B` 与 `GPT-5.3` 更完整结果分布，避免只保留 headline reductions
  - 后续可与 `Instruction Boosting`、`PR-CoT` 对照，区分“层级失败诊断”与“后验修订增益”两类机制