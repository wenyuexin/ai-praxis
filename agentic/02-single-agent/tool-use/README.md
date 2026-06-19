# Tool Use

> 适用范围：单智能体工具发现、调用与约束研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理单智能体如何发现工具、调用工具、理解工具约束，并在执行过程中处理工具失败与外部接口边界。

## 快速入口

- 结合上层 `../README.md` 理解 tool-use 与 reasoning-and-acting-loop、memory、environments 的关系。
- 重点关注 API 调用、代码解释器、MCP、网页浏览与工具错误恢复。
- 如出现工具协议、接口边界或版本口径冲突，优先记录到 `../conflict.md`；若只是临时判断，可先做临时冲突整理，但不要长期替代稳定冲突入口。

## 边界说明

- 放在这里：工具发现、调用协议、参数约束、错误恢复、外部接口边界。
- 不放在这里：执行环境隔离与权限治理（放 `../../05-environments/`）、多智能体通信协议（放 `../../03-multi-agent/communication-protocols/`）、单纯规划逻辑（放 `../planning/`）。

## 同目录导航

- 相关主干专题：`tool-orchestration.md`，用于展开多工具调用序列的组织、约束与失败恢复。
- 相关主干专题：`tool-invocation-reliability.md`，用于展开工具调用中的参数构造、结果校验、失败分类与恢复路径。
- 相关跨目录专题：`../planning/bilevel-planning-tool-navigation.md`，用于展开复杂工具生态下 planner 与 tool navigation 的中间层结构。
- 相关上位主轴：`../tool-centric-vs-monolithic.md`，用于比较工具中心化路径与统一模型路径的系统组织差异。
- 相关边界专题：`../agent-vs-tool-workflow-boundary.md`，用于区分 tool-use 能力与真正形成执行闭环的 agent 系统。
- 如发现工具相关结论冲突，优先记录到 `../conflict.md`。

## 目录结构

```text
tool-use/
├── api-calling/
├── code-interpreter/
├── mcp/
└── web-browsing/
```
