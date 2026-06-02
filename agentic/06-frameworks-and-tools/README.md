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

```text
06-frameworks-and-tools/
├── 01-frameworks/                 # 通用 Agent 框架、SDK 与编排框架
│   ├── autogen/
│   ├── crewai/
│   └── langchain-agents/
├── 02-coding-tools/               # 面向软件工程任务的 Agent 工具与产品
│   ├── claude-code/
│   ├── claw-code/
│   └── codex/
├── 03-project-studies/            # 完整 Agent 系统或开源项目案例研究
│   └── hermes-agent/
├── 04-skill-and-tool-systems/     # Skill、Tool、插件与能力系统
│   └── skill-based-agents/
└── 05-comparisons/                # 横向对比、选型与生态地图
```

## 子目录说明

| 目录 | 定位 | 示例 |
|------|------|------|
| `01-frameworks/` | 可复用的通用 Agent 框架、SDK、编排框架 | AutoGen, CrewAI, LangGraph |
| `02-coding-tools/` | 面向软件工程场景的 Agent 工具、产品或平台 | Claude Code, Codex, OpenHands |
| `03-project-studies/` | 完整 Agent 系统或开源项目的系统化拆解 | Hermes Agent, OpenClaw |
| `04-skill-and-tool-systems/` | Skill、Tool、插件、能力注册等工程实现对象 | Skill-based agents |
| `05-comparisons/` | 框架选型、产品对比、生态地图 | framework selection, coding tools comparison |


---

*最后更新: 2026-05-31*
