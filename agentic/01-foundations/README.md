# 01 Foundations

> 适用范围：Agentic AI 的基础概念、分类边界与系统模型研究
> 阶段状态：概念收敛中，调研优先

## 一句话定位

本目录用于沉淀 Agentic AI 的概念底座，回答“什么算 agent、核心概念由哪些维度组成、认知架构与执行编排如何区分、哪些系统模型正在形成稳定解释框架”等上位问题。

## 快速入口

- 从 `definition-and-taxonomy/` 进入基础定义、范式分类与术语边界问题。
- 从 `cognitive-architectures/` 进入认知负担如何被 memory、skills、protocols 等基础设施外部化的问题。
- 从 `agent-system-modeling/` 进入 harness、runtime substrate、控制面与系统分层等上位建模问题。
- 从 `behavioral-paradigms/` 进入反应式、审慎式与混合连续谱问题。
- 如遇概念口径冲突、术语漂移或结论力度不稳，优先回写对应专题材料，不要直接把临时判断写成主线定论。

## 分类依据

本目录按 foundations 层最核心的四类问题划分：

- 定义与分类：关注 workflow、agentic workflow、agent、多智能体等概念边界
- 认知架构：关注模型如何与外部设施分担认知负担
- 系统模型：关注 harness、runtime substrate、控制面、可观测性与治理等上位抽象
- 反应式 / 审慎式连续谱：关注执行风格、规划强度与 autonomy 结构之间的关系

这些维度共同回答的不是“某个具体框架怎么实现”，而是“Agentic AI 的基础概念空间如何组织”。

## 边界说明

- 放在这里：基础定义、分类依据、概念边界、认知架构、系统模型、执行风格连续谱。
- 不放在这里：单智能体能力机制（放 `../02-single-agent/`）、多智能体协调与协作（放 `../03-multi-agent/`）、执行环境与沙箱问题（放 `../05-environments/`）、框架或项目案例本身（放 `../06-frameworks-and-tools/`）。

## 同目录导航

- 核心主题入口：`definition-and-taxonomy/`、`cognitive-architectures/`、`agent-system-modeling/`、`behavioral-paradigms/`。
- 当前研究重点：workflow / agent 边界、harness 作为系统模型、认知架构与执行编排的区分。

## 目录结构

```text
01-foundations/
├── agent-system-modeling/
├── cognitive-architectures/
├── definition-and-taxonomy/
└── behavioral-paradigms/
```

## 与其他目录的关系

- `../02-single-agent/` 在本目录提供的概念底座之上讨论 planning、tool use、memory、reflection 等单体能力。
- `../03-multi-agent/` 在本目录定义的 agent 边界之上讨论 coordination、collaboration 与 communication。
- `../05-environments/` 依赖本目录澄清 environment、runtime、execution container 等上位区分。
- `../06-frameworks-and-tools/` 提供这些概念在具体框架、工具与项目案例中的工程实现。
- `../07-evaluation/` 依赖本目录中的工作定义判断“什么算被评估的 agent 系统”。
