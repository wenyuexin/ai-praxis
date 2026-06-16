# AI 智能体 (Agentic AI)

LLM 驱动的自主智能系统：从基础概念、单智能体、多智能体，到人机交互、环境、框架工具与评估体系。

> **入口说明**：第一次进入时，先读本页的分类依据与边界说明建立整体方向；如需查找目录结构、子目录定位或学习路径，转向 [`index.md`](./index.md)。

## 分类依据

Agentic 目录按抽象层级组织：先建立 Agent 的基础概念与系统模型，再分别沉淀单智能体能力、多智能体关系、人机交互、环境约束、工程生态与评估方法。

## 边界说明

| 内容 | 归属 | 说明 |
|------|------|------|
| Agent 基础定义、分类边界、系统模型 | `01-foundations/` | 建立跨主题的概念底座，并提供可被下游目录引用的工作定义锚点 |
| 单智能体认知能力（planning/memory/tool-use/reflection） | `02-single-agent/` | 认知能力是架构组成部分，不放入框架工具层重复建设 |
| 多智能体特有问题（协作、协调、通信、共享状态） | `03-multi-agent/` | 关注多 Agent 关系、组织与交互机制 |
| 人与 Agent 的任务委托、控制与交互体验 | `04-human-agent-interaction/` | 关注人机协作边界与交互模式 |
| Agent 运行、交互和评测所依赖的环境 | `05-environments/` | 关注环境建模、沙箱、浏览器与代码执行上下文 |
| 具体框架、工具、产品、开源项目 | `06-frameworks-and-tools/` | 只承载工程生态对象与实现案例，不重建底层能力体系 |
| Agent 评估方法、benchmark、可观测性 | `07-evaluation/` | 关注评估、调试和可靠性判断 |
| Agent 驱动的 RAG | `../rag/03-advanced-patterns/agentic-rag/` | RAG 视角 |
| LLM 推理优化 | `../llm/04-serving/` | Agent 底层引擎 |

## 与其他目录的关系

- [LLM](../llm/) — Agent 的核心引擎
- [RAG](../rag/) — Agent 的知识检索能力
- [知识图谱](../knowledge-graph/) — Agent 的结构化知识
- [具身智能](../embodied-intelligence/) — Agent 的物理落地

---

*最后更新: 2026-05-31*
