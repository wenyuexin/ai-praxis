# Is Grep All You Need? How Agent Harnesses Reshape Agentic Search

- **arxiv**：2605.15184
- **标题**：Is Grep All You Need? How Agent Harnesses Reshape Agentic Search
- **作者**：Sahil Sen, Akhil Kasturi, Elias Lumer, Anmol Gulati, Vamse Kumar Subbiah
- **机构**：PricewaterhouseCoopers, U.S.
- **论文类型**：研究论文（system / empirical study）
- **核心主题**：Agentic Search（agent 自主发起检索与工具调用的搜索流程）、RAG（检索增强生成）、Lexical Search、Semantic Search、Context Engineering、Agent Harness Evaluation
- **评测任务**：LongMemEval（长记忆多轮对话问答基准）的 116 个问题子集

## TL;DR
- 这篇论文研究的不是单独某种 retriever，而是 **retrieval strategy、agent harness 与 tool 结果交付方式** 如何共同影响 agentic search 的最终表现。
- 论文围绕 LongMemEval（一个用于评估 LLM 聊天助手长期记忆能力 的综合基准测试）的 116 个问题子集，比较 lexical retrieval（词汇检索，如 `grep`）与 semantic retrieval（语义检索，如vector search）在不同 agent stack 中的差异。
- **实验一** 比较 `retrieval mode × harness × 结果交付方式`；**实验二** 固定 inline 条件，观察随着 distractor session 增加，两类检索方法的行为如何变化。
- 论文的主要结论是：在这项长记忆问答任务上，`inline grep` 在多组配置里表现较强，但结果会随着 harness 与结果交付方式改变而变化。
- 论文同时明确给出边界：这些结论依赖当前任务分布，不能直接外推为“grep 普遍优于 vector”。

> **一句话总结**：论文研究的重点不是孤立比较 `grep` 和向量检索，而是分析在 agentic workflow 中，retrieval strategy、agent harness、tool 输出交付方式与上下文噪声如何共同影响最终表现；在 LongMemEval 这类长记忆问答任务上，`grep` 在多组实验中表现较强，但这种优势会随着 harness 和结果交付方式的变化而变化。

## 1. 论文要解决什么问题

### 1.1 核心问题
论文聚焦一个非常实际但过去常被忽略的问题：

> 当 LLM agent 不是一次性拿到检索结果，而是要在真实工具循环中“自己发起搜索—读结果—决定是否继续搜索—再组织答案”时，`lexical retrieval` 和 `semantic retrieval` 的优劣是否还和传统 RAG 评测里一样？

更具体地说，作者要回答三个问题：

1. **检索策略本身**：`grep/BM25/regex` 一类 lexical 方法，与 embedding/ANN 一类 vector 方法，在 agent 环境里谁更有效？
2. **harness 影响**：同样的模型、同样的语料，放进 custom harness（开发者自定义的代理运行框架）与 provider-native CLI harness（模型厂商原生提供的命令行代理框架）里，结果会不会变？
3. **交付路径影响**：tool 结果是直接 inline（直接嵌入上下文）注入上下文，还是先写进文件、让 agent 再自己读取，这个差异会不会改变检索优劣？

### 1.2 为什么这是个重要问题
传统信息检索和传统 RAG 往往默认一个静态 pipeline：

- 给定 query
- retriever 返回 top-k
- 把结果拼到 prompt
- LLM 作答

但 agentic search 不是这样。真实 agent 会：

- 决定搜什么词
- 决定搜多少轮
- 决定何时停止
- 决定读取全部结果还是局部结果
- 决定如何把工具结果整合进自己的推理过程

因此，**retrieval 的“效果”不再只是 ranking quality（排序质量），而是 end-to-end task performance（端到端任务表现）**。论文的关键出发点就是：

> 在 agent setting（agent 工作场景）中，retrieval 已经嵌入到 orchestration loop（编排循环）里，如果还把 retriever 当成孤立模块来评测，会低估系统层面的方差。

### 1.3 论文的动机
作者观察到两个现实趋势：

- 向量检索已经成了多数 RAG 系统默认选项；
- 但在很多工程实践里，`grep` 依然因为简单、便宜、稳定而被大量使用。

与此同时，Claude Code、Codex、Gemini CLI 这类 provider-native CLI agent 的出现，又带来了新的变量：

