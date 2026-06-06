# Claw Code 介绍文档

Claw Code 是一个开源的高性能 AI 代码代理框架，作为 Claude Code 的清洁室重写版本，于 2026 年 3 月 31 日在 GitHub 上正式发布。


## 一、项目背景与起源

2026 年 3 月 31 日，Anthropic 公司的 Claude Code 因一个 59.8 MB 的 source map 文件在 npm 包 `@anthropic-ai/claude-code` 中未剥离，导致 512,000 行未混淆 TypeScript 源代码（横跨 1,900 个文件）意外泄露。

韩国开发者 Sigrid Jin（GitHub 账号 `instructkr`）——此前曾被《华尔街日报》报道为 Claude Code 最活跃的超级用户之一——在第一时间采取了行动。面对 Anthropic 发起的 DMCA 撤诉行动，Jin 在凌晨 4 点开始从零用 Python 重写 Claude Code 的核心能力，而不是依赖泄露的快照作为长期分支。这一重写项目被命名为 **Claw Code**（全称：Rewriting Project Claw Code）。

项目发布后的成绩：
- **2 小时内** GitHub Stars 突破 50,000，创下 GitHub 历史上最快增长纪录
- **24 小时内** 超过 100,000 Stars
- **当前 Stars** 已突破 179K+，至今仍是 GitHub 增长最快的仓库之一

