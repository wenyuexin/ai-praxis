# OpenHands Environment / Runtime / Sandbox Architecture

> 面向读者的案例架构总览，不包含源码核验细节
> Evidence 状态：以 `Verified` / `Observed` 为主；缺乏直接证据的结论标为 `Inferred` 或 `Unverified`
> 源码核验与证据表见 `./runtime-and-sandbox.md`

## 一、定位

OpenHands 是一个开源 coding agent / software agent platform。就当前已核验的 environment / runtime / sandbox 范围看，其 V1 架构把原先集中在 runtime 口径下的环境生命周期能力，拆分到 app server、sandbox service、SDK workspace 和 agent server 等层次中。

本文件着眼的是 OpenHands 在 **agent 执行环境**方面的架构选择：sandbox service、workspace 抽象、conversation / session 管理和持久化/恢复机制。它不覆盖 OpenHands 的 tool use、agent loop 或前端设计。

## 二、核心组件关系

```text
                      OpenHands App Server
         ┌─────────────────────────────────────┐
         │  AppConversation                     │
         │  ├─ sandbox_id → SandboxService      │
         │  ├─ conversation_id → Event Store    │
         │  └─ agent_server_url → Agent Server  │
         ├─────────────────────────────────────┤
         │  SandboxService (Abstract)           │
         │  ├─ DockerSandboxService  (default)  │
         │  ├─ ProcessSandboxService (local)    │
         │  └─ RemoteSandboxService (legacy)    │
         └─────────────────────────────────────┘
                          │
                          ▼
          ┌───────────────────────────────┐
          │ SDK / Workspace / Agent Server│
          │  ┌─────────────────────────┐  │
          │  │ AsyncRemoteWorkspace    │  │
          │  │ DockerWorkspace         │  │
          │  │ APIRemoteWorkspace      │  │
          │  └─────────────────────────┘  │
          │  ┌─────────────────────────┐  │
          │  │ Conversations / Events  │  │
          │  │ Persistence / Restore   │  │
          │  └─────────────────────────┘  │
          └───────────────────────────────┘
```

三层关系：

- **App Server 层**（当前 `openhands/app_server/`）：管理 conversation 生命周期和 sandbox 服务。不直接拥有 workspace 实例，而是通过 `sandbox_id` / `session_api_key` 建立到 agent server 的连接。
- **Sandbox Service 层**（`app_server/sandbox/`）：三种实现——Docker（默认）、独立进程（`RUNTIME=process`）、远程 runtime（`RUNTIME=remote`，向下兼容 V0）。提供 start / resume / pause / delete 的显式生命周期 API。
- **SDK / Workspace / Agent Server 层**（位于独立 SDK monorepo 的 `openhands-sdk` / `openhands-workspace` / `openhands-agent-server` 三个包中）：承接 workspace 抽象、具体后端和远端 agent-server 服务。`DockerWorkspace`、`APIRemoteWorkspace`、`OpenHandsCloudWorkspace` 等作为 `RemoteWorkspace` 子类，分别接入不同远端执行载体的启动、恢复与清理逻辑。`AsyncRemoteWorkspace` 是 app server 与 agent server 之间的异步 transport 封装。

## 三、Sandbox 生命周期

OpenHands V1 以 SandboxService 承接环境生命周期管理职责。这是一个架构重组（`Observed`），而非仅仅术语迁移：当前已核验的 V1 app server 路径中没有独立 V0 runtime 模块，`RUNTIME` 环境变量主要作为向下兼容的 fallback。

Sandbox 有完整的生命周期状态机：`STARTING → RUNNING → PAUSED → (resume → RUNNING / delete → removed)`，外加 `ERROR` 和 `MISSING` 状态。

- **创建**：`start_sandbox()` 启动对应实现载体（Docker 容器 / 独立进程 / 远端 runtime）。Docker 容器默认使用 `init=True`（tini init 进程），容器名包含 `sandbox_id`。
- **暂停**：`pause_sandbox()` 调用容器 `pause()`（Docker）/ 进程信号（process）/ 远端 API（remote）。
- **恢复**：`resume_sandbox()` 调用容器 `unpause()`（Docker）/ 进程恢复（process）/ 远端 API（remote）。
- **删除**：`delete_sandbox()` 停止容器并清理关联卷（Docker）/ 终止进程并删除目录（process）。

