# Evidence 与 Traceability 工作流

本文件是当前仓库知识质量流程的统一入口。

相关入口：[`CONTRIBUTING.md`](../../CONTRIBUTING.md) / [`docs/contributing/README.md`](./README.md) / [`Evidence 规则`](./evidence-rules.md) / [`Traceability 规则`](./traceability-rules.md) / [`元信息文件模型`](./metadata-files.md)。

核心策略：**正式 Evidence + 轻量 Traceability**。

- Evidence 负责回答：这条知识凭什么成立？
- Traceability 负责回答：这条内容从哪里来、为什么落在这里？

## 1. 为什么保留统一入口

Evidence 和 Traceability 分别维护细则，但在执行时不应割裂。

新增或修改知识文档时，贡献者需要同时判断：

- 这条内容是否有证据支撑
- 这条内容的 Evidence 状态是什么
- 这条内容来自哪里
- 为什么落在当前文件
- 后续还缺什么证据或验证

因此本文件保留为工作流入口，避免读者只看 Evidence 或只看 Traceability。

## 2. 规则分工

| 文件 | 回答的问题 | 当前实现强度 |
|---|---|---|
| [`evidence-rules.md`](./evidence-rules.md) | 这条知识凭什么成立？ | 正式规则 |
| [`traceability-rules.md`](./traceability-rules.md) | 这条内容从哪里来、为什么落在这里？ | 轻量规则 |
| [`metadata-files.md`](./metadata-files.md) | 不同元信息文件分别承接什么认知任务？ | 正式规则 |

## 3. 最小执行流程

新增或修改知识文档时，按以下顺序执行：

```text
材料输入
  → 判断 Claim 类型
  → 判断 Evidence 状态
  → 选择落位
  → 写入正文或元信息文件
  → 添加 Evidence 小节或条目标注
  → 记录轻量 Trace：Source / Decision / Placement / Gap
```

如果无法判断 Evidence 状态，默认降级为 `Unverified`。

如果无法说明 Source，默认不要写成正文定论。

如果无法说明 Placement 理由，优先放入 `backlog.md` 或 `research-queue.md`。

## 4. 最小字段组合

正文或元信息条目中至少应能回答：

```md
- Status: Inferred / Observed / Verified / Unverified / Conflicting / Deprecated
- Sources: 支撑材料、仓库上下文或输入文档
- Trace: 内容如何从输入材料进入当前文件
- Needs: 还缺什么证据、验证或对照
```

其中：

- `Status` 和 `Sources` 由 [`Evidence 规则`](./evidence-rules.md) 约束。
- `Trace` 和 `Needs` 由 [`Traceability 规则`](./traceability-rules.md) 约束。

## 5. `temp/` 回流主线条件

`temp/` 内容默认只是输入材料，不是主线知识。只有同时满足以下条件，才可以迁移出 `temp/`：

- **材料类型已明确**：已经判断它是整体认知、主题缺口、待研究对象、冲突问题，还是正文专题材料。
- **目标落位已明确**：已经知道应进入 `overview.md`、`backlog.md`、`research-queue.md`、`conflict.md`，还是某篇正文专题。
- **Evidence 状态已明确**：至少能标为 `Observed` 或 `Inferred`；如果只能标为 `Unverified`，不得写成正文定论。
- **Trace 字段已明确**：能写清 `Source / Decision / Placement / Gap`。
- **缺口已明确**：如果证据不足，必须写明还缺论文、官方文档、开源实现、社区观察或跨系统比较中的哪一类。

回流方式按 Evidence 状态区分：

- `Verified`：可进入 `overview.md` 或正文专题，仍需保留来源说明。
- `Observed`：可进入正文，但必须保留观察对象、适用范围和限制；不应写成通用规律。
- `Inferred`：可进入正文作为结构化归纳或主线草稿，但必须显式标注推断依据和待补证方向。
- `Unverified`：不得进入正文定论；优先留在 `temp/`，或转入 `backlog.md` / `research-queue.md` 作为待验证线索。
- `Conflicting`：优先进入 `conflict.md`，等待核验后再决定是否修改正文。
- `Deprecated`：不回流正文定论，只能作为历史说明或废弃记录。

最小回流标注：

```md
## Evidence

- Status: Inferred
- Sources: `agentic/temp/ai-context.md`
- Trace: Moved from temporary context into a first-pass mainline topic.
- Needs: External validation from papers, official docs, or open-source implementations.
```

## 6. 与现有文件的关系

- `CONTRIBUTING.md`：保留仓库级强约束和本工作流入口。
- `docs/contributing/`：承接 Evidence、Traceability、README、文档工作流等细则。
- `overview.md`：沉淀领域全貌和相对稳定的结构化判断。
- `backlog.md`：记录待补主题、待验证方向和证据缺口。
- `research-queue.md`：记录值得研究的具体对象。
- `conflict.md`：记录冲突来源和待核验问题。
- `temp/`：保留尚未分级、尚未验证、尚未落位的临时材料。

## 7. 当前落地策略

当前仓库采用分阶段落地：

1. 新增文档先执行本工作流。
2. 已有文档不做一次性大规模返工。
3. 优先补齐 `agentic/04-human-agent-interaction/`、`agentic/05-environments/`、`agentic/07-evaluation/` 中新增主线文档的 Evidence 小节。
4. 对明显由 AI 综合写成、但缺少来源的内容，先标为 `Inferred` 或 `Unverified`，再逐步补论文、官方文档、开源实现或社区观察。
5. 当某个主题的 Evidence 或 Trace 记录变多，再考虑建立主题级 registry；不要一开始为全仓库建立过重索引。
