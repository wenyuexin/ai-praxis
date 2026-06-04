# OpenHands 调度性能风险

> 本文档聚焦 OpenHands 的 runtime / sandbox / workspace 调度层，整理可能影响执行环境编排效率的性能风险；不讨论模型推理速度或 Python 通用语言性能
> Evidence 状态：以 `Observed`（源码可观察）为主，`Inferred`（架构推断）为辅，`Unverified`（需运行实验）标识无法通过静态代码分析确认的问题
> 上游源码：`https://github.com/OpenHands/OpenHands`（`openhands/app_server/` 为主），`https://github.com/OpenHands/software-agent-sdk`（`openhands-sdk` / `openhands-workspace` / `openhands-agent-server`）

## 1. 分析范围

本文只分析 OpenHands 中 execution-environment orchestration 路径的潜在性能瓶颈：

- sandbox 生命周期调度：start / pause / resume / delete
- conversation restore / resume 路径
- app server → agent server → workspace backend 的 bridge
- event persistence（JSON 文件 / S3 / GCS）
- workspace 文件操作（上传 / 下载 / command execution）
- 并发控制、超时重试、清理回收
- 可观测性（metrics / logging / tracing / profiling）

不分析：模型推理延迟、Python 通用语言性能、前端渲染性能、工具调用语义性能。

## 2. 性能关键路径

### 2.1 sandbox 创建与启动路径

```text
request → sandbox service → start_sandbox()
  → pause_old_sandboxes() (optional cleanup)
  → fetch sandbox spec (DB)
  → generate API keys, find unused port (Docker: socket bind trial)
  → docker.containers.run() (sync, potentially 5-30s: image pull, container init)
  → container health check: _container_to_checked_sandbox_info()
    → httpx GET /api/conversations health (per-container)
  → wait_for_sandbox_running() polling loop (up to 120s, poll_interval=2s)
    → every 2s: get_sandbox() → check status → optionally HTTP health check
```

### 2.2 conversation 启动路径

```text
_start_app_conversation():
1. validate parent conversation (DB query)
2. _wait_for_sandbox_start: find running OR start new sandbox (up to 120s)
3. _seed_sandbox_profiles (HTTP to agent server)
4. get sandbox spec (DB query)
5. run_setup_scripts:
   a. git clone (up to 120s per repo)
   b. setup script (up to 600s)
   c. git hooks (workspace file ops)
   d. skills (HTTP to agent server)
6. build start conversation request:
   a. get_user_info (DB)
   b. setup secrets for git providers (auth calls)
   c. configure LLM and MCP
   d. register agents (CPU)
   e. load hooks from agent server (HTTP)
   f. load skills (HTTP)
7. POST /api/conversations to agent server (up to 120s)
8. save conversation info (DB write)
9. save event callbacks (DB writes)
10. _process_pending_messages (serial HTTP POSTs, 30s each)
```

### 2.3 conversation 恢复与重开路径

```text
frontend → GET /api/v1/app-conversations?ids= (fetch metadata)
  → read persisted sandbox_id
  → POST /api/v1/sandboxes/{sandbox_id}/resume (trigger resume)
    → Docker: container.unpause() / container.start() (sync)
    → Remote: POST /resume with runtime_id
  → wait_for_sandbox_running() polling (up to 120s)
  → WebSocket reconnect conversation_url
```

### 2.4 event 读取与检索路径

```text
GET /api/conversations/{id}/events/search?kind=&limit=&order=
  → get_conversation_path() (DB query per conversation)
  → _search_paths(): list all event file paths
    - Filesystem: glob.glob() (sync, offloaded to thread pool)
    - S3: list_objects_v2 without pagination guarantee
    - GCS: bucket.list_blobs() iterator
  → load ALL events into memory (run_in_executor per file)
    - asyncio.gather(*) — N simultaneous thread pool tasks
  → filter kind/timestamp in Python (after load)
  → sort all (O(n log n))
  → slice to limit (offset-based pagination on already-loaded list)
```

### 2.5 workspace 命令执行路径

```text
execute_command("git clone ...")
  → RemoteWorkspaceMixin._execute_command_generator()
    → POST /api/bash/start_bash_command
    → polling loop: GET /api/bash/bash_events/search
      → time.sleep(0.1) per iteration (sync, blocks calling thread)
      → up to timeout seconds (default 30s)
    → accumulate stdout/stderr/exit_code
```

