# academic-research-skills deep-research 深度分析

**日期**: 2026-05-17 (updated)  
**来源**: [academic-research-skills](https://github.com/Imbad0202/academic-research-skills) v3.8.2  
**目标**: 深入理解 deep-research 技能的设计理念、架构与实现方式

> 本文档基于实际读取的 Agent prompt 原文、SKILL.md 编排细节、Material Passport schema 真实内容、以及 PERFORMANCE.md 官方性能数据编写。

---

## 1. 综述：这是一个纯 prompt-as-code 系统

经过深度阅读代码和全部 13 个 Agent 的 prompt 文件后，最根本的发现是：

**deep-research 没有任何 Python/TypeScript 代码驱动管线。** 整个系统的"编排层"就是 SKILL.md 文件本身。当用户触发 deep-research 时，LLM（Claude Code）读取 SKILL.md 的完整指令，按照 6 阶段工作流**自主**顺序调用各 Agent 的 prompt。没有 LangChain、没有 LangGraph、没有 Function Calling 触发式调度——全部由一个长 context 的 LLM 会话完成。

这与常规认知（以为有某种代码编排框架）截然不同。

---

## 2. 管线编排的真实机制

### 2.1 无代码编排

真实答案是：**由 LLM 自身根据 SKILL.md 指令编排**。

SKILL.md 中的编排工作流（§Orchestration Workflow）以 ASCII 流程图和自然语言描述了完整流程：

```
User: "Research [topic]"
     |
=== Phase 1: SCOPING ===
     |-> [research_question_agent] -> RQ Brief
     |-> [research_architect_agent] -> Methodology Blueprint
     +-> [devils_advocate_agent] -- CHECKPOINT 1
     ** User confirmation before Phase 2 **
```

LLM 的推理过程大致为：
1. 读取 SKILL.md → 确定当前在 Phase 1
2. （在 prompt context 中）"扮演" research_question_agent，产出 RQ Brief
3. 切换 persona 扮演 research_architect_agent，产出 Methodology Blueprint
4. 切换 persona 扮演 devils_advocate_agent，运行 CHECKPOINT 1
5. 输出检查结果，等待用户确认
6. 进入 Phase 2，重复上述过程

所有 Agent 间的**状态传递通过会话文本本身完成**——一个 Agent 的输出文本成为下一个 Agent 输入 context 的一部分。没有序列化、没有消息队列、没有共享内存。

### 2.2 并行评审的真实含义

Phase 5 被描述为"并行评审"。经过深入阅读 SKILL.md 和编排指令后，确认了物理执行方式。

#### 物理执行方式：单请求内依次扮演

Phase 5 的编排在 SKILL.md 中以 ASCII 流程图描述为 `+->` 分支符号，但没有找到任何 `<parallel>`、`<sequential>`、`<invoke>` 等编排标记。

真实执行方式为：**同一个 API 请求中，LLM 依次扮演 3 个角色，全量 context 保留**。

```
同一个 LLM session（同一个 API 请求）
  ┌─────────────────────────────────────────┐
  │  context: 完整论文草稿 + SKILL.md 指令      │
  │                                          │
  │  Step 1 → 扮演 editor_in_chief_agent     │
  │           产出: Editorial Verdict          │
  │           输出追加到 context               │
  │                                          │
  │  Step 2 → 扮演 ethics_review_agent       │
  │           输入: 原始论文草稿 + Step 1 输出   │
  │           产出: Ethics Clearance           │
  │           输出追加到 context               │
  │                                          │
  │  Step 3 → 扮演 devils_advocate_agent     │
  │           输入: 原始论文草稿 + Step 1+2 输出 │
  │           产出: DA Checkpoint 3 报告      │
  │                                          │
  │  输出: 3 份评审报告的合并结果               │
  └─────────────────────────────────────────┘
```

#### 关键推论

1. **不存在独立的 API 调用**。3 个角色共享同一个 LLM 推理上下文。
2. **状态传递方式**：所有中间输出追加到同一段 context 文本中。LLM 从历史对话文本中自行识别需要输入的部分，没有显式的状态序列化。
3. **`+->` 符号的含义**：评审维度独立，但执行引擎（LLM）是同一个。这是流程设计层面的并行（no coupling between reviews），不是执行层面的并行。
4. **与 scripts/adapters 的对比**：仓库中 scripts/adapters/ 下确实有 Python 代码，但这些代码是**用户在自己的环境中运行的适配器脚本**（生成 YAML 文件），不是管线编排代码。它们运行在 ARS 管线之外，产出 Material Passport 的输入文件。
5. **进程级并行的唯一实例**：如果设置了 `ARS_CROSS_MODEL`，devils_advocate_agent 在生成报告后会**额外**调用一个独立模型（如不同提供商）进行第二意见。这个调用是真正独立的 API 请求，但仅限于 DA 一个 Agent，且是可选功能。

### 2.3 状态跟踪机制

状态不是由代码维护，而是由 `state_tracker_agent`（另一个 prompt 文件）维护。它定义了状态跟踪的 JSON schema：

```json
{
  "pipeline_id": "ars-20260517-xxxx",
  "current_stage": "scoping",
  "completed_stages": [],
  "materials": {
    "rq_brief": { "status": "pending", "produced_by": null },
    "methodology_blueprint": { "status": "pending" },
    "bibliography": { "status": "pending" },
    "synthesis": { "status": "pending" },
    "draft": { "status": "pending" },
    "review_report": { "status": "pending" }
  },
  "dialogue_log_ref": "turns #1..#15",
  "checkpoint_results": []
}
```

但注意：**这个 JSON 是 LLM 在对话中自行维护的文本**，不是写入数据库或文件的持久化数据。每次 LLM 推理时，从历史对话中重读这个 JSON 块来恢复状态。

---

## 3. 实际 Prompt 内容分析

### 3.1 socratic_mentor_agent.md（550 行）— 最精密的 prompt

这是整个 deep-research 中最大、最复杂的 prompt 文件。使用的 prompt engineering 技巧包括：

**CoT 内嵌于角色定义**：
```
You are the Socratic Mentor — a Q1 international journal editor-in-chief
with 20+ years of academic experience. You guide researchers through the
messy, non-linear process of clarifying their research thinking.
```
角色定义包含了完整的温度/风格/行为约束，本质上是 CoT 的变体——LLM 被要求始终以主编心智模型思考。

**隐式 ReAct**：
prompt 没有显式使用 ReAct 的"思考-行动-观察"循环，但通过以下机制实现了等价效果：
- **SCR 协议**（状态 → 挑战 → 反思）：每个对话步骤有明确的状态跟踪
- **4 类问题标记** `[Q:CLARIFY]`、`[Q:PROBE]`、`[Q:STRUCTURE]`、`[Q:CHALLENGE]`：相当于动作空间定义
- **分类平衡规则**：每 3 个连续问题必须包含至少 2 种类型 → 类似动作选择策略

**对话健康自检（Dialogue Health Indicator）**：
每 5 轮进行一次自我评估，检测 3 种病态模式：
```
[HEALTH-CHECK: Turn X | Agreement: Y/5 | Conflict-Avoidance: detected/clear
| Premature-Convergence: detected/clear | Intervention: none/injected-challenge/...]
```
这是为防止 LLM 训练奖励带来的 sycophancy 倾向——LLM 倾向于同意用户以获得"高质量"评分，但苏格拉底对话中这违背原则。

**意图检测层（Intent Detection Layer）**：
通过 6 个信号维度判断用户是"探索型"还是"目标导向型"：
- 探索型：最大轮数 40 → 60，禁用自动收敛，禁止导师主动总结
- 目标导向型：标准收敛规则

**INSIGHT 提取机制**：
使用 `[INSIGHT: ...]` 标签逐轮标记成熟观点。每个 INSIGHT 的判定标准严格定义——重述已知事实、简单同意、表面观察都不算。

**让步阈值协议（Concession Threshold Protocol）**：
当用户反驳 DA 的发现时，DA 不能自动让步。必须按 1-5 评分反驳质量：
| 分数 | 含义 | 行为 |
|------|------|------|
| 5 | 直接击中核心，新证据/逻辑 | 让步 |
| 3 | 部分相关但转移焦点 | 坚守 |
| 1 | 无证据的主张/诉诸权威 | 升级攻击 |

**收敛信号机制（Convergence Signals）**：
5 个信号（S1-S5）由 LLM **自我评估**检测——没有外部代码判断：
- S1: 论点清晰度
- S2: 反论点意识
- S3: 方法论理由
- S4: 范围稳定性
- S5: 自我校准

**可选阅读探针**：
通过环境变量 `ARS_SOCRATIC_READING_PROBE=1` 激活。在 Layer 2→3 转换时，对目标导向型用户，导师要求释义其首次引用论文的一段话。这是一个**诚实性探针**——非评估性门控，用户可跳过，不产生惩罚。

### 3.2 devils_advocate_agent.md（193 行）— 反方辩手的设计

核心设计亮点：

**三检查点门控**：在 Phase 1/3/5 设置强制检查点。每个检查点有预设的问题集。

**逻辑谬误检测**：内置 10 种学术研究中常见的逻辑谬误检测框架（确认偏误、诉诸权威、后此谬误、幸存者偏差等）。

**偏误检测框架**：分认知偏误和研究设计偏误两大类。

**让步阈值协议（此处原始定义）**：评分 1-5 的反驳评估 + 反阿谀规则：
- 从不因为用户反驳而让步
- 连续让步后门控提升到 5/5
- 让步率 > 50% 时暂停
- Frame-lock 检测——每轮检查是否有未质疑的前提

**跨模型 DA（可选）**：设置 `ARS_CROSS_MODEL` 时，将材料发送给另一个独立模型进行第二意见，防止单一模型的系统性盲点。

### 3.3 source_verification_agent.md（183 行）— 证据验证引擎

核心设计亮点：

**7 级证据层级**：I（系统评价/荟萃分析）→ VII（专家意见）。

**三重验证策略**：
- Tier 0: Semantic Scholar API（**100%** 覆盖）
- Tier 1: DOI 验证（**100%** 覆盖）
- Tier 2: WebSearch 抽查（**50%** 覆盖，优先验证 Tier 3/4 来源）

**DOI 不匹配检测**：处理一种已知的幻觉模式——DOI 指错（Compound Deception Pattern #5: DOI Misdirection）。当 Semantic Scholar 中 DOI 解析出的标题与引用标题 Levenshtein < 0.70 时，标记为 `DOI_MISMATCH`。

**验证结果状态**：
- `S2_VERIFIED`: Semantic Scholar 匹配
- `VERIFIED`: DOI 解析成功
- `PLAUSIBLE`: 无 DOI 但 WebSearch 确认存在
- `UNVERIFIABLE`: 所有方法均无法确认
- `FABRICATED`: 存在不存在证据 → CRITICAL

### 3.4 synthesis_agent.md（260 行）— 综合与引用可信度

核心设计亮点：

**四层引用发射（v3.7.3）**：
```
Smith (2024) <!--ref:smith2024--><!--anchor:page:14-->
```
- 可见层：author-year 标准格式
- 隐藏层 1：`<!--ref:slug-->` 引用键
- 隐藏层 2：`<!--anchor:kind:value-->` 定位锚（quote/page/section/paragraph/none）
这允许下游 audit agent 逐条验证引用是否真的支持对应声明。

**声明意图清单（v3.8）**：
在撰写前，synthesis_agent 必须预提交一份 JSON 格式的声明意图清单：
```json
{
  "claims": [{
    "claim_id": "C-001",
    "claim_text": "Preprint hallucinations survive into the published record at 85.3%.",
    "intended_evidence_kind": "empirical",
    "planned_refs": ["zhao2026"]
  }],
  "manifest_negative_constraints": [
    {"rule": "No unqualified causal language across the synthesis."}
  ]
}
```
这作为**预承诺基线**，供下游 `claim_ref_alignment_audit_agent` 对比实际输出，检测声明漂移（EMITTED_NOT_INTENDED）。

**反模式保护**：明确禁止 3 种综合失败模式（顺序总结、选择性引用、未解决矛盾）。

### 3.5 research_question_agent.md（186 行）— 问题工程

核心设计亮点：

**FINER 框架**：5 维度 1-5 评分，最低阈值平均 >= 3.0，单项 >= 2。

**苏格拉底模式分支**：在 socratic 模式下，该 Agent 行为完全改变——不直接产生 RQ Brief，而是通过 FINER 引导性问题帮助用户自己推导出研究问题。

---

## 4. 性能与成本数据

> 以下数据来自 academic-research-skills 仓库自带的 `docs/PERFORMANCE.md` 和 `hooks/hooks.json`（SessionStart 通知脚本），整合了 ARS 官方提供的成本估算。部分数据是"估算值，待实测验证"。**建议验证计划**：运行一次 `deep-research` quick 模式，记录实际 token 消耗和耗时，与官方估算（~30K 输入 + ~15K 输出，~$0.60）对比。如偏差在 ±20% 以内则可信任官方全量估算；如偏差超过 ±50% 说明实际场景与官方基准（15,000 词 / 60 篇参考文献）差异大，需按自身论文规模重新测算。

### 4.1 ARS 官方 token 和成本估算

ARS 仓库自带的 `docs/PERFORMANCE.md` 提供了以下官方数据，基于 ~15,000 词论文 + ~60 篇参考文献，在 **Claude Opus 4.7** 上运行：

| 技能/模式 | 输入 Tokens | 输出 Tokens | 估算成本 (Opus 4.7) |
|-----------|-----------|------------|-------------------|
| `deep-research` socratic | ~30K | ~15K | ~$0.60 |
| `deep-research` full | ~60K | ~30K | ~$1.20 |
| `deep-research` systematic-review | ~100K | ~50K | ~$2.00 |
| `academic-paper` plan | ~40K | ~20K | ~$0.80 |
| `academic-paper` full | ~80K | ~50K | ~$1.80 |
| `academic-paper-reviewer` full | ~50K | ~30K | ~$1.10 |
| `academic-paper-reviewer` quick | ~15K | ~8K | ~$0.30 |
| **Full pipeline (10 阶段)** | **~200K+** | **~100K+** | **~$4-6** |
| + 跨模型验证（可选） | +~10K (外部) | +~5K (外部) | +~$0.60-1.10 |

来源：PERFORMANCE.md 第11-23行。成本按 Anthropic 2026 年 4 月 API 定价计算。

### 4.2 deep-research 各阶段的成本分布

根据 PERFORMANCE.md，deep-research full 模式（~60K 输入 + ~30K 输出，~$1.20）是"单轮运行"成本。实际执行时需要注意：

1. **Context 随阶段增长**：Phase 1 → Phase 6 的 context 累积可达 ~200K+ tokens（当保留全部早期输出时）。注意：**~200K+ 是 10 阶段全管线（research → write → review → revise → finalize）的终态值**，单独的 deep-research full 模式（6 阶段）的终态 context 约为 ~100-150K tokens。PERFORMANCE.md 没有给出各 phase 的中间值。full 模式 + 10 阶段管线才会达到 200K+。
2. **Cache 命中率**：Anthropic prompt cache TTL 为 5 分钟。因管线是人工参与的（每个 Stage 等待用户确认），跨 checkpoint 的 cache 通常 miss。
3. **跨会话续跑**：`resume_from_passport=<hash>` 机制允许在新会话中继续，避免已膨胀的 context 费用（详见下方补充说明）。

### 4.3 影响成本的因素

| 因素 | 影响方向 | 说明 |
|------|---------|------|
| 论文字数 | 正相关 | 默认基准 ~15,000 词，更长论文按比例增加 |
| 参考文献数 | 正相关 | 默认基准 ~60 篇，每多 20 篇增加 ~5-10K 输入 |
| 修改轮数 | 正相关 | Phase 6 最多 2 轮，每轮增加 ~10-15K 输入 + ~8K 输出 |
| 对话深度 (socratic) | 正相关 | 探索型模式最大 60 轮 vs 目标导向型 40 轮 |
| 跨模型验证 | +~$0.60-1.10 | 仅当设置 `ARS_CROSS_MODEL` |
| 系统评价合规检查 | +~$0.15-0.30 | systematic-review 的 Stage 2.5/4.5 合规 agent 额外成本 |
| 文献语料预筛 | 线性增长 | ~3-5K 输入 per 50 条 corpus 条目 |

### 4.4 耗时估计

PERFORMANCE.md 未提供端到端耗时数据。基于模式特点和官方数据推断：

| 模式 | 预估耗时（人工参与） | 说明 |
|------|-------------------|------|
| socratic | 30-90 分钟 | 高度交互式，取决于用户响应速度 |
| full | 1-4 小时 | 6 阶段 + 3 个 DA checkpoints + 用户确认 |
| quick | 15-30 分钟 | 精简管线 |
| lit-review | 20-40 分钟 | 搜索 + 验证 + 综合 |
| systematic-review | 2-8 小时 | 包含 RoB 评估 + 荟萃分析 |
| fact-check | 10-15 分钟 | 仅来源验证 |

以上耗时是"模型推理时间 + 用户交互时间"的总和，**不是纯推理时间**。每个 checkpoint 都等待用户确认。纯推理时间约占 30-50%。

### 4.5 性能风险

- **上下文膨胀**：full 模式 6 阶段后 context 大幅增长，会增加推理延迟和输出质量衰减风险。建议使用 `ARS_PASSPORT_RESET=1` 在 checkpoint 触发 context 重置。
- **token 消耗波动**：不同论文/话题的文献检索结果差异极大，实际 token 消耗可能偏离基准 2-3 倍。
- **Opus 依赖**：ARS 官方明确推荐 Opus 4.7。使用 Sonnet 等较弱模型时成本降低但输出质量可能下降。

---

## 5. 上下文管理策略

由于无代码编排，context 管理完全依赖 LLM 平台的对话管理能力：

- **滑动窗口**（Claude Code 默认策略）：保留最近的 N 轮对话，丢弃早期轮次。这意味着早期 Agent 的输出可能在后继阶段被压缩或丢失。
- **摘要压缩**：prompt 中没有显式的摘要机制，但 Claude Code 本身可能对历史进行自动压缩。
- **context 增长**：500K tokens 的 context 会显著影响推理速度和输出质量。这是该架构的主要瓶颈。

`academic-pipeline` SKILL.md 的 `resume_from_passport=<hash>` 机制正是为了解决这个问题——通过 Material Passport 的持久化路径，允许跨会话执行管线。

### 5.1 滑动窗口与摘要压缩的影响

| 因素 | 影响 |
|------|------|
| Phase 1 输出丢失 | 如果 Phase 1（Scope）的输出在 Phase 4 时被压缩，report_compiler 可能缺少 RQ Brief 和方法论蓝图 |
| INSIGHT 丢失 | 苏格拉底模式早期轮次的 INSIGHT 标签可能被压缩，导致最终 Research Plan Summary 不完整 |
| DA 检查点结果丢失 | Phase 5 的 DA 报告可能遗漏早期检查点的发现 |
| 缓解措施 | `ARS_PASSPORT_RESET=1` 在 FULL checkpoint 执行 context 重置；`resume_from_passport` 跨会话恢复 |

### 5.2 苏格拉底模式的收敛可靠性

#### 收敛信号检测机制

收敛信号 S1-S5 的检测完全由 **LLM 自我评估**完成，没有外部代码判断。每个信号的具体判定标准来自 socratic_mentor_agent.md 原文：

| 信号 | 检测方式 | 判断标准 |
|------|---------|---------|
| S1 (论点清晰度) | LLM 判断用户是否能用一句清晰陈述研究问题 | 无指标词（"maybe"、"sort of"），非被动回答 |
| S2 (反论点意识) | LLM 判断用户是否主动提出反论点 | 自发提及（非被问到）2+ 个反论点 |
| S3 (方法论理由) | LLM 判断用户能否解释"为什么此方法优于其他" | 阐述"为什么选这个"而非仅"选了什么" |
| S4 (范围稳定性) | LLM 跟踪研究问题在过去 3 轮是否显著变化 | 核心问题的变体而非根本改变 |
| S5 (自我校准) | LLM 比较早期与后期承诺的准确性 | 后期预测更细致、更恰当、更具体 |

#### 伪收敛风险分析

**结论：存在伪收敛风险，程度中等。**

| 风险 | 可能性 | 影响 |
|------|--------|------|
| LLM 为结束对话而虚假标记收敛信号 | **中等** | LLM 被训练为"有用且追求完成"，存在提前结束的激励。但 SKILL.md 对此有明确约束：`Never give direct conclusions` 和收敛后输出完整摘要 |
| LLM 误判 S1-S4 为活跃 | **低-中等** | 问题较为明确（用户能否一句陈述 RQ），LLM 有一定判断力。但用户的模糊表述可能被错误解读为清晰 |
| LLM 未检测到 S5（错过用户进步） | **中等** | 需要跨轮次比较承诺质量，对 context 窗口敏感 |
| 用户"配合性"同意结束 | **高** | 用户可能礼貌性同意导师的结束建议，即使研究问题尚未充分收敛 |

#### 缓解措施

- **自我检测**：Dialogue Health Indicator 每 5 轮检查 Persistant Agreement / Conflict Avoidance / Premature Convergence 三个维度，防止 LLM 过度友好。
- **INSIGHT 数量门控**：如果 < 3 个 INSIGHT 就触发收敛，导师会被要求暂缓结束。
- **多轮要求**：每层至少 2 轮对话才可推进（Layer 5 至少 1 轮）。
- **强制推进**：每层最多 8 轮后自动进入下一层，防止无限发散。
- **用户退出选项**：在任何时候用户可以要求结束或切换模式。

#### Context 管理对收敛的影响

40-60 轮的对话量存在以下风险：
1. **早期 INSIGHT 丢失**：第 1-10 轮的 INSIGHT 可能在滑动窗口中被压缩。
2. **导师"遗忘"早期承诺**：SCR 协议的早期 Commitment Gate 可能不再可用。
3. **建议**：对话每 15-20 轮建议用户确认一次 INSIGHT 摘要。

#### 收敛可靠性总结

| 维度 | 评价 | 依据 |
|------|------|------|
| 收敛检测方式 | LLM 自我评估，无外部验证 | socratic_mentor_agent.md 中 Convergence Rules 明确为 self-assessment |
| 伪收敛风险 | 存在，但不致命 | 多重防御机制（健康检查、INSIGHT 门控、多轮要求） |
| 早期信息丢失 | 有风险 | Context 管理中滑动窗口压缩可能丢失早期 INSIGHT |
| 建议优化方向 | 加入客户端 INSIGHT 持久化 | 每轮对话后将 INSIGHT 写入独立文件，不依赖 context 保留 |

### 5.3 resume_from_passport 跨会话恢复机制

跨会话恢复是解决 context 膨胀的核心方案，以下技术细节来自 `docs/PERFORMANCE.md` 和 `academic-pipeline/SKILL.md` 和 `academic-pipeline/references/passport_as_reset_boundary.md`。

**passport 文件存储位置**：默认位于 `./passports/<slug>/` 目录下，或匹配当前工作目录中 `./material_passport*.yaml` 的文件。自定义路径可通过项目 `CLAUDE.md` 配置，或在调用 orchestrator 前由集成工具设置。

**hash 生成算法**：passport 文件的内容 hash（内容变更后 hash 不同）。orchestrator 通过扫描 passport 文件中 `kind: boundary` 条目来匹配传入的 `resume_from_passport=<hash>`。每个 FULL checkpoint 都会写入一个 `kind: boundary` 条目。

**恢复行为**：**完全重置 LLM context**。新 session 只加载 passport ledger（YAML 文件），**不重放旧 session 的任何对话轮次**。通过 `resume_from_passport=<hash>` 命令触发，可选 `stage=<n>` 覆盖起始阶段，`mode=<m>` 指定运行模式。

**resume 命令语法**：
```
resume_from_passport=<hash> [stage=<n>] [mode=<m>]
```

**使用建议**：
- **适合 reset 的场景**：长管线 >100K input tokens，阶段间依赖性强，或命中 5 分钟 cache TTL
- **不适合的场景**：短管线（<30K），或存在 passport 无法捕获的隐式状态（如苏格拉底对话分支）

> **待实测验证**：PERFORMANCE.md 注明了"Empirical token savings: measurement pending"，即跨session 恢复的实际 token 节省量尚未经过真实 systematic-review 运行测量。

---

## 6. Material Passport 的实际定义

Material Passport 是 ARS 套件的跨技能数据契约。其核心 schema 定义在 `shared/contracts/passport/literature_corpus_entry.schema.json`（257 行）：

```json
{
  "required": ["citation_key", "title", "authors", "year", "source_pointer"],
  "additionalProperties": false,
  "properties": {
    "citation_key": { "type": "string", "pattern": "^[A-Za-z][A-Za-z0-9_:-]*$" },
    "title": { "type": "string" },
    "authors": { "type": "array", "items": { "$ref": "#/$defs/csl_name" } },
    "year": { "type": "integer", "minimum": 1000, "maximum": 2100 },
    "source_pointer": { "type": "string" },
    "venue": { "type": "string" },
    "doi": { "type": "string", "pattern": "^10\\.[0-9]{4,9}/[^\\s]+$" },
    "obtained_via": {
      "type": "string",
      "enum": ["zotero-api", "zotero-bbt-export", "obsidian-vault", "folder-scan", "manual", "other"]
    },
    "abstract": { "type": "string" },
    "semantic_scholar_id": { "type": "string" },
    "verification_status": {
      "type": "string",
      "enum": ["S2_VERIFIED", "VERIFIED", "PLAUSIBLE", "UNVERIFIABLE", "FABRICATED"]
    },
    "contamination_signals": {
      "type": "object",
      "properties": {
        "co_author_leak": { "type": "boolean" },
        "veneer_reference": { "type": "boolean" },
        "uncited_source_overlap": { "type": "boolean" }
      }
    }
  }
}
```

关键设计：
- **`additionalProperties: false`**：严格 schema，不允许扩展字段
- **CSL-JSON 作者格式**：兼容 Zotero/Obsidian 等引用管理工具
- **`source_pointer`**：不存储 PDF 内容，只存储 URI 指针
- **信任链**：`verification_status` + `semantic_scholar_id` + `contamination_signals` 组成完整的证据溯源链路
- **持久化方式**：YAML 文件，由用户编写的适配器脚本生成，ARS 管线只读取不写入

### 6.1 研究成果进入评审阶段（数据向下游流转）

Material Passport 不仅是 deep-research 的产出物，也是下游 academic-paper 和 academic-paper-reviewer 的**输入契约**。数据在 ARS 管线中的流转路径：

```
deep-research (raw)
  → Material Passport (YAML) + 研究报告草稿
  → academic-paper 加载 Passport，撰写论文（redacted）
  → Integrity Check（Stage 2.5）通过
  → academic-paper-reviewer 接收已验证论文（verified_only）
```

**数据级别转换过程**：

| 阶段 | 数据级别 | 可访问内容 | 被移除/遮蔽的内容 |
|------|---------|-----------|----------------|
| deep-research | **raw** | 全部研究数据：文献语料、Semantic Scholar 原始查询结果、DA 检查点原始输出、论证过程的所有中间产物 | 无 |
| academic-paper | **redacted** | 通过 Passport 中 `verification_status` 过滤后的可信来源；论文草稿中引用的已验证文献 | 未验证来源（PLAUSIBLE/UNVERIFIABLE）、`contamination_signals` 原始记录、DA 早期批评 |
| academic-paper-reviewer | **verified_only** | 论文最终稿 + 已通过完整性门控的参考文献 | 原始实验数据、脱敏的访谈记录、Passport ledger 中全部污染信号和历史版本 |

**关键约束**：Passport 的 `verification_status` 和 `contamination_signals` 字段专为上游技能设计。Reviewer 不直接读取 Passport，而是消费 academic-paper 输出的已过滤论文。这意味着 reviewer 看到的引用已经是经过验证的（`S2_VERIFIED` / `VERIFIED`），无需重复进行 source_verification。

---

## 7. 错误处理与降级策略

### 7.1 12 种故障场景全景

deep-research 在 `references/failure_paths.md` 中定义了 12 种故障场景。以下是按严重程度分组的完整列表：

| # | 故障场景 | 严重程度 | 触发阶段 | 触发条件 | 检测方式 | 恢复路径 |
|---|---------|---------|---------|---------|---------|---------|
| F1 | RQ 无法收敛 | Medium | Phase 1 / Socratic | 用户经过 8 轮对话仍未形成明确 RQ；或收缩中的 RQ 在后续轮次反复发散 | socratic_mentor 检测收敛信号 S1-S5，部分信号持续为负；或 Dialogue Health Indicator 标记循环模式 | 提供 3 个候选 RQ 供用户选择，或降级为 lit-review 模式先做文献扫描 |
| F2 | 文献不足（< 5 篇） | **High** | Phase 2 | bibliography 的标准搜索策略返回 < 5 篇满足质量阈值的来源 | bibliography_agent 执行计数后触发 | 扩展关键词、跨学科搜索、放宽时间范围；仍不足则建议用户调整 RQ |
| F3 | 方法论不匹配 | **High** | Checkpoint 1 | research_architect 设计的方法论与 RQ 要求的范式不一致（如实验法不适合描述性 RQ） | devils_advocate 在 Checkpoint 1 评估方法论适配性后标记 | 返回 Phase 1，由 research_architect 提供 3 种替代方案供用户选择 |
| F4 | DA CRITICAL 发现 | **Critical** | 任何 Checkpoint | devils_advocate 评定存在 Fatal 级别缺陷（核心假设/逻辑链/数据-结论匹配/反叙事之一） | DA 按 Severity Classification 判定 CRITICAL，输出 `[DA-FINDING: CRITICAL]` | **工作流暂停**；等待用户修正后重新 Checkpoint；2 次后建议根本性重设计 |
| F5 | 伦理 BLOCKED | **Critical** | Phase 5 | ethics_review_agent 检测到不可修复的设计缺陷，或可修复但用户拒绝修复 | ethics_review_agent 按 4 类伦理问题逐项审计，标记不可修复条目 | **停止交付**；可修复项要求用户补充声明，不可修复项建议重设计或放弃 |
| F6 | 苏格拉底对话不收敛 | Medium | Socratic | 超过 40 轮（探索型 60 轮）对话仍至少有一个 S1-S4 未达标 | socratic_mentor 持续检测收敛信号 | 提供 3 选项：继续聚焦（加最多 10 轮）/ 切换 full 模式 / 暂停保存进度 |
| F7 | 用户中途放弃 | Low | 所有模式 | 用户显式终止对话或超过 24 小时无响应 | 会话自然超时，或用户明确退出 | 保存当前进度摘要和 Material Passport 状态，提供重入路径 `resume_from_passport=<hash>` |
| F8 | 仅中文文献可用 | Medium | Phase 2 | bibliography 在所有语言搜索策略下返回结果中 > 90% 为中文文献 | bibliography_agent 统计语言分布后标记 | 切换到中文数据库；在报告中标注语言分布局限性（`[LANGUAGE-BIAS: zh-only]`） |
| F9 | 全部来源质量低于阈值 | **High** | Phase 2 | 所有可用来源的 `verification_status` 均低于 S2_VERIFIED（最高为 PLAUSIBLE 或 UNVERIFIABLE） | source_verification_agent 对全部文献执行验证后汇总统计 | 降级为"初步探索"报告格式；添加"Evidence Quality Limitations"显式章节 |
| F10 | 结论与证据不一致 | **High** | Checkpoint 3 | synthesis_agent 产出的结论未被 bibliography 中的证据充分支持，或存在矛盾方向 | devils_advocate 在 Checkpoint 3 执行 claim-evidence 对齐检查 | 返回 Phase 3，削弱未被支持的声明强度，或补充遗漏的证据 |
| F11 | 修改轮超限（2 次） | Medium | Phase 6 | Phase 6 已完成 2 轮修改后仍有未解决的 Major 问题 | report_compiler 在每次修改迭代后执行问题清单对照检查 | 未解决 Major 问题转为"Acknowledged Limitations"章节；交付最终报告；用户不接受则建议从 Phase 1 重设计 |
| F12 | 跨学科整合失败 | Low | Phase 3 | synthesis_agent 无法在单个框架内融合多个学科视角（如心理学 + 计算机科学的概念体系不可通约） | synthesis_agent 在综合阶段检测到框架冲突，标记 `[CROSS-DISCIPLINARY-CONFLICT]` | 选定主学科框架作为统一基底，其他学科视角放入"Alternative Perspectives"章节 |

### 7.2 典型故障场景的完整处理流程

#### 场景一：DA CRITICAL 发现（Checkpoint 阻断）

这是最严格的错误处理路径：

```
触发条件：devils_advocate_agent 在任何 Checkpoint 发现 CRITICAL 问题
            ↓
检测方式：DA 按 Severity Classification 评定 CRITICAL
  定义: 致命缺陷，使核心论点或方法论失效
            ↓
立即动作：
  1. 完整呈现问题描述、影响范围和修正方向
  2. **暂停工作流**，不允许进入下一 Phase
  3. 等待用户响应或修正
            ↓
恢复路径：
  路径 A → 用户修正问题 → 重新执行 Checkpoint
          → PASS → 继续
          → 再次 CRITICAL → 提示用户根本性重新思考研究方向
  路径 B → 用户修改 RQ/方法 → 回到对应 Phase
  路径 C → 用户放弃 → 进入 F7（用户中途放弃）流程
```

#### 场景二：文献不足（F2）

```
触发条件：bibliography_agent 发现 < 5 篇可用来源
            ↓
检测机制：标准搜索策略执行后计数
            ↓
处理步骤：
  1. 扩展搜索关键词（同义词、更宽泛术语、相关概念）
  2. 扩展数据库范围（灰色文献、政策报告、工作论文）
  3. 放宽时间范围（5 年 → 10 年）
  4. 尝试相邻学科关键词
  5. 如果仍不足 → 建议用户调整 RQ 或接受为探索研究
            ↓
恢复路径：
  路径 A → 成功扩展 → 继续
  路径 B → 接受探索研究 → 调整报告定位
  路径 C → 调整 RQ → 返回 Phase 1
```

#### 场景三：修改轮超限（F11）

```
触发条件：Phase 6 已执行 2 次修改，仍有未解决 Major 问题
            ↓
处理：
  1. 编译已解决和未解决问题清单
  2. 未解决 Major 问题转化为"Acknowledged Limitations"章节
  3. 交付最终报告
            ↓
  用户可接受 → 交付
  用户不接受 → 建议从 Phase 1 重设计
```

### 7.3 反阿谀设计与对抗性稳健性

deep-research 包含多层次的"防阿谀"设计：

| 层级 | 机制 | 所在 Agent |
|------|------|-----------|
| 1 | 让步阈值协议：评分 1-5 才可让步，连续让步后门控提升到 5/5 | devils_advocate_agent |
| 2 | 对话健康自检：每 5 轮检查 Persistant Agreement / Conflict Avoidance / Premature Convergence | socratic_mentor_agent |
| 3 | 意图检测：探索型模式下禁用自动收敛 | socratic_mentor_agent |
| 4 | 反模式清单：明确禁止 7 种反模式（§7 Anti-Patterns），其中 Concession 相关有多条铁律 | SKILL.md |

### 7.4 与 academic-paper-reviewer 的反阿谀设计对比

两个 skill 都各有 4 层防御，但设计方向和侧重不同：

| 维度 | deep-research | academic-paper-reviewer |
|------|--------------|----------------------|
| **总层数** | 4 层 | 4 层 |
| **核心威胁** | LLM 提前结束对话（过早收敛）或过度同意用户方向 | LLM 评审时软化批评（分数膨胀）或审稿人之间相互影响 |
| **第 1 层** | 让步阈值协议（DA 评评分 1-5 才让步） | 攻击强度保持协议（DA 评评分 1-5 + 反连续让步 + 让步率跟踪） |
| **第 2 层** | 对话健康自检（每 5 轮检查 Agreement/Convergence） | 置信度分数加权（5 分专家意见权重 > 2 分意见） |
| **第 3 层** | 意图检测（探索型模式禁用自动收敛） | Sprint Contract 两阶段隔离（Phase 1 物理上看不到论文） |
| **第 4 层** | 反模式清单（7 条通用铁律） | 反模式清单（7 条评审特异铁律，如不编造意见、不形式化再评审） |
| **独有设计** | Dialogue Health Indicator + 收敛信号 S1-S5 自我评估 | Cross-Reviewer 物理隔离 + Forbidden Operations 机械协议 |
| **复用设计** | DA 让步阈值协议（deep-research 原创，reviewer 改进后复用） | 同上 |

**关键发现**：reviewer 的反阿谀设计比 deep-research 更"机械"——引入了 Sprint Contract 的物理调用隔离和 Forbidden Operations 的硬约束。这是因为评审场景的对抗性更强（审稿人在"找问题"时受 sycophancy 影响更大），需要比研究阶段更严格的协议保护。deep-research 的对话健康自检机制（Dialogue Health Indicator、收敛信号）更偏"软性"，依赖 LLM 的自我评估能力。

---

## 8. 安全与隐私边界

### 8.1 数据流图

deep-research 是一个纯 prompt-as-code 系统，在 LLM 会话内运行。其数据流如下：

```
用户研究兴趣/问题文本
  │
  ▼
LLM Session (Claude Code / Claude API)
  │  ├── 读取 SKILL.md prompt（本地文件）
  │  ├── 读取 Agent prompt 文件（本地文件）
  │  ├── 读取 Reference 文件（本地文件）
  │  └── 用户交互（同一会话内）
  │
  ├── 输出 APA 格式报告 Markdown（本地文件）
  │
  └── [可选] ARS_CROSS_MODEL → 外部 LLM API
       └── 发送 DA 原始材料（去除用户标识信息）
```

### 8.2 数据离开本地的环节

| 环节 | 数据内容 | 目标 | 可控性 |
|------|---------|------|--------|
| **LLM 会话** | 用户的研究兴趣/问题、全部对话、中间产物 | Anthropic Claude API | 取决于部署方式（self-hosted vs cloud） |
| **Semantic Scholar API** (Tier 0) | 论文标题、DOI、作者。**不含用户研究兴趣或论文内容** | Semantic Scholar 公开 API | 可控——可选择是否启用 |
| **WebSearch** (Tier 2) | 引用论文的标题和作者 | 用户默认搜索引擎 | 可控——可禁用 |
| **DOI 验证** (Tier 1) | DOI 字符串 | `https://doi.org/{doi}` | 低风险——仅验证 DOI 是否存在 |
| **ARS_CROSS_MODEL** | **DA 评审的原始材料 + 论文草稿内容** | 第三方 LLM 提供商 | **最高风险**——需要用户明确 opt-in |
| **codex CLI audit** (v3.6.7+) | 合成/报告 Agent 的输出内容 | OpenAI API (gpt-5.5) | 仅部署端使用 |

### 8.3 跨模型 DA 的隐私风险与实现细节

`ARS_CROSS_MODEL` 是最大隐私风险点。当启用时：

devils_advocate_agent.md 中描述（第 171 行）：
```
当 ARS_CROSS_MODEL 设置时，在完成每个 Checkpoint 报告后，
**将审查材料（不含 DA 自身的发现，防止锚定效应）发送到跨模型**进行独立评审。
如果跨模型 API 失败，记录 [CROSS-MODEL-ERROR] 并继续使用单模型 DA。
```

**风险等级：高**
- 发送的内容：**论文草稿的全部内容** + DA 检查点的上下文
- 目标：另一个 LLM 提供商
- 无数据脱敏
- 缓解：**opt-in** 功能，默认关闭

> **实现细节待确认**：仓库中没有文档化以下内容——
> 1. `ARS_CROSS_MODEL` 调用的是另一个 Anthropic 账户，还是不同提供商（OpenAI/Gemini）？
> 2. 请求格式是否与主 session 相同（完整 prompt 结构），还是简化版本？
> 3. 第二意见与 DA 主意见不一致时的仲裁机制是什么？（谁最终决定）
> 
> 这是一个真实的信息缺口。ARS 的 v3.6.7 Step 6 虽然使用了 codex CLI + gpt-5.5 进行审计，但那是独立的 artifact-as-contract 审计功能，与 ARS_CROSS_MODEL 无关。当前的 ARS_CROSS_MODEL 只定义了"发-发出去、收回来"，没定义"不一致怎么办"。

### 8.4 Ethics BLOCK 的实际触发条件

来自 `ethics_review_agent.md` 和 `failure_paths.md`：

| 类型 | 具体条件 | 是否可修复 |
|------|---------|-----------|
| 未经同意的个人数据使用 | 研究涉及使用未获同意的个人数据 | 可修复（补充知情同意声明） |
| 潜在歧视性影响 | 研究结论可能对特定群体造成歧视 | 可修复（调整研究设计） |
| 双重用途风险 | 研究方法和工具可能被用于恶意目的 | 可修复（增加安全声明） |
| 无法修复的设计缺陷 | 研究设计存在本质伦理问题 | **不可修复**（建议重设计或放弃） |

> **无 BLOCK 率的历史数据**。`failure_paths.md` 中未记录 BLOCK 发生的频率或案例。`ethics_review_agent.md` 中也没有日志统计。这是一个信息缺口。

---

## 9. 总结：关键发现一览

| 维度 | 实际发现 |
|--------------|---------|
| 管线编排机制 | **纯 prompt-as-code**，LLM 根据 SKILL.md 自主编排，无代码层 |
| 状态传递 | 通过对话文本传递 + LLM 维护的 JSON 状态块 |
| 并行"并发" | 逻辑上的多维度评审，物理上仍是单 LLM 分步执行 |
| 错误处理 | 12 种故障场景定义在 `references/failure_paths.md`，含触发条件/检测/恢复/降级路径 |
| 数据持久化 | Material Passport（YAML 文件），`resume_from_passport=<hash>` 跨会话恢复 |
| LLM 交互模式 | 纯对话，无 Function Calling 触发式调度 |
| 成本估算 | **实测数据**：deep-research full ~60K 输入 + ~30K 输出，~$1.20（Opus 4.7）；10 阶段全管线 ~$4-6 |
| Tier 2 抽样率 | 50%（非 20%）|
| Schema 细节 | `additionalProperties: false` 严格 schema，CSL-JSON 作者格式，信任链含 verification_status + contamination_signals |
| socratic_mentor prompt 技巧 | CoT 角色定义 + 隐式 ReAct（SCR 协议） + 自我健康自检 + 收敛信号自我评估 + INSIGHT 提取 + 阅读探针 |
| 跨模型机制 | `ARS_CROSS_MODEL` opt-in 默认关闭，发送论文草稿到第三方 LLM |
| 隐私风险 | 主要风险点：LLM 会话（全部对话内容）、ARS_CROSS_MODEL（论文全文发送到第三方） |
| 收敛可靠性 | LLM 自我评估 S1-S5，伪收敛风险中等，Dialogue Health Indicator 有缓解 |
| 反阿谀设计 | 4 层防御：让步阈值协议、对话健康自检、意图检测、反模式清单 |