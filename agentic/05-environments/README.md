# 05 Environments

> 适用范围：Agent 执行环境与安全边界研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于梳理 Agent 执行环境的类型、隔离边界、安全机制与生命周期管理，明确环境如何影响能力上限、稳定性与可治理性。

## 快速入口

- 从 `overview.md` 进入整体理解框架，先把 `permission / execution / observability-recovery` 三层结构与 `minimal safety composition` 的总览建立起来。
- 从 `code-execution-environments/` 进入代码执行与依赖隔离问题，优先阅读 `code-execution-environments/workspace-structure.md`、`code-execution-environments/workspace-lifecycle.md`、`code-execution-environments/workspace-checkpoint.md`、`code-execution-environments/workspace-traceability.md`。
- 从 `sandboxing-and-safety/` 进入权限控制与安全边界问题，优先阅读 `sandboxing-and-safety/permission-policy.md`、`sandboxing-and-safety/permission-vs-execution-boundary.md`、`sandboxing-and-safety/sandbox-layers.md`、`sandboxing-and-safety/safety-composition.md`。
- 从 `browser-environments/`、`simulated-environments/` 了解特定任务环境，并从 `benchmarking-frameworks/` 进入环境评测与复现基座问题。
- 如遇证据冲突或版本口径不一致，统一记录到 `conflict.md`；若只影响临时判断，再回写 `../temp/conflict.md`。

## 分类依据

本目录按 Agent 环境中的核心治理问题划分：

- 代码执行环境：关注 workspace、依赖、artifact、副作用边界与恢复路径
- 沙箱与安全：关注 permission、execution isolation、governance 与最小安全组合
- 浏览器环境：关注网页交互、会话状态与页面级安全边界
- 仿真环境：关注受控任务世界与实验环境
- 环境相关评测框架：关注环境如何影响 benchmark 的可信度与可复现性

## 边界说明

- 放在这里：执行环境类型、隔离分层、权限模型、workspace/artifact、生命周期管理、恢复与审计。
- 不放在这里：单智能体内部推理与工具策略（放 `../02-single-agent/`）、多智能体协作机制（放 `../03-multi-agent/`）、具体框架案例本身（放 `../06-frameworks-and-tools/`）。

## 同目录导航

- 核心主题入口：`code-execution-environments/`、`sandboxing-and-safety/`、`browser-environments/`、`simulated-environments/`、`benchmarking-frameworks/`。
- 建议阅读顺序：先建立环境三层框架，再进入代码执行、沙箱安全与特定任务环境。

## 目录结构

```text
05-environments/
├── benchmarking-frameworks/
├── browser-environments/
├── code-execution-environments/
├── sandboxing-and-safety/
└── simulated-environments/
```

## 与其他目录的关系

- `../02-single-agent/` 研究环境如何约束工具调用与执行策略。
- `../03-multi-agent/` 研究环境如何影响通信、隔离与协作成本。
- `../06-frameworks-and-tools/` 中的项目案例可为环境机制提供实证材料。
- `../07-evaluation/` 研究如何验证环境策略在稳定性、安全性和成本上的效果。
