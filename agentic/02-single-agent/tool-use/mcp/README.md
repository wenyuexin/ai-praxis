# MCP（Model Context Protocol）

本目录用于沉淀 MCP 作为 Agent 工具使用与上下文接入协议的基础内容，重点关注它如何把外部工具、数据源和执行能力暴露给 Agent。

## 定位

MCP 更适合作为 `tool-use` 下的协议型专题，而不是放在 `06-frameworks-and-tools/` 下作为框架工具目录。

原因是：

- MCP 不是某个具体框架或产品，而是跨框架的工具与上下文接入协议。
- MCP 解决的是 Agent 如何发现、连接、调用外部能力的问题，属于 tool-use 的基础机制。
- 具体框架或工具对 MCP 的支持，应放在对应对象目录中作为实现案例。

## 适合沉淀的内容

后续可逐步补充：

- MCP 的设计动机与核心概念
- MCP Server / Client / Tool / Resource / Prompt 等基本抽象
- MCP 与 function calling、tool calling 的关系
- MCP 如何支持 tool discovery 与 capability exposure
- MCP 的安全边界、权限控制与执行风险
- MCP 在 Claude Code、OpenClaw、IDE 工具链等系统中的实现案例

## 与其他目录的关系

| 内容 | 归属 |
|------|------|
| MCP 的协议抽象、工具接入机制、设计动机 | `agentic/02-single-agent/tool-use/mcp/` |
| MCP 在多 Agent 通信或互操作中的角色 | `agentic/03-multi-agent/communication-protocols/` |
| MCP 在 Claude Code 中的具体配置和使用 | `agentic/06-frameworks-and-tools/02-coding-tools/claude-code/` |
| MCP 在 OpenClaw、Hermes Agent 等项目中的实现 | `agentic/06-frameworks-and-tools/03-project-studies/` |
| MCP server、插件、工具注册等工程实现 | `agentic/06-frameworks-and-tools/04-skill-and-tool-systems/` |

## 建议后续结构

```text
mcp/
├── README.md
├── overview.md
├── protocol-design.md
├── server-and-tools.md
└── security.md
```

当前先用 `README.md` 建立归属和边界，后续有系统材料后再拆分专题文档。

---

*最后更新: 2026-05-31*
