# 能力前置路由如何改变落位判断：`Navigate` / `Place` / `Ingest` 的模拟测试案例

本案例用于说明：**当 AI 面对“这个对象/主题/材料该放在哪里”这类问题时，为什么不能只依赖规则层与现有目录直觉，还应先显式经过 capability layer 的问题分型；以及这种前置路由如何改善目录定位、层级判断与材料吸收的执行质量。**

## 1. 这个案例回答什么问题

它回答的不是某个具体对象本身的知识结论，而是下面这些跨层判断：

- 为什么 AI 即使最终给出一个大体正确的落位，也可能仍然没有按仓库规定的方式工作
- 为什么“先读规则再看目录”不等于已经正确使用了 `Navigate`、`Place`、`Ingest`
- capability layer 在真实落位题里到底提升了什么，而不只是多一层解释文档
- 什么情况下一个案例已经不只是规则复盘，而同时说明 capability 使用方式与入口设计问题

## 2. 案例背景

这次问题不是从抽象规则讨论开始，而是从一次真实偏差暴露出来的。

在最初回答“`glm5.2` 模型的公开报告和博客应该放在哪里”时，AI 的实际动作是：

1. 先读 `AGENTS.md` 与 `CONTRIBUTING.md`
2. 再读 `docs/contributing/research-artifacts.md` 与 `docs/contributing/metadata-files.md`
3. 然后查看目标主题目录的 `README.md` / `index.md`
4. 最后给出落位建议

这个过程在结果上并没有明显跑偏，但它跳过了 capability layer 的显式使用：

- 没有先把问题识别成 `Navigate` 题还是 `Place` 题
- 没有先走 `docs/capabilities/index.md` 或对应能力文档
- 没有把“找到主入口”和“决定应停在哪层”分成两个动作

也就是说，AI 做了类似 `Navigate + Place` 的事情，但没有**按 capability layer 的方式**来做。

## 3. 暴露出的核心问题

这次偏差最关键的地方，不是 AI 完全不会落位，而是：

- 它把 capability layer 误当成“解释层”，而不是“执行前工作流”
- 它默认把这类问题压缩成“规则层 + 目录现状”即可完成的任务
- 它能在事后回顾时说出自己其实做了 `Navigate` / `Place`，却不会在执行当下先触发这条能力链

这说明问题不在“能力文档有没有价值”，而在“能力入口是否足够前置、足够强触发”。

## 4. 为什么这不是单纯的执行失误

如果只是一次性答错目录，问题更像局部判断失误。

但这次更值得沉淀，是因为它呈现出一种稳定复发风险：

- 规则层已经很强，AI 很容易直接走最短路径
- capability layer 只要写得像补充阅读，就会在执行前被跳过
- 一旦 capability layer 不前置，AI 就容易把“定位”“落位”“材料吸收”重新揉成一步

所以这里不是“某个对象放错了”，而是：**仓库已经显式定义了能力层，但入口还不足以稳定让 AI 在操作前激活它。**

## 5. 本案例中的三轮模拟测试

为验证 capability layer 是否真能提升执行质量，而不只是增加术语，后续围绕三类问题做了连续模拟测试。

### 5.1 测试一：具体对象目录定位

问题：`接下来研究 https://github.com/Zleap-AI/Zleap-Agent，放在哪里合适`

更稳的起手不是直接看规则层，而是：

- 先识别这是 `Navigate + Place` 问题
- 先判断它属于哪片主题树
- 再判断它是框架、coding tool，还是完整项目案例

在这一轮里，AI 先通过 `Navigate` 把问题收敛到 `agentic/`，再通过 `Place` 区分：

- `06-frameworks-and-tools/01-frameworks/`
- `06-frameworks-and-tools/02-coding-agents-and-tools/`
- `06-frameworks-and-tools/03-project-studies/`

最后收敛到更稳的主归属：完整开源 agent 项目案例研究。

### 5.2 测试二：主题级落位判断

问题：`我想研究 llm agent 的 skill 进化，放在哪里`

这题如果不先做能力分型，很容易直接按关键词把它扔进 `02-single-agent/`，因为表面上 skill 看起来像单智能体能力模块。

但经过 `Navigate + Place` 后，判断重点变成：

- 研究的是 skill 作为单体能力机制，还是 skill 作为工程化能力单元/系统
- 主问题是能力原理，还是 skill / tool system 的演进与治理

因此更稳的主落位不再是能力闭环目录，而是 `agentic/06-frameworks-and-tools/04-skill-and-tool-systems/`。

### 5.3 测试三：材料吸收与目录判断的串联

问题不是单独再问一次“放哪”，而是继续追问：如果后续真的开始研究这个主题/对象，接下来该做什么。

这里进一步暴露出一个关键点：

- `Navigate` 负责找到主入口
- `Place` 负责判断停在哪层
- `Ingest` 负责在真正拿到 README、博客、issue、第三方材料后，判断哪些能进入正文、哪些应先进入中间层

也就是说，能力链不是彼此替代，而是天然串联。

### 5.4 这几轮测试的验收边界

这几轮测试已经不是沿用旧入口的模拟，而是**在入口修改完成后，立即按新治理层复测**：AI 已开始在起手阶段显式区分 `Navigate`、`Place` 与 `Ingest`，而不是只在事后解释自己其实做了这些动作。

