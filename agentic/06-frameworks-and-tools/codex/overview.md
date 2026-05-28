# Codex（全方位调研指南）

> ⚠️ **本文档是综合性调研材料**，不是主线文档。主线阅读路径请从 `codex-agent-overview.md` 开始。
> 以下内容整合了 OpenAI 官方文档、开源社区实战经验、架构分析，按 **认知 → 架构 → 使用 → 进阶 → 资源** 五层组织。

---

## 一、Codex 到底是什么？（5W1H 速览）

| 维度 | 内容 |
|------|------|
| **What** | OpenAI 面向软件开发场景的 **AI 编程代理（Coding Agent）**，不是"代码补全玩具"，而是能读仓库、改文件、跑命令、做审查的"工程级协作者" |
| **Why** | 开发者日常 47% 时间在等待工具串行处理，Codex 通过并发引擎将多任务执行时间缩短 60%+ |
| **Who** | 适合：需快速理解陌生代码库 / 重复性编码交给代理 / 终端直接改代码跑结果 / IDE 边写边改 |
| **When** | 命令模式 → CI/CD 自动化、快速生成代码；交互模式 → 开发项目、重构调试、长任务（**强烈推荐交互模式**） |
| **Where** | CLI 终端 / IDE 插件（VS Code / Cursor / Windsurf）/ 桌面 App / Web Cloud |
| **How** | 自然语言 Prompt → 模型推理 → 调用工具（读写文件/运行命令/检索资源）→ 迭代输出。以上流程为官方和社区综合描述 |

---

## 二、架构与核心实现（重点）

### 2.1 整体技术栈

| 层级 | 技术选型 |
|------|----------|
| **语言** | Rust 2024 Edition（性能 + 安全性） |
| **运行时** | Tokio 异步运行时（轻量级任务，非 OS 线程） |
| **构建系统** | Cargo 工作区（统一管理 48+ 内部 crate）+ just |
| **协议** | MCP（Model Context Protocol）连接外部工具 |
| **模型** | GPT-5-Codex（2025 年中发布）+ 可切换其他模型 |

📌 开源仓库（镜像）：`https://gitcode.com/GitHub_Trending/codex31/codex`

---

### 2.2 并发引擎架构（5 大核心组件）

这是 Codex 区别于传统 AI 编程工具的核心设计之一——将"请求-响应"同步模式转为 **"多生产者-多消费者"异步模式**。

```
用户请求
   │
   ▼
┌─────────────────────────┐
│  ① 任务解析与拆分器       │  codex-rs/core/src/tasks/task_parser.rs
│  递归下降算法，按依赖关系   │  将复杂任务拆为可并行子任务
│  拆分为 TaskGraph        │  例："分析代码并生成文档" → 语法分析/结构提取/文档生成
└───────────┬─────────────┘
            ▼
┌─────────────────────────┐
│  ② 优先级调度器           │  codex-rs/core/src/scheduler/priority_scheduler.rs
│  多级反馈队列算法          │  动态分配：IO 密集型→独立线程池，CPU 密集型→优先执行
│  考虑优先级 + 系统负载     │
└───────────┬─────────────┘
            ▼
┌─────────────────────────┐
│  ③ 资源隔离沙箱           │  codex-rs/core/src/sandboxing/resource_isolation.rs
│  命名空间 + 资源限制       │  每个任务独立环境：CPU limit=1, Memory limit=512MB
│  防止任务间资源竞争        │
└───────────┬─────────────┘
            ▼
┌─────────────────────────┐
│  ④ 异步通信层             │  codex-rs/core/src/communication/async_channel.rs
│  Tokio Channel (mpsc)     │  多生产者-多消费者，结果实时流式传输
│  任务间高效数据流转        │
└───────────┬─────────────┘
            ▼
┌─────────────────────────┐
│  ⑤ 结果聚合器             │  codex-rs/core/src/aggregator/result_aggregator.rs
│  收集子任务结果 → 合并     │  部分失败 → 重试 + 结果补偿 → 保证完整性
└─────────────────────────┘
```