- 模型可以直接用 shell 工具；
- retrieval 不再只是“调用一个 search API”，而可能变成“构造 bash 命令并在文件系统上操作”；
- harness 如何暴露工具、如何组织 transcript、如何把 stdout 拼进上下文，都可能改变最终结果。

作者的核心动机因此很明确：**把 retrieval、harness、presentation 三个维度作为一个联合系统来评测。**

## 2. 论文的核心观点与主要贡献

论文在引言中明确给出三类贡献：

| 贡献方向 | 论文实际做了什么 | 论文强调的意义 |
|---|---|---|
| Retrieval × Harness × Presentation | 比较 lexical / vector 检索在不同 harness、不同 tool output delivery（工具结果交付方式）下的效果 | 说明检索效果需要结合 orchestration layer（代理编排层）与结果交付方式一起看 |
| Noise and Scale | 研究随着无关上下文增多，系统表现如何变化 | 观察 retrieval behavior 与 broader agent loop（更完整的 agent 工具循环）的互动 |
| Heterogeneity Across Agent Stacks | 对比 Chronos 与 Claude Code / Codex / Gemini CLI | 说明同一语料下，不同 harness 的结果并不稳定一致 |

## 3. 技术背景：如何定义问题空间

作者在第 2 节把问题拆成两个正交维度，再加一个交付维度，这个抽象非常重要。

### 3.1 维度一：Retrieval Strategies

论文把检索策略分成三类：

1. **Lexical Search**
   - 代表：
     - `grep`：直接在原始文本里做关键词、子串或正则匹配，适合快速定位字面证据。
     - `BM25`：按关键词匹配强度给文档排序，更像传统信息检索里的关键词打分检索。
     - `regex`：用模式表达式匹配固定格式或结构化片段，适合找日期、编号、特定模板语句。
   - 特点：按词面、子串或模式精确匹配
   - 优势：
     - 不需要 embedding model
     - 不需要向量索引基础设施
     - 成本低、可解释性强
   - 劣势：
     - 容易因为 query phrasing 不对而 miss 掉答案

2. **Semantic Search**
   - 代表：
     - dense retrieval：把 query 和 document 编码成稠密向量，再按语义相似度检索。
     - vector retrieval：工程语境里常用的叫法，本质上就是基于向量库的 dense retrieval。
     - ANN：近似最近邻搜索，是 dense / vector retrieval 常见的高效召回实现方式，而不是另一种独立检索范式。
   - 特点：把 query 和 document 投影到同一嵌入空间，按语义相似度召回
   - 优势：
     - 擅长处理 paraphrase、同义表达、语义近邻
   - 劣势：
     - 依赖 embedding 质量
     - 需要索引和基础设施
     - 可能引入“主题相近但不相关”的 false friends

3. **Hybrid Retrieval**
   - 代表：
     - `RRF`：对多路检索结果做倒数排名融合，把 lexical 与 dense 的候选列表合并后再统一排序。
     - `ColBERT`：保留更细粒度的 token 级匹配信息，兼顾一定语义能力与较强匹配精度。
     - `sparse+dense`：同时结合稀疏检索信号与稠密向量检索信号，希望同时覆盖字面匹配和语义相关性。
   - 论文没有在实验中直接测试 hybrid，但把它作为未来自然方向提出。

### 3.2 维度二：Agent Harnesses

作者把 harness 定义为：负责管理 tool-calling loop 的环境层，它决定：

- prompt 如何构建
- tool 如何暴露
- tool result 如何回传
- 何时停止搜索并输出答案

论文区分两类 harness：

这里原文强调的核心差别，不只是“谁来调用工具”，而是**对 agent loop 各阶段的控制程度**。也就是说，custom harness 允许开发者显式设计 prompt、tool 接口、上下文管理和停止条件，而 provider-native CLI harness 则把其中相当一部分决策封装在厂商内部实现里。

| 类别 | 代表 | 特征 | 代价/问题 |
|---|---|---|---|
| Custom Harness | Chronos | 开发者可精细控制 system prompt、tool 定义、上下文构建、结果格式与终止条件，并可加入 dynamic prompting、结果截断策略、reranking 等优化 | 构建和维护成本高，要求较强的 prompt engineering、tool interface design 与 context management 能力 |
| Provider-Native CLI Harness | Claude Code / Codex / Gemini CLI | 模型可直接调用 `grep`、`find`、`cat` 等系统工具；当 `grep` 作为原生 bash 工具可用时，检索策略与 agent 工具能力的边界会变模糊 | 部署成本低，但内部实现对用户基本不透明，上下文构建与迭代控制难以精细掌控 |

