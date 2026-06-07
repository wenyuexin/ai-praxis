# 07 Evaluation

> 适用范围：Agent 系统的评测、调试、可观测性与可靠性验证研究
> 阶段状态：目录骨架已建立，后续持续补充主干内容

## 一句话定位

本目录用于沉淀如何评价 Agent 是否有效、可靠、可解释、可复现，以及不同 benchmark、观测手段和人工评估方式分别回答什么问题。

## 快速入口

- 从 `agent-benchmarks/`、`swe-benchmarks/`、`observability-and-debugging/` 进入当前已有的主要子主题。
- 若关注环境如何影响评测可信度，可联动阅读 `../05-environments/`。
- 若关注框架或工具在真实任务中的效果，可再回看 `../06-frameworks-and-tools/` 的对象案例。

## 分类依据

本目录按 Agent 评估中的核心问题划分：

- `task-completion-metrics/`：任务完成度、成功定义与结果质量指标
- `agent-benchmarks/`：通用 Agent 基准、长程任务与多工具任务评测
- `swe-benchmarks/`：面向软件工程任务的专项基准
- `safety-and-robustness/`：安全性、稳定性、鲁棒性与失败边界
- `human-evaluation/`：人工评估、主观反馈与协作体验验证
- `observability-and-debugging/`：轨迹、日志、回放、归因与调试方法

## 边界说明

- 放在这里：benchmark、指标设计、人工评估、可观测性、调试、失败分析与可靠性验证。
- `papers/` 与 `images/` 只作为材料辅助目录存在，不承载独立主题分类。
- 不放在这里：Agent 内部能力机制（放 `../02-single-agent/`）、多智能体组织机制（放 `../03-multi-agent/`）、运行环境本体（放 `../05-environments/`）、具体产品案例（放 `../06-frameworks-and-tools/`）。

## 同目录导航

- 核心主题入口：`task-completion-metrics/`、`agent-benchmarks/`、`swe-benchmarks/`、`safety-and-robustness/`、`human-evaluation/`、`observability-and-debugging/`。
- 当前较适合优先补强的方向包括：任务成功标准、评测环境约束、轨迹归因与人工评估方法。

## 目录结构

```text
07-evaluation/
├── agent-benchmarks/
├── human-evaluation/
├── observability-and-debugging/
├── safety-and-robustness/
├── swe-benchmarks/
└── task-completion-metrics/
```

## 与其他目录的关系

- `../02-single-agent/` 提供被评估的单体能力对象。
- `../03-multi-agent/` 提供多 Agent 协作质量、成本与稳定性的评测对象。
- `../04-human-agent-interaction/` 提供人工评估、信任建立与协作体验验证语境。
- `../05-environments/` 提供 benchmark 是否可信、环境是否可复现的关键前提。
- `../06-frameworks-and-tools/` 提供具体产品与系统案例，便于把评测方法映射到真实工程对象。