**关键代码片段（任务拆分）**：
```rust
pub fn parse_task(request: &str) -> Result<TaskGraph> {
    let mut parser = TaskParser::new(request);
    let root_task = parser.parse()?;
    let task_graph = split_into_concurrent_tasks(root_task);
    // 根据依赖关系和资源需求拆分任务
}
```

**关键代码片段（Tokio 异步任务）**：
```rust
#[tokio::main]
async fn main() -> Result<()> {
    let stdin_handle = tokio::spawn(stdin_processor.run());
    let processor_handle = tokio::spawn(message_processor.run());
}
```

---

### 2.3 多智能体系统（Manager-Subagent 架构）

| 角色 | 职责 |
|------|------|
| **Manager（指挥官）** | 任务拆解、冲突仲裁、上下文路由 |
| **Subagent（执行者）** | 独立编码、并行开发，利用 **git worktree** 物理隔离环境 |

任务流转本质是 **有向无环图（DAG）分发与执行**。

---

## 三、使用方式（从入门到精通）

### 3.1 安装方式对比

| 方式 | 命令 | 推荐度 | 适用场景 |
|------|------|--------|----------|
| **npm 全局安装（CLI）** | `npm install -g @openai/codex` | ⭐⭐⭐⭐⭐ | 开发者首选，终端直接用 |
| **Homebrew（Mac）** | `brew install --cask codex` | ⭐⭐⭐⭐ | Mac 用户，不想装 Node.js |
| **GitHub Release 二进制** | 下载 `codex-x86_64-unknown-linux-musl.tar.gz` 等 | ⭐⭐⭐ | 不想用 npm |
| **IDE 插件** | VS Code / Cursor / Windsurf 插件市场搜 "Codex" | ⭐⭐⭐⭐ | 边写边改，上下文天然在编辑器 |
| **桌面 App** | `https://openai.com/zh-Hans-CN/codex` | ⭐⭐⭐⭐ | 图形界面，快速体验 |

> ⚠️ Node.js 建议 20+，国内用户可用 `--registry=https://registry.npmmirror.com` 加速

---

### 3.2 两种使用模式（核心认知！）

| | 命令模式（一次性调用） | 交互模式（Agent，强烈推荐） |
|---|---|---|
| **用法** | `codex "帮我写一个 FastAPI 接口"` | 输入 `codex` 进入对话界面 |
| **本质** | 一次输入 → 一次输出 → 结束 | 持续对话，能读项目、改文件、跑命令 |
| **适合** | CI/CD、快速生成代码、脚本 | 开发项目、重构调试、长任务 |
| **缺点** | 无上下文、不能迭代、每次重说需求 | 需要学习成本 |

**一句话总结：命令模式是"调用 AI"，交互模式是"雇一个 AI"**

---

### 3.3 核心命令速查

| 命令 | 功能 |
|------|------|
| `codex` | 启动交互模式 |
| `codex exec "xxx"` | 非交互执行（CI/自动化） |
| `codex --auto-edit` | 自动修改文件模式 |
| `codex --full-auto` | 全自动模式（自动执行所有操作） |
| `/model` | 切换模型 / 查看可用模型 |
| `/fast` | 切换 fast mode |
| `/permissions` | 控制是否允许修改文件/执行命令 |
| `/status` | 查看当前模型、目录、权限、MCP 状态 |
| `/init` | 自动生成 AGENTS.md 起始版 |
| `/plan` | 先出计划再动手（**高手必备**） |
| `/review` | 以 reviewer 视角审查自己写的代码 |
| `codex mcp` | 管理 MCP 服务器 |
| `codex mcp-server` | 以 stdio 形态启动 MCP server |
| `codex sandbox` | 在本地沙箱内运行命令 |
| `codex doctor` | 诊断本地安装/配置/认证/运行健康度 |
| `codex app` | 打开桌面 App（若缺失则引导安装） |
| `codex cloud` | 云端任务浏览与本地应用变更 |
| `codex resume` | 恢复之前的会话 |

