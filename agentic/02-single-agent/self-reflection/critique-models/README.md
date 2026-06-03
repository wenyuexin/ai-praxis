# Critique Models

> 适用范围：单智能体中的自批评、独立 critic 与评审角色设计研究
> 阶段状态：占位入口，后续补充主干内容

## 一句话定位

本目录用于整理 single-agent 中用于识别问题、提出批评与辅助修正的 critique 机制与 critic 角色设计。

## 快速入口

- 结合上层 `../README.md` 理解 critique models 在 self-reflection 体系中的位置。
- 如需先理解 reflection 的收益边界，优先查看 `../reflection-cost-tradeoff.md`。
- 如需先理解触发与停止控制，优先查看 `../reflection-trigger-design.md`。

## 边界说明

- 放在这里：self-critique、separate critic、评审角色分工、批评输出结构。
- 不放在这里：整体反思收益评估（放 `../reflection-cost-tradeoff.md`）、触发策略（放 `../reflection-trigger-design.md`）、初始规划逻辑（放 `../../planning/`）。
