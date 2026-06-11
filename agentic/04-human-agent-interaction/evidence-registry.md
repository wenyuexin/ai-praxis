# Human-Agent Interaction Evidence Registry

> 适用范围：`04-human-agent-interaction/` 中 human-in-the-loop、delegation granularity、control surface、handoff / takeover 等主题的共享证据登记
> 阶段状态：目录级证据 registry，服务主线专题回填与后续核验
> 使用说明：本文件不是正文专题，也不替代各专题的 `Evidence` 小节；它只登记已回填证据、待核验证据与不可外推边界，避免从 `temp/` 材料直接写成主线定论

## 一、当前已回填的证据

| 证据对象 | 来源类型 | Evidence 状态 | 已回填位置 | 支撑的谨慎表述 |
|---|---|---|---|---|
| OpenHands `Security & Action Confirmation` | 官方文档 | Observed | `human-in-the-loop/human-in-the-loop-patterns.md`、`delegation-and-control/control-surface-design.md`、`delegation-and-control/delegation-granularity.md` | OpenHands 将 action control 拆成 confirmation policy 与 security analyzer，支持全量确认、无需确认和风险分级确认。 |
| Microsoft Learn `GitHub Copilot Edits` | 官方文档 | Observed | `human-in-the-loop/human-in-the-loop-patterns.md`、`delegation-and-control/control-surface-design.md`、`delegation-and-control/delegation-granularity.md` | Copilot Edits 提供建议变更的 diff review、逐项接受 / 拒绝、summary diff 和 reset，适合作为 review / artifact-level fallback 案例。 |
| Cursor `agent-computer-use` | 官方博客 | Observed | `delegation-and-control/control-surface-design.md` | Cursor cloud agents 运行在独立 VM 中，用户可接管远程桌面并亲自编辑，可作为 remote takeover 与 VM sandbox 的产品案例。 |
| Cursor SDK changelog | 官方 changelog | Observed | `delegation-and-control/control-surface-design.md`、`delegation-and-control/delegation-granularity.md` | Cursor SDK 支持 agent / run metadata 持久化、`requestId` 追踪、自定义存储和嵌套子 agent，可作为 traceability 与 delegation tree 案例。 |
| Claude Code permission modes / settings / subagents | 官方文档 | Observed | `delegation-and-control/control-surface-design.md`、`delegation-and-control/delegation-granularity.md` | Claude Code 官方文档确认 `default`、`acceptEdits`、`plan`、`auto`、`dontAsk`、`bypassPermissions` 等 permission modes，settings 中的 allow / ask / deny 规则，以及 named subagent 的独立 prompt、tools、permissionMode、background / worktree isolation 等配置。 |
| Claude Code Auto Mode stress-test | 独立论文 | Observed / Inferred | `delegation-and-control/control-surface-design.md` | 论文在 scope escalation 压测任务中报告 Auto Mode 的高 false negative rate，并指出项目内文件编辑绕过 classifier 的 Tier 2 覆盖缺口；可用于说明 classifier gate 不是完整 safety boundary。 |
| `Handoff Debt` | arXiv 论文 | Observed / Inferred | `human-in-the-loop/human-in-the-loop-patterns.md` | 论文定义 agent-to-agent handoff 的 rediscovery cost，并比较不同 handoff views；可支持 structured handoff 影响恢复成本，但不直接支持 human takeover 结论。 |
| `Hedwig` | arXiv 论文 / ACM CAIS demo | Observed / Inferred | `delegation-and-control/delegation-granularity.md`、`delegation-and-control/control-surface-design.md` | 研究原型将 coding-agent autonomy 视为可随任务、信任和历史交互变化的量，可作为 dynamic autonomy / autonomy budget 线索。 |
| `Authenticated Delegation and Authorized AI Agents` | 论文 / 框架提案 | Inferred | `delegation-and-control/delegation-granularity.md`、`delegation-and-control/control-surface-design.md` | agent-specific credentials、delegation tokens、scope limitation 与 audit trail 可作为 capability-bounded delegation 的框架思路。 |

## 二、当前待核验的证据

