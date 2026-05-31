# CLAUDE.md 项目配置完全指南

> 调研时间：2025-07-03（2026 有效性已复核）
> 标注规则：[官方明确] / [社区实践] / [待验证]

## 1. 加载顺序与继承优先级

| 优先级 | 路径 | 说明 |
|---|---|---|
| 1（最高） | `.claude/agents/xxx.md` 规则 | Agent 级覆盖 |
| 2 | `.claude/CLAUDE.md` | 项目级 |
| 3 | `~/.claude/CLAUDE.md` | 用户级 |
| 4（最低） | 默认内置规则 | 平台默认 |

- [官方明确] 参考：`QueryEngineConfig.ts` 合并逻辑

## 2. 冲突处理规则

- 项目级优先于用户级
- Agent 规则优先于 CLAUDE.md
- 同文件内部后定义覆盖先定义

## 3. 格式约束（2026 复核）

- CLAUDE.md 必须是 Markdown 文件
- 可包含 YAML frontmatter
- 不支持“纯 YAML 文件”作为 CLAUDE.md

示例：

```markdown
---
key: value
---
# 规则正文
```

## 4. 推荐写什么 / 不建议写什么

| ✅ 推荐 | ❌ 不建议 |
|---|---|
| 编码规范与边界 | 大量格式细节（交给 linter） |
| 构建/测试命令 | 高频变化的临时信息 |
| 禁止修改目录 | 大段实现代码 |
| 协作流程与验收标准 | 空泛口号 |

## 5. 篇幅与粒度建议

- 官方建议控制在 200 行内
- 过大时按子目录拆分
- 配合 `.claudeignore` 降低噪音上下文

## 6. 三套模板

### A. 个人全局模板

```markdown
# 个人偏好
- TypeScript 严格模式
- 日志统一 pino

# 禁止修改
- 禁止删除 .git/
```

### B. 团队项目模板

```markdown
# 编码规范
- 2 空格缩进
- 错误统一 AppError

# 运行命令
- Build: npm run build
- Test: npm test
- Lint: npm run lint:fix

# 禁止修改
- tsconfig.json
- migrations/
- .github/workflows/
```

### C. 企业多目录模板

```text
.claude/
├── CLAUDE.md
├── frontend/CLAUDE.md
├── backend/CLAUDE.md
└── .claudeignore
```

## 7. 反模式与替代

| 反模式 | 替代写法 |
|---|---|
| 把 README 全量复制进 CLAUDE.md | 只保留 AI 运行必需约束 |
| 单文件写 500 行规则 | 拆分目录级 CLAUDE.md |
| 写抽象口号 | 写可验证规则和命令 |
| 复制所有 lint 细节 | 写“执行 lint 命令” |
| 频繁大改 CLAUDE.md | 保持稳定，特例放 Agent |

## 8. 来源

- `https://docs.anthropic.com/en/docs/claude-code/claude-md`
- `https://github.com/anthropics/claude-code/blob/main/claude-code/src/QueryEngineConfig.ts#L45`
- `https://github.com/anthropics/claude-code/blob/main/claude-code/src/ConfigLoader.ts#L89`
