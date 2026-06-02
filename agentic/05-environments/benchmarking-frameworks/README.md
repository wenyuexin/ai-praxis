# Benchmarking Frameworks

> 适用范围：环境相关评测框架与任务运行基座研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理与 Agent 环境直接相关的评测框架、任务运行基座与可复现实验设置，关注环境如何塑造 benchmark 的可信度、可比较性与可复现性。

## 快速入口

- 结合上层 `../README.md` 理解 benchmarking-frameworks 在环境体系中的位置。
- 重点关注任务环境建模、状态/动作空间、运行约束、评测可复现性与环境依赖假设。
- 如出现 benchmark 口径、环境假设或评测边界冲突，优先记录到 `../conflict.md`；仅临时判断再落到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：环境相关 benchmark、任务运行基座、环境假设、复现约束与评测接口。
- 不放在这里：具体单智能体能力评测维度（放 `../07-evaluation/`）、浏览器/代码执行/仿真环境本体（放本层其他子目录）。

## 同目录导航

- 相关上位主题：`../overview.md`，用于回到环境层的三层框架并理解 benchmark 环境的落位。
- 相关邻近主题：`../simulated-environments/README.md`，用于查看受控任务世界与环境评测基座之间的关系。
- 如发现 benchmarking-frameworks 相关结论冲突，优先记录到 `../conflict.md`。
