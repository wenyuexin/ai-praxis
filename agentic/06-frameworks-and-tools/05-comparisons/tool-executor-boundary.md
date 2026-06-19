# Tool Executor Boundary

> 适用范围：Agent 系统中 Tool Executor 的职责边界、执行分层与相邻专题关系
> 阶段状态：第一版横向专题，基于现有主干文档做保守收口
> 使用说明：本文回答“Tool Executor 在系统里到底负责什么、它与 tool orchestration / code execution environment / skill system 的边界如何划分、哪些维度已经可以稳定讨论”，不直接冻结某一实现方案，也不把单对象实现写成通用标准。

## 一、问题定位

当一个 Agent 系统开始同时面对工具调用、代码执行、依赖隔离、workspace、artifact、权限控制与失败恢复时，问题就不再只是“模型会不会调用工具”。

更核心的问题变成：

- 工具调用请求如何被接住
- 工具到底在哪里执行
- 执行前后需要声明哪些约束
- 结果、错误、副作用与 trace 如何被统一表达
- 哪些能力属于 Tool Executor，哪些应交给 planning、runtime environment 或 skill system

因此，Tool Executor 更适合被理解为：**连接工具调用意图与真实执行实体的系统边界**，而不是一个狭义的函数调用器。

---

## 二、为什么它值得单独收口

当前与 Tool Executor 相关的研究已经分散长在三类目录里：

- `../../../../02-single-agent/tool-use/`：更关注工具发现、调用、编排、失败恢复与执行闭环
- `../../../../05-environments/code-execution-environments/`：更关注执行位置、依赖隔离、workspace、artifact、生命周期与恢复
- `../04-skill-and-tool-systems/`：更关注 Skill / Tool 如何被工程化为可复用、可验证、可管理的制品

这种分散本身并不错误，因为 Tool Executor 本来就横跨“调用逻辑”“执行环境”“制品治理”三层。

真正缺少的不是把材料搬到同一个目录，而是：**给这三层之间补一个稳定的边界视图**。

---

## 三、一个最小工作定义

当前阶段，可以先用一个保守工作定义来理解 Tool Executor：

> Tool Executor 负责把“可调用工具”转成“受约束的真实执行”，并把执行结果重新转成系统可消费的结构化产物。

按这个定义，Tool Executor 至少涉及五类最小职责：

1. **execution path selection**：决定某类 tool 走哪条执行路径
2. **input / output adaptation**：把调用输入映射到执行实体，再把结果映射回系统结果
3. **dependency / environment binding**：决定依赖、runtime、workspace、artifact 的绑定方式
4. **policy / side-effect control**：决定权限、风险、副作用声明与审批边界
5. **trace / error / recovery hooks**：决定执行日志、错误分类、超时、取消、恢复入口如何暴露

这个定义故意保持保守：它不要求 Tool Executor 自己完成 planning、multi-agent coordination、完整 sandbox 治理或高层产品交互，但它必须为这些上层能力提供可接入的执行边界。

---

## 四、它不负责什么

为了防止概念膨胀，当前也应明确 Tool Executor 不直接等同于以下东西：

### 4.1 不等于 tool orchestration

`tool orchestration` 关注的是：

- 多工具调用顺序如何组织
- 步骤依赖如何表达
- 失败后是局部重试、换工具还是回到重规划
- 中间状态如何在执行链中传递

这些问题更偏执行闭环和工作流结构；Tool Executor 更偏“某一步一旦决定调用某类 tool，系统如何把它安全、可追踪地执行出来”。

可以粗分为：

- orchestration 决定“什么时候、按什么结构调用工具”
- Tool Executor 决定“这个工具调用具体如何落到执行实体上”

### 4.2 不等于 code execution environment

`code execution environment` 更关注：

- 本地进程、subprocess、venv、container、remote runtime 等执行形态
- workspace、artifact、rollback、checkpoint、lifecycle
- 执行环境和恢复语义的边界

