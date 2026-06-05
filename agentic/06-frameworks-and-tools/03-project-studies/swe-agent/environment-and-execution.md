# SWE-agent Environment and Execution

> 本文档是 SWE-agent 作为 OpenHands 对照案例的第一层机制专题
> Evidence 状态：以 `Observed`（源码 / 官方文档可观察）为主；跨项目泛化结论标为 `Inferred`；尚未核验 SWE-ReX 内部实现的问题标为 `Unverified`
> 上游源码：`https://github.com/SWE-agent/SWE-agent`；相关运行时项目：`https://github.com/SWE-agent/SWE-ReX`

## 一、定位

SWE-agent 是面向软件工程任务的 coding agent。与 OpenHands 的 app server / sandbox service / SDK workspace / agent-server 多层服务化结构不同，SWE-agent 当前可观察路径更像 **CLI / config 驱动的一次任务执行系统**：用户通过 `sweagent run` 或 `sweagent run-batch` 提供 problem statement、repo、agent config 和 environment config，系统启动 deployment / runtime，把 repo 放入环境，执行 agent loop，并把 trajectory / patch / logs 写入输出目录。

因此，本文件只分析 SWE-agent 中与 environment / deployment / repository / trajectory / batch execution 相关的路径，不覆盖模型策略、prompt 模板、工具 bundle 设计或 SWE-bench 评测指标。

需要注意，SWE-agent 官方资料已明确推荐优先使用 `mini-swe-agent`，并说明当前大部分开发精力转向 mini-swe-agent。这里继续研究 SWE-agent，不是因为它代表最新推荐入口，而是因为它提供了更完整的 SWE-ReX deployment / runtime、stateful shell session、repo reset、trajectory 与 batch execution 对照样本。

## 二、核心执行链路

```text
sweagent run / run-batch
  → RunSingleConfig / RunBatchConfig
  → EnvironmentConfig
    → deployment: Docker / Local / Dummy / other SWE-ReX deployment
    → repo: GitHub / local / preexisting / SWE-Smith repo config
  → SWEEnv.from_config()
    → get_deployment(config.deployment)
    → SWEEnv.start()
      → deployment.start()
      → runtime.create_session()
      → repo.copy(deployment)
      → repo reset commands
      → post_startup_commands
  → Agent.run(env=SWEEnv)
  → trajectory / patch / logs / predictions
```

关键差异在于：SWE-agent 自身的 `SWEEnv` 不直接实现 Docker runtime 的底层细节，而是通过 `swerex.deployment` 与 `swerex.runtime` 抽象执行命令、读写文件、上传 repo 和管理 session。换句话说，SWE-agent 的环境层是 **agent task environment wrapper**，底层 deployment / runtime 主要下沉到 SWE-ReX。

SWEEnv 在 SWE-ReX 之上承担了完整的 environment lifecycle 编排角色，而非简单的薄封装。其独立逻辑包括：repo copy / reset 生命周期管理、`post_startup_commands` 执行、hooks 系统以及 `communicate()` 的错误检查与 auto-close 行为。SWE-agent 仅在 deployment 启动、runtime session 创建、命令执行和文件操作这四个底层操作上依赖 SWE-ReX，上层编排全部由 SWEEnv 自行承担。

官方架构资料曾把 SWEEnv 描述为 SWE-ReX 的 thin wrapper；源码核验表明这个描述夸大了 SWE-ReX 的占比，低估了 SWEEnv 的独立编排职责（约 150 行独立逻辑 vs 约 30 行 SWE-ReX 直接调用）。

## 三、EnvironmentConfig 与 SWEEnv

`EnvironmentConfig` 的核心字段包括：

- `deployment`：默认是 `DockerDeploymentConfig(image="python:3.11", python_standalone_dir="/root")`。
- `repo`：可选的 repository 配置。
- `post_startup_commands`：环境启动后、agent loop 前执行的命令。
- `post_startup_command_timeout`：每条 startup command 的超时。
- `name`：环境名称。

