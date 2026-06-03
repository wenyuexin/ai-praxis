# OpenHands Official Docs Study

> 阶段：官方资料原文复核完成，待进入源码阶段
> Evidence 状态：以 `Observed / Inferred / Unclear` 为主；只有官方直接明说的 Claim 才标 `Verified by official docs`
> 上游材料：`agentic/temp/web-search/6.md`

## 一、定位

本文件只整理 OpenHands 官方资料阶段可直接保留的证据地图，用于支撑后续源码研究。

使用原则：

- 官方文档、官方 GitHub README / docs、release notes、官方博客或维护者正式说明优先。
- DeepWiki、第三方博客、部署平台文档、二手教程只能作为旁证或源码线索，不能作为核心官方证据。
- 如果官方原文没有直接说明，不能标 `Verified by official docs`。
- 如果结论需要拼接多个官方段落才能得到，统一降级为 `Inferred from official docs`。
- 如果当前官方资料没有回答，统一标为 `Unclear / requires source code`。

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

Evidence 状态：`Verified by official docs`

来源：

- `sdk/arch/design`

#### Workspace 的官方最小定义

- `BaseWorkspace` 是执行与文件操作的抽象接口。
- `LocalWorkspace` 是本地 subprocess 执行。
- `RemoteWorkspace` 是通过 HTTP 调用 agent-server 的远程执行。
- 官方直接给出了 Local vs Remote 的对比：`Process-level` vs `Container/VM-level` isolation。
- Conversation factory 会根据 workspace 类型选择 LocalConversation 或 RemoteConversation。

Evidence 状态：`Verified by official docs`

来源：

- `sdk/arch/workspace`
- `sdk/guides/agent-server/overview`

#### DockerWorkspace 的最小生命周期行为

- 官方示例明确将 `DockerWorkspace` 作为 context manager 使用。
- 示例直接说明：进入 `with DockerWorkspace(...)` 时容器处于运行状态；退出后容器会被自动停止和清理。
- 用 `DockerWorkspace` 创建 `Conversation(...)` 时，示例断言其为 `RemoteConversation`。

Evidence 状态：`Verified by official docs`

来源：

- `sdk/guides/agent-server/docker-sandbox`

#### Remote Agent Server 的官方口径

- Remote Agent Server 是 HTTP/WebSocket 服务。
- 它负责运行 agents、管理 workspaces、流式传输 events，并处理命令与文件操作。
- “switching is just changing the workspace argument; your Conversation code stays the same” 是官方直接表述。
- 官方将 workspace 描述为 isolated environment（local、Docker、或 remote VM）where the agent code runs。

Evidence 状态：`Verified by official docs`

来源：

- `sdk/guides/agent-server/overview`

#### Conversation persistence 的最小官方口径

- 官方明确支持 `persistence_dir` + `conversation_id` 的保存与恢复。
- 当前页面明确给出持久化目录包含 `base_state.json` 和 `events/`。
- 当前页面把 `base_state.json` 描述为保存 agent configuration、execution status、statistics、secrets、agent_state。
- 当前页面把 `events/event-*.json` 描述为保存 message history、tool calls、observations、all conversation events。

Evidence 状态：`Verified by official docs`

来源：

- `sdk/guides/convo-persistence`

#### V1 配置页面中可直接保留的配置口径

- `OH_PERSISTENCE_DIR`：where OpenHands stores local state。
- `SANDBOX_VOLUMES`：mount host directories into the sandbox。
- 一些部署仍使用 legacy `RUNTIME` 环境变量选择 sandbox provider：`docker` / `process` / `remote`。

Evidence 状态：`Verified by official docs`

来源：

- `openhands/usage/advanced/configuration-options`

### 4.2 需要降级的结论

#### OpenHands 项目定位

可以保留为：OpenHands 是开源 coding agent / software agent platform，官方资料强调 agent、workspace / sandbox、agent server 与远程执行能力。

但不应再写成过强的“官方唯一定位定义”。

Evidence 状态：`Observed from official docs`

原因：多页文档共同形成这个认识，但当前复核页中缺少单一页面对整个平台定位的统一、强定义式陈述。

#### V0 → V1 迁移关系

可以保留为：V1 是 OpenHands V1 effort / architectural rework，文档明确在重构 V0 的若干问题（mandatory sandboxing、mutable state、tight coupling）。

不应保留为：

- “V0 将于 2026 年 4 月移除”
- “已经确认完整迁移时间线”

Evidence 状态：`Observed / Inferred from official docs`

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

Evidence 状态：`Inferred from official docs`

原因：官方页面支持“术语重心迁移”和“legacy runtime knob 仍存在”，但没有直接给出完整替代关系图。

