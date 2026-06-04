# Environments Candidates

> 适用范围：`05-environments/` 中值得持续研究的论文、仓库、官方文档、产品、协议和 benchmark
> 使用原则：本文件记录候选研究对象，不直接承接正文结论；对象研究完成后，再回填正文、backlog 或 conflict

## 一、收录规则

适合进入本文件的对象通常满足以下条件之一：

- 能直接支撑 `05-environments/` 中仍待补证的关键专题，例如 workspace、sandbox、checkpoint、traceability 或 recovery
- 代表主流产品、开源实现或官方机制，适合为 `Observed` / `Verified` 级结论提供候选证据
- 能帮助校正当前主干中的概念边界，避免把单一路径误写成通用定论
- 已经形成稳定对象队列，不适合继续塞进 `backlog.md` 的问题缺口条目中

不适合进入本文件的内容：

- 纯粹的问题缺口、待写主题或抽象疑问（应进入 `backlog.md`）
- 已经足够稳定、应直接回填正文的定论
- 只有单一博客或社区经验支撑、且尚未形成稳定研究对象的线索
- 与 `05-environments/` 证据缺口无直接关系的泛产品清单

---

## 二、P0：Workspace Lifecycle / Sharing Model 补证对象

### 2.1 OpenHands

- **对象类型**：开源 coding agent / runtime 系统
- **关联问题**：runtime / sandbox / workspace 的绑定关系；per-task runtime 与 workspace 状态边界；多轮执行中的工作域延续方式
- **为什么值得研究**：它直接涉及 agent 执行环境、runtime client、sandbox service、workspace 组织与 action execution，适合补 `workspace-lifecycle.md` 中“workspace 是否必须绑定 runtime / sandbox”的证据
- **当前状态**：`openhands/app_server/` 层源码核验完成；SDK 层已确认 `Workspace` 工厂分派、基础 `RemoteWorkspace` / `AsyncRemoteWorkspace` 的 client / transport 角色，以及 `DockerWorkspace` / `APIRemoteWorkspace` 对远端执行载体生命周期的接入方式；app_server 启动链路也已确认会把 `SandboxInfo` 转成 `AsyncRemoteWorkspace` 后继续 repo/setup/skills/conversation 启动，并具备“显式 `sandbox_id` 直连 / `PAUSED` 时 resume / 无 `sandbox_id` 时优先复用运行中 sandbox / 否则新建”的分支；对子会话还会在未显式指定 `sandbox_id` 时继承父会话环境；已持久化 conversation 的读取 / send-message 路径也会直接按持久化 `sandbox_id` 查找 sandbox；前端 reopen 入口现已补到一层：既有会话页会先走 `GET /api/v1/app-conversations?ids=...` 读取 metadata，再把 `conversation_url` / `session_api_key` 交给 WebSocket provider 直接重连既有 agent-server，若 `sandbox_status === "PAUSED"` 则页面初次打开或标签重新可见时实际先走 `POST /api/v1/sandboxes/{sandbox_id}/resume`；当前还能更具体地确认两类连续性线索：Docker 路径依赖同一容器 + `working_dir` + volume mounts，Remote runtime 路径依赖同一 `sandbox_id` / `session_id` 与既有 `runtime_id` 的 `/resume`；当前仓库内尚未看到“通过事件回放重建完整工作区文件系统”的直接证据
- **Evidence need / 证据优先级**：GitHub 源码、runtime / workspace / sandbox 相关实现、release notes；当前已完成 `https://github.com/OpenHands/OpenHands` 中 `openhands/app_server/` 及关联已核验范围的源码核验；SDK 层（`openhands.sdk.workspace`）仍缺跨 `app_server`、agent-server 与 runtime/container 的端到端 workspace ↔ sandbox 绑定关系证据，当前已定位到候选 SDK 仓库 `https://github.com/OpenHands/software-agent-sdk`
- **当前产出**：`../06-frameworks-and-tools/03-project-studies/openhands/official-docs.md`、`../06-frameworks-and-tools/03-project-studies/openhands/runtime-and-sandbox.md`（已从占位升级为 `openhands/app_server/` 相关源码核验结果）
- **预期产出**：继续增强 SDK 层 workspace ↔ sandbox 绑定关系的 Evidence；`runtime-and-sandbox.md` 已补充 V1 SandboxService 体系、三种沙箱实现、事件持久化路径等当前核验范围内的源码级结论，并进一步确认 `https://github.com/OpenHands/software-agent-sdk` 中 `RemoteWorkspace` / `AsyncRemoteWorkspace` 主要承担 agent-server client / transport 职责，而 `DockerWorkspace` / `APIRemoteWorkspace` 负责容器或 Runtime API 驱动的远端执行载体接入；后续仍需继续核验跨层绑定与生命周期语义
- **Source / Decision / Placement / Gap**：来源于 `agentic/temp/web-search/4.md`、`agentic/temp/web-search/6.md` 与 `workspace-lifecycle.md` 当前 Needs；官方资料阶段与 `openhands/app_server/` 层源码核验结果已集中沉淀到 `agentic/06-frameworks-and-tools/03-project-studies/openhands/`，本文件只保留候选对象状态和环境层补证指针；仍缺跨 SDK 与其他未核验范围的源码级证据，对 runtime、workspace、sandbox、persistence 与 replay 边界做交叉核验

