# Environments Index

> 如果你是第一次进入本目录，先读 [`README.md`](./README.md) 建立方向感；如果你已经知道想找什么，但不知道在哪个文件，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `overview.md` 的整体理解主线。

## 目录结构

```text
05-environments/
├── evaluation-environments/
├── browser-environments/
├── code-execution-environments/
├── sandboxing-and-safety/
└── simulated-environments/
```

## 按问题找入口

### 我想找 workspace、依赖隔离、artifact、副作用边界或恢复路径

优先进入 `code-execution-environments/`，再按问题继续细分：

- 想看 workspace 在环境层中到底是什么、有哪些组成部分：读 [`code-execution-environments/workspace-structure.md`](./code-execution-environments/workspace-structure.md)
- 想看 workspace 从创建到销毁的阶段划分与设计取舍：读 [`code-execution-environments/workspace-lifecycle.md`](./code-execution-environments/workspace-lifecycle.md)
- 想看 checkpoint、回滚与恢复问题：读 [`code-execution-environments/workspace-checkpoint.md`](./code-execution-environments/workspace-checkpoint.md) 和 [`code-execution-environments/rollback-recovery-design-paths.md`](./code-execution-environments/rollback-recovery-design-paths.md)
- 想看 workspace 中对象与轨迹如何建模：读 [`code-execution-environments/traceability-object-model.md`](./code-execution-environments/traceability-object-model.md) 和 [`code-execution-environments/workspace-traceability.md`](./code-execution-environments/workspace-traceability.md)

### 我想找 permission、sandbox、隔离边界、安全组合或 autonomy/confirmation

优先进入 `sandboxing-and-safety/`，再按问题继续细分：

- 想看权限策略与授权模型：读 [`sandboxing-and-safety/permission-policy.md`](./sandboxing-and-safety/permission-policy.md)
- 想看 permission 与 execution boundary 的关系：读 [`sandboxing-and-safety/permission-vs-execution-boundary.md`](./sandboxing-and-safety/permission-vs-execution-boundary.md)
- 想看 sandbox 分层与隔离层次：读 [`sandboxing-and-safety/sandbox-layers.md`](./sandboxing-and-safety/sandbox-layers.md)
- 想看最小安全组合与多层防线如何拼接：读 [`sandboxing-and-safety/safety-composition.md`](./sandboxing-and-safety/safety-composition.md)
- 想看 autonomy 与 confirmation 的边界：读 [`sandboxing-and-safety/autonomy-vs-confirmation.md`](./sandboxing-and-safety/autonomy-vs-confirmation.md)

### 我想找 browser environments

优先进入 [`browser-environments/README.md`](./browser-environments/README.md)。

适合这里的问题包括：网页交互环境、页面状态、浏览器任务中的会话边界，以及浏览器作为 Agent 执行环境时的特殊约束。

### 我想找 simulated environments

优先进入 [`simulated-environments/README.md`](./simulated-environments/README.md)。

适合这里的问题包括：受控任务世界、实验环境、仿真环境中的行为约束，以及仿真环境作为 Agent 训练或验证载体时的设计差异。

### 我想找 evaluation environments 或环境如何影响 benchmark

优先进入 [`evaluation-environments/README.md`](./evaluation-environments/README.md)。

适合这里的问题包括：环境如何影响 benchmark 的可信度、复现基座、评测环境约束，以及环境层与评测设计的耦合关系。

## 按阅读目标找入口

- 想先建立整体理解框架：读 [`overview.md`](./overview.md)
- 想看本目录当前还有哪些内容缺口：读 [`backlog.md`](./backlog.md)
- 想看接下来值得研究哪些对象或材料：读 [`candidates.md`](./candidates.md)
- 想看当前已记录的冲突、口径分歧或版本不一致：读 [`conflict.md`](./conflict.md)

## 按子目录定位

- `code-execution-environments/`：workspace、依赖、artifact、副作用边界、恢复与审计
- `sandboxing-and-safety/`：permission、execution isolation、governance、最小安全组合
- `browser-environments/`：网页交互环境、会话状态与页面级安全边界
- `simulated-environments/`：受控任务世界与实验环境
- `evaluation-environments/`：环境对 benchmark、复现与验证的影响

## 当你不确定问题属于哪个子目录

- 如果问题和**代码执行、workspace、依赖、checkpoint、回滚、traceability** 更相关，先看 `code-execution-environments/`
- 如果问题和**权限、确认机制、沙箱隔离、安全边界、分层防线** 更相关，先看 `sandboxing-and-safety/`
- 如果问题和**网页操作、浏览器会话、页面状态** 更相关，先看 `browser-environments/`
- 如果问题和**仿真任务世界、受控实验环境** 更相关，先看 `simulated-environments/`
- 如果问题和**环境如何影响评测、benchmark 或复现** 更相关，先看 `evaluation-environments/`

## 与 README / overview 的边界

- [`README.md`](./README.md) 回答“`05-environments/` 这个目录是什么、为什么存在、和相邻目录怎么区分”
- [`overview.md`](./overview.md) 回答“如果只读一篇，这个主题最值得先理解什么”
- `index.md` 回答“这个目录下面有什么，以及我知道想找什么时该去哪”
