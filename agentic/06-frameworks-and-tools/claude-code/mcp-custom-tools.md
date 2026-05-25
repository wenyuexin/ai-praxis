# Claude Code 自定义工具（MCP）完全指南

> 调研时间：2025-07-03（2026 有效性已复核）
> 标注规则：[官方明确] / [社区实践] / [待验证]
>
> 复核结论：MCP 仍为官方主路径；插件文档页可访问但规范细节仍以源码为准。

## 1. 官方推荐接入方式

| 方式 | 正规度 | 判定 |
|---|---|---|
| MCP 服务器（stdio） | ⭐⭐⭐ | [官方明确] |
| Agent SDK `registerTool()` | ⭐⭐⭐ | [官方明确] |
| Shell 脚本直连 | ⭐ | [社区实践] |

## 2. 标准项目结构

```text
my-mcp-server/
├── mcp.json
├── package.json
└── src/
    ├── index.ts
    └── tools/
        ├── code-metrics.ts
        └── git-analyze.ts
```

## 3. `mcp.json` 示例

```json
{
  "mcpServers": {
    "my-tools": {
      "command": "npx",
      "args": ["-y", "tsx", "src/index.ts"],
      "env": { "NODE_ENV": "production" }
    }
  }
}
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `mcpServers` | ✅ | 服务映射 |
| `command` | ✅ | 启动命令 |
| `args` | ✅ | 命令参数 |
| `env` | 否 | 运行环境变量 |

## 4. Tool Schema 建议

- 参数必须显式 schema（常用 Zod）[官方明确]
- 错误要结构化返回 [官方明确]
- 长任务建议后台执行 [官方明确]
- 工具描述要清晰写明输入与输出 [官方明确]

示例：

```typescript
server.tool(
  "code-metrics",
  "计算文件的行数、复杂度、依赖数",
  { path: z.string().describe("要分析的文件路径") },
  async ({ path }) => {
    return { content: [{ type: "text", text: `analyzing ${path}` }] };
  }
);
```

## 5. 安全红线

| 红线 | 说明 |
|---|---|
| 路径穿越 | 禁止访问非白名单路径 |
| 硬编码凭证 | 凭证必须走环境变量 |
| 无上限执行 | 需要超时/轮数限制 |
| 过度写权限 | 默认只读，写操作需显式授权 |

## 6. 调试与诊断

| 步骤 | 操作 |
|---|---|
| 1 | `claude --mcp-log-level debug` |
| 2 | 查看 `~/.claude/logs/mcp-*.log` |
| 3 | 使用 `/mcp` 查看已注册工具 |
| 4 | 单工具测试（如支持） |

## 7. 三套配置模板

### A. 本地个人

```json
{
  "mcpServers": {
    "my-tools": {
      "command": "npx",
      "args": ["-y", "tsx", "/Users/me/my-mcp/src/index.ts"]
    }
  }
}
```

### B. 团队共享

```json
{
  "mcpServers": {
    "team-lint": {
      "command": "npx",
      "args": ["-y", "tsx", "./mcp/src/index.ts"],
      "env": { "TEAM": "frontend" }
    }
  }
}
```

### C. CI 自动化

```yaml
- name: Run Claude Code MCP
  run: |
    npx -y @anthropic-ai/claude-code \
      --mcp-config .github/mcp.json \
      --command "run tests"
```

## 8. 安全 Checklist

- 路径白名单已配置
- 所有参数均有 schema
- 无硬编码密钥
- 每个工具有超时策略
- 默认只读，写入受控
- 日志可审计

## 9. 常见故障排查

| 现象 | 原因 | 修复 |
|---|---|---|
| 工具不出现 | 配置路径/命令错误 | 校验 `command` 和路径 |
| 调用报错 | schema 不匹配 | 对齐参数定义 |
| 调用超时 | 任务过长 | 拆分或后台执行 |
| 权限拒绝 | 路径未授权 | 调整白名单 |
| 重复注册 | 多处重复配置 | 去重配置项 |

## 10. 来源

- `https://docs.anthropic.com/en/docs/claude-code/mcp`
- `https://github.com/anthropics/claude-code/tree/main/claude-code/src/mcp`
- `https://www.anthropic.com/engineering/mcp-protocol-design`
- `https://www.anthropic.com/engineering/advanced-tool-use`
