# OpenHands Official Docs Study

> 阶段：官方资料阶段结论归档完成；后续源码核验结论已转入 `runtime-and-sandbox.md`
> Evidence 状态：以 `Verified` / `Observed` / `Inferred` / `Unverified` 为主；只有官方直接明说的 Claim 才标 `Verified`，并在来源中明确其依据来自官方文档
> 来源处理说明：本文件已把可保留结论回收到 OpenHands 官方文档与公开仓库对象，不再把临时整理稿作为正式来源终点

## 一、定位

本文件只整理 OpenHands 官方资料阶段可直接保留的证据地图，并作为 `runtime-and-sandbox.md` 之前的上游资料归档。

使用原则：

- 官方文档、官方 GitHub README / docs、release notes、官方博客或维护者正式说明优先。
- DeepWiki、第三方博客、部署平台文档、二手教程只能作为旁证或源码线索，不能作为核心官方证据。
- 如果官方原文没有直接说明，不能标 `Verified`。
- 如果结论需要拼接多个官方段落才能得到，统一降级为 `Inferred`，并在说明中明确其依据来自官方文档拼接。
- 如果当前官方资料没有回答，统一标为 `Unverified`，并说明仍需进入源码阶段核验。

## 二、本轮实际复核的官方 URL

本轮优先复核了以下官方资料：

1. `https://docs.openhands.dev/sdk/arch/workspace`
2. `https://docs.openhands.dev/sdk/arch/design`
3. `https://docs.openhands.dev/sdk/guides/agent-server/docker-sandbox`
4. `https://docs.openhands.dev/sdk/guides/convo-persistence`
5. `https://docs.openhands.dev/openhands/usage/architecture/runtime`
6. `https://docs.openhands.dev/sdk/guides/agent-server/overview`
7. `https://docs.openhands.dev/openhands/usage/advanced/configuration-options`

这些页面当前可直接支撑的内容主要包括：

- V1 的设计原则与层次划分。
- V0 runtime 的 Docker sandbox / ActionExecutionServer 口径。
- Workspace 的抽象接口、Local / Remote 区分以及与 Conversation 的关系。
- DockerWorkspace 的 context manager 生命周期示例。
- Conversation persistence 的保存 / 恢复方式与 `base_state.json`、`events/` 结构。
- `SANDBOX_VOLUMES`、`OH_PERSISTENCE_DIR`、`RUNTIME` 等配置项在当前文档中的口径。

## 三、官方资料索引与当前用途

| 资料 | URL | 当前可回答问题 | 当前处理 |
|---|---|---|---|
| Runtime Architecture | https://docs.openhands.dev/openhands/usage/architecture/runtime | V0 runtime、Docker sandbox、ActionExecutionServer | 只作为 V0 legacy runtime 口径，不外推到 V1 |
| Workspace | https://docs.openhands.dev/sdk/arch/workspace | Workspace 抽象、Local / Remote 区别、与 Conversation / agent-server 的关系 | 可用于 workspace 官方定义 |
| Design Principles | https://docs.openhands.dev/sdk/arch/design | V1 设计原则、SDK / tools / workspace / agent server 分层、stateless by default | 可作为 V1 架构入口 |
| Docker Sandbox | https://docs.openhands.dev/sdk/guides/agent-server/docker-sandbox | DockerWorkspace、自动创建 / 清理容器、RemoteConversation | 可用于官方文档阶段的 docker sandbox 关系 |
| Persistence | https://docs.openhands.dev/sdk/guides/convo-persistence | Conversation 持久化、恢复、`base_state.json` / `events/` | 可用于 conversation persistence 口径 |
| Agent Server Overview | https://docs.openhands.dev/sdk/guides/agent-server/overview | Remote agent server、workspace 切换、隔离执行 | 可用于 remote workspace / conversation 关系 |
| Configuration Options | https://docs.openhands.dev/openhands/usage/advanced/configuration-options | `SANDBOX_VOLUMES`、`OH_PERSISTENCE_DIR`、legacy `RUNTIME` | 需区分 V1 页面与 legacy V0 兼容项 |

## 四、保留、降级与删除后的关键结论

### 4.1 可保留的结论

#### OpenHands V1 的设计原则

