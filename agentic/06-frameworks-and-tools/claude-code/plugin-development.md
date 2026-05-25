# Claude Code 插件开发指南

> 调研时间：2025-07-03（2026 H1 可用性已复核）
> 标注规则：[官方明确] / [社区实践] / [待验证]

## 1. 官方文档状态说明

- `https://code.claude.com/docs/en/plugins` 页面可访问
- 页面内容目前仍较精简
- 详细规范以官方源码 `src/plugin/` 为准

## 2. 最小插件结构

```text
my-plugin/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json (optional)
├── skills/
│   └── project-init/SKILL.md
├── commands/
├── agents/
└── README.md
```

## 3. `plugin.json` 字段

| 字段 | 必填 | 说明 | 示例 |
|---|---|---|---|
| `name` | ✅ | 插件唯一名（kebab-case） | `my-plugin` |
| `version` | ✅ | 语义化版本 | `1.0.0` |
| `description` | ✅ | 插件描述 | `Lint + format tools` |
| `skills` | 否 | skills 路径列表 | `["./skills/"]` |
| `commands` | 否 | commands 路径列表 | `["./commands/"]` |
| `agents` | 否 | agents 路径列表 | `["./agents/"]` |
| `hooks` | 否 | hooks 路径列表 | `["./hooks/"]` |

## 4. 最小可运行示例

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My custom Claude Code plugin",
  "skills": ["./skills/"],
  "agents": ["./agents/code-reviewer.md"]
}
```

```yaml
---
name: code-reviewer
description: Reviews code for quality
tools: [Read, Glob, Grep]
---
你是代码审查员。
```

## 5. 测试与发布 Checklist

- `plugin.json` JSON 合法
- `skills` 中使用 `SKILL.md`（大写）
- Agent frontmatter 最低字段齐全
- 版本号采用 semver
- 路径使用相对路径
- 通过 CLI 进行加载测试

## 6. 常见失败点

| 现象 | 原因 | 修复 |
|---|---|---|
| 插件不加载 | 清单文件错误 | 校验 `plugin.json` |
| Skill 不生效 | 文件名大小写错误 | 使用 `SKILL.md` |
| Agent 调用失败 | 名称不规范 | 改为 kebab-case |
| 版本冲突 | 版本号管理混乱 | 采用 semver 与变更日志 |

## 7. 来源

- `https://github.com/anthropics/claude-code/tree/main/claude-code/src/plugin`
- `https://code.claude.com/docs/en/plugins`
