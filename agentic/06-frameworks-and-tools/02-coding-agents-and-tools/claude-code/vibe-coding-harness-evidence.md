# Claude Code: Vibe Coding Harness Evidence

> 适用范围：只整理 Claude Code 在 vibe coding 场景下与 human harness 相关的最小证据，不扩写 Claude Code 全局产品分析
> 阶段状态：第一版 intake，当前以 `Observed / Inferred / Stop-line` 的保守收敛为主
> 使用说明：本文件服务 `04-human-agent-interaction/human-in-the-loop/vibe-coding-human-harness.md` 等主线专题的对象内引用；不把第三方经验直接写成 Claude Code 的稳定官方工作流

## 一、当前可直接保留的 Observed

- Claude Code 提供多种 permission mode，包括 `plan`、`acceptEdits`、`auto` 等，覆盖从先规划后执行到较高自治的工作流。
- `plan` mode 支持只读规划；仓库现有 `best-practices.md` 已把“复杂任务先 Plan 后执行”标为 `[官方明确]`。
- 仓库现有 `best-practices.md` 还保留了“任务开始前写目标、约束、验收标准”“修改前确认影响范围与回滚策略”等实践，这些至少说明 Claude Code 生态把 goal / constraint / acceptance 视为显式工作流要素。
- 本轮联网 intake 还观察到 `/goal`、Agent View、后台会话与模式切换等交互面，可作为“用户在运行中查看状态、切换阶段与决定何时介入”的候选证据，但证据强度不完全一致。

## 二、当前最稳的 Inferred

- Claude Code 较稳定支持 `constraint / acceptance / task-switch` 三类 harness：用户可以通过 permission mode 和 plan mode 持续收紧委托边界。
- 如果把 vibe coding 理解为“人类持续补 harness”，Claude Code 更接近“在交互中即时切换控制模式”的路径，而不是完全依赖预先配置的 system policy。
- `/goal` 更适合暂写成“预设完成条件”能力，而不是直接写成强 `goal harness` 证据。

## 三、当前必须保留的 Stop-line

- 不要把第三方文章中的 `CLAUDE.md` 使用技巧、`/clear` 经验或操作习惯，直接写成官方主工作流。
- 不要把 `/goal` 直接写成“人类持续补 goal”的稳定产品证据。
- 当前材料更能支撑“Claude Code 提供了相关能力”，还不能支撑“用户典型地低摩擦持续使用这些机制”。
- 在没有更强官方来源前，不要把 Agent View、后台会话等交互面直接扩大成“Claude Code 已经完整支持 human harness loop”。

## 四、对主线专题的最小可引用结论

如果只服务 `vibe-coding-human-harness.md`，目前最保守的对象内引用可以压缩成两句：

- Claude Code 的现有证据更稳定支持 `constraint / acceptance / task-switch harness`，尤其是 plan mode 与 permission mode 的阶段切换。
- `goal harness` 在当前材料中仍偏弱，`/goal` 更接近预设完成条件，而不是运行中持续重写目标。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources:
  - `./best-practices.md`
  - `./claude-code-guide.md`
  - Anthropic Claude Code docs：`https://docs.anthropic.com/en/docs/claude-code/overview`
  - Anthropic Claude Code docs：`https://docs.anthropic.com/en/docs/claude-code/common-workflows`
  - Anthropic Claude Code docs：`https://docs.anthropic.com/en/docs/claude-code/settings`
- Trace: 本文不是新增 Claude Code 总览，而是把此前为 vibe coding human harness 收集的对象内线索，尽量回收到更直接的官方文档入口与既有对象专题；后续若继续补强，应优先继续沿官方文档与对象专题补证，而不是放大第三方转述。
- Needs:
  - 用更直接的官方文档确认 `/goal`、Agent View、permission mode 的正式语义。
  - 区分哪些是官方推荐工作流，哪些只是社区实践或转述经验。
