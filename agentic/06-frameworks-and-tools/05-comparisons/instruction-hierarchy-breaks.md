# Where Instruction Hierarchy Breaks

> 论文：*Where Instruction Hierarchy Breaks: Diagnosing and Repairing Failures in Reasoning Language Models*
> Authors：Sanjay Kariyappa, G. Edward Suh
> ArXiv：`2606.07808`
> 适用范围：instruction hierarchy、规则分层失效、self-monitoring 修复机制
> Evidence 状态：`Observed`（基于 arXiv 原文 HTML / ar5iv 可观察正文）；对纯文本 skill governance 的迁移判断单独标为 `Inferred`

## 背景知识和术语

这一节只放读后续内容所需的最少背景，不是这篇论文自己的工作。

### Instruction hierarchy 的定义

`instruction hierarchy` 指 LLM 在同一上下文里面对来自不同来源、可能彼此冲突的指令时，应当按预定 privilege ordering 选择哪一条指令生效、并据此生成的层级化约束机制。就这篇论文的直接研究范围看，它主要讨论的是不同消息来源之间的 authority / privilege hierarchy，例如 system prompt、developer policy、user request、conversation history、tool output 等 source 之间谁优先，而不是显式展开单条 prompt 内部的语义层次结构。

这也意味着：system / developer / user 各自当然都可能在一条消息内部再带有更细的规则层次，但这不是原文明确建模和实验化的主对象。若把论文里的三阶段失效框架外推到“单条消息内部也有多层规则”的情形，这更适合视为一种 `Inferred` 用法，而不是论文已经直接证明的适用范围。

`Where Instruction Hierarchy Breaks` 的工作正是在这一机制之上，把 non-compliance（不合规/非遵从）拆成 instruction identification（指令识别）、conflict resolution（冲突解析）、response realization（响应实现）三类。也就是说，它不是只问“最后有没有听话”，而是继续追问：模型究竟是没找到该听的高优先级指令、找到了但层级判断算错，还是层级判断已经做对、最终输出却没有照着实现。

### 核心符号

为了不卡在符号上，先把后面会反复出现的几个符号一次说清：

- `r`：模型自己产出的推理轨迹，也就是它在回答前如何理解规则、如何做层级判断
- `y`：模型最终给用户的输出，也就是最后真正交付出去的回答
- `C`：当前输入上下文，包含模型此刻能看到的全部相关指令与请求
- `A(C)`：论文定义的、在该上下文下“应当生效的指令集合”；可以先粗略理解为把不同层级的指令按优先级筛过一遍后，最后真正该遵守的那批指令

论文定义 non-compliant‌ 说的是：最终输出 `y` 没有遵守 `A(C)`，并把这类违反按“推理轨迹 `r` 第一个坏掉的阶段”拆成三段。

## 1. 研究问题

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

## 2. 三种失效模式

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

## 3. PIM 与 SOM

这一章只先讲读实验所需的最小机制框架：论文提出了两种 training-free 的 self-monitoring 机制，用来在不同插入点修复 instruction hierarchy compliance。

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

### 3.1 PIM：Parallel Input Monitor

论文定义：

> “A parallel input monitor (PIM) checks the shared context for IH conflicts before the main response is produced...”

从正文与 Appendix G 可见，它的关键点是：

- monitor 看的是共享输入上下文
- 主响应与 monitor 并行运行
- 在对齐 / 无害样本下，如果 monitor 不触发，就直接使用预先生成的主响应
- 它主要针对输入侧的 conflict detection

换成更工程一点的话说，`PIM` 更像一个**生成前的冲突扫描器**：它的重点不是“必须由哪个模型来做”，而是把“生成前扫描 instruction hierarchy 冲突”这一步显式拉出来。如果上下文里没有高优先级规则与当前请求打架，就让主响应直接放行；如果有，再进入额外处理路径。

### 3.2 SOM：Sequential Output Monitor

论文定义：

