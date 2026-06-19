# Comparisons Index

> 如果你已经知道自己要找的是横向比较、选型判断、生态地图或跨对象边界收口问题，从这里开始。
> 边界：本文件负责 `05-comparisons/` 下的结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代具体比较专题正文。

## 目录结构

```text
05-comparisons/
├── orchestration-implementations.md   # 编排实现与 Agent contract 对照
├── tool-executor-boundary.md          # Tool Executor 的职责边界与执行分层
├── execution-governance-layers.md     # rules/skills/hooks/approval/sandbox 的执行约束分层
├── text-skill-governance.md           # 纯文本 skill agent 的规则遵守与输出治理
├── action-capable-agent-governance.md # 含代码/工具/环境副作用的复杂 agent 治理
└── candidates.md                      # 本目录下一步值得继续跟踪的论文与机制样本
```

## 按问题找入口

- **我想比较不同系统如何表达 task / execution / result / error / adapter contract**：读 [`orchestration-implementations.md`](./orchestration-implementations.md)
- **我想判断 Tool Executor 在系统里负责什么、和 tool orchestration / runtime environment / skill system 如何分工**：读 [`tool-executor-boundary.md`](./tool-executor-boundary.md)
- **我想直接看 Tool Executor 应该停在哪、哪些职责不该继续吞进去**：读 [`tool-executor-boundary.md`](./tool-executor-boundary.md) 中的 `Tool Executor Stop-line` 一节
- **我想判断 rules / skills / hooks / approvals / sandbox 分别在哪一层真正约束执行**：读 [`execution-governance-layers.md`](./execution-governance-layers.md)
- **我想研究当前仓库这种纯文本 skill agent，怎样更稳定地按规则工作**：读 [`text-skill-governance.md`](./text-skill-governance.md)
- **我想研究具备代码执行、工具调用和环境副作用的复杂 agent，如何把约束写进执行系统**：读 [`action-capable-agent-governance.md`](./action-capable-agent-governance.md)
- **我想看这一支接下来值得继续跟踪哪些论文、产品机制或边界样本**：读 [`candidates.md`](./candidates.md)

## 按阅读目标找入口

- **我想先判断这个目录是不是我要找的地方**：回读 [`README.md`](./README.md)
- **我想回到框架与工具层的整体入口**：读 [`../index.md`](../index.md)
- **我想恢复本层整体理解与对象分工**：读 [`../overview.md`](../overview.md) 和 [`../landscape.md`](../landscape.md)

## 按比较类型找入口

- **编排 / 协议 / workflow contract 对照**：`orchestration-implementations.md`
- **执行边界 / Tool Runtime / Skill-Tool 分工对照**：`tool-executor-boundary.md`
- **执行约束分层 / rules-skills-hooks-approval-sandbox 对照**：`execution-governance-layers.md`
- **纯文本 skill 治理 / 规则遵守率 / 输出约束**：`text-skill-governance.md`
- **复杂 agent 治理 / tool-loop / approval / sandbox / runtime**：`action-capable-agent-governance.md`
- **候选研究对象 / 论文队列 / 边界样本**：`candidates.md`
