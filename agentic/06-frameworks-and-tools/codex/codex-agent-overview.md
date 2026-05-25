# Codex Agent 总览

> 目标：从“代码能力 Agent 本体”理解 Codex，而不是把它当作分散产品功能集合。

## 1. Codex Agent 是什么

在当前语境下，Codex 更准确的理解是：

- 以代码理解/生成/推理为核心的模型能力
- 叠加不同入口（Web/IDE/API/CLI）的执行编排
- 在权限、审计、成本、安全约束下运行的工程系统

一句话：**Codex Agent = 代码能力 + 执行闭环 + 工程治理**。

## 2. Codex Agent 不是什么

- 不是历史上的独立 Codex API（该 API 已弃用）
- 不是“会补全代码”的 IDE 插件同义词
- 不是天然具备自治交付能力的单模型

## 3. 为什么同名能力体验差异很大

核心原因不在模型本身，而在执行链路：

- Web 往往是“建议型”
- IDE 是“差异应用型”
- API 是“可编排型”
- CLI/Agent 工具是“执行循环型”

你看到的差异，本质是闭环能力差异。

## 4. 与其他工具的关系（简版）

- 与 Copilot：Codex 代码能力可作为底层能力之一，Copilot 更偏 IDE 生产力交付。
- 与 Claude Code：两者都能服务代码 Agent 场景，但在本地执行自治和治理路径上侧重点不同。

## 5. 阅读顺序

1. `codex-agent-overview.md`（本篇）
2. `codex-agent-mechanisms.md`（原理与边界）
3. `codex-agent-practice.md`（使用方法）
4. `codex-agent-evidence.md`（证据与核验）

## 6. 版本变更记录

- 2026-05-25：从多专题体系收敛为 Agent 本体导向主线。

*最后更新: 2026-05-25*
