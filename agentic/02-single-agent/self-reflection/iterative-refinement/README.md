# Iterative Refinement

> 适用范围：单智能体中的多轮修正、渐进改进与反思后迭代研究
> 阶段状态：占位入口，后续补充主干内容

## 一句话定位

本目录用于整理 single-agent 如何基于批评或反馈，对中间产物执行多轮修正与渐进改进。

## 快速入口

- 结合上层 `../README.md` 理解 iterative refinement 在 self-reflection 体系中的位置。
- 如需先理解何时值得进入多轮修正，优先查看 `../reflection-cost-tradeoff.md`。
- 如需先理解进入与退出修正循环的条件，优先查看 `../reflection-trigger-design.md`。

## 边界说明

- 放在这里：多轮 revise、渐进优化、反思后修补、局部改进循环。
- 不放在这里：触发与 stopping policy（放 `../reflection-trigger-design.md`）、critic 角色设计（放 `../critique-models/`）、更高层重规划（放 `../../planning/`）。

## 目录结构

```text
iterative-refinement/
└── README.md
```
