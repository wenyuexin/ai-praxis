# Claude Code 官方文档与最佳实践指南

> 整理时间：2025-07-03｜2026-05 复核：通过｜适用版本：Claude Code 2025.x–2026.x

---

## 📘 主文档站

| 用途 | 链接 |
|---|---|
| 🇨🇳 中文主文档 | https://docs.anthropic.com/zh-CN/docs/claude-code/overview |
| 🇺🇸 英文主文档 | https://docs.anthropic.com/en/docs/claude-code/overview |
| 📖 完整参考手册（英文） | https://docs.anthropic.com/en/docs/claude-code/reference |
| 🏁 快速入门（中文） | https://docs.anthropic.com/zh-CN/docs/claude-code/quickstart |
| 🏁 快速入门（英文） | https://docs.anthropic.com/en/docs/claude-code/quickstart |

---

## 📂 官方文档目录结构

```
https://code.claude.com/docs/zh-CN
│
├── overview/                          # 概述
├── quickstart/                        # 快速开始
│   ├── macos/
│   ├── windows/
│   ├── linux/
│   └── wsl/
│
├── reference/                         # 完整参考手册
│   ├── cli/                           # CLI 参考
│   ├── mcp/                           # MCP 协议文档
│   └── claude-md/                     # CLAUDE.md 规范
│
├── claude-code/                       # 核心功能文档
│   ├── overview/
│   ├── quickstart/
│   ├── install/                       # 安装指南（全平台）
│   ├── terminal/                      # 终端 CLI
│   ├── vscode/                        # VS Code 扩展
│   ├── desktop/                       # 桌面应用
│   ├── web/                           # 网页版
│   ├── jetbrains/                     # JetBrains 插件
│   ├── automation/                    # 自动化 & Headless 模式
│   ├── enterprise/                    # 企业部署
│   ├── troubleshooting/               # 故障排除 FAQ
│   ├── skills/                        # Skills / Agent 规范 ⭐
│   ├── claude-md/                     # CLAUDE.md 项目配置 ⭐
│   ├── mcp/                           # MCP 工具脚本规范 ⭐
│   ├── best-practices/                # 最佳实践（50条技巧）⭐
│   ├── sandbox-security/              # 沙盒安全机制
│   ├── code-review/                   # GitHub Code Review 集成
│   ├── dispatch/                      # Dispatch 远程任务
│   └── channels/                      # Channels 推送事件
│
├── documentation-index/              # 文档索引（llms.txt 生成器）
│
└── (其他按需加载的子页面，共约 30+ 页)
```

---

## 🔧 核心模块文档（必读）

| 模块 | 官方链接 | 说明 |
|---|---|---|
| 🤖 **Skills / Agent 规范** | https://code.claude.com/docs/en/skills | Agent 文件格式（`.md` + YAML frontmatter）、`SKILL.md` 目录结构、字段定义 |
| 🔌 **MCP 协议（自定义工具脚本）** | https://docs.anthropic.com/en/docs/claude-code/mcp | MCP 服务器怎么写、怎么配、工具脚本组织方式 |
| 📄 **CLAUDE.md 项目配置** | https://docs.anthropic.com/en/docs/claude-code/claude-md | 继承链、配置优先级（全局 → 项目 → 模块） |
| ⚡ **最佳实践（50条技巧）** | https://code.claude.com/docs/en/best-practices | Plan Mode、并行会话、Hooks 等官方推荐 |
| 🛡️ **沙盒安全机制** | https://docs.anthropic.com/en/docs/claude-code/sandbox-security | 本地/云端沙盒实现原理 |

---

## 🔧 辅助模块文档

| 模块 | 官方链接 |
|---|---|
| 🔧 安装指南（全平台） | https://docs.anthropic.com/en/docs/claude-code/install |
| 🖥️ 桌面应用指南 | https://docs.anthropic.com/en/docs/claude-code/desktop |
| 🌐 网页版使用 | https://docs.anthropic.com/en/docs/claude-code/web |
| 🧪 自动化 & Headless 模式 | https://docs.anthropic.com/en/docs/claude-code/automation |
| 🏢 企业部署 | https://docs.anthropic.com/en/docs/claude-code/enterprise |
| 🐛 故障排除 FAQ | https://docs.anthropic.com/en/docs/claude-code/troubleshooting |

---

## 🔗 其他官方资源

