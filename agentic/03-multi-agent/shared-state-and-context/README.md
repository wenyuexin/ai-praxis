# Shared Memory

> 适用范围：多智能体共享状态与上下文管理研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理多智能体系统中的共享记忆、共享状态、上下文同步与污染控制问题。

## 快速入口

- 结合上层 `../README.md` 理解 shared memory 与 communication-protocols、coordination 的关系。
- 重点关注共享上下文的一致性、可见性、检索策略、写回冲突与污染风险。
- 如出现术语定义、状态边界或版本口径冲突，优先记录到上层 `../conflict.md`；若只是临时判断，可先做临时冲突整理，但不要长期替代稳定冲突入口。

## 边界说明

- 放在这里：共享状态模型、上下文同步、读写冲突、污染控制、可见性策略、记忆隔离。
- 不放在这里：单智能体记忆机制（放 `../../02-single-agent/memory/`）、通信消息协议（放 `../communication-protocols/`）、全局调度与依赖控制（放 `../coordination/`）。

## 同目录导航

- 建议结合 `../communication-protocols/README.md` 区分共享状态与消息协议的职责边界。
- 建议结合 `../../02-single-agent/memory/README.md` 对比单智能体记忆与多智能体共享记忆的差异。