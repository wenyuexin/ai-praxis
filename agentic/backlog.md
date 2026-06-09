# Agentic Backlog

> 适用范围：`agentic/` 顶层领域的跨目录内容缺口、边界问题与待补主线
> 使用原则：本文件只记录已经识别但尚未充分覆盖的主题缺口，不是任务列表，也不代表相关结论已进入主干定论

## 一、收录规则

适合进入本文件的内容通常满足以下条件之一：

- 跨越 `01-foundations/` 到 `07-evaluation/` 多个目录，难以只放在单一子目录的 backlog 中。
- 会影响 Agentic 顶层知识体系的组织边界、概念分工或主线阅读路径。
- 已在多个子目录中反复出现，但还缺少顶层视角的统一问题意识。
- 当前证据不足以写成 `overview.md` 或正文专题，但值得持续跟踪。

不适合进入本文件的内容：

- 具体论文、仓库、产品、benchmark 或协议对象；这类对象应放入对应子目录的 `candidates.md` 或项目 / 框架研究目录。
- 已经有明确单一归属的主题缺口；这类问题优先放入对应子目录的 `backlog.md`。
- 仅来自外部 AI 调研、网页搜索或二手摘要且尚未核验的结论。
- 具体实现设计、产品路线或项目管理事项。

---

## 二、P0：顶层高优先级缺口

### 2.1 Agentic 顶层 `overview.md`

- **关联目录**：`README.md`、`01-foundations/`、`02-single-agent/`、`03-multi-agent/`、`05-environments/`、`07-evaluation/`
- **为什么重要**：`README.md` 已承担目录定向和边界说明，但 `agentic/` 顶层仍需要一个面向读者的领域综述，用来回答 Agentic AI 的整体脉络、核心矛盾、能力边界和跨层关系。
- **当前状态**：`overview.md` 已形成早期版本，基于现有主干文档整理了概念、单智能体、多智能体、人机交互、环境、框架工具与评估之间的跨层关系；但由于 `agentic/` 仍存在主题缺失，该文件只能视为早期综述，后续需要随子目录专题继续修订。
- **Evidence need**：继续基于已有子目录综述和已核验案例做归纳；外部综述或行业判断只能作为线索，不能直接写成主线定论。
- **后续方向**：补齐人机交互、评估、浏览器环境、仿真环境和跨层对比专题后，再回头更新顶层综述。

### 2.2 Agentic 核心矛盾的跨层表达

- **关联目录**：`01-foundations/overview.md`、`02-single-agent/overview.md`、`03-multi-agent/overview.md`、`05-environments/overview.md`、`07-evaluation/overview.md`
- **为什么重要**：Agentic AI 的关键问题不是单一能力清单，而是 autonomy vs control、planning vs execution、tool capability vs tool reliability、coordination gain vs coordination cost、connection recovery vs filesystem reconstruction、benchmark score vs real utility 等跨层矛盾。
- **当前状态**：这些矛盾已分散出现在各子目录，但缺少顶层问题地图来避免读者把 Agentic 简化成“更多工具 + 更多自治 + 更多 Agent”。
- **Evidence need**：需要从已有主干文档和已形成的 case-study / framework-study 中抽取稳定边界；对仍在补证的性能、安全、恢复和评估结论应保留 `Observed / Inferred / Unverified` 口径。
- **建议产物**：可先在顶层 `overview.md` 中用短节承接；若材料继续增厚，再考虑拆成专题正文。

### 2.3 Framework / Tool 案例向通用主线的回填边界