这些问题提供了 Tool Executor 所依赖的运行时地基，但 Tool Executor 不是环境本身。它更像站在调用侧与环境侧之间的连接层。

可以粗分为：

- environment 定义“动作在哪里发生、隔离多强、状态如何保存”
- Tool Executor 决定“哪类 tool 应绑定哪类环境，以及调用结果如何接回系统”

### 4.3 不等于 skill system

`skill system` 更关注：

- Skill 如何声明、加载、组合、验证与治理
- Tool registry、tool discovery、plugin / market、协议对象如何工程化
- Tool / Skill 如何成为可复用、可验证、可管理的制品

Tool Executor 可能消费 skill manifest、tool registry 与 capability metadata，但它不等于这些制品系统本身。

可以粗分为：

- skill system 负责“系统里有哪些能力、如何被声明与发现”
- Tool Executor 负责“这些能力一旦被选中，如何被真实执行” 

---

## 五、当前可以稳定讨论的执行分层

结合现有问题地图与环境层材料，当前至少可以保守讨论这样一组执行分层：

| 工具类型 | 候选执行路径 | 当前可稳定讨论的边界 |
|---|---|---|
| 内置可信 tool | 同进程函数调用 | 受版本与测试约束，执行开销低，但副作用与依赖边界需要更清楚声明 |
| 项目级 tool | subprocess + 当前项目环境或指定 venv | 适合需要依赖隔离、文件产物、测试运行的任务 |
| 第三方 / 不可信 tool | 隔离 venv 或容器 | 重点是限制默认文件系统与网络访问，以及审计与清理路径 |
| 高风险代码执行 tool | 容器 / sandbox | 需要明确权限、资源限制、超时、日志与恢复入口 |

当前这张表仍属于 `Observed / Inferred` 层的保守抽象，不代表所有系统都采用同一路线，但它已经足以支撑下一轮对象补证。

---

## 六、Tool Executor 最值得看的五个边界面

### 6.1 Execution Path Boundary

首先要回答：**这类 tool 应该在哪个执行面上运行。**

关键区分包括：

- 同进程 vs subprocess
- 当前项目环境 vs 独立 venv
- 本地 runtime vs 容器 / sandbox
- 长驻 worker vs 一次性执行

### 6.2 Dependency Boundary

其次要回答：**依赖属于谁，如何声明、安装、缓存与清理。**

关键问题包括：

- tool 是否允许携带独立依赖
- 依赖是否污染主进程或项目环境
- 依赖版本与执行结果是否可复现
- 不同 tool 是否共享环境还是独立隔离

### 6.3 Workspace / Artifact Boundary

再往下要回答：**tool 对哪些工作域与产物拥有读写能力。**

关键问题包括：

- tool 是否绑定 task-specific workspace
- 中间产物、patch、logs、测试结果如何归档
- tool 输出的是结构化结果、文件产物，还是两者都有
- 失败后哪些 artifact 应保留用于诊断与恢复

### 6.4 Side-Effect / Permission Boundary

还要回答：**tool 有权造成哪些副作用，哪些动作需要审批或限制。**

关键问题包括：

- 写文件、运行命令、访问网络、修改依赖是否默认允许
- 哪些 tool 属于可信内置能力，哪些属于高风险执行能力
- side effects 是否需要显式声明与用户确认
- policy 拒绝、权限不足、审批中断如何进入结果与错误模型

### 6.5 Trace / Error / Recovery Boundary

最后要回答：**执行后系统如何解释发生了什么。**

关键问题包括：

- 工具调用日志、stdout/stderr、structured result 如何记录
- error type 如何分类：input、dependency、execution、timeout、policy reject
- cancel、retry、resume、rollback 的入口落在哪一层
- traceability 是记录调用行为，还是还能支持恢复入口

---

## 七、与现有专题的关系

### 7.1 与 `../../../../02-single-agent/tool-use/tool-orchestration.md`

