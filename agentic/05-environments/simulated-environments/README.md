# Simulated Environments

> 适用范围：受控任务世界与仿真交互环境研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理 Agent 所运行的受控任务世界、仿真交互环境与可重复实验设置，关注环境如何为 agent 提供可控状态空间、反馈机制与实验边界。

## 快速入口

- 结合上层 `../README.md` 理解 simulated-environments 在环境体系中的位置。
- 重点关注 world state、observation/action interface、环境反馈、可重复性与实验控制。
- 如出现环境定义、任务口径或实验边界冲突，优先记录到 `../conflict.md`；仅临时判断再落到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：受控任务环境、世界状态设计、仿真反馈机制、实验控制与环境接口。
- 不放在这里：真实浏览器环境（放 `../browser-environments/`）、通用代码执行环境（放 `../code-execution-environments/`）、评测框架总论（放 `../benchmarking-frameworks/`）。

## 同目录导航

- 相关上位主题：`../overview.md`，用于回到环境层的三层框架并理解仿真环境的落位。
- 相关邻近主题：`../benchmarking-frameworks/README.md`，用于查看仿真环境与环境评测基座之间的关系。
- 如发现 simulated-environments 相关结论冲突，优先记录到 `../conflict.md`。
