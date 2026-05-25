# Claude Code 底层架构深度拆解

> 调研时间：2025-07-03（2026 H1 有效性已复核）
> 标注规则：[官方明确] / [社区实践] / [待验证]

## 1. 五层架构

```text
L1 交互层：REPL / 终端 UI / Slash 命令
L2 查询引擎：query.ts（循环、压缩、流式返回）
L3 工具系统：Tool.ts / tools.ts（工具协议与调度）
L4 Agent 系统：Agent.ts（主 Agent + SubAgent）
L5 记忆系统：Memory.ts + CLAUDE.md（长期/项目记忆）
```

## 2. 请求执行链路（时序）

```text
用户输入
-> query.ts 接收
-> 并行加载记忆与 skills
-> 提供基础工具并按需发现扩展工具
-> 必要时派发 SubAgent
-> 执行工具（只读并发/写入串行）
-> 流式返回并判断 compact/collapse
-> 终端渲染输出
```

## 3. 系统职责

| 系统 | 职责 | 核心文件 |
|---|---|---|
| 工具系统 | 工具协议、发现、执行与结果整合 | `Tool.ts` / `tools.ts` |
| 权限系统 | 写入控制、命令限制、风险分级 | `Permission.ts` |
| 记忆系统 | 项目规则、会话记忆与缓存 | `Memory.ts` |
| Agent 系统 | 任务分派、上下文隔离、协作策略 | `Agent.ts` |
| 查询引擎 | 主循环、容错、流式响应 | `query.ts` |
| Hooks 系统 | 生命周期扩展点（2026 H1 无新增） | `hooks.ts` |

## 4. SubAgent / Task / 并发意图

- Fork SubAgent：共享上下文，适合调查任务 [官方明确]
- `subagent_type`：独立上下文，避免污染 [官方明确]
- Task 面板：可视化任务状态与推进 [官方明确]
- 后台任务：前台继续思考，后台等待长任务 [官方明确]
- 多 worktree：并行任务隔离 [官方明确]

## 5. 关键设计权衡

| 权衡 | 选择 | 理由 |
|---|---|---|
| 性能 vs 安全 | 安全优先 | 高风险操作默认受控 |
| 上下文预算 vs 工具暴露 | 延迟工具发现 | 节省上下文预算 |
| 灵活性 vs 规范性 | MCP 主路径 | 提升可控与可审计 |
| 自治 vs 可控 | Plan 先行 | 降低跑偏概率 |

## 6. 证据与推断（更新）

- Agent frontmatter、工具数组、skills 结构： [官方明确]
- 延迟工具发现、工程控制面： [官方明确]
- Hooks 在 2026 H1 未见新增： [官方明确]
- `tools` 对象写法：可运行但非官方推荐写法 [社区实践]

## 7. 来源

- `https://github.com/anthropics/claude-code/tree/main/claude-code/src`
- `https://github.com/anthropics/claude-code/commits/main/claude-code/src/hooks.ts`
- `https://www.anthropic.com/engineering/claude-code-best-practices`
- `https://www.anthropic.com/engineering/advanced-tool-use`
