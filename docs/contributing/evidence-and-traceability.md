# Evidence 与 Traceability 工作流

本文件是 Evidence 与 Traceability 的组合执行入口，不重复定义两套规则。

相关入口：[`CONTRIBUTING.md`](../../CONTRIBUTING.md) / [`docs/contributing/README.md`](./README.md) / [`Evidence 规则`](./evidence-rules.md) / [`Traceability 规则`](./traceability-rules.md)。

核心策略：**正式 Evidence + 轻量 Traceability**。

- Evidence 负责回答：这条知识凭什么成立？
- Traceability 负责回答：这条内容从哪里来、为什么落在这里？

## 1. 文件职责

`evidence-rules.md` 和 `traceability-rules.md` 是并列的原子规则文件：

- [`evidence-rules.md`](./evidence-rules.md)：定义 Claim、Evidence、Evidence 状态、Evidence 写法和 Evidence 放置位置。
- [`traceability-rules.md`](./traceability-rules.md)：定义 Trace、Trace Unit、Source / Decision / Placement / Gap 和轻量标注方式。

本文件只处理两者一起执行时的顺序和最低门槛，避免形成第三套 Evidence 或 Traceability 规则。

## 2. 为什么需要组合入口

新增或修改知识文档时，Evidence 和 Traceability 通常同时发生：

- 判断 Claim 是否可信时，需要知道来源和证据状态。
- 判断内容该放在哪里时，需要知道材料来源、处理决策和证据缺口。
- 从 `temp/`、外部 AI 调研或网页搜索回流正文时，既不能缺 Evidence 状态，也不能缺 Trace 说明。

因此贡献者可以先读本文件获得最小流程，再进入两个原子规则文件查细则。

## 3. 最小执行流程

新增或修改知识文档时，按以下顺序执行：

```text
材料输入
  → 判断材料类型和目标落位
  → 识别关键 Claim
  → 判断 Evidence 状态
  → 写入正文、元信息文件或继续保留在 temp/
  → 记录 Sources / Trace / Needs
```

执行原则：

- 无法判断 Evidence 状态时，默认标为 `Unverified`。
- 无法说明 Source 时，不要写成正文定论。
- 无法说明 Placement 理由时，优先放入 `backlog.md` 或 `candidates.md`。
- 存在来源冲突时，优先进入 `conflict.md`，不要直接在正文中选边。

## 4. 最小字段组合

正文或元信息条目中至少应能回答：

```md
- Status: Inferred / Observed / Verified / Unverified / Conflicting / Deprecated
- Sources: 支撑材料、仓库上下文或输入文档
- Trace: 内容如何从输入材料进入当前文件
- Needs: 还缺什么证据、验证或对照
```

字段归属：

- `Status`、`Sources` 的细则见 [`evidence-rules.md`](./evidence-rules.md)。
- `Trace`、`Needs` 的细则见 [`traceability-rules.md`](./traceability-rules.md)。
- 同一文档反复引用同一个外部仓库或文档站时，可以先声明一次基准 URL，后续用相对路径补充；前提是读者仍能还原完整来源。

## 5. `temp/` 回流门槛

`temp/` 内容默认只是输入材料，不是主线知识。迁移出 `temp/` 前，至少满足：

- 已判断材料类型：整体认知、主题缺口、待研究对象、冲突问题或正文专题材料。
- 已判断目标落位：`overview.md`、`backlog.md`、`candidates.md`、`conflict.md` 或某篇正文专题。
- 已判断 Evidence 状态：至少能标为 `Observed` 或 `Inferred`；只能标为 `Unverified` 时，不得写成正文定论。
- 已记录 Trace：能说明 `Source / Decision / Placement / Gap` 或等价信息。
- 已说明缺口：写清还缺论文、官方文档、源码实现、社区观察或跨系统比较中的哪一类。

最小回流标注：

```md
## Evidence

- Status: Inferred
- Sources: `agentic/temp/ai-context.md`
- Trace: Moved from temporary context into a first-pass mainline topic.
- Needs: External validation from papers, official docs, or open-source implementations.
```

## 6. 适用边界

本文件不负责：

- 重新定义 Evidence 状态、Claim 写法或 evidence registry 条件；见 [`evidence-rules.md`](./evidence-rules.md)。
- 重新定义 Source / Decision / Placement / Gap 或 Trace Unit；见 [`traceability-rules.md`](./traceability-rules.md)。
- 说明 `overview.md`、`backlog.md`、`candidates.md`、`conflict.md` 的完整职责；见 [`metadata-files.md`](./metadata-files.md)。
- 说明案例研究、机制专题、`notes/` 和 `notes/evidence.md` 的组织方式；见 [`research-artifacts.md`](./research-artifacts.md)。

## 7. 当前落地策略

当前仓库采用分阶段落地：

1. 新增文档先执行本工作流。
2. 已有文档不做一次性大规模返工。
3. 优先补齐新增主线文档的 Evidence 小节和必要 Trace。
4. 对明显由 AI 综合写成、但缺少来源的内容，先标为 `Inferred` 或 `Unverified`，再逐步补证。
5. 当某个主题的 Evidence 或 Trace 记录变多，再考虑建立主题级 registry；不要一开始为全仓库建立过重索引。
