# OpenHands Runtime and Sandbox

> **本文档定位**：机制专题 / 源码核验型文档。它不是 OpenHands 完整案例总览，而是聚焦 runtime / sandbox / workspace 这一组环境的源码核验结果和证据表。
>
> 面向读者的案例架构总览见：`./architecture.md`
>
> 如果你只需要理解 OpenHands 在 environment / runtime / sandbox 方面的架构决定和主要结论，建议先读 `architecture.md`；如果需要查看具体 Claim 的源码证据、搜索路径和待补证问题，再回到本文。

> 阶段：源码核验完成
> 证据状态：以 `Verified` / `Observed` / `Unverified` 为主；其中"在当前核验范围内未找到"按 `Unverified` 处理，需要跨仓库或跨层补证的内容按 `Observed` 或 `Unverified` 处理，并在说明中明确搜索范围、已核验边界与缺失对象
> 源码仓库：`https://github.com/OpenHands/OpenHands`
> 分析对象：OpenHands 仓库中的 `openhands/app_server/`（V1 应用服务器）核心模块

## 一、源码仓库确认

本轮核验所对应的上游源码仓库：`https://github.com/OpenHands/OpenHands`

本轮实际聚焦的仓库内路径：`openhands/app_server/`

主包结构：

```
openhands/
├── app_server/               # V1 应用服务器（主分析对象）
│   ├── app_conversation/     # Conversation 生命周期管理
│   ├── sandbox/              # Sandbox（runtime）管理
│   ├── event/                # Event 存储服务
│   ├── config.py             # 全局配置，含 persistence / sandbox 选择
│   ├── conversation_paths.py # 持久化路径辅助
│   └── ...
├── server/                   # 旧版/替代服务器入口
├── analytics/                # 遥测
```

## 二、源码阅读地图

### 2.1 Sandbox Service 体系（替代 V0 Runtime）

当前源码中**没有** V0 "runtime" 概念的独立模块。V0 runtime 已被 SandboxService 体系完全取代。Legacy `RUNTIME` 环境变量仅作为向下兼容的 fallback，映射到沙箱实现选择。

**核心抽象：**

| 文件 | 类 | 职责 |
|------|-----|------|
| `app_server/sandbox/sandbox_service.py` | `SandboxService` (ABC) | 沙箱服务抽象接口：search / get / start / resume / pause / delete / wait_for_running |
| `app_server/sandbox/sandbox_models.py` | `SandboxInfo`、`SandboxStatus` | 沙箱数据模型，状态含 STARTING / RUNNING / PAUSED / ERROR / MISSING |

**三个实现：**

| 实现 | 文件 | 执行载体 |
|------|------|----------|
| `DockerSandboxService` | `app_server/sandbox/docker_sandbox_service.py` | Docker 容器 |
| `ProcessSandboxService` | `app_server/sandbox/process_sandbox_service.py` | 独立 Python 进程（subprocess + psutil） |
| `RemoteSandboxService` | `app_server/sandbox/remote_sandbox_service.py` | HTTP 远程 Runtime API（适配旧协议） |

### 2.2 Workspace（SDK / workspace 仓库对象，定义不在当前 OpenHands 仓库范围）

**关键发现：** `BaseWorkspace`、`LocalWorkspace`、`RemoteWorkspace`、`AsyncRemoteWorkspace`、`Workspace` 这些类的对象定义**不在当前已核验的 OpenHands 仓库源码范围中**。它们属于独立 SDK 仓库中的 `openhands-sdk` 包（`openhands.sdk.workspace`）；而 `DockerWorkspace`、`DockerDevWorkspace`、`ApptainerWorkspace`、`APIRemoteWorkspace`、`OpenHandsCloudWorkspace` 这些更具体的后端实现，则位于同一 monorepo 下的 `openhands-workspace` 包。

当前已进一步确认的 SDK / workspace 层事实：

- `BaseWorkspace` 是抽象基类，定义了 context manager、`execute_command`、`file_upload`、`file_download`、`git_changes`、`git_diff` 等基础契约；`pause()` / `resume()` 作为统一接口保留给具体实现覆盖。
- `LocalWorkspace` 直接运行在宿主文件系统上，命令执行走本地 shell，`pause()` / `resume()` 是 no-op。
- `RemoteWorkspace` 通过 HTTP 客户端连接远端 agent-server，命令执行、文件上传下载、git 查询等都通过远端 API 发起；当前已确认它自身主要承担“远端 agent-server 客户端”职责，而不是容器或 runtime 生命周期拥有者。
- `Workspace` 工厂按是否提供 `host`，在 `LocalWorkspace` 与 `RemoteWorkspace` 之间分派。
- `AsyncRemoteWorkspace` 是 SDK 中的异步远端工作区实现，具备 `/health` 探测、异步命令执行、文件上传下载与 git 查询能力；当前更像 `app_server` 使用的异步 transport / client 封装，而不是容器或 runtime 生命周期拥有者。

此外，`openhands-workspace` 中的具体后端实现现在已能明确分层：

- `DockerWorkspace` 是 `RemoteWorkspace` 的子类，会在初始化阶段启动预构建的 agent-server Docker 容器，等待健康检查通过后，把容器 URL 设为 `host`，再初始化远端客户端；`pause()` / `resume()` 直接调用同一容器的 `docker pause` / `docker unpause()`，`__exit__` / `__del__` 都会触发 cleanup。
- `DockerDevWorkspace` 继承 `DockerWorkspace`，支持基于开发镜像 / base image 构建 agent-server 容器。
- `ApptainerWorkspace` 也是 `RemoteWorkspace` 的子类，只是底层执行载体从 Docker 换成了 Apptainer。
- `APIRemoteWorkspace` 是 `RemoteWorkspace` 的子类，通过 Runtime API 启动或附着远端 runtime，待 runtime 可用后把 runtime URL 设为 `host`、把 session API key 写回客户端认证，再初始化远端客户端；`__exit__` / `__del__` 都会触发 cleanup，按配置执行 pause 或 stop。
- `OpenHandsCloudWorkspace` 同样继承 `RemoteWorkspace`，通过 Cloud API 创建或恢复 sandbox，再把 agent-server URL 暴露给客户端；当前已确认它支持 `resume()` 恢复 sandbox，但 `pause()` 仍是 `NotImplementedError`。