---

### 3.4 三种安全/审批模式

| 模式 | 行为 |
|------|------|
| **Suggest（默认）** | 只建议修改，需你确认 |
| **Auto Edit** | 自动修改文件 |
| **Full Auto** | 自动执行所有操作（含跑命令） |

底层由 `config.toml` 中的 `approval_policy` 控制：`untrusted` / `on-failure` / `on-request` / `never`

---

## 四、工程化配置（这才是高手和新手的分水岭）

### 4.1 配置文件层级（优先级：命令行 > 项目级 > 用户级）

```
~/.codex/config.toml          ← 全局配置（默认模型、审批偏好、MCP）
项目根目录/AGENTS.md          ← 项目级长期说明（必须掌握！）
子目录/.agents/skills/        ← 技能库
~/.codex/rules/*.rules        ← 规则文件（允许/提示/禁止的命令）
```

---

### 4.2 AGENTS.md（最重要的文件，没有之一）

**作用**：不是替代 prompt，而是把 **每次都应遵守的工程规则固定下来**，让 Agent 每次开工前先读取。

**推荐模板**：
```markdown
# AGENTS.md
Project Overview
This is a Spring Boot monorepo for HRMS-related services.

Important Directories
- adapter/: controllers and API entrypoints
- app/: application services
- domain/: domain models and domain services
- infrastructure/: persistence and external integrations

Commands
- Build: mvn clean install -DskipTests
- Test: mvn test
- Run: mvn spring-boot:run

Rules
- Do not change public API contracts unless explicitly asked.
- Prefer minimal changes.
- Add tests for bug fixes when feasible.
- Keep SQL compatible with MySQL 8.

Done Criteria
- Code compiles
- Tests pass
- No unrelated refactors
```

> 💡 用 `/init` 快速生成起始版，但生成后 **必须自己改一遍**。当 Agent 反复犯同一个错时，把教训总结成规则写进 AGENTS.md——"你在一点点训练它"。

---

### 4.3 MCP（Model Context Protocol）

| 概念 | 说明 |
|------|------|
| **本质** | 让 Codex 连接外部工具/服务的协议 |
| **Codex 本体** | 负责推理 |
| **MCP Server** | 负责提供外部能力（访问数据库、文档服务等） |
| **协议传输形态（仓库证据）** | 基于 `https://github.com/openai/codex` 仓库核验，MCP 配置类型明确是 `stdio`（本地进程）与 `streamable_http`（远程） |
| **产品形态边界** | 本地仓库 README 将 CLI、Desktop App、Codex Web（cloud-based agent）分层展示；Code Interpreter 的云端沙箱结论不可直接外推到 Codex CLI |
| **配置位置** | `~/.codex/config.toml` 中 `mcp_servers` 部分 |
| **建议数量** | MCP + Skills 合计维持 **2-3 个**，多了上下文被稀释 |

---

### 4.4 Skills vs AGENTS.md（别搞混！）

| | AGENTS.md | Skills |
|---|---|---|
| **放什么** | 规则（无条件执行） | 流程（按需触发） |
| **例** | "提交信息必须用英文" | "分析这批日志" / "生成 Release notes" |
| **更新方式** | 手动写 / Agent 犯错后你加进去 | 用 `$skill-creator` 让 Agent 帮你生成 |
| **存储** | 仓库根目录 | `$HOME/.agents/skills`（个人）/ `.agents/skills`（团队） |
| **创建门槛** | 随时 | 只有任务反复出现才建 |

---

### 4.5 Automations（调度）

- Skills 决定"做什么"，Automations 决定"什么时候做"
- 例：每天扫 lint 警告、每周生成站会代码摘要
- ⚠️ **不稳定的流程千万别自动化**，先跑稳再加调度