`SWEEnv.start()` 的顺序是：

1. `_init_deployment()`：启动 deployment，创建 bash session，设置基础环境变量。
2. `reset()`：切到 `/`，复制 repo，执行 repo reset，触发环境启动 hook。
3. 执行 `post_startup_commands`。

这说明 SWE-agent 的 environment 生命周期偏“一次任务 / 一次 attempt”的执行容器组织，而不是像 OpenHands 那样围绕 conversation / sandbox identity 做长期可恢复会话管理。

SWE-ReX 官方资料进一步说明，它支持本地 Docker、Modal、AWS Fargate 等 deployment，也有本地 runtime / shell session / close session 等 API 说明。但这些在线资料只能证明官方支持范围和接口语义，不能直接证明不同 deployment 的资源回收、隔离强度或高并发表现。

| 结论 | Evidence 状态 |
|---|---|
| `EnvironmentConfig` 默认使用 Docker deployment | Observed |
| `SWEEnv` 通过 SWE-ReX deployment / runtime 执行命令、读写文件和创建 session | Observed |
| SWE-agent 1.0 官方资料把 `SWEEnv` 描述为 SWE-ReX 的 thin wrapper | Observed |
| SWE-ReX 官方资料说明支持本地 Docker、Modal、AWS Fargate 等 deployment | Observed |
| `SWEEnv.start()` 会启动 deployment、创建 bash session、复制并 reset repo | Observed |
| `SWEEnv.close()` 调用 `deployment.stop()`，更接近任务结束后的环境关闭 | Observed |
| 底层 Docker / local / cloud deployment 的 cleanup、resource lifecycle 与性能边界仍需 SWE-ReX 源码或实验核验 | Unverified |

## 四、Repository copy / reset 模型

SWE-agent 把 repository 作为 environment 的一部分显式配置。当前可观察的 repo 类型包括（`reset=False` 时不执行 reset commands，残留修改状态可能被保留）:

- `GithubRepoConfig`：从 GitHub clone 到 sandbox 中，目录名形如 `org__repo`。
- `LocalRepoConfig`：要求本地 git repo 干净，然后上传到 deployment 的根目录。
- `PreExistingRepoConfig`：假设 repo 已经存在于 deployment 根目录。
- `SWESmithRepoConfig`：面向 SWE-Smith 的特殊预置路径。

repo reset 使用一组 git 命令：`git fetch`、`git status`、`git restore .`、`git reset --hard`、`git checkout <base_commit>`、`git clean -fdq`。其中 `git status` 主要用于诊断日志，不改变 reset 核心效果。

这与 OpenHands 的 workspace 连续性问题形成对照：SWE-agent 更明确地把每个任务实例的 repo 状态重置到 base commit，而不是强调 conversation reopen 后继续连接同一 workspace。它更适合作为 **per-instance repo reset / per-attempt execution environment** 的案例，而不是 conversation-level long-running workspace 的案例。

| 结论 | Evidence 状态 |
|---|---|
| repo 是 `EnvironmentConfig` 的显式组成部分 | Observed |
| GitHub repo 通过 runtime command 在 sandbox 内执行 shallow fetch / checkout | Observed |
| local repo 上传前要求本地 git repo 非 dirty | Observed |
| reset 依赖 Git 命令恢复 repo 文件状态 | Observed |
| 这种 reset 能否覆盖非文件状态、外部服务状态或进程状态 | Unverified |

## 五、RunSingle 与 RunBatch

### 5.1 `sweagent run`

`RunSingle` 的执行路径是：

1. 解析 `RunSingleConfig`。
2. 构造 `SWEEnv` 与 agent。
3. `env.start()`。
4. `agent.run(problem_statement, env, output_dir)`。
5. 保存 predictions。
6. `env.close()`。

