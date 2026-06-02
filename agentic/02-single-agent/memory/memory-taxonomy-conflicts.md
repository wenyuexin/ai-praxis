# Memory Taxonomy Conflicts

> 适用范围：单智能体 memory 分类体系的冲突、重叠与统一理解框架
> 阶段状态：研究主干，持续补充综述论文与系统案例证据
> 使用说明：本文件回答“为什么 agent memory 很难用单一 taxonomy 概括、常见分类体系分别来自什么视角、它们之间为什么不兼容、应如何避免过早写成标准答案”，不直接预设某一种分类法为最终定论

## 一、定位

如果说 `memory/` 目录关注单智能体如何表示、存储、检索、更新与利用记忆，那么本文件进一步讨论的是：**这些 memory 究竟应该怎么分类，以及为什么当前没有一套可以直接覆盖所有语境的稳定 taxonomy**。

在 agent 研究里，memory taxonomy 不应被狭义理解为“短期记忆 vs 长期记忆”这一个二分法。

更准确地说，不同资料往往在回答不同问题：

- 记忆按时间跨度怎么分
- 按功能怎么分
- 按存储位置怎么分
- 按动态过程怎么分
- 按信息类型怎么分

因此，taxonomy 冲突的根源并不一定是谁对谁错，而是**很多分类本来就不在同一维度上**。

---

## 二、为什么它必须单列成主题

### 2.1 当前 `memory/` 目录最容易被误读为稳定分类体系

现在目录里有：

- `short-term-memory/`
- `long-term-memory/`
- `retrieval-methods/`

这对组织材料很有帮助，但也容易让人误以为：memory 领域已经稳定收敛为这一套 taxonomy。

实际上，很多论文和系统并不按这个维度组织，或者会同时采用另一套完全不同的分类方法。

### 2.2 taxonomy 冲突会直接影响知识组织方式

如果不把冲突单列出来，后续容易出现两个问题：

- 把 `working / episodic / semantic` 强行映射到 `short-term / long-term`
- 把 `parametric / contextual / external` 与功能分类混成同一个层级

一旦混写，memory 文档会很快失去解释力。

### 2.3 这不是纯术语问题，而是研究视角问题

不同 taxonomy 往往反映不同研究重点：

- 系统工程视角更关心生命周期与检索路径
- 认知类比视角更关心 working / episodic / semantic
- LLM 系统视角更关心 parametric / contextual / external memory
- agent runtime 视角更关心 state persistence、writeback 和 contamination

因此 taxonomy 冲突本身，就是理解该领域的一部分。

---

## 三、几类常见 taxonomy 到底在分什么

### 3.1 Time-Span Taxonomy

最常见的一类是：

- short-term memory
- long-term memory

它主要回答：**信息保留多久、跨不跨步骤或跨任务**。

优点：

- 直观
- 适合工程组织
- 容易与上下文窗口、外部存储、跨任务 persistence 连接

局限：

- 只能覆盖时间跨度，不足以表达记忆内容与功能差异
- “长期”并不自动说明可检索性、写回机制或语义类型

### 3.2 Function-Oriented Taxonomy

另一类常见划分借用了认知科学语汇，例如：

- working memory
- episodic memory
- semantic memory
- sometimes procedural memory

它主要回答：**记忆在系统里承担什么功能**。

优点：

- 更适合解释“记住什么、为什么记住”
- 对任务经验、事实知识、当前工作状态的区分更细

局限：

- 与工程实现不一定一一对应
- 同一个系统组件可能同时承担多种功能角色

### 3.3 Storage / Location Taxonomy

还有一类划分关注记忆“存在哪里”，例如：

- parametric memory
- contextual memory
- external memory
- latent memory

它主要回答：**信息位于模型参数、上下文窗口、外部存储还是中间隐变量中**。

优点：

- 与 LLM 系统设计直接相关
- 适合讨论 long context、RAG、vector store、memory writeback

局限：

- 更偏实现载体，不直接说明信息类型或功能目标
- 很容易与 functional taxonomy 混层

### 3.4 Process-Oriented Taxonomy

有些研究更关注记忆的动态过程，例如：

- memory formation
- memory consolidation
- retrieval
- update / overwrite
- forgetting / decay
- writeback / summarization

它主要回答：**记忆是如何生成、维护、衰减和被调用的**。

优点：

- 更适合分析 agent runtime 与长期稳定性
- 有助于讨论污染、过时、压缩与持续学习

局限：

- 它更像过程维度，而不是对象分类本身
- 容易与“记忆类型”混写

### 3.5 Content-Oriented Taxonomy

还有一些分类直接按信息类型区分，例如：

- factual memory
- experiential memory
- task memory
- user preference memory
- environment state memory

它主要回答：**到底记住了哪类信息**。

优点：

- 更贴近真实 agent 工作内容
- 对产品系统、长期个性化和任务经验尤其实用

局限：