### 2.6 清理与资源回收路径

```text
pause_old_sandboxes():
  → search sandboxes (paginated, fetches all)
  → for each sandbox: pause_sandbox()
    → Docker: container.pause() (sync)
  → except Exception: pass (silent failure)

delete_sandbox():
  → container.stop(timeout=10) (sync, up to 10s blocking)
  → container.remove() (sync)
  → volume.remove() (sync, if volume exists)
```

## 3. 源码可观察风险

### 3.1 sandbox 生命周期

| 风险 | 路径 | 源码证据 | 严重程度 |
|------|------|----------|----------|
| Docker SDK 完全同步阻塞 | 所有 sandbox CRUD 操作 | `docker_sandbox_service.py` 全部 Docker SDK 调用；源码注释第 86 行自我承认 "The Docker API does not currently support async operations" | 高 —— 每次容器操作都会阻塞调用线程；若直接处于 async 请求路径中，会放大事件循环延迟 |
| `wait_for_sandbox_running()` 轮询 | sandbox start / resume | `sandbox_service.py` L104-L125：`while time.time() - start <= timeout: await asyncio.sleep(2)` | 高 —— 每个 sandbox 启动占用一个 asyncio 任务最多 120s |
| `search_sandboxes()` 全量拉取无过滤 | sandbox 列表 | `docker_sandbox_service.py` L290：`docker_client.containers.list(all=True)` 拉取所有容器 → Python 侧过滤排序分页；`process_sandbox_service.py` 同样模式 | 中 —— 系统中有大量 sandbox 时随容器数线性退化 |
| `batch_get_sandboxes()` 无并发限制 | 批量 sandbox 查询 | `sandbox_service.py` L50-L57：`asyncio.gather()` 不加 semaphore，100 个 id 同时 100 个查询 | 中 —— 批量操作可能压垮下游 |
| `pause_old_sandboxes()` 静默吞异常 | 自动清理 | `sandbox_service.py` L220-L227：`except Exception: pass` | 中 —— 部分 sandbox 暂停失败不被知晓 |
| Docker SDK 端口查找串行/浪费 | `start_sandbox` | `docker_sandbox_service.py` 未暴露但 `DockerWorkspace.find_available_tcp_port()` 试 50 个端口，失败路径最多浪费 5s | 中 |

### 3.2 conversation 恢复与续接

| 风险 | 路径 | 源码证据 | 严重程度 |
|------|------|----------|----------|
| `_start_app_conversation` 深度串行管道 | conversation 启动 | `live_status_app_conversation_service.py` L244-L450：一个 async generator 中串行 11 大步骤，其中 setup script 等子步骤存在分钟级超时 | 高 —— 新 conversation 从请求到就绪的尾延迟可能很高 |
| `_process_pending_messages` 串行逐个发 | conversation 启动尾声 | 同文件 L1708-L1789：`for msg in pending_messages: httpx_client.post(... timeout=30)` | 中 —— 消息逐个发送，无批处理 |
| `_wait_for_sandbox_start` 嵌套等待 | 启动步骤 2 | 同文件 L716-L784：`wait_for_sandbox_running()` polling 120s + `_check_agent_server_alive()` | 高 —— start pipeline 的 120s 阻塞直接挂在用户启动路径上 |
| `_find_running_sandbox_for_user` 分页遍历 | 启动步骤 2 分支 | 同文件 L586-L631：`while True: search_sandboxes(limit=100) ... next_page_id` | 低 —— 多数用户只有 1-2 个运行中 sandbox |
| `AppConversationInfoLoadTasks` 不清理 | 全生命周期 | `event_service_base.py` 中 `app_conversation_info_load_tasks` dict 永久保留 task 引用 | 中 —— 长期运行服务中内存泄漏 |

### 3.3 workspace bridge

