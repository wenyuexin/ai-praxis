# Planning

> 适用范围：单智能体规划与任务分解研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理单智能体如何生成计划、拆解任务、调整步骤并在执行过程中维持目标导向。

## 快速入口

- 结合上层 `../README.md` 理解 planning 与 reasoning-and-acting、self-reflection 的关系。
- 重点关注计划生成、计划修正、任务分解、多步执行与失败后重规划。
- 如出现规划术语、方法边界或论文结论冲突，优先记录到 `../conflict.md`；仅临时判断再落到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：任务分解、计划生成、计划修订、多步目标组织、重规划机制。
- 不放在这里：推理—行动循环本身（放 `../reasoning-and-acting/`）、工具调用协议（放 `../tool-use/`）、多智能体协同调度（放 `../../03-multi-agent/coordination/`）。

## 同目录导航

- 相关边界专题：`explicit-planning-necessity.md`，用于区分 planning 能力本身与显式 planner 是否为必要结构。
- 相关主干专题：`bilevel-planning-tool-navigation.md`，用于展开复杂工具生态下高层规划与低层工具导航的分层设计。
- 如发现规划相关结论冲突，优先记录到 `../conflict.md`。

## 目录结构

```text
planning/
├── README.md
├── bilevel-planning-tool-navigation.md
├── explicit-planning-necessity.md
├── plan-and-execute/
├── task-decomposition/
└── tree-of-thoughts/
```
