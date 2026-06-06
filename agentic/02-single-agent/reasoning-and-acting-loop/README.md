# Reasoning and Acting Loop

> 适用范围：单智能体中的推理—行动—观察循环研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理单智能体在任务执行中如何进行推理、采取行动、接收观察并更新后续决策的核心机制。

## 快速入口

- 结合上层 `../README.md` 理解本主题与 planning、tool-use、self-reflection 的关系。
- 重点关注推理—行动—观察循环如何承接规划、触发工具调用并吸收反馈。
- 如出现术语、机制或版本口径冲突，优先记录到 `../conflict.md`；仅临时判断再落到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：ReAct 及其变体、推理-行动循环、观察反馈、执行轨迹组织。
- 不放在这里：纯规划方法（放 `../planning/`）、工具协议本身（放 `../tool-use/`）、反思与自我批评（放 `../self-reflection/`）。

## 同目录导航

- 相关边界专题：`../agent-vs-tool-workflow-boundary.md`，用于理解 reasoning-action-observation loop 为什么是 agent 边界的关键组成。
- 相关上位主轴：`../planning-vs-execution.md`，用于理解局部推理循环与显式规划之间的耦合关系。
- 相关模式入口：`../architectural-patterns/react/README.md`，用于查看该执行循环在 `ReAct` 模式中的典型展开。
- 如发现 reasoning-and-acting 相关结论冲突，优先记录到 `../conflict.md`。
