# 世界模型 (World Models)

环境的内部模拟器：使智能系统能够预测未来状态、理解因果关系，并在不依赖真实交互的情况下做出决策。

> **入口说明**：第一次进入时，先读本页的分类依据与边界说明建立整体方向；如需查找目录结构、子目录定位或学习路径，转向 [`index.md`](./index.md)。

## 分类依据

World Models 目录按"基础 → 方法架构 → 推理 → 大规模模型 → 评估 → 应用"组织。

## 边界说明

| 内容 | 适合放 World Models | 不适合放 World Models |
|------|-------------------|----------------------|
| 潜动态模型、Dreamer 架构 | 02-methods-and-architectures | Dreamer 的 RL 训练细节放 `reinforce-learning/04-model-based-rl/` |
| 视频预测与生成（Sora 等） | 04-large-scale-world-models | 视频理解（动作识别等）放 `cv/04-video-and-3d-vision/` |
| 基于模型的规划 | 03-reasoning-models | LLM 推理/CoT 放 `llm/` |
| 因果世界模型 | 03-reasoning-models | 通用因果推理放 `interdisciplinarity/` |
| 机器人仿真 | 06-applications/robotics-simulation | 机器人感知与控制放 `embodied-intelligence/` |
| 图像生成（SD、DALL-E） | — | 放 `cv/05-generative-and-multimodal/` |
| 扩散模型原理 | — | 放 `deep-learning/03-architectures/generative-models/` |

## 与其他目录的关系

| 本目录内容 | 关联目录 | 说明 |
|-----------|---------|------|
| Dreamer/MBRL 架构 | [../reinforce-learning/04-model-based-rl/](../reinforce-learning/04-model-based-rl/) | RL 侧的训练与应用细节 |
| 视频预测与生成 | [../cv/04-video-and-3d-vision/](../cv/04-video-and-3d-vision/) | CV 侧重视频理解，WM 侧重视频预测/生成 |
| 扩散模型原理 | [../deep-learning/03-architectures/generative-models/](../deep-learning/03-architectures/generative-models/) | 扩散模型的通用架构原理 |
| 具身世界模型 | [../embodied-intelligence/06-foundation-models/](../embodied-intelligence/06-foundation-models/) | 具身智能中的世界模型应用 |
| 因果推理 | [../interdisciplinarity/](../interdisciplinarity/) | 因果推断的跨学科视角 |

---

*最后更新: 2026-05-11*
