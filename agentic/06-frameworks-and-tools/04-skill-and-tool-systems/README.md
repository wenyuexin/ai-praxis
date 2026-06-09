# Skill and Tool Systems（技能与工具系统）

本目录用于沉淀 Skill、Tool、插件、能力注册与工具生态相关的工程实现对象。

## 目录结构

```text
04-skill-and-tool-systems/
├── mcp/
└── skill-based-agents/
```

## 收录范围

适合放入本目录的内容：

- Skill 定义、加载、组合、隔离和验证机制
- Tool registry、tool discovery、tool generation 的工程实现
- 插件系统和能力市场
- 与 MCP、function calling、工具权限控制相关的具体实现案例或协议对象

## 边界

- tool-use 的通用设计原则主归属 `../../02-single-agent/tool-use/`。
- 本目录重点关注 Skill / Tool 系统如何被工程化为可复用、可验证、可管理的组件。

*最后更新: 2026-05-31*
