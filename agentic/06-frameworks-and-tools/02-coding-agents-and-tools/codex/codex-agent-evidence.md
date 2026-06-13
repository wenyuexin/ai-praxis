# Codex Agent 证据与核验

> 目标：把关键结论压缩到可核验证据，避免"信息很多但结论不稳"。
>
> 本文与 `overview.md`、`codex-agent-overview.md`、`codex-agent-mechanisms.md` 可能共享同一组核心结论，但这里只负责给出证据分级、核验口径和不可直接比较项，不重复承担综述、机制解释或使用方法的主体角色。

## 1. 关键事实（高优先）

### 通用事实

- Codex API 已弃用，代码能力并入后续模型体系。
- Function calling 支持并行 tool calls（上限见官方文档）。
- 模型上下文窗口有标称值，但工程有效值受检索/提示/输出预算影响。
- Copilot 与企业治理能力（策略/审计）在官方文档有明确说明。

### Benchmark 数据

| 指标 | Codex 得分 | 数据来源 | 证据等级 |
|------|-----------|---------|---------|
| SWE-bench Verified | GPT-5.2: **80.0%** | Anthropic 官方 | A |
| SWE-bench Pro | GPT-5.3-Codex: **56.8%** | 社区实测 | B |
| Token 效率（Express.js 重构任务） | **~150 万 token** | 社区量化分析 | B |
| 成本（UI 克隆任务） | **~$2.50** | Composio 实测 | B |
| 成本（Lint 修复） | **~$0.25** | Composio 实测 | B |
| 社区采用（npm 下载量，2026.5） | **8610 万次/周** | TickerTrends | A |

> ⚠️ SWE-bench Verified 和 SWE-bench Pro 是**不同难度等级**的评测集。Pro 更贴近真实开发场景，难度高于 Verified。GPT-5.3-Codex 的 Pro 得分（56.8%）与 GPT-5.2 的 Verified 得分（80.0%）**不可直接比较**。详情见 `conflict.md`。

### 关键事实结论

- 官方实践案例（OpenAI 团队使用 Codex）显示：高频收益主要来自重构迁移、性能优化、测试补齐和任务异步派发（证据等级：A，经验型实践证据）。

| 结论 | 证据等级 |
|------|---------|
| Token 效率高，同任务消耗约 150 万 token | B |
| 成本极低，日常任务仅 $0.2-0.25/次 | B |
| 安全治理采用 OS 内核级沙箱（Landlock+Seccomp/Seatbelt/ACL） | A |
| SWE-bench Pro（场景设计）比 Verified 更接近真实开发 | B |
| 2026 年社区采用增长显著（npm 周下载 8610 万次） | A |

## 2. 证据分级规则

- A：官方一手（官方文档/公告/博客/权威 benchmark）
- B：可复现二手（社区实测/量化分析/知名博主验证）
- C：经验判断（Reddit/HN/Dev.to 社区讨论/个案反馈）

原则：

- 数字结论优先 A
- B/C 结论必须写适用条件
- C 不能写成"确定事实"

## 3. 常见冲突点

涉及术语冲突、评测可比性、沙箱歧义等问题，统一见本目录 `conflict.md`。本文不再单独维护。

## 4. 时间轴

- 2023-03：Codex API 弃用迁移
- 2024-05：GPT-4o 发布
- 2025-03：OpenAI Agents SDK 发布
- 2025-04：OpenAI CLI 发布（CLI 版 Codex 雏形）
- 2026-02：Codex 桌面 App（macOS）发布
- 2026-04-16：Codex 大版本升级——Computer Use + 90+ 官方插件 + Automations + 桌面 App（Windows）

## 5. Rollback / Restore 边界

当前与 `rollback`、`restore`、`undo turns` 相关的官方证据，最稳妥的分层理解是：

- `Codex CLI` 官方文档已明确 approval policy、sandbox、subagents、`/goal` 与 `codex resume` 等控制入口。
- `rollback` 相关更直接的官方材料，目前主要来自 changelog / release notes 中的 `app-server thread rollback` 与 IDE 客户端撤销最近 N 个回合能力。
- 现阶段**仍未找到 `Codex CLI` 原生 rollback / checkpoint 覆盖范围的官方定义**，因此不能把 IDE / app-server 层的恢复能力直接外推为 CLI 原生的完整恢复语义。
- 尤其不能直接推定它覆盖 `workspace files`、`shell side effects`、`external side effects` 或 `runtime identity`。

这意味着：如果讨论 `Codex` 作为对象本身的恢复能力，应区分 `CLI`、`app-server` 与 `IDE integration` 三层；如果讨论它在跨主题中的控制面意义，则只能保守写成“最近 N 个回合的 thread rollback 线索已存在，但 CLI 原生 rollback scope 未闭环”。

## 6. 维护建议

- 每月核验一次核心数字（窗口、成本、benchmark）
- 每季度更新一次时间轴
- 新结论先标 B/C，证据补齐后再升 A

## Evidence

- Status: Mixed
- Sources: A 级来源以 OpenAI 官方文档、官方实践材料与公开 benchmark 页面为主；B/C 级来源为社区实测、量化分析和讨论帖，详见本文参考来源与 `conflict.md`。
- Trace: 本文作为 `codex/` 目录的证据收口文档，负责压缩关键事实、benchmark、时间线与分级口径；冲突与不可直接比较项不在正文硬下结论，而是转交 `conflict.md` 维护。
- Needs: 继续核验 benchmark 版本差异、社区成本样本的可重复性，以及“社区采用”类指标的统计口径与时间窗口。

## 6. 版本变更记录

- 2026-05-25：合并核验、术语、时间轴与失败案例素材为单一证据文档。
- 2026-05-26：补充 Benchmark 数据表，扩展时间轴至 2026.4。后续移除跨框架对比内容，聚焦 Codex 自身证据。
- 2026-05-28：新增 OpenAI 官方实践案例作为经验型 A 级证据来源。

## 参考来源

- https://openai.com/index/api-changes-coming-to-chat-completions-and-the-completions-endpoint/
- https://platform.openai.com/docs/guides/function-calling
- https://platform.openai.com/docs/models/gpt-4o
- https://openai.com/index/introducing-agents-sdk/
- https://openai.com/index/introducing-the-openai-cli/
- https://openai.com/zh-Hans-CN/business/guides-and-resources/how-openai-uses-codex/
- https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-for-your-organization
- https://www.swebench.com/
- https://arxiv.org/abs/2406.01939

*最后更新: 2026-06-04*
