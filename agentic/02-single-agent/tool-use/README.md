# Tool Use

> 适用范围：单智能体工具发现、调用与约束研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理单智能体如何发现工具、调用工具、理解工具约束，并在执行过程中处理工具失败与外部接口边界。

## 快速入口

- 结合上层 `../README.md` 理解 tool-use 与 reasoning-and-acting、memory、environments 的关系。
- 重点关注 API 调用、代码解释器、MCP、网页浏览与工具错误恢复。
- 如出现工具协议、接口边界或版本口径冲突，记录到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：工具发现、调用协议、参数约束、错误恢复、外部接口边界。
- 不放在这里：执行环境隔离与权限治理（放 `../../05-environments/`）、多智能体通信协议（放 `../../03-multi-agent/communication-protocols/`）、单纯规划逻辑（放 `../planning/`）。

## 同目录导航

- 如后续需要整体综述，使用 `overview.md`。
- 如需要记录内容缺口，使用 `backlog.md`。
- 如需要学习或建设顺序，使用 `roadmap.md`。
- 如发现目录范围内冲突，使用 `conflict.md`。

## 目录结构

```text
tool-use/
├── README.md
├── api-calling/
├── code-interpreter/
├── mcp/
│   └── README.md
└── web-browsing/
```