### 3.3 维度三：Tool-Calling Architectures

原文把这一维单独拿出来讨论，因为它直接影响 retrieval 结果如何被模型消费。换句话说，这里讨论的就是：搜索结果最终以什么结果交付方式进入模型可消费的上下文。

作者把 tool output 的交付方式分成两种：

1. **Standard / Inline**
   - 搜索结果直接作为 tool message 注入上下文
   - 好处：简单，模型立刻可见
   - 坏处：结果与 system prompt、conversation history 争夺上下文窗口，容易造成 context pressure / context rot（上下文拥挤与上下文质量退化）

2. **Programmatic / File-Based**
   - 搜索结果先写到文件里
   - 模型拿到的是路径或简短指针，需要自己再去 `cat`、`read_file` 或继续 `grep`
   - 好处：大结果集不必立刻占用上下文
   - 坏处：把原本一步的检索变成多步“定位—读取—整合”任务，要求 agent 具有更强的组合式工具使用能力

### 3.4 论文采用的系统视角
按照第 2 节的定义，论文并不把检索看成一个脱离 agent loop 的独立步骤，而是把 retrieval strategy、agent harness 和 tool-calling architecture 放在同一个系统框架里讨论。后续实验设计也是围绕这个框架展开的。

## 4. 方法与实验设计

这篇论文不是提出一个新模型，而是设计了一套系统化对比实验。

### 4.1 数据集与任务
作者使用 **LongMemEval** 的一个 **116 题代表性子集**。

LongMemEval 的特点是：

- 问题来自多 session 长对话；
- 每个问题对应：
  - **oracle sessions**：真正含答案的信息会话
  - **distractor sessions**：无关干扰会话
- 问题覆盖 6 类：

| 类别 | 含义 |
|---|---|
| knowledge-update | 跟踪状态变化 |
| multi-session | 跨多 session 聚合信息 |
| single-session-assistant | 回忆模型生成过什么 |
| single-session-preference | 用户偏好 |
| single-session-user | 用户陈述事实 |
| temporal-reasoning | 日期、时长、顺序等时间推理 |

### 4.2 为什么使用 Chronos 的 structured event layer
论文在 retrieval layer（检索前处理层）前加入了 Chronos pipeline（作者的自定义 agent harness 流程），把对话中的 **时间事件** 抽取成结构化记录，并与原始 utterance（原始对话文本）一起存储。

原文给出两个原因：

1. **研究重点是 search，而不是 temporal reasoning 本身**。很多 LongMemEval 题依赖日期、区间和时序信息，加入结构化事件后，可以更集中地观察 agent 是否找到了相关证据。
2. **这种预处理更接近作者设想的生产配置**。也就是说，实验设置并不是只为 benchmark 临时构造，而是和 long-memory agent 的实际部署思路保持一致。

### 4.3 两种检索实现

#### 4.3.1 Lexical Search (Grep)
- 对 per-question 文件中的 conversation turns 与 temporal events 做 regex 匹配；
- 按 match count 计分并返回结果；
- 不依赖 embedding、vector index 或外部服务。

#### 4.3.2 Semantic Search (Vector)
- 在 ingestion（数据入库/索引构建）阶段对每条 turn/event 建索引；
- query 时对自然语言问题做 embedding（向量表示）；
- 用 ANN（近似最近邻）搜索召回；
- 再做 reranking（重排序）；
- top-k 的 `k` 可由 agent 决定。

### 4.4 两类 harness 的具体实现

按照原文设定，harness 本身就是实验变量之一。前面第 2 节已经说明，custom harness 与 provider-native CLI harness 在 prompt、tool interface、context construction 和结果呈现方式上存在系统性差异；这里的方法部分则给出这些差异在实验中的具体实现。

#### Chronos（Custom Harness）
- 基于 LangChain；
- 提供 4 个搜索工具：对 turns 和 events 的 grep / vector search；
- programmatic mode 下额外开放 grep 工具；
- 使用 **dynamic prompting**：根据题目类别给不同 system instructions 和 search hints；
- episode 开始前先给一个初始 broad context block（top-15 vector 结果）。

这部分实现对应了前文对 custom harness 的定义：Chronos 不只负责调用检索工具，也负责以类别条件化方式组织搜索与回答过程。

