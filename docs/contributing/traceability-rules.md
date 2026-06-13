# Traceability 规则

本文件定义当前仓库的轻量内容链路规则。

相关入口：[`CONTRIBUTING.md`](../../CONTRIBUTING.md) / [`docs/contributing/README.md`](./README.md) / [`Evidence 与 Traceability 工作流`](./evidence-and-traceability.md) / [`Evidence 规则`](./evidence-rules.md)。

Traceability 负责回答：**这条内容从哪里来、为什么落在这里？**

## 1. 当前定位

当前仓库采用轻量 Traceability。

它不要求为每个 Claim 建复杂对象模型，也不追踪每一次编辑历史；编辑历史交给 Git。

它只记录对知识治理有用的链路：

- 输入材料来自哪里
- 为什么选择当前落位
- 是否从 `temp/`、`backlog.md`、`candidates.md` 或 `conflict.md` 回流
- 当前还缺哪类证据或验证

## 2. 基本对象

### 2.1 Trace

Trace 是内容进入仓库的轻量链路记录。

它关联输入材料、处理决策、目标位置和未解决缺口。

### 2.2 Trace Unit

当前最小 Trace 单位不是每一句话，而是一次有治理意义的内容落位。

常见 Trace Unit：

- 一篇正文专题的新增
- 一个 `overview.md` 判断的补充
- 一个 `backlog.md` 条目的新增
- 一个 `candidates.md` 对象的登记
- 一个 `conflict.md` 冲突的记录
- 从 `temp/` 回流到主线的内容迁移

## 3. 四个最小字段

### 3.1 Source

记录内容输入来自哪里。

来源标识应优先使用对读者和协作者可共享、可复现的引用形式。对于当前仓库内的材料，正式 Trace 应优先记录仓库内相对路径；对于 GitHub 等开源社区的公开仓库，即使实际核验发生在本机本地副本上，正式 Trace 也应优先记录上游仓库 URL，而不是机器私有的本地文件系统路径。

在满足“可共享、可复现”前提下，引用表达可以保持适度灵活，避免机械重复冗长前缀：如果一篇文档会反复引用同一个外部仓库、文档站或其他外部资源，可以先在文中声明一次上游基准 URL，后续条目使用“基准 URL + 相对路径 / 相对资源路径”的写法；只要读者能够稳定还原到完整来源，这种缩写式引用是允许的。禁止的是机器私有路径和无法还原来源的模糊简称，不是对外部 URL 的合理去重。

示例：

- `agentic/temp/ai-context.md`
- `docs/contributing/documentation-workflow.md`
- 某篇论文笔记
- `https://github.com/openai/codex`
- 基准来源：`https://github.com/OpenHands/OpenHands`；后续引用：`frontend/src/routes/conversation.tsx`、`openhands/app_server/sandbox/remote_sandbox_service.py`
- 基准来源：`https://docs.openhands.dev/sdk/`；后续引用：`arch/workspace`、`guides/convo-persistence`
- AI 协作者基于已有目录的综合整理

### 3.2 Decision

记录为什么这样处理。

常见 Decision：

- 进入正文：证据较稳定，适合作为主线知识
- 进入 overview：适合建立领域全貌，但不适合作为单篇专题
- 进入 backlog：问题重要但材料不足
- 进入 candidates：对象值得研究，但尚未产出结论
- 进入 conflict：存在冲突，不能直接写成定论
- 保留 temp：材料不稳定或来源不足

### 3.3 Placement

记录落位位置。

示例：

- `agentic/07-evaluation/observability-and-debugging/failure-attribution.md`
- `agentic/04-human-agent-interaction/backlog.md`
- `agentic/05-environments/conflict.md`

### 3.4 Gap

记录还缺什么证据或验证。

示例：

- 缺官方文档验证
- 缺论文原文支撑
- 缺开源实现对照
- 缺跨项目比较
- 缺版本边界说明

### 3.5 代码库版本链路字段

**Stop-line：只要结论基于源码观察，就不要省略 `Version Basis` 和 `Observed At`。**

当 Source 指向开源代码库、release notes、issue / PR 或源码文件时，除 `Source / Decision / Placement / Gap` 外，Trace 可补版本链路信息；如果正文太轻，可写在 `notes/evidence.md`、`source-notes.md` 或条目内部：

- `Version Basis`：本次判断基于 `branch`、`tag/release`、`commit` 中的哪一种，并包含可还原的具体标识，例如 `branch main`、`tag v0.14.0` 或至少 7 位 commit hash。
- `Observed At`：核验日期，而不是文件系统最后修改时间。
- `Scope`：结论适用于当前默认分支、某个 release、某个历史 commit，还是仅适用于实验分支。
- `Drift Risk`：可选；当对象处于快速迭代期时，可标 `high / medium / low`。

最低要求：

- 基于源码实现写结论时，至少给出 `Version Basis` 和 `Observed At`。
- 如果引用的是 release notes 或 tag，对应源码观察应尽量指向同一版本窗口。
- 如果只观察了当前默认分支，必须让读者看出这是“当前截面观察”，不是无条件长期定论。

## 4. 标注方式

### 4.1 正文专题

正文专题中可以在 `Evidence` 小节内合并记录 Trace：

```md
## Evidence

- Status: Inferred
- Sources: `agentic/temp/ai-context.md`
- Trace: First-pass synthesis from repository context.
- Needs: External validation.
```

### 4.2 Backlog / Candidates

`backlog.md` 或 `candidates.md` 中可以按条目记录 Trace。字段顺序尽量稳定，便于能力较弱的模型照着填：

```md
- Topic: Trust calibration in agent UI
  - Source: Current overview gap
  - Decision: Keep as backlog
  - Placement: `04-human-agent-interaction/trust-and-alignment/`
  - Gap: Need HCI papers
```

### 4.3 Conflict

`conflict.md` 中的 Trace 应重点记录冲突来源和待核验问题。最少保留 `Sources / Decision / Placement / Gap` 这组骨架，不要随意省字段：

```md
- Conflict: Workspace traceability core unit
  - Sources: `workspace-traceability.md`
  - Decision: Keep conflict open
  - Placement: `05-environments/conflict.md`
  - Gap: Need cross-system comparison
```

## 5. 与 Evidence 的关系

Traceability 不替代 Evidence。

- Evidence 判断 Claim 是否可信。
- Traceability 记录内容如何进入仓库。
- 一段内容可以有 Trace，但 Evidence 很弱。
- 一段内容可以 Evidence 很强，但仍需要 Trace 说明为什么落在当前位置。

当前执行时，优先保证 Evidence 状态清楚，同时用轻量 Trace 记录 Source / Decision / Placement / Gap。

## 6. 不做什么

当前不做以下重型实现：

- 不为每个句子建立 Trace ID
- 不手工维护完整编辑历史
- 不复制 Git 已经能提供的版本历史
- 不建立全仓库 lineage 图
- 不要求所有旧文档一次性补齐 Trace

## 7. 后续扩展方向

如果后续 Traceability 独立复杂化，可以扩展：

- Trace ID
- 内容迁移记录
- temp → backlog / candidates / conflict → 正文的回流链路
- 跨文档引用关系
- Claim 与文档版本之间的 lineage
- 主题级 trace registry
