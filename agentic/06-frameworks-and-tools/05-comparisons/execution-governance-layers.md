# Execution Governance Layers

> 适用范围：比较不同 Agent 系统中 rules、skills、hooks、approvals、sandbox 与 tool contracts 如何真正进入执行面。
> 阶段状态：第一版问题框架，先回答“执行约束有哪几层、各层 stop-line 在哪里”，不冻结某一实现方案。
> 使用说明：本文不试图回答某个具体系统“最好怎么设计”，而是先区分：哪些约束只是 prompt-level guidance，哪些已经进入 tool-loop，哪些属于 runtime / policy / environment 的硬边界。

## 一、为什么要单独研究执行约束分层

很多 Agent 系统都会声称自己支持：

- instructions / rules
- skills / procedures
- tools / plugins / MCP
- permissions / approvals
- sandbox / worktree / runtime isolation
- hooks / guards / policy controls

但这些名词同时出现，并不等于 agent 就会稳定遵守它们。

真正关键的问题是：**这些约束到底进入了哪一层执行链，以及它们的约束力度到底有多强。**

如果不先把执行约束分层，系统很容易把下面几种东西混写成同一类能力：

- 写在 prompt 里的行为要求
- 在 tool call 前拦截的 hook
- 需要用户确认的 approval gate
- 由 sandbox 强制执行的硬边界
- 执行完成后的审计与评估信号

这些东西都会影响 agent 行为，但它们不是同一层约束，也不具备同样的可信度。

---

## 二、一个最小问题定义

本文先把“规则和技能如何真正约束执行”收窄为一个比较问题：

> **一个 Agent 系统中的 rules / skills / hooks / approvals / sandbox / tool contracts，分别在哪一层进入执行链，它们能改变什么，不能改变什么？**

如果一句约束只能改变模型“倾向怎么想”，它和能直接拦下 tool call 的机制，不应被写成同一种执行约束。

---

## 三、执行约束的五层模型（第一版）

### 3.1 Prompt-Level Guidance

这是最弱的一层。约束以 instructions、rules、system prompt、session prompt、memory context、skill text 等形式进入模型上下文。

它的特点是：

- 能影响模型偏好、风格、步骤顺序与自我约束倾向
- 能改变模型是否更愿意调用某类工具或避免某类行为
- 不能直接阻止已经形成的 tool call
- 无法单独保证 side-effect boundary

因此，这一层更像**行为建议与认知引导层**，而不是硬执行边界。

### 3.2 Capability Surface / Tool Contract

这一层决定系统向模型暴露了什么可调用能力，以及这些能力如何被描述。

典型形式包括：

- tool schema
- MCP `tools/list` / `tools/call`
- input / output schema
- tool annotations
- 可发现 skill / plugin / connector surface

它的特点是：

- 能影响模型“看见什么、以什么结构调用”
- 能限制模型只能在显式暴露的 capability surface 中行动
- 仍不能单独决定权限、审批、sandbox 或 rollback

因此，这一层更像**能力暴露层**，不是完整执行治理层。

### 3.3 Tool-Loop Interception

这一层开始真正进入执行链。约束在 tool call 形成之后、实际执行之前或之后介入。

典型形式包括：

- `PreToolUse`
- `PermissionDenied`
- `PostToolUse`
- `PostToolUseFailure`
- `PostToolBatch`
- input rewrite / deny / ask / defer

它的特点是：

- 可以在 tool execution 前真正阻止、延迟、修改或要求确认
- 可以在 tool execution 后记录、审计、补充上下文或要求后续动作
- 仍依赖 host runtime、hook engine 或 tool harness 才能生效
- 一般只覆盖 tool-loop，而不天然覆盖更下层 runtime 行为

因此，这一层是**执行拦截层**，比 prompt guidance 强，但还不等于环境级硬边界。

### 3.4 Policy / Approval Enforcement

这一层负责把“是否允许执行”从局部 hook 逻辑提升为更稳定的治理机制。