#### Provider-Native CLI Harnesses
- 包括 Claude Code、Codex、Gemini CLI；
- 模型拿到问题和动态生成的 search strategy；
- 通过绝对路径调用 bash wrapper script 做 grep / vector search；
- standard mode 下把访问范围限制在相关文件集。

### 4.5 评测模型
论文测试了 5 个模型 / 能力档：

从这里也能看出，这篇工作更接近 **empirical study / system evaluation**：重点是比较不同 retrieval strategy、agent harness 与 tool-calling architecture 在统一实验框架下的表现。

- Claude Opus 4.6
- Claude Haiku 4.5
- GPT-5.4
- Gemini 3.1 Pro
- Gemini 3.1 Flash-Lite

### 4.6 评测方法
- 使用 **GPT-4o 作为辅助 grader（自动判分模型）**；
- 输入：问题、reference answer（参考答案）、agent hypothesis（模型作答）；
- 输出：binary judgment（二元判断，正确/错误）；
- metric：accuracy（准确率）。

作者强调固定 grader、prompt template、decoding setting，从而把差异尽量归因到 agent pipeline，而不是评估噪声。

### 4.7 两个实验分别回答什么

| 维度 | 实验一 | 实验二 |
|---|---|---|
| 主要目标 | 比较 retrieval mode、harness、结果交付方式 的联合作用 | 观察噪声增加时 grep 与 vector 的相对行为 |
| 固定条件 | 完整候选语料 | standard / inline delivery |
| 变化条件 | `grep-only / vector-only`、Chronos / CLI harness、inline / programmatic | `s5 / s10 / s20 / s30 / full` session limit |
| 回答的问题 | 不同 stack 和结果交付方式是否会改变 retrieval 表现 | 当 distractor 增加时，两类检索方法的排序是否稳定 |

## 5. 实验一：检索方式 × Harness × Tool 输出方式

### 5.1 实验目标
实验一要回答：

> 在完整候选语料下，`grep-only vs vector-only`、`Chronos vs provider CLI`、`inline vs programmatic` 三个因素如何共同影响 end-to-end 长记忆问答准确率？

这实际上是一个三因素系统对比实验。

### 5.2 关键结果总表
下表对应原文 Experiment 1 的主结果表，横向比较 retrieval mode 与 tool-calling method，纵向比较不同 harness-model 组合。

| Model | Harness | Inline Grep | Inline Vector | Programmatic Grep | Programmatic Vector |
|---|---:|---:|---:|---:|---:|
| Claude Opus 4.6 | Chronos | **93.1** | 83.6 | 80.2 | 81.9 |
| Claude Opus 4.6 | Claude Code | **76.7** | 75.0 | 68.1 | **79.3** |
| Claude Haiku 4.5 | Chronos | **83.6** | 76.7 | **83.6** | 81.9 |
| Claude Haiku 4.5 | Claude Code | **55.2** | 44.0 | **37.1** | 32.8 |
| GPT-5.4 | Chronos | **89.7** | 81.9 | **87.1** | 75.0 |
| GPT-5.4 | Codex CLI | **93.1** | 75.9 | 55.2 | **67.2** |
| Gemini 3.1 Pro | Chronos | **91.4** | 82.8 | **79.3** | 76.7 |
| Gemini 3.1 Pro | Gemini CLI | **81.9** | 75.0 | 81.0 | **82.8** |
| Gemini 3.1 Flash-Lite | Chronos | **86.2** | 62.9 | **85.3** | 72.4 |
| Gemini 3.1 Flash-Lite | Gemini CLI | **87.1** | 67.2 | 68.1 | **74.1** |

### 5.3 最核心的发现

#### 发现 1：在 inline delivery 下，grep 全部高于 vector
原文明确指出：

> **inline 模式下，10 个 harness-model pair 全部是 grep 优于 vector。**

作者给出的解释是，LongMemEval 奖励的是 **literal witnesses（字面证据片段）** 的恢复，例如：

- 精确日期
- 明确计数
- 个人偏好
- 原话中的稳定片段

对照原文讨论，这类信息更容易通过 lexical matching 被直接定位，因此 inline grep 在该任务设置下表现更强。

#### 发现 2：harness 变化会显著影响结果
例如同一个 Claude Opus 4.6：

- 在 Chronos + inline grep 下达到 **93.1%**
- 在 Claude Code + inline grep 下是 **76.7%**

