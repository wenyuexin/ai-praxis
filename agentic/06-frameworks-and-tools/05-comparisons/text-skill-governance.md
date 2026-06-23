# Text Skill Governance

> 适用范围：当前仓库这类以纯文本 skill、规则文本和上下文注入为主的 agent 系统。
> 阶段状态：第一版问题框架，先回答“如何用结构化机制提高规则遵守率”，不预设某个具体实现。
> 使用说明：本文关注的是模型主要通过文本完成工作的场景。重点不是阻止外部副作用，而是让 skill 和 rules 稳定塑造推理过程、步骤执行与输出结构。

## 一、为什么这一类 agent 要单独研究

对于纯文本 skill 型 agent，执行面的主要载体仍然是文本生成。

这类系统通常：

- 没有太强的环境级硬边界可依赖
- 主要通过 system / developer / task / skill 文本来约束模型
- 更依赖输出结构、自检步骤、上下文组织与失败回路来提升遵守率

现有外部研究至少提供了两类高信号提醒：

- 即使模型能够完成目标任务，也不代表它能稳定遵守 scaffold、rules 或 procedures；“任务完成”和“规则遵守”是不同能力维度。
- 长上下文中的指令保持并不稳定，规则位置、重复方式和上下文组织方式会明显影响遵守率。

因此，它的核心问题不是“别乱跑命令”，而是：

> **怎样把 skill 从一段建议性文字，提升成一种高遵守率、可检查、可复用的认知支架。**

---

## 二、这类系统真正想约束的是什么

纯文本 skill agent 主要想约束的不是外部副作用，而是以下几类行为：

- 是否按指定步骤思考和推进
- 是否遵守输出格式
- 是否在关键判断处说明依据
- 是否进行自检而不是直接结束
- 是否在遇到冲突、缺信息、超出权限时停下来
- 是否在长任务中维持全局规则，而不是被局部目标带偏

因此，这类治理更接近**认知约束与输出约束**，而不是 runtime hard boundary。

---

## 三、纯文本 skill agent 的五类治理机制

### 3.1 Instruction Layering

最基础的机制，是把不同层的文本约束分开，而不是全部塞进一段大 prompt。

常见层级包括：

- system-level rule
- developer-level operating rule
- task-specific instruction
- skill-specific procedure
- output-specific requirement

这层的目标不是“写更长”，而是让模型更清楚：

- 哪些规则全局有效
- 哪些规则只作用于当前任务
- 哪些步骤来自 skill，而不是临时建议
- 哪些输出要求是必须满足的停线

已有研究提示，instruction layering 的失效并不是单一现象，至少可以拆成：指令识别失败、冲突解析失败、响应实现失败。`Where Instruction Hierarchy Breaks` 进一步把这三类 failure 做成白盒诊断框架，并显示 dominant failure mode 会随模型、任务类型与 context length 变化。换句话说，把多层规则塞进同一上下文，并不会自然得到更强遵守率；层级关系、冲突关系和实现路径本身也需要设计。

### 3.2 Structured Procedure

skill 如果只是经验描述，遵守率通常不稳定。

更强的做法是把它写成结构化 procedure，例如显式给出：

- 前置判断
- 步骤顺序
- 何时停下
- 何时切换分支
- 何时要求澄清或补证
- 完成前必须做的检查

这一层的重点是：**让模型不只是“知道一个好习惯”，而是面对一个可执行流程。**

外部研究对这一点的支持目前更多来自相邻机制样本：结构化步骤、多维度反思和自我修订都表明，把任务过程显式拆开，通常比只给一段原则性建议更有机会提高执行稳定性。但“纯文本 skill procedure 的最小字段应该是什么”，现阶段仍然是开放问题。

### 3.3 Output Constraint

很多纯文本 agent 的失败，并不是推理完全错，而是：

- 输出结构不稳
- 缺关键字段
- 没按指定格式给出依据
- 忘记暴露不确定性

因此，output constraint 很关键。常见形式包括：

- schema-like section requirement
- checklist-driven output
- fixed answer skeleton
- explicit evidence / trace / needs fields
- “未满足这些条件不得结束”的 completion rule

