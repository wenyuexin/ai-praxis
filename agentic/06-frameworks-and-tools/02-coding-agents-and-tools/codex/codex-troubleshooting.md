# Codex 故障排查边界

> **专题文档**。本文只整理目前能从 OpenAI 官方 Codex 文档中保守确认的故障排查入口、恢复线索与常见卡住场景，并明确哪些“状态机 / 超时 / 内部错误语义”仍不宜写成稳定规格。

> 截至当前，可以较明确确认的点包括：Codex App 存在官方 `Troubleshooting` 页面；官方公开了日志位置、session transcript 与 archived sessions 路径；线程卡住时优先检查 approval、终端基本命令和更小任务重开线程；误取消 worktree 创建后可用输入框上箭头找回 prompt；权限与 sandbox 模式会直接影响可执行行为。至于完整状态机、`TurnAbortReason`、固定超时规则、`resume` 恢复窗口、以及 Cloud 任务可稳定运行多长时间等说法，当前都不应写成稳定一手规格。

---

## 1. 当前可直接确认的排查入口

### 1.1 官方确实提供了 Troubleshooting 页面

官方 `app/troubleshooting` 页面当前可直接确认：Codex App 存在专门的故障排查页面，并提供常见问题与恢复建议。

这至少说明两件事：

- Codex 的故障排查已经有**产品层公开入口**；
- 更稳的排查知识应优先围绕官方已公开的问题场景，而不是先假定完整内部状态机。

### 1.2 日志与会话材料位置有明确公开口径

官方 `app/troubleshooting` 页面当前可直接确认：

- App logs（macOS）：`~/Library/Logs/com.openai.codex/YYYY/MM/DD`
- Session transcripts：`$CODEX_HOME/sessions`（默认 `~/.codex/sessions`）
- Archived sessions：`$CODEX_HOME/archived_sessions`（默认 `~/.codex/archived_sessions`）

因此，当前较稳的排查入口应先落在：

- **日志**
- **session transcripts**
- **archived sessions**

而不是先从社区推断的内部状态名切入。

---

## 2. 当前可直接确认的卡住与恢复模式

### 2.1 Thread 卡住时，官方建议先排查 approval、终端与任务粒度

官方 `app/troubleshooting` 页面明确给出 stuck state 的恢复模式：

1. 检查 Codex 是否正在等待 approval。
2. 打开 terminal，运行一个基础命令，比如 `git status`。
3. 用更小、更聚焦的 prompt 新开一个 thread。

这意味着当前最稳妥的“卡住排查顺序”是：

- 先看是不是**审批等待**；
- 再看 terminal 是否还能正常工作；
- 最后再考虑把大任务拆小并重开线程。

### 2.2 Worktree 创建误取消有明确恢复动作

官方 `app/troubleshooting` 页面还明确写到：

- 如果你在创建 worktree 时误取消并丢失 prompt，可以在 composer 中按上箭头恢复输入。

这类信息比“会话恢复窗口”之类社区推测更适合直接进入故障排查正文。

---

## 3. 权限、审批与 sandbox 是真实故障来源

### 3.1 Approval 与 sandbox 直接决定“为什么它没继续跑”

结合 `agent-approvals-security` 与 `cli/features` 当前可直接确认：

- `Sandbox mode` 决定技术边界，例如可写目录和网络能力；
- `Approval policy` 决定什么时候必须先询问用户；
- `Auto` 默认允许在工作目录中读、改、跑命令，但对越界或网络仍会询问；
- `Read-only` 更偏咨询模式；
- `Full Access` 能力最强，但风险也最高。

因此，很多“线程没动”“命令没跑”“网络请求失败”之类问题，更稳妥的第一解释常常不是“内部状态异常”，而是：

- 正在等 approval；
- 当前 sandbox mode 不允许该操作；
- 当前网络权限或工作目录边界阻止了执行。

### 3.2 网络相关故障要先区分 blocked 还是 live access 未开启

当前官方口径还可确认：