对照原文，这说明即使模型、语料和 retrieval mode 不变，仅更换 harness，也会带来明显差异。作者据此强调，harness 会直接影响 end-to-end 表现。

#### 发现 3：Programmatic delivery 会改变 grep 与 vector 的相对排序
原文指出，一旦从 inline 变成 file-based，结果就不再保持同样排序：

- programmatic vector 在 10 组里有 5 组高于 programmatic grep；
- 例如 Claude Code + Opus、Codex + GPT-5.4、Gemini CLI + Flash-Lite 这些组合里，programmatic vector 都高于 programmatic grep。

结合原文 discussion，这说明 file-based delivery 不只是改变结果存放位置，还改变了 agent 获取和整合结果的方式。

#### 发现 4：结果交付路径本身会带来额外压力
原文举出的一个代表性例子是：

- `Codex CLI + GPT-5.4`
  - inline grep：**93.1%**
  - programmatic grep：**55.2%**

在这个例子里，检索语料和检索方法类别都没有变化，但结果交付方式改变后，准确率明显下降。按照原文 discussion，这提示 file-based routing 本身就是 agent tool use 的一部分。

### 5.4 作者如何解释 grep 在 inline 下占优
原文的解释主要包括以下几点：

- `grep` 更偏向高精度的字面匹配；
- `vector` 能覆盖 paraphrase 和间接提及，但也可能带来更多语义上接近却不相关的内容；
- 在 LongMemEval 这类依赖 literal spans（字面片段）的任务里，lexical matching 的 precision bias（偏高精度的匹配倾向）更容易转化为优势；
- 当结果以 inline 方式直接注入上下文时，这种优势更容易体现出来。

因此，原文并没有把 lexical 与 dense 的差别写成绝对优劣，而是把它们理解为在不同任务条件下表现不同的两类检索方法。

## 6. 实验二：随着噪声增加，系统如何变化

### 6.0 先说明本实验的边界
这一组实验和实验一不同，**它只比较 standard / inline 条件下的 `grep-only` 与 `vector-only`，不再覆盖 programmatic / file-based delivery**。原文这里的核心目的是在固定 tool-delivery convention 的前提下，观察随着 distractor session 增加，lexical 与 dense retrieval 的相对行为如何变化，而不是再次比较文件式结果交付。

### 6.1 实验目标
实验二不只是看“完整候选语料谁更强”，而是进一步问：

> 当每道题周围加入越来越多 distractor session 时，grep 和 vector 的行为轨迹是否一致？

作者设置了 `s5 / s10 / s20 / s30 / full` 五档 session limit：

- oracle session（真正包含答案证据的会话）始终保留；
- 其余位置用同一 bundle 内采样来的 distractor session（无关干扰会话）填充；
- 这样可以观察随着噪声增长，系统性能如何变化。
- 原文还特别强调：两类 retriever 在每个配置下看到的是**相同的 distractor exposure（相同的干扰暴露）**，因此比较的是 paired comparison（配对比较），而不是两个独立采样实验。

### 6.2 Grep-only 结果
下表对应原文 Experiment 2 中的 grep-only 配置结果。

| Model | Harness | s5 | s10 | s20 | s30 | full |
|---|---:|---:|---:|---:|---:|---:|
| Claude Opus 4.6 | Chronos | 89.3 | 89.7 | **90.5** | 85.3 | 89.7 |
| Claude Opus 4.6 | Claude Code | 91.4 | 94.0 | **95.7** | 90.5 | 94.0 |
| Claude Haiku 4.5 | Chronos | 83.7 | 84.5 | **86.2** | 85.3 | 83.6 |
| Claude Haiku 4.5 | Claude Code | **89.7** | 87.1 | 83.6 | 80.2 | 81.9 |
| GPT-5.4 | Chronos | **83.2** | 82.8 | 81.9 | 78.5 | 78.5 |
| Gemini 3.1 Pro | Chronos | 86.2 | 86.6 | **87.4** | 82.2 | 86.6 |
| Gemini 3.1 Pro | Gemini CLI | 78.1 | 78.5 | **79.2** | 74.7 | 78.5 |
| Gemini 3.1 Flash-Lite | Chronos | 88.8 | 89.2 | **90.0** | 84.8 | 89.2 |
| Gemini 3.1 Flash-Lite | Gemini CLI | 70.4 | 70.7 | **71.3** | 67.2 | 70.7 |

### 6.3 Vector-only 结果
下表对应原文 Experiment 2 中的 vector-only 配置结果。

