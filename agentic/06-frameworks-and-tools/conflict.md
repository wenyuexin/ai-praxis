# Frameworks and Tools Conflict（冲突与口径校验）

本文件记录 `06-frameworks-and-tools` 范围内确实存在相反来源、术语口径冲突或边界判断不一致的问题。纯粹的内容缺口、单向证据不足和候选对象队列分别记录在 [`backlog.md`](./backlog.md) 和 [`candidates.md`](./candidates.md)；冲突解决后，应同步更新相关主干文档，并移除或归档对应条目。

---

## 一、待核验的口径与边界

### 1.1 框架成熟度与版本状态

| 对象 | 待核验点 | 建议来源 |
|------|----------|----------|
| `OpenAI Agents SDK / Responses API` | “七层架构”应继续视为外部归纳；当前主线只可稳定写到 handoffs、results/state、sessions、tracing、sandbox、tools/MCP 等公开能力面，不宜上推成统一 runtime 契约 | OpenAI 官方 docs / SDK 文档 / 对象目录正文 |
| `DSPy` | 是否应作为 Agent framework 主干对象，还是只作为 prompt / program optimization 对比对象 | Stanford DSPy 官方文档与论文 |

---

## 二、维护规则

- 只有当某项存在相反来源、事实冲突、术语冲突或边界口径不一致时，才保留在本文件。
- 单向证据不足、版本状态未确认或尚未覆盖的问题，转入 `backlog.md`；具体对象仍同时可在 `candidates.md` 中保留队列状态。
- 候选对象与前沿观察不在本文件维护，统一由 `candidates.md` 承接。
- 如果核验后形成稳定结论，应同步更新 `overview.md`、相关子目录 README 或具体对象文档。

本轮迁移已将原先的候选项目、热度数据和前沿观察条目分别回流 `candidates.md` 与 `backlog.md`；本文件只保留仍需处理的口径和边界问题。

---

*最后更新: 2026-05-31*