---

## 五、OpenAI 内部最佳实践（7 大场景）

| 场景 | 用法示例 |
|------|----------|
| **理解陌生代码** | "总结一下请求从入口到返回响应在整个服务中如何流转" |
| **重构迁移** | "把这个文件按功能拆分成独立模块，并为每个模块生成测试" |
| **性能优化** | "优化这个循环提升内存效率，并解释为什么更快" |
| **提升测试覆盖** | "为这个函数编写单元测试，包含边缘案例和失败路径" |
| **修复 Bug** | 用 **Ask mode**："这个验证流程在哪里？" 粘贴 stack trace 让它定位 |
| **快速原型** | 规划新功能时用 5 轮提示词快速生成多个原型 |
| **反向采访** | 需求模糊时让 Codex 先问你澄清问题，再动手 |

> 💡 **Prompt 四要素**：Goal（做什么）+ Context（相关文件/报错，用 @ 引用）+ Constraints（架构约束/规范）+ **Done when（完成标准）**——最后一个最容易被忽略，加上后确认来回少一大半。

---

## 六、远程连接能力（2026 新特性）

| 功能 | 说明 | 官方文档 |
|------|------|----------|
| **Control this Mac** | 手机/别的设备遥控这台 Mac | [developers.openai.com/codex/app/computer-use](https://developers.openai.com/codex/app/computer-use) |
| **Control other devices** | 这台 Mac 遥控别的 Codex 设备 | 同上 |
| **SSH 远程连接** | Codex 进入远程服务器/开发机干活 | [developers.openai.com/codex/remote-connections](https://developers.openai.com/codex/remote-connections) |

---

## 七、资源链接汇总（建议收藏）

| 类别 | 链接 | 说明 |
|------|------|------|
| 🌐 **官方 Web App** | [https://chatgpt.com/codex](https://chatgpt.com/codex) | 直接浏览器用，需 ChatGPT 账号 |
| 🌐 **官方桌面版下载** | [https://openai.com/zh-Hans-CN/codex](https://openai.com/zh-Hans-CN/codex) | Windows / macOS |
| 📘 **官方开发者文档** | [https://developers.openai.com/codex](https://developers.openai.com/codex) | 完整 API + 指南 |
| 📘 **Remote Connections** | [https://developers.openai.com/codex/remote-connections](https://developers.openai.com/codex/remote-connections) | SSH / 远程设备连接 |
| 📘 **Computer Use** | [https://developers.openai.com/codex/app/computer-use](https://developers.openai.com/codex/app/computer-use) | 远程操控 GUI |
| 📦 **GitHub Releases（二进制）** | [https://github.com/openai/codex/releases](https://github.com/openai/codex/releases) | 各平台二进制下载 |
| 🐙 **开源镜像（Gitee）** | [https://gitcode.com/GitHub_Trending/codex31/codex](https://gitcode.com/GitHub_Trending/codex31/codex) | Rust 源码，48 个 crate |
| 📄 **OpenAI 内部实践 PDF** | [cdn.openai.com/pdf/6a2631dc.../how-openai-uses-codex.pdf](https://cdn.openai.com/pdf/6a2631dc-783e-479b-b1a4-af0cfbd38630/how-openai-uses-codex.pdf) | 251 篇实战整理，强烈推荐 |
| 📝 **CSDN 架构分析** | [blog.csdn.net/gitblog_01166/article/details/157419484](https://blog.csdn.net/gitblog_01166/article/details/157419484) | 并发引擎 5 组件代码级解析 |
| 📝 **CSDN 使用指南** | [blog.csdn.net/Daleuion/article/details/160021183](https://blog.csdn.net/Daleuion/article/details/160021183) | AGENTS.md / MCP / Skills 详解 |
| 📝 **CSDN 5W1H 全集** | [blog.csdn.net/qq_20042935/article/details/157217063](https://blog.csdn.net/qq_20042935/article/details/157217063) | 7 篇打通，可落地可复用 |
| 📝 **贡献指南** | [blog.csdn.net/gitblog_00432/article/details/152357019](https://blog.csdn.net/gitblog_00432/article/details/152357019) | 想参与开源必读 |

---

---

## 九、能力边界与局限

> 以下内容旨在提供客观的边界评估，避免工具选择时的盲点。非贬低 Codex 能力，而是帮助读者判断"什么场景用它、什么场景换别的"。

### 9.1 适用场景 vs 不适用场景

| 场景 | 适合 Codex | 更适合其他方案 |
|------|-----------|--------------|
| **大型单体仓库重构** | 有 AGENTS.md 和沙箱隔离时可行 | 对超大规模仓库（>10GB 源码），令牌消耗和上下文窗口仍是瓶颈，更适合人类分步拆解 |
| **非主流技术栈** | 基础支持，效果取决于模型训练数据覆盖 | Go/Python/TS/Java 等主流生态最佳；冷门语言或内部 DSL 表现显著下降 |
| **高安全 / 合规环境** | 有审批模式和沙箱，可管控 | 完全离线 / 空气间隙环境无法使用云端 API；敏感代码建议使用本地模型替代 |
| **创造性架构设计** | 可辅助生成候选方案 | 高层的系统级架构决策仍需人类主导，Agent 缺乏业务语境和长期战略视角 |
| **长周期持续任务** | 交互模式 + 恢复功能，可跨会话 | 超过单次上下文的超长任务可能丢失早期信息，需人工分段校验 |

### 9.2 已知局限

- **上下文窗口硬约束**：尽管支持长上下文，但工具调用历史、文件内容、MCP 通信会快速消耗令牌，实际可用窗口小于理论值
- **闭源黑箱风险**：模型推理过程不可完全审计，关键决策（如文件修改范围）依赖 Agent 的自我报告
- **工具生态依赖**：Codex 的工程化能力（读/写/运行）依赖底层工具链（MCP Server、系统命令）。外部工具故障或权限不足时，Agent 可能产生误导性输出
- **并发性能并非无限**：2.2 节的并发引擎在 IO 密集型任务中优势明显，但在 CPU 密集型计算（如大型编译器调用、密集数值计算）中提升有限，资源隔离本身也带来开销

### 9.3 竞品定位对比

| 维度 | Codex | Cursor | Copilot |
|------|-------|--------|---------|
| **本质定位** | AI 工程师（有手有脚） | AI 编辑器（深度 IDE 集成） | AI 补全助手（行级建议） |
| **任务复杂度** | 高（多文件、跨步骤） | 中（文件级重构） | 低（行/函数级） |
| **自动化程度** | 全自动到手动审批可选 | 半自动（需确认） | 被动触发 |
| **远程/云端能力** | 支持（SSH + Computer Use） | 不支持 | 不支持 |

---

## 十、调研路线建议

```
第 1 步：先装 CLI，用交互模式跑一遍（30 分钟建立体感）
  ↓
第 2 步：读官方文档 developers.openai.com/codex（建立完整认知）
  ↓
第 3 步：读 OpenAI 内部实践 PDF（理解真实工程用法）
  ↓
第 4 步：在自己项目里建 AGENTS.md + 配置 MCP（工程化落地）
  ↓
第 5 步：读 CSDN 架构分析文章（理解并发引擎实现）
  ↓
第 6 步：clone 开源仓库，看 Rust 源码（深入实现细节）
  ↓
第 7 步：尝试贡献 PR / 建自己的 Skill（社区参与）
```

> **一句话总结**：Codex 的定位不是"更聪明的代码补全"，而是一个 **能读仓库、改文件、跑命令、做审查的 AI 编程代理**。以上内容综合了官方文档和社区材料，部分细节仍需实际使用验证。

*最后更新: 2026-05-28*