# Codex Agent（代码能力）

围绕 **Codex 作为代码 Agent 本体** 的系列文档，聚焦原理、使用方法与证据核验。

---

## 该从哪开始读？

### 第一次接触 Codex

如果你还不确定 Codex 是什么、值不值得花时间了解：

1. **先读** `codex-agent-overview.md`（5 分钟）—— 知道它是什么、不是什么、能干什么
2. **再读** `codex-agent-mechanisms.md`（15 分钟）—— 理解它的原理和运作机制
3. **决定是否继续**。需要更多全景时，查阅 `overview.md`（补充调研材料）或 `codex-official-docs.md`（官方导航）

### 打算用 Codex 做开发

1. `codex-agent-mechanisms.md` —— 理解它的原理、闭环、工具和权限机制
2. `codex-agent-practice.md` —— 掌握怎么用、什么场景适合、什么反模式要避开
3. `codex-troubleshooting.md` —— 遇到问题时排查

### 做选型评估或深度研究

- `codex-agent-evidence.md` —— benchmark 数据、关键事实、时间轴
- `codex-cloud-sandbox.md` —— Cloud 执行与本地沙箱的区别
- `codex-computer-use.md` —— GUI 远程操控能力
- `codex-automations-ci.md` —— Automations 和 CI/CD 集成
- `codex-open-source.md` —— 什么开源了什么没开，能不能本地部署

### 技术爱好者 / 源码研究者

- `codex-architecture.md` —— Rust 源码架构（Tokio + MLFQ + Landlock + MCP）
- `codex-multi-agent.md` —— Manager-Subagent 实现、Worktree 隔离、冲突仲裁
- `conflict.md` —— 已知内容冲突和待核验项

---

## 文档结构一览

### 主线（建议按顺序读）

| 文件 | 职责 | 阅读时间 |
|------|------|---------|
| `codex-agent-overview.md` | 定义、边界、定位，回答"这是什么" | ~5 分钟 |
| `codex-agent-mechanisms.md` | 原理、闭环、上下文、权限、工具扩展 | ~15 分钟 |
| `codex-agent-practice.md` | 使用方法、任务范式、场景建议、反模式 | ~15 分钟 |
| `codex-agent-evidence.md` | 证据分级、关键事实、Benchmark、时间轴 | ~10 分钟 |

### 专题（按需查阅）

| 文件 | 定位 | 适合谁 |
|------|------|--------|
| `codex-official-docs.md` | 官方文档导航（结构+UI+优先级） | 需要查阅官方资料时 |
| `codex-computer-use.md` | Computer Use 远程操控 GUI | 评估 GUI 自动化能力 |
| `codex-automations-ci.md` | Automations & CI/CD 集成 | 评估工程化落地 |
| `codex-cloud-sandbox.md` | Cloud 执行与沙箱对比 | 评估安全/隔离/成本 |
| `codex-architecture.md` | Rust 源码架构分析 | 技术研究者 / 源码读者 |
| `codex-multi-agent.md` | Manager-Subagent 系统详解 | 技术研究者 |
| `codex-troubleshooting.md` | 故障排查手册 | 使用中遇到问题 |
| `codex-open-source.md` | 开源状态总结 | 评估本地部署可行性 |

### 补充材料

| 文件 | 定位 |
|------|------|
| `overview.md` | 产品全貌调研材料（非主线，想看完整全景时翻阅） |
| `conflict.md` | 内容冲突记录（L6 校验） | 发现矛盾时查询 |

### 工具

| 文件 | 用途 |
|------|------|
| `tmp.md` | 调研原始素材暂存，不属于正式文档 |

---

## 写作与维护规则

- 主线文档（`codex-agent-*`）职责清晰，不互相串位
  - `overview`：定义、边界、定位
  - `mechanisms`：原理、闭环、上下文、权限、工具扩展
  - `practice`：方法、任务范式、场景建议、反模式
  - `evidence`：证据分级、关键事实、时间轴。具体冲突收口到 `conflict.md`
- 专题文档聚焦单一主题，在文件开头标注定位和适用读者
- 推断性内容标明证据等级，不写成确定事实
- 新调研先写入 `tmp.md`，再归并入对应文档
- 每次更新补充版本变更记录

## 相关资源

- 上级目录：`../README.md`

*最后更新: 2026-05-26*