| Model | Harness | s5 | s10 | s20 | s30 | full |
|---|---:|---:|---:|---:|---:|---:|
| Claude Opus 4.6 | Chronos | 94.0 | **94.8** | 92.2 | 84.5 | 92.2 |
| Claude Opus 4.6 | Claude Code | 77.6 | 72.4 | 75.0 | **78.4** | 72.4 |
| Claude Haiku 4.5 | Chronos | 87.9 | 89.7 | **90.5** | 87.1 | 87.9 |
| Claude Haiku 4.5 | Claude Code | 50.0 | 47.4 | **50.9** | 48.3 | 44.0 |
| GPT-5.4 | Chronos | 88.8 | **94.0** | 86.2 | 82.8 | 82.8 |
| Gemini 3.1 Pro | Chronos | **92.2** | 91.4 | 87.1 | 85.3 | 84.5 |
| Gemini 3.1 Pro | Gemini CLI | 84.5 | 83.6 | 82.8 | 85.3 | **89.7** |
| Gemini 3.1 Flash-Lite | Chronos | 88.8 | 88.8 | 87.9 | **88.8** | 83.6 |
| Gemini 3.1 Flash-Lite | Gemini CLI | 69.8 | 69.8 | **76.7** | 70.7 | 74.1 |

### 6.4 实验二最关键的发现

#### 发现 1：性能变化不是简单单调下降
从表格可以看到，结果并不总是随着 session limit 增大而线性下降：

- 有些配置在中间档位更高；
- 有些配置在后续档位下降后又回升；
- grep 与 vector 的相对排序也会发生变化。

原文据此强调，这里观察到的不只是“噪声增多后准确率下降”，还包括两类检索方法与 agent loop 的相互作用。

#### 发现 2：vector 在部分低噪声设置下更强，但这种关系并不固定
在 Chronos 上，vector 在一些较小 session limit 配置下表现更强；例如 Claude Opus 4.6 和 GPT-5.4 在 `s5`、`s10` 时的 vector 结果都高于 grep，而 Gemini 3.1 Pro 在 `full` 时又出现了 grep 高于 vector 的情况。

原文对这种现象给出的解释是条件性的：vector retrieval 在部分较小候选集下可能更容易覆盖语义相关内容，但随着 distractor 增加，两类检索方法的相对表现会和 harness、search trace（搜索轨迹）以及 stop policy（停止策略）一起变化。

#### 发现 3：不同 provider stack 上的排序关系并不一致
原文总结了几个稳定模式：

- **Claude Code**：对 Opus / Haiku，grep 在所有报告配置里都更高；
- **Gemini CLI + Gemini 3.1 Pro**：vector 在所有报告配置里都更高；
- **Chronos**：grep 与 vector 会随着 session limit 变化而交叉。

这也是论文强调 agent stack 异质性的一个直接例子。

### 6.5 图 1 给出的总体趋势
除了表格，原文还给出了一张总体趋势图（Figure 1）。按照图注，其结论是：随着噪声加入，两类方法的平均性能都没有明显崩塌，但从整体平均来看，GREP 高于 vector。

这一点与表格形成互补：表格展示的是不同 harness / model / session limit 下的局部差异，而 Figure 1 展示的是跨配置汇总后的平均趋势。

## 7. Appendix 的分类别结果说明了什么
> **口径声明**：以下分类别结果按**附录表头版本**解读，即 `Chronos + grep-only + inline + 完整候选语料`。这样做是为了与附录表格 caption 保持一致；同时需要注意，论文正文与附录表头在这一点上存在前后不一致。

下表对应附录中的分类别准确率切片，但需要先处理原文在设置描述上的前后不一致。

附录只给出了一个切片，但这里需要特别说明：**论文正文与附录表头对该切片的描述存在前后不一致**。

- 正文在实验一结果后写的是：`Chronos harness (grep-only, programmatic tool calling method, full haystack)`；
- 附录表头写的是：`Chronos harness under grep-only retrieval, inline tool calling method, and the full haystack`。

下文的分类别结果解读，采用的是**附录表头版本**，也就是：**Chronos + grep-only + inline + 完整候选语料**。这样处理是为了与附录表格 caption 保持一致；同时需要记住，原文在这里存在前后不一致。

