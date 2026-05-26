# Codex Agent 证据与核验

> 目标：把关键结论压缩到可核验证据，避免"信息很多但结论不稳"。

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

## 5. 维护建议

- 每月核验一次核心数字（窗口、成本、benchmark）
- 每季度更新一次时间轴
- 新结论先标 B/C，证据补齐后再升 A

## 6. 版本变更记录

- 2026-05-25：合并核验、术语、时间轴与失败案例素材为单一证据文档。
- 2026-05-26：补充 Benchmark 数据表，扩展时间轴至 2026.4。后续移除跨框架对比内容，聚焦 Codex 自身证据。

## 参考来源

- https://openai.com/index/api-changes-coming-to-chat-completions-and-the-completions-endpoint/
- https://platform.openai.com/docs/guides/function-calling
- https://platform.openai.com/docs/models/gpt-4o
- https://openai.com/index/introducing-agents-sdk/
- https://openai.com/index/introducing-the-openai-cli/
- https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-for-your-organization
- https://www.swebench.com/
- https://arxiv.org/abs/2406.01939

*最后更新: 2026-05-26*