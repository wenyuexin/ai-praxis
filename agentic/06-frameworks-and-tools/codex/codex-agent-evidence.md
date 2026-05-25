# Codex Agent 证据与核验

> 目标：把关键结论压缩到可核验证据，避免“信息很多但结论不稳”。

## 1. 关键事实（高优先）

- Codex API 已弃用，代码能力并入后续模型体系。
- Function calling 支持并行 tool calls（上限见官方文档）。
- 模型上下文窗口有标称值，但工程有效值受检索/提示/输出预算影响。
- Copilot 与企业治理能力（策略/审计）在官方文档有明确说明。

## 2. 证据分级规则

- A：官方一手（官方文档/公告/博客）
- B：可复现二手（论文/公开 benchmark）
- C：经验判断（社区讨论/个案）

原则：

- 数字结论优先 A
- B/C 结论必须写适用条件
- C 不能写成“确定事实”

## 3. 常见冲突点

- 标称上下文 vs 实际有效上下文
- 功能“支持” vs “稳定可用”
- 模型能力强 vs 工程交付稳定
- 社区体感 vs 官方口径

## 4. 时间轴（极简）

- 2023-03：Codex API 弃用迁移
- 2024-05：GPT-4o 发布
- 2025-03：OpenAI Agents SDK 发布
- 2025-04：OpenAI CLI 发布

## 5. 维护建议

- 每月核验一次核心数字（窗口、成本、benchmark）
- 每季度更新一次时间轴
- 新结论先标 B/C，证据补齐后再升 A

## 6. 版本变更记录

- 2026-05-25：合并核验、术语、时间轴与失败案例素材为单一证据文档。

## 参考来源

- https://openai.com/index/api-changes-coming-to-chat-completions-and-the-completions-endpoint/
- https://platform.openai.com/docs/guides/function-calling
- https://platform.openai.com/docs/models/gpt-4o
- https://openai.com/index/introducing-agents-sdk/
- https://openai.com/index/introducing-the-openai-cli/
- https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-for-your-organization
- https://www.swebench.com/
- https://arxiv.org/abs/2406.01939

*最后更新: 2026-05-25*