该文关注多工具调用序列如何组织，重点是顺序、依赖、分支与失败恢复。它可以回答“为什么工具使用不只是单次调用”，但不负责定义执行路径、依赖隔离与 workspace 绑定。

### 7.2 与 `../../../../05-environments/code-execution-environments/README.md`

该目录关注代码执行、测试运行、依赖隔离、workspace、artifact 与生命周期。它为 Tool Executor 提供运行时地基，但不单独回答 Tool / Skill 在系统层如何形成统一执行边界。

### 7.3 与 `../04-skill-and-tool-systems/README.md`

该目录关注 Skill / Tool 的工程化对象：声明、加载、组合、验证、registry 与协议实现。它更像 Tool Executor 的上游 capability surface，而非执行本身。

### 7.4 与 `orchestration-implementations.md`

后者更关注 `TaskSpec / ExecutionSpec / ResultSpec / ErrorSpec / AdapterSpec` 这类 orchestrator / adapter contract。Tool Executor 边界与它高度相关，但范围更窄，更聚焦“单个 tool capability 如何真实执行并回流结果”。

---

## 八、当前研究推进到哪里

当前可以确认的不是“Tool Executor 方案已经定了”，而是：

- 顶层 backlog 已经把它立为独立问题，且明确需要 Tool Executor 分层表和 Skill manifest 最小字段清单
- 单智能体层已经补出 tool orchestration、tool reliability 等相邻主线
- 环境层已经补出 workspace、checkpoint、traceability、lifecycle 与 recovery 相关主线
- 框架 / 工具层已经开始沉淀 skill / tool system 对象，以及 orchestration contract 的横向抽象

因此，当前最准确的状态不是“空白”，也不是“已定型”，而是：**边界问题已被明确提出，相关证据已分散长出，当前这一篇负责做第一轮横向收口。**

## 九、现有对象的执行路径对照（第一版）

下面这张表只收当前主干中已经有相对明确支撑的对象，不把未核验细节外推成统一规格：

| 对象 | 当前可确认的执行入口 / 载体 | 对 Tool Executor 边界最有价值的观察 |
|---|---|---|
| `OpenHands` | `app_server` 协调 sandbox；可走 Docker sandbox、process sandbox、remote sandbox；workspace 通过 agent-server / SDK remote workspace 接入 | 更像“conversation / sandbox identity 驱动的远端执行体系”；Tool Executor 不能只理解为单次命令执行，还要处理 workspace、session、resume、artifact 与 agent-server 桥接 |
| `SWE-agent` | `sweagent run` / `run-batch` 驱动 `SWEEnv`；底层通过 SWE-ReX deployment / runtime 执行；默认 Docker deployment，可切 local / cloud deployment | 更像“task / instance 驱动的一次执行环境”；Tool Executor 边界和 repo reset、trajectory、batch worker、stateful shell session 强相关，而不是长期 conversation restore |
| `Claude Code` | 主循环中执行工具；MCP 是官方主路径；自定义工具以 `stdio` MCP server 或 Agent SDK `registerTool()` 接入 | 更像“本地工具系统 + 受控扩展工具入口”；Tool Executor 要同时面对工具发现、权限控制、只读并发 / 写入串行、以及 MCP server 作为外部执行面 |
| `Codex` | Local / Worktree 在本机运行；Cloud 在 OpenAI 托管容器中运行；MCP 作为外部能力扩展协议；本地还有 `codex mcp-server` / `codex sandbox` 等入口 | 更明确地区分“本地受限执行边界”和“Cloud 远程容器边界”；Tool Executor 不能把 function/tool calling 与真实执行位置混写，approval policy 与 sandbox mode 也不能混成一个概念 |
| `MCP` | 协议层支持 `stdio` subprocess 与 `streamable_http` transport；定义 tool discovery / invocation / schema / structured result | 它更像 Tool Executor 的 capability surface / protocol surface，而不是完整执行器；能说明工具怎么被发现和调用，但不能单独说明依赖隔离、workspace 生命周期或 side-effect recovery |

