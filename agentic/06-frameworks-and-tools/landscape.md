# Frameworks And Tools Landscape

`landscape.md` 解释 `06-frameworks-and-tools/` 为什么采用当前结构，以及各子目录、相邻层级与辅助元信息文件如何长期协作。它不替代 `overview.md` 的人类向主线，也不替代 `backlog.md`、`conflict.md` 或未来可能出现的 `roadmap.md`。

## 一、为什么本目录采用对象视角

`06-frameworks-and-tools/` 的核心任务，不是重新定义 Agent 的底层能力，而是观察现实框架、工具、项目和产品如何把这些能力实现、组合与产品化。

因此，本目录采用对象视角，而不是在 `agentic/` 体系内部再建一套按 planning / memory / tool-use / multi-agent / environment / evaluation 切分的能力型目录。这样做的目的有两点：

- 避免与 `02-single-agent/`、`03-multi-agent/`、`05-environments/`、`07-evaluation/` 形成结构性重叠。
- 让框架、工具、项目案例与 Skill/Tool 系统保留对象级的完整阅读体验，而不是被拆散到多个能力目录中。

## 二、主干子目录的结构角色

当前主干子目录承担的是不同的对象角色，而不是不同的底层能力：

- `01-frameworks/`：通用框架、SDK、编排框架等可复用底座对象
- `02-coding-agents-and-tools/`：面向软件工程场景的编码 Agent 工具、产品与平台
- `03-project-studies/`：完整 Agent 系统与开源项目的系统化案例
- `04-skill-and-tool-systems/`：Skill、Tool、插件、能力注册与工具生态等工程实现对象
- `05-comparisons/`：横向比较、选型矩阵与生态地图

这些编号同时提供一个轻量阅读顺序，但更重要的作用是稳定主干分组。它们不要求其他 `agentic/` 二级领域也采用相同的编号式三级目录。

## 三、与其他层级的边界

`06-frameworks-and-tools/` 必须坚持工程实现视角。边界判断的核心不是"这个概念属于哪个技术名词"，而是"当前文档是在解释能力原理，还是在解释对象如何实现、组合和选型"。

主归属判断：

- 回答"这个能力/模式/原理是什么"时，回到 `02-single-agent/`、`03-multi-agent/`、`05-environments/`、`07-evaluation/`
- 回答"这个框架/工具/项目具体怎么实现、怎么集成、怎么选型"时，留在 `06-frameworks-and-tools/`
- 横跨多层的问题采用主归属 + 交叉引用，不在多个目录重复维护同一份对象级分析

与相邻层级的典型分工：

| 内容 | 主归属 | 在 `06` 中的角色 |
|------|--------|------------------|
| planning / memory / tool-use / reflection | `02-single-agent/` | 只在具体框架或项目分析中作为实现章节出现 |
| multi-agent collaboration / coordination | `03-multi-agent/` | 只分析具体框架如何支持多 Agent 协作 |
| sandbox / browser / code execution environment | `05-environments/` | 只分析具体工具或项目如何接入环境 |
| benchmark / observability / evaluation | `07-evaluation/` | 只分析具体工具链或产品如何支持评估与调试 |
| framework / tool / product / project | `06-frameworks-and-tools/` | 本目录主线 |

## 四、子目录归属规则

### 4.1 开源项目研究组织原则

系统性开源项目研究优先集中放在 `03-project-studies/<project>/`，提供项目级的完整阅读体验；其他专题目录只在需要支撑某个机制、证据或对比时克制回填，不复制完整项目分析。

例如 OpenHands 的整体研究应集中到 `03-project-studies/openhands/`；`05-environments/` 只回填 workspace / runtime / sandbox 相关证据，`02-single-agent/` 或 `03-multi-agent/` 只在确有专题需要时引用对应机制。

### 4.2 `02-coding-agents-and-tools/` 与 `03-project-studies/` 的归属规则

- `02-coding-agents-and-tools/` 放**面向用户的编码 Agent / coding tool / 产品对象视角**：如何使用、如何选型、产品定位和工作流整合。
- `03-project-studies/` 放**系统性开源项目研究、架构拆解和实现机制视角**：如何拆解、如何理解内部机制。
- 当同一对象同时具备两种视角时（例如 OpenHands 既是编码 Agent 产品又是可系统拆解的开源项目），**系统性研究主归属放 `03-project-studies/<project>/`**，`02-coding-agents-and-tools/` 中只保留简要入口或交叉引用。
- `02-coding-agents-and-tools/` 下不复制 `03-project-studies/` 中已有的架构分析内容。如果需要跨类对象比较，应进入 `05-comparisons/`。

### 4.3 框架对象的边界规则

- `AutoGen`、`Semantic Kernel` 与 `Microsoft Agent Framework` 的对象边界应分开写：前两者是历史来源与独立对象，后者是后继整合框架，不宜直接合并成同一对象下结论。
- `OpenAI Agents SDK / Responses API` 更适合作为框架候选对象继续补证：官方 docs 已公开多个与 agent orchestration 相关的模块入口，但当前目录尚未形成可直接回写主线的对象专题；涉及 handoffs、sessions、results/state、MCP、tracing、guardrails / human review、sandbox agents 的具体边界，以 `backlog.md` 与 `conflict.md` 中的待核验记录为准，不应把外部归纳的"七层架构"写成官方命名。

## 五、调研吸收与回流原则

临时调研稿或外部资料回流时，先判断它是对象、比较、候选线索，还是底层能力主题：

- 成熟框架、工具、项目对象进入 `01-frameworks/`、`02-coding-agents-and-tools/`、`03-project-studies/` 或 `04-skill-and-tool-systems/`
- 对象之间的横向比较进入 `05-comparisons/`
- 前沿但尚不稳定的对象进入 `backlog.md`，不直接创建主干目录
- 底层能力主题回流到 `02/03/05/07` 的相应目录，`06` 只保留工程实现引用
- 证据不足或来源不稳定的结论不进入 `overview.md` 作为定论，而留在 `backlog.md`、`conflict.md` 或具体对象文档的待核验区域

## 六、与其他元信息文件的非重叠边界

`landscape.md` 解释的是结构原则与文档体系协作，不承接其他元信息文件的职责：

- `backlog.md`：记录内容缺口、前沿观察和下一步值得补的对象或问题
- `candidates.md`：记录候选研究对象队列（本目录当前主要由 `backlog.md` 承担这部分工作）
- `conflict.md`：记录事实冲突、证据不足与待核验问题
- `roadmap.md`：如未来需要，负责明确的顺序型建设计划或阅读路径

因此，`landscape.md` 不应变成：

- 候选对象列表
- 待核验问题账本
- 证据不足项堆栈
- 明确的优先级任务清单
- 另一篇扩展版 `overview.md`

它的职责只到"为什么这样组织、各部分如何协作、材料如何回流"为止。

---

*最后更新: 2026-06-16*
