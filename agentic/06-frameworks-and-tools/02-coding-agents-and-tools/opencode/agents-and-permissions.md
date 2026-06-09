# OpenCode Agents and Permissions

> 本文解释 OpenCode 如何用 agent、subagent、tool registry 和 permission system 组合出 Plan / Build、人类门控与任务分工。

## 核心判断

OpenCode 的 Plan / Build 更像“agent 配置与权限边界”，而不是两个完全独立的执行器。agent 定义决定可用工具、默认权限、prompt reminder 和可见性；工具执行时再通过 permission request 把敏感动作交给人类确认。

这套设计让 human-in-the-loop 分成两层：

- **模式级门控**：Plan / Build 选择不同 primary agent。
- **动作级门控**：工具调用触发 allow / ask / deny。

## Agent 模型

OpenCode 的 agent 信息包含名称、描述、模式、权限、模型、prompt、选项等字段。模式包括：

- `primary`：可作为用户当前会话模式直接交互。
- `subagent`：由主 agent 通过 task 或提及调用。
- `all`：可同时作为两类用途的自定义 agent。

内置 agent 包括：

- **build**：默认 primary agent，面向开发执行，允许更完整的工具能力。
- **plan**：受限 primary agent，面向分析和计划，限制编辑与命令执行。
- **general**：subagent，面向复杂搜索和多步任务。
- **explore**：subagent，面向只读代码探索和调研。
- **compaction / title / summary**：隐藏内部 primary agent，用于压缩、标题、摘要等系统任务。

配置文件和 agent markdown 文件可以覆盖或新增 agent；配置也可以禁用内置 agent。

## Plan / Build 机制

Plan / Build 的实现由三部分组合：

1. **内置 agent 权限**：Plan 的权限比 Build 更受限，默认限制 file edits 和 bash 等动作。
2. **session reminder**：进入 Plan 时注入只读/计划提示；从 Plan 切到 Build 时注入模式切换提示。
3. **可选 experimental plan mode**：更严格地要求计划文件，并通过 `plan_exit` 工具引导切换回 Build。

因此，Plan / Build 的关键不只是 UI 上的模式切换，而是 agent identity、permission rules 和 prompt context 共同改变了模型的行动空间。

## Subagent 与任务分工

OpenCode 的 subagent 主要通过 `task` 工具启动。`task` 工具要求指定 `subagent_type`、描述和 prompt，也支持 resume 已有 subagent session。

这让主 agent 可以把复杂搜索、代码库探索和多步研究委派给权限更窄或提示更专门的 agent。General 与 Explore 的差异尤其明显：General 面向复杂多步任务，Explore 更偏只读探索。

## Skill 与 Agent 的边界

Skill 不是 agent 本身，而是可被 `skill` 工具加载的能力材料。系统上下文会列出可用 skills，但 skill 正文在需要时通过工具加载。

这意味着：

- agent 决定当前执行身份、权限和任务边界。
- skill 提供可调用的专业知识或工作流材料。
- custom tool / MCP tool 提供外部动作能力。

三者共同构成 OpenCode 的能力扩展模型。

## Permission System

OpenCode 的权限系统支持 allow / ask / deny。权限既可以来自默认配置，也可以来自 agent、session input、项目配置和具体工具规则。工具执行时通过 `ctx.ask(...)` 发起权限请求；统一的 permission service 再评估规则，并在需要时发出 permission asked 事件。

权限键覆盖常见工具和安全防护：

- 文件与搜索：read、edit、glob、grep、lsp。
- 执行与代理：bash、task、skill。
- 网络与上下文：webfetch、websearch、external_directory。
- 人类交互与安全：question、doom_loop、plan_enter、plan_exit。

`.env` 默认拒绝、外部目录默认 ask、doom loop 默认 ask 等规则显示 OpenCode 把安全边界放在工具层，而不是只依赖模型自律。

## 设计取舍

这种设计的优点是组合性强：同一个 agent loop 可以通过 agent 配置、工具注册、权限规则和 reminder 组合出不同工作模式。缺点是行为边界分散在多个层面：agent 定义、config schema、tool registry、permission evaluator、session reminder 都可能影响实际能力。

因此，分析 OpenCode 的某个能力时，不能只读工具实现或只读 agent 文档；必须同时看 agent permission、tool registry 和 session prompt/reminder。

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - <https://opencode.ai/docs/zh-cn/agents>
  - <https://opencode.ai/docs/zh-cn/permissions>
  - <https://opencode.ai/docs/zh-cn/tools>
  - `packages/opencode/src/agent/agent.ts`
  - `packages/opencode/src/agent/subagent-permissions.ts`
  - `packages/opencode/src/config/agent.ts`
  - `packages/core/src/v1/config/agent.ts`
  - `packages/opencode/src/session/reminders.ts`
  - `packages/opencode/src/session/prompt.ts`
  - `packages/opencode/src/tool/task.ts`
  - `packages/opencode/src/tool/plan.ts`
  - `packages/opencode/src/tool/registry.ts`
  - `packages/opencode/src/permission/index.ts`
  - `packages/opencode/src/permission/evaluate.ts`
  - `packages/opencode/src/skill/index.ts`
- Trace: 本文将 `backlog.md` 中“Plan / Build”“工具权限”“Agent / Subagent / Skill”缺口合并为一个机制专题。
- Needs: 后续可通过一次实际 Plan → Build → permission ask → subagent task 的运行记录补充行为证据。