当前可以先保守提炼出四条跨对象观察：

1. **执行入口并不统一。** 有的对象从 conversation / sandbox 进入，有的从 CLI / batch task 进入，有的从本地工具循环进入。
2. **执行载体至少分三类。** 同进程 / 本机受限执行、subprocess / session、远程容器 / sandbox 是当前最稳定的三类观察面。
3. **workspace 是否是一等对象，会改变 Tool Executor 的职责厚度。** OpenHands、SWE-agent 都把 workspace / repo / trajectory 放进执行链；MCP 则没有承担这层语义。
4. **协议面与执行面必须拆开。** MCP、function calling、tool schema 可以表达调用意图，但执行路径、依赖隔离、权限策略、artifact 与恢复入口仍要靠更下层实现补足。

### 9.1 字段级执行器对照（第一版）

下面这张矩阵不追求“所有对象字段完全对齐”，只挑当前已有主干支撑、且最接近 Tool Executor 边界的几个观察维度：

| 对象 | 主要执行载体 | workspace / repo 绑定 | 权限 / side-effect 控制 | trace / recovery / cleanup |
|---|---|---|---|---|
| `OpenHands` | Docker sandbox、process sandbox、remote sandbox；workspace 通过 agent-server / SDK remote workspace 接入 | workspace 是显式对象；conversation 可复用既有 sandbox；working_dir、volume、agent-server URL、session API key 都进入执行链 | 具体 policy 仍分散在 sandbox / service / product 层；当前更稳的是它有显式 sandbox 边界，而不是已收敛的 per-tool policy 规格 | event 持久化、conversation restore、sandbox pause / resume、workspace backend cleanup 都已存在；但“完整文件系统可独立重建”仍未证实 |
| `SWE-agent` | `SWEEnv` + SWE-ReX deployment / runtime；默认 Docker deployment，也支持 local / cloud deployment | repo 是 `EnvironmentConfig` 的显式组成部分；每次启动会 copy / reset repo，更偏 per-instance / per-attempt 工作域 | side effects 主要通过 repo reset、deployment 边界与环境配置间接控制；更细 per-tool approval / policy 仍未形成稳定主线 | trajectory、patch、logs、predictions 明确产出；`deployment.stop()` 提供关闭入口；git reset 能恢复 repo 文件状态，但不能外推为完整副作用恢复 |
| `Claude Code` | 本地主循环执行工具；官方文档确认可连接 MCP，并支持在 CLI 中调度 recurring tasks；现有主干还能观察到 `stdio` MCP server 与 Agent SDK `registerTool()` 两条扩展入口 | 当前更像“本地工作目录 + 项目记忆 + skills 上下文”的工具系统；workspace 是隐含执行背景，不像 OpenHands 那样被显式建模为远端对象 | 权限系统是独立一层；写入控制、命令限制、风险分级明确存在；只读并发 / 写入串行说明执行调度本身也受治理约束 | 工具执行结果会回流查询引擎；官方文档可确认它支持流式返回、schedule / loop 这类长任务形态；但 cleanup / recovery 的统一执行器语义在现有主干中仍偏薄 |
| `Codex` | Local / Worktree 在本机；Cloud 在 OpenAI 托管容器中；官方文档还明确了 sandbox applies to spawned commands，以及 background worktrees / local environments / MCP server config 这些入口 | Local / Worktree 更像本机目录或 Git worktree 绑定；官方 worktree 文档确认 background worktrees 与 local environment setup；Cloud 走远程环境；执行位置与修改隔离需要分开讨论 | governance 由 sandbox mode + approval policy 共同控制；官方文档明确 `workspace-write`、`read-only`、`danger-full-access` 与 `untrusted` / `on-request` / `never` 等审批模式；MCP 还支持 server-level / per-tool approval | Cloud 有环境缓存与恢复语义，但不宜外推为固定时长或统一恢复规格；官方文档已能确认本地与 worktree 路径存在稳定 workspace 绑定，但 cleanup / retry / rollback 仍待更细核验 |
| `MCP` | `stdio` subprocess 或 `streamable_http` transport 承载协议消息 | 只有 `roots` 这类 filesystem boundary 协议能力，不承担 workspace / repo 生命周期语义 | transport-level authorization、cancellation、progress 可确认；但这不等于 per-tool approval、side-effect policy 或组织级治理 | 有 structured result、progress、cancellation 等协议信号；没有 checkpoint、workspace recovery、side-effect rollback 或统一 cleanup 契约 |

