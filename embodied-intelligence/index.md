# Embodied Intelligence Index

> 如果你是第一次进入本目录，先读 [`README.md`](./README.md) 建立方向感；如果你已经知道想找什么，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `overview.md` 的整体理解主线。

## 目录结构

```
embodied-intelligence/
├── 01-foundations/                    # 基础
│   ├── embodiment-hypothesis/         # 具身假设
│   ├── sensors-and-actuators/         # 传感器与执行器
│   └── simulation-environments/       # 仿真环境
│
├── 02-perception/                     # 感知闭环
│   ├── object-detection-and-tracking/ # 目标检测与跟踪
│   ├── 3d-scene-understanding/        # 3D场景理解
│   └── tactile-and-multimodal-sensing/ # 触觉与多模态感知
│
├── 03-motor-control-and-policies/     # 运动控制与策略
│   ├── imitation-learning/            # 模仿学习
│   ├── reinforcement-learning-for-robotics/ # 机器人强化学习
│   └── sim-to-real-transfer/          # Sim-to-Real迁移
│
├── 04-planning-and-navigation/        # 规划与导航
│   ├── path-planning/                 # 路径规划
│   ├── task-and-motion-planning/      # 任务与运动规划
│   └── llm-based-planning/            # LLM驱动规划
│
├── 05-manipulation/                   # 操作与交互
│   ├── grasping/                      # 抓取
│   ├── dexterous-manipulation/        # 灵巧操作
│   └── human-robot-interaction/       # 人机交互
│
├── 06-foundation-models/              # 大模型驱动具身
│   ├── vision-language-action-models/ # VLA模型
│   ├── foundation-models-for-robotics/ # 机器人基础模型
│   └── language-conditioned-policies/  # 语言条件策略
│
└── 07-evaluation-and-benchmarks/      # 评估与基准
    ├── standardized-tasks/            # 标准化任务
    └── robust-evaluation/             # 鲁棒评估
```

## 学习路径

**基础阶段**
- `01-foundations/` — 具身假设、传感器、仿真环境
- `02-perception/` — 感知闭环

**核心阶段**
- `03-motor-control-and-policies/` — 运动控制与策略
- `04-planning-and-navigation/` — 规划与导航
- `05-manipulation/` — 操作与交互

**大模型驱动**
- `06-foundation-models/` — VLA 模型、基础模型

**评估与实践**
- `07-evaluation-and-benchmarks/` — 标准化任务、鲁棒评估

## 按子目录定位

- `01-foundations/`：具身假设、传感器与执行器、仿真环境
- `02-perception/`：目标检测与跟踪、3D 场景理解、触觉与多模态感知
- `03-motor-control-and-policies/`：模仿学习、机器人 RL、Sim-to-Real 迁移
- `04-planning-and-navigation/`：路径规划、任务与运动规划、LLM 驱动规划
- `05-manipulation/`：抓取、灵巧操作、人机交互
- `06-foundation-models/`：VLA 模型、机器人基础模型、语言条件策略
- `07-evaluation-and-benchmarks/`：标准化任务、鲁棒评估

## 按阅读目标找入口

- 想建立整体理解框架：读 [`overview.md`](./overview.md)
