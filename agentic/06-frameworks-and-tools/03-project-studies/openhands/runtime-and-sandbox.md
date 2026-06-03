# OpenHands Runtime and Sandbox

> 阶段：待源码核验
> 当前状态：官方资料阶段已复核；本文件继续作为源码阶段占位，不承接稳定结论

本文件保留给 OpenHands 第二阶段源码研究，用于集中分析 runtime、sandbox、workspace、conversation / session 的代码层关系。

当前不根据 `agentic/temp/web-search/6.md` 或官方资料阶段的拼接推断直接写入稳定结论，原因是第一阶段复核后仍存在以下边界：

- V0 runtime 页面与 V1 workspace / sandbox / agent-server 页面处于不同架构口径下。
- 官方文档能够直接支撑 workspace 抽象、DockerWorkspace context manager、conversation persistence 等局部事实，但仍不足以给出完整生命周期模型。
- workspace 文件系统状态范围、pause / resume、checkpoint、rollback、snapshot、event replay 与 artifact traceability 等问题仍未被官方原文充分回答。

源码阶段开始后，优先补充：

- runtime / sandbox / workspace / conversation 的核心模块和类图。
- DockerWorkspace / RemoteWorkspace / LocalWorkspace 的创建、销毁、复用与清理逻辑。
- container mount、host path 映射与 `SANDBOX_VOLUMES` 的真实实现。
- `persistence_dir` / `OH_PERSISTENCE_DIR` 的实际状态范围。
- event replay、trajectory、artifact attribution 与 traceability 机制。
- 与 `agentic/05-environments/code-execution-environments/workspace-lifecycle.md` 可安全回填的 Evidence Claim。