| 资源 | 链接 |
|---|---|
| 🐙 GitHub 官方仓库 | https://github.com/anthropics/claude-code |
| 🎓 Anthropic Academy（官方教程） | https://www.anthropic.com/learn |
| 📘 官方 Skills 仓库（46.9k ⭐） | https://github.com/anthropics/skills |
| 📘 MCP 连接器目录 | https://github.com/anthropics/claude-code/tree/main/claude-code/src/mcp |
| 📘 工程博客：MCP 设计理念 | https://www.anthropic.com/engineering/mcp-protocol-design |
| 📘 工程博客：高级工具使用原理 | https://www.anthropic.com/engineering/advanced-tool-use |
| 📘 工程博客：最佳实践 | https://www.anthropic.com/engineering/claude-code-best-practices |
| 🔑 API 控制台 | https://console.anthropic.com/claude-code |

---

## 📋 官方建议 vs 社区习惯对照表

| 内容 | 官方建议 | 社区习惯 | 判定 |
|---|---|---|---|
| Agent 文件格式（`.md` + YAML frontmatter） | ✅ 官方源码 `Tool.ts` 定义 | ✅ 社区一致采用 | 🟢 官方 |
| YAML 字段（`tools:` 列表写法） | ✅ 官方 `Tool.ts` 定义 | ✅ 社区完全沿用 | 🟢 官方 |
| MCP 服务器组织方式（`mcp.json` + `src/tools/*.ts`） | ✅ 官方推荐 | ✅ 社区也这么干 | 🟢 官方 |
| Skills 目录结构（`skills/kebab-case/SKILL.md`） | ✅ 官方最佳实践 | ✅ 社区 100% 跟随 | 🟢 官方 |
| CLAUDE.md 配置继承链 | ✅ 官方设计（源码 `QueryEngineConfig`） | ✅ 社区当成协作规范 | 🟢 官方 |
| Hooks 机制（`PreToolUse` / `PostToolUse`） | ✅ 官方源码 `hooks.ts` | ✅ 社区大量使用 | 🟢 官方 |
| `tools: [Read, Glob, ...]` 写数组 vs 写对象 | ✅ 官方源码返回数组，写数组符合协议 | ⚠️ 有人写对象，是习惯不是要求 | 🟢 官方（数组写法） |
| 工具脚本放 `~/.claude/tools/` 用 Bash 调用 | ❌ 官方不推荐 | ✅ 纯社区野路子 | 🔴 社区 |
| `memory: project` 自动生成 `.claude/agent-memory/` | ✅ 官方源码 `ToolUseContext` 有此机制 | ✅ 社区当成最佳实践 | 🟢 官方 |
| 并行开多个 git worktree 跑多个 Claude 会话 | ✅ 官方创始人 Boris 亲自推荐 | ✅ 社区当成被低估的技巧 | 🟢 官方 |
| 先规划再执行（Plan Mode） | ✅ 官方强烈推荐（`Task` 工具设计初衷） | ✅ 社区当成铁律 | 🟢 官方 |

---

## ⚡ 快速定位：常见问题对应文档

| 你问的 | 对应官方文档 |
|---|---|
| Agent 文件怎么写（`.md` + YAML） | https://code.claude.com/docs/en/skills ✅ |
| `tools: [Read, Glob, ...]` 写法 | https://docs.anthropic.com/en/docs/claude-code/reference ✅ |
| MCP 服务器组织脚本 | https://docs.anthropic.com/en/docs/claude-code/mcp ✅ |
| CLAUDE.md 继承链 | https://docs.anthropic.com/en/docs/claude-code/claude-md ✅ |
| Skills 目录 `skills/kebab-case/SKILL.md` | https://code.claude.com/docs/en/skills ✅ |
| Plan Mode / Hooks / 并行会话 | https://code.claude.com/docs/en/best-practices ✅ |

---

## 🎯 核心判断标准

> 看 `src/Tool.ts` 和 `src/tools.ts`（官方 TypeScript 源码）里有没有对应的实现——**有的就是官方建议，没有的就是社区习惯**。

---

## 📌 最核心的两个链接（收藏这两个就够了）

| 用途 | 链接 |
|---|---|
| Skills / Agent 规范 | **https://code.claude.com/docs/en/skills** |
| MCP 工具脚本规范 | **https://docs.anthropic.com/en/docs/claude-code/mcp** |

---

*本文档基于 Claude Code 官方文档 + 官方 GitHub 源码整理，持续更新中。*