在当前已核验的 OpenHands 仓库源码范围中，仅看到以下 workspace 引用：

- `app_server/app_conversation/app_conversation_service.py` — 从 SDK 导入 `AsyncRemoteWorkspace`：
  ```python
  from openhands.sdk.workspace.remote.async_remote_workspace import AsyncRemoteWorkspace
  ```

### 2.3 Conversation（V1 模型，非 SDK 中的 Conversation 类）

源码中**没有** `Conversation` / `LocalConversation` / `RemoteConversation` 类。V1 使用独立的 `AppConversation` 模型：

| 文件 | 类 | 职责 |
|------|-----|------|
| `app_server/app_conversation/app_conversation_models.py` | `AppConversation`、`AppConversationInfo` | V1 Conversation 数据模型 |
| `app_server/app_conversation/app_conversation_service.py` | `AppConversationService` (ABC) | Conversation CRUD + start task |
| `app_server/app_conversation/sql_app_conversation_info_service.py` | SQL-based 实现 | 持久化到数据库 |

Conversation 与 Sandbox 的关系当前已能进一步落实到 app_server 启动链路：

- `AppConversationInfo` 持久化保存 `sandbox_id`。
- `AppConversation` 在读取侧补充 `sandbox_status`、`execution_status`、`conversation_url`、`session_api_key` 等运行态字段。
- `AppConversationStartTask` 在启动过程中显式携带 `sandbox_id` 与 `agent_server_url`，并通过 `WAITING_FOR_SANDBOX` → `PREPARING_REPOSITORY` → `RUNNING_SETUP_SCRIPT` → `SETTING_UP_GIT_HOOKS` → `SETTING_UP_SKILLS` → `STARTING_CONVERSATION` → `READY` 状态推进。
- `AppConversationStartRequest` 同时允许显式传入 `conversation_id` 与 `sandbox_id`：若请求携带 `sandbox_id`，app_server 会直接 `get_sandbox(sandbox_id)`，若该 sandbox 为 `PAUSED` 则先 `resume_sandbox(sandbox.id)`；若请求不携带 `sandbox_id`，则先尝试复用当前用户已有运行中 sandbox，找不到时才 `start_sandbox(...)` 新建。
- 若请求携带 `parent_conversation_id`，app_server 会先读取父会话的 `AppConversationInfo`，并在未显式提供 `sandbox_id` 时继承父会话的 `sandbox_id`，以共享同一 workspace / environment。
- 已持久化 conversation 的读取路径（`get_app_conversation(...)` / router 侧 message 入口）会直接使用持久化的 `conversation.sandbox_id` 去 `get_sandbox(...)`；若 sandbox 为 `PAUSED`，当前 router 不会自动重建或自动恢复，而是把该会话视为“closed conversation”或要求调用方先执行 `POST /api/v1/sandboxes/{sandbox_id}/resume`。
- 更上层的 reopen 入口当前也已落实：前端进入既有会话页时，`frontend/src/routes/conversation.tsx` 会通过 `useActiveConversation()` 拉取 `GET /api/v1/app-conversations?ids={conversation_id}`，`WebSocketProviderWrapper` 再把返回的 `conversation_url` / `session_api_key` 交给 `ConversationWebSocketProvider` 直接重连既有 agent-server；若读取到 `sandbox_status === "PAUSED"`，`useSandboxRecovery()` 会在页面初次打开或标签页重新可见时调用统一恢复 hook，走显式 resume 路径恢复该 sandbox，而不是隐式新建另一套 sandbox。

### 2.4 持久化与 Event 存储

当前在 `software-agent-sdk` 本地源码中还能进一步确认两点：

- `LocalConversation.__init__(...)` 接受显式 `persistence_dir`，并在创建 `_state` 时通过 `ConversationState.create(...)` 走“create-or-resume”路径；源码注释直接写明工厂会根据 `BASE_STATE` 决定创建还是恢复。
- agent-server 的 `ConversationService` 同时区分了两种暂停语义：`pause_conversation(...)` 是“等待当前 LLM 调用自然结束后进入 paused”，`interrupt_conversation(...)` 则会直接取消运行中的 `arun()` 任务并立即进入 paused；后者对应 `POST /{conversation_id}/interrupt`，前者对应 `POST /{conversation_id}/pause`。这也意味着在语义分析时不能把两者都简单记成“resume 前的 pause”，因为它们在中断时机和运行中请求处理上并不等价。
- 结合 `openhands-agent-server/AGENTS.md`，还可进一步确认：`/api/**` 下的 REST 契约被视为公开 API，任何后续若发现 `pause` / `interrupt` / `resume` 的请求或响应差异，都必须按 5 个 minor release 的弃用窗口处理，而不能把当前行为差异当成可随意变动的内部实现细节。

**`persistence_dir` 默认值：**

- `config.py:get_default_persistence_dir()` — 优先读 `OH_PERSISTENCE_DIR`，fallback 到旧 `FILE_STORE_PATH`，默认 `~/.openhands`
- `AppServerConfig.persistence_dir` — 全局配置字段

**Event 存储路径：**

`{persistence_dir}/{user_id}/v1_conversations/{conversation_hex_id}/{event_id_hex}.json`

- `conversation_paths.py` — `V1_CONVERSATIONS_DIR = 'v1_conversations'`，`get_conversation_dir()` / `get_conversation_path()` 路径辅助
- `event/filesystem_event_service.py` — `FilesystemEventService` 读写单个 JSON 文件
- `event/aws_event_service.py` — S3 版
- `event/google_cloud_event_service.py` — GCS 版

**`base_state.json`**：在当前已核验的 OpenHands 仓库源码范围内**未找到**。官方文档阶段的 `base_state.json` 可能是 V0 遗留或独立 SDK 中的概念。

**blocked secret names（`constants.py`）：**
- `OH_CONVERSATIONS_PATH` — agent-server 中的会话路径
- `OH_BASH_EVENTS_DIR` — agent-server 中的 bash 事件目录

