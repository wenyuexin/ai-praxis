# 07 Evaluation Index

> 如果你已经知道自己要找的是 Agent 评估中的哪类 benchmark、指标、人工评估或可观测性问题，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `overview.md` 的整体理解主线。

## 目录结构

```text
07-evaluation/
├── task-completion-metrics/      # 任务完成度与成功标准
├── agent-benchmarks/             # 通用 Agent 基准
├── swe-benchmarks/               # 软件工程任务基准
├── safety-and-robustness/        # 安全性、稳定性与鲁棒性
├── human-evaluation/             # 人工评估与主观反馈
├── observability-and-debugging/  # 可观测性、轨迹与调试
├── overview.md                   # 评估整体理解主线
└── backlog.md                    # 当前内容缺口
```

## 按问题找入口

- 想看任务成功标准、完成度指标或结果质量定义：先看 `task-completion-metrics/`
- 想看通用 Agent benchmark、长程任务或多工具任务评测：先看 `agent-benchmarks/`
- 想看 SWE-bench 等软件工程专项基准：先看 `swe-benchmarks/`
- 想看安全性、鲁棒性、稳定性或失败边界：先看 `safety-and-robustness/`
- 想看人工评估、主观反馈或协作体验验证：先看 `human-evaluation/`
- 想看轨迹、日志、回放、归因或调试方法：先看 `observability-and-debugging/`

## 按阅读目标找入口

- 想先恢复 Agent 评估的整体理解主线：读 [`overview.md`](./overview.md)
- 想看当前还有哪些内容缺口：读 [`backlog.md`](./backlog.md)
- 想回到 Agentic 目录整体导航：读 [`../index.md`](../index.md)

## 边界提醒

- `papers/` 与 `images/` 是辅助材料目录，不作为本层主题分类入口。

## 最小分工

- `README.md`：回答本目录是什么、为什么存在、与相邻目录如何分工
- `index.md`：回答进入本目录后，如果我已经知道自己要找哪类评估问题，该去哪一支
- `overview.md`：回答评估层当前最值得先恢复的整体理解主线