- V1 明确强调分层：SDK（agent core）、tools、workspace（sandbox）、agent server。
- V1 明确强调：sandboxing should be opt-in, not universal。
- V1 明确强调：stateless by default，唯一可变状态是 conversation state。

Evidence 状态：`Verified`

来源：

- `sdk/arch/design`

#### Workspace 的官方最小定义

- `BaseWorkspace` 是执行与文件操作的抽象接口。
- `LocalWorkspace` 是本地 subprocess 执行。
- `RemoteWorkspace` 是通过 HTTP 调用 agent-server 的远程执行。
- 官方直接给出了 Local vs Remote 的对比：`Process-level` vs `Container/VM-level` isolation。
- Conversation factory 会根据 workspace 类型选择 LocalConversation 或 RemoteConversation。

Evidence 状态：`Verified`

来源：

- `sdk/arch/workspace`
- `sdk/guides/agent-server/overview`

#### DockerWorkspace 的最小生命周期行为

- 官方示例明确将 `DockerWorkspace` 作为 context manager 使用。
- 示例直接说明：进入 `with DockerWorkspace(...)` 时容器处于运行状态；退出后容器会被自动停止和清理。
- 用 `DockerWorkspace` 创建 `Conversation(...)` 时，示例断言其为 `RemoteConversation`。

Evidence 状态：`Verified`

来源：

- `sdk/guides/agent-server/docker-sandbox`

#### Remote Agent Server 的官方口径

- Remote Agent Server 是 HTTP/WebSocket 服务。
- 它负责运行 agents、管理 workspaces、流式传输 events，并处理命令与文件操作。
- “switching is just changing the workspace argument; your Conversation code stays the same” 是官方直接表述。
- 官方将 workspace 描述为 isolated environment（local、Docker、或 remote VM）where the agent code runs。

Evidence 状态：`Verified`

来源：

- `sdk/guides/agent-server/overview`

#### Conversation persistence 的最小官方口径

- 官方明确支持 `persistence_dir` + `conversation_id` 的保存与恢复。
- 当前页面明确给出持久化目录包含 `base_state.json` 和 `events/`。
- 当前页面把 `base_state.json` 描述为保存 agent configuration、execution status、statistics、secrets、agent_state。
- 当前页面把 `events/event-*.json` 描述为保存 message history、tool calls、observations、all conversation events。
- 但这些表述在官方资料阶段只能直接支撑“conversation-level persistence / restore”存在，不能单独外推出“完整 workspace 文件系统状态可被独立恢复”；这一点需要结合源码阶段另行核验。

Evidence 状态：`Verified`

来源：

- `sdk/guides/convo-persistence`

#### V1 配置页面中可直接保留的配置口径

- `OH_PERSISTENCE_DIR`：where OpenHands stores local state。
- `SANDBOX_VOLUMES`：mount host directories into the sandbox。
- 一些部署仍使用 legacy `RUNTIME` 环境变量选择 sandbox provider：`docker` / `process` / `remote`。

Evidence 状态：`Verified`

来源：

- `openhands/usage/advanced/configuration-options`

### 4.2 需要降级的结论

#### OpenHands 项目定位

可以保留为：OpenHands 是开源 coding agent / software agent platform，官方资料强调 agent、workspace / sandbox、agent server 与远程执行能力。

但不应再写成过强的“官方唯一定位定义”。

Evidence 状态：`Observed`

原因：多页文档共同形成这个认识，但当前复核页中缺少单一页面对整个平台定位的统一、强定义式陈述。

#### V0 → V1 迁移关系

可以保留为：V1 是 OpenHands V1 effort / architectural rework，文档明确在重构 V0 的若干问题（mandatory sandboxing、mutable state、tight coupling）。

不应保留为：

- “V0 将于 2026 年 4 月移除”
- “已经确认完整迁移时间线”

Evidence 状态：`Observed / Inferred`

原因：设计页明确说这是 V1 effort，但本轮复核的官方页面没有直接给出你当前文档里那种具体迁移时间线承诺。

#### Runtime / Sandbox / Workspace 三者的精确映射

可以保留为：

- V0 页面使用 runtime 口径。
- V1 页面更突出 workspace（sandbox）与 agent server。
- V1 配置页说明有些部署仍使用 legacy `RUNTIME` 环境变量来选择 sandbox provider。

不应保留为：