| 风险 | 路径 | 源码证据 | 严重程度 |
|------|------|----------|----------|
| `RemoteWorkspace` 使用同步 `httpx.Client` | 所有 workspace 操作 | `remote/base.py` L87-L106：`cached_property` 返回同步 Client；`_execute()` 使用 `.request()` 阻塞 | 高 —— 所有 workspace 操作阻塞调用线程 |
| `time.sleep(0.1)` 命令输出轮询 | command execution | `remote_workspace_mixin.py` L172：每个轮询迭代阻塞 100ms，直到远端返回退出状态或达到超时 | 高 —— 对短命令会引入轮询粒度级延迟；对长命令会长期占用调用线程 |
| `RemoteWorkspace.alive` 同步 `urlopen` | 健康检查 | `remote/base.py` L227-L240：使用同步 `urllib.request.urlopen(timeout=5)` | 中 —— 阻塞调用线程；若处于 async 请求路径中，会放大事件循环延迟 |
| `AsyncRemoteWorkspace.alive` 仍同步 | 异步路径中的健康检查 | `async_remote_workspace.py` L152-L165：同样使用同步 `urlopen` | 中 |
| `OpenHandsCloudWorkspace._send_api_request` 每次都新建 `httpx.Client` | Cloud workspace 所有请求 | `cloud/workspace.py` L478-L501：`with httpx.Client(timeout=timeout) as api_client:` | 高 —— 无连接池复用，每次请求建立新 TCP 连接 |
| 重试装饰器重复模式无共享配置 | `get_llm`/`get_mcp_config`/secret 等 | `remote/base.py`：`tenacity` 装饰器 `stop_after_attempt(3)` + `wait_exponential(1,5)` 重复出现在多个方法上 | 中 —— 每次操作最多额外 ~7s 退避延迟 |
| `_file_upload` / `_file_download` 无流式 | 文件传输 | `remote_workspace_mixin.py` L202-L316：整个文件读入内存 / 整个响应写入磁盘 | 中 —— 大文件 O(N) 内存 |

### 3.4 event 持久化

| 风险 | 路径 | 源码证据 | 严重程度 |
|------|------|----------|----------|
| `search_events` 无服务端过滤 | 所有事件读取 | `event_service_base.py` L75-L127：先 `_search_paths` 列出全部路径 → `asyncio.gather` 加载全部事件到内存 → Python 侧过滤 → 排序 → 分页 | 高 —— 长会话（5000+ events）每次查询加载全部事件到内存，无法仅拉取指定 kind 的最近 N 条 |
| `run_in_executor` 默认线程池无限制 | 所有事件 I/O | `event_service_base.py` 使用 `loop.run_in_executor(None, ...)` 使用默认 ThreadPoolExecutor（Python 3.12+ 最多 min(32, cpu+4)），多个 `search_events` 调用间共享且无隔离 | 中 —— 线程池可能被高并发事件读取打满 |
| 每个 event 独立文件 | event 存储 | `filesystem_event_service.py`：每个 event 单独 JSON 文件，path 包含用户 / conversation / event_id | 中 —— 目录数随 event 数线性增长，文件系统 inode 可能压力 |
| `_search_paths` 无存储层过滤 | 事件列表 | S3 `list_objects_v2` 无分页处理；GCS `bucket.list_blobs` 无 filter；Filesystem `glob.glob` 全量列表 | 中 |
| `count_events` 带 filter 时加载全部再计数 | 事件计数 | `event_service_base.py` L129-L153 | 中 |
| `AppConversationInfoLoadTasks` task 永久缓存 | event path 解析 | 同文件 cache task 不清理，长期运行内存泄漏 | 低 |

### 3.5 清理与资源泄漏

| 风险 | 路径 | 源码证据 | 严重程度 |
|------|------|----------|----------|
| `pause_old_sandboxes()` 吞异常 | 自动清理 | `sandbox_service.py` L220-L227：bare `except Exception: pass` | 高 —— 单 sandbox 暂停失败后整个暂停操作静默完成，调用者不知有沙箱遗留 |
| Docker SDK 同步 `container.stop(timeout=10)` | sandbox 删除 | `docker_sandbox_service.py` L538 | 中 —— 阻塞调用线程；是否阻塞事件循环取决于调用点是否处于 async 请求路径 |
| `_process_pending_messages` 无超时隔离 | conversation 启动 | 30s 超时，单个失败不影响其他，但无区分错误类型 | 低 |
| `DockerSandboxService` 无并发锁 | 多线程安全 | 全局无锁、无 semaphore，`_find_unused_port()` 等操作可能竞争 | 中 |
| `RemoteWorkspaceMixin._execute_command_generator` 超时后不 kill 远程进程 | command execution | `remote_workspace_mixin.py` 轮询超时返回 -1 但不 kill 远程命令 | 中 —— 远程服务器上的孤立 bash 进程 |
| `DockerWorkspace._stream_docker_logs` 无限线程 | workspace 生命周期 | `docker/workspace.py` daemon thread 不清理 | 低 |

