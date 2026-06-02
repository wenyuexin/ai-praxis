# Plan and Execute

> 适用范围：规划—执行分离式单智能体模式研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理先生成计划、再按步骤执行的单智能体模式，以及相关的修正、重规划与执行反馈机制。

## 快速入口

- 结合上层 `../README.md` 理解 plan-and-execute 在 planning 体系中的位置。
- 重点关注规划与执行分离、计划粒度、执行反馈、失败后重规划。
- 如出现术语定义、模式边界或论文结论冲突，记录到 `../../../temp/conflict.md`。

## 边界说明

- 放在这里：规划—执行分离、计划落实、执行反馈驱动修正、计划重建。
- 不放在这里：一般任务拆解方法（放 `../task-decomposition/`）、搜索式思维扩展（放 `../tree-of-thoughts/`）、推理—行动循环（放 `../../reasoning-and-acting/`）。

## 同目录导航

- 相关边界专题：`../../explicit-planning-necessity.md`，用于说明 plan-and-execute 代表显式 planning 路径，但不是唯一标准答案。
- 建议结合 `../task-decomposition/README.md` 查看任务拆解如何进入完整规划流程。
- 建议结合 `../tree-of-thoughts/README.md` 对比线性计划执行与搜索式规划扩展。