- “V1 明确以 SandboxService 替代 Runtime”
- “workspace 就是 sandbox 的代码层等价物”
- “runtime 与 sandbox 在 V1 中已经一一替换完毕”

Evidence 状态：`Inferred`

原因：官方页面支持“术语重心迁移”和“legacy runtime knob 仍存在”，但没有直接给出完整替代关系图。

### 4.3 必须删除或留到源码阶段的问题

以下结论当前不能写成官方文档阶段定论：

- V1 是否明确以 `SandboxService` 替代 Runtime。
- workspace 是否明确保存完整 file system state。
- conversation persistence 恢复的状态范围是否包含真实工作区文件内容。
- event replay 是否等同于 trajectory replay。
- pause / resume、checkpoint、rollback、snapshot 是否存在。
- workspace、sandbox、conversation / session 之间的完整生命周期绑定关系。

这些内容统一降为：`Unverified`，并标明仍需进入源码阶段核验。

补充说明：即使后续源码已经证明 conversation-level restore、workspace backend pause/resume、agent-server conversation persistence 等能力存在，官方资料阶段本身仍不能被回写成“完整 workspace 文件系统可被独立 checkpoint / replay / restore”的强结论；这一边界必须保持。

## 五、DeepWiki / 第三方资料的降级处理

以下材料已明确降级：

- DeepWiki 页面：只作为源码聚合旁证 / 第二阶段线索，不再作为核心官方证据。
- 第三方博客、部署平台文档、社区教程：只作旁证或待核验线索。
- 早期临时整理中曾把 DeepWiki 近似写成官方 / maintainer comment；当前已明确降级，不再直接沿用到主干项目研究文档。

## 六、当前可用的谨慎结论

### 6.1 Runtime / Sandbox / Workspace

当前可以保留的工作性理解：

- V0 runtime architecture 直接描述的是 Docker runtime + ActionExecutionServer 的执行模型。
- V1 design / workspace / agent-server 文档描述的是 SDK、workspace（sandbox）、agent server 的分层与远程执行路径。
- Workspace 至少是执行与文件操作的抽象接口；Local 与 Remote 的隔离级别由官方页面直接给出。
- DockerWorkspace 是通过 agent-server 运行的 docker-based remote workspace，并带有自动容器生命周期管理示例。

Evidence 状态：`Observed / Verified`

### 6.2 Persistence / Recovery

当前只能保留：

- OpenHands 官方页面直接支持 conversation persistence。
- 当前页面直接说明持久化包含 `base_state.json` 与 `events/`，并支持同一 `conversation_id` 的恢复。
- 这些官方表述更稳妥地应理解为 conversation-level persistence / restore，而不是完整 workspace checkpoint 机制；单看官方资料阶段，仍不能把它外推成“恢复连接 = 恢复完整任务工作域”或“文件系统状态可脱离原 runtime 独立重建”。
- 当前页面没有直接把 pause / resume、checkpoint、rollback、snapshot 列为正式功能。
- 当前页面也没有直接给出“保存完整 workspace 文件系统状态”的明确原文。

Evidence 状态：`Verified / Unverified`

### 6.3 Workspace Lifecycle / Sharing

当前官方文档阶段只能保留：

- Conversation 与 workspace 存在直接关系。
- DockerWorkspace / LocalWorkspace / RemoteWorkspace 的生命周期行为可能不同。
- `SANDBOX_VOLUMES` 提供宿主目录挂载到 sandbox 的官方配置入口。
- 官方架构页与 API Reference 已把 `BaseWorkspace` / `LocalWorkspace` / `RemoteWorkspace` / `Workspace` 作为 SDK 层对象公开；官方架构页还把 `DockerWorkspace`、`RemoteAPIWorkspace` 标到 `openhands-workspace` 包路径。结合候选 SDK 仓库中 `openhands-sdk/openhands/sdk/workspace/__init__.py`、`workspace.py`、`remote/base.py`、`remote/async_remote_workspace.py` 与 `openhands-workspace/openhands/workspace/__init__.py`、`docker/workspace.py`、`remote_api/workspace.py` 的公开源码，可进一步确认这些对象的包级分布、工厂分派关系，以及基础 `RemoteWorkspace` / `AsyncRemoteWorkspace` 更偏向 agent-server client、而 Docker / Runtime API 工作区作为 `RemoteWorkspace` 子类承担远端执行载体生命周期接入；但完整生命周期与绑定关系仍需继续读源码。