默认输出目录位于当前工作目录的 `trajectories/<user>/<config>__<model>___<problem_id>`。

### 5.2 `sweagent run-batch`

`RunBatchConfig` 支持：

- `instances`：批量任务来源。
- `num_workers`：并行 worker 数。
- `random_delay_multiplier`：启动前随机延迟，用于缓解多 worker 同时启动时的资源压力。
- `redo_existing`：是否跳过已有 trajectory。
- `output_dir`：批量输出目录。

当前已从源码确认以下事实：`num_workers` 默认值为 `1`（`RunBatchConfig` Pydantic 默认值）；batch 并发模型是 `ThreadPoolExecutor(max_workers=num_workers)` 多线程；`random_delay_multiplier` 默认值为 `0.3`，每个 worker 启动前等待 `random.random() * num_workers * random_delay_multiplier` 秒，用于缓解少 CPU 平台无法及时启动所有容器时的瓶颈。`num_workers` 与 `random_delay_multiplier` 均已同时在源码和官方 `docs/usage/batch_mode.md` 中可观察，不应再写成"官方文档未突出说明"。

在线官方资料还提供了另一个并发约束：Claude API key 的 cache breakpoint 限制使每个 key 只能并行运行两个 SWE-agent 实例，可通过 `:::` 拼接多个 key 扩展并行度。多个 API key 传入时，每个 worker thread 在运行期间固定使用一个 API key。这个限制属于 LLM service / API quota 维度，不应与 Docker container startup 成本混为一谈。

| 结论 | Evidence 状态 |
|---|---|
| `run` 是单实例执行入口 | Observed |
| `run-batch` 支持多实例并行执行 | Observed |
| `num_workers` 默认值为 `1` | Observed |
| batch 并发模型是 `ThreadPoolExecutor(max_workers=num_workers)` 多线程 | Observed |
| `num_workers` 控制 batch 并发 worker 数，官方 batch 文档和本地源码均可观察 | Observed |
| `random_delay_multiplier` 默认 0.3，用于缓解并发启动时的资源压力；源码和官方文档均可观察 | Observed |
| Claude API key 数量会限制 batch 并发扩展，每个 key 可并行实例数有限 | Observed |
| Docker backend 并发启动可能遇到 CPU / container startup 瓶颈，具体瓶颈排序需要实验 | Inferred / Unverified |


## 五-A、SWE-ReX Deployment / Cleanup 边界

SWE-ReX 提供 7 种 Deployment 实现：`DockerDeployment`、`LocalDeployment`、`ModalDeployment`、`FargateDeployment`、`RemoteDeployment`、`DaytonaDeployment`、`DummyDeployment`。其中 `RemoteDeployment` 仅连接到已有服务器，`DummyDeployment` 仅供测试。

DockerDeployment 是默认实现：
- **启动**：`subprocess.Popen` 运行 `docker run --rm -p <port>:8000 --name <name> <image> <cmd>`，然后 `_wait_until_alive` 轮询最长 180s。
- **停止**：`RemoteRuntime.close()` → `docker kill <container_name>`（10s 超时，最多 3 次重试，每次间隔 5s）→ 可选 `docker rmi <image>`。`--rm` 确保容器退出后自动删除。`AbstractDeployment.__del__` 在 GC 时尝试再次调用 `stop()`，但通过 `try/except Exception: pass` 吞异常，cleanup 失败对上层不可见。

不同 deployment 的 stop 语义差异较大：`FargateDeployment.stop()` 只停止 ECS task 而不清理 `start()` 中创建或依赖的集群、执行角色、安全组等 AWS 基础设施资源。

`LocalRuntime` 是实际执行引擎，管理多个命名 bash session（`pexpect.spawn`）。`run_in_session()` 向 session 写命令并解析输出；`execute()` 使用 `subprocess.run()` 创建独立子进程。