但这仍属于**同一协作者、同一长上下文中的即时复测**，还不等于冷启动验收。

它能证明的是：

- 新入口已经足够显眼，能改变当前会话里的默认起手式
- capability layer 已经从“补充阅读”前移成了可被即时调用的工作流

它暂时还不能独立证明的是：

- 在没有当前长上下文提醒时，协作者仍会稳定先走 `Navigate` → `Place` → `Ingest`
- 其他协作者或后续冷启动会话也会自动被同样触发

因此，这几轮测试更准确的性质应理解为：**入口修改后的第一轮即时复测，而不是最终冷启动验收。**

## 6. 这些测试说明 capability layer 到底提升了什么

本案例最重要的结论不是“AI 会说出 Navigate / Place / Ingest 这些词了”，而是 capability layer 在真实落位题里确实提升了三个方面。

### 6.1 把“找到位置”和“决定怎么放”拆开

没有 capability layer 时，AI 很容易：

- 刚看到一个相关目录就直接下结论
- 还没确认主入口就开始讨论该建什么文件
- 把目录定位、文件类型与材料状态混成一团

而 `Navigate` / `Place` 的前置触发，会强制先回答：

- 这是不是这里
- 如果是这里，下一层去哪
- 到这一层后，是正文、元信息文件，还是还不该进入正文

### 6.2 提高“停在何处”的稳定性

`Place` 的价值不只是选文件名，而是先判断：

- 当前最高稳定归属在哪层
- 下沉一级是否也已经稳定

这会直接降低一种很常见的误判：**为了显得具体而过早下沉到细目录。**

### 6.3 给材料吸收留出中间层

当问题从“目录放哪”转向“这篇博客今天怎么进入系统”时，`Ingest` 会阻止 AI 直接把外部材料润色进正文。

这能改善的不是目录美观，而是 Evidence 边界。

## 7. 这个案例最容易诱发的三类误判

### 误判 A：只要最终目录没错，就算已经正确使用了 capability layer

不是。

这次偏差最初就说明：结果大体正确，不等于过程已经对齐仓库的能力工作流。

### 误判 B：规则层够强，所以 capability layer 可以只作为解释材料存在

不是。

规则层当然能约束执行，但如果 capability layer 不前置，AI 在执行当下仍会优先走最短路径，把“定位”“落位”“吸收”重新压成一步。

### 误判 C：只要在事后能解释自己其实做了 `Navigate` / `Place`，就说明入口已经没问题

也不是。

真正需要的是：在**执行前**就被触发，而不是靠事后反思补回能力命名。

## 8. 本案例最终带来的治理动作

这次案例没有直接去重写 `navigate.md` / `place.md` / `ingest.md` 的正文，而是优先修改了入口层：

- `AGENTS.md`
- `docs/capabilities/README.md`
- `docs/capabilities/index.md`

目标不是增加更多解释，而是把 capability layer 从“补充阅读”前移成“执行前默认路由”。

新增后的最小工作流是：

```text
问题 / 新材料
  → Navigate
  → Place
  → Ingest
  → Contributing rules
```

这一步本身就说明：该案例不只是在复盘某次落位失误，而是在推动 capability layer 的入口治理。

## 9. Cross-layer signal

本案例已经不只是单一规则复盘，而同时说明了三层问题：

- **Capability**：`Navigate` / `Place` / `Ingest` 在真实场景中如何前置触发
- **Contributing**：目录定位、元信息文件分工与材料回流仍需回到规则层执行
- **Design**：为什么当前更应该强化入口触发，而不是急着新增 `docs/capabilities/cases/`

因此，它属于典型的 **cross-layer case**。

但按当前仓库设计，cross-layer case 仍应先暂放在 `docs/contributing/cases/`，并在案例内显式标出这一点；长期升级判断见 [`../../design/cases-layer-design.md`](../../design/cases-layer-design.md)。

## 10. 这个案例给出的最小判断顺序

以后再遇到“这个东西该放哪里”时，更稳的顺序不是直接找最像的目录，而是先问：

1. 这主要是 `Navigate`、`Place`，还是 `Ingest` 问题？
2. 我是在找主入口，还是已经进入主入口后判断应停在哪层？
3. 如果手上是材料而不是纯对象名，它今天能不能直接进入正文？
4. 只有把上面三步分清后，再回规则层查 stop-line 与文件职责。

如果 AI 无法在执行开始时明确回答第 1 条，就说明 capability layer 还没有被真正激活。

## 11. 什么时候才该回看这个案例

只有在以下情况才需要回看本案例：

- 已经读过主规则与 capability 文档，但 AI 仍反复跳过能力前置路由
- 需要判断“结果没错，但过程没有按 capability layer 来做”是否值得治理
- 需要说明为什么 capability layer 不是装饰层，而会实质影响目录定位与落位质量
- 需要复盘某次对象/主题落位判断为什么总是把 `Navigate`、`Place`、`Ingest` 混成一步

如果只是日常判断对象或主题该放哪里，优先先走 `docs/capabilities/navigate.md`、`place.md`、`ingest.md`，再回到 `docs/contributing/` 规则层；不要默认先回看本案例。 
