# Evidence 规则

本文件定义当前仓库的知识可信度规则。

相关入口：[`CONTRIBUTING.md`](../../CONTRIBUTING.md) / [`docs/contributing/README.md`](./README.md) / [`Evidence 与 Traceability 工作流`](./evidence-and-traceability.md) / [`Traceability 规则`](./traceability-rules.md)。

Evidence 负责回答：**这条知识凭什么成立？**

## 1. 适用范围

本规则适用于进入主题目录的知识内容，尤其是：

- `overview.md` 中的领域判断、分类体系、关键 trade-off
- 正文专题中的定义、机制解释、方法分类、评价结论
- `backlog.md` 中的待补主题和缺口判断
- `candidates.md` 中的待研究对象和研究理由
- `conflict.md` 中的冲突记录、证据与待核验问题

不要求对以下内容做完整 Evidence 标注：

- README 的目录导航和路径说明
- 纯模板、渲染测试、格式示例
- 明显的文件组织说明
- 临时草稿中尚未准备回流主线的原始摘录

## 2. 基本对象

### 2.1 Claim

Claim 是正文中可以被判断真伪、强弱或适用边界的知识陈述。

常见 Claim 包括：

- 定义：某个概念是什么
- 分类：某类系统可以如何划分
- 因果关系：某机制会导致某现象
- 对比判断：A 相比 B 的优势、劣势或适用条件
- 趋势判断：某方向正在成为主流或仍处早期
- 工程建议：在某条件下应采用某设计

不必把每句话都登记为 Claim。优先标注会影响读者理解、判断或后续建设方向的关键结论。

### 2.2 Evidence

Evidence 是支持 Claim 的材料或观察。

常见 Evidence 来源包括：

- 论文原文
- 官方文档
- GitHub 源码、release notes、issue、PR
- benchmark 或实验报告
- 高质量技术博客或社区讨论
- 仓库内已有文档和用户调研上下文
- AI 协作者或作者基于多材料形成的综合判断

## 3. Evidence 状态

### Verified

可追溯到强一手材料，且 Claim 能被材料直接支撑。

适用来源：

- 论文原文中的方法、实验结果、定义或结论
- 官方文档中的功能说明、接口行为、版本说明
- 源码或 release notes 中可直接观察到的实现行为
- 多个高质量来源相互印证的事实

使用要求：

- 标明来源名称或路径
- 能说明 Claim 与来源之间的对应关系
- 避免把来源没有覆盖的外推结论也标为 Verified

### Observed

在具体系统、项目、案例或多个二手来源中观察到，但尚不足以成为稳定通用结论。

适用场景：

- 某个开源项目采用了某种设计
- 多篇博客或社区讨论反复提到同一问题
- 某个 benchmark 或产品案例显示出某种趋势

使用要求：

- 必须说明观察对象和范围
- 不应写成“行业共识”或“通用规律”
- 如要进入正文，应保留适用范围说明

### Inferred

基于已有材料、仓库上下文或相邻主题推导出的结论，但没有直接证据完全支撑。

适用场景：

- 从多个文档中归纳分类框架
- 基于已有实现和概念关系推断设计 trade-off
- 将用户调研上下文整理成主线结构

使用要求：

- 必须说明推断依据
- 不应伪装成论文或官方结论
- 后续应进入 evidence backlog 等待补证

### Unverified

作为工作假设、待验证观点或临时结构占位存在。

适用场景：

- AI 协作者先行整理的主线草稿
- 尚未找到来源的概念归纳
- 需要后续论文、官方文档或开源实现验证的判断

使用要求：

- 不能写成稳定定论
- 应明确标注待验证方向
- 优先放入 `backlog.md`、`candidates.md` 或正文的 `Open Questions`

### Conflicting

不同来源之间存在术语、事实、版本、适用边界或结论力度冲突。

使用要求：

- 优先记录到 `conflict.md`
- 标明冲突来源和冲突类型
- 不应在正文中直接选择一方作为定论，除非解释选择理由

### Deprecated

曾经成立，但因版本变化、来源失效或新证据出现而不再适用。

使用要求：

- 保留废弃原因
- 指向替代结论或新来源
- 必要时从正文移入历史说明或删除

## 4. 来源类型与使用边界

