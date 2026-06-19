# Action-Capable Agent Governance

> 适用范围：具备代码执行、工具调用、文件修改、环境副作用或 workspace mutation 的复杂 agent。
> 阶段状态：第一版问题框架，先回答“如何把约束写进执行系统”，不冻结具体架构实现。
> 使用说明：本文关注的不是文本层建议，而是 agent 已经能够调用工具、修改文件、运行命令、接触环境时，系统如何用更强的机制约束、指导和审计这些动作。

## 一、为什么这类 agent 需要单独研究

一旦 agent 可以：

- 调用工具
- 运行命令
- 写文件
- 操作 workspace / worktree
- 访问网络
- 派发 subagent

问题就不再只是“模型有没有理解规则”，而是：

> **即使模型局部判断失误，系统还能不能把它限制在可接受边界内。**

因此，这类治理研究的重点从“提高遵守率”转向：

- 拦截不该发生的动作
- 限制已允许动作的副作用范围
- 让执行过程可追踪、可中断、可清理
- 把 approval、sandbox、runtime、audit 组合成真正的执行治理

---

## 二、这类系统真正想约束的是什么

复杂 agent 主要想约束的是以下几类外部行为：

- 工具调用是否合理
- 命令执行是否越界
- 文件修改是否落在许可工作域内
- 网络与依赖操作是否受限
- 长任务是否可中断、可检查、可恢复
- 子 agent 是否在可控边界内协作

因此，它的核心不再只是认知约束，而是**执行约束 + 环境约束 + 治理约束**。

---

## 三、复杂 agent 的六类治理机制

### 3.1 Capability Exposure

第一层是：系统到底暴露了什么能力给模型。

关键问题包括：

- 哪些 tools 可见
- 哪些能力被隐藏或按需暴露
- schema 是否足够清楚
- 是否区分 read / write / exec / network 等不同能力

如果 capability surface 一开始就过宽，后面的治理压力会急剧上升。

### 3.2 Tool-Loop Interception

第二层是：当模型已经形成 tool call 时，系统能否在执行前后介入。

典型机制包括：

- deny / ask / defer
- tool input rewrite
- pre-tool validation
- post-tool audit
- batch-level interception
- failure hook

这是把规则真正推进执行链的第一层硬化步骤。

### 3.3 Approval / Permission Governance

第三层是：动作是否允许继续，不只由某个 hook 决定，而是由稳定的治理策略决定。

典型问题包括：

- 哪些动作默认允许
- 哪些动作必须确认
- 哪些动作完全禁止
- side-effecting action 怎么分级
- per-tool / per-server / per-workspace 的 policy 是否不同

这一层的目标不是解释动作，而是稳定决定“放不放行”。

### 3.4 Runtime / Workspace Binding

第四层是：动作到底在什么环境中发生。

关键问题包括：

- 当前 workspace / repo / root 是什么
- 是否在 worktree 中执行
- 是否是本地环境、subprocess、独立 venv、container 或 remote sandbox
- 输出产物写到哪里
- session / workspace 与 tool execution 是什么关系

这是执行治理与环境治理真正接壤的地方。

### 3.5 Environment Hard Boundary

第五层是最强约束层：即使上面几层出错，环境边界仍能限制动作。

典型形式包括：

- read-only
- workspace-write
- container isolation
- network isolation
- filesystem root restriction
- worktree isolation

这层的特点是：它不是建议，也不是审批，而是直接决定动作做不做得到。

### 3.6 Audit / Recovery Surface

第六层关注的是：动作发生后，系统还能不能解释、终止、清理和归因。

关键问题包括：

- result / error / progress 如何回流
- background task 如何观察与停止
- cleanup entry 是否明确
- rollback / compensation 是不是存在
- audit trail 是否足以追踪规则失效点
- post-hoc evaluation 能否定位到具体执行层

这一层不等于完整恢复系统，但它决定系统能不能在出错后保持可治理。

---

## 四、什么算“把约束写进执行系统”

对于复杂 agent，真正科学的机制，不是把更多规则塞进 prompt，而是至少做到下面四点：

1. **能力面受控**：不是所有能力都默认暴露给模型
2. **执行前可拦截**：tool call 在落地前可以被 deny / ask / defer / rewrite
3. **执行中有硬边界**：sandbox、workspace、network、runtime 至少有一层系统级限制
4. **执行后可归因**：结果、错误、副作用、停止与清理面是可追踪的

如果缺少这四点，很多所谓“agent rule”其实仍只是文字建议。

---

## 五、最常见的失效模式

### 5.1 规则只停在 prompt，没有进入 tool loop

模型知道“最好别这样做”，但系统在真正执行前没有任何拦截机制。

### 5.2 approval 和 sandbox 被混写成同一层

实际上 approval 决定“要不要放行”，sandbox 决定“就算放行了还能做多少”。

### 5.3 tool contract 很清楚，但 side-effect boundary 不清楚

参数和返回值都结构化了，但文件、网络、依赖、副作用边界仍然模糊。

### 5.4 有 runtime 隔离，但没有 audit / cleanup 面

系统能把动作困在环境里，却不能在失败后稳定解释或清理现场。

### 5.5 有后验评估，但定位不到执行链哪一层失效

最后只知道结果不好，却不知道是 prompt、tool loop、approval、runtime，还是 sandbox 出的问题。

---

## 六、和纯文本 skill agent 的边界

本文不重点回答：

- skill procedure 应该怎么写
- 纯文本输出结构怎么约束
- 自检 loop 如何提高遵守率
- 长上下文中规则如何保持可见

这些更适合纯文本 skill 治理专题。

本文重点回答：

- 规则如何真正拦动作
- 执行链上哪些层在做治理
- 哪些约束是软的，哪些是硬的
- 复杂 agent 如何把“听话”变成“越不了界”

---

## 七、下一轮最值得继续补证的问题

1. Tool-loop interception 和 approval policy 的 stop-line 如何稳定区分
2. runtime binding 与 environment hard boundary 的交界在哪里
3. cleanup entry、rollback、compensation action 应该分几层写
4. subagent 行为应继承哪些规则，哪些需要单独治理
5. post-hoc evaluation 如何回流到具体执行层，而不是只给一个总失败结论

## Evidence

- Status: `Observed / Inferred`
- Sources:
  - `../../../backlog.md`
  - `./execution-governance-layers.md`
  - `./tool-executor-boundary.md`
  - `../02-coding-agents-and-tools/claude-code/architecture-deep-dive.md`
  - `../04-skill-and-tool-systems/mcp/overview.md`
  - 基准来源：`https://code.claude.com/docs/en/`；后续引用：`hooks-guide`、`agent-sdk/agent-loop`
  - 基准来源：`https://developers.openai.com/codex/`；后续引用：`agent-approvals-security`、`concepts/sandboxing`、`cli/slash-commands`
  - `https://developers.openai.com/codex/llms-full.txt`
- Trace: 从顶层“Skill、Rules 与执行约束如何真正进入 Agent Runtime”这一跨目录问题继续下沉，把具备代码执行、工具调用和环境副作用的复杂 agent 单独抽出来，优先研究如何把约束写进执行系统；本轮先形成能力暴露、tool-loop interception、approval、runtime binding、environment hard boundary 与 audit/recovery 六层治理框架。
- Needs:
  - 补 OpenHands、SWE-agent、Cursor / Cline 等对象对照
  - 补 cleanup / rollback / compensation action 的更细边界
  - 补 subagent 继承规则与跨 agent 治理的专题

*最后更新: 2026-06-18*
