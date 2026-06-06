# 03 Multi-Agent

> 适用范围：多智能体协作机制研究与知识组织
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于系统整理多智能体协作的核心机制、通信结构、协调成本与典型失效模式，而不是直接复刻某个编排框架。

## 快速入口

- 从 `collaboration/`、`coordination/`、`communication-protocols/` 进入核心协作问题。
- 从 `shared-state-and-context/` 关注共享状态与上下文污染问题。
- 如遇证据冲突或口径不一致，统一记录到 `../temp/conflict.md`。

## 分类依据

本目录按多智能体系统中的关键问题划分：协作、通信、协调、共享状态与上下文、组织结构与竞争关系。

## 边界说明

- 放在这里：多智能体任务拆解、角色分工、通信协议、共享状态、协调与冲突处理。
- 不放在这里：单智能体内部能力（放 `../02-single-agent/`）、执行环境与隔离基础设施（放 `../05-environments/`）、框架案例材料（放 `../06-frameworks-and-tools/`）。

## 同目录导航

- 核心主题入口：`collaboration/`、`coordination/`、`communication-protocols/`、`shared-state-and-context/`、`organizational/`、`competition-and-conflict/`。
- 建议优先从协作、协调与通信协议三条主线建立多智能体的基本结构。
## 目录结构

```text
03-multi-agent/
├── collaboration/
├── communication-protocols/
├── competition-and-conflict/
├── coordination/
├── organizational/
└── shared-state-and-context/
```

## 与其他目录的关系

- `../02-single-agent/` 提供多智能体协作所依赖的单体能力基础。
- `../05-environments/` 提供通信、隔离与执行边界的环境约束。
- `../06-frameworks-and-tools/` 中的项目案例可作为多智能体机制的证据载体。
- `../07-evaluation/` 提供评估多智能体质量、成本、时延与稳定性的方法。