- CLI / 本地沙箱默认网络受限；
- Cloud agent phase 默认离线；
- 某些功能如 OTel export，在网络关闭时就无法连到 collector；
- web search 也可能运行在缓存模式，而不是直接 live fetch。

因此，碰到“联网失败”时，更稳妥的排查顺序应是：

- 先确认当前运行形态是本地还是 Cloud；
- 再确认是否卡在 approval / sandbox / network policy；
- 最后才讨论是否存在更深层的执行错误。

---

## 4. 当前不宜继续写死的说法

下列说法在当前一手材料下仍不宜继续写成正文定论：

| 不宜写死的说法 | 当前状态 |
|----------------|----------|
| 存在稳定公开的完整 Agent 状态机：`PendingInit → Running → Interrupted ...` | 当前未取得稳定官方锚点 |
| `Interrupted` / `Errored` / `Shutdown` / `NotFound` 是统一公开产品状态名 | 未找到当前稳定一手锚点 |
| 存在稳定公开的 `TurnAbortReason` 枚举（`Interrupted / Replaced / ReviewEnded / BudgetLimited`） | 更接近实现层或社区归纳 |
| 官方已公布固定单任务超时与会话级空闲超时规则 | 未找到 |
| `resume` 有明确官方恢复窗口时长 | 未找到 |
| Cloud 任务可稳定连续运行 `7h+` | 社区样本，不宜写成规格 |
| `/status` 是会话状态查看命令 | 当前没有稳定一手锚点支撑这一排查说法 |
| 已确认存在专门的 Known Issues 页面并覆盖所有故障类型 | 当前未重新取得稳定一手锚点 |

这意味着：**故障排查专题更适合先写官方公开的排查动作、日志位置、权限/网络边界与 thread 恢复动作，而不是内部状态机图。**

---

## 5. 当前可接受的保守排查框架

### 5.1 先看“有没有被边界拦住”

当前较稳的排查顺序是：

- 是否正在等待 approval；
- 当前 sandbox mode 是否允许写文件、跑命令或出网；
- 当前任务是否超出了项目根目录 / worktree / 当前项目边界。

### 5.2 再看“有没有可回看的材料”

当前较稳的排查材料包括：

- App logs
- Session transcripts
- Archived sessions
- Review pane / diff / transcript 中已暴露的动作记录

### 5.3 最后再把任务缩小重试

如果线程卡住、终端异常或 prompt 太大，当前官方更明确支持的做法是：

- 跑一个最基础的 terminal 命令确认环境还活着；
- 新开一个更小、更聚焦的 thread；
- 若是 worktree 创建时误操作，可先尝试恢复丢失的 prompt。

---

## 6. 当前可接受的一句话总结

> 截至当前，更稳妥的说法是：Codex 官方已经公开了日志位置、thread 卡住时的恢复动作、以及 approval / sandbox / network 边界这类第一层排查入口；但完整状态机、固定超时规则、内部中止原因与恢复窗口等更细语义，仍不应写成已核实规格。

---

## Evidence

- Status: Mixed
- Sources: OpenAI 官方页面 `https://developers.openai.com/codex/app/troubleshooting`、`https://developers.openai.com/codex/cli/features`、`https://developers.openai.com/codex/agent-approvals-security`；既有社区案例材料。
- Trace: 本文已从“状态机 / 超时 / 恢复语义混写”收缩为“官方可确认排查动作 + 未找到项 + 社区样本外推限制”结构；凡缺少当前稳定一手锚点的状态名、时长规则与中止原因，均不再写成主线定论。
- Needs: 若继续补证，应核验 App / CLI 是否还有更细的会话恢复说明、Known Issues / changelog 对故障语义的公开覆盖范围，以及哪些实现层状态名适合只保留在候选或冲突层。

## 参考来源

- `https://developers.openai.com/codex/app/troubleshooting`
- `https://developers.openai.com/codex/cli/features`
- `https://developers.openai.com/codex/agent-approvals-security`

*最后更新: 2026-06-04*
