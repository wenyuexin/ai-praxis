# Claude Code Agent / Skills 完全指南

> 调研时间：2025-07-03（2026 有效性已复核）
> 证据范围：仅采信官方来源（docs.anthropic.com / code.claude.com / anthropic.com/engineering / github.com/anthropics/*）
> 标注规则：[官方明确] / [社区实践] / [待验证]

## 1. Agent 与 Skill 的官方定义

| 术语 | 官方定义 | 边界 |
|---|---|---|
| Agent（子代理） | 可独立执行任务的 AI 实例，拥有独立 System Prompt 与工具集，可通过 `@name` 调用 | 有独立上下文，可 Fork 或独立运行 |
| Skill（技能） | 可复用的指令包，存放在 `skills/kebab-case/SKILL.md`，按需加载 | 不主动执行，本质是可复用说明/指令 |

- [官方明确] 参考：`https://code.claude.com/docs/en/skills`

## 2. Agent Frontmatter 字段

| 字段 | 必填 | 默认 | 说明 | 示例 | 判定 |
|---|---|---|---|---|---|
| `name` | ✅ | — | 调用名，供 `@name` 使用 | `code-reviewer` | [官方明确] |
| `description` | ✅ | — | 任务匹配说明 | `Reviews code for security` | [官方明确] |
| `tools` | 推荐 | 工具数组 | 可使用工具列表 | `[Read, Glob, Grep]` | [官方明确] |
| `model` | 否 | `sonnet`（常见） | 模型选择 | `sonnet`/`opus` | [社区实践] |
| `maxTurns` | 否 | `50`（常见） | 最大轮数防止循环 | `30` | [社区实践] |
| `memory` | 否 | `none` | 持久记忆级别 | `project` | [官方明确] |
| `include` | 否 | `**/*` | 允许路径过滤 | `["src/**/*.ts"]` | [社区实践] |
| `exclude` | 否 | `[]` | 禁止路径过滤 | `["node_modules/**"]` | [社区实践] |

## 3. `tools` 字段推荐写法

| 写法 | 判定 | 说明 |
|---|---|---|
| `tools: [Read, Glob, Grep]` | [官方明确] | 与官方工具数组协议一致 |
| `tools: {name: Read}` | [社区实践] | 能运行但非官方推荐，不建议用于规范文档 |

## 4. Skills 目录强约束

```text
skills/
└── kebab-case/
    └── SKILL.md
```

- 目录使用 `kebab-case`：[官方明确]
- 文件名必须 `SKILL.md`（大写）：[官方明确]

## 5. 三个最小可运行模板

### 模板 A：最简 Agent

```yaml
---
name: reviewer
description: Reviews code for security
tools: [Read, Glob, Grep]
---
你是代码审查员。
```

### 模板 B：带记忆

```yaml
---
name: devops
description: Deploy pre-check and git analysis
tools: [Read, Glob, Grep, Bash]
memory: project
maxTurns: 30
---
部署前必须运行 lint 和 git log -10 --stat
```

### 模板 C：路径约束

```yaml
---
name: test-writer
description: Write unit tests for TypeScript files
tools: [Read, Write, Glob]
include: ["src/**/*.ts"]
exclude: ["**/*.test.ts", "node_modules/**"]
---
只为 src/ 下的 .ts 文件写测试，不修改已有测试文件。
```

## 6. 常见错误与修复

| # | 错误 | 现象 | 修复 |
|---|---|---|---|
| 1 | `name` 含空格或大写 | `@name` 调用失败 | 改为 kebab-case |
| 2 | 缺失 `description` | 难以触发匹配 | 补全清晰描述 |
| 3 | `skill.md` 小写 | Skill 不加载 | 改为 `SKILL.md` |
| 4 | 目录非 kebab-case | 加载失败 | 改为短横线命名 |
| 5 | `tools` 写对象 | 解析不稳定 | 改为数组写法 |

## 7. 来源

- `https://code.claude.com/docs/en/skills`
- `https://docs.anthropic.com/en/docs/claude-code/reference`
- `https://github.com/anthropics/skills`
- `https://github.com/anthropics/claude-code/blob/main/claude-code/src/Tool.ts#L30`
- `https://github.com/anthropics/claude-code/blob/main/claude-code/src/tools.ts#L15`