> “A sequential output monitor (SOM) reviews and revises a drafted response..."

它的关键点是：

- 先生成草稿
- 再做顺序式复核 / 修复
- 能覆盖任何在输出中显现出来的上游 failure

换成更工程一点的话说，`SOM` 更像一个**发布前复核器**：它的重点也不是“必须由另一个专门模型来做”，而是把“草稿完成后再显式检查一次最终输出”单独拉出来。先让模型把回答写出来，再问一次“这份回答有没有真正遵守前面的高优先级规则”，必要时再修订。

### 3.3 作为实验配置的使用方式

这一点如果不专门说清，读者很容易误以为论文先有一个更高层的策略器，在不同对话间动态决定“这次该用 PIM 还是 SOM”。

原文更接近下面这套实验逻辑：

- `baseline`、`PIM`、`SOM` 是三条独立的运行配置
- 评测时，同一批样本分别走这几条配置，再与 baseline 对比
- 也就是说，不是“先判别再选 PIM / SOM”，而是“给定一种 defense path，整批样本按这条路径运行”

其中，最值得先抓住的是两者各自的实际使用方式：

```text
PIM: 主响应并行生成 + 输入侧 monitor
     -> 无冲突则直接放行预先生成的主响应
     -> 有冲突再进入额外处理路径

SOM: 先生成草稿
     -> 再做顺序式复核
     -> 如果输出有问题，再修订或重写
```

- `PIM`：主响应与输入侧 monitor 并行运行；如果 monitor 没发现 conflict，就直接使用预先生成的主响应；如果发现 conflict，再进入额外处理路径
- `SOM`：先生成草稿，再做顺序式复核；如果复核认为输出有问题，再修订或重写

这两句几乎就是区分两种方法的最短工作流：`PIM` 是“并行监控，必要时介入”，`SOM` 是“先出草稿，再顺序复核并修订”。

所以它们的“条件性”主要发生在**机制内部**：

- `PIM` 内部决定要不要直接放行预先生成的主响应
- `SOM` 内部决定复核后要不要重写

而不是在机制外部先做一层“场景路由器”，动态选择今天该上哪种 monitor。

## 4. 实验设计

### 4.1 基础任务来源

论文不是凭空构造任务，而是基于已有 instruction hierarchy benchmark 家族：

- `IHEval`
- `IHChallenge`

论文做的关键改造是：把基础样本变成长上下文版本，在前面的相关高优先级 instruction 和后面的冲突 instruction 之间插入不同数量的普通、无害中间对话轮次（benign chat turns）。这样做不是为了制造新的冲突，而是为了在不改变原有 conflict edge 的前提下，故意把两者拉远，从而单独测试：上下文变长后，模型会不会更容易忘记前面的高优先级规则、看漏冲突，或最后执行失败。

这一步很重要，因为它不只是测试“会不会遵守层级要求”，而是在测试：

- 长上下文中是否还能找回相关规则
- 多轮历史中是否还能恢复原有的权限关系
- 分离距离增大时，各类 failure 的分布如何变化
### 4.2 任务类型

从可见正文与搜索片段看，至少包括两大类：

- `Rule following`
- `Safety`

其中 `Rule following` 子集关注输出约束，如：

- 大小写要求（casing）
- 引号要求（quoting）
- 列表项数量（bullet counts）
- 词数要求（word counts）
- 必须 / 禁止出现的 token（required / forbidden tokens）

`Safety` 子集则包括：

- prompt hijack（提示劫持）
- system prompt extraction（系统提示词抽取）
- 与高优先级保护指令冲突的请求

### 4.3 模型设置

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
| 长上下文改造方式 | 插入普通、无害中间对话轮次，拉开相关 instruction 与冲突 instruction 的分离距离 |

### 4.4 整体实验流程

如果把这篇论文当成一个可复现实验，而不只是“读结论”，最小流程可以按下面理解：