当前不能保留为定论：

- workspace 是否随 session / sandbox 自动创建或销毁。
- 是否支持跨 session / cross-task / cross-agent workspace 共享。
- 是否存在 overlay、copy-on-write、git worktree 级共享模型。

Evidence 状态：`Observed / Unverified`

## 七、源码阶段待核验问题

源码阶段应优先核验：

1. V1 中 workspace、sandbox、runtime、conversation / session 的核心类、接口与依赖关系。
2. `DockerWorkspace` / `RemoteWorkspace` / `LocalWorkspace` 的创建、销毁、复用与清理逻辑；当前已定位到候选 SDK 仓库 `https://github.com/OpenHands/software-agent-sdk`，后续应优先在该仓库中核验 `openhands-sdk` 与 `openhands-workspace` 包。官方架构页目前把 `DockerWorkspace` 标到 `openhands-workspace/openhands/workspace/docker`，把 `RemoteAPIWorkspace` 标到 `openhands-workspace/openhands/workspace/remote_api`。
3. `SANDBOX_VOLUMES`、工作目录、容器挂载与宿主路径映射的真实实现。
4. `persistence_dir` / `OH_PERSISTENCE_DIR` 保存的状态范围，尤其是否包含真实文件系统内容，还是仅 conversation state / events；当前交叉源码更支持后者至少是核心可确认部分，而不是直接支持“完整文件系统独立恢复”。
5. conversation 恢复时 sandbox / workspace 是否重建、复用或重新挂载；当前已在上游仓库 `https://github.com/OpenHands/OpenHands` 的前端与 `app_server` V1 路由中补到一部分 reopen 入口证据：既有会话页会先走 `GET /api/v1/app-conversations?ids=...` 读取持久化 metadata，再用 `conversation_url` / `session_api_key` 直连既有 agent-server，若 `sandbox_status === "PAUSED"` 则页面初次打开或标签重新可见时实际先走 `POST /api/v1/sandboxes/{sandbox_id}/resume`。进一步看底层实现，Docker 路径是复用同一容器、`working_dir` 与 volume mounts，Remote runtime 路径是复用同一 `sandbox_id` / `session_id` 并按既有 `runtime_id` 调 `/resume`；当前仓库内尚未看到“单靠事件回放重建完整文件系统状态”的直接证据，因此更稳妥的口径应是“先恢复 conversation / runtime identity，再尽量沿用既有 runtime / volume / working_dir 连续性”。
6. event / trajectory 是否能够支撑 action → observation → artifact 的可追溯链。
7. 是否存在 pause / resume、checkpoint、rollback、snapshot、overlay / copy-on-write 机制。
8. V0 runtime 与 V1 sandbox / workspace 的迁移关系到底是术语迁移还是架构重组。

## 八、官方资料阶段对 `runtime-and-sandbox.md` 的历史建议

本节记录的是本文件形成时的阶段性判断：当时官方资料仍不足以支撑稳定的 runtime / sandbox / workspace 生命周期结论。

因此当时建议：

- `runtime-and-sandbox.md` 不应在官方资料阶段扩写成稳定分析文档。
- 该文件更适合先作为源码阶段占位，并只保留更清晰的待核验问题。

补充说明：上述建议对应的是官方资料阶段判断；当前 `runtime-and-sandbox.md` 已基于 `https://github.com/OpenHands/OpenHands` 中 `openhands/app_server/` 及关联已核验范围的源码核验升级为专题结果文档。

## 九、官方资料阶段对环境主线的最小回填建议

本节记录的是本文件形成时的阶段性回填建议：

- 在 `agentic/05-environments/candidates.md` 中把 OpenHands 状态更新为“官方原文复核完成，待源码核验”。
- 暂不大改 `workspace-lifecycle.md`；等 `openhands/app_server/` 之外的后续源码范围继续核验后，再把具体 Claim 回填到 `## Evidence`。
- 如果后续源码阶段发现 workspace 与 sandbox / runtime 的关系和当前主干分类冲突，再更新 `agentic/05-environments/conflict.md`。

补充说明：其中第一条建议已完成；`candidates.md` 现已更新为“`openhands/app_server/` 层源码核验完成，SDK 层 workspace 待补充”。