这张矩阵支持一个更窄的判断：Tool Executor 至少要同时面对 **执行载体选择、工作域绑定、治理约束、以及结果/恢复信号回流** 四类问题；其中任何一类被忽略，都会把执行器误缩成“函数调用适配器”。

### 9.2 timeout / retry / cleanup / result envelope（第一版）

如果继续把 Tool Executor 往更细字段压，当前主干里最有支撑的一组字段是：`timeout`、`retry`、`cleanup`、`result envelope`。

| 对象 | timeout / 长任务控制 | retry / cancel | cleanup | result envelope |
|---|---|---|---|---|
| `OpenHands` | 已有显式 sandbox 生命周期与 pause / resume；长任务控制更多体现为 sandbox / conversation service 级生命周期，而非单一 tool timeout 字段 | resume / pause / delete 已存在；但更细的 per-tool retry / cancel contract 仍未在当前主干中收敛成统一字段 | Docker 路径可确认 stop / remove container / delete volume；process / remote 路径也有显式 delete / resume / pause 语义 | event、conversation metadata、agent-server URL、session API key、workspace backend 等共同构成结果回流上下文；不是单个 tool result JSON 就能覆盖 |
| `SWE-agent` | `post_startup_command_timeout` 可确认；DockerDeployment 启动 alive 轮询最长 180s；关闭容器时 `docker kill` 10s timeout、最多 3 次重试 | `docker kill` 路径明确存在重试；更细的任务级 retry / rollback 仍更多依赖外层 batch / rerun / replay 机制 | `deployment.stop()` 是明确 cleanup 入口，但不同 deployment 清理完整性差异很大，尤其 Fargate 不等于全资源清理 | trajectory、patch、logs、predictions、batch config 一起构成结果面；它明显比单次 tool output 更厚 |
| `Claude Code` | 官方文档可确认 `schedule` / `loop` 等长任务形态，现有主干还能确认“长任务建议后台执行”；但统一 timeout / budget contract 仍未显式公开 | hooks 文档已能确认 `PermissionDenied` 可返回 `{retry: true}`，`PreToolUse` 可 `allow / deny / ask / defer`，且存在 `PostToolUseFailure`、`PostToolBatch`、`StopFailure` 等失败与批次边界；这说明它有 tool-loop 级 retry / defer 钩子，但还不能外推成统一 cancel contract | 当前主干对 cleanup 仍偏薄；官方 hooks 文档只可确认 `WorktreeCreate` / `WorktreeRemove`、`SessionEnd` 等生命周期事件存在，但尚不足以推出统一 cleanup 规格 | Agent loop 文档可确认每次 tool execution 的结果会以 `UserMessage` 回流给 Claude，最终以 `ResultMessage` 收束，并带 final text、token usage、cost、session ID；说明其 result lifecycle 已明显超出单一 tool return |
| `Codex` | 官方配置文档可确认 MCP server 默认 startup timeout `10s`、per-tool timeout `60s`；Cloud 另有 Setup / Agent 两阶段，但固定执行持续时长仍不宜写成稳定规格 | approval policy 与 sandbox mode 明确影响执行继续方式；官方文档可确认 side-effecting app / MCP tool calls 需要 approval review，MCP 还支持 server-level / per-tool approval；CLI 文档还能确认 `/stop` 可停止当前 session 的 background terminals，但更细 retry / rollback 行为仍需更具体对象文档补证 | Cloud / Local cleanup 语义当前未在主干中收敛成统一字段；不过官方 changelog 与 slash commands 已能确认 resume、background terminals、`/stop`、state DB first for `resume --last` 等生命周期信号存在，说明它至少有 session / terminal 级停止与恢复面 | Local / Worktree / Cloud 的结果面与执行位置耦合很深；而 MCP config / schema 又补出 protocol-level result surface，CLI 还提供 `/ps` 查看后台终端状态，说明 Codex 同时承接环境级结果面、工具级结果面和运行中状态面 |
| `MCP` | 协议没有统一 timeout budget；progress 只提供长任务可观测信号 | cancellation 是协议级取消通知，不是强制中断 / retry / rollback 契约 | 协议不定义 cleanup；资源释放由 receiver / host / implementation 自行处理 | tool result 明确支持 `content`、`structuredContent`、`outputSchema`，是当前最清楚的 protocol-level result envelope 样本 |