| 来源类型 | 默认强度 | 可支持的内容 | 注意事项 |
|---|---|---|---|
| 论文原文 | 高 | 方法、实验、定义、理论主张 | 不要把单篇论文外推成领域共识 |
| 官方文档 | 高 | 功能、接口、版本行为、设计意图 | 注意版本和发布日期 |
| 源码 / release notes | 高 | 实际实现、变更、工程行为 | 注意 commit、tag、分支 |
| issue / PR | 中 | 设计讨论、问题背景、维护者解释 | 区分正式结论和讨论过程 |
| benchmark / 实验报告 | 中到高 | 评测结果、任务定义、指标 | 注意实验设置和适用范围 |
| 技术博客 / 社区讨论 | 中到低 | 趋势、痛点、经验观察 | 不直接支持强事实定论 |
| 仓库内已有文档 | 中 | 本仓库上下文、已有分类和建设方向 | 要区分仓库约束与领域知识 |
| AI / 作者综合判断 | 低 | 结构草稿、初步分类、待验证假设 | 默认标为 Inferred 或 Unverified |

### 4.1 研究开源代码库时的版本规则

对于具有分支、tag、release 和持续迭代历史的开源代码库，**不允许把一次静态源码观察写成无版本边界的长期定论**。

最小要求：

- 基于代码得出的 Claim，必须说明观察基线至少属于哪一种：`branch`、`tag/release`、`commit`。
- 如果 Claim 依赖源码当前实现而不是官方稳定文档，优先记录到具体 `commit`；只有明确是发布版本结论时，才优先记录 `tag` 或 release。
- `commit` 应使用至少 7 位简写或完整 hash，确保在目标仓库中可唯一还原。
- 如果不同分支可能功能不同，不要只写“该仓库支持 X”，而应写成“在某个 branch/tag/commit 上观察到 X”。
- 如果同一分支会持续演进，不要把 `main` / `master` 上一次观察写成长期事实；应同时记录观察时间，必要时补充“高漂移风险”。
- 如果结论来自 release notes 与源码联合判断，要说明两者是否对应同一版本窗口；不要把不同时间点的材料拼成一个无时间边界的结论。

推荐写法（只保留最小示例）：

- `Observed on branch main at commit abc123 (observed_at: 2026-06-08)`

以下情况不能写成稳定主线结论：

- 只看了当前默认分支的一次代码截面，却没有写 branch / commit / 观察时间。
- 混用了不同分支、不同 tag、不同 release notes 的信息，却没有说明版本边界。
- 把 issue / PR 中讨论中的未来功能写成当前分支已具备的事实。
- 把某个 feature branch、实验目录或未发布 PR 的能力写成该项目总体能力。

## 5. 先决定写多少 Evidence

Evidence 标注的对象是关键 Claim，不是每个普通句子。

### 5.1 不需要标注的内容

以下内容通常不需要 Evidence 标注：

- 目录导航
- 文件位置说明
- 章节过渡句
- 常识性背景句
- 仅复述本文结构的句子

### 5.2 必须标注的内容

以下内容必须有 Evidence 状态：

- 文档核心定义
- 主要分类框架
- 关键机制解释
- 重要 trade-off
- 趋势判断
- 工程建议
- 安全、可靠性、评估相关结论
- 从 `temp/`、AI 调研或模型综合判断回流到正文的内容

### 5.3 最低门槛与推荐密度

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

## 6. Evidence 写在哪里

Evidence 不默认新增独立元信息文档。

当前仓库优先采用“写在被支撑内容所在文档里”的方式：

- 正文专题：在同一篇正文文档中写 `## Evidence` 小节。
- `overview.md`：在文档末尾写 `## Evidence` 小节，支撑其中的关键分类、判断和 trade-off。
- `backlog.md` / `candidates.md`：在具体条目内部用字段标注 evidence need 或 status。
- `conflict.md`：在具体冲突条目中记录当前证据、冲突来源和待核验问题。
- README：一般不写 Evidence，除非 README 中包含实质性判断，而不只是目录导航。

只有当某个主题的证据记录变多、正文 `## Evidence` 小节明显过长时，才考虑新增主题级 evidence registry。当前不要求为每个目录机械创建 evidence 元信息文档。

对于具体案例研究，`notes/evidence.md` 或 `evidence-notes.md` 可以作为该案例内部的证据对照层：它从 `notes/general.md`、`notes/source.md` 等研究过程材料中提炼 Claim-Source 对照，服务案例正文的关键判断。它仍不属于七层元信息文件，也不替代正文 `## Evidence` 摘要；正文应保留关键证据结论，并在证据过长时链接到对应的案例 evidence notes。

### 6.1 主题级 evidence registry 的启用条件

