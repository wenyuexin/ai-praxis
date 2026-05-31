# Claude Code 官方规范与社区习惯（2026 复核版）

> 复核时间：2026-05
> 判定规则：官方文档与官方源码优先；社区实践可用但不替代官方规范。

## 判断标准

- 官方文档直接声明：判定为 [官方明确]
- 官方源码行为可证实：判定为 [官方明确]
- 文档未声明但社区广泛使用：判定为 [社区实践]
- 与官方建议冲突：判定为 [不推荐]

## 2026 最终判定表

| 主题 | 常见写法 | 判定 | 说明 | 依据 |
|---|---|---|---|---|
| Skills 目录结构 | `skills/kebab-case/SKILL.md` | [官方明确] | 目录和文件名是硬约束 | `https://code.claude.com/docs/en/skills` |
| Agent 最低字段 | `name` + `description` | [官方明确] | 任务匹配与调用依赖这两个字段 | `https://code.claude.com/docs/en/skills` |
| `tools` 写法 | `tools: [Read, Glob]` | [官方明确] | 数组写法与工具协议一致 | `https://github.com/anthropics/claude-code/blob/main/claude-code/src/tools.ts#L15` |
| `tools` 对象写法 | `tools: {name: Read}` | [社区实践] | 能运行但非官方推荐格式 | 官方示例均为数组 |
| 工具接入主路径 | MCP server（stdio） | [官方明确] | 官方主推接入方式 | `https://docs.anthropic.com/en/docs/claude-code/mcp` |
| 插件文档使用 | 仅依赖 plugins 文档页 | [谨慎使用] | 页面可访问但内容较精简 | `https://code.claude.com/docs/en/plugins` |
| 插件规范依据 | `src/plugin/` 源码 | [官方明确] | 细节规则以官方源码为准 | `https://github.com/anthropics/claude-code/tree/main/claude-code/src/plugin` |
| CLAUDE.md 文件格式 | Markdown（可含 frontmatter） | [官方明确] | 不支持纯 YAML 文件 | `https://docs.anthropic.com/en/docs/claude-code/claude-md` |
| CLAUDE.md 继承顺序 | Agent > 项目 > 用户 > 默认 | [官方明确] | 配置合并逻辑稳定 | `https://github.com/anthropics/claude-code/blob/main/claude-code/src/QueryEngineConfig.ts#L45` |
| Plan Mode 用法 | 复杂任务优先启用 | [官方明确] | 先规划后执行是推荐流程 | `https://code.claude.com/docs/en/best-practices` |
| 并行会话 | 多 worktree 并行 | [官方明确] | 用于任务隔离与提效 | `https://www.anthropic.com/engineering/claude-code-best-practices` |
| 脚本野路子 | `~/.claude/tools/` + Bash 直跑 | [不推荐] | 非官方主路径，可维护性差 | 与 MCP 官方路径冲突 |

## 可用但非推荐清单

- `tools` 对象写法：兼容性不稳，不建议作为团队规范
- 只看 plugins 文档页：会漏掉实现细节，应同时看 `src/plugin/`
- 以脚本替代 MCP：可临时验证，不适合长期治理

## 维护建议

- 每次版本更新后优先复核 4 个点：Skills、MCP、CLAUDE.md、Hooks
- 将“可用但非推荐”集中记录，避免团队误当标准
- 对外部经验贴统一加来源标签，防止社区习惯覆盖官方规范

## 来源

- `https://code.claude.com/docs/en/skills`
- `https://docs.anthropic.com/en/docs/claude-code/mcp`
- `https://docs.anthropic.com/en/docs/claude-code/claude-md`
- `https://code.claude.com/docs/en/best-practices`
- `https://code.claude.com/docs/en/plugins`
- `https://github.com/anthropics/claude-code/blob/main/claude-code/src/tools.ts#L15`
- `https://github.com/anthropics/claude-code/blob/main/claude-code/src/QueryEngineConfig.ts#L45`
- `https://github.com/anthropics/claude-code/tree/main/claude-code/src/plugin`
- `https://www.anthropic.com/engineering/claude-code-best-practices`