典型形式包括：

- approval policy
- permission gate
- risk classification
- side-effecting tool review
- transport-level authorization
- per-tool approval mode

它的特点是：

- 可以决定某次动作是否必须停下来等待确认
- 可以按工具、风险、环境、用户身份或组织策略做差异化判断
- 不一定直接实现执行本身，但能决定是否放行
- 如果没有更下层硬边界兜底，仍可能被实现缺陷或错误配置削弱

因此，这一层更像**治理放行层**。

### 3.5 Environment Hard Boundary

这是最强的一层。约束不再只是“建议、拦截或审批”，而是由执行环境本身强制实现。

典型形式包括：

- sandbox mode
- workspace-write / read-only / danger-full-access
- container boundary
- worktree isolation
- filesystem roots
- network isolation

它的特点是：

- 即使模型、hook 或 policy 判断失误，环境仍能物理或系统级地限制动作
- 可以稳定限制文件、网络、进程、工作域等副作用边界
- 不负责解释“为什么要这么限制”，但负责真正把限制落地

因此，这一层是**执行硬边界层**。

---

## 四、这五层分别能做什么，不能做什么

| 层级 | 能做什么 | 不能做什么 |
|---|---|---|
| Prompt-Level Guidance | 影响模型偏好、计划顺序、行为风格、自我提醒 | 不能直接阻止工具调用，也不能单独保证副作用边界 |
| Capability Surface / Tool Contract | 限定可见能力、输入输出结构、调用入口 | 不能单独提供审批、sandbox、rollback 或治理闭环 |
| Tool-Loop Interception | 在执行前后拦截、修改、延迟、拒绝、记录 tool call | 不天然覆盖 runtime 下层行为，也不等于完整组织级 policy |
| Policy / Approval Enforcement | 决定动作是否允许、是否需要确认、是否触发审查 | 不等于真正的环境隔离；错误配置时仍可能被放宽 |
| Environment Hard Boundary | 强制限制文件、网络、工作域、执行位置 | 不解释业务意图，也不负责上层比较细的 reasoning 约束 |

这张表最想防止的一种误判是：

> 只要系统“有 rules / skills / approvals”，就等于这些规则已经真正进入执行面。

事实上，很多约束只停在前两层，离“真正拦动作”还很远。

---

## 五、用现有对象校验这五层模型

### 5.1 Claude Code

当前主干与官方文档已能支持的最稳判断：

- Prompt-Level Guidance：session instructions、memory、skills 会和主循环一起进入上下文
- Capability Surface：基础工具 + 扩展工具 + MCP tools 共同形成可调用能力面
- Tool-Loop Interception：官方 hooks 明确支持 `PreToolUse`、`PermissionDenied`、`PostToolUseFailure`、`PostToolBatch` 等
- Policy / Approval Enforcement：权限系统独立存在，写入控制、命令限制、风险分级明确存在
- Environment Hard Boundary：当前公开材料更适合写成“存在执行治理与权限边界”，但不宜外推成已公开完整 sandbox contract

这说明 Claude Code 在前四层都有比较清晰的可观察面，但最底层“环境硬边界”仍不宜写满。

### 5.2 Codex

当前主干与官方文档已能支持的最稳判断：

- Prompt-Level Guidance：thread / goal / hooks / memories 等可进入执行上下文
- Capability Surface：内置执行能力、MCP server、tool config 形成能力面
- Tool-Loop Interception：hooks、background terminals、status inspection 等说明其存在运行中控制面
- Policy / Approval Enforcement：官方明确 `approval_policy`、`approvals_reviewer`、per-tool approval mode
- Environment Hard Boundary：官方明确 sandbox applies to spawned commands，且有 `read-only`、`workspace-write`、`danger-full-access`

Codex 是当前最适合用来区分“approval policy”和“sandbox hard boundary”不是同一层的对象。

### 5.3 MCP