### 3.6 可观测性与 metrics

| 风险 | 路径 | 源码证据 | 严重程度 |
|------|------|----------|----------|
| 无结构化 metrics | 全系统 | 当前已核验范围内未发现 Prometheus / OpenTelemetry / statsd 等 metrics 出口 | 高 —— 无法在生产环境中监控 sandbox lifecycle 延迟、conversation 启动时间、event 读延时等关键指标 |
| 无 profiling 入口 | 全系统 | 未发现 py-spy / cProfile / aiometer 等慢路径检测工具集成 | 高 —— 无法定位具体哪个子步骤是性能瓶颈 |
| 无显式慢路径日志 | sandbox start / event read / workspace cmd | 当前日志更多是 info/debug，未发现对 >N 秒操作的 warn 级日志 | 中 —— 无法通过日志定位慢在哪里 |
| `_check_agent_server_alive` 吞异常降级 | sandbox 健康检查 | `sandbox_service.py` L145-L150：anything 异常都只是 `log.debug("Agent server not alive")` | 中 —— 真实错误被覆盖 |
| 无 request-level tracing | 全系统 | 未发现 trace ID / span 等跨层请求追踪 | 中 —— 一个请求跨 app server → sandbox → agent server 时无法串联 |

## 4. 架构推断风险

以下风险基于架构观察推断，尚未通过运行实验验证：

| 风险 | 推断依据 | 推断理由 |
|------|----------|----------|
| 多 conversation 并发创建时 sandbox 启动竞争 | `DockerSandboxService` 无锁、`_find_unused_port()` 无原子分配 | 两个 concurrent `start_sandbox` 可能分配到同一端口导致启动失败 |
| 高并发 sandbox start 下 thread pool 压力 | `run_in_executor` 默认 pool 被所有 event I/O + sandbox 共用 | Docker API 调用 + event 读放在同一默认线程池中 |
| Event 目录深度导致 filesystem 压力 | 路径 `persistence_dir/user_id/v1_conversations/conversation_hex_id/event_id_hex.json` | 每个 conversation 数千事件，目录树压力上升；S3/GCS 无此问题 |
| 长 idle conversation 后 resume 延迟线性累积 | resume 路径包含 container unpause + `wait_for_sandbox_running` 120s + health check | 若多个 conversation 同时被大量事件唤醒，启动风暴可能叠加 |
| RemoteSandboxService 的远端 API 调用在高延迟网络中成为瓶颈 | 通过 HTTP start/resume/pause remote runtime | 未在源码中看到超时调整或重试策略之外的网络延迟防御 |

## 5. 尚不能定论的问题

以下问题在当前源码核验范围内**不能写成定论**：

- **Python 是性能瓶颈** —— 大部分延迟来自 Docker daemon 同步 API、HTTP 往返、线程池竞争和文件 IO，而非 Python 解释器本身。不应把上述问题归因为"Python 慢"。
- **OpenHands 在高并发下一定慢** —— 未经过压测，上述瓶颈在设计负载下的实际影响未知。有些问题（如 `asyncio.gather` 无限并发）只在特定并发量级下才显现。
- **事件存储方式是性能瓶颈的主因** —— 每个 event 独立文件的设计对会话数增长有可预见的影响，但当前仓库内未形成足够的性能测试证据来断言这是首要瓶颈。
- **`time.sleep(0.1)` 在 workspace command polling 中是实际瓶颈** —— 这在 `RemoteWorkspaceMixin` 中是同步命令执行机制，但 agent-server 和 workspace 之间是否真的受此影响取决于命令频次和运行场景。
- **清理路径（`pause_old_sandboxes`）的资源泄漏是生产问题** —— 静默吞异常的设计确实有风险，但未用真实负载验证实际泄漏率。