### 2.5 SANDBOX_VOLUMES

在 `config.py:config_from_env()`（第 361-387 行）中解析：

- 格式：`host_path:container_path:mode`（mode 默认为 `rw`）
- 解析为 `VolumeMount(host_path, container_path, mode)` 对象列表
- 传入 `DockerSandboxServiceInjector(mounts=...)`
- 在 `docker_sandbox_service.py` 的 `start_sandbox()`（第 439-445 行）中转为 Docker volumes 字典

## 三、关键结论

### 3.1 Sandbox 生命周期

**结论：OpenHands V1 使用 SandboxService 替代了 V0 Runtime。Sandbox 有显式的 start / resume / pause / delete 生命周期 API。**

Evidence 状态：`Verified`

来源：
- `app_server/sandbox/sandbox_service.py:L29-L187` — `SandboxService` ABC，定义 start_sandbox / resume_sandbox / pause_sandbox / delete_sandbox / wait_for_sandbox_running
- `app_server/sandbox/docker_sandbox_service.py:L360-L554` — DockerSandboxService 的完整实现

**结论：DockerSandboxService 是默认实现。RUNTIME=local/process 使用 ProcessSandboxService，RUNTIME=remote 使用 RemoteSandboxService。**

Evidence 状态：`Verified`

来源：
- `app_server/config.py:L334-L396` — `config_from_env()` 中根据 `RUNTIME` 环境变量选择注入器

### 3.2 Container 生命周期管理

**结论：DockerSandboxService 创建容器时使用 `docker_client.containers.run()`，启动后跟踪容器名称（`oh-agent-server-{sandbox_id}`）。暂停使用 `container.pause()` / `container.unpause()`。删除时先 `container.stop(timeout=10)`，再 `container.remove()`，最后删除关联卷 `openhands-workspace-{sandbox_id}`。**

Evidence 状态：`Verified`

来源：
- `app_server/sandbox/docker_sandbox_service.py:L360-L491` — start_sandbox（容器创建）
- `app_server/sandbox/docker_sandbox_service.py:L496-L513` — resume_sandbox（unpause / start）
- `app_server/sandbox/docker_sandbox_service.py:L515-L527` — pause_sandbox（pause）
- `app_server/sandbox/docker_sandbox_service.py:L529-L554` — delete_sandbox（stop + remove + 卷清理）

**结论：容器使用 `init=True`（Docker tini init 进程）以确保正确的信号处理和僵尸进程回收。**

Evidence 状态：`Verified`

来源：
- `app_server/sandbox/docker_sandbox_service.py:L476`

### 3.3 ProcessSandboxService

**结论：ProcessSandboxService 将每个沙箱实现为独立的 Python subprocess，各自监听不同端口，使用独立工作目录。**

Evidence 状态：`Verified`

来源：
- `app_server/sandbox/process_sandbox_service.py:L67-L75` — 类文档字符串
- `app_server/sandbox/process_sandbox_service.py:L103-L107` — `_create_sandbox_directory()` 创建 `{base_working_dir}/{sandbox_id}`
- `app_server/sandbox/process_sandbox_service.py:L124-L155` — `_start_agent_process()` 启动 `python -m openhands.agent_server --port N`
- `app_server/sandbox/process_sandbox_service.py:L374-L408` — delete_sandbox 终止进程 + 清理目录

### 3.4 Conversation Persistence

**结论：Event 以单个 JSON 文件存储在 `{persistence_dir}/{user_id}/v1_conversations/{conversation_hex_id}/{event_id_hex}.json` 路径下。支持 S3 和 GCS 替代后端。Conversation info 通过 SQLAlchemy 持久化到数据库。**

Evidence 状态：`Verified`

来源：
- `app_server/conversation_paths.py:L12-L73` — 路径辅助，`V1_CONVERSATIONS_DIR = 'v1_conversations'`
- `app_server/event/filesystem_event_service.py:L17-L43` — `FilesystemEventService`，读写 JSON 文件
- `app_server/event/event_router.py` — Event REST API（`/conversation/{id}/events/search` / `count` / `batch`）
- `app_server/config.py:L75-L89` — `get_default_persistence_dir()`

**结论：在本轮已核验的 OpenHands 仓库源码范围内，未确认 `base_state.json` 持久化机制。官方文档阶段提到的 `base_state.json` 未在当前核验范围中出现。**

Evidence 状态：`Unverified`

说明：在 OpenHands 仓库中的 `openhands/app_server/`、`openhands/server/`、`openhands/analytics/` 三个 Python 包范围内搜索 `base_state.json`，未找到匹配项。当前只能说明它**不在本轮已核验的源码范围内**；此文件可能是 V0 遗留、SDK 中的概念，或官方资料中的过期表述，因此暂按 `Unverified` 处理。

### 3.5 SANDBOX_VOLUMES 实现

**结论：`SANDBOX_VOLUMES` 环境变量以 `host_path:container_path:mode` 格式解析为 `VolumeMount` 对象，传入 DockerSandboxService，最终转为 Docker volumes 字典。**

Evidence 状态：`Verified`

来源：
- `app_server/config.py:L361-L387` — 解析 `SANDBOX_VOLUMES`
- `app_server/sandbox/docker_sandbox_service.py:L61-L68` — `VolumeMount` 模型
- `app_server/sandbox/docker_sandbox_service.py:L438-L445` — 卷挂载

### 3.6 Workspace 与 Sandbox 的关系