Evidence registry 不是默认元信息文件，也不是每个目录都要补齐的额外层。启用条件和命名建议见 `evidence-and-traceability.md` 第 8 节。

## 7. Evidence 放在文档什么位置

### 7.1 默认位置：文档末尾集中写

普通正文专题默认在正文主体之后、`Open Questions` 或参考资料之前，集中放一个 `## Evidence` 小节。

适用场景：

- 一篇文档主要围绕一个主题展开
- 关键结论数量不多
- Evidence 说明可以集中阅读，不影响正文流畅性

最小示例（完整组合见 `evidence-and-traceability.md` 第 5 节）：

```md
## Evidence

- Status: Inferred
- Sources: `agentic/temp/ai-context.md`
- Trace: First-pass synthesis from repository context.
- Needs: External validation.
```

### 7.2 局部位置：紧跟高风险段落

当某个段落包含高风险判断时，可以紧跟段落下方写短标注。

高风险判断包括：

- 数值、性能、规模、趋势判断
- 安全、可靠性、合规判断
- 明确声称“主流”“行业共识”“最佳实践”
- 容易被误解为官方结论的归纳

最小示例：

```md
Evidence: Observed; needs official docs or implementation examples.
```

局部标注应短，不展开长表格；详细来源仍可放到文末 `## Evidence`。

### 7.3 条目位置：写在 backlog / candidates / conflict 条目内部

对于元信息文件，不再额外建全局 Evidence 小节，而是在条目内部标注。

```md
- Topic: Trust calibration in agent UI
  - Status: Unverified
  - Evidence need: HCI papers and product docs
```

## 8. Claim 写在哪里

Claim 不默认写到单独文件。

Claim 应写在它支撑的知识文档里，位置取决于文档类型：

| 文档类型 | Claim 写法 | 位置 |
|---|---|---|
| 正文专题 | `## Evidence` 小节中的 Claim 表格或摘要 bullet | 文档末尾集中写，必要时局部短标注 |
| `overview.md` | 支撑核心分类、边界、trade-off 的 Claim 列表 | 文档末尾 `## Evidence` |
| `backlog.md` | 待验证主题不是稳定 Claim，写成 `Topic` + `Status` + `Evidence need` | 条目内部 |
| `candidates.md` | 研究理由不是稳定 Claim，写成对象条目的 `Why` / `Evidence need` | 条目内部 |
| `conflict.md` | 冲突双方都可以是 Claim，写明来源和冲突类型 | 冲突条目内部 |

## 9. Claim 怎么写

### 9.1 Claim 表格

适合概念密集、争议较多、或者一篇文档包含多个关键判断的情况。若模型容易漏字段，优先直接复用这一列顺序：`Claim / Status / Sources / Notes`。

```md
## Evidence

| Claim | Status | Sources | Notes |
|---|---|---|---|
| Workspace traceability should include task, action, state, artifact and actor. | Inferred | `workspace-traceability.md` | Needs validation. |
```

### 9.2 摘要式 bullet

适合普通正文专题。它比表格轻，不打断正文；对能力较弱的模型，也更容易直接照写。

```md
## Evidence

- Main claim: Inferred from current repository context.
- Sources: `agentic/temp/ai-context.md`
- Trace: First-pass synthesis.
- Needs: External validation.
```

### 9.3 局部短标注

适合只想提醒读者"这句还没完全验证"的高风险段落。若模型容易漏字段，优先改用文末 `## Evidence`，不要在正文里发明新格式。

```md
Evidence: Unverified synthesis; needs official docs or implementation examples.
```

## 10. Claim 写作要求

一个 Claim 应尽量满足。对能力较弱的模型，最稳妥的写法是先写一句 Claim，再补 `Status`，然后补 `Sources` 或 `Needs`：

- **可判断**：能被来源支持、反驳或限定范围。
- **不太宽**：不要写成"Agent UI 很重要"这类空泛判断。
- **有边界**：说明适用对象、系统范围或待验证范围。
- **有状态**：必须标注 `Verified / Observed / Inferred / Unverified / Conflicting / Deprecated` 之一。
- **有来源或缺口**：有来源就写 Sources；没有来源就写 Needs。

## 11. AI 协作者要求

与 `evidence-and-traceability.md` 中的 AI 协作者要求一致，详见该文件第 2 节。

## 12. 后续扩展方向

当前先实现文档级 Evidence 标注。后续可按需要扩展：

- 主题级 evidence registry
- claim id 与 source id
- 跨主题证据复用索引
- 证据冲突与废弃记录
- 来源版本、commit、发布日期等结构化字段