- 维度更贴近业务语义，不一定天然形成通用理论框架

---

## 四、它们为什么经常彼此不兼容

### 4.1 因为很多 taxonomy 不是在同一层分问题

最根本的原因是：

- `short-term / long-term` 是时间尺度
- `working / episodic / semantic` 是功能语义
- `parametric / contextual / external` 是存储位置
- `retrieval / writeback / forgetting` 是动态过程
- `factual / experiential / preference` 是内容类型

这些本来就是不同维度，因此无法强行一一映射。

### 4.2 同一个 memory 组件可能同时落在多个 taxonomy 上

例如一个外部向量存储：

- 在时间维度上可能是 long-term
- 在功能维度上可能承载 semantic 或 episodic information
- 在存储维度上又属于 external memory
- 在过程维度上还涉及 retrieval、summarization、decay

这说明“一个组件只能属于一个 taxonomy 类别”的想法本身就不成立。

### 4.3 不同论文的目标函数不同

有些论文想解释 cognition，有些想优化工程系统，有些想描述模型内部记忆形态。

所以它们在分类时关注的主轴天然不同，并不一定试图对齐成统一标准。

---

## 五、当前更可取的统一理解方式

### 5.1 不追求单一总 taxonomy，先承认多维分类

当前更稳妥的做法不是强求一个“总表”，而是承认 memory 至少可以沿几条正交维度描述：

- 时间跨度
- 功能角色
- 存储位置
- 动态过程
- 信息类型

这比强行选一个“标准分类法”更能保持知识组织弹性。

### 5.2 先问“这篇材料在按什么维度分类”

面对任何论文或系统时，优先判断：

- 它是在说 memory 保留多久
- 还是在说 memory 的功能角色
- 还是在说 memory 存储在哪里
- 还是在说 memory 如何更新与检索

这个判断步骤本身，就是避免 taxonomy 混乱的关键。

### 5.3 目录组织可以服务阅读，但不应冒充理论终局

当前 `memory/` 子目录按 `short-term / long-term / retrieval-methods` 组织，是一种有用的知识管理方式。

但应明确：

- 这是一种组织入口
- 不是对领域 taxonomy 已经收敛的宣告

否则目录结构会被误读成理论结论。

---

## 六、最重要的结构性矛盾

### 6.1 Organizational Clarity vs Theoretical Accuracy

目录和知识库需要清晰结构；但结构过于清晰时，容易假装领域已经统一收敛。

### 6.2 Cognitive Analogy vs Engineering Usefulness

认知类比 taxonomy 更有解释力；工程 taxonomy 更利于实现与系统设计。两者都重要，但不一定能直接对齐。

### 6.3 Broad Coverage vs Category Precision

类别越少，覆盖越广；但定义越容易变模糊。类别越细，表达越准；但越难形成稳定共识。

### 6.4 Stable Labels vs Fast-Moving Research

taxonomy 一旦写死，阅读体验会更稳定；但 memory 领域变化很快，过早冻结标签会降低后续文档弹性。

---

## 七、容易被误写成定论的问题

当前阶段应避免把下列说法直接写成共识：

- `short-term / long-term` 已经足够覆盖 agent memory
- `working / episodic / semantic` 可以直接映射到现有目录结构
- external memory 就等于 long-term memory
- long context 可以直接替代 memory taxonomy 问题
- 只要选定一套 taxonomy，后续 memory 组织问题就解决了

这些说法都会把本来多维的问题压扁成单一分类法，反而妨碍后续扩展。

---

## 八、与其他专题的关系

- 与 `overview.md`：本文件展开其中“memory 分类体系仍然碎片化”这一关键提醒。
- 与 `memory/README.md`：后者给出 memory 目录边界；本文件进一步说明该目录结构不应被误读为稳定理论 taxonomy。
- 与 `conflict.md`：对应 `Memory Taxonomy 不兼容` 这一条目，并为后续收敛“哪些维度值得作为长期主线”提供主干支撑。
- 与 `tool-use/`：当 memory 被用来缓存工具结果、失败历史与中间状态时，taxonomy 问题会反馈到 tool orchestration 设计。
- 与 `../05-environments/`：执行环境中的 workspace state、traceability 与 memory writeback 也会影响“哪些状态应被视为 memory”的边界。
- 与后续 `long context vs external memory` 讨论：该问题不是本文件的全部，但会直接影响 taxonomy 在工程语境中的表达方式。

---

## 九、当前最值得继续补证的方向

- 近年 agent memory 综述中，哪些 taxonomy 维度最常出现，哪些只是局部研究语汇
- 不同系统里 working / episodic / semantic memory 是否真的有稳定工程映射
- parametric、contextual、external、latent memory 四类说法是否能形成更稳定的工程解释框架
- long context 扩展后，time-span taxonomy 与 storage taxonomy 的边界是否发生变化
- memory taxonomy 是否需要按研究语境分层组织，而不是继续追求单表统一
