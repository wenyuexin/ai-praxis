# Codex Agent 原理与机制

> 目标：解释 Codex Agent 如何形成“从理解到执行”的能力闭环。

## 1. 核心机制：三层模型

- 能力层：代码理解、生成、重构、推理、tool/function calling 决策。
- 编排层：任务分解、上下文注入、执行循环、失败恢复。
- 治理层：权限、审计、成本与安全策略。

只有三层都成立，才是“可用的代码 Agent”。

### 1.1 治理层详解

治理层是 Codex 区别于“补全工具”的关键工程能力，核心组件如下：

**权限模型（approval_policy 四级）**

| 级别 | 行为 | 风险 |
|------|------|------|
| untrusted（默认） | 所有操作需确认 | 最低 |
| on-failure | 仅在操作失败时需确认 | 中低 |
| on-request | 仅在特定类型操作时确认（如写文件、跑命令） | 中 |
| never | 完全自动，无需确认 | 最高 |

**沙箱隔离**

Codex 采用 OS 内核级沙箱（非应用层模拟），技术选型因平台而异：
- Linux：Landlock + Seccomp + bubblewrap
- macOS：Seatbelt
- Windows：ACL + WFP
- 隔离粒度：**按任务**（每个任务独立环境，结束后交还结果）
- ⚠️ 此描述适用于本地沙箱。Cloud 沙箱采用不同的隔离模型（远程容器），安全保证与本地不同。详见 `codex-cloud-sandbox.md` 和 `conflict.md`

**审计日志**
- 记录内容：执行命令、修改文件、决策过程
- Agent 状态机：7 种状态（PendingInit → Running → Interrupted → Completed → Errored → Shutdown → NotFound）
- 回滚原因（TurnAbortReason）：4 种（Interrupted / Replaced / ReviewEnded / BudgetLimited）。

## 2. 执行闭环最小条件

判断是否是 Agent，不看名称，看是否具备：

1. Plan：任务规划
2. Execute：执行动作（工具/命令/修改）
3. Check：结果校验
4. Retry/Rollback：失败重试或回滚

缺任一环节，通常只能算“辅助生成系统”。

## 3. Tool/Function Calling 的真实语义

- Function calling 是模型输出结构化调用意图。
- 执行动作由外部系统完成，不是模型直接执行。
- 因此超时、重试、幂等、回滚、审计都必须由执行器实现。

## 4. 上下文机制与“记忆”边界

Codex Agent 的上下文常由三部分构成：

- 会话历史（短期）
- 代码索引/RAG（检索）
- 系统规则（静态约束）

边界：

- 标称窗口不等于有效上下文
- 索引召回不等于全局理解
- 默认无跨会话强持久记忆

### 4.1 Token 效率设计

Codex 在上下文管理上的设计哲学是**任务拆分并行，多会话独立**，避免上下文污染。核心思路：

- 复杂任务拆分为多个独立子任务，分别在独立沙箱中执行
- 每个子任务使用独立的会话上下文，不互相污染
- 并行执行完成后由聚合器合并结果

这种设计的优势：
- **Token 效率高**：同任务消耗约 150 万 token（社区实测），远低于在单一会话中反复探索的做法
- **上下文干净**：每个子任务只看到自己的上下文，不会因累积信息导致推理退化
- **失败影响小**：一个子任务失败不影响其他子任务

代价：
- **长任务稳定性受限**：超过 50 轮交互的会话可能出现上下文丢失
- **跨文件关联**：需要全局理解的任务难以充分利用分散的上下文

相关数据见 `codex-agent-evidence.md` 的 Benchmark 表。

## 5. 关键 trade-off

- 速度 vs 推理深度
- 自治执行 vs 风险暴露
- 长上下文覆盖 vs 结果稳定性
- 自动化比例 vs 审查成本

## 6. 生产可用性四问

1. 闭环是否完整
2. 权限是否最小化且可审计
3. 上下文策略是否可解释
4. 失败是否可恢复

## 7. 工具扩展机制：MCP 与 Skills

Codex 通过两种互补机制扩展能力边界：**MCP（Model Context Protocol）** 连接外部数据/服务，**Skills** 封装工作流。

### 7.1 MCP（外部能力扩展）

| 概念 | 说明 |
|------|------|
| **本质** | 让 Codex 连接外部工具/服务的标准协议 |
| **Codex 角色** | 负责推理和决策 |
| **MCP Server 角色** | 负责提供外部能力（数据库、SaaS API、浏览器等） |
| **官方插件** | 90+（Jira、CircleCI、GitLab、Slack 等），2026.4.16 发布 |
| **社区生态** | 快速增长中，热门 Server 含 postgres、filesystem、github、brave-search 等 |

### 7.2 Skills（工作流封装）

| 维度 | Skills | MCP |
|------|-------|-----|
| **本质** | 流程/规则（按需触发的工作流） | 外部能力/数据（连接第三方服务） |
| **调用方式** | 用户主动触发，封装工作流 | 模型自动调用，连接外部服务 |
| **触发优先级** | Skills 优先（用户明确指令） | MCP 按需（模型判断） |


### 7.3 AGENTS.md

`AGENTS.md` 是 Codex 的项目级指令文件，存放在项目根目录，每次开工前自动读取。它提供项目维度的规则上下文，内容包含：
- 项目结构说明
- 常用命令
- 不可触碰的规则
- 验收标准

与 MCP/Skills 的区别：
- AGENTS.md 放"规则"（无条件执行）
- Skills 放"流程"（按需触发）
- MCP 放"能力"（外部连接）

## 8. 版本变更记录

- 2026-05-25：从原"原理与机制"文档收敛为 Agent 本体机制版。
- 2026-05-26：补充治理层详解（权限/沙箱/审计）、Token 效率对比、MCP/Skills 扩展机制。


## 参考来源

- https://platform.openai.com/docs/guides/function-calling
- https://openai.com/index/introducing-agents-sdk/
- https://openai.com/index/introducing-the-openai-cli/
- https://platform.openai.com/docs/models/gpt-4o
- https://platform.openai.com/docs/guides/rag

*最后更新: 2026-05-26*