### 4.3 必须删除或留到源码阶段的问题

以下结论当前不能写成官方文档阶段定论：

- V1 是否明确以 `SandboxService` 替代 Runtime。
- workspace 是否明确保存完整 file system state。
- conversation persistence 恢复的状态范围是否包含真实工作区文件内容。
- event replay 是否等同于 trajectory replay。
- pause / resume、checkpoint、rollback、snapshot 是否存在。
- workspace、sandbox、conversation / session 之间的完整生命周期绑定关系。

这些内容统一降为：`Unclear / requires source code`。

## 五、DeepWiki / 第三方资料的降级处理

以下材料已明确降级：

- DeepWiki 页面：只作为源码聚合旁证 / 第二阶段线索，不再作为核心官方证据。
- 第三方博客、部署平台文档、社区教程：只作旁证或待核验线索。
- `agentic/temp/web-search/6.md` 中把 DeepWiki 近似写成官方 / maintainer comment 的表述，不再直接沿用到主干项目研究文档。

## 六、当前可用的谨慎结论

### 6.1 Runtime / Sandbox / Workspace

当前可以保留的工作性理解：

- V0 runtime architecture 直接描述的是 Docker runtime + ActionExecutionServer 的执行模型。
- V1 design / workspace / agent-server 文档描述的是 SDK、workspace（sandbox）、agent server 的分层与远程执行路径。
- Workspace 至少是执行与文件操作的抽象接口；Local 与 Remote 的隔离级别由官方页面直接给出。
- DockerWorkspace 是通过 agent-server 运行的 docker-based remote workspace，并带有自动容器生命周期管理示例。

Evidence 状态：`Observed / Verified by official docs`

### 6.2 Persistence / Recovery

当前只能保留：

- OpenHands 官方页面直接支持 conversation persistence。
- 当前页面直接说明持久化包含 `base_state.json` 与 `events/`，并支持同一 `conversation_id` 的恢复。
- 当前页面没有直接把 pause / resume、checkpoint、rollback、snapshot 列为正式功能。
- 当前页面也没有直接给出“保存完整 workspace 文件系统状态”的明确原文。

Evidence 状态：`Verified / Unclear`

### 6.3 Workspace Lifecycle / Sharing

当前官方文档阶段只能保留：

- Conversation 与 workspace 存在直接关系。
- DockerWorkspace / LocalWorkspace / RemoteWorkspace 的生命周期行为可能不同。
- `SANDBOX_VOLUMES` 提供宿主目录挂载到 sandbox 的官方配置入口。

当前不能保留为定论：

- workspace 是否随 session / sandbox 自动创建或销毁。
- 是否支持跨 session / cross-task / cross-agent workspace 共享。
- 是否存在 overlay、copy-on-write、git worktree 级共享模型。

Evidence 状态：`Observed / Unclear`

## 七、源码阶段待核验问题

源码阶段应优先核验：

1. V1 中 workspace、sandbox、runtime、conversation / session 的核心类、接口与依赖关系。
2. `DockerWorkspace` / `RemoteWorkspace` / `LocalWorkspace` 的创建、销毁、复用与清理逻辑。
3. `SANDBOX_VOLUMES`、工作目录、容器挂载与宿主路径映射的真实实现。
4. `persistence_dir` / `OH_PERSISTENCE_DIR` 保存的状态范围，尤其是否包含真实文件系统内容，还是仅 conversation state / events。
5. conversation 恢复时 sandbox / workspace 是否重建、复用或重新挂载。
6. event / trajectory 是否能够支撑 action → observation → artifact 的可追溯链。
7. 是否存在 pause / resume、checkpoint、rollback、snapshot、overlay / copy-on-write 机制。
8. V0 runtime 与 V1 sandbox / workspace 的迁移关系到底是术语迁移还是架构重组。

## 八、对 `runtime-and-sandbox.md` 的处理建议

当前官方资料仍不足以支撑稳定的 runtime / sandbox / workspace 生命周期结论。

因此：

- `runtime-and-sandbox.md` 不应在本阶段扩写成稳定分析文档。
- 该文件更适合继续作为源码阶段占位，并只保留更清晰的待核验问题。

## 九、对环境主线的最小回填建议

当前阶段建议只做以下最小回填：

- 在 `agentic/05-environments/candidates.md` 中把 OpenHands 状态更新为“官方原文复核完成，待源码核验”。
- 暂不大改 `workspace-lifecycle.md`；等源码阶段核验后，再把具体 Claim 回填到 `## Evidence`。
- 如果源码阶段发现 workspace 与 sandbox / runtime 的关系和当前主干分类冲突，再更新 `agentic/05-environments/conflict.md`。
