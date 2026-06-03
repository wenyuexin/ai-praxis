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
- 是否从 `temp/`、`backlog.md`、`research-queue.md` 或 `conflict.md` 回流
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
- 一个 `research-queue.md` 对象的登记
- 一个 `conflict.md` 冲突的记录
- 从 `temp/` 回流到主线的内容迁移

## 3. 四个最小字段

### 3.1 Source

记录内容输入来自哪里。

示例：

- `agentic/temp/ai-context.md`
- `agentic/05-environments/backlog.md`
- 某篇论文笔记
- 某个官方文档或开源仓库
- AI 协作者基于已有目录的综合整理

### 3.2 Decision

记录为什么这样处理。

常见 Decision：

- 进入正文：证据较稳定，适合作为主线知识
- 进入 overview：适合建立领域全貌，但不适合作为单篇专题
- 进入 backlog：问题重要但材料不足
- 进入 research-queue：对象值得研究，但尚未产出结论
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

## 4. 标注方式

### 4.1 正文专题

正文专题中可以在 `Evidence` 小节内合并记录 Trace：

```md
## Evidence

- Status: Inferred
- Sources: `agentic/temp/ai-context.md`, existing `05-environments/` documents
- Trace: Synthesized from repository context to define a first-pass taxonomy.
- Needs: External validation from papers, official docs, or open-source implementations.
```

### 4.2 Backlog / Research Queue

`backlog.md` 或 `research-queue.md` 中可以按条目记录 Trace：

```md
- Topic: Trust calibration in agent UI
  - Source: Current `04-human-agent-interaction/` overview gap
  - Decision: Keep as backlog until external evidence is collected
  - Placement: `04-human-agent-interaction/trust-and-alignment/`
  - Gap: Need HCI papers and product documentation
```

### 4.3 Conflict

`conflict.md` 中的 Trace 应重点记录冲突来源和待核验问题：

```md
- Conflict: Workspace traceability core unit
  - Sources: `workspace-traceability.md`, `traceability-object-model.md`
  - Decision: Keep conflict open until cross-system mappings are verified
  - Placement: `05-environments/conflict.md`
  - Gap: Need comparison across multiple agent runtime systems
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
- temp → backlog / research-queue / conflict → 正文的回流链路
- 跨文档引用关系
- Claim 与文档版本之间的 lineage
- 主题级 trace registry