这张表又把边界往前推了一点：

- `MCP` 在 **result envelope** 上最清楚，但在 `timeout / retry / cleanup` 上最弱。
- `SWE-agent` 与 `OpenHands` 在 **cleanup / lifecycle / workspace-result coupling** 上更厚，但它们的 result 面已经明显超出“单次工具调用返回值”。
- `Claude Code` 与 `Codex` 当前更像产品级执行治理样本：能看到权限、长任务、执行模式与扩展入口，但要抽成稳定执行器 contract 还需要更多对象内核验。

但当前仍明显缺少：

- 更细的对象级字段映射，例如 timeout、retry、cleanup、result envelope、side-effect policy
- Skill manifest 最小字段清单的正式主文
- Tool Executor 与 permission / policy / recovery 的更窄口径 stop-line

---

## 十、Tool Executor Stop-line（第一版）

在当前材料下，最稳妥的 stop-line 不是问“所有和 tool 相关的东西算不算 Tool Executor”，而是问：**一个系统若声称自己有 Tool Executor，它最少应该负责到哪里；再往外哪些职责就不该继续吞进去。**

### 10.1 一句工作定义

当前可以先把 Tool Executor 收窄为：

> **把一次工具调用意图，变成受约束、可观测、可回流的真实执行。**

这里故意只保留四个关键词：

- **调用意图**：上层已经决定要调用某类 tool
- **受约束**：执行路径、权限、工作域与副作用边界不是空白
- **可观测**：执行中至少要暴露结果、失败、进度或停止信号
- **可回流**：结果必须能重新进入 agent loop、orchestrator 或 host system

这一定义刻意不把 tool discovery、task planning、完整 runtime persistence、组织级治理一起吞进来。

### 10.2 Tool Executor 至少要负责的五个边界面

#### 10.2.1 Execution Path

Tool Executor 至少要回答：**这次调用落到哪类执行载体上。**

当前已稳定出现的载体包括：

- 同进程函数调用
- subprocess / shell session
- 本地受限执行
- worktree / workspace 绑定执行
- 远程 container / sandbox

如果一个系统只会“发起 tool call”，却不负责把调用绑定到具体执行面上，那它更像 protocol client 或 planner，而不是完整 Tool Executor。

#### 10.2.2 Invocation Lifecycle

Tool Executor 至少要承接：**一次调用从开始到停止的最小生命周期。**

最低限度应包括：

- execute / start
- success / failure
- timeout 或长任务边界
- cancel / defer / permission-denied 这类中断信号
- cleanup entry
- result handoff

但 stop-line 要写窄：它负责的是**单次调用或单批调用的生命周期边界**，不是完整 session recovery、conversation persistence 或 thread resume。

#### 10.2.3 Result Envelope