**结论：Workspace 抽象与基础实现的对象定义不在当前已核验的 OpenHands 仓库源码范围中，而在独立 SDK monorepo 内的 `openhands-sdk` / `openhands-workspace` / `openhands-agent-server` 三个包中。SDK 层已可确认 `Workspace` 工厂按 `host` 在 `LocalWorkspace` 与 `RemoteWorkspace` 之间分派，基础 `RemoteWorkspace` / `AsyncRemoteWorkspace` 主要承担 agent-server HTTP client / transport 职责；`DockerWorkspace`、`ApptainerWorkspace`、`APIRemoteWorkspace`、`OpenHandsCloudWorkspace` 则作为 `RemoteWorkspace` 子类，分别接入不同远端执行载体的启动 / 恢复 / 清理逻辑。进一步结合 app_server 启动链路，可以确认：`AppConversationStartTask` 会先等待 sandbox 就绪，再通过 `sandbox.exposed_urls` 解析 `agent_server_url`、通过 `sandbox.session_api_key` 建立 `AsyncRemoteWorkspace(host=agent_server_url, api_key=..., working_dir=...)`，随后在该 workspace 上完成仓库准备、setup script、git hooks、skills 加载与 conversation 启动请求构造。关于 restore / reuse，当前源码已能确认的是：app_server 并不会在启动入口无条件新建 sandbox；若请求携带 `sandbox_id`，它会直接查找该 sandbox，并在 `PAUSED` 时先恢复；若未携带 `sandbox_id`，则先尝试复用当前用户已有运行中 sandbox，只有找不到可用 sandbox 时才新建；若请求携带 `parent_conversation_id`，则还会在未显式提供 `sandbox_id` 时继承父会话的 `sandbox_id`，以共享同一 workspace / environment。另一方面，已持久化 conversation 的读取 / send-message 路径会直接使用持久化的 `conversation.sandbox_id` 去查 sandbox；若 sandbox 为 `PAUSED`，当前 router 不会自动重建或自动恢复，而是把该会话视作 closed conversation 或要求调用方先显式执行 sandbox resume。进一步沿前端 reopen 链路往下追，可以确认 `conversation_url` 实际指向 sandbox 暴露出的 agent-server `/api/conversations/{conversation_id}` 资源；页面恢复时优先走 `POST /api/v1/sandboxes/{sandbox_id}/resume`，恢复后再用 `conversation_url` / `session_api_key` 去重连 `/sockets/events/{conversation_id}` 等 agent-server 端点，而不是在 app_server 内重放事件后重建一套新 conversation。当前还能进一步落实到三类底层连续性：其一，Docker 路径直接把 `sandbox_id` 编进容器名，`pause_sandbox()` / `resume_sandbox()` 分别调用同一容器的 `pause()` / `unpause()` 或 `start()`，并把 `sandbox_spec.working_dir` 与显式 volume mounts 直接挂到该容器上；其二，Remote runtime 路径会把 OpenHands `sandbox_id` 写成远端 `session_id`，resume 时再按已存在的 `runtime_id` 调 `/resume`，同时更新新的 `session_api_key`；其三，SDK / agent-server 自身已显式存在 conversation persistence / restore 语义：workspace 抽象层定义 `pause()` / `resume()`，`tests/workspace/test_workspace_pause_resume.py` 覆盖 Docker / API / Cloud 等分支，`LocalConversation` 接受 `persistence_dir`，`tests/cross/test_conversation_restore_behavior.py` 对 restore 兼容性做专门约束，而 agent-server 启动时还会扫描持久化 `conversations_dir` 并恢复已存在 conversation。这说明“恢复”并不只停留在 app_server 外层路由，而是贯穿 SDK、workspace backend 与 agent-server 的显式能力边界。尽管如此，现有证据仍更像“沿用既有 container/runtime 身份与其工作目录状态，并在其上恢复 conversation/service 元数据”，而不是“根据持久化 events / base state 重建完整文件系统”。当前仓库内能够直接确认的是：conversation / event 历史会持久化并可重新拉取，conversation restore 行为也被当作一等语义测试，但尚未看到“仅靠事件回放即可重建完整工作区文件系统状态”的证据；现有线索更指向 pause / resume 主要依赖既有 sandbox / container / process / runtime 连续性与既有 `working_dir` / volume 状态。因此，“是否复用既有 sandbox” 已可部分回答；但“恢复既有 conversation 时是否总能沿用其持久化 `sandbox_id`、以及新旧 sandbox 下文件状态是否严格连续”仍需继续确认。**

Evidence 状态：`Observed`

