# Agent Harness Engineering: A Survey（ETCLOVG Harness 分类调查）

**论文信息**
- 论文标题：Agent Harness Engineering: A Survey
- 中文标题：Agent Harness 工程：一份调查（ETCLOVG 七层分类法）
- 作者：Junjie Li, Xi Xiao, Yunbei Zhang, Chen Liu, Lin Zhao, Xiaoying Liao, Yingrui Ji, Janet Wang, Jianyang Gu, Yingqiang Ge, Weijie Xu, Xi Fang, Xiang Xu, Tianchen Zhao, Youngeun Kim, Tianyang Wang, Jihun Hamm, Smita Krishnaswamy, Jun Huan, Chandan Reddy 等 20 余位作者
- 发布时间：2026 年
- PDF 论文：[https://picrew.github.io/LLM-Harness/main.pdf](https://picrew.github.io/LLM-Harness/main.pdf)
- 项目主页：[https://picrew.github.io/LLM-Harness/](https://picrew.github.io/LLM-Harness/)
- GitHub 文档仓库：[https://github.com/Picrew/awesome-agent-harness](https://github.com/Picrew/awesome-agent-harness)
- Evidence 说明：本文件基于 PDF 论文 `main.pdf`、项目主页与 `awesome-agent-harness` GitHub 文档仓库撰写。`main.pdf` 是当前最高优先级来源；项目主页与 GitHub 仓库主要用于补充分类清单、项目映射和数据文件。此前因网页抓取受前端渲染影响而无法完整提取的部分，后续应优先以 PDF 原文复核。**本调查（Li et al.）与另一篇同名调查（Meng et al., "Agent Harness for Large Language Model Agents: A Survey", Preprints 2026）是不同的工作。** 后者提出 6 组件（ECTSLV）分类法，非本文的 ETCLOVG 七层分类法，不应混淆。

---

> **导读**
>
> **问题**：Agent 在生产部署中反复出现一个模式——任务执行的可靠性越来越不依赖于底层模型，而依赖于包裹模型的**基础设施层（harness）**。但已有的讨论（prompt、agent framework、ACI、agent OS）都无法作为一个统一的分类系统来比较不同系统的 harness 设计取舍。
>
> **核心贡献**：本调查将 Agent Harness Engineering 定义为独立系统层，提出 **ETCLOVG 七层分类法**（Execution, Tooling, Context, Lifecycle, Observability/Operations, Verification/Evaluation, Governance/Security），并将 250+ 个开源项目映射到该分类法上，以揭示生态模式、覆盖缺口和涌现的设计原则。
>
> **三大主张（Claims）**：
> 1. **Harness 是独立系统层**：实际可靠性由执行控制、反馈回路、治理、评估和运营设计塑造，而不只是模型能力。
> 2. **ETCLOVG 分离了生产关注点**：七层架构暴露了早期框架经常混淆的架构边界。
> 3. **生态系统图揭示缺口**：对开源生态的系统性映射揭示采纳模式、能力缺口和新兴设计原则。
>
> **结构**：摘要与三大主张 → ETCLOVG 分类法（7 层 × 范围/范围/项目密度） → 三个工程阶段（定位该调查在 agent 工程演化中的位置） → 开源项目映射 → 开放问题 → 与当前仓库主题的关系
>
> **阅读建议**：
> - 想了解 Harness 生态的系统截面：读 §1（三大主张）+ §2（ETCLOVG 七层）。
> - 想了解 Harness 工程的演化脉络：读 §3（三个工程阶段）。
> - 想了解调查的实证基础：读 §4（开源项目映射的方法论和项目分布）。
> - 想了解这份调查仍留下了哪些跨层未决问题：读 §5（五个开放问题）。
> - 想了解它与当前仓库主题的关系：读 §6（与当前仓库主题的关系）。

---

## 1. 三个核心主张

本调查以三条主张（Claims）为纲领，将 Harness 从松散的设计讨论提升为一个可系统分类和比较的研究领域。

### Claim 1: Harnesses are independent system layers.

> Real-world reliability is shaped by execution controls, feedback loops, governance, evaluation, and operational design, not only by model capability.

本调查在这里的独特贡献在于**用生态证据来支持这一主张**：通过系统性地映射 250+ 个开源项目，展示 Harness 相关的工程实践已经是一个可独立观察和分类的现象。它不再只是抽象主张，而是可以在 250+ 个项目中观察到的实际趋势。

### Claim 2: ETCLOVG separates production concerns.

> Execution, Tooling, Context, Lifecycle, Observability, Verification, and Governance expose architectural boundaries that earlier frameworks often conflate.

这七层往往被混淆成一个模糊的 "infrastructure" 概念。本调查将它们分开，不是因为它们在所有系统中物理分离，而是因为在生产部署中，每一层都有自己的工具栈、由不同的团队负责、面临不同的失效模式。**Observability 和 Governance 在这里作为独立层出现**，因为在生产部署中，每个层都有自己的工具栈，由不同的团队负责——这是与其他框架相比的显著差异。

> 原文："Compared with earlier six-component frameworks, Observability and Governance appear here as independent layers because, in production deployments, each has its own tooling stack and is owned by a different team."

### Claim 3: A broad ecosystem map reveals gaps.

> A systematic mapping of the open-source ecosystem surfaces adoption patterns across sandboxes, protocols, memory systems, orchestrators, observability platforms, benchmarks, and governance stacks.

开源项目映射揭示了一个贯穿全语料库的显著转变：

> "A related shift runs through the corpus: from **agent frameworks**, which package local abstractions (agents, tools, memory, execution loops), to **agent platforms**, which add durable workspaces, identity, observability, evaluation, governance, and human handoff across many runs and many users."

这一转变为 ETCLOVG 分类法提供了直接的生态验证：如果生态系统正在从"轻量框架"走向"平台级基础设施"，那么对 Harness 各层进行独立分类就不仅是理论需求，而是工程实践已经走在前面的验证。

> Evidence：`Observed`。三条 Claim 的原文来自项目主页的 "Three Claims" 节。framework→platform 转变描述来自 "Mapping Open-Source Projects" 节。

---

## 2. ETCLOVG 七层分类法

本调查将 Harness 组织为七层。一个关键的架构划分是：

> "The first four describe the **structural core** of a harness; the last three describe the **control plane** around it."

这种"核心 + 控制面"的二分组本身就是一个重要的设计洞见：前三层（E/T/C）决定了 agent 能看到什么、能用什么工具、在哪里运行——它们是运行时的基础设施输入面；L 层组织这些基础设施在时间上的行为；O/V/G 层约束、监测和验证全部的行为。

### 2.1 Structural Core（结构核心层）

前三层（E, T, C）从三个不同方向满足 agent 执行的基本前提：在哪里运行、用什么工具、能看到什么。

#### E — Execution Environment

决定 agent 代码在哪里运行以及哪些沙箱约束限制它：

- Managed sandboxes、microVMs、code-specialized runtimes
- Computer-use environments、browser sandboxes
- OS-level permission models

**生态密度**：20 个主分配项目。代表项目：Docker 容器方案（managed sandboxes）、gVisor/microVM（OS 级隔离）、browser sandbox（computer-use）。

**设计张力**：隔离程度与便利性之间的权衡。更强的隔离（如 microVM）意味着更高的安全保证和更强的防御，但也意味着更高的开销和对可用工具能力的更多限制。code-specialized runtime（如 Pyodide 等）虽然更快且功能受限，但同时意味着 agent 可以用的工具受到限制。

#### T — Tool Interface & Protocol

规定外部能力如何被描述、发现和调用：

- Protocol standards（MCP、A2A）
- Tool description / registration / selection
- Tool-augmented training
- Session management

**生态密度**：12 个主分配项目。三个核心层（E/T/C）中最低，可能是因为 MCP/A2A 等协议标准仍在快速演进中。

**与 single-agent / multi-agent 边界的关系**：MCP 定位为工具发现/连接协议，A2A（Agent-to-Agent）定位为跨 agent 通信协议。两者被合并在同一层中反映了本 survey 的视角——它们都属于"外部能力如何被描述和调用"，上层不区分是"工具"还是"其他 agent"。

#### C — Context & Memory Management

控制模型在短时、会话级和持久化时间范围内的可见内容：

- Long-horizon context techniques（RAG、压缩、排序）
- Context drift mitigations（窗口管理、上下文刷新）
- Memory hierarchy（短期/长期/共享记忆）

**生态密度**：9 个主分配项目。三个核心层中最低。可能原因：大多数项目的 context/memory 实现是以库或中间件嵌入到大框架中的，而非作为独立的 Harness 层暴露。也就是说，这一层的实践价值可能高于其开源项目计数暗示的密度。

### 2.2 Lifecycle Orchestration 层（结构核心 → 控制面的过渡）

#### L — Lifecycle & Orchestration

组织读写状态的控制流——从单 Agent 内部循环到多 Agent 模式和完整的任务流水线：

- Single-agent inner loop（think-act-observe）
- Multi-agent orchestration patterns
- Task pipelines（issue-to-PR 等）

**生态密度**：47 个主分配项目——ETCLOVG 中密度最高。这可能反映了该层"什么都在里面"的历史：编排是大多数 agent framework 宣称解决的首要问题，且最容易从公开文档中识别。但这种高密度也有另一面——层的边界在这层最模糊。

### 2.3 Control Plane（控制面）

后三层（O, V, G）构成 survey 所说的 "control plane"——不是支撑 agent 执行的基础设施，而是**约束、监测和验证**全部行为的控制系统。

#### O — Observability & Operations

捕获 trace、成本、失败和可靠性信号：

- Tracing platforms（OpenTelemetry 等）
- Agent-specific operations tools（agent logs、会话重放）
- Cost tracking（token 使用分析）
- Unified observability（跨层聚合）

**生态密度**：15 个主分配项目。偏低，表明可观测性在开源 agent 生态中可能被低估为独立层。

#### V — Verification & Evaluation

将任务和 trace 转化为评估、失败归因和回归反馈：

- Benchmark grounding（SWE-bench 等）
- Controlled execution（消融实验框架）
- Multi-level judgement（任务级 + 行为级）
- Deployment-time evaluation loops

**生态密度**：21 个主分配项目。

#### G — Governance & Security

在模型级、系统级和组织级子层上约束行为：

- Permission models（声明式权限策略）
- Lifecycle hooks（审批关卡）
- Component hardening（沙箱加固）
- Declarative constitutions（行为规范）
- Audit infrastructure

**生态密度**：14 个主分配项目。

> Evidence：`Observed`。七层定义和分组来自项目主页的 "The ETCLOVG Taxonomy" 节。"The first four describe the structural core of a harness; the last three describe the control plane around it" 为原文。每一层的具体范围和示例来自该节子段落。生态密度计数来自 "Mapping Open-Source Projects" 节的主分配表格。各层的分析判断（"可能原因是……"）为整合者推断（`Inferred`）。

---

## 3. 三个工程阶段

在介绍了 ETCLOVG 分类法之后，Survey 提供了一个历史定位——这三个阶段为理解**为什么现在需要 Harness 分类法**提供了背景。

> "Read across 2022–2026, agent engineering has gone through a coherent shift in where the marginal effort lands. The three phases overlap in time and concept; they describe what the field has chosen to engineer, not a clean sequence of replacements."

三个阶段**不是**严格的时间序列替换，而是边际精力投向的变化：

- **2022–2024: Prompt Engineering.** "The primary lever is the input prompt text: instructions, few-shot examples, and reasoning templates, all optimized for a single model call."

- **2025: Context Engineering.** "The question shifts from 'what is the input?' to 'what should the model see at each step?' The scope expands to retrieval, compaction, tool-result ranking, and managing context-window saturation across turns."

- **2026–: Harness Engineering.** 第三阶段明确被定位为 Harness Engineering。`main.pdf` 中最关键的判断有三点：第一，研究对象已经从“单次调用的输入”与“每一步该看到什么”继续扩展到**包裹模型运行的整套系统层**；第二，**“Each phase subsumes the previous: harness engineering includes context engineering, which includes prompt engineering.”**；第三，**“The three phases also overlap in time and in concept rather than succeeding one another by clean boundaries.”** 因而 prompt engineering 仍然是今天 harness 实践的一部分，context engineering 也没有消失，而是被纳入更大的系统工程范围。

> Evidence：`Observed`（三个阶段的命名、前两个阶段的原文描述，以及第三阶段“包含前阶段且彼此重叠”的关键判断已可由 `main.pdf` 确认）。

---

## 4. 开源项目映射

### 4.1 编码方法论

调查的实证核心是一个系统的开源项目映射。其方法设计本身构成了论文的重要贡献：

1. **语料库构建**：从 GitHub、论文、精选清单、软件包注册中心和公司工程博客等多个渠道收集候选项目。
2. **包含标准**：只纳入有公开制品（README、文档、论文、示例、发布说明）的项目；编码依据是这些公开制品，而不是作者的私有知识。
3. **Multi-label 编码**：项目的**主层**标记其最核心的机制；**副层**只在该公开文档暴露了独立能力时才被分配。
4. **证据锚定**：对每一个候选项目，记录项目名称、URL、制品类型、来源类型、可用性状态、发布年份、GitHub 元数据，以及用于 ETCLOVG 编码的公开证据。

### 4.2 项目分布

**调查依赖两层不同的分类结构，不应混淆：**

- **ETCLOVG 七层**（E/T/C/L/O/V/G）是调查的概念分类系统，用于分析 Harness 的架构维度。各层的主分配计数为快照值。
- **awesome-agent-harness 九类**是该调查的**实践生态清单**，用于组织开源资源。其类别与 ETCLOVG 七层不完全对应（增加了 "Reference Harness Implementations" 和 "Essential Readings & Ecosystem Maps" 两个跨层类别）。

**ETCLOVG 主分配计数**（来自项目主页 Mapping 节）：

| 层 | 主分配 |
|----|--------|
| Lifecycle & Orchestration | 47 |
| Execution Substrates & Sandboxing | 26 |
| Verification & Evaluation | 21 |
| Observability & Operations | 15 |
| Governance & Security | 14 |
| Tool Interface & Protocol | 12 |
| Context & Memory Management | 9 |

**awesome-agent-harness 生态清单**（来自 README，2026-06-01 快照，共 251 条）：

| 类别 | 条目 |
|------|------|
| Reference Harness Implementations | 65 |
| Harness Architecture & Orchestration | 39 |
| Evaluation Harnesses & Benchmarks | 27 |
| Execution Substrates & Sandboxing | 26 |
| Guardrails, Security & Governance | 18 |
| Protocols, Tool Interfaces & Agent Contracts | 16 |
| Context & Working-State Engineering | 15 |
| Observability & Reliability Operations | 14 |
| Essential Readings & Ecosystem Maps | 31 |

**两者之间的差异值得注意**：Reference Harness Implementations（65 个项目）在 ETCLOVG 七层中没有对应——这些项目是全栈实现参考，不是按单层分类的。Essential Readings & Ecosystem Maps（31 个项目）同样如此。这些跨层项目表明：Harness 工程不仅是一个概念分类问题，也是一个实践集成问题——实际部署的项目往往需要跨越多个层，而这些"全栈"项目是生态中最常见的形态之一。

> Evidence：`Observed`。方法论描述来自项目主页 Mapping 节。ETCLOVG 主分配计数来自同一页面。awesome-agent-harness 分类计数来自 README "Category Overview" 表和 `data/projects.yaml`（截至 2026-06-01 共 251 条，GitHub 条目 224/251）。

---

## 5. 五个开放问题

> "Five questions remain open across the taxonomy. Each follows from the cross-layer synthesis rather than from a single ETCLOVG layer in isolation."

这一节的 framing 现在已经可以直接由 `main.pdf` 支撑，而不必只依赖项目主页。作者先写道：**“The open problems collected here follow from the binding-constraint thesis and the cross-layer synthesis...”**，随后又强调：**“Rather than treating the seven ETCLOVG layers as independent component lists, this section asks where the whole harness remains scientifically under-specified.”** 也就是说，这五个问题不是把七层逐层补缺，而是在问：当 harness 变成长期运行的控制系统后，整套系统还缺哪些成熟答案。

同一段还给出一个总括：**“The central pattern is that agent harnesses are becoming long-running control systems...”**，而当前缺口集中在 execution substrate hardening、state preservation、failure diagnosis、responsibility transfer 和 harness 更新五类问题上。

结合 `main.pdf` 当前可提取内容，五个开放问题的标题级表述已经可以较可靠地恢复为：

1. **Hardening and Scaling Execution Environments**
2. **Maintaining Reliable State in Long-Running Agents**
3. **Diagnosing Failures from Agent Traces**
4. **Standard Handoffs Across Agents, Tools, and Humans**
5. **Keeping Harnesses Useful as Models Improve**

按当前可确认信息，它们分别关注：

1. **如何硬化并扩展执行环境**：随着 agent 变成长时程控制系统，execution substrate 的安全性、隔离性与可扩展性仍缺乏成熟答案。`main.pdf` 现在已可确认一个关键 framing：**“This design space creates a recurring tension between capability, control, and cost.”** 紧接着作者把不同 workload 拉向不同 substrate：高风险任务推动 microVM 与 managed cloud，交互式本地工作流偏向轻量权限边界，大规模训练与评测偏向可快速重置的环境。随后又明确提出开放点：**“The open problem is to make the runtime substrate both measurable and composable.”** 因而 runtime defense、经济性、可移植性、bundle vs compose 设计和跨层耦合，不是孤立的执行层细节。
2. **如何保持可靠状态**：长时程运行中的状态保存、恢复与一致性维护仍是基础难题。按当前已恢复内容，作者关注的核心方向不是单纯“如何塞进更多 token”，而更像是**如何让 agent 的工作状态在长时程内持续对齐真实任务状态**；压缩、检索、清理 tool results 和上下文排序可视为这一问题的典型机制，但这里的具体论证链条仍需继续回到 PDF 原文逐段复核。
3. **如何从 agent traces 中诊断失败**：系统虽然越来越能产生日志和轨迹，但如何把 trace 稳定转化为失败归因与改进反馈，仍未收敛。按当前已恢复内容，这一节大致指向对 final-score-centric evaluation 的不满，以及对多源失败因素（模型、工具、环境、上下文、评测器、编排）的区分需求；但这些点之间在原文中是如何组织展开的，仍未完全恢复。
4. **如何在 agents、tools 与 humans 之间建立标准化 handoff**：当前可较稳妥确认的是，这一节关心的不是局部协议是否存在，而是跨层 handoff contract 是否足够完整；责任转移时，作者大概率关注的不只是文本摘要，还包括 intent、constraints、permissions、artifacts、provenance、budget state 一类结构化信息。不过，这里仍应视为基于已恢复方向的保守重述，而非完整原文复现。
5. **如何在模型能力变化下保持 harness 有效**：当底层模型能力持续变化时，harness 的分层、权限、验证与控制逻辑如何随之更新，仍是开放问题。当前可较稳妥确认的关键提醒是，不应假定 harness 设计会单调地走向“更多脚手架”；每个 wrapper、reset、verifier、planner、memory rule 和 permission gate 都编码了“模型当前还不能可靠完成什么”的假设，因此随着模型变化，这些 interventions 需要重新估计。但这一节的完整论证仍待原文进一步复核。

这意味着，作者眼中的 Harness 工程难点不只是“七层各自是否完善”，而更在于这些层在长期运行、失败诊断、责任转移和能力演化中的协同问题。

> Evidence：`Observed`（五个开放问题的存在、cross-layer synthesis 定位、五个问题的标题级表述，以及第 1 个问题的关键 framing 句来自 `main.pdf` 当前可提取内容）；`Inferred`（第 2–5 个问题当前的方向性重述）；`Unverified`（第 2–5 个问题的逐条原文展开论证仍未完全恢复）。

---

## 6. 与当前仓库主题的关系

本调查与 `agent-system-modeling/` 当前主题直接相关，因为它提供的不是单个 runtime 配置定义，而是一个**面向生态比较的 Harness 分类系统**：ETCLOVG 七层、项目映射方法、主分配计数，以及对 framework→platform 转变的观察，都能帮助当前仓库理解 Harness 在开源生态中的实际分布与缺口。

但本文件的重点仍是**忠于本 survey 的解读**，不在这里继续展开“三篇 Harness 论文如何分工互补”“ETCLOVG 七层与 11 组件如何映射”这类跨论文整合判断。

这些超出单篇调查解读范围的对照、映射与主题级待核验问题，继续保留在：

- `./candidates.md`：跟踪 Harness 方向候选研究对象与阅读顺序
- `../../backlog.md:31`：记录 `Agent Harness as System Model` 的主题级待核验问题

---

## 🔗 相关资源

- [2605.13357_AI_Harness_Engineering.md](./2605.13357_AI_Harness_Engineering.md)：本仓库中对应的 AI Harness Engineering 论文笔记。
- [Externalization_2604.08224.md](../cognitive-architectures/Externalization_2604.08224.md)：外部化统一框架的完整笔记。
- [candidates.md](./candidates.md)：本调查在候选项中的阅读优先级定位。
- [Awesome Agent Harness](https://github.com/Picrew/awesome-agent-harness)：250+ 项目的实时生态清单。

## 材料来源

- PDF 论文：`https://picrew.github.io/LLM-Harness/main.pdf`（当前最高优先级事实来源）
- 项目主页：`https://picrew.github.io/LLM-Harness/`（纯 HTML/CSS 静态页面，带内联 BibTeX `@misc{li2026agentharness`）
- Awesome 清单：`https://github.com/Picrew/awesome-agent-harness`（251 条目，224 GitHub，9 分类，最后验证 2026-06-01）
- 数据文件：`https://raw.githubusercontent.com/Picrew/awesome-agent-harness/main/data/projects.yaml`

## 警告说明

**本调查（Li et al.）与以下同名/同主题工作不同，不应混淆：**
- Meng et al., "Agent Harness for Large Language Model Agents: A Survey"（Preprints 2026）——提出 6 组件 ECTSLV 分类法，非 ETCLOVG。
- Zhong & Zhu, "AI Harness Engineering: A Runtime Substrate..."（arXiv 2605.13357）——概念框架论文，与本调查互补，不是同一工作。
- "Code as Agent Harness"（arXiv 2605.18747）——以代码为中心的视角，非分类法调查。

## Evidence

- Status: `Observed`（Claims、ETCLOVG 七层定义、三个工程阶段的核心判断、Mapping Methodology、生态计数、五个开放问题的存在、标题级表述、section framing，以及第 1 个开放问题的关键原文论证）；`Inferred`（层间分析判断，以及第 2–5 个开放问题当前基于已恢复方向所做的保守重述）；`Unverified`（第 2–5 个开放问题的逐条原文展开论证，以及第三工程阶段更完整的原文展开仍未完全恢复）。
- Sources: `main.pdf`（最高优先级来源）、`https://picrew.github.io/LLM-Harness/`（项目主页）、`awesome-agent-harness` README 与 `data/projects.yaml`。
- Trace: 从 `candidates.md` 的“第二优先级：核验 ETCLOVG 分类综述”出发，完成该调查的笔记；主题级跨论文判断继续留在 `candidates.md` 与 `../../backlog.md:31`。
- Needs: 后续应优先继续从 `main.pdf` 复核第 2–5 个开放问题与第三工程阶段的逐条原文展开论证；在未恢复更完整原文前，不应把这些小节写得比当前证据更确定。此外，ETCLOVG 的领域适用边界（对话 Agent、具身 Agent）需要更多案例验证。

---

*最后更新: 2026-06-08*