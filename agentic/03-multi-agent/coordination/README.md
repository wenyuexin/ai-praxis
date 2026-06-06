# Coordination

> 适用范围：多智能体协调、调度与依赖管理研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理多智能体系统中与协调相关的关键机制，包括任务分配、执行顺序、依赖控制、冲突消解与全局一致性维护。

## 快速入口

- 结合上层 `../README.md` 理解 coordination 与 collaboration、communication-protocols 的区别。
- 重点关注调度策略、依赖管理、角色切换、失败恢复与死锁避免。
- 当前专题：`topology.md` 用于比较 centralized / decentralized / hybrid / independent 等协调拓扑。
- 当前专题：`failure-modes.md` 用于整理多智能体协调中的典型失败模式与问题地图。
- 当前专题：`when-multi-agent-helps.md` 用于判断多智能体何时相对单智能体更可能带来净收益。
- 如出现机制边界或术语口径冲突，记录到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：调度、依赖管理、执行顺序、冲突消解、协调成本、一致性维护。
- 不放在这里：通信协议本身（放 `../communication-protocols/`）、协作分工模式（放 `../collaboration/`）、共享上下文与状态存储（放 `../shared-state-and-context/`）。

## 同目录导航

- 当前专题：`topology.md`，用于比较 centralized / decentralized / hybrid / independent 等协调拓扑。
- 当前专题：`failure-modes.md`，用于整理多智能体协调中的典型失败模式与问题地图。
- 当前专题：`when-multi-agent-helps.md`，用于判断多智能体何时相对单智能体更可能带来净收益。
