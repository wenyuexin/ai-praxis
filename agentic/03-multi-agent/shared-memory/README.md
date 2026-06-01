# Shared Memory

> 适用范围：多智能体共享状态与上下文管理研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理多智能体系统中的共享记忆、共享状态、上下文同步与污染控制问题。

## 快速入口

- 结合上层 `../README.md` 理解 shared memory 与 communication-protocols、coordination 的关系。
- 重点关注共享上下文的一致性、可见性、检索策略、写回冲突与污染风险。
- 如出现术语定义、状态边界或版本口径冲突，记录到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：共享状态模型、上下文同步、读写冲突、污染控制、可见性策略、记忆隔离。
- 不放在这里：单智能体记忆机制（放 `../../02-single-agent/memory/`）、通信消息协议（放 `../communication-protocols/`）、全局调度与依赖控制（放 `../coordination/`）。

## 同目录导航

- 如后续需要整体综述，使用 `overview.md`。
- 如需要记录内容缺口，使用 `backlog.md`。
- 如需要学习或建设顺序，使用 `roadmap.md`。
- 如发现目录范围内冲突，使用 `conflict.md`。