- **关联目录**：`06-frameworks-and-tools/`、`05-environments/`、`02-single-agent/`、`03-multi-agent/`、`07-evaluation/`
- **为什么重要**：OpenHands、SWE-agent、LangGraph 等对象研究已经形成较多机制证据，但如果回填边界不清，容易把单个系统的实现路径误写成 Agentic 通用结论。
- **当前状态**：`06-frameworks-and-tools/` 已有对象内聚的研究入口，`05-environments/` 已进行少量稳定回填；但顶层仍需要明确“对象研究 -> 子领域主线 -> 顶层综述”的证据流动边界。
- **Evidence need**：只有跨对象稳定出现、且有源码 / 官方文档 / 论文支撑的结论，才适合进入顶层；单对象观察优先留在对象目录或子目录 backlog。
- **建议产物**：在顶层 `overview.md` 或本 backlog 中保留回填原则；不为每个案例在顶层重复写项目介绍。

### 2.4 Agentic 恢复语义的统一停线

- **关联目录**：`05-environments/`、`06-frameworks-and-tools/01-frameworks/langgraph/`、`06-frameworks-and-tools/03-project-studies/openhands/`、`06-frameworks-and-tools/03-project-studies/swe-agent/`、`07-evaluation/observability-and-debugging/`
- **为什么重要**：conversation restore、runtime resume、workflow checkpoint、trajectory replay、workspace checkpoint、filesystem reconstruction、external side-effect recovery 经常被混写；如果没有顶层停线，恢复能力会被过度概括。
- **当前状态**：`05-environments/overview.md` 已写下 connection recovery 不等于 filesystem reconstruction，LangGraph / OpenHands / SWE-agent 的对照也已提供稳定边界；但顶层尚未把这组恢复语义放入 Agentic 全局问题地图。
- **Evidence need**：继续区分 `Verified`、`Observed`、`Inferred`、`Unverified`，尤其不能把 graph state checkpoint、trajectory replay 或 sandbox resume 外推为完整 workspace 文件系统恢复。
- **建议产物**：先作为顶层 `overview.md` 的核心 stop-line；后续如果 `07-evaluation/observability-and-debugging/` 补齐 failure attribution，可与其联动。

### 2.5 Agent Adapter 与 Orchestrator 契约

- **关联目录**：`02-single-agent/tool-use/`、`03-multi-agent/coordination/`、`05-environments/code-execution-environments/`、`06-frameworks-and-tools/`、`07-evaluation/observability-and-debugging/`
- **为什么重要**：当系统需要调度内部 Agent、外部 Agent、工具执行和长任务工作流时，共性问题不是某个 provider 如何接入，而是任务输入、执行边界、状态、产物、错误、取消、重试和 trace 如何被统一表达。
- **当前状态**：已用 `MCP` 和 `LangGraph` 完成第一轮窄口径样本核验，并在 `06-frameworks-and-tools/05-comparisons/orchestration-implementations.md` 形成 `TaskSpec / ExecutionSpec / ResultSpec / ErrorSpec / AdapterSpec` 维度初稿；当前只能支持协议层与 workflow runtime 层的保守抽象，不能外推为统一行业标准或完整 orchestrator contract。
- **Evidence need**：继续比较不同 agent framework、coding agent、workflow framework 与工具协议如何描述 capability、session、workspace、trace、cost、abort、retry 和 result normalization；单对象机制只能先保留为对象观察。下一步优先补 OpenAI Agents SDK / Responses API、OpenHands 与 SWE-agent 的 session / workspace / artifact / trace / recovery 边界。
- **建议产物**：继续迭代 `06-frameworks-and-tools/05-comparisons/orchestration-implementations.md`，每引入一个对象都先补对象内 `Observed` 证据；顶层只保留跨目录问题推进状态，对象细节留在 `06-frameworks-and-tools/`。

### 2.6 Tool Executor 与 Skill 制品边界

