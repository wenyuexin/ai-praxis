# Codex Agent 原理与机制

> 目标：解释 Codex Agent 如何形成“从理解到执行”的能力闭环。
>
> 本文默认会与 `overview.md` 在治理、环境、能力闭环等结论上存在适度重复；区别在于，`overview.md` 负责给出读者可快速吸收的整体判断，本文负责把这些判断拆成机制层结构、执行条件与证据边界。

## 1. 核心机制：三层模型

- 能力层：代码理解、生成、重构、推理、tool/function calling 决策。
- 编排层：任务分解、上下文注入、执行循环、失败恢复。
- 治理层：权限、审计、成本与安全策略。

只有三层都成立，才是“可用的代码 Agent”。

### 1.1 治理层详解

治理层是 Codex 区别于“补全工具”的关键工程能力，核心组件如下：

**权限模型（approval policy）**

官方 `agent-approvals-security` 页面当前可直接确认：Codex 的治理控制来自两层——**sandbox mode** 决定技术边界，**approval policy** 决定什么时候必须先征得用户同意。

| 级别 | 当前可保守理解的行为 | 风险 |
|------|----------------------|------|
| `untrusted` | 更偏严格确认；高风险动作通常需要先确认 | 最低 |
| `on-failure` | 更偏在失败或越界时触发确认 | 中低 |
| `on-request` | 更偏在特定类型操作或离开当前边界时确认 | 中 |
| `never` | 尽量不等待人工确认 | 最高 |

> 更稳妥的理解是：approval policy 控制“何时询问”，sandbox mode 控制“技术上能做什么”。二者相关，但不是同一个概念。

**沙箱与执行边界**

当前更稳的一手口径应分成本地与 Cloud 两类，而不是笼统写成一种“统一沙箱”模型：

- **Codex CLI / IDE extension / 本地执行路径**：官方文档明确是 **OS-level mechanisms** 在执行本地沙箱策略；默认无网络、默认只允许写活动 workspace。
- **Codex Cloud**：运行在 `isolated OpenAI-managed containers` 中，属于远程托管容器边界，不应与本地 OS 级沙箱混写。
- **Cloud 运行时模型**：采用 `setup → agent phase` 两阶段；Setup 可联网安装依赖，Agent 阶段默认离线，只有为环境显式开启 internet access 时才可联网。
- **Cloud Secrets**：仅在 Setup 阶段可用，Agent 阶段开始前移除。

因此，这里更适合把 Codex 的治理层写成：**本地受限执行边界 + Cloud 远程容器边界 + approval policy 协同控制**。

> ⚠️ `沙箱` 一词在 Codex 语境里至少覆盖两种不同实体：本地 OS 级沙箱，与 Cloud 远程容器隔离。两者安全属性和适用场景不同，交叉口径见 `codex-cloud-sandbox.md` 与 `conflict.md`。

**审计与可回溯性**
- 从官方治理口径看，重点不只是“能否执行”，还包括审批流、事件记录与异常操作回看。
- 现有资料可支撑“执行命令、修改文件、审批/越界事件应可审计”这一治理方向。
- Agent 状态机、回滚原因等更细内部状态名仍更接近仓库/实现层线索，不宜在没有逐项源码锚点时过度外推为统一产品规格。

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

可暂时把 Codex 的上下文管理理解为一种**偏任务拆分、偏会话隔离**的工程取向。当前较稳妥的理解是：

- 复杂任务往往会被拆成更小的执行单元
- 不同任务单元倾向于在相对独立的上下文中运行
- 最终结果需要再经过聚合或人工审查

可能的收益：
- **Token 使用更可控**：部分社区样本显示，拆分式执行有助于减少单会话反复探索带来的额外消耗
- **上下文更干净**：相互独立的任务单元更不容易累积无关噪声
- **失败影响可局部化**：单个子任务失败时，理论上更容易局部重试

已知代价或风险：
- **长任务稳定性仍有限**：关于“超过多少轮会话后明显衰减”的数字，目前更多来自社区观察，不宜写成硬阈值
- **跨文件 / 全局关联更难**：需要整体理解的大任务，未必能充分受益于拆分式上下文

相关数字与案例见 `codex-agent-evidence.md`，使用时应结合其 Evidence 分级理解，不宜把社区样本直接外推为稳定产品特性。

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
| **协议传输形态（仓库证据）** | 基于 `https://github.com/openai/codex` 仓库核验，MCP 配置类型明确是 `stdio`（本地进程）与 `streamable_http`（远程）。⚠️ 仓库内大量 SSE 代码更多属于响应流层，不能直接等同 MCP transport 分类 |
| **配置约束（仓库证据）** | `stdio` 与 `streamable_http` 字段互斥：`command` 分支不可带 `url`/HTTP 认证字段，`url` 分支不可带 `args/env/cwd`；非法组合会报 `invalid transport` |
| **远程 stdio 约束** | 当 `environment_id` 非默认值时，remote stdio MCP server 必须提供绝对路径 `cwd` |
| **源码锚点（便于核验）** | `README.md`（CLI/App/Web 边界）；`codex-rs/cli/src/main.rs`（`mcp-server`/`sandbox`/`cloud` 子命令）；`codex-rs/config/src/mcp_types.rs`（transport 与字段约束） |
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

## Evidence

- Status: Inferred
- Sources: OpenAI function calling / Agents SDK / CLI 相关官方文档，`https://github.com/openai/codex` 仓库代码与 README，`https://developers.openai.com/codex/agent-approvals-security`、`https://developers.openai.com/codex/cloud/internet-access`、`https://developers.openai.com/codex/app/features`，`codex-agent-evidence.md` 的 benchmark 汇总，`conflict.md` 中关于沙箱与 MCP 口径的未决项。
- Trace: 本文将模型能力、执行编排、工程治理整理为三层机制框架；其中权限模型、MCP transport、CLI 本地执行路径与 Cloud / Local 主边界已补入更直接的官方锚点，App 执行模型细节与部分 token / 稳定性数字仍属综合判断。
- Needs: 对高风险段落继续补一手来源锚点，尤其是 Cloud 环境更细规格、App 执行边界，以及社区 token / 轮次数据的适用条件。

## 8. 版本变更记录

- 2026-05-25：从原"原理与机制"文档收敛为 Agent 本体机制版。
- 2026-05-26：补充治理层详解（权限/沙箱/审计）、Token 效率对比、MCP/Skills 扩展机制。
- 2026-05-28：补充 MCP 传输形态边界说明（HTTP/SSE 与 stdio 属于协议层；产品形态能力需按官方页面分别核验）。
- 2026-05-28：根据外部联网复核结果，更新 MCP 双传输形态表述并保留产品形态边界约束。
- 2026-05-28：结合 `https://github.com/openai/codex` 仓库核验结果，将 MCP 传输术语精确为 `stdio + streamable_http`（并区分 SSE 响应流语义）。


## 参考来源

- https://platform.openai.com/docs/guides/function-calling
- https://openai.com/index/introducing-agents-sdk/
- https://openai.com/index/introducing-the-openai-cli/
- https://platform.openai.com/docs/models/gpt-4o
- https://platform.openai.com/docs/guides/rag

*最后更新: 2026-06-04*
