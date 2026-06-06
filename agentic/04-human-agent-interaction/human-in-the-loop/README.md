# Human in the Loop（人在回路）

> 适用范围：人类在 Agent 规划、执行、审核、纠偏与接管环节的介入模式研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于沉淀人类在 Agent 执行闭环中介入的时机、角色与模式。

## 边界说明

### 语义界定

- 本目录中的 `human-in-the-loop` 特指人类在 agent 执行闭环中的介入时机、角色、权限和模式。
- **不泛指所有"有人参与"的系统**——用户使用 Agent 产出结果但不主动介入执行过程的行为不属于本目录的讨论范围。
- 与 `delegation-and-control/` 的区别：前者关注介入点和介入方式（何时、以什么角色介入），后者关注任务委托、控制权分配和责任边界（把什么交给 Agent、保留什么控制权）。

### 放在这里

人类在规划、执行、审核、纠偏、接管中的介入点设计、审批模式、异常处理与人工兜底机制。

### 不放在这里

任务委托与控制权分配（放 `../delegation-and-control/`）、交互表面设计与状态呈现（放 `../interaction-surfaces/`）、信任与对齐理论（放 `../trust-and-alignment/`）。

## 同目录导航

- 当前已包含专题：`human-in-the-loop-patterns.md`，用于展开不同介入模式的分类、场景和 trade-off。
- 建议结合 `../delegation-and-control/README.md` 区分"介入点"与"控制权分配"的边界。
- 建议结合 `../interaction-surfaces/README.md` 理解状态呈现如何支撑人工介入决策。
