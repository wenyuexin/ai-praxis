# Coding Agents and Tools（编码智能体与工具）

本目录用于沉淀面向软件工程任务的 Agent 工具与产品，包括 IDE、CLI、云端编码 Agent 与代码自动化工作流。

## 目录结构

```text
02-coding-agents-and-tools/
├── claude-code/
├── claw-code/
└── codex/
```

## 为什么独立成类

Coding Agent / SWE Agent 已形成独立问题域：它们面向仓库级软件工程任务，依赖代码定位、补丁生成、测试验证、版本控制、IDE/CLI 集成和 PR 流程，评估上也有 SWE-bench、SWE-Gym、SWE Atlas 等专门基准。

## 收录范围

适合放入本目录的对象：

- IDE / CLI 编码 Agent
- 云端 coding agent 产品
- repo understanding / code localization / patch generation 工具
- 与软件工程任务强绑定的 Agent 工具链

当前代表对象：

- `claude-code/`
- `claw-code/`
- `codex/`

后续可考虑补充：

- `openhands/`
- `swe-agent/`
- `cline/`
- `github-ace/`（若从 research prototype 进入正式产品或形成稳定资料）

## 边界

- SWE-bench 等 benchmark 的设计方法论主归属 `../../07-evaluation/swe-benchmarks/`。
- Docker、代码执行沙箱等环境设计主归属 `../../05-environments/code-execution-environments/`。
- 本目录重点分析具体编码工具的产品形态、架构、工作流和工程集成。

*最后更新: 2026-05-31*