- **关联目录**：`02-single-agent/tool-use/`、`05-environments/sandboxing-and-safety/`、`06-frameworks-and-tools/04-skill-and-tool-systems/`、`07-evaluation/safety-and-robustness/`
- **为什么重要**：工具能否扩展、审计和隔离，取决于 Tool Executor、Skill manifest、依赖声明、副作用声明、权限策略和失败恢复是否被视为系统边界，而不是临时 prompt 或函数包装。
- **当前状态**：`02-single-agent/` 已有 tool reliability 相关主题，`06-frameworks-and-tools/` 关注 Skill / Tool 系统对象，但顶层仍缺少 Tool 与 Skill 如何形成可治理制品的跨层问题。
- **Evidence need**：需要比较内置可信工具、项目级工具、第三方工具和高风险代码执行工具在真实框架中的执行路径、权限模型、隔离策略和 trace 形态。
- **建议产物**：形成 Tool Executor 分层表和 Skill manifest 最小字段清单，优先作为跨目录研究问题而非实现方案。

---

## 三、P1：值得持续跟踪的跨目录专题

### 3.1 Autonomy、permission 与 human control 的统一视角

- **关联目录**：`04-human-agent-interaction/`、`05-environments/sandboxing-and-safety/`、`07-evaluation/safety-and-robustness/`
- **为什么值得关注**：自治能力、人类确认、权限策略、治理模式与安全评估并不是相互独立的主题；它们共同决定系统能否在真实任务中可控地运行。
- **当前状态**：`04-human-agent-interaction/backlog.md` 已记录 delegation / takeover / trust 相关缺口，`05-environments/` 已形成 permission / execution / recovery / governance 的安全组合；但跨目录统一表达仍不足。
- **不宜直接定论的原因**：不同产品形态、任务风险等级和交互模式差异很大，当前不适合写成单一标准。

### 3.2 Memory、workspace 与 trace 的状态边界

- **关联目录**：`02-single-agent/memory/`、`05-environments/code-execution-environments/`、`07-evaluation/observability-and-debugging/`
- **为什么值得关注**：Agent 系统中的状态可能来自 model context、external memory、workspace artifacts、execution trace、checkpoint 或 evaluation logs；这些状态的读写、污染、恢复和归因边界需要统一辨析。
- **当前状态**：`02-single-agent/backlog.md` 已记录 memory writeback 与 state persistence 问题，`05-environments/` 已补 workspace / checkpoint / traceability 专题，`07-evaluation/` 仍缺 failure attribution 正文。
- **建议推进方式**：优先在各子目录补证，再由顶层综述做轻量归纳。

### 3.3 Single-Agent 与 Multi-Agent 的收益边界

- **关联目录**：`02-single-agent/`、`03-multi-agent/`、`06-frameworks-and-tools/05-comparisons/`、`07-evaluation/agent-benchmarks/`
- **为什么值得关注**：多智能体并不天然优于单智能体；任务分解、专业化、通信成本、状态共享和评测基线都会影响结论。
- **当前状态**：`03-multi-agent/coordination/when-multi-agent-helps.md` 已形成第一版专题，`03-multi-agent/backlog.md` 仍保留 hidden cost、specialization granularity 与 benchmark interpretation 等缺口。
- **Evidence need**：需要更多 task type、baseline strength、coordination overhead 与 benchmark setting 的对照证据。

### 3.4 Agentic 评估如何回流到系统改进

- **关联目录**：`07-evaluation/`、`02-single-agent/`、`03-multi-agent/`、`05-environments/`、`06-frameworks-and-tools/`
- **为什么值得关注**：评估如果只停留在分数或排行榜，无法指导 tool、memory、environment、coordination、human control 的改进。
- **当前状态**：`07-evaluation/backlog.md` 已记录 failure attribution、partial completion、benchmark transferability 与 human evaluation rubrics；但顶层还缺“评估结果如何定位到系统层”的问题意识。
- **建议产物**：等 `07-evaluation/observability-and-debugging/failure-attribution.md` 或相关专题成形后，再回填顶层综述。

### 3.5 Cross-domain 边界：Agentic RAG、LLM Serving 与具身环境

- **关联目录**：`README.md`、`../rag/`、`../llm/`、`../embodied-intelligence/`
- **为什么值得关注**：`agentic/README.md` 已标注 Agentic RAG、LLM 推理优化和具身智能的外部归属，但随着内容增加，交叉主题可能再次回流并造成落位混乱。
- **当前状态**：当前边界说明足以支撑目录落位，但仍缺持续维护的交叉主题判断清单。
- **建议推进方式**：优先在相关主题目录建立主视角；`agentic/` 只保留与 Agent 系统结构直接相关的少量桥接说明。