| Model | KU | MS | SS-A | SS-P | SS-U | TR |
|---|---:|---:|---:|---:|---:|---:|
| Claude Opus 4.6 | 94.4 | 83.9 | 100.0 | 100.0 | 87.5 | 87.1 |
| Claude Haiku 4.5 | 83.3 | 71.0 | 100.0 | 85.7 | 87.5 | 87.1 |
| GPT-5.4 | 77.8 | 74.2 | 92.3 | 85.7 | 93.8 | 67.7 |
| Gemini 3.1 Pro | 88.8 | 69.3 | 100.0 | 85.7 | 81.3 | 100.0 |
| Gemini 3.1 Flash-Lite | 94.3 | 72.6 | 100.0 | 100.0 | 81.3 | 100.0 |

从这个表可以看出几类现象：

1. `single-session assistant` 和 `single-session preference` 两类题型上，多数模型分数较高；
2. `multi-session` 的分数相对更低；
3. `temporal reasoning` 上，不同模型之间仍然存在明显差异。

结合原文方法部分，这个附录表可以作为一个补充切片：即使使用了 structured event preprocessing，最终结果仍然会受到模型能力影响。

## 8. 论文的主要技术认识
这一节对前文结果做收束性归纳，尽量保持在原文已经论证到的范围内；其中如果出现 retrieval-in-the-loop（检索嵌入工具循环）、结果交付方式这类术语，都应理解为“检索被嵌入 agent 工具循环后形成的端到端系统行为”，而不是孤立的检索模块指标。

### 8.1 检索效果与agent loop绑定
根据原文的整体论证，这篇论文强调：在 agent 系统中，检索效果不是孤立产生的，而是与 query 生成、tool 调用、结果展示和结果消费过程一起决定的。换句话说，论文关注的是 **retrieval-in-the-loop**，而不是只比较静态检索器本身。

### 8.2 tool结果交付方式会影响最终表现
原文把 `inline` 和 `file-based` 明确视为 tool-calling architecture 的组成部分，而不是单纯实现细节。前者把结果直接放进上下文，后者要求 agent 再去读取文件，因此二者对上下文占用和 tool 使用路径的要求不同。这也是为什么实验一里，同一 retriever 在不同结果交付方式下会出现不同表现。

### 8.3 lexical 与 dense retrieval 在任务上各有侧重
按照原文讨论：

- lexical retrieval 更依赖字面匹配，适合寻找稳定的显式证据；
- dense retrieval 更依赖语义相似性，能够覆盖 paraphrase 或间接提及；
- 在 LongMemEval 这类任务中，很多答案依赖 literal spans，因此 lexical retrieval 更容易在部分设置下把这种任务特征转化成端到端优势。

论文并没有把这种差异表述成“谁绝对更高级”，而是把它理解为两类检索方法在不同任务条件下的不同表现。

### 8.4 harness 本身是研究变量
原文多次强调，同一模型在不同 harness 上可能出现显著差异。因此，论文把 harness 视为系统变量，而不是中性的运行容器。这也是全文反复比较 Chronos 与 provider-native CLI harnesses 的原因。

### 8.5 如何把这些结果变成成工程上可操作的判断
这一小节不是额外引入新结论，而是把原文 discussion 中已经出现的条件性分析重新组织出来，便于工程实践理解。

#### 8.5.1 当任务依赖字面证据时，grep 更值得优先评估
原文在实验一 discussion 和 limitations 里都强调，LongMemEval 这类任务经常要求恢复 **日期、计数、偏好、原话片段** 这类字面证据。在这种情况下，lexical retrieval 的高精度匹配更容易直接转化为端到端优势。因此，如果工程任务的答案本身就表现为稳定字符串、明确字段值或容易被模式捕捉的短片段，那么 `grep` 一类 lexical 工具值得优先纳入候选方案。

#### 8.5.2 当问题表达和证据表达不完全一致时，vector 更可能补上覆盖面
原文同时指出，dense retrieval 更擅长处理 paraphrase、间接提及和语义近邻。换句话说，如果用户问题的说法和语料里的原始表达经常不一致，那么 vector 检索更可能先把相关候选找出来。这里的代价是，它也更容易把语义上接近但并不真正相关的 distractors 一起带回来。

#### 8.5.3 grep 和 vector 的优劣，部分取决于 agent 能否提出合适的搜索式
原文把两类方法的 failure mode 说得很明确：grep 更“窄”，如果 agent 没有猜到足够有区分度的子串，就可能什么也找不到；vector 更“宽”，即使 query 不完全贴词，也可能召回相关内容，但也更容易把主题相近的无关项抬上来。因此，从工程角度看，检索策略不只是 retriever 选择问题，也和 agent 的 query generation 能力绑定在一起。