Tool Executor 至少要把结果回收成系统可消费的统一面，而不只是“命令跑完了”。

最小结果面通常包括：

- text / content
- structured result
- failure type / exit signal
- progress 或 completion signal
- artifact pointer 或 workspace side effect reference

如果一个系统只有 tool schema，却没有结果如何回流的稳定面，那它仍更接近 protocol surface，而不是执行器边界。

#### 10.2.4 Side-effect Control Surface

Tool Executor 不一定自己做最终审批，但至少要暴露足够的副作用面，供上层治理系统判断。

最少需要让上层看见：

- 是否写文件
- 是否运行命令
- 是否访问网络
- 是否修改依赖或系统状态
- 是否需要 approval / escalation

也就是说，它至少要提供**可治理的执行表面**，而不是把所有副作用都藏在黑箱里。

#### 10.2.5 Workspace / Environment Binding

Tool Executor 至少要说明：**执行与哪块工作域绑定。**

最小问题包括：

- 这次执行针对哪个 workspace / repo / root
- 是否在 worktree 中执行
- 是否使用独立环境、远程环境或 sandbox
- 输出产物写到哪里

但 stop-line 同样不能越界：完整 workspace snapshot、filesystem rollback、checkpoint store、time-travel recovery 仍属于更下层的 environment / recovery system。

### 10.3 四类不该继续吞进去的职责

#### 10.3.1 不要吞掉 Tool Orchestration

以下问题不应继续归给 Tool Executor：

- 为什么此时调用这个 tool
- 多步工具链如何排序
- 失败后是重规划、换工具还是退出
- 多工具依赖图如何组织

这些问题更接近 `tool orchestration` 或更高层 planning loop。

#### 10.3.2 不要吞掉 Code Execution Environment

以下问题也不应被 Tool Executor 一把吞掉：

- sandbox / container 本身如何实现
- runtime persistence 如何落盘
- workspace snapshot / rollback / checkpoint store 如何设计
- 容器、远程 worker 或本地 session 的完整生命周期治理

Tool Executor 需要依赖这些环境能力，但不等于环境层本身。

#### 10.3.3 不要吞掉 Skill / Tool System

以下问题属于 capability surface 或制品治理，不应混成执行器本体：

- tool registry
- skill manifest
- discovery / listing / packaging
- plugin / marketplace / connector distribution

Tool Executor 消费这些声明和入口，但不替代它们。

#### 10.3.4 不要吞掉最终 Policy Authority

以下问题也不应直接写成 Tool Executor 的内生职责：

- 组织级审批策略
- 风险等级体系
- 审计与合规策略
- 全局权限模型

更稳的写法是：Tool Executor 负责暴露可治理的执行面，真正的 policy authority 由 host、permission system 或 org-level governance 承接。

### 10.4 用现有对象反校验这条 stop-line

当前几类对象正好能帮我们校验 stop-line 是否写得过宽或过窄：

- `MCP` 说明 protocol surface 不等于 Tool Executor：它标准化了 discovery、invocation、schema、structured result，但没有定义统一 cleanup、rollback 或 workspace recovery。
- `Claude Code` 说明 tool-loop hooks、result lifecycle、permission-denied retry 信号已经进入执行器邻近面；但完整 session-end cleanup 仍不能直接写成执行器统一 contract。
- `Codex` 说明 sandbox、approval、background terminals、resume 都与执行器高度相关；但它们仍不能被混写成同一层职责。
- `OpenHands` 与 `SWE-agent` 说明一旦 workspace、repo、trajectory、sandbox lifecycle 深度进入执行链，Tool Executor 会变厚；但它依然不等于完整 runtime 或 recovery system。

### 10.5 当前最值得继续收紧的 stop-line 问题

在这条边界下，接下来最值得继续补证的不是“Tool Executor 是不是重要”，而是更窄的三个问题：

