# 训练数据与综述资源（当前目录待重组）

本目录整理具身AI/VLA模型训练相关的数据集资源。

> 当前目录路径仍名为 `07-evaluation-and-benchmarks/`，但现有主体内容是训练数据与综述论文集，不应被视为已经形成的评估与基准知识目录。后续需要在确认材料去向后，再决定拆分、迁移或重命名。

## 数据金字塔

根据NVIDIA提出的VLA训练数据金字塔，数据分为以下几类：

### 1. 互联网图文数据

用于预训练VLM部分，赋予模型语义理解能力。

| 数据集 | 规模 | 用途 | 链接 |
|--------|------|------|------|
| COCO | 330k图片 | 目标检测、图像描述 | [cocodataset.org](https://cocodataset.org) |
| VQAv2 | 265k图片 | 视觉问答 | [visualqa.org](https://visualqa.org) |
| WebLI | 10B图文对 | 多模态预训练 | Google |
| LAION-400M | 400M图文对 | 图文检索、预训练 | [laion.ai](https://laion.ai) |

### 2. 视频数据

学习操作技能和物理常识。

| 数据集 | 规模 | 特点 | 链接 |
|--------|------|------|------|
| Ego4D | 3670小时 | 第一人称视角，74个地点 | [ego4d-data.org](https://ego4d-data.org) |
| Ego-Exo-4D | 1286小时 | 多视角同步捕捉 | [ego-exo4d-data.org](https://ego-exo4d-data.org) |
| EPIC-KITCHENS | 100小时 | 厨房环境动作识别 | [epic-kitchens.github.io](https://epic-kitchens.github.io) |
| Something-Something V2 | 220k视频 | 174种基本动作 | [developer.qualcomm.com](https://developer.qualcomm.com) |
| HowTo100M | 136M视频片段 | 教学视频 | [www.rocq.inria.fr](https://www.rocq.inria.fr) |

### 3. 仿真数据

低成本大规模生成训练数据。

| 数据集/环境 | 仿真引擎 | 任务数 | 链接 |
|-------------|----------|--------|------|
| RLBench | CoppeliaSim | 100种 | [github.com/stepjam/RLBench](https://github.com/stepjam/RLBench) |
| CALVIN | PyBullet | 长序列任务 | [github.com/mees/calvin](https://github.com/mees/calvin) |
| LIBERO | MuJoCo | 130种 | [github.com/Lifelong-Robot-Learning/LIBERO](https://github.com/Lifelong-Robot-Learning/LIBERO) |
| Meta-World | MuJoCo | 50种 | [github.com/rlworkgroup/metaworld](https://github.com/rlworkgroup/metaworld) |
| RoboCasa | - | 120种场景 | [robocasa.ai](https://robocasa.ai) |

### 4. 真实机器人数据

核心训练数据，包含完整动作信息。

| 数据集 | 规模 | 机器人类型 | 链接 |
|--------|------|------------|------|
| OXE (Open X-Embodiment) | 1M+轨迹 | 22种机器人 | [robotics-transformer-x.github.io](https://robotics-transformer-x.github.io) |
| DROID | 76k轨迹 | 多种 | [droid-dataset.github.io](https://droid-dataset.github.io) |
| BridgeData V2 | 60k轨迹 | WidowX | [rail-berkeley.github.io](https://rail-berkeley.github.io) |
| RT-1 | 130k视频片段 | Google Robot | Google |
| AgiBot World | 1M轨迹 | AgiBot | [agibot-world.com](https://agibot-world.com) |
| RDT-1B | 1M+轨迹 | 多种形态 | [github.com/thu-ml/rdt](https://github.com/thu-ml/rdt) |

## 数据处理方法

### 视频数据利用

- **SWIM**: 第一人称视频预训练 + 机器人数据微调
- **MimicPlay**: 视频学"做什么" + 机器人数据学"怎么做"
- **EgoMimic**: 人类数据与机器人数据联合训练
- **HRP**: 从人类交互提取可操作性先验

### 数据增强

- 域随机化（Domain Randomization）
- 合成数据生成（世界模型）
- 跨域迁移学习

## 数据挑战

1. **数据稀缺**: 机器人操作数据采集成本高
2. **长尾分布**: 罕见场景数据不足
3. **异构数据**: 不同机器人动作空间不统一
4. **Sim-to-Real Gap**: 仿真与真实环境差异
5. **多模态缺失**: 缺乏触觉、力觉等信息

## 参考资料

- 论文: `../papers/VLA_Models_Survey_2508.15201.md` 第四节
- 论文: `../papers/LM_Empowered_Embodied_AI_2508.10399.md` 第五节
