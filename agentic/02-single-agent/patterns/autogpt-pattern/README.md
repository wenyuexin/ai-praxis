# AutoGPT Pattern

> 适用范围：AutoGPT 风格循环式单智能体模式研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理 AutoGPT 风格的目标驱动、自循环、自调用式单智能体模式，以及相关收益与失效机制。

## 快速入口

- 结合上层 `../README.md` 理解 AutoGPT pattern 在 patterns 体系中的位置。
- 重点关注目标维持、自循环执行、工具调用扩展、上下文膨胀与失控风险。
- 如出现模式定义、历史版本差异或项目口径冲突，优先记录到 `../../conflict.md`；仅临时判断再落到 `../../../temp/conflict.md`。

## 边界说明

- 放在这里：AutoGPT 风格循环、目标驱动执行、自主扩展与失控风险。
- 不放在这里：具体 AutoGPT 项目案例全集（放 `../../../06-frameworks-and-tools/`）、一般规划方法（放 `../../planning/`）、环境隔离问题（放 `../../../05-environments/`）。

## 同目录导航

- 相关上位主轴：`../../agent-vs-tool-workflow-boundary.md`，用于判断 AutoGPT 风格循环何时只是工具工作流，何时已经形成 agent 闭环。
- 相关上位主轴：`../../tool-centric-vs-monolithic.md`，用于理解其工具扩展路径与统一模型路径的组织差异。
- 如发现 AutoGPT pattern 相关结论冲突，优先记录到 `../../conflict.md`。