### 2.2 SWE-agent

- **对象类型**：开源软件工程 agent / benchmark-oriented coding agent
- **关联问题**：coding agent 执行环境如何组织 workspace、Docker sandbox、任务尝试与测试循环；代码任务中的工作域隔离边界
- **为什么值得研究**：它能提供代码任务环境组织、测试执行循环与 workspace 隔离的真实工程案例，适合作为 per-task isolated workspace / execution environment 的对照样本
- **当前状态**：候选 / 待研究
- **Evidence need / 证据优先级**：官方文档、GitHub 源码、环境抽象相关实现
- **预期产出**：补充 `workspace-lifecycle.md` 对 per-task isolated workspace / execution environment 边界的案例证据；必要时补 `rollback-recovery-design-paths.md` 对尝试循环与恢复路径的观察
- **Source / Decision / Placement / Gap**：来源于 `agentic/temp/chat/instruction.md` 的推荐对象与 `workspace-lifecycle.md` 当前 Needs；作为候选研究对象登记到本文件；服务代码任务环境隔离的补证；仍缺对环境抽象、尝试边界与测试循环的源码级核验

### 2.3 LangGraph checkpoint

- **对象类型**：Agent workflow runtime / checkpoint 机制
- **关联问题**：checkpoint 与 runtime / process 解耦；graph state checkpoint 是否覆盖 workspace 文件系统状态；恢复点与任务状态指针的边界
- **为什么值得研究**：它能帮助校正 checkpoint、workspace 文件状态和 runtime 生命周期之间的边界，避免把 checkpoint 误写成文件系统 snapshot
- **当前状态**：候选 / 待研究
- **Evidence need / 证据优先级**：官方 persistence / checkpoint 文档、源码接口、版本说明
- **预期产出**：增强 `code-execution-environments/workspace-lifecycle.md` 与 `code-execution-environments/workspace-checkpoint.md` 的 Evidence，特别是“checkpoint 不等于完整文件系统状态”的边界说明
- **Source / Decision / Placement / Gap**：来源于 `workspace-checkpoint.md`、`workspace-lifecycle.md` 当前 Needs 与委托指令中的推荐对象；作为 checkpoint 边界核验对象登记到本文件；服务 checkpoint 与 workspace 状态边界的补证；仍缺官方接口与版本行为对照

### 2.4 Claude Code / Git worktree parallel agents