| 待核验对象 | 当前状态 | 暂不回填为定论的原因 | 优先核验问题 |
|---|---|---|---|
| Claude Code Auto Mode / checkpoint / permission edge cases | permission modes、settings 和 subagents 已有官方文档支撑；Auto Mode 压测来自独立论文；checkpoint 仍主要来自第三方工具 | Auto Mode classifier 的完整规则、fallback 条件、checkpoint 原生性和恢复范围仍未完全由官方材料闭合 | Auto Mode classifier / fallback 的官方精确定义、checkpoint 是否为原生能力、rollback 是否覆盖 workspace / conversation / runtime / 外部副作用。 |
| Claude Code subagent permission isolation | 官方文档已确认 named subagent 可配置 prompt、tools、permissionMode、background 和 worktree isolation | 仍需区分“可配置工具 / permissionMode”和“安全隔离”；background permission prompt、worktree isolation 与权限继承关系仍需更细核验 | subagent 是否能作为安全隔离边界，foreground / background 权限提示差异，worktree isolation 覆盖范围。 |
| OpenAI Codex CLI approval modes | 第三方技术博客和集成文档为主 | OpenAI 官方文档尚未核验 | approval modes 名称、文件编辑与 shell command 的审批边界、sandbox 具体作用。 |
| OpenAI Agents SDK `needsApproval` | DeepWiki 整理材料，需上游源码 / 官方文档复核 | SDK 机制不等于 Codex CLI 产品能力 | `needsApproval`、interruptions、state serialization、tracing 的官方定义和版本基线。 |
| Cursor checkpoint / auto-review 细节 | 官方 changelog 已有线索，但 schema / 优先级未核 | 产品机制仍在快速演进 | `permissions.json` schema、auto-review 默认行为、checkpoint 存储与审计边界。 |
| SWE-agent human oversight / handoff | 暂无足够材料 | SWE-agent 更偏 benchmark / research environment，不一定有 production HITL | 是否存在明确的人类审批、接管、handoff 或 review 机制。 |

## 三、不可外推边界

- 不要把单个产品的 approval 策略写成 coding-agent 行业标准。
- 不要把 approval popup 等同于完整 human-in-the-loop；HITL 还包括 correction、review、validation、handoff、takeover 和 fallback。
- 不要把 `Handoff Debt` 的 agent-to-agent 交接成本直接外推为 human takeover 成本。
- 不要把 `Hedwig` 的 dynamic autonomy 原型写成生产系统最佳实践。
- 不要把 Cursor cloud agents 的 VM sandbox + remote takeover 写成所有 coding agent 都应采用的控制面。
- 不要把 Claude Code Auto Mode 的 classifier gate 写成完整安全边界；独立压测显示项目内文件编辑等路径可能不经过 classifier，且该结果只适用于特定 scope escalation 压测。
- 不要把 Claude Code subagent 写成安全隔离机制；即便它支持独立 prompt、tools、permissionMode 或 worktree isolation，也需要继续核验权限继承和隔离边界。
- 不要把 checkpoint / reset 写成完整恢复语义；它可能只覆盖 artifact 或某一轮建议变更，不等于 workspace、conversation、runtime 和外部副作用的整体恢复。
- 不要混淆 UI surface、control surface 和 permission system：UI 负责呈现与操作，permission system 负责执行限制，control surface 负责人类可理解和可操作的控制语义。

## 四、与主线专题的关系

- `human-in-the-loop/human-in-the-loop-patterns.md`：优先使用 OpenHands、Copilot Edits、Handoff Debt 支撑 approval、review、validation、handoff 的模式边界。
- `delegation-and-control/delegation-granularity.md`：优先使用 Authenticated Delegation、Hedwig、Cursor SDK 支撑 capability-bounded delegation、dynamic autonomy 与 delegation tree。
- `delegation-and-control/control-surface-design.md`：优先使用 OpenHands、Copilot Edits、Cursor cloud agents / SDK、Hedwig 支撑 approval point、autonomy budget、takeover entry、fallback path 与 traceability。
- `human-in-the-loop/vibe-coding-human-harness.md`：只可轻量吸收 handoff、review、validation、context restart 等经验支撑，不应转写为产品机制综述。

## 五、后续补证优先级

1. 核验 Anthropic 官方 Claude Code permission modes、Auto Mode、Checkpoints 和 Subagents 文档。
2. 核验 OpenAI 官方 Codex CLI approval modes、sandbox 和 rollback / checkpoint 支持。
3. 核验 Cursor `permissions.json`、checkpoint 存储、auto-review 优先级和子 agent 权限继承。
4. 找到 SWE-agent / OpenHands / Cursor / Claude Code 在真实长任务中 handoff、takeover、fallback 的更多案例。
5. 对比 dynamic autonomy 是否只是研究原型，还是已经在产品中以 auto-review / classifier approval / adaptive permissions 的形式出现。
