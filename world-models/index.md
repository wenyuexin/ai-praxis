# World Models Index

> 如果你是第一次进入本目录，先读 [`README.md`](./README.md) 建立方向感；如果你已经知道想找什么，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `overview.md` 的整体理解主线。

## 目录结构

```
world-models/
├── 01-foundations/                   # 基础
│   ├── definition-and-motivation/    # 定义与动机
│   ├── predictive-models-in-rl/      # RL中的预测模型
│   └── learning-latent-dynamics/     # 潜动态学习
│
├── 02-methods-and-architectures/     # 方法与架构
│   ├── dreamer-family/               # Dreamer系列
│   │   ├── dreamerv1-v2/             # DreamerV1/V2
│   │   └── dreamerv3/                # DreamerV3
│   ├── autoregressive-world-models/  # 自回归世界模型
│   ├── diffusion-world-models/       # 扩散世界模型
│   └── jepa-and-energy-based-models/ # JEPA与能量模型
│
├── 03-reasoning-models/              # 推理用世界模型
│   ├── model-based-planning/         # 基于模型的规划
│   └── causal-world-models/          # 因果世界模型
│
├── 04-large-scale-world-models/      # 大规模世界模型
│   ├── video-prediction-models/      # 视频预测模型
│   ├── sora-and-video-generation/    # Sora与视频生成
│   └── embodied-world-models/        # 具身世界模型
│
├── 05-evaluation/                    # 评估
│   ├── prediction-metrics/           # 预测指标
│   └── downstream-task-transfer/     # 下游任务迁移
│
└── 06-applications/                  # 应用
    ├── robotics-simulation/          # 机器人仿真
    ├── autonomous-driving/           # 自动驾驶
    └── game-play-and-agi-exploration/ # 游戏与AGI探索
```

## 学习路径

**基础阶段**
- `01-foundations/` — 理解世界模型的基本概念
- `02-methods-and-architectures/` — 掌握不同架构的特点和适用场景

**算法阶段**
- `03-reasoning-models/` — 基于模型的规划与因果推理

**应用阶段**
- `04-large-scale-world-models/` — 视频生成、大规模预训练
- `05-evaluation/` — 评估方法
- `06-applications/` — 各领域的具体应用

## 按子目录定位

- `01-foundations/`：世界模型的定义与动机、RL 中的预测模型、潜动态学习
- `02-methods-and-architectures/`：Dreamer 系列、自回归/扩散世界模型、JEPA 与能量模型
- `03-reasoning-models/`：基于模型的规划、因果世界模型
- `04-large-scale-world-models/`：视频预测、Sora 与视频生成、具身世界模型
- `05-evaluation/`：预测指标、下游任务迁移
- `06-applications/`：机器人仿真、自动驾驶、游戏与 AGI 探索
