# Evidence 记录规则

本文件回答：**完成 Evidence 判断后，是否需要记录，记录在哪里，以及用什么格式呈现？**

组合流程见 [`evidence-and-traceability.md`](./evidence-and-traceability.md)；Claim 和 Evidence Status 见 [`evidence-assessment-rules.md`](./evidence-assessment-rules.md)；来源类型边界见 [`evidence-source-rules.md`](./evidence-source-rules.md)。

## 1. Evidence 标注的适用范围与密度

Evidence 标注的对象是关键 Claim，不是每个普通句子。

### 1.1 不需要标注的内容

以下内容通常不需要 Evidence 标注：

- 目录导航
- 文件位置说明
- 章节过渡句
- 常识性背景句
- 仅复述本文结构的句子

### 1.2 必须标注的内容

以下内容必须有 Evidence 状态：

- 文档核心定义
- 主要分类框架
- 关键机制解释
- 重要 trade-off
- 趋势判断
- 工程建议
- 安全、可靠性、评估相关结论
- 从 `temp/`、AI 调研或模型综合判断回流到正文的内容

### 1.3 最低门槛与推荐密度

最低门槛和推荐密度不是同一件事：

- **最低门槛**：一篇进入主线的正文专题，至少要有 1 个 `## Evidence` 小节，并说明整体 `Status / Sources / Trace / Needs`。
- **推荐密度**：一篇普通正文专题，建议覆盖 3–7 条关键 Claim 或摘要式证据说明。
- **局部标注**：通常只保留 0–2 个，用于高风险段落。

如果只有 1 条 Evidence 说明，该文档可以作为低配主线草稿存在，但通常应标为 `Inferred` 或 `Unverified`，并写明缺什么证据。

如果超过 10 条 Claim，优先考虑：

- 合并相近 Claim
- 只保留影响结论的 Claim
- 把待补证内容移入 `backlog.md`
- 后续再考虑主题级 evidence registry

如果完全没有 Evidence 说明，通常说明文档还没有达到主线正文可信度要求；不要把它写成稳定定论。

## 2. Evidence 的承载层级

Evidence 不默认新增独立元信息文档。

当前仓库优先采用“写在被支撑内容所在文档里”的方式：

- 正文专题：在同一篇正文文档中写 `## Evidence` 小节。
- `overview.md`：在文档末尾写 `## Evidence` 小节，支撑其中的关键分类、判断和 trade-off。
- `backlog.md` / `candidates.md`：在具体条目内部用字段标注 evidence need 或 status。
- `conflict.md`：在具体冲突条目中记录当前证据、冲突来源和待核验问题。
- README：一般不写 Evidence，除非 README 中包含实质性判断，而不只是目录导航。

只有当某个主题的证据记录变多、正文 `## Evidence` 小节明显过长时，才考虑新增主题级 evidence registry。当前不要求为每个目录机械创建 evidence 元信息文档。Evidence registry 的元信息身份见 [`metadata-files.md`](./metadata-files.md)。

对于具体案例研究，`notes/evidence.md` 或 `evidence-notes.md` 可以作为该案例内部的证据对照层：它从 `notes/general.md`、`notes/source.md` 等研究过程材料中提炼 Claim-Source 对照，服务案例正文的关键判断。它仍不属于七层元信息文件，也不替代正文 `## Evidence` 摘要；正文应保留关键证据结论，并在证据过长时链接到对应的案例 evidence notes。

### 2.1 主题级 Evidence registry 的启用条件

Evidence registry 不是默认元信息文件。仅当证据对照本身已经成为主题维护负担时，才按需启用，例如：

- 单篇正文的 `## Evidence` 已超过约 10 条关键 Claim，继续放在正文中会明显影响阅读。
- 多篇正文反复引用同一批来源，或需要长期维护版本、适用范围、废弃状态或证据冲突。
- 证据对照本身已经成为读者需要查询的对象，而不只是正文结论的附属说明。

以下情况不应启用 Evidence registry：

- 只有少量来源：直接写入正文 `## Evidence`。
- 只是发现待补证方向：进入 `backlog.md` 或 `candidates.md`。
- 只是存在来源冲突：进入 `conflict.md`。
- 只是为了目录完整：不要机械创建 `evidence.md`。

名称应体现主题或作用范围，例如 `<topic>-evidence.md`；覆盖整个目录时可使用 `evidence-registry.md`，不默认使用通用 `evidence.md`。registry 中的证据仍需保留 `Status / Sources / Trace / Needs` 或等价字段，并指向正文中的关键 Claim；registry 不替代正文解释或其他元信息文件。

## 3. 文档内的 Evidence 位置

### 3.1 默认位置：文档末尾集中写

普通正文专题默认在正文主体之后、`Open Questions` 或参考资料之前，集中放一个 `## Evidence` 小节。

适用场景：

- 一篇文档主要围绕一个主题展开
- 关键结论数量不多
- Evidence 说明可以集中阅读，不影响正文流畅性

具体格式使用 §4.2 的摘要式 bullet；不要在位置规则中维护第二份模板。

### 3.2 局部位置：紧跟高风险段落

当某个段落包含高风险判断时，可以紧跟段落下方写短标注。

高风险判断包括：

- 数值、性能、规模、趋势判断
- 安全、可靠性、合规判断
- 明确声称“主流”“行业共识”“最佳实践”
- 容易被误解为官方结论的归纳

局部标注使用 §4.3 的单行格式，应保持简短，不展开长表格；详细来源仍放到文末 `## Evidence`。

### 3.3 条目位置：写在 backlog / candidates / conflict 条目内部

对于元信息文件，不再额外建全局 Evidence 小节，而是在条目内部标注。

```md
- Topic: Trust calibration in agent UI
  - Status: Unverified
  - Evidence need: HCI papers and product docs
```

## 4. Claim 证据记录的呈现格式

### 4.1 Claim 表格

适合概念密集、争议较多、或者一篇文档包含多个关键判断的情况。若模型容易漏字段，优先直接复用这一列顺序：`Claim / Status / Sources / Trace / Needs`。

```md
## Evidence

| Claim | Status | Sources | Trace | Needs |
|---|---|---|---|---|
| Evidence and Traceability need distinct roles. | Inferred | `docs/contributing/rules/evidence-and-traceability.md` | Synthesized into this document. | Validate in the next documentation task. |
```

### 4.2 摘要式 bullet

适合普通正文专题。它比表格轻，不打断正文；对能力较弱的模型，也更容易直接照写。

```md
## Evidence

- Status: Inferred
- Sources: `docs/contributing/rules/evidence-assessment-rules.md`; `docs/contributing/rules/traceability-rules.md`
- Trace: Synthesized the current Evidence and Traceability rules into this document.
- Needs: Validation against the next concrete documentation task.
```

### 4.3 局部短标注

适合只想提醒读者“这句还没完全验证”的高风险段落。若模型容易漏字段，优先改用文末 `## Evidence`，不要在正文里发明新格式。

```md
Evidence: Unverified synthesis; needs official docs or implementation examples.
```