```text
IHEval / IHChallenge 基础样本
        ↓
明确高低优先级 instruction 的冲突关系
        ↓
插入普通、无害中间对话轮次，构造长上下文版本
        ↓
模型运行，收集最终输出 y（有 trace 时再收集 r）
        ↓
判断是否合规；对不合规样本按三阶段标注 failure
        ↓
在同一批样本上对比 baseline / Warn / PIM / SOM
        ↓
比较不合规率、failure 分布、monitor 质量与延迟
```

1. 先从 `IHEval` 和 `IHChallenge` 取基础样本。
2. 为每个样本明确冲突关系，也就是哪条高优先级 instruction 应压过哪条低优先级 instruction；从可见正文看，论文至少有一大类已知冲突是 `system -> user`。
3. 把基础样本改造成长上下文版本：在前面的相关 instruction 和后面的冲突 instruction 中间插入不同数量的普通、无害中间对话轮次，控制两者之间的分离距离。
4. 让模型在这些样本上实际运行，收集两类输出：
   - 最终输出 `y`
   - 若模型可见 reasoning trace，则同时收集推理轨迹 `r`
5. 先判断最终输出是否合规；如果 `y` 不合规，再结合 `r` 按三阶段定义把 failure 归到 instruction identification、conflict resolution 或 response realization。
6. 在同一批样本上分别跑不同配置，再比较结果：
   - `baseline`：不加 monitor
   - `Warn`：在冲突 user message 后追加固定 hierarchy reminder
   - `PIM`：输入侧 monitor 路径
   - `SOM`：输出侧 monitor 路径
7. 最后分别比较：
   - 总体不合规率
   - 各类 failure 的构成
   - monitor 质量（如 `TPR` / `UWR`）
   - 延迟权衡

这样看，论文不是只做“加个 monitor 看会不会更好”，而是先建立一套**可诊断的运行与标注流程**，再把 baseline、warning、PIM、SOM 放到同一流程里比较。

## 5. 核心发现

### 5.1 失效模式分解

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

### 5.2 长上下文影响

论文的长上下文改造说明：相关规则和冲突请求被多轮 benign context 隔开后，IH compliance 会恶化，而且恶化不只是一种模式。

对当前仓库更稳的表述是：

- 长上下文不仅影响“能不能遵守规则”
- 还会改变 failure 出现在哪一层

这比只说“长上下文注意力衰减”更贴当前主线，因为它直接落在规则执行链上。

### 5.3 检测能力与默认执行的脱节

论文提出 PIM / SOM 的出发点，是一个很有价值的观察：

- 模型在聚焦提示下，往往能检测 context conflict 或 output violation
- 但在默认单线程生成路径里，这种能力并不会自动转化成稳定的 hierarchy compliance

这对纯文本 skill governance 很重要，因为它提示：

- “知道规则”
- “能在单独检查时说出哪里错了”
- “主线程里稳定按规则执行”

不是同一件事。

## 6. 关键结果

### 6.1 Monitor 改进结果

论文给出的核心结果是：

- 在 rule-following 场景里，`SOM` 相对 baseline 将不合规率降低 `81%–99%`
- 搜索片段还给出：`GPT-5.3` 在静态攻击（static attacks）下的降幅为 `86%`，在自适应攻击（adaptive attacks）下的降幅为 `45%`

| 结果项 | 原文可直接支持的数值 |
| --- | --- |
| Rule-following 场景中的最强 monitor 改善幅度 | 不合规率降低 `81%–99%` |
| GPT-5.3 静态攻击 | 降幅 `86%` |
| GPT-5.3 自适应攻击 | 降幅 `45%` |

下面这张 utility / ASR 表来自论文 appendix 中可见的 monitor 对照结果，适合快速把握“monitor 不是免费修复”的另一面：