项目采用 **MIT 开源许可证**，完全开放源代码，项目地址：[github.com/instructkr/claw-code](https://github.com/instructkr/claw-code)。


## 二、项目定位

根据官方 README 的定义，Claw Code 是一个 **高性能、安全、原生工具执行环境**，旨在将 LLM 转化为自主编码代理。

项目围绕 **“clawable”哲学** 构建：构建一个确定性、机器可读、无需人工干预即可恢复的 Harness。它超越了“以人为先”的终端假设，转向一个事件原生架构——Agent 通过结构化钩子、会话和生命周期状态进行编排。

更具体地说，Claw Code 是一个可以理解自然语言指令，自主完成代码读取、修改、测试、Git 提交等全流程开发任务的“可编程的 AI 工程助手操作系统”。

Claw Code 也是 **UltraWorkers 工具链** 的核心组件，与以下工具协同工作：
- **clawhip**：事件流和协调层
- **OmX（oh-my-codex）**：持久化执行循环和并行代码审查的方法论
- **OmO（oh-my-openagent）**：开源 Agent 框架集成


## 三、技术架构

### 3.1 双语言混合架构

Claw Code 采用 **Python + Rust** 的双层架构设计，这是其区别于其他 AI 编程工具的核心特征。

| 层次 | 语言 | 职责 | 代码占比 |
|------|------|------|----------|
| 编排层 | Python | CLI 界面、配置解析、插件管理、用户交互，以及命令和工具的元数据镜像 | 约 27.1% |
| 执行层 | Rust | 代码解析、计划生成、执行监控、API 通信和工具沙箱执行环境 | 约 72.9% |



两层之间通过 **compat-harness 桥接层** 进行通信，确保数据结构在语言边界保持稳定。

### 3.2 五层架构

Claw Code 采用 **垂直分层 + 水平模块化** 设计，分为五层，层间依赖严格单向向下：

- **Layer 5（UI/CLI）** ：`claw-cli` Crate，基于 rustyline 实现 REPL，pulldown-cmark 渲染 Markdown，支持 200+ 语言的代码高亮
- **Layer 4（对话运行时）** ：`ConversationRuntime`，Agent 主循环所在地，管理 Session 生命周期
- **Layer 3a/3b（API + 工具）** ：`api` Crate 封装 HTTP 客户端 + SSE 流解析，支持 Anthropic/Grok/OpenAI 三个 Provider；`tools` Crate 管理 17 个内置工具
- **Layer 2（运行时支撑）** ：`runtime` Crate 的 12 个子模块——session、compact、permissions、hooks、oauth、config、prompt、usage、mcp、mcp_stdio、bash、file_ops
- **Layer 1（扩展服务）** ：commands（18 个 Slash 命令）、plugins（Hook 流水线）、lsp（LSP 诊断集成）、server（axum HTTP/SSE 服务）

### 3.3 核心 Crate 组件

官方文档将 Rust 工作区中的核心 Crate 划分如下：

| 组件 | 描述 |
|------|------|
| CLI (claw) | 主入口点，处理 REPL、参数解析和终端渲染 |
| Runtime | 系统的“大脑”，管理会话循环、权限和 MCP 生命周期 |
| API | 提供商无关的客户端层，支持 Anthropic 和 OpenAI 兼容模型 |
| Tools | 内置工具实现（bash、edit、grep）和 Agent 编排 |
| Plugins | 本地和远程钩子/工具的扩展层 |

### 3.4 Agent、任务与 Worker 编排

Claw Code 的编排层为后台进程、多 Agent 团队和自主编码 Worker 提供 **内存生命周期管理**，超越了原始终端传输，提供机器可读的控制平面以实现可靠执行和恢复。

编排系统通过一系列 Registry 实现管理：
- **TaskRegistry**：管理子 Agent 任务从创建到终态的全生命周期
- **WorkerRegistry**：实现编码 Worker 的状态机，处理信任门禁和 Prompt 传递
- **TeamRegistry 与 CronRegistry**：协调多 Agent 团队和定时任务
- **LspRegistry**：管理 LSP 动作以实现代码智能

Worker 的状态转换通过显式生命周期进行管理，包括：Spawning → TrustRequired → ReadyForPrompt → Running → Finished / Failed 等状态。

### 3.5 Agent 对话循环

Claw Code 的主循环 `run_turn()` 采用标准化的 Agent 对话循环设计：
- **无限循环与流式处理**：设定无限迭代阈值，支持无缝的工具调用链；使用 SSE 流式接收模型响应，边解析边累积文本缓冲区
- **工具权限与拦截**：执行任何工具前通过 PermissionPolicy 校验权限模式（ReadOnly / WorkspaceWrite / DangerFullAccess），支持生命周期钩子（pre_tool_use / post_tool_use）
- **容错机制**：工具执行失败不会直接中断对话流，而是将错误信息作为 ToolResult 返回给模型，由模型决定重试或调整策略


## 四、核心特性

### 4.1 工具系统

Claw Code 复刻了 Claude Code 的工具系统，提供 18 个内置工具，涵盖文件操作、命令执行、代码搜索、Web 抓取等核心能力。

工具管理体系分为三层：
- **Built-in Tools**：read_file、write_file、bash、web_fetch 等基础能力
- **Plugin Tools**：通过动态加载的扩展层
- **MCP Tools**：通过 stdio、HTTP 或 WebSocket 连接外部 MCP 服务器，自动发现可用工具并注册到运行时

### 4.2 权限系统

Claw Code 的三级权限系统确保操作安全可控：
- **ReadOnly**：仅读取操作
- **WorkspaceWrite**：工作区内写入操作
- **DangerFullAccess**：完全系统访问权限

### 4.3 子 Agent 架构

Claw Code 内置三种特化的子 Agent 角色：
- **Explore Agent**：只读探索和分析
- **Plan Agent**：规划和拆解任务
- **Verification Agent**：验证和验收

这种分工的根本原因在于：一个 Agent 同时做研究、规划、实现、验证，每件事都做不扎实。实现者和验证者不能是同一个角色。

### 4.4 MCP 协议

Claw Code 完整实现了 **模型上下文协议（Model Context Protocol）** 的客户端和服务端，支持与各种外部 MCP 服务器集成。

### 4.5 多 LLM 后端支持

Claw Code 设计了灵活的 LLM 后端适配系统，支持多种 AI 模型提供商，包括 OpenAI API、Anthropic Claude API 和 Google Gemini API。后端适配器采用工厂模式，提供统一的接口。

### 4.6 沙箱执行环境

出于安全考虑，AI 生成的代码会被放入隔离的 Docker 容器中执行：
- **权限限制**：以非特权用户运行，无法访问宿主机文件系统
- **资源隔离**：独立的网络命名空间和文件系统
- **超时控制**：默认 5 分钟超时，防止无限循环
- **资源配额**：限制 CPU、内存和磁盘使用量




## 五、与其他项目的对比

| 项目 | 架构 | 特点 |
|------|------|------|
| **claw-code** | Python + Rust 双层 | 清单驱动、可验证移植、多 Agent 编排、MCP 集成 |
| OpenClaw | 单层编排层 | 简单易用、Skills 生态、专注任务编排与状态追踪 |
| Claude Code | 闭源 | Anthropic 官方实现 |



需要注意的是，OpenClaw（原名 Clawdbot/Moltbot）是与 Claw Code 不同的项目：OpenClaw 是一个单层编排框架，专注任务编排、会话管理、状态追踪和异常重试，通过 ACP 协议对接执行引擎；而 Claw Code 是一个从零实现的双层 AI Agent Harness，直接包含完整的执行能力。

关于这一点的更多细节，你可以参考已有的对比分析。

## 六、部署与使用

### 6.1 环境要求
- Rust 工具链（通过 rustup 安装）
- Python 3.10+
- API Key（Claude / OpenAI / Gemini 之一）



### 6.2 构建与运行

```bash
git clone https://github.com/instructkr/claw-code.git
cd claw-code/rust
cargo build --release
./target/release/claw
```

进入交互式 REPL 后即可使用流式对话、工具调用和 Markdown 渲染输出等功能。

### 6.3 健康检查

构建完成后可通过 `claw doctor` 命令进行初始健康检查，验证环境配置是否正确。


## 七、社区与生态影响

Claw Code 的出现被视为“开源 AI 编程 Agent 领域的一场革命”。它不仅将闭源的 Claude Code 架构开源化，更提供了一个可研究、可修改、可扩展的工业级 AI Agent 参考实现。

在协同工程的发展方向上，Claw Code 提出了 **G-MAT（Generalized Multi-Agent Team）通用团队模型**——一套可落地、可审计、可进化的多 Agent 协同工程方案，将 AI 团队的协作从“临场发挥”转变为可复用的工程能力。

目前 Claw Code 仍在积极开发中，其团队工具和多 Agent 编排功能正在持续演进。如果你想深入了解某个具体模块，可以参考官方 DeepWiki（[deepwiki.com/instructkr/claw-code](https://deepwiki.com/instructkr/claw-code)）获取更详尽的技术文档。