它的作用是把“最后长什么样”从风格问题变成完成条件。

现有外部材料能够稳定支持的，是“输出结构塌陷”和“遗漏 required sections”确实是常见失效模式，因此 output constraint 不只是美观问题，而是治理机制的一部分。但更强的 API / schema 级硬约束，已经开始越过本文的纯文本边界，只能作为邻近参考，不能直接拿来替代纯文本 skill 治理本身。

### 3.4 Self-check / Critique Loop

如果系统只靠第一次生成结果，skill 很容易在长任务中失效。

更稳的做法，是显式引入自检回路：

- 先产出初稿
- 再检查是否违反规则
- 再检查是否漏做步骤
- 再检查是否缺依据或超出权限
- 最后才结束

这一层不是为了让模型“更会反思”，而是为了把校验变成固定机制，而不是临时自觉。

这一节目前有相对扎实的外部支撑：生成后修订、自我批判、多维度反思都已被多项研究当作可量化提升指令遵守率与输出质量的候选机制。`Where Instruction Hierarchy Breaks` 还进一步区分了两种 training-free self-monitoring 插入点：输入侧的 `PIM`（parallel input monitor）更像 generation 前的 conflict scan，输出侧的 `SOM`（sequential output monitor）更像 draft release 前的 review / repair；两者分别对应低延迟窄覆盖与高延迟宽覆盖的治理 trade-off。更稳妥的理解不是“让模型再想一遍”，而是把检查维度、修订目标和结束条件显式化；但这些机制仍有明显失效边界，不能被写成万能方案。

### 3.5 Context Shaping

很多 skill 失效，不是 skill 本身不对，而是它进入上下文的方式不稳定。

关键问题包括：

- skill 何时注入
- 是否全程保留，还是按需调用
- 示例和规则的顺序如何排列
- 当前任务状态是否回写
- 长上下文中怎样防止 skill 被局部目标淹没

因此，这一层更像：**skill 不只是文本内容，还包括它如何被装配进当前认知环境。**

现有研究至少支持两点谨慎判断：第一，长上下文中的规则保持存在明显位置效应与注意力衰减问题；第二，把执行经验回写成持续演化的 guidelines，可能比静态 prompt 更能对抗“局部目标压过全局规则”。`Where Instruction Hierarchy Breaks` 还补充了另一层更贴执行链的证据：随着长上下文 separation length 增大，instruction hierarchy 的主导 failure mode 不只会更频繁，还会在“规则识别 / 冲突解析 / 响应实现”之间重新分布。但“全程常驻”和“按需装配”到底哪种更稳，目前仍缺足够硬的对象级对照。

---

## 四、什么算有效的“科学机制”

对于纯文本 skill agent，比较科学的机制，不是简单“提示词工程”，而是至少同时具备下面四点：

1. **规则可分层**：不同约束有明确层级与作用范围
2. **流程可显式化**：skill 不是模糊建议，而是结构化 procedure
3. **结果可检查**：输出有明确 completion rule 与 failure surface
4. **偏离可纠正**：系统内有自检、重试或回退机制，而不是只靠第一次生成

如果缺少这四点，skill 往往更像“有帮助的文字”，而不是“稳定的执行指导”。

到目前为止，外部材料更能支持的是一种方向性判断：比较可靠的文本治理机制，通常不只是“写得更清楚”，而是同时具备规则分层、流程显式化、结果可检查和偏离后可纠正的组合特征。这个判断仍应保持 `Observed / Inferred` 口径，而不是写成已被单一对象完全证明的通用原则。

---

## 五、最常见的失效模式

### 5.1 规则冲突但没有分层

全局规则、任务目标、skill 步骤、输出要求混在一起，模型不知道谁优先。

现有研究表明，这类失效不宜笼统归结为“模型没听话”；更细地看，至少可能发生在规则识别、冲突解析和响应实现三个不同阶段。这个拆法的重要价值，不只是更好命名 failure，而是能反过来指导治理机制该插在哪：补 instruction retrieval、补 precedence reasoning，还是补最终 release 前的 output review。

