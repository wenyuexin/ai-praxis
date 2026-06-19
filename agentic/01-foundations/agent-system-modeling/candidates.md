# Agent System Modeling Candidates

> 适用范围：`agent-system-modeling/` 方向的候选研究对象清单
> 使用原则：本文件记录下一步值得研究的论文、文档与候选对象，不替代 `01-foundations/backlog.md` 的问题缺口职责；对象研究完成后，应转化为正式专题或论文笔记

## 一、当前主题

当前队列聚焦 `Agent Harness as System Model`，用于跟踪将 harness / runtime substrate / harness engineering 上升为 foundations 层系统建模的候选研究对象。

## 二、待研究对象

| 对象 | 类型 | 关联问题 | 为什么值得研究 | 当前状态 | 产出链接 |
|------|------|----------|----------------|----------|----------|
| **Externalization in LLM Agents: A Unified Review of Memory, Skills, Protocols and Harness Engineering** | paper | Agent Harness as System Model / Cognitive Architecture vs Orchestration Architecture | 已从认知架构视角给出 harness 的统一理论坐标，可作为阅读其他 harness 论文的上位框架；下一步需要基于论文原文补齐 harness 作为 unification layer 的精读证据 | 已完成；优先复核 | `../cognitive-architectures/Externalization_2604.08224.md` |
| **AI Harness Engineering: A Runtime Substrate for Foundation-Model Software Agents** | paper | Agent Harness as System Model | 直接使用 `runtime substrate` 表述，是 foundations 层最关键的概念信号之一；当前已形成论文笔记，后续主要用于继续复核哪些判断可以稳定上升为 harness 作为 system modeling 对象的 foundations 层表述 | 已形成笔记；优先复核 | `2605.13357_AI_Harness_Engineering.md` |
| **LLM Harness / Harness Engineering Survey（ETCLOVG 分类）** | survey / project | Agent Harness as System Model | 当前已形成首轮笔记，可作为 harness 生态分类与开放问题的对照材料；后续主要继续复核第 2–5 个开放问题，以及它与其他 harness 笔记之间的收敛边界 | 已形成笔记；继续复核 | `ETCLOVG_Harness_Survey.md` |
| **Natural-Language Agent Harnesses** | paper | Agent Harness as System Model | 明确把 harness 提升为一等研究对象，并提出自然语言规范 + runtime 的双层结构；适合作为可编程性 / 可移植性方向的补充材料 | 次级精读 | - |
| **Agentic Harness Engineering: Observability-Driven Automatic Evolution of Coding-Agent Harnesses** | paper | Agent Harness as System Model | 将 observability 与 harness evolution 直接绑定，能连接 system modeling 与执行反馈闭环；适合用于核验 observability 是 harness 内在维度还是演化驱动 | 次级精读 | - |
| **Meta-Harness: End-to-End Optimization of Model Harnesses** | paper | Agent Harness as System Model | 代表 harness 自动搜索与端到端优化方向，能补足 harness 从静态配置走向自演化系统的视角 | 后续参考 | - |
| **The Last Harness You'll Ever Build** | paper | Agent Harness as System Model | 代表“通用 harness / 通用运行基座”设计哲学，适合观察其抽象边界是否稳定 | 后续参考 | - |
| **Effective Harness Engineering for Algorithm Discovery with Coding Agents** | paper | Agent Harness as System Model | 展示 harness 在特定任务域中的工程化最佳实践，可观察通用模型与场景特化之间的关系 | 后续参考 | - |
| **Affordance Agent Harness** | paper | Agent Harness as System Model | 外部调研称其使用 closed-loop runtime / verification-gated skill orchestration 视角；更偏特定任务域，适合作为后期实证对照 | 待核验 | - |

## 三、使用说明

- 若对象仅提供线索、尚未完成系统阅读，保留在本队列中。
- 若对象已完成研究并形成稳定输出，应转化为对应子目录的正式专题或论文笔记，并在 `产出链接` 中回填。
- 若某个对象最终只支持局部判断、不再值得继续跟踪，可从本队列移除，而不是长期堆积。

## 四、当前阅读顺序

- **第一优先级**：继续复核 `Externalization in LLM Agents` 与 `AI Harness Engineering`，先把 harness 作为 unification layer / runtime substrate 的最小概念轮廓压实，再判断哪些结论可以上升为更稳定的 foundations 层表述。
- **第二优先级**：继续复核 `ETCLOVG_Harness_Survey.md` 中第 2–5 个开放问题的原文展开，同时再选择 `Natural-Language Agent Harnesses` 或 `Agentic Harness Engineering` 补充可编程性、可观测性与演化视角。
- **暂缓平均精读**：`Meta-Harness`、`The Last Harness You'll Ever Build`、`Effective Harness Engineering`、`Affordance Agent Harness` 更适合作为后续实证或特定任务域对照。

## 五、与其他文件的关系

- `../../01-foundations/backlog.md` 记录 foundations 层的问题缺口；本文件只记录候选研究对象与阅读优先级。
- `../cognitive-architectures/Externalization_2604.08224.md` 提供 harness 进入外部化认知架构框架后的上位解释，可作为本队列对象的理论参照。
- 一轮外部调研输入仅作为候选对象排序与问题识别的上游材料；其中趋势判断和强结论仍需回到原论文、官方文档或对象材料精读后，才能回流为正文定论。
- 后续若 `agent-system-modeling/` 下形成正式 harness 总论，相关条目应继续下沉，不长期停留在队列里。
