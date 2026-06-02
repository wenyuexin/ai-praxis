# Agent System Models Research Queue

> 适用范围：`agent-system-models/` 方向的待研究对象清单
> 使用原则：本文件记录下一步值得研究的论文、文档与对象，不替代 `01-foundations/backlog.md` 的问题缺口职责；对象研究完成后，应转化为正式专题或论文笔记

## 一、当前主题

当前队列聚焦 `Agent Harness as System Model`，用于跟踪将 harness / runtime substrate / harness engineering 上升为 foundations 层系统模型的研究对象。

## 二、待研究对象

| 对象 | 类型 | 关联问题 | 为什么值得研究 | 当前状态 | 产出链接 |
|------|------|----------|----------------|----------|----------|
| **Meta-Harness: End-to-End Optimization of Model Harnesses** | paper | Agent Harness as System Model | 代表 harness 自动搜索与端到端优化方向，能补足 harness 从静态配置走向自演化系统的视角 | 待研究 | - |
| **Natural-Language Agent Harnesses** | paper | Agent Harness as System Model | 明确把 harness 提升为一等研究对象，并提出自然语言规范 + runtime 的双层结构 | 待研究 | - |
| **Agentic Harness Engineering: Observability-Driven Automatic Evolution of Coding-Agent Harnesses** | paper | Agent Harness as System Model | 将 observability 与 harness evolution 直接绑定，能连接 system model 与执行反馈闭环 | 待研究 | - |
| **The Last Harness You'll Ever Build** | paper | Agent Harness as System Model | 代表“通用 harness / 通用运行基座”设计哲学，适合观察其抽象边界是否稳定 | 待研究 | - |
| **AI Harness Engineering: A Runtime Substrate for Foundation-Model Software Agents** | paper | Agent Harness as System Model | 直接使用 `runtime substrate` 表述，是 foundations 层最关键的概念信号之一 | 待研究 | - |
| **Effective Harness Engineering for Algorithm Discovery with Coding Agents** | paper | Agent Harness as System Model | 展示 harness 在特定任务域中的工程化最佳实践，可观察通用模型与场景特化之间的关系 | 待研究 | - |
| **Externalization in LLM Agents: A Unified Review of Memory, Skills, Protocols and Harness Engineering** | paper | Agent Harness as System Model / Cognitive Architecture vs Orchestration Architecture | 已从认知架构视角给出 harness 的统一理论坐标，可作为阅读其他 harness 论文的上位框架 | 已完成 | `../cognitive-architectures/Externalization_2604.08224.md` |

## 三、使用说明

- 若对象仅提供线索、尚未完成系统阅读，保留在本队列中。
- 若对象已完成研究并形成稳定输出，应转化为对应子目录的正式专题或论文笔记，并在 `产出链接` 中回填。
- 若某个对象最终只支持局部判断、不再值得继续跟踪，可从本队列移除，而不是长期堆积。

## 四、与其他文件的关系

- `../../01-foundations/backlog.md` 记录 foundations 层的问题缺口；本文件只记录待研究对象。
- `../cognitive-architectures/Externalization_2604.08224.md` 提供 harness 进入外部化认知架构框架后的上位解释，可作为本队列对象的理论参照。
- 后续若 `agent-system-models/` 下形成正式 harness 总论，相关条目应继续下沉，不长期停留在队列里。
