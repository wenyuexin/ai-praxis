# SWE-agent

SWE-agent 是一个面向软件工程任务的开源 coding agent。本目录用于把 SWE-agent 作为 OpenHands 之后的对照案例，重点观察它如何组织代码执行环境、repository setup、trajectory 输出、batch 并发与 sandbox / runtime 边界。

## 研究边界

本目录优先回答：

- SWE-agent 如何把 issue / problem statement、repo、agent、environment 组合成一次代码任务执行。
- SWE-agent 的 environment 与 deployment / runtime 如何分层，哪些能力交给 SWE-ReX。
- 它如何处理 repository copy / reset、trajectory 输出和 batch parallel execution。
- trajectory / replay 与 workspace 文件系统恢复之间的边界。
- 它与 OpenHands、mini-swe-agent 在 runtime / sandbox / workspace 组织方式上的差异。

不在本目录泛化为所有 coding agent 的通用结论；环境机制的主线归纳只在证据稳定时少量回填到 `../../../../05-environments/` 下对应专题。

## 当前阶段

- **阶段一：入口与环境边界建档**：已完成。已基于本地 SWE-agent 仓库和 SWE-ReX 仓库的源码核验，建立 environment / deployment / repo / batch / trajectory / cleanup 边界。
- **后续补证**：SWE-ReX 不同 deployment 在高并发下的资源回收完整性与性能表现仍需实验验证；trajectory state 的可扩展边界仍需样例确认；mini-swe-agent 暂不另开 case-study，保留为对照方向。

## 目录结构

```text
swe-agent/
├── README.md
└── environment-and-execution.md
```

## 阅读入口

- **`environment-and-execution.md`**（推荐优先阅读）：机制专题 / 初步源码核验。记录 SWE-agent 的 environment config、SWEEnv 生命周期、SWE-ReX 分层、repo copy/reset、run / run-batch、trajectory / replay、mini-swe-agent 对照和当前证据边界。

## 与 OpenHands 的对照价值

OpenHands 当前案例更偏 app server / sandbox service / SDK workspace / agent-server 的多层服务化结构；SWE-agent 当前可观察路径更偏 CLI / config 驱动的一次任务执行，并把底层 deployment / runtime 交给 SWE-ReX。这个差异适合用来校正 `05-environments` 中关于 workspace lifecycle、sandbox start 成本、batch 并发和 trajectory 可追溯性的通用表述。

同时，官方资料已明确推荐优先使用 mini-swe-agent，且当前开发重心已转向 mini-swe-agent。因此 SWE-agent 当前更适合作为 SWE-ReX 与 stateful shell session 的历史 / 机制样本；后续若继续比较轻量 runtime / environment backend，应考虑单独研究 mini-swe-agent。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources: `https://github.com/SWE-agent/SWE-agent`（README、官方 docs、`sweagent/environment/`、`sweagent/run/`）；`https://github.com/SWE-agent/SWE-ReX`（`swerex/deployment/`、`swerex/runtime/`、`swerex/server.py`、`swerex/exceptions.py`）；`agentic/temp/web-search/7.md` 与 `agentic/temp/web-search/8.md` 的在线官方资料补证。
- Trace: 从 `05-environments/candidates.md` 中的 SWE-agent 候选对象出发，先建立对象内聚的 case-study 入口，再决定是否回填环境层专题。
- Needs: SWE-ReX 不同 deployment 在高并发下的资源回收完整性核验；trajectory state 的工具配置扩展边界仍需样例验证；batch 真实负载下的 Docker / CPU / file I/O 瓶颈排序；mini-swe-agent 暂不另开案例。
