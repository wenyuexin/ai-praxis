# DeepSeek Harness approval：决策闸门

> 版本锚点：`0.1.0-rc.7`（commit `99f6f02fecdb`，2026-08-17）。本文基于本地 checkout 源码核验（`Status: Observed`，锚点见 `notes/evidence.md` C14）；官方口径见 `notes/source.md` 三。

`ctx.approval` 是 dsh 的权限决策 seam（`packages/interaction/user-approval`，图谱 owner 字段有轻微漂移写 `approval`）：一次请求、一组 answerer、确定性兜底，且每次问询都落到会话日志审计。

## 1. 决策链

```
ApprovalService.request(req)
  前置：必须处于 open turn（审计对必须 turn 内——裸事件在 reload 时是 crash-tail 垃圾）
  append approval/asked（审计①）
  decide():
    signal 已中止 → 'cancelled'
    会话策略为 'never' → 'rejected'（在 dispatch 前决定，见下）
    ctx.waterfall(scopeTarget(agent), 'approval/request', req, fallback='unavailable')
      answerer 返回词汇表内 outcome → 采纳
      answerer throw / 非词汇表返回值 → 'unavailable'（fail-closed 归一化）
  append approval/decided（审计②，恰一次）
```

**三层 fail-closed**：

- 请求中止 → `cancelled`（晚到的 answer 被竞态丢弃——abort 赢后 resolve 是已结算 promise 的 no-op）。
- 无 answerer / answerer 抛错（含同步 throw，`Promise.resolve().then` 链保证进 containment）/ 返回非词汇表值 → 全部归一化 `unavailable`。
- 会话策略 `'never'` → 确定性 `rejected`：**在 dispatch 前**由服务自身决定，而不是 listener 门——因为 `prepend: true` 的 listener 可能注册在服务之后、排在门 listener 之前；只有服务的 request 路径能守住"never 无条件拒绝"的承诺（CI/无人值守的严格姿态）。

## 2. answerer 与 scope 过滤

`approval/request` 是 **scope-filtered waterfall**（`@deepseek-ai/dsh-scope`）：`scopeTarget(this, req.agent)` 使 agent-scoped listeners 只收到本 agent 的请求。answerer 是 listener——例如 ACP bridge 为自己的 agents 应答；无 answerer 时瀑布落到 fallback `unavailable`（fail closed）。

## 3. 审计与策略持久化

- **审计对**：`approval/asked`（id、toolName、可选 callId/reason）+ `approval/decided`（id、outcome）——log-only（非 surface 事件，不进模型转录），append 失败即 reject（未记录的决策不得返回）。
- **策略事件**：`approval/policy` 是持久 session 事件——**重放日志即策略状态**（resume 无需 catch-up 机制）；`effectiveApprovalPolicy` 是纯 fold：日志中最后一个 `approval/policy` 事件，否则用配置默认（`'ask'`）。
- **模型可见性双通道**：`approval:policy` systemPrompt context section（模型从 runtime-context 快照得知当前策略）；`setPolicy` 切换时 `agent.inject()` 注入"策略从 X 变为 Y（由用户更改）"通知。

## 4. 周边治理件（官方口径，未源码核验）

- `ctx.permissionPresets`：`workspace-write` / `danger-full-access` 预设表，一次写一个 `permission/preset` 事件同时打通 sandbox-mode 与 approval-policy 两个 knob。
- `ctx.sandboxPolicy`：部署默认 mode + workspace root 的唯一归属（bash 与 fs 读同一服务，避免不同 root）。
- `ctx.userQuestions`：人类问答 seam，`tool-ask-user` 在 provider-neutral `ask()` promise 上暂停工具调用。
- 与 guard（`guard.md`）分工：approval 是**执行前**闸门，guard 是**执行后**卫生。

## Evidence

- **Status**: 本文件 1-3 节结论为 `Observed`（本地 checkout `0.1.0-rc.7` 源码直接核验，锚点见 `notes/evidence.md` C14）；第 4 节为 `Verified`（图谱官方口径，`notes/source.md` 三）。
- **Sources**: `packages/interaction/user-approval/src/index.ts`；`docs/capability-seams.md`。
- **Trace**: 2026-08-21 由 ai-praxis 侧深研脚手架回流提炼（安全治理主线压实后）；版本锚点 `0.1.0-rc.7`。
- **Needs**: `permission-presets` 的双 knob 代码实现未核验（可选深度，见问题树 §4.2）。