| 模型 | Baseline 效用 | Baseline ASR | PIM 效用 | PIM ASR | SOM 效用 | SOM ASR |
| --- | --- | --- | --- | --- | --- | --- |
| Gemma-4-31B-IT | `89.46%` | `3.69%` | `89.99%` | `0.00%` | `89.99%` | `0.00%` |
| GPT-5.3 | `85.25%` | `0.53%` | `84.30%` | `0.00%` | `84.19%` | `0.42%` |
| Claude Sonnet 4.6 | `86.51%` | `0.00%` | `87.25%` | `0.00%` | `86.72%` | `0.00%` |

更保守的写法应是：论文在其所研究的模型与 benchmark 设置中，观察到无需训练改动的 monitor，尤其是 SOM，对 rule-following 场景中的不合规率有大幅改善。

### 6.2 PIM / SOM 的延迟权衡

正文给出的 aligned-control 延迟数据：

- baseline 中位延迟：`46.8s`
- `PIM`：`54.8s`，约 `+17%`
- `SOM`：`69.4s`，约 `+48%`

95 分位延迟也同步上升：

- baseline：`116.5s`
- PIM：`125.9s`
- SOM：`154.5s`

| 方法 | 中位延迟 | 95 分位延迟 | 相对 baseline |
| --- | --- | --- | --- |
| Baseline | `46.8s` | `116.5s` | — |
| PIM | `54.8s` | `125.9s` | 中位延迟 `+17%` |
| SOM | `69.4s` | `154.5s` | 中位延迟 `+48%` |

所以它不是“免费修复”，而是很典型的：

- PIM 用更低延迟换更窄覆盖
- SOM 用更高延迟换更广覆盖

### 6.3 残余 failure 的结构变化

论文还指出，monitor 不只是减少 failure 总量，还会改变剩余 failure 的结构：

- 当 diagnostic traces 可用时，剩余 failures 会从 identification / resolution errors，转向更多 response realization

这说明：把前两层修掉之后，系统会暴露出更后段的 failure 面。

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

## 7. 外推边界

这部分很重要。

### 7.1 不是通用的纯文本 skill procedure 理论

论文研究的是 `instruction hierarchy compliance`，不是通用 skill procedure 设计理论。

所以它能很好支撑：

- rule layering
- conflict precedence
- monitor placement

但不能直接证明：

- 纯文本 skill 的最小字段应该是什么
- 所有 self-check 都有效
- 所有 structured procedure 都会提升遵守率

### 7.2 不是复杂 agent runtime hard boundary 研究

PIM / SOM 虽然是 runtime-intervention flavor 的机制，但它们仍然主要围绕文本 instruction hierarchy 和 response compliance。

所以不应把它直接写成：

- approval policy
- sandbox hard boundary
- tool call gate
- environment isolation

这些复杂 agent 治理的现成证据。

### 7.3 外推受 benchmark、模型与攻击设置限制

论文自己明确写了限制：

- 评估范围只覆盖其研究所用的 benchmark 与模型
- 更强的自适应攻击（adaptive attacks）可能暴露出更多 failure mode
- 不同的 chat template 行为可能改变实验结果
- 部署相关的工具接口可能引入额外 failure
- `SOM` 仍可能放过脆弱的字符串级约束或 safety constraint 问题

所以更稳的回流方式应该是：

- 把它作为高信号的机制论文来使用
- 不把这里观察到的 improvement 幅度直接写成跨系统通用收益

## 8. 结论

这篇论文最重要的贡献，不是笼统地再次强调 instruction hierarchy 重要，而是把这类问题稳定拆成了更可诊断、也更可修复的结构。

它表明：

- instruction hierarchy failure 不是单一现象，而至少可以拆成 instruction identification、conflict resolution、response realization 三个阶段
- self-monitoring 的有效插入点至少可以分成输入侧和输出侧两类
- 长上下文与多轮历史不只会放大问题，还会改变主导 failure 的分布
- 这些结论在论文所研究的 benchmark、模型与攻击设置内成立，但不应直接外推成跨系统通用规律

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