| 结论 | Evidence 状态 |
|---|---|
| SWE-ReX 提供 7 种 deployment 实现 | Observed |
| DockerDeployment 使用 `docker run --rm` + 轮询 alive | Observed |
| `AbstractDeployment.__del__` 调用 stop() 但吞异常 | Observed |
| FargateDeployment.stop() 只停 ECS task，不清理全部 AWS 基础设施 | Observed |
| LocalRuntime 管理多个 bash session，`run_in_session()` 向 session 写命令并解析输出 | Observed |
| 不同 deployment 在高并发下的资源回收完整性和性能表现 | Unverified |

## 六、Trajectory 与输出文件

官方文档说明，`trajectories/` 是实验结果默认位置。每个实例的主要输出是 `<instance_id>.traj`，其中包含 agent 每一步的：

- response
- thought
- action
- observation
- state
- query

同一实例还会生成：

- `config.yaml`
- `*.log`

batch run 还会生成：

- `run_batch.config.yaml`
- `preds.json`
- `run_batch.*.log`
- `run_batch_exit_statuses.yaml`

这说明 SWE-agent 在 traceability 上有明确的 trajectory 产物，但当前证据只能证明它记录执行轨迹和可复现实验配置，不能直接证明 trajectory 可以重建完整 workspace 文件系统状态。

在线官方资料和源码共同确认了 `state` 字段边界：trajectory 示例中的 `state` 主要包含 `open_file` 与 `working_dir`，其用途是让 prompt template 感知当前工作目录和打开文件；它不是 workspace 文件系统快照，也不包含完整文件内容。`state` 由 tool bundle 的 `state_command` 负责生成，默认轻量，但可通过工具配置扩展更多字段（这不等于完整文件系统快照）。`run-replay` 的官方语义是从 trajectory 或 demo 中取出 actions，并在环境中重新执行这些 actions；这更接近 action replay，而不是 checkpoint resume 或 snapshot recovery。

| 结论 | Evidence 状态 |
|---|---|
| SWE-agent 明确输出 trajectory 文件 | Observed |
| trajectory 记录 thought / action / observation / state / query | Observed |
| 官方 trajectory 示例显示 `state` 主要包含 `open_file` 与 `working_dir` | Observed |
| `run-replay` 是重新执行 trajectory / demo 中的 actions，不是恢复完整 workspace 文件系统 | Observed |
| run-batch 生成 batch config、predictions、logs 和 exit statuses | Observed |
| trajectory 可以作为 demonstration / replay / debugging 来源 | Observed |
| trajectory 不足以证明完整 workspace 文件系统恢复能力 | Unverified |

## 七、mini-SWE-agent 对照

SWE-agent 官方 Getting Started 与 mini-SWE-agent FAQ 明确建议优先使用 mini-swe-agent，并说明当前大部分开发精力已转向 mini-swe-agent。这个信息会改变本案例的定位：SWE-agent 仍然适合作为 SWE-ReX / stateful shell session / repo reset / trajectory / run-batch 的机制样本，但后续如果研究“当前推荐的轻量 coding agent runtime”，mini-swe-agent 可能是更贴近上游演进方向的对象。

当前官方资料给出的关键差异包括：mini-swe-agent 只使用 bash，不使用 SWE-agent 的专用 tools；历史更线性；每个 action 通过独立命令执行，不保留 SWE-agent 这类 stateful shell session；环境后端更轻量，官方列出 local、docker、singularity / apptainer、swerex_docker、swerex_modal、bubblewrap、contree 等选项。

| 结论 | Evidence 状态 |
|---|---|
| 官方明确推荐优先使用 mini-swe-agent，且当前开发重心已转向 mini-swe-agent | Observed |
| SWE-agent 仍适合研究 SWE-ReX deployment / runtime integration、stateful shell session、repo reset、trajectory 与 run-batch | Inferred |
| mini-swe-agent 更适合作为后续轻量 runtime / environment backend 对照对象 | Inferred |
| mini-swe-agent 的具体 environment 后端实现仍需另开案例或源码核验 | Unverified |

