# 02 Single-Agent Index

> 如果你已经知道自己要找的是单智能体中的哪类能力、模式或跨主题问题，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `overview.md` 的整体理解主线。

## 目录结构

```text
02-single-agent/
├── planning/                    # 规划与任务拆解
├── memory/                      # 记忆机制与外部记忆设计
├── tool-use/                    # 工具使用与工具编排
├── reasoning-and-acting-loop/   # 推理-行动闭环
├── self-reflection/             # 反思与迭代修正
├── architectural-patterns/      # 单智能体架构模式
├── overview.md                  # 单智能体整体理解主线
├── backlog.md                   # 当前内容缺口
├── conflict.md                  # 已记录的口径冲突
└── *.md                         # 跨目录主轴专题与边界判断
```

## 按问题找入口

- 想看任务拆解、显式计划、plan-and-execute 或规划与执行边界：先看 `planning/`
- 想看短期记忆、长期记忆、检索式记忆或长上下文 vs 外部记忆：先看 `memory/`
- 想看 API calling、code interpreter、web browsing、MCP、tool orchestration：先看 `tool-use/`
- 想看推理与行动如何形成闭环：先看 `reasoning-and-acting-loop/`
- 想看 critique、iterative refinement、reflection trigger：先看 `self-reflection/`
- 想看 `ReAct`、RA-AID、AutoGPT pattern 等模式：先看 `architectural-patterns/`
- 想直接看单智能体与 tool workflow、planning/execution、tool-centric/monolithic 的边界问题：先看根目录下的跨主题专题文档

## 按阅读目标找入口

- 想先恢复单智能体的整体理解主线：读 [`overview.md`](./overview.md)
- 想看当前还有哪些内容缺口：读 [`backlog.md`](./backlog.md)
- 想看当前已记录的冲突：读 [`conflict.md`](./conflict.md)
- 想回到 Agentic 目录整体导航：读 [`../index.md`](../index.md)

## 跨主题专题

- `agent-vs-tool-workflow-boundary.md`：单智能体与 tool workflow 的边界
- `planning-vs-execution.md`：规划与执行如何分工
- `tool-centric-vs-monolithic.md`：工具中心与单体中心的结构对照

## 最小分工

- `README.md`：回答本目录是什么、为什么存在、与相邻目录如何分工
- `index.md`：回答进入本目录后，如果我已经知道自己要找哪类单智能体问题，该去哪一支
- `overview.md`：回答单智能体当前最值得先恢复的整体理解主线
