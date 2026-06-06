# Frameworks and Tools（框架与工具）

本目录用于沉淀 Agent 工程生态中的具体对象：通用框架、编码工具、完整项目案例、Skill/Tool 系统与横向对比。

## 边界说明

`06-frameworks-and-tools/` 不负责重新讲解 Agent 的底层能力体系，而负责分析现实框架、工具和项目如何实现、组合、产品化这些能力。

| 内容 | 主归属 | 在本目录中的角色 |
|------|--------|------------------|
| planning / memory / tool-use / reflection | `../02-single-agent/` | 只在具体框架或项目分析中作为实现章节出现 |
| multi-agent collaboration / coordination | `../03-multi-agent/` | 只分析具体框架如何支持多 Agent 协作 |
| sandbox / browser / code execution environment | `../05-environments/` | 只分析具体工具或项目如何接入环境 |
| benchmark / observability / evaluation | `../07-evaluation/` | 只分析具体工具链或产品如何支持评估与调试 |
| framework / tool / product / project | `./` | 本目录主线 |

## 目录结构

`06-frameworks-and-tools/` 下的编号表达本目录内对象类型的阅读顺序与稳定分组，不代表其他 `agentic/` 二级领域也需要采用编号三级目录。

```text
06-frameworks-and-tools/
├── 01-frameworks/
├── 02-coding-agents-and-tools/
├── 03-project-studies/
├── 04-skill-and-tool-systems/
└── 05-comparisons/
```

## 子目录说明

| 目录 | 定位 | 示例 |
|------|------|------|
| `01-frameworks/` | 可复用的通用 Agent 框架、SDK、编排框架 | AutoGen, CrewAI, LangGraph |
| `02-coding-agents-and-tools/` | 面向软件工程场景的 Agent 工具、产品或平台 | Claude Code, Codex, OpenHands |
| `03-project-studies/` | 完整 Agent 系统或开源项目的系统化拆解 | Hermes Agent, OpenClaw |
| `04-skill-and-tool-systems/` | Skill、Tool、插件、能力注册等工程实现对象 | Skill-based agents |
| `05-comparisons/` | 框架选型、产品对比、生态地图 | framework selection, coding tools comparison |

## 开源项目研究组织原则

系统性开源项目研究优先集中放在 `03-project-studies/<project>/`，提供项目级的完整阅读体验；其他专题目录只在需要支撑某个机制、证据或对比时克制回填，不复制完整项目分析。

例如 OpenHands 的整体研究应集中到 `03-project-studies/openhands/`；`05-environments/` 只回填 workspace / runtime / sandbox 相关证据，`02-single-agent/` 或 `03-multi-agent/` 只在确有专题需要时引用对应机制。

### `02-coding-agents-and-tools/` 与 `03-project-studies/` 的归属规则

- `02-coding-agents-and-tools/` 放**面向用户的编码 Agent / coding tool / 产品对象视角**：如何使用、如何选型、产品定位和工作流整合。
- `03-project-studies/` 放**系统性开源项目研究、架构拆解和实现机制视角**：如何拆解、如何理解内部机制。
- 当同一对象同时具备两种视角时（例如 OpenHands 既是编码 Agent 产品又是可系统拆解的开源项目），**系统性研究主归属放 `03-project-studies/<project>/`**，`02-coding-agents-and-tools/` 中只保留简要入口或交叉引用。
- `02-coding-agents-and-tools/` 下不复制 `03-project-studies/` 中已有的架构分析内容。如果需要跨类对象比较，应进入 `05-comparisons/`。

---

*最后更新: 2026-05-31*