## 八、与 OpenHands 的关键对照

| 维度 | OpenHands | SWE-agent |
|---|---|---|
| 入口形态 | app server / conversation / sandbox service | CLI / config / run or run-batch |
| 环境生命周期 | conversation 与 sandbox identity 强相关，支持 reopen / resume 路径 | task / instance 执行路径更明显，`RunSingle` 结束后关闭 env |
| workspace / repo | workspace 抽象分散在 SDK / workspace / agent-server | repo 是 environment config 的显式组成部分，reset 依赖 Git 命令 |
| runtime 底层 | Docker / Process / Remote sandbox service + SDK workspace backend | SWE-ReX deployment / runtime |
| 恢复语义 | conversation restore / sandbox resume / agent-server reconnect 较突出 | trajectory / config / git reset 较突出，长期 conversation restore 不明显 |
| 并发成本 | sandbox start / resume、workspace bridge、event persistence 等多层编排风险 | run-batch 多 worker + Docker startup 明确存在资源压力提醒 |

这个对照支持一个更稳妥的环境层判断：不同 coding agent 系统可能都使用 Docker / sandbox / trajectory 等词，但它们的生命周期锚点不同。OpenHands 更偏 conversation / sandbox identity；SWE-agent 更偏 instance / repo / trajectory / batch worker。

## 九、未验证 / 待继续补证的问题

以下问题当前不能写成定论：

- SWE-ReX 的 deployment / cleanup 边界已在 Section 五-A 记录；不同 deployment 在高并发或异常路径下的资源回收完整性仍需实验验证。
- SWE-agent 是否存在类似 OpenHands conversation resume 的长期会话恢复语义；当前没有看到官方把它作为长期会话恢复系统描述。
- trajectory 中的 `state` 字段默认主要覆盖打开文件和工作目录；state 由 tool bundle 的 `state_command` 生成，可通过工具配置扩展更多字段（但不等于完整文件系统快照）。
- batch 并发启动在真实负载下的瓶颈排序：CPU、Docker daemon、镜像拉取、repo reset、LLM API 还是文件 I/O。
- `random_delay_multiplier` 的调度行为和默认值（0.3）已确认；与 Docker startup 瓶颈的真实耦合关系仍需实验验证。
- Git reset 能否作为可靠 recovery 机制覆盖所有任务副作用；当前只能证明它覆盖 repo 文件状态的一部分。
- mini-swe-agent 暂不另开案例，保留为 SWE-agent 小节中的后续对照方向。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: `https://github.com/SWE-agent/SWE-agent` 的 `README.md`、`docs/usage/hello_world.md`、`docs/usage/batch_mode.md`、`docs/usage/trajectories.md`、`docs/config/environments.md`、`sweagent/environment/swe_env.py`、`sweagent/environment/repo.py`、`sweagent/run/run_single.py`、`sweagent/run/run_batch.py`、`sweagent/run/batch_instances.py`、`sweagent/run/run_replay.py`；`https://github.com/SWE-agent/SWE-ReX` 的 `swerex/deployment/`、`swerex/runtime/`、`swerex/server.py`、`swerex/exceptions.py`、`swerex/utils/wait.py`；在线补证来自 `agentic/temp/web-search/7.md` 与 `agentic/temp/web-search/8.md`。
- Trace: 从 `agentic/05-environments/candidates.md` 的 SWE-agent 候选对象出发，先建立与 OpenHands 对照的 environment / execution 机制专题；当前只回流对象目录，不回填 `05-environments` 主干。
- Needs: 继续核验 SWE-ReX 源码；核验 trajectory state 是否可被工具配置扩展；补 run-batch worker 调度、API key 分配与实例隔离；必要时为 mini-swe-agent 另建轻量 runtime / environment 对照案例。