1. `cancel / defer / retry` 到底停在哪一层，哪些属于 tool-loop，哪些属于 session/runtime
2. `result envelope` 至少要统一到什么程度，artifact / side effect / structured result 是否应进入同一结果面
3. `cleanup entry` 与真正的 rollback / compensation action 应如何分层

这三个问题如果继续收紧，Tool Executor 就能从“边界专题”推进成“可比较的执行 contract 专题”。

---

## 十一、下一轮最值得补证的对象与问题

优先补证对象：

1. `OpenHands / SWE-agent`：补代码执行、workspace、artifact、恢复路径
2. `Claude Code / Codex`：补权限确认、工具执行边界、交互式治理
3. `MCP / skill-based systems`：补 discovery、schema、manifest、protocol surface

优先补证问题：

- Tool Executor 是否需要独立 manifest，还是消费 tool / skill manifest 即可
- 同进程、subprocess、venv、container 四类执行路径的最小适用边界
- trace / result / artifact / side effects 是否应统一进入一个结果 envelope
- Tool Executor 与 runtime gateway / adapter / orchestrator 的 stop-line 如何保持清楚

## Evidence

- Status: `Observed / Inferred`
- Sources:
  - `../../../backlog.md`
  - 当前仓库中围绕 Ravel Agent 的关键问题整理与后续对象级补证
  - `../../../../02-single-agent/tool-use/README.md`
  - `../../../../02-single-agent/tool-use/tool-orchestration.md`
  - `../../../../05-environments/code-execution-environments/README.md`
  - `../../../../05-environments/code-execution-environments/workspace-checkpoint.md`
  - `../04-skill-and-tool-systems/README.md`
  - `./orchestration-implementations.md`
  - `../03-project-studies/openhands/runtime-and-sandbox.md`
  - `../03-project-studies/swe-agent/environment-and-execution.md`
  - `../02-coding-agents-and-tools/claude-code/architecture-deep-dive.md`
  - `../02-coding-agents-and-tools/claude-code/mcp-custom-tools.md`
  - `../02-coding-agents-and-tools/codex/codex-cloud-sandbox.md`
  - `../02-coding-agents-and-tools/codex/codex-agent-mechanisms.md`
  - `../04-skill-and-tool-systems/mcp/overview.md`
  - 基准来源：`https://code.claude.com/docs/en/`；后续引用：`overview`、`hooks-guide`、`agent-sdk/agent-loop`
  - 基准来源：`https://developers.openai.com/codex/`；后续引用：`agent-approvals-security`、`concepts/sandboxing`、`app/worktrees`、`changelog`、`cli/slash-commands`
  - `https://developers.openai.com/codex/llms-full.txt`
- Trace: 从 `agentic/backlog.md` 中 `Tool Executor 与 Skill 制品边界` 这一跨目录缺口出发，结合当前仓库中围绕 Ravel Agent 的关键问题整理、单智能体工具使用、代码执行环境、Skill / Tool 系统与 orchestration contract 现有主干，先形成第一版边界收口；随后回填 OpenHands、SWE-agent、Claude Code、Codex、MCP 的执行入口与执行载体，并压到 `timeout / retry / cleanup / result envelope` 这一组更细字段。本轮继续补入 Claude Code 与 Codex 的官方一手文档，重点收紧 tool lifecycle hooks、retry/defer 信号、final result lifecycle、background terminals、resume 与 `/stop` 这类运行中断/停止面，因此部分原本偏薄的 cancel / result lifecycle / session-stop 表述现在有了更明确的一手支撑；但 cleanup / rollback 的完整实现语义仍未在公开材料中收敛，因此继续保持 `Observed / Inferred` 口径。
- Needs:
  - 补 Claude Code 的更细 cleanup / session-end / worktree removal 行为证据
  - 补 Codex Cloud / Local cleanup 与 rollback 的更细对象级证据
  - 补 Skill manifest 最小字段专题
  - 补 Tool Executor 与 policy / permission / recovery 的窄口径边界

*最后更新: 2026-06-18*