### 5.2 skill 只有描述，没有 procedure

模型知道要“认真做”，但不知道具体先后顺序和停止条件。

### 5.3 输出要求没有变成完成条件

模型给出了一段看起来合理的话，但没有满足结构、证据、边界或校验要求。

### 5.4 长任务中局部目标压过全局规则

任务进行到后半程后，模型只顾着完成局部问题，忘了最初的规则和 skill。

这一类失效已有较强外部旁证：长上下文中的规则保持存在位置效应，而动态积累和演化 guidelines 则被一些研究视为缓解该问题的候选方向。但目前仍不足以据此断言哪一种上下文装配策略最优。

### 5.5 自检只是口号，不是机制

系统写了“完成前检查”，但没有把检查变成稳定步骤或显式输出要求。

---

## 六、和复杂 agent 的边界

本文不回答：

- tool call 前如何拦截
- approval policy 如何分级
- sandbox 如何形成硬边界
- workspace mutation 如何回滚
- runtime / environment 如何隔离

这些问题属于含代码执行、工具调用和环境副作用的复杂 agent，应转到后续复杂 agent 治理专题。

本文只回答：

- 规则怎样进入文本执行链
- skill 怎样变成高遵守率 procedure
- 输出怎样变成可检查结果
- 模型偏离时怎样靠文本机制把它拉回来

---

## 七、下一轮最值得继续补证的问题

1. skill 从自由文本升级为结构化 procedure 时，最小字段应该是什么
2. output constraint 与 self-check 哪种更能提高遵守率，还是两者必须组合
3. skill 注入应是全程常驻，还是按需装配
4. 长任务里怎样让全局规则持续可见，而不被局部目标覆盖
5. 哪些 failure pattern 最适合进入 post-hoc evaluation，而不是继续塞回 prompt

## Evidence

- Status: `Observed / Inferred`
- Sources:
  - `../../../backlog.md`
  - `./execution-governance-layers.md`
  - `../../../../docs/contributing/evidence-and-traceability.md`
  - `../../../../docs/contributing/documentation-workflow.md`
  - `../../../../docs/contributing/metadata-files.md`
  - `../../../../docs/capabilities/place.md`
  - `../../../../docs/capabilities/navigate.md`
  - `Instruction Boosting` — https://arxiv.org/html/2510.14842v1
  - `PR-CoT` — https://arxiv.org/html/2601.07780v1
  - `OctoBench` — https://export.arxiv.org/abs/2601.10343
  - `./instruction-hierarchy-breaks.md`
  - `Instruction Hierarchy 的三种失效模式` — https://arxiv.org/abs/2606.07808
  - `SCOPE` — https://browse-export.arxiv.org/pdf/2512.15374
  - `PRISM` — https://browse-export.arxiv.org/abs/2605.15665
  - `指令遵循率退化的实证量化` — https://arxiv.org/abs/2512.20662v1
  - `Self-Refine`、`Constitutional AI` 的对应论文与官方公开资料
  - `../../../../llm/01-foundations/transformer-architecture/attention-mechanisms/position-bias/papers/LostInTheMiddle_2307.03172.md`
- Trace: 从顶层“Skill、Rules 与执行约束如何真正进入 Agent Runtime”这一跨目录问题继续下沉，先把当前仓库这类纯文本 skill agent 单独抽出来，研究如何通过结构化文本机制提高规则遵守率；第一轮先形成问题框架，随后再根据外部调研中保留下来的真实论文与公开来源，把 instruction layering、self-check、context shaping 与 failure modes 相关段落补到可复核的一手或近一手来源。外部调研过程本身仍只是输入材料，因此正文中的综合判断继续保持 `Observed / Inferred`，没有把单篇论文、单个平台文档或二手材料直接写成主线定论。
- Needs:
  - 补更多纯文本 skill agent 或 prompt-governed agent 的对象样本
  - 补 skill procedure 最小字段与 failure pattern 的更细对照
  - 补 output constraint 在纯文本边界内的更强证据，避免过早滑向 API / infra 级硬约束
  - 补文本约束如何与后验评估闭环的机制研究

*最后更新: 2026-06-18*
