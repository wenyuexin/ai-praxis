# OpenCode Context and Configuration

> 本文解释 OpenCode 如何通过配置层级、`AGENTS.md`、instructions、skills、compaction 和文件读取路径，把项目知识注入 coding agent runtime。MCP、ACP、plugins、custom tools、SDK 的扩展边界另见 `ecosystem-and-extensions.md`。

## 核心判断

OpenCode 的上下文工程不是单一 repo map，而是一组 context source 和治理层叠加：

- 全局、远程、项目级和显式配置共同决定 runtime 行为。
- `AGENTS.md`、可选 `CLAUDE.md`、deprecated `CONTEXT.md` 和 config `instructions` 共同进入系统上下文。
- 读取文件时会发现嵌套 instruction，把局部目录规则追加到 read tool 输出中。
- Skills 作为可按需加载的能力材料进入系统上下文。
- Compaction、watcher、LSP、file attachments 等机制处理上下文规模和变化。

## 配置层级

OpenCode 配置来源包括：

- global config。
- remote organization default config。
- explicit `OPENCODE_CONFIG`。
- project `opencode.json` / `opencode.jsonc`。
- `.opencode` config directories。
- explicit `OPENCODE_CONFIG_DIR`。
- `OPENCODE_CONFIG_CONTENT`。

项目配置会从当前目录向 worktree 向上查找，再按从远到近的顺序合并，使靠近当前目录的配置覆盖更上层配置。数组合并中 `instructions` 有特殊处理：会拼接和去重，而不是简单替换。

配置 schema 覆盖 agent、permission、provider、lsp、formatter、watcher、compaction、instructions、command、skills、reference 等域；MCP 与 plugin 也通过配置进入系统，但它们的扩展生命周期和职责边界由 `ecosystem-and-extensions.md` 单独承接。这里的重点是：配置不只是偏好设置，而是 runtime governance 的核心入口。

## `/init` 与 `AGENTS.md`

OpenCode 的 `/init` 是内置 command，描述为 guided `AGENTS.md` setup。源码显示它注入初始化 prompt，把工作区路径传给模型，引导生成或更新项目规则文件。

规则加载并不只依赖 `/init`：session instruction discovery 会查找 `AGENTS.md`，可选支持 `CLAUDE.md`，并兼容 deprecated `CONTEXT.md`。全局 instruction 采用“第一个存在的全局文件优先”；项目 instruction 则按文件类别选择，并沿目录向上收集匹配文件。

这使 OpenCode 的项目规则机制具备两个特点：

- `/init` 是生成入口，不是唯一来源。
- `AGENTS.md` 是后续 session context 的长期输入 artifact。

## Nested Instructions

当 read tool 读取某个文件时，OpenCode 会从该文件所在路径向上查找尚未加载的 instruction 文件，并把它们作为 `<system-reminder>` 附加到 read 输出中。

这个设计让局部目录规则在模型真正接触相关文件时才进入上下文，避免一开始加载整个仓库所有规则，同时保留 scoped instruction 的语义。

## System Context 与 Prompt Assembly

OpenCode 的系统上下文会包含模型、cwd、worktree、VCS 状态、平台和日期等基础信息。不同模型族还会选择不同 prompt template。

Prompt attachments 会被转换为 synthetic read/resource text；文件附件会通过 read tool 读取，并可绕过 cwd 检查。这说明附件不是 UI 层文本拼接，而会进入工具化上下文路径。

## Skills 与能力材料

Skills 在系统上下文中以可用列表呈现，正文按需通过 `skill` 工具加载。它们更像可调用的知识、流程或专业能力说明，而不是独立 agent 或外部协议工具。

这使 OpenCode 可以把“当前 agent 应该知道有哪些专业流程”与“真正把某个 skill 内容加载进上下文”分开处理。MCP、custom tools 和 plugins 则属于扩展生态层，见 `ecosystem-and-extensions.md`。

## Compaction 与 Context Epoch

OpenCode 有 compaction 配置和 session compaction 逻辑。源码和设计文档中还出现 System Context、Session History、Context Source、Context Epoch 等术语，说明项目正在把上下文变化建模为更明确的 provider-turn 边界和 epoch 状态。

当前需要保守表述：Context Epoch 是设计方向和部分实现线索，不应把它写成所有默认路径都已完全迁移的稳定结论。

## 设计意义

OpenCode 的 context/configuration 机制体现了一种重要路径：把项目知识、组织策略、工具边界和模型输入分开管理，但在 agent runtime 中汇合。

这对 coding agent 工程化有三点启发：

- 项目规则应该成为可版本化 artifact，而不是只存在于 prompt 中。
- 局部目录规则应该按访问触发，而不是全部预加载。
- 配置层级既是便利性问题，也是企业治理和安全边界问题。

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - <https://opencode.ai/docs/zh-cn/config>
  - <https://github.com/anomalyco/opencode>
  - `packages/opencode/src/config/config.ts`
  - `packages/opencode/src/config/paths.ts`
  - `packages/core/src/v1/config/config.ts`
  - `packages/opencode/src/command/index.ts`
  - `packages/opencode/src/session/instruction.ts`
  - `packages/opencode/src/session/system.ts`
  - `packages/opencode/src/session/prompt.ts`
  - `packages/opencode/src/session/compaction.ts`
  - `packages/opencode/src/tool/read.ts`
  - `packages/opencode/src/skill/index.ts`
  - `CONTEXT.md`
- Trace: 本文将 `backlog.md` 中“/init 与项目规则注入”“配置层级”“上下文、压缩与文件监视”回流为机制专题；MCP / ACP / 插件边界已拆到 `ecosystem-and-extensions.md`。
- Needs: 后续可补 scoped instruction、`/init` 实际输出、配置合并冲突和 compaction 行为的运行样例。
