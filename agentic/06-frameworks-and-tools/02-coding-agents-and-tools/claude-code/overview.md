# Claude Code 总览

Claude Code 是终端原生的 AI 编程代理，能够在项目上下文中执行从分析、修改到验证的完整开发流程。

## 一句话定义

Claude Code = 终端原生 AI 编程代理 + Agent 调度器 + MCP 工具平台。

## 核心能力

- Skills / Agent：通过结构化定义复用角色能力
- MCP Tools：接入外部工具和内部系统
- CLAUDE.md：项目级持久化规则与记忆
- Plan Mode：先规划后执行
- Permission Modes：按阶段切换确认强度与执行边界
- SubAgent：并行与上下文隔离
- Checkpoints：变更快照与回滚

## 与 vibe coding human harness 的关系

在当前已整理的材料里，Claude Code 最稳定支撑的是 `constraint / acceptance / task-switch` 三类 human harness：用户可以通过 `plan`、`acceptEdits`、`auto` 等模式切换，在“先规划、再执行、再放大自治”之间调节控制强度。相比之下，`goal harness` 的显式产品支持仍弱，现有 `/goal` 材料更适合先保守写成“预设完成条件”能力，而不是稳定的结构化 goal 管理交互面。

## 与 IDE 补全工具的差异

- 工作粒度：不仅补全代码，还能完成跨文件任务
- 运行环境：终端优先，适合本地/远程/CI
- 工具调用：支持多轮工具编排与闭环执行
- 项目理解：面向仓库级上下文，而非单文件上下文

## 快速开始

```bash
npm install -g @anthropic-ai/claude-code
cd your-project
claude
```

## 延伸阅读

- `skills-agent.md`
- `mcp-custom-tools.md`
- `claude-md-config.md`
- `best-practices.md`