MCP 很适合拿来校验“能力面不等于执行治理”这条边界：

- Capability Surface：非常强，`tools/list`、`tools/call`、schema、structured result 都清楚
- Tool-Loop Interception：较弱，协议本身不定义 host-side hook engine
- Policy / Approval Enforcement：只有 transport-level authorization 与 human-in-the-loop 建议，不等于 per-tool approval policy
- Environment Hard Boundary：几乎不承接，只通过 `roots` 等提供有限工作域边界语义

所以 MCP 最容易被误写成“已经解决工具治理”，其实它主要解决的是 **protocol surface**。

### 5.4 Tool Executor 这条线和本专题的关系

`tool-executor-boundary.md` 更关注：

- 一次工具调用如何被执行
- 结果如何回流
- stop-line 停在哪

本文更关注：

- skill / rules / hooks / approvals / sandbox 分别在哪层约束执行
- 哪些是软约束，哪些是硬约束
- 哪些约束属于 Tool Executor，哪些已经超出它

两篇是邻近问题，但不重复。

---

## 六、最容易出现的五种误判

### 6.1 有 skill 就等于 skill 已进入执行面

很多 skill 仍只是 prompt-level procedure，并没有进入 tool-loop、approval 或 runtime boundary。

### 6.2 有 hooks 就等于有硬边界

hook 可以拦截、记录、延迟、拒绝，但如果没有环境级边界兜底，它仍不是最强约束。

### 6.3 有 approval 就等于动作一定安全

approval 只是治理放行层；真正副作用还能不能发生，仍取决于执行环境与实际实现。

### 6.4 有 sandbox 就等于系统已经可治理

sandbox 只解决最底层硬边界；规则是否解释清楚、结果是否可追踪、tool-loop 是否可拦截，仍是上层问题。

### 6.5 有 protocol 就等于有完整执行 contract

tool schema、structured result、transport、authorization 可以很成熟，但仍不等于 runtime、cleanup、rollback、permission 和 evaluation 已经闭环。

---

## 七、下一轮最值得继续补证的问题

1. skill 到底何时只是 prompt，何时已成为可治理制品
2. tool-loop interception 和 approval policy 的 stop-line 如何区分
3. environment hard boundary 和 Tool Executor stop-line 的交界在哪里
4. result lifecycle、audit trail、post-hoc evaluation 如何形成闭环，而不只是事后记录
5. 哪些系统真正实现了“规则进入执行面”，哪些只是“规则进入上下文” 

## Evidence

- Status: `Observed / Inferred`
- Sources:
  - `../../../backlog.md`
  - `./tool-executor-boundary.md`
  - `../02-coding-agents-and-tools/claude-code/architecture-deep-dive.md`
  - `../04-skill-and-tool-systems/mcp/overview.md`
  - 基准来源：`https://code.claude.com/docs/en/`；后续引用：`overview`、`hooks-guide`、`agent-sdk/agent-loop`
  - 基准来源：`https://developers.openai.com/codex/`；后续引用：`agent-approvals-security`、`concepts/sandboxing`、`cli/slash-commands`
  - `https://developers.openai.com/codex/llms-full.txt`
- Trace: 从顶层 `agentic/backlog.md` 中“Skill、Rules 与执行约束如何真正进入 Agent Runtime”这一跨目录问题出发，先不急着定义实现方案，而是先区分执行约束的层级、约束强度与 stop-line；本轮优先用 Claude Code、Codex 与 MCP 作为第一批高信号对象，建立 prompt guidance、tool contract、tool-loop interception、approval enforcement 与 environment hard boundary 五层模型。
- Needs:
  - 补更多对象对照，特别是 OpenHands、SWE-agent、Cursor / Cline 一类 coding agent
  - 补 skill 何时从 prompt 升格为可治理制品的更细判断标准
  - 补 post-hoc evaluation、audit trail 与 runtime governance 的衔接边界

*最后更新: 2026-06-18*
