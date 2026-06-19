# MCP

> 适用范围：工具接入与上下文暴露协议研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理 MCP 作为 Agent 工具使用与上下文接入协议的基础机制，重点关注它如何暴露外部工具、资源与执行能力。

## 快速入口

- 结合上层 `../README.md` 理解 MCP 在 tool-use 体系中的位置。
- 重点关注 tool discovery、capability exposure、client/server 结构、协议边界与安全问题。
- 如出现协议定义、版本差异或实现口径冲突，优先记录到上层 `../conflict.md`；若只是临时判断，可先做临时冲突整理，但不要长期替代稳定冲突入口。

## 边界说明

- 放在这里：MCP 的协议抽象、工具接入机制、上下文暴露、能力发现与安全边界。
- 不放在这里：MCP 在具体框架或产品中的实现案例（放 `../../../06-frameworks-and-tools/` 相关目录）、多智能体通信中的一般协议问题（放 `../../../03-multi-agent/communication-protocols/`）。

### 补充：MCP 在 single-agent 与 multi-agent 语境中的角色区分

MCP 当前在 `tool-use/` 下的定位适用于 **single-agent 语境**：

- 在 single-agent 语境下，MCP 作为工具发现、工具连接、工具调用和外部能力集成协议讨论。
- MCP 的跨 agent 通信或多 agent 协议角色，应放到 `../../../03-multi-agent/communication-protocols/` 中讨论。
- 两者通过交叉引用衔接；本轮不迁移 `tool-use/mcp/`。

## 同目录导航

- 如后续需要整体综述，使用 `overview.md`。
- 如需要记录内容缺口，使用 `backlog.md`。
- 如需要学习或建设顺序，使用 `roadmap.md`。
- 如发现目录范围内冲突，使用 `conflict.md`。
