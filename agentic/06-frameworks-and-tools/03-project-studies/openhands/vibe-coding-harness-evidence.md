# OpenHands: Vibe Coding Harness Evidence

> 适用范围：只整理 OpenHands 在 vibe coding 场景下与 human harness 相关的最小证据，不扩写 OpenHands 的环境、架构或源码主线
> 阶段状态：第一版 intake，当前以 `Observed / Inferred / Stop-line` 的保守收敛为主
> 使用说明：本文件服务 `04-human-agent-interaction/human-in-the-loop/vibe-coding-human-harness.md` 等主线专题的对象内引用；不把 system-level 控制机制直接写成“用户持续补 harness”的已验证定论

## 一、当前可直接保留的 Observed

- OpenHands 官方文档明确提供 `confirmation policy`、`security analyzer`、`hooks`、CLI `Esc` 暂停、planning agent 示例等机制。
- `AlwaysConfirm / NeverConfirm / ConfirmRisky` 为人工介入强度提供不同 acceptance 路径。
- `hooks` 可在多个生命周期节点阻断、放行或追加检查，形成结构化约束层。
- 官方文档明确支持 CLI 暂停并提供澄清入口；同时也建议任务完成后人工审查全部变更。
- OpenHands 现有案例目录主要聚焦 environment / runtime / sandbox / workspace 主线；本次补的是一个面向 vibe coding 小专题的对象内证据侧片，不替代既有案例正文。

## 二、当前最稳的 Inferred

- OpenHands 较稳定支持 `acceptance / constraint / correction` 三类 harness，其中 `confirmation policy`、`hooks`、`Esc` 暂停的证据最强。
- 相比 Claude Code 主要通过模式切换调节控制强度，OpenHands 更像通过 policy / hook / control-plane 提供系统化控制层。
- 这种支持更偏“预设和配置型 harness”，而不是用户在对话中高频即时补充所有约束。

## 三、当前必须保留的 Stop-line

- 不要把 planning agent、critic 或 control plane 直接写成“用户持续补 harness”的直接证据；其中不少更接近系统配置、自动化控制或企业管理层能力。
- goal harness 证据仍弱，不能写成 OpenHands 已提供显式结构化 goal 管理。
- “中途追加 constraint”更多来自 hooks / policy 的预设能力，不等于已验证的低摩擦即时交互。
- OpenHands 现有对象研究重点不在 vibe coding 交互，因此这里的结论只能作为补充切片，不能反过来覆盖其架构/环境主线。

## 四、对主线专题的最小可引用结论

如果只服务 `vibe-coding-human-harness.md`，目前最保守的对象内引用可以压缩成两句：

- OpenHands 的现有证据更稳定支持 `acceptance / constraint / correction harness`，尤其是 confirmation policy、hooks 与 CLI 暂停。
- 在当前材料里，OpenHands 更像通过 system-level 控制层支持 human harness，而不是把 `goal harness` 做成显式一等交互面。

## Evidence

- Status: Observed / Inferred / Unverified
- Sources:
  - `./official-docs.md`
  - `./architecture.md`
  - OpenHands docs：`https://docs.openhands.dev/openhands/usage/security`
  - OpenHands docs：`https://docs.openhands.dev/openhands/usage/hooks`
  - OpenHands docs：`https://docs.openhands.dev/openhands/usage/advanced/cli-mode`
  - OpenHands docs：`https://docs.openhands.dev/openhands/usage/advanced/agents/planning-agent`
- Trace: 本文不是新增 OpenHands 总览，而是把此前为 vibe coding human harness 收集的对象内线索，收束回更直接的 OpenHands 官方文档入口与既有对象专题；OpenHands 现有案例主线仍以 architecture / runtime / sandbox 为主，这里只补一个与小专题对接的保守证据切片。
- Needs:
  - 用更直接的官方文档确认 confirmation policy、hooks、CLI 暂停在用户运行时介入中的正式语义。
  - 区分哪些能力主要面向 system policy / enterprise control，哪些真的服务用户即时纠偏与 acceptance。
