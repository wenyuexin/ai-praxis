# Memory

> 适用范围：单智能体记忆机制与状态外化研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理单智能体在任务执行中如何表示、存储、检索、更新与利用记忆。

## 快速入口

- 结合上层 `../README.md` 理解 memory 与 planning、tool-use、self-reflection 的关系。
- 重点关注短期记忆、长期记忆、检索方法、写回策略与记忆污染问题。
- 如出现记忆定义、机制边界或版本口径冲突，优先记录到 `../conflict.md`；仅临时判断再落到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：短期/长期记忆、记忆检索、写回、压缩、记忆污染与过时问题。
- 不放在这里：多智能体共享状态（放 `../../03-multi-agent/shared-state-and-context/`）、协议化外部资源（放 `../tool-use/` 相关主题）、执行环境状态管理（放 `../../05-environments/`）。

## 同目录导航

- 相关主干专题：`memory-taxonomy-conflicts.md`，用于展开 memory 分类体系的冲突、重叠与统一理解框架。
- 相关主轴专题：`long-context-vs-external-memory.md`，用于比较长上下文窗口与独立 memory system 的替代边界与协同关系。
- 如发现记忆相关结论冲突，优先记录到 `../conflict.md`。

## 目录结构

```text
memory/
├── images/
├── long-term-memory/
├── retrieval-methods/
└── short-term-memory/
```
