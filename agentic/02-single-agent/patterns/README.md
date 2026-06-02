# Patterns

> 适用范围：单智能体典型工作流模式与工程抽象研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理单智能体系统中反复出现的典型工作流模式、组合范式与工程抽象。

## 快速入口

- 结合上层 `../README.md` 理解 patterns 与 planning、tool-use、reasoning-and-acting 的联系。
- 重点关注 ReAct、AutoGPT 风格流程、RA.Aid 等可迁移模式。
- 如出现模式定义、机制边界或项目口径冲突，优先记录到 `../conflict.md`；仅临时判断再落到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：可复用工作流模式、典型执行闭环、跨机制组合范式、工程抽象模板。
- 不放在这里：单一理论方法本身（放 `../planning/`、`../reasoning-and-acting/`）、具体框架案例全集（放 `../../06-frameworks-and-tools/`）。

## 同目录导航

- 相关上位主轴：`../tool-centric-vs-monolithic.md`，用于理解不同 workflow pattern 背后到底是统一模型主导，还是工具链主导。
- 相关边界专题：`../agent-vs-tool-workflow-boundary.md`，用于区分 pattern、tool-use workflow 与 agent 执行闭环的边界。
- 如发现 patterns 相关结论冲突，优先记录到 `../conflict.md`。

## 目录结构

```text
patterns/
├── README.md
├── autogpt-pattern/
├── images/
├── ra-aid/
└── react/
```