来源：
- `https://github.com/OpenHands/OpenHands/blob/main/openhands/app_server/app_conversation/app_conversation_service.py` — import `AsyncRemoteWorkspace` from SDK；`run_setup_scripts(...)` 明确接收 `workspace: AsyncRemoteWorkspace`
- `https://github.com/OpenHands/OpenHands/blob/main/openhands/app_server/app_conversation/app_conversation_models.py` — `AppConversationStartRequest.sandbox_id / conversation_id`、`AppConversationInfo.sandbox_id`、`AppConversation.sandbox_status / execution_status / conversation_url / session_api_key`、`AppConversationStartTask.sandbox_id / agent_server_url / 状态机`
- `https://raw.githubusercontent.com/OpenHands/OpenHands/main/openhands/app_server/app_conversation/live_status_app_conversation_service.py` — `_wait_for_sandbox_start(...)` 中：有 `request.sandbox_id` 时直接 `get_sandbox(...)`，若为 `PAUSED` 则 `resume_sandbox(...)`；无 `request.sandbox_id` 时先尝试复用当前用户运行中 sandbox，找不到才 `start_sandbox(...)`；`_inherit_configuration_from_parent(...)` 中若请求携带 `parent_conversation_id` 则可继承父会话 `sandbox_id` 共享环境；sandbox 就绪后再通过 `_get_agent_server_url(sandbox)` 与 `sandbox.session_api_key` 构造 `AsyncRemoteWorkspace(...)`，并在其上执行 setup / skills / start conversation 流程
- `https://raw.githubusercontent.com/OpenHands/OpenHands/main/openhands/app_server/app_conversation/app_conversation_router.py` — 已持久化 conversation 的读取 / send-message 路径直接使用 `conversation.sandbox_id` 查找 sandbox；若 sandbox 为 `PAUSED`，当前接口返回 closed / conflict 状态，要求调用方先执行显式 resume，而不是自动重建或自动恢复
- `https://github.com/OpenHands/OpenHands/tree/main/frontend/src/routes`、`https://github.com/OpenHands/OpenHands/tree/main/frontend/src/hooks/query`、`https://github.com/OpenHands/OpenHands/blob/main/frontend/src/api/conversation-service/v1-conversation-service.api.ts` — 既有会话页的 reopen 入口会走 `GET /api/v1/app-conversations?ids={conversation_id}`，先取回持久化 conversation 元数据
- `https://github.com/OpenHands/OpenHands/blob/main/frontend/src/contexts/websocket-provider-wrapper.tsx`、`https://github.com/OpenHands/OpenHands/tree/main/frontend/src/hooks`、`https://github.com/OpenHands/OpenHands/blob/main/frontend/src/api/sandbox-service/sandbox-service.api.ts`、`https://github.com/OpenHands/OpenHands/blob/main/openhands/app_server/sandbox/sandbox_router.py` — 页面加载后会把 `conversation_url` / `session_api_key` 交给 WebSocket provider 直接重连既有 agent-server；若 `sandbox_status === "PAUSED"`，则在页面初次打开或标签重新可见时调用统一恢复 hook，实际先走 `POST /api/v1/sandboxes/{sandbox_id}/resume` 恢复 sandbox
- `https://github.com/OpenHands/OpenHands/blob/main/frontend/src/utils/websocket-url.ts`、`https://github.com/OpenHands/OpenHands/blob/main/frontend/src/contexts/conversation-websocket-context.tsx`、`https://github.com/OpenHands/OpenHands/blob/main/openhands/app_server/app_conversation/live_status_app_conversation_service.py` — `conversation_url` 实际指向 sandbox 暴露出的 agent-server `/api/conversations/{conversation_id}` 资源；前端重连会据此拼出 `/sockets/events/{conversation_id}` WebSocket 地址，而不是回到 app_server 内部重建 conversation
- `https://github.com/OpenHands/OpenHands/blob/main/openhands/app_server/sandbox/sandbox_spec_models.py`、`https://github.com/OpenHands/OpenHands/blob/main/openhands/app_server/sandbox/docker_sandbox_spec_service.py` — `SandboxSpecInfo` 包含 `working_dir` 字段；当前默认值为 `/home/openhands/workspace`，而 Docker 默认 spec 会把 agent-server 容器工作目录设为 `/workspace/project`
- `https://github.com/OpenHands/OpenHands/blob/main/openhands/app_server/sandbox/docker_sandbox_service.py` — Docker 路径会把 `sandbox_id` 编进容器名，显式传入 `working_dir=sandbox_spec.working_dir` 与 volume mounts；pause / resume 直接调用同一容器的 `pause()` / `unpause()` 或 `start()`，delete 时才停止容器并尝试删除关联 volume
- `https://github.com/OpenHands/OpenHands/blob/main/openhands/app_server/sandbox/remote_sandbox_service.py` — Remote runtime 路径会本地保存 `StoredRemoteSandbox`，并把 OpenHands `sandbox_id` 写入远端 `session_id`；resume 时先取回既有 `runtime_id`，再调用 runtime API `/resume`，并更新新的 `session_api_key`
- `https://raw.githubusercontent.com/OpenHands/OpenHands/main/openhands/app_server/sandbox/sandbox_models.py` — `SandboxInfo` 暴露 `id`、`sandbox_spec_id`、`status`、`session_api_key`、`exposed_urls`
- `https://raw.githubusercontent.com/OpenHands/OpenHands/main/openhands/app_server/sandbox/sandbox_service.py` — `start_sandbox` / `resume_sandbox` / `pause_sandbox` / `delete_sandbox` / `_get_agent_server_url`
- `https://github.com/OpenHands/OpenHands/blob/main/openhands/app_server/sandbox/docker_sandbox_service.py` — `working_dir=sandbox_spec.working_dir`
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-sdk/openhands/sdk/workspace/__init__.py` — SDK 导出 `BaseWorkspace`、`LocalWorkspace`、`RemoteWorkspace`、`AsyncRemoteWorkspace`、`Workspace`
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-sdk/openhands/sdk/workspace/base.py` — `BaseWorkspace` 定义 context manager 与 command/file/git 抽象契约，默认 `__exit__` 不做清理
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-sdk/openhands/sdk/workspace/local.py` — `LocalWorkspace` 直接在宿主文件系统执行，本地 shell 命令，`pause()` / `resume()` 为 no-op
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-sdk/openhands/sdk/workspace/workspace.py` — `Workspace` 工厂按是否提供 `host` 在 `LocalWorkspace` 与 `RemoteWorkspace` 之间分派
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-sdk/openhands/sdk/workspace/remote/base.py` — `RemoteWorkspace` 通过 HTTP 客户端连接 agent-server，并通过远端 API 发起命令执行；其自身更像远端执行 client，默认不拥有容器级 cleanup 生命周期
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-sdk/openhands/sdk/workspace/remote/async_remote_workspace.py` — `AsyncRemoteWorkspace` 是异步远端工作区实现，支持 `/health` 探测、异步命令、文件操作与 git 查询；当前更像异步 transport / client 封装
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-workspace/openhands/workspace/__init__.py` — `openhands-workspace` 导出 `DockerWorkspace`、`DockerDevWorkspace`、`ApptainerWorkspace`、`APIRemoteWorkspace`、`OpenHandsCloudWorkspace` 等后端实现
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-workspace/openhands/workspace/docker/workspace.py` — `DockerWorkspace` 是 `RemoteWorkspace` 子类，初始化阶段启动容器、等待健康检查后写回 `host`，`pause()` / `resume()` 直接控制同一容器，并在 `__exit__` / `__del__` 时 cleanup
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-workspace/openhands/workspace/docker/dev_workspace.py` — `DockerDevWorkspace` 继承 `DockerWorkspace`，支持基于开发镜像 / base image 构建 agent-server 容器
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-workspace/openhands/workspace/apptainer/workspace.py` — `ApptainerWorkspace` 也是 `RemoteWorkspace` 子类，但底层执行载体换成 Apptainer
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-workspace/openhands/workspace/remote_api/workspace.py` — `APIRemoteWorkspace` 是 `RemoteWorkspace` 子类，通过 Runtime API 启动或附着远端 runtime，写回 runtime URL 与 session API key，并在 `__exit__` / `__del__` 时按配置 pause 或 stop
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-workspace/openhands/workspace/cloud/workspace.py` — `OpenHandsCloudWorkspace` 通过 Cloud API 创建或恢复 sandbox，再暴露 agent-server URL 给客户端；当前支持 `resume()`，但 `pause()` 仍未实现
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/tests/workspace/test_workspace_pause_resume.py` — workspace backend 的 pause / resume 语义存在专门测试，覆盖 Docker / API / Cloud 等分支
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-sdk/openhands/sdk/conversation/impl/local_conversation.py` — `LocalConversation` 显式接受 `persistence_dir`，说明 conversation restore 不是只停留在外层 UI / router
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/tests/cross/test_conversation_restore_behavior.py` — restore lifecycle 在测试中被当作显式能力边界约束
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-agent-server/pyproject.toml` — agent-server 是 monorepo 中的独立包，entrypoint 为 `agent-server = "openhands.agent_server.__main__:main"`
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-agent-server/openhands/agent_server/api.py` — agent-server 内部明确组装 conversations、events、bash、git、file、skills、workspace 等 API router
- `https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-agent-server/openhands/agent_server/conversation_service.py` — agent-server 启动时会扫描持久化 conversations 目录并尝试恢复已存在 conversation

当前已能确认 SDK monorepo 内对象分布、工厂入口，以及 Local / Remote / Docker / Apptainer / Runtime API / Cloud 几类 workspace backend 的实现层次；同时也已确认 agent-server 并非抽象依赖名词，而是 monorepo 中真实存在的独立包，负责 conversations、events、bash、git、file、skills、workspace 等 API。结合 OpenHands app_server，可进一步确认其启动链路确实以 `sandbox_id` / `agent_server_url` / `session_api_key` 为桥，把 `SandboxInfo` 转成 `AsyncRemoteWorkspace` 后再继续 conversation 启动。关于 restore / reuse，当前源码已经证明 app_server 具备“优先复用 / 恢复既有 sandbox、必要时再新建”的分支逻辑，并且子会话路径会在未显式指定 `sandbox_id` 时继承父会话的 `sandbox_id` 共享环境；而已持久化 conversation 的 reopen / send-message 路径则会优先按持久化 `sandbox_id` 取回既有 sandbox，并在 `PAUSED` 时要求显式 resume。进一步结合前端 V1 路由也已可确认：用户重新打开既有 conversation 页面时，会先经 `GET /api/v1/app-conversations?ids=...` 取回 conversation 元数据，再用 `conversation_url` / `session_api_key` 直接重连既有 agent-server；仅当 `sandbox_status === "PAUSED"` 时，页面初次打开或标签重新可见才会显式触发 resume。与此同时，software-agent-sdk 本地源码还把“conversation restore”落实成一等能力：workspace 抽象层统一暴露 `pause()` / `resume()`，workspace backend 具备专门测试，`LocalConversation` 支持 `persistence_dir`，agent-server 启动时也会扫描持久化 conversation 目录并恢复既有会话。因此，“哪些上层 reopen 入口会真正触发恢复链路”已从未知收缩为已部分确认，“恢复”也不再只是前端或 app_server 的外层流程词汇。仍待继续确认的部分主要集中在：working_dir 与文件系统状态的严格连续性，以及 agent-server / SDK 内部是否存在超出 container/runtime 连续性之外的 workspace 文件状态重建机制。

## 四、与当前主干文档的关系

### 4.1 支持已有说法

- **`workspace-lifecycle.md` 中 `Per-Task Isolated Workspace` 模型**：DockerSandboxService 每个沙箱一个独立容器 + 独立卷，符合此模型。Evidence 状态：`Observed`（需注意当前已核验范围内仅有容器级隔离，workspace 层抽象在 SDK 中）
- **`official-docs.md` 中 V1 design principles（stateless by default, sandboxing opt-in）**：源码层面确认 SandboxService 是可选注入，`RUNTIME=process` 时不使用容器。Evidence 状态：`Verified`
- **`official-docs.md` 中 Conversation persistence 的 `events/event-*.json` 描述**：源码确认事件存储为 JSON 文件，但路径与官方文档阶段描述不完全一致（V1 使用 `v1_conversations/` 目录）。Evidence 状态：`Verified`

### 4.2 修正已有说法

- **`official-docs.md` 中 `BaseWorkspace` / `LocalWorkspace` / `RemoteWorkspace` / `DockerWorkspace` 的存在范围**：这些类不在当前已核验的 OpenHands 仓库源码范围中。OpenHands 官方资料中提到的 workspace 抽象属于独立 SDK 包。当前核验范围内的 `app_server` 代码只通过 SDK import 使用它们。原始官方资料阶段的 workspace 结论需要缩小适用范围。
- **`official-docs.md` 中 `base_state.json`**：在当前已核验的 OpenHands 仓库源码范围内未找到。如果实际存在，可能在 SDK 或 V0 legacy 目录中。需要进一步确认。
- **V0 → V1 迁移的实质**：源码层确认 V0 runtime 概念已被 SandboxService 完全取代。`RUNTIME` 环境变量仅作为向后兼容的 fallback，映射到三种 sandbox 实现之一。这不是简单的"术语迁移"，而是架构重组。

### 4.3 仍证据不足的位置

- **Workspace 完整对象模型**：已确认 SDK monorepo 中的基础抽象、工厂入口、agent-server 独立包，以及 Docker / Apptainer / Runtime API / Cloud 几类 workspace backend 的实现层次；因此“workspace 定义根本不在 OpenHands 仓库里”这一边界已经收敛。但完整生命周期与对象绑定关系仍未完全闭合，尤其是 `app_server -> sandbox -> agent-server -> workspace backend -> conversation persistence` 各层之间哪些对象拥有文件系统状态、哪些对象只负责连接信息，仍需继续梳理
- **Conversation 恢复时 sandbox 是否重建/复用**：当前已确认 app_server 启动入口存在明确分支：有 `sandbox_id` 时直接查找既有 sandbox，`PAUSED` 时优先 resume；无 `sandbox_id` 时先尝试复用当前用户已有运行中 sandbox，只有找不到时才新建；对子会话还会在未显式指定 `sandbox_id` 时继承父会话的 `sandbox_id`。同时，已持久化 conversation 的读取 / send-message 路径会直接按持久化 `sandbox_id` 查找 sandbox，遇到 `PAUSED` 时要求显式 resume，而不是自动重建。SDK / agent-server 层还进一步证明“conversation restore”本身被当作显式能力维护与测试，因此“是否可能复用既有 sandbox”已得到肯定答案；但在某些异常分支下是否会切换到其他 sandbox 后再附着，仍需继续核验
- **Event replay / trajectory → artifact traceability**：当前本地仓库里已能确认 trajectory 相关配置项、前端下载入口、拉取接口与 event 持久化机制，例如 `config.template.toml` 中的 `save_trajectory_path` / `replay_trajectory_path`、前端的 `download-trajectory.ts` 与 `use-get-trajectory.ts`，以及 app_server 的 event 存储服务；但无论在当前 OpenHands 仓库还是 software-agent-sdk 本地源码中，仍未看到一个可直接指认的 replay engine，或能把 event / observation / tool call / artifact / workspace file change 串成稳定追溯链的实现，因此当前更适合表述为“存在 trajectory / replay 入口与 event 存储，但底层 replay / artifact traceability 机制仍未闭合”
- **Checkpoint / snapshot / overlay / copy-on-write**：在当前已核验的 OpenHands 仓库与 software-agent-sdk 本地源码范围内，仍未找到可直接落位的相关实现
- **Pause / Resume 后的状态完整性**：sandbox pause/resume、`AsyncRemoteWorkspace` 重连入口、workspace backend 的 `pause()` / `resume()` 语义、conversation restore 测试与 agent-server 的持久化恢复入口都已分别确认。现有证据更偏向“依赖既有 sandbox / container / process / runtime 与 `working_dir` / volume 连续性，并在其上恢复 conversation/service 元数据”而不是“从事件持久化中重建文件系统”；同时，前端 `use-v1-resume-conversation.ts` 在 resume 后会显式失效 sandbox 与 VS Code URL 查询，以重新获取新的 runtime metadata / URL。这说明当前至少支持“resume 后重新获取运行时连接信息”，但仍未提供直接证据证明 `working_dir` 或文件系统状态一定被完整保留。因此，resume 之后 workspace 文件状态是否完整、working_dir 是否保持一致，以及 reused sandbox 与 newly started sandbox 的文件可见性差异，仍需要继续做跨 SDK / agent-server / backend 的更细核验

## 五、建议落位

### 5.1 已写入本文件（`runtime-and-sandbox.md`）

本文件已从占位状态升级为源码核验结果文档，包含上述全部 Verified / Observed / Unverified 结论。

### 5.2 应更新 `candidates.md`

`candidates.md` 中的 OpenHands 条目已从“官方原文复核完成，待源码核验”更新为“`openhands/app_server/` 层源码核验完成，SDK 层待补充”。

### 5.3 暂不修改的文件

- `workspace-lifecycle.md`：当前发现不足以回填。需要 SDK 层 workspace 源码才能补充 workspace ↔ sandbox 的完整绑定关系。
- `conflict.md`：没有发现有足够证据修改现有冲突条目。

### 5.4 暂未进入稳定结论的问题

见第六节。

## 六、待核验项

以下问题需要补充信息后才能继续：

1. **Conversation / workspace / agent-server 的跨层绑定图**：`software-agent-sdk` 本地源码已确认 `openhands-sdk`、`openhands-workspace`、`openhands-agent-server` 三层都真实存在；后续需要把 `app_server -> sandbox -> agent-server -> workspace backend -> LocalConversation / persistence` 的对象边界继续画清，尤其区分“谁拥有执行环境 / 文件系统状态”与“谁只是连接、恢复或编排入口”
2. **V0 runtime 遗留代码**：当前 OpenHands 仓库中的 `openhands/` 目录结构与官方文档中的 V0 描述差异较大。需要确认旧 V0 代码是否已被移除或不在当前仓库中
3. **Event replay / trajectory**：Event 存储路径已确认；此外当前本地仓库还可直接看到 trajectory 相关配置与前端入口，例如 `config.template.toml` 中的 `save_trajectory_path` / `replay_trajectory_path`、前端的轨迹下载与拉取 hook。software-agent-sdk 本地源码又补进了 conversation restore、workspace pause/resume 与 agent-server persistence 的证据；但事件如何被底层回放、如何关联 artifact 与 workspace 变化、是否存在独立 replay engine，仍未在当前已核验范围内闭合
4. **Conversation 全生命周期跨层验证**：当前已确认 `SandboxInfo -> _get_agent_server_url(...) -> AsyncRemoteWorkspace(...) -> run_setup_scripts / start conversation request` 这段 app_server 启动链，也已确认启动入口存在“显式 `sandbox_id` 直连 / `PAUSED` 时 resume / 无 `sandbox_id` 时优先复用运行中 sandbox / 否则新建”的分支，且子会话会在未显式指定 `sandbox_id` 时继承父会话 `sandbox_id`；同时，已持久化 conversation 的读取 / send-message 路径会直接按持久化 `sandbox_id` 查找 sandbox，并在 `PAUSED` 时要求显式 resume。software-agent-sdk 本地源码又进一步确认了 workspace backend 的 `pause()` / `resume()` 语义、`LocalConversation` 的 `persistence_dir`、agent-server 启动时的持久化 conversation 恢复入口，以及 `pause_conversation(...)` 与 `interrupt_conversation(...)` 的非等价暂停语义。后续仍需继续补齐这些恢复入口与 workspace 文件状态连续性之间的关系，并区分“自然暂停”“立即中断后暂停”与“恢复后重连”的跨层行为差异
5. **Pause / Resume 后的文件系统连续性**：当前证据已经能够说明多层都支持“恢复会话 / 恢复连接 / 恢复 sandbox”，但尚不能直接证明 `working_dir` 与文件系统内容一定总是完整延续。后续若继续核验，需要更细地追 `DockerWorkspace` / `APIRemoteWorkspace` / `OpenHandsCloudWorkspace` / agent-server 在恢复前后对工作目录、挂载卷、runtime metadata 和持久化目录的处理细节

## 七、Evidence 摘要

| # | 结论 | Status | 来源文件 |
|---|---|---|---|
| 1 | SandboxService 替代 V0 Runtime，三种实现（Docker/Process/Remote） | Verified | `sandbox_service.py`, `config.py` |
| 2 | Docker 容器生命周期 start/resume/pause/delete + 卷清理 | Verified | `docker_sandbox_service.py` |
| 3 | Process 沙箱为独立 subprocess + 独立目录 | Verified | `process_sandbox_service.py` |
| 4 | `persistence_dir` 默认 ~/.openhands，events 存为 JSON | Verified | `config.py`, `conversation_paths.py`, `filesystem_event_service.py` |
| 5 | `SANDBOX_VOLUMES` 解析为 VolumeMount | Verified | `config.py`, `docker_sandbox_service.py` |
| 6 | Workspace 核心类定义位于独立 SDK 仓库的 `openhands-sdk` / `openhands-workspace` 包，不在当前已核验的 OpenHands 仓库源码范围内 | Verified | OpenHands 仓库当前核验范围搜索 + SDK `__init__.py` 导出 |
| 7 | `base_state.json` 未在当前核验范围内找到 | Unverified | OpenHands 仓库内已核验范围搜索 |
| 8 | V0 Runtime 概念已被 SandboxService 取代，架构重组而非术语迁移 | Observed | `config.py`, `sandbox_service.py` |
| 9 | `Workspace` 工厂按 `host` 在 `LocalWorkspace` 与 `RemoteWorkspace` 之间分派；`DockerWorkspace` / `APIRemoteWorkspace` 是 `RemoteWorkspace` 子类，并在各自入口中负责容器或 runtime 生命周期接入 | Verified | SDK `workspace.py` + `docker/workspace.py` + `remote_api/workspace.py` |
| 10 | app_server 启动链路会在 sandbox 就绪后，通过 `sandbox.exposed_urls` + `session_api_key` 构造 `AsyncRemoteWorkspace`，再完成 repo/setup/skills/conversation 启动 | Verified | `app_conversation_models.py`, `app_conversation_service.py`, `live_status_app_conversation_service.py`, `sandbox_models.py`, `sandbox_service.py` |
| 11 | app_server 启动入口具备明确的 sandbox 选择分支：显式 `sandbox_id` 直连既有 sandbox，`PAUSED` 时先 resume；未提供 `sandbox_id` 时优先复用当前用户运行中 sandbox，找不到才新建 | Verified | `app_conversation_models.py`, `live_status_app_conversation_service.py` |
| 12 | 子会话启动会在未显式指定 `sandbox_id` 时继承父会话 `sandbox_id`，以共享同一 workspace / environment | Verified | `app_conversation_models.py`, `live_status_app_conversation_service.py` |
| 13 | 已持久化 conversation 的读取 / send-message 路径会直接按持久化 `sandbox_id` 查找 sandbox；若 sandbox 为 `PAUSED`，当前接口要求显式 resume，而不是自动重建 | Verified | `app_conversation_router.py`, `live_status_app_conversation_service.py` |
| 14 | 当前本地 OpenHands 仓库明确依赖 `openhands.agent_server`，但在该 checkout 中未见其实现源码；现有证据只覆盖 import、启动命令与镜像引用，不能把 agent-server 内部机制写成本地已核验事实 | Verified | `openhands/app_server/config.py`, `openhands/app_server/sandbox/process_sandbox_spec_service.py`, `openhands/app_server/sandbox/remote_sandbox_spec_service.py`, `docker-compose.yml` |
| 15 | 当前本地 OpenHands 仓库可直接确认 trajectory 配置项、前端下载/拉取入口与 event 持久化，但尚未在本地仓库内定位到可直接指认的 replay engine 或 artifact traceability 实现 | Observed | `config.template.toml`, `frontend/src/utils/download-trajectory.ts`, `frontend/src/hooks/mutation/use-get-trajectory.ts`, `openhands/app_server/event/filesystem_event_service.py` |
| 16 | 当前前端 resume 流程会在恢复后主动失效 sandbox 与 VS Code URL 相关查询，以重新获取 runtime metadata / URL；这支持“恢复后连接信息可能需要刷新”，但不足以直接证明 `working_dir` 或文件系统状态连续 | Observed | `frontend/src/hooks/mutation/use-v1-resume-conversation.ts` |
| 17 | `software-agent-sdk` 本地源码确认 workspace 抽象分为 `openhands-sdk`（基础 local / remote 抽象）、`openhands-workspace`（Docker / Apptainer / Runtime API / Cloud backend）与 `openhands-agent-server`（独立服务端实现）三层，而不是单一仓库内的同层对象 | Verified | `openhands-sdk/openhands/sdk/workspace/`, `openhands-workspace/openhands/workspace/`, `openhands-agent-server/pyproject.toml` |
| 18 | `software-agent-sdk` 本地源码确认 conversation restore 是跨层显式能力：workspace 抽象暴露 `pause()` / `resume()`，backend 有专门测试，`LocalConversation` 接受 `persistence_dir`，agent-server 启动时还会扫描并恢复持久化 conversations | Observed | `tests/workspace/test_workspace_pause_resume.py`, `openhands-sdk/openhands/sdk/conversation/impl/local_conversation.py`, `tests/cross/test_conversation_restore_behavior.py`, `openhands-agent-server/openhands/agent_server/conversation_service.py` |
| 19 | agent-server 在 conversation 层显式区分 `pause` 与 `interrupt`：前者等待当前 LLM 调用完成，后者立即取消运行中的 `arun()`；两者都可恢复，但不能被视为同一种暂停语义 | Verified | `openhands-agent-server/openhands/agent_server/conversation_router.py`, `openhands-agent-server/openhands/agent_server/conversation_service.py` |
| 20 | `openhands-agent-server/AGENTS.md` 明确把 `/api/**` 下 REST 契约视为公开 API，因此与 pause / interrupt / resume 相关的接口差异应按兼容性和弃用窗口理解，而不是纯内部实现细节 | Verified | `openhands-agent-server/AGENTS.md` |
