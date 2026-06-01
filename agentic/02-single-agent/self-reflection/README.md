# Self Reflection

> 适用范围：单智能体自检、批评与迭代修正研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理单智能体如何评估自身输出、识别错误、执行批评并进行迭代修正。

## 快速入口

- 结合上层 `../README.md` 理解 self-reflection 与 planning、reasoning-and-acting 的关系。
- 重点关注 critique model、iterative refinement、自检触发条件与修正收益。
- 如出现反思定义、评价标准或论文结论冲突，记录到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：自检、批评、错误识别、修正循环、反思触发条件。
- 不放在这里：初始规划逻辑（放 `../planning/`）、工具调用协议（放 `../tool-use/`）、多智能体互评与协作反馈（放 `../../03-multi-agent/` 相关主题）。

## 同目录导航

- 如后续需要整体综述，使用 `overview.md`。
- 如需要记录内容缺口，使用 `backlog.md`。
- 如需要学习或建设顺序，使用 `roadmap.md`。
- 如发现目录范围内冲突，使用 `conflict.md`。

## 目录结构

```text
self-reflection/
├── README.md
├── critique-models/
└── iterative-refinement/
```
