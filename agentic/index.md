# Agentic AI Index

> 如果你是第一次进入本目录，先读 [`README.md`](./README.md) 建立方向感；如果你已经知道想找什么，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `overview.md` 的整体理解主线。

## 目录结构

```text
agentic/
├── 01-foundations/                   # 基础概念
│   ├── definition-and-taxonomy/      # 定义与分类
│   ├── behavioral-paradigms/         # 行为范式连续谱
│   ├── cognitive-architectures/      # 认知架构总论
│   └── agent-system-modeling/        # Agent 系统建模
│
├── 02-single-agent/                  # 单智能体
│   ├── planning/                     # 规划
│   ├── memory/                       # 记忆
│   ├── tool-use/                     # 工具使用
│   ├── reasoning-and-acting-loop/    # 推理行动循环
│   ├── self-reflection/              # 自我反思
│   └── architectural-patterns/       # 架构模式
│
├── 03-multi-agent/                   # 多智能体
│   ├── collaboration/                # 协作模式
│   ├── competition-and-conflict/     # 竞争与冲突
│   ├── organizational/               # 组织架构
│   ├── shared-state-and-context/     # 共享状态与上下文
│   ├── coordination/                 # 协调机制
│   └── communication-protocols/      # 通信协议
│
├── 04-human-agent-interaction/       # 人机交互
│   ├── human-in-the-loop/            # 人在回路
│   ├── interaction-surfaces/         # 交互表面
│   ├── delegation-and-control/       # 委托与控制
│   └── trust-and-alignment/          # 信任与对齐
│
├── 05-environments/                  # 环境与仿真
│   ├── simulated-environments/       # 仿真环境
│   ├── browser-environments/         # 浏览器环境
│   ├── code-execution-environments/  # 代码执行环境
│   ├── sandboxing-and-safety/        # 沙箱与安全
│   └── evaluation-environments/      # 评测环境
│
├── 06-frameworks-and-tools/          # 框架与工具
│   ├── 01-frameworks/                # 通用框架与 SDK
│   ├── 02-coding-agents-and-tools/   # 编码 Agent 工具与产品
│   ├── 03-project-studies/           # 项目案例研究
│   ├── 04-skill-and-tool-systems/    # 技能与工具系统
│   └── 05-comparisons/               # 对比与选型
│
└── 07-evaluation/                    # 评估与可靠性
    ├── task-completion-metrics/      # 任务完成度
    ├── agent-benchmarks/             # 通用 Agent 基准
    ├── swe-benchmarks/               # 软件工程基准
    ├── safety-and-robustness/        # 安全与鲁棒性
    ├── human-evaluation/             # 人工评估
    └── observability-and-debugging/  # 可观测性与调试
```

## 按问题找入口

- 想看 Agent 的定义、分类、行为范式或认知架构：先看 `01-foundations/`
- 想看单智能体中的规划、记忆、工具使用、反思或 ReAct 类循环：先看 `02-single-agent/`
- 想看多智能体中的协作、协调、通信协议、共享状态或冲突：先看 `03-multi-agent/`
- 想看人机协作、委托控制、交互界面或信任对齐：先看 `04-human-agent-interaction/`
- 想看环境、沙箱、安全、浏览器环境、仿真环境或评测环境：先看 `05-environments/`
- 想看框架、SDK、编码 Agent、项目案例、skill / tool 系统或横向对比：先看 `06-frameworks-and-tools/`
- 想看 benchmark、评估指标、鲁棒性、人工评估、可观测性或调试：先看 `07-evaluation/`

## 按阅读目标找入口

- 如果你的目标是先建立整体理解，这个问题不由本页展开，转读 [`overview.md`](./overview.md)
- 如果你的目标是查看当前还有哪些内容缺口，转读 [`backlog.md`](./backlog.md)
- 如果你的目标是查看接下来值得研究哪些对象或材料，转读 [`candidates.md`](./candidates.md)
- 如果你的目标是查看当前已记录的冲突、口径分歧或边界不一致，转读 [`conflict.md`](./conflict.md)

## 开源仓库与工具存放指南

| 内容类型 | 放入目录 | 示例 |
|---------|---------|------|
| 通用 Agent 框架 / SDK / 编排框架 | `06-frameworks-and-tools/01-frameworks/` | LangGraph, AutoGen, CrewAI |
| 编码 Agent 工具 / 产品 | `06-frameworks-and-tools/02-coding-agents-and-tools/` | Claude Code, Codex, OpenHands |
| 完整 Agent 系统或开源项目案例 | `06-frameworks-and-tools/03-project-studies/` | Hermes Agent, OpenClaw |
| Skill / Tool / 插件生态实现 | `06-frameworks-and-tools/04-skill-and-tool-systems/` | Skill-based agents |
| 单智能体架构模式 | `02-single-agent/architectural-patterns/` | ReAct, RA-AID, AutoGPT |
| 多智能体协作系统与模式 | `03-multi-agent/` | MetaGPT, ChatDev |
| 评估基准与论文 | `07-evaluation/` | AgentBench, SWE-bench |
| 环境仿真平台 | `05-environments/` | WebArena, OSWorld |
