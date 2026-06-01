# Task Decomposition

> 适用范围：任务拆解策略与子问题组织研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理单智能体如何把复杂任务拆成更小的子问题、子步骤与执行单元。

## 快速入口

- 结合上层 `../README.md` 理解 task decomposition 与 plan-and-execute、tree-of-thoughts 的区别。
- 重点关注拆解粒度、依赖关系、递归分解、错误传播与组合成本。
- 如出现拆解定义、方法边界或论文结论冲突，记录到 `../../../temp/conflict.md`。

## 边界说明

- 放在这里：任务切片、子问题组织、分解层次、依赖关系、组合与回收。
- 不放在这里：完整计划执行模式（放 `../plan-and-execute/`）、搜索式推理树（放 `../tree-of-thoughts/`）、多智能体任务分工（放 `../../../03-multi-agent/collaboration/`）。

## 同目录导航

- 如后续需要整体综述，使用 `overview.md`。
- 如需要记录内容缺口，使用 `backlog.md`。
- 如需要学习或建设顺序，使用 `roadmap.md`。
- 如发现目录范围内冲突，使用 `conflict.md`。
