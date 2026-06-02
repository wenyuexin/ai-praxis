# Code Execution Environments

> 适用范围：代码执行、测试运行与依赖隔离环境研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理 Agent 执行代码、运行测试、操作文件系统与管理依赖时所依赖的环境模型与隔离边界。

## 快速入口

- 结合上层 `../README.md` 理解代码执行环境在整体环境体系中的位置。
- 重点关注本地进程、subprocess、venv、container、remote runtime 等执行形态。
- 如出现版本口径、安全边界或运行时假设冲突，优先记录到 `../conflict.md`；仅临时判断再落到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：代码执行环境、依赖隔离、workspace、artifact、测试运行、生命周期管理。
- 不放在这里：通用安全策略与权限治理（放 `../sandboxing-and-safety/`）、浏览器自动化环境（放 `../browser-environments/`）。

## 同目录导航

- 当前主干专题：`workspace-structure.md`，用于区分 workspace、sandbox 与任务工作上下文。
- 当前主干专题：`workspace-checkpoint.md`，用于展开 snapshot、checkpoint、rollback 与恢复粒度问题。
- 当前主干专题：`workspace-traceability.md`，用于展开日志、轨迹、artifact 归因与审计边界。
- 当前主干专题：`traceability-object-model.md`，用于讨论 traceability 的最小必要对象集、对象关系与归因结构。
- 当前主干专题：`rollback-recovery-design-paths.md`，用于比较 checkpoint restore、overlay revert、replay 与逻辑重建等恢复路径。
- 相关上位主题：`../overview.md`，用于回到环境层的三层框架与最小安全组合。
- 如发现代码执行环境相关结论冲突，优先记录到 `../conflict.md`。