**结论列表：**

| 结论 | Evidence 状态 |
|---|---|
| SandboxService 体系在 V1 中承担了原 runtime 相关的环境生命周期职责 | Observed |
| 三种沙箱实现：Docker / Process / Remote | Verified |
| Sandbox 有显式 start / resume / pause / delete API | Verified |
| DockerSandboxService 在容器创建时使用 `init=True` | Verified |
| ProcessSandboxService 为每个沙箱启动独立 subprocess + 独立工作目录 | Verified |
| `SANDBOX_VOLUMES` 以 `host_path:container_path:mode` 格式解析为 Docker volumes | Verified |

## 四、Workspace 抽象与后端实现

Workspace 在 OpenHands 中分为两层：

**SDK 抽象层**（`openhands-sdk` 包）：`BaseWorkspace` 是抽象基类（context manager + command / file / git 接口），`LocalWorkspace` 运行在宿主文件系统上（pause/resume 为 no-op），`RemoteWorkspace` 通过 HTTP 连接远端 agent-server。`Workspace` 工厂按是否提供 `host` 在 `LocalWorkspace` 与 `RemoteWorkspace` 之间分派。

**具体后端层**（`openhands-workspace` 包）：`DockerWorkspace`、`ApptainerWorkspace`、`APIRemoteWorkspace`、`OpenHandsCloudWorkspace` 等作为 `RemoteWorkspace` 子类，分别在初始化时启动或附着远端执行载体。其中 `DockerWorkspace` 会在初始化阶段启动预构建的 agent-server Docker 容器，将容器 URL 设为 `host`，`__exit__` / `__del__` 触发 cleanup。

**与 app server 的桥接**：app server 在 sandbox 就绪后，通过 `sandbox.exposed_urls` 解析 `agent_server_url`、通过 `sandbox.session_api_key` 建立 `AsyncRemoteWorkspace`，随后在该 workspace 上完成 setup / skills / conversation 启动。

| 结论 | Evidence 状态 |
|---|---|
| workspace 核心类定义位于独立 SDK monorepo（不在 OpenHands 仓库本身） | Verified |
| `Workspace` 工厂按 `host` 在 Local / Remote 之间分派 | Verified |
| `DockerWorkspace` 是 `RemoteWorkspace` 子类，启动容器 + 管理生命周期 | Verified |
| `APIRemoteWorkspace` 通过 Runtime API 附着远端 runtime | Verified |
| app server 通过 `AsyncRemoteWorkspace` 桥接 sandbox 与 agent server | Verified |
| workspace backend 的 `pause()` / `resume()` 行为在当前核验范围内可见测试覆盖 | Observed |

## 五、Conversation 生命周期与持久化

OpenHands V1 使用 `AppConversation` 模型（非 SDK 中的 `Conversation` / `LocalConversation` / `RemoteConversation` 类）。

**Conversation 与 Sandbox 的绑定**：

- `AppConversationInfo` 持久化保存 `sandbox_id`。
- Conversation 启动时，app server 有明确的 sandbox 选择逻辑：
  - 提供 `sandbox_id` 时：直接查找该 sandbox，`PAUSED` 时先 resume。
  - 未提供 `sandbox_id` 时：先尝试复用当前用户已有的运行中 sandbox，找不到才新建。
  - 提供 `parent_conversation_id` 时：未显式指定 `sandbox_id` 则继承父会话的 `sandbox_id`，共享同一 workspace/environment。
- 已持久化 conversation 的读取 / send-message 路径直接使用持久化 `sandbox_id` 查找 sandbox；若 sandbox 为 `PAUSED`，当前接口不会自动重建，而是返回 closed/conflict 状态，要求调用方先显式执行 sandbox resume。

**事件与持久化**：

- Event 以单个 JSON 文件存储在 `{persistence_dir}/{user_id}/v1_conversations/{conversation_hex_id}/{event_id_hex}.json` 路径下。
- 支持 S3 和 GCS 替代后端。
- Conversation metadata 通过 SQLAlchemy 持久化到数据库。

**前端 reopen 路径**：用户重新打开既有 conversation 页面时，前端先走 `GET /api/v1/app-conversations?ids=...` 取回 metadata，再用 `conversation_url` / `session_api_key` 直接重连既有 agent-server WebSocket。若 `sandbox_status === "PAUSED"`，页面初次打开或标签重新可见时显式触发 `POST /api/v1/sandboxes/{sandbox_id}/resume`。

