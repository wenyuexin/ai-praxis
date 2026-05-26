# Codex 官方文档导航

> **专题文档**。本文梳理官方文档的菜单结构、App UI 布局和优先级推荐，方便快速定位官方资料。不属于主线阅读路径，可与 `codex-agent-overview.md` 搭配查阅。



> 基于 `developers.openai.com/codex` 官方文档体系整理（2026.5 最新），可作为查阅官方资料的路线图。

---

## 一、官方文档的菜单/页面结构（6 大板块）

官方文档 hub 在 **`https://developers.openai.com/codex`**，导航分为以下 **6 大菜单页面**：

```
developers.openai.com/codex
│
├── 📘 Overview（概览）
│   └── Introducing Codex / What is Codex
│       → 定位、核心能力、与 ChatGPT 的区别
│
├── 🚀 Quickstart（快速开始）
│   └── Getting Started
│       → 安装方式（npm / Homebrew / 二进制）
│       → 三种工作模式：codex（交互式TUI）/ codex "..."（带提示）/ codex exec（非交互）
│       → 配置文件 config.toml 入门
│
├── 🖥️ App（桌面应用 —— 2026.2 上线，核心板块）
│   ├── App Features（功能清单）⭐ 最长的一页
│   │   ├── Multi-Agent Workflow（多 Agent 并行）
│   │   ├── Skills（技能系统）
│   │   ├── Automations（定时任务）
│   │   ├── Worktrees（Git 独立工作树）
│   │   ├── Built-in Terminal（内置终端）
│   │   ├── Built-in Browser（应用内浏览器）
│   │   ├── Computer Use（远程操控 GUI）⭐ 2026.4.16 新增
│   │   └── Non-code Files（非代码文件处理：PDF/Excel/PPT）
│   ├── Computer Use（专门页面）
│   │   → 远程操控浏览器、点击元素、填写表单
│   └── Remote Connections（远程连接）
│       → SSH 远程连接 / 远程设备控制
│
├── # CLI（命令行界面）
│   ├── CLI Reference（子命令大全）
│   │   ├── codex（默认，启动 TUI）
│   │   ├── codex exec（非交互批量执行）
│   │   ├── codex review（代码审查模式）
│   │   ├── codex login/logout（认证）
│   │   ├── codex mcp（MCP 服务器管理）
│   │   ├── codex app-server（App Server 模式）
│   │   ├── codex sandbox（沙箱操作）
│   │   ├── codex resume / fork（会话管理）
│   │   ├── codex apply（应用 diff）
│   │   └── codex cloud（云端执行）
│   └── Configuration（config.toml 完整配置）
│       → 模型选择 / 审批策略 / 沙箱模式 / Profiles
│
├── 📖 Guides（使用指南）
│   ├── AGENTS.md Guide ⭐ 最重要的一篇
│   │   → 规则文件怎么写、三层加载顺序、AGENTS.override.md 优先级
│   ├── MCP Guide（Model Context Protocol）
│   │   → 怎么连外部工具、官方 MCP Server 清单、自定义开发
│   ├── Sandboxing（沙箱机制）
│   │   → OS 内核级隔离（Landlock+Seccomp / Seatbelt / ACL+WFP）
│   ├── Config（配置管理）
│   │   → approval_policy 四级（untrusted / on-failure / on-request / never）
│   └── Profiles（配置集切换）
│       → full_auto / readonly_quiet 等预设
│
└── 🔧 Advanced（高级主题）
    ├── Custom MCP Servers（自定义 MCP 开发）
    │   → TypeScript / Python SDK、官方模板
    ├── Skills vs AGENTS.md（分工对比）
    ├── Security Model（安全模型）
    └── Architecture（架构解析：Rust + Tokio + ratatui TUI）
```

---

## 二、App 桌面端的 UI 菜单结构（实际使用时看到的）

Codex Desktop App（macOS / Windows）的界面分为 **左侧边栏 + 主工作区**：

```
┌─────────────────────────────────────────────────────┐
│  📁 Projects    │  💬 Conversations   │  🤖 Automations │  🧩 Skills    │  ⚙️ Settings  │
│  (项目列表)      │  (对话历史)          │  (定时任务)      │  (技能库)     │  (设置)       │
├─────────────────┴─────────────────────┴─────────────────┴───────────────┴───────────────┤
│                                                                                     │
│   ┌─────────────┐  ┌──────────────────────────────────────────────────────────┐     │
│   │  对话窗口    │  │              多功能区域                                    │     │
│   │  (流式输出)  │  │  · 终端 / 浏览器预览 / 文件预览 / Computer Use           │     │
│   │             │  │  · @引用文件  /  /命令  /  Skill 调用                     │     │
│   └─────────────┘  └──────────────────────────────────────────────────────────┘     │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

| 菜单 | 功能 | 官方文档对应页 |
|------|------|--------------|
| **Projects** | 项目管理，每个项目独立 AGENTS.md | App → Features → Worktrees |
| **Conversations** | 对话历史，支持 /plan /goal /review /status | CLI → commands |
| **Automations** | 定时任务（日报/CI 监控/告警） | App → Features → Automations |
| **Skills** | 技能库（个人 `~/.agents/skills` + 团队 `.agents/skills`） | Guides → Skills |
| **Settings** | 模型选择、审批策略、语言（中文/英文）、日常工作模式 | CLI → Configuration |

---

## 三、一张图总结：官方文档 → 实际产品的对应关系

| 官方文档页面 | 对应产品能力 | 优先级 |
|-------------|-------------|--------|
| `/codex/overview` | "这是什么" | ⭐ 先读 |
| `/codex/quickstart` | 装好能用 | ⭐⭐ 必须 |
| `/codex/app/features` | App 全部功能清单 | ⭐⭐⭐ 核心 |
| `/codex/guides/agents-md` | **AGENTS.md 怎么写** | ⭐⭐⭐⭐ 最实用 |
| `/codex/guides/mcp` | 怎么连外部工具 | ⭐⭐⭐ 进阶 |
| `/codex/concepts/sandboxing` | 安全机制 | ⭐⭐⭐ 企业必读 |
| `/codex/app/computer-use` | 远程操控 GUI | ⭐⭐⭐⭐ 2026 新特性 |
| `/codex/remote-connections` | SSH / 远程设备 | ⭐⭐ 特定场景 |
| `/codex/cli` | 命令行完整参考 | ⭐⭐⭐ CLI 用户必读 |

> 💡 **官方文档的隐藏逻辑**：`/codex/app/features` 是最长最全的一页（涵盖 Multi-Agent / Skills / Automations / Worktrees / Computer Use），相当于"产品说明书"；`/codex/guides/agents-md` 是社区公认**最值得反复读**的一页，因为 80% 的使用质量取决于 AGENTS.md 写得好不好。

---

## 参考来源

- https://developers.openai.com/codex
- https://openai.com/index/introducing-upgrades-to-codex/
- https://openai.com/index/introducing-the-codex-app/

*最后更新: 2026-05-26*