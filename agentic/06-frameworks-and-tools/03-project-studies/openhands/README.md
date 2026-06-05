# OpenHands

OpenHands 是一个开源 coding agent / software agent platform。本目录用于集中沉淀 OpenHands 的项目级研究，提供完整阅读体验；其他专题目录只按需要克制回填与其机制直接相关的证据。

## 研究边界

本目录优先回答：

- OpenHands 的项目定位、架构分层与 V0 / V1 演进。
- runtime、sandbox、workspace、conversation / session、agent loop 等核心对象之间的关系。
- 官方文档能直接支撑哪些结论，哪些问题必须进入源码阶段核验。
- OpenHands 对 agentic 工程系统的可迁移设计启发。

不在本目录泛化为所有 coding agent 的通用结论；跨项目比较应进入 `../05-comparisons/`，环境机制的主线归纳应回填到 `../../../../05-environments/` 下对应专题。

## 当前阶段

- **阶段一：官方资料建档**：已完成，整理了官方资料索引、官方口径下的 runtime / sandbox / workspace 关系和待源码核验问题。
- **阶段二：源码研究**：`openhands/app_server/` 层源码核验已完成，SDK / workspace / agent-server 三层已做一轮交叉核验，重点结论已写入运行时与沙箱专题，并提炼为面向读者的架构总览；跨层恢复链路、replay / artifact traceability 与文件系统连续性仍待继续补证。

## 目录结构

```text
openhands/
├── README.md
├── architecture.md                # 读者正文：环境/运行时/沙箱架构总览
├── scheduling-performance.md       # 机制专题：runtime/sandbox/workspace 调度性能风险地图
├── official-docs.md               # 官方资料阶段归档与证据地图
└── runtime-and-sandbox.md         # 机制专题/源码核验：环境相关源码证据表
```

## 阅读入口

推荐阅读顺序：先读 `architecture.md` 建立整体架构图；需要核验证据时进入 `runtime-and-sandbox.md`；关心调度性能时进入 `scheduling-performance.md`；需要追溯官方资料来源时再看 `official-docs.md`。

- **`architecture.md`**（推荐优先阅读）：面向读者的案例架构总览。回答 OpenHands 在 environment / runtime / sandbox 方面解决什么问题、主要组件关系是什么、哪些结论来自官方文档 / 源码观察 / 推断。
- `runtime-and-sandbox.md`：机制专题 / 源码核验型文档。基于 `https://github.com/OpenHands/OpenHands` 中 `openhands/app_server/` 及 `https://github.com/OpenHands/software-agent-sdk` 中 SDK / workspace / agent-server 相关包的源码核验结果，集中记录 SandboxService、workspace 后端、持久化路径、卷挂载与当前证据边界。需要查具体 Claim 的源码证据时进入本文。
- `scheduling-performance.md`：机制专题 / 性能风险地图。聚焦 runtime / sandbox / workspace 调度层的 20+ 个源码可观察瓶颈，区分 `Observed`（源码直接证据）、`Inferred`（架构推断）和 `Unverified`（需运行实验）。不讨论模型推理速度或 Python 通用语言性能，也不把静态源码风险直接写成高并发定论。
- `official-docs.md`：OpenHands 官方资料阶段调研整理，包含资料清单、可用结论、风险点和源码阶段核验清单。作为上游资料归档，不作为当前案例主入口。

## 与专题目录的关系

- `agentic/05-environments/code-execution-environments/workspace-lifecycle.md`：只回填 OpenHands 与 workspace / runtime / sandbox 生命周期直接相关的证据。
- `agentic/05-environments/candidates.md`：保留 OpenHands 作为环境层补证对象的状态和产出链接。
- `agentic/02-single-agent/`、`agentic/03-multi-agent/`：仅在后续研究确实涉及 agent loop、tool use 或多 agent 机制时做少量证据引用。
