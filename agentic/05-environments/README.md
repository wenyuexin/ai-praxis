# 05 Environments

> 适用范围：Agent 执行环境与安全边界研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于梳理 Agent 执行环境的类型、隔离边界、安全机制与生命周期管理，明确环境如何影响能力上限、稳定性与可治理性。

## 快速入口

- 从 `code-execution-environments/` 进入代码执行与依赖隔离问题，优先阅读 `code-execution-environments/workspace-structure.md`。
- 从 `sandboxing-and-safety/` 进入权限控制与安全边界问题，优先阅读 `sandboxing-and-safety/sandbox-layers.md`、`sandboxing-and-safety/permission-policy.md` 与 `sandboxing-and-safety/autonomy-vs-confirmation.md`。
- 从 `browser-environments/`、`simulated-environments/` 了解特定任务环境。
- 如遇证据冲突或版本口径不一致，统一记录到 `../temp/conflict.md`。

## 分类依据

本目录按执行环境的任务形态与治理问题划分：代码执行、沙箱与安全、浏览器环境、仿真环境，以及环境相关评测框架。

## 边界说明

- 放在这里：执行环境类型、隔离分层、权限模型、workspace/artifact、生命周期管理、恢复与审计。
- 不放在这里：单智能体内部推理与工具策略（放 `../02-single-agent/`）、多智能体协作机制（放 `../03-multi-agent/`）、具体框架案例本身（放 `../06-frameworks-and-tools/`）。

## 同目录导航

- 如后续需要整体综述，使用 `overview.md`。
- 如需要记录内容缺口，使用 `backlog.md`。
- 如需要学习或建设顺序，使用 `roadmap.md`。
- 如发现目录范围内冲突，使用 `conflict.md`。

## 目录结构

```text
05-environments/
├── README.md
├── benchmarking-frameworks/
├── browser-environments/
│   └── README.md
├── code-execution-environments/
│   └── README.md
├── sandboxing-and-safety/
└── simulated-environments/
```

## 与其他目录的关系

- `../02-single-agent/` 研究环境如何约束工具调用与执行策略。
- `../03-multi-agent/` 研究环境如何影响通信、隔离与协作成本。
- `../06-frameworks-and-tools/` 中的项目案例可为环境机制提供实证材料。
- `../07-evaluation/` 研究如何验证环境策略在稳定性、安全性和成本上的效果。