## 6. 需要补充的实验

需要运行实验或压测来验证的问题：

| 实验 | 目标 | 方法 |
|------|------|------|
| 多 conversation 并发启动压力 | 测量 sandbox 创建的平均延迟、P95、P99 和失败率 | 并发发送 10 / 50 / 100 个 start conversation 请求，观察吞吐、端口冲突、`wait_for_sandbox_running` 的回落分布 |
| Event 读取性能基准 | 测量 `search_events` 在不同 conversation 事件规模下的响应时间 | 构造 100 / 1000 / 5000 / 10000 events 的 conversation，测试 limit=100 和 limit=所有 的读取延迟 |
| Workspace command 轮询延迟 | 验证 `time.sleep(0.1)` 是否增加感知延迟 | 发送一个立即返回的命令，测量从发出到收到结果的总延迟 |
| Container 停止超时对清理的影响 | 测量 `container.stop(timeout=10)` 在大量容器下的累积耗时 | 顺序清理 50 个 DOCKER sandbox，统计总耗时 |
| `search_sandboxes()` 随容器数增长耗时曲线 | 验证 O(N) 扫全表的退化 | 构造 10 / 100 / 500 个 Docker 容器，测量 search API 响应时间 |
| 长时间运行服务的线程池竞争 | 确认 event I/O 和 sandbox 操作在同一默认线程池中是否会互相影响 | 在持续事件读取负载下创建 sandbox，对比隔离线程池的延迟 |

## Evidence

| Claim | Status | Sources | Notes |
|-------|--------|---------|-------|
| Docker SDK 同步阻塞所有 sandbox CRUD 操作 | Observed | `docker_sandbox_service.py` L86 自述 + 全模块所有 `docker_client.*` 调用 | 源码明确确认 |
| `wait_for_sandbox_running` polling loop 120s / 2s | Observed | `sandbox_service.py` L104-L125 | 默认 120s 超时，2s 间隔轮询 |
| `_start_app_conversation` 为 11 步深度串行 pipeline | Observed | `live_status_app_conversation_service.py` L244-L450 | 含 await/async 但逻辑上每步等待前一步完成 |
| `_process_pending_messages` 串行逐个处理 | Observed | 同文件 L1708-L1789 | `for msg in pending_messages: httpx.post()` |
| `search_events` 无服务端过滤，加载全部到内存 | Observed | `event_service_base.py` L75-L127 | 所有 event 文件被 gather 加载后过滤分页 |
| `RemoteWorkspaceMixin` 使用同步 `time.sleep(0.1)` 轮询命令输出 | Observed | `remote_workspace_mixin.py` L172 | 同步 sleep 阻塞调用线程；是否阻塞事件循环取决于调用点是否处于 async 路径 |
| `OpenHandsCloudWorkspace._send_api_request` 每次新建 httpx.Client | Observed | `cloud/workspace.py` L478-L501 | 无连接池复用 |
| 当前已核验范围内无 metrics / profiling / tracing 框架 | Observed | 全局搜索未发现 Prometheus/OpenTelemetry/cProfile/py-spy | 仅基于当前核验范围 |
| `pause_old_sandboxes()` 使用 bare `except Exception: pass` | Observed | `sandbox_service.py` L220-L227 | 单 sandbox 失败不通知调用者 |
| `container.stop(timeout=10)` 同步阻塞调用线程 | Observed | `docker_sandbox_service.py` L538 | Docker SDK 自身同步；是否阻塞事件循环取决于调用点是否处于 async 请求路径 |
| 多 sandbox 并发启动可能存在端口竞争 | Inferred | `DockerSandboxService` 无锁、`DockerWorkspace.find_available_tcp_port()` 非原子 | 需运行实验验证 |
| 长 conversation 的 event 读取高频场景下可能遇到 IO 瓶颈 | Inferred | `search_events` O(n) 加载 + 无存储层过滤 | 未压测，数量级依赖场景 |
| 长时间运行服务中 `app_conversation_info_load_tasks` 可能导致内存堆积 | Observed | `event_service_base.py` task dict 永久缓存 | 泄漏量正比于 conversation 数 |