**两种暂停语义**：agent-server 在 conversation 层显式区分 `pause`（等待当前 LLM 调用自然结束后进入 paused）与 `interrupt`（立即取消运行中的 `arun()` 任务并进入 paused）。两者都可恢复，但不能被视为同一种暂停语义。

| 结论 | Evidence 状态 |
|---|---|
| Conversation 持久化保存 `sandbox_id`，支持恢复时按 ID 查找 | Verified |
| App server 具备 sandbox 选择/复用/新建的分支逻辑 | Verified |
| 子 conversation 可继承父会话的 `sandbox_id` 共享环境 | Verified |
| 已持久化 conversation 读取路径不会自动重建 PAUSED sandbox | Verified |
| Event 以 JSON 文件存储在 `v1_conversations/` 目录 | Verified |
| 前端 reopen 路径优先走显式 resume 而非重建 sandbox | Verified |
| `pause` 与 `interrupt` 在 agent-server 层有不同语义 | Verified |
| `base_state.json` 在当前已核验的 OpenHands 仓库源码范围内未找到 | Unverified |

## 六、对 Agentic 环境研究的启发

1. **SandboxService 承接 runtime 生命周期职责**：当前已核验路径展示了一个从 monolithic runtime 口径走向分层 sandbox service 的架构演进路径。其对子领域的启发是：sandbox 不必作为 agent 框架的隐式假设，而应作为独立的生命周期管理服务。

2. **Workspace 抽象与运行时解耦**：workspace 核心类和具体后端分布在 SDK / workspace backend / agent-server 三层，而非绑定在单一仓库中。这支持了 "workspace 可以独立于特定 runtime 实现" 的模型——`LocalWorkspace` 不需要任何容器基础设施。

3. **Conversation 恢复能力与文件系统恢复能力的分离**：OpenHands 在 conversation-level restore、workspace pause/resume、sandbox reuse 方面做得相当充分，但证据并不指向 "文件系统可被独立重建"。这个分离本身就是一个值得记录到子领域的设计现象。

4. **Pause / Interrupt 的语义区分**：`pause`（等待自然收束）与 `interrupt`（立即取消）的区分，超出了常规 sandbox lifecycle 的范围，进入了 agent governance 层面。这说明 recovery layer 的设计不能脱离 agent loop 的控制流特性。

5. **公开 REST 契约约束**：`openhands-agent-server/AGENTS.md` 明确将 `/api/**` 下的 REST 契约视为公开 API，意味着与 pause / interrupt / resume 相关的接口差异应按兼容性和弃用窗口理解。这对子领域 "API 契约治理是否属于 environment 设计" 是一个有用案例。

## 七、未验证 / 待继续补证的问题

以下问题在 OpenHands + software-agent-sdk 的当前核验范围内尚未找到直接证据：

- 底层 replay engine 是否存在，是否能支撑 event → workspace file state 的完整重建。
- Checkpoint / snapshot / overlay / copy-on-write 的独立实现。
- 恢复后 `working_dir` 与文件系统内容的严格连续性证明。
- V0 遗留代码是否已被完整移除，还是仍存在于当前仓库的未核验目录中。
- SDK / agent-server 内部是否存在超出容器/runtime 连续性之外的 workspace 文件状态重建机制。

## Evidence

- Status: Verified / Observed / Unverified
- Sources: `./runtime-and-sandbox.md`（源码核验证据表），`./official-docs.md`（官方资料归档），基准来源 `https://github.com/OpenHands/OpenHands` 下的 `openhands/app_server/` 源码核验，基准来源 `https://github.com/OpenHands/software-agent-sdk` 下的 `openhands-sdk` / `openhands-workspace` / `openhands-agent-server` 源码核验
- Trace: 从 OpenHands 案例研究中的 `runtime-and-sandbox.md`（约 330 行源码核验证据）提炼面向读者的架构总览，只保留证据较稳定的结论；未验证问题显式标记为 Unverified，不写成定论
- Needs: 跨层端到端绑定图（app_server ↔ sandbox ↔ agent-server ↔ workspace backend ↔ conversation persistence）；replay engine / artifact traceability；pause/resume 后文件系统连续性的细粒度证据