#### 8.5.4 file-based 并不天然更强，它是在拿上下文带宽换工具组合能力
原文对 programmatic / file-based delivery 的分析非常有工程意义：它确实可以缓解 inline 结果直接占满上下文的问题，但代价是把任务变成“定位结果—读取文件—整合证据”的多步流程。如果 agent 不能稳定完成这个 read-integrate-retry 循环，那么理论上的检索优势并不会自动转化成最终答案质量。换句话说，file-based routing 本身就是一种 tool-use stress test。

#### 8.5.5 所谓“规模增大后 vector 一定更好”，论文并没有支持这个简单说法
原文在实验二 discussion 里专门反驳了一种常见直觉：不是说语料一变大，vector 就会稳定接管优势。论文观察到的情况更接近：在较小 session limit 下，vector 往往更强；但随着 distractor 增加，是否继续领先，要看 harness、backbone、tool transcript 以及 agent 的 stop policy。也就是说，噪声规模不是唯一决定因素，系统交互方式同样在改写结果。

#### 8.5.6 对工程实践最重要的不是“谁绝对更强”，而是先识别自己落在哪类条件里
如果把全文 discussion 收束成最实用的一点，那么就是：

- 当证据更字面、结果可直接 inline 消费、agent 能提出高精度搜索词时，grep 往往更值得优先评估；
- 当问题与证据之间存在较多改写、需要语义覆盖，或者特定 harness / stack 对 vector 更友好时，vector 可能更合适；
- 但无论哪种情况，论文都不支持把 retriever 脱离 harness 和结果交付方式单独下结论。

## 9. 论文明确给出的局限与边界

### 9.1 结论依赖当前任务分布
原文在 Limitations 中明确指出，本文结论绑定在 long-memory conversational QA 这一任务形态上；这里的问题经常涉及多轮对话、显式时间表达、用户事实与偏好，因此 lexical tools 可能会得到更大帮助。作者同时说明，这并不意味着 grep 在所有 RAG 场景里都优于 vector，尤其是在 scientific synthesis、视觉文档或代码语义等证据不以字面形式出现的场景中，结果可能不同。

### 9.2 实验覆盖还不完整
原文也明确承认，实验二里的部分 provider CLI 结果尚未补齐，例如 Codex 的若干中间配置缺失。因此，论文能够说明“不同 stack 之间存在异质性”，但还不足以给出一个覆盖所有 provider stack 的完整结论。

### 9.3 机制解释仍有条件性
论文正文里对一些现象给出了机制解释，例如 file-based delivery 会把任务变成更复杂的 artifact readback（回读外部结果文件）流程、弱模型可能更难完成 iterative refinement（迭代式细化），但作者也保留了条件性表达，没有把这些解释写成严格因果定论。对照原文，这些解释更适合作为对结果的理解辅助，而不是独立结论。

## 10. 结论
这一节对应对 Introduction、Experiments、Limitations 与 Conclusion 的合并概括。

结合原文的 Introduction、Experiments、Limitations 和 Conclusion，可以把这篇论文的结论概括为：

- 在本文比较范围内，最强的经验性结论来自实验一：在已报告的完整候选语料配置里，`inline grep` 对所有已报告的 harness-model pair 都高于 `inline vector`；
- 但 retrieval 的最终效果并不只取决于 `grep` 或 `vector` 本身，还取决于 agent harness 与 tool 结果交付方式；即使底层语料不变，当结果交付方式从 inline 改成 file-based 时，lexical advantage 也可能被削弱、抹平，甚至反转；
- 实验二进一步说明：随着 distractor 增加，两类检索方法的相对排序并不会稳定保持不变；而且这种变化形状还依赖具体观察的是哪一类 agent stack；
- 因此，论文主张把 retrieval mechanics、harness orchestration 与结果交付方式作为一个联合系统来评估，而不是分开看待；
- 同时，作者也明确给出边界：这些结论绑定在 long-memory conversational QA 的任务分布上，且当前实验结果还不足以构成一个覆盖所有 provider stack 的 vendor-complete picture。


## 🔗 可关联主题
- `LongMemEval`
- `Chronos`
- `agent harness`
- `context engineering`
- `tool-calling architecture`
- `lexical vs dense retrieval`
- `provider-native CLI agents`
