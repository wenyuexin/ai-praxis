# 具身智能 (Embodied Intelligence)

具身智能：物理世界中的感知、控制、规划与交互，以及大模型驱动的机器人智能。

> **入口说明**：第一次进入时，先读本页的分类依据与边界说明建立整体方向；如需查找目录结构、子目录定位或学习路径，转向 [`index.md`](./index.md)。

## 分类依据

Embodied Intelligence 目录按"基础 → 感知 → 控制与策略 → 规划与导航 → 操作与交互 → 大模型驱动 → 评估"组织：

## 边界说明

| 内容 | 适合放 Embodied Intelligence | 不适合放 Embodied Intelligence |
|------|---------------------------|------------------------------|
| 具身假设、传感器、仿真环境 | 01-foundations | — |
| 机器人视觉感知（场景理解、目标跟踪） | 02-perception | 通用视觉方法放 `cv/` |
| 模仿学习、Sim-to-Real | 03-motor-control-and-policies | 模仿学习的通用 RL 理论放 `reinforce-learning/` |
| 路径规划、任务规划 | 04-planning-and-navigation | LLM 规划（CoT、ToT）放 `agentic/02-single-agent/planning/` |
| 人机交互（物理） | 05-manipulation/human-robot-interaction | 数字人机交互放 `agentic/04-human-agent-interaction/` |
| VLA 模型、机器人基础模型 | 06-foundation-models | 通用多模态 LLM 放 `llm/07-multimodal/` |
| 具身世界模型 | 06-foundation-models | 世界模型原理放 `world-models/` |

## 与其他目录的关系

| 本目录内容 | 关联目录 | 说明 |
|-----------|---------|------|
| 机器人视觉感知 | [../cv/](../cv/) | CV 提供通用视觉方法，本目录聚焦机器人场景 |
| 机器人 RL | [../reinforce-learning/](../reinforce-learning/) | RL 提供算法原理，本目录聚焦机器人控制应用 |
| LLM 驱动规划 | [../agentic/02-single-agent/planning/](../agentic/02-single-agent/planning/) | Agent 规划是数字层面，具身规划需要物理约束 |
| 物理人机交互 | [../agentic/04-human-agent-interaction/](../agentic/04-human-agent-interaction/) | 数字人机交互在 agentic，物理人机交互在本目录 |
| VLA/机器人基础模型 | [../llm/07-multimodal/](../llm/07-multimodal/) | 通用多模态 LLM 在 llm，具身专用模型在本目录 |
| 具身世界模型 | [../world-models/04-large-scale-world-models/embodied-world-models/](../world-models/04-large-scale-world-models/embodied-world-models/) | 世界模型原理在 world-models |
| 具身假设与哲学 | [../interdisciplinarity/03-philosophy-of-mind/](../interdisciplinarity/03-philosophy-of-mind/) | 具身认知的哲学基础 |

---

*最后更新: 2026-05-11*