- **对象类型**：产品实践 / coding agent 工作流
- **关联问题**：git worktree / branch isolation 是否适合作为多 agent 并行 workspace model；代码任务语义隔离与执行环境隔离是否能部分分离
- **为什么值得研究**：它直接对应 `workspace-lifecycle.md` 中 git worktree / branch isolation 的待补证方向，也能帮助判断 worktree 更像独立 sharing model，还是代码任务中的一种实现路径
- **当前状态**：候选 / 待研究
- **Evidence need / 证据优先级**：官方文档、官方技术博客、发布说明；社区博客只能作为 `Observed` 或待核验线索
- **预期产出**：为 `workspace-lifecycle.md` 中 git worktree / branch isolation 提供产品实践证据；若官方证据不足，则维持为 `Observed` 级线索，不回写成强定论
- **Source / Decision / Placement / Gap**：来源于 `workspace-lifecycle.md` 当前分类与委托指令中的推荐对象；作为多 agent 并行工作域实践对象登记到本文件；服务 git worktree 是否应保留为独立共享模型的判断；仍缺官方材料对并行代理、工作域隔离与合并边界的直接说明

### 2.5 AgentFS / astrid-vfs / overlay filesystem cases

- **对象类型**：开源工具 / filesystem abstraction / overlay 案例
- **关联问题**：shared baseline + isolated overlay、copy-on-write、overlay revert 的适用边界；workspace sharing model 与 checkpoint / snapshot 的区分
- **为什么值得研究**：它们对应 `workspace-lifecycle.md` 中 shared baseline + isolated overlay 的关键实现路径，也能为 `rollback-recovery-design-paths.md` 提供 overlay revert 的更细证据
- **当前状态**：候选 / 待研究
- **Evidence need / 证据优先级**：源码、官方 README、设计文档；博客材料需降级处理
- **预期产出**：用于区分 overlay revert、workspace sharing model 与完整 checkpoint / snapshot 的边界，并增强 `workspace-lifecycle.md`、`workspace-checkpoint.md`、`rollback-recovery-design-paths.md` 的 Evidence
- **Source / Decision / Placement / Gap**：来源于 `workspace-lifecycle.md`、`workspace-checkpoint.md`、`rollback-recovery-design-paths.md` 当前 Needs 与委托指令中的推荐对象；作为 overlay 路径补证对象登记到本文件；服务 shared baseline + isolated overlay 的实现级补证；仍缺不同实现对并发语义、非文件状态与回退边界的正式说明

---

## 三、对象与证据缺口映射

- `OpenHands`：主要服务 workspace 与 runtime / sandbox 的生命周期绑定缺口
- `SWE-agent`：主要服务代码任务中的 per-task isolation、测试循环与环境隔离案例缺口
- `LangGraph checkpoint`：主要服务 checkpoint、process state 与 workspace 文件状态边界缺口
- `Claude Code / Git worktree parallel agents`：主要服务 git worktree / branch isolation 是否应视为独立 workspace sharing model 的缺口
- `AgentFS / astrid-vfs / overlay filesystem cases`：主要服务 shared baseline + isolated overlay、copy-on-write、overlay revert 与完整 checkpoint 区分的缺口

---

## 四、与现有主干的关系

- `backlog.md` 继续记录 `Workspace Lifecycle Binding / Sharing Model` 这一问题缺口；本文件只登记为该缺口服务的候选研究对象
- `workspace-lifecycle.md` 继续保留当前 `Inferred / Observed` 的工作性分类；对象核验完成后，再按 Evidence 状态、来源与 Trace 补充其 `## Evidence`
- `conflict.md` 当前已把 “workspace 是否等于 sandbox 内路径” 收窄为 lifecycle 绑定与 sharing model 问题；本文件不新增冲突，只为后续核验提供候选对象队列
- 当前不创建 `evidence.md` 或 `evidence-registry.md`，因为证据治理尚未复杂到需要主题级 registry
