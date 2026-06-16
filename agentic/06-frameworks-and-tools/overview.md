# AI Agent 框架与工具生态综述

`06-frameworks-and-tools` 关注 Agent 工程生态中的具体对象：框架、SDK、编排平台、编码工具、完整项目案例、Skill/Tool 系统与横向对比。它不负责重新讲解 planning、memory、tool-use、multi-agent、environment、evaluation 等底层能力，而是观察这些能力如何在真实框架和工具中被实现、组合和产品化。

---

## 一、定位

本目录回答的问题是：

- 有哪些值得长期跟踪的 Agent 框架、工具、产品和开源项目？
- 它们分别代表了哪些工程实现路线？
- 它们如何组合 `agentic/` 其他层级中的能力？
- 面向实际系统建设时，应该如何选型、比较和拆解这些对象？

因此，本目录采用**对象视角**组织，而不是按底层能力重新建一套 taxonomy。

---

## 二、主干对象类型

### 2.1 通用框架与编排框架

这一类对象提供 Agent 构建、工具绑定、状态管理、工作流编排、多 Agent 组织等工程抽象。它们通常不是单一能力点，而是把 planning、tool-use、memory、human-in-the-loop、multi-agent collaboration 等能力组合成可复用框架。

代表对象：

- `LangChain / LangGraph`
- `AutoGen`
- `CrewAI`
- `Semantic Kernel`
- `Microsoft Agent Framework`
- `DSPy`（更偏 LM program / prompt optimization，但与 Agent orchestration 有交集）

长期观察重点：

- 单 Agent 抽象与多 Agent 抽象如何统一
- workflow / graph runtime 是否成为生产级 Agent 的主流底座
- 框架如何处理状态、checkpoint、human-in-the-loop 和工具权限
- 大厂框架整合是否会改变生态格局

这些框架之间的对象边界，以及 `OpenAI Agents SDK / Responses API` 当前仍需谨慎区分的范围，见 `landscape.md` §4.3。

### 2.2 编码智能体工具

Coding Agent / SWE Agent 已经形成相对独立的问题域。它们面向软件工程任务，依赖 repo understanding、code localization、patch generation、test execution、IDE/CLI 集成和 PR workflow 等专门化能力。

代表对象：

- `Claude Code`
- `Codex`
- `OpenHands / OpenDevin`
- `GitHub Copilot Agent Mode / GitHub Next Ace`
- `SWE-agent`
- `Cline` 等 IDE/CLI 型编码 Agent

长期观察重点：

- 从代码补全到仓库级任务委托的演化
- 代码定位、编辑、验证、回滚的闭环工作流
- 与 Docker、Git、测试框架、MCP、IDE 的集成方式
- 与 `07-evaluation/swe-benchmarks/` 中 SWE-bench、SWE-Gym、SWE Atlas 等评估体系的关系

### 2.3 完整项目案例

这一类对象不是单纯 SDK，也不是单一工具，而是完整 Agent 系统或开源项目。它们的价值在于展示一个真实系统如何把 Agent 的各层能力组装成可运行的工程产品。

代表对象：

- `Hermes Agent`
- `OpenClaw`

长期观察重点：

- Gateway、Agent runtime、Skill、Memory、Context、Automation 等模块如何协同
- 系统如何处理多渠道接入、多 Agent 隔离、权限、安全、部署与运维
- 与根目录中 `02/03/05/07` 各层能力的映射关系

### 2.4 Skill 与 Tool 系统

这一类对象关注 Skill、Tool、插件、能力注册、工具生态与可复用动作单元。它与 `02-single-agent/tool-use/` 的区别在于：`02` 讨论工具使用的原理和模式，`06` 讨论具体 Skill / Tool 系统如何工程化实现。

长期观察重点：

- Skill 的声明、加载、隔离、验证和组合方式
- Tool registry、tool discovery、tool generation 的工程形态
- Skill 是否应被视为可验证制品，而不是普通 prompt 附件
- 与 MCP、function calling、工具权限控制的关系

### 2.5 对比与选型

这部分用于横向比较框架、工具、项目和产品，不重复建设底层能力理论。

适合沉淀：

- 框架选型矩阵
- 编码工具对比
- 多 Agent 编排实现对比
- 生态地图
- 成熟度、可维护性、适用场景和风险对比

---

如需理解本目录为什么按当前结构组织、与其他层级如何分工、以及新材料如何吸收进体系，转到 `landscape.md`。

---

*最后更新: 2026-06-16*