### 3.6 项目级工具的运行时隔离与 workspace 契约

- **关联目录**：`05-environments/code-execution-environments/`、`05-environments/sandboxing-and-safety/`、`06-frameworks-and-tools/03-project-studies/`、`07-evaluation/observability-and-debugging/`
- **为什么值得关注**：项目级工具、长任务 worker 和代码执行能力常常需要在 subprocess、virtual environment、container 或 remote runtime 之间取舍；隔离方式会影响依赖、文件产物、权限、trace、timeout 和 cleanup。
- **当前状态**：环境层已有 workspace / checkpoint / traceability 主线，OpenHands、SWE-agent 和 LangGraph 也提供了对照样本，但还缺少面向项目级工具执行的窄口径契约总结。
- **建议推进方式**：优先研究 subprocess / virtual environment / workspace artifact 契约，再扩展到容器级或远程运行时隔离；避免把环境隔离直接等同于完整系统恢复。

---

## 四、P2：保留观察线索

### 4.1 Agentic 系统分层术语是否会收敛

- **关联目录**：`01-foundations/`、`06-frameworks-and-tools/`
- **说明**：harness、runtime substrate、orchestrator、control plane、execution environment、agent framework 等术语目前仍有漂移。应继续观察论文、官方文档和开源项目是否形成更稳定的层级语言。

### 4.2 Agentic workflow 与传统 workflow automation 的分界

- **关联目录**：`01-foundations/definition-and-taxonomy/`、`02-single-agent/agent-vs-tool-workflow-boundary.md`
- **说明**：随着更多 workflow 系统加入 LLM、tool use 和 human approval，传统 workflow automation 与 agentic workflow 的边界可能继续变化。

### 4.3 长任务 Agent 的可靠性组合

- **关联目录**：`02-single-agent/`、`04-human-agent-interaction/`、`05-environments/`、`07-evaluation/`
- **说明**：长任务可靠性通常同时依赖 planning、memory、workspace、permission、recovery、human takeover 与 evaluation loop；目前各层已有局部材料，但顶层组合框架仍需长期观察。

---

## 五、与子目录 backlog 的关系

- `01-foundations/backlog.md`：承接定义、分类、认知架构、系统建模等概念底座缺口。
- `02-single-agent/backlog.md`：承接规划、记忆、工具使用、反思、架构模式等单智能体能力缺口。
- `03-multi-agent/backlog.md`：承接协调拓扑、失败模式、收益边界、通信与组织等多智能体缺口。
- `04-human-agent-interaction/backlog.md`：承接人在回路、委托控制、交互表面、信任与对齐等缺口。
- `05-environments/backlog.md`：承接执行环境、权限、安全、workspace、traceability、recovery 等环境层缺口。
- `06-frameworks-and-tools/backlog.md`：承接框架、工具、产品、项目案例和前沿对象观察；对象队列过多时应进一步拆入 candidates 或对象目录。
- `07-evaluation/backlog.md`：承接任务完成度、benchmark、人工评估、安全鲁棒与可观测调试等评估缺口。

如果某个问题已经能明确归入单一子目录，应优先移动到对应子目录 `backlog.md`；顶层只保留跨目录、跨层级或会影响 Agentic 总体理解的问题。

---

## 六、维护提醒

- 本文件不追踪责任人、deadline 或任务完成状态。
- 当某条缺口获得足够证据并形成正文专题后，应从本文件中降级为“已形成主干，继续补证”或移除。
- 当某条缺口主要变成具体研究对象时，应转入对应目录的 `candidates.md` 或项目 / 框架研究目录。
- 回流顶层综述时必须保守：未验证材料不写成主线定论，单对象观察不外推为通用结论。
