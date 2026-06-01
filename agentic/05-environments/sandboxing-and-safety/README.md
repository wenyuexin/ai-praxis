# Sandboxing and Safety

> 适用范围：执行隔离、安全边界与权限治理研究
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于整理 Agent 执行环境中的沙箱机制、安全边界、权限控制、回滚审计与风险治理问题。

## 快速入口

- 结合上层 `../README.md` 理解 sandboxing-and-safety 在环境体系中的位置。
- 重点关注进程/容器/远程环境隔离、文件系统与网络权限、凭证管理、资源限制与回滚。
- 如出现安全模型、隔离假设或版本口径冲突，记录到 `../../temp/conflict.md`。

## 边界说明

- 放在这里：沙箱模型、权限控制、资源限制、回滚、审计、风险分类、安全治理。
- 不放在这里：具体代码执行环境形态（放 `../code-execution-environments/`）、浏览器任务环境（放 `../browser-environments/`）。

## 同目录导航

- 当前主干专题：`permission-policy.md`，用于区分权限策略与执行沙箱的边界。
- 当前主干专题：`sandbox-layers.md`，用于系统拆分 sandbox 的多层结构与作用边界。
- 相关专题：`autonomy-vs-confirmation.md`，用于展开默认自治与人工确认之间的环境层冲突。
- 如后续需要整体综述，使用 `overview.md`。
- 如需要记录内容缺口，使用 `backlog.md`。
- 如需要学习或建设顺序，使用 `roadmap.md`。
- 如发现目录范围内冲突，使用 `conflict.md`。
