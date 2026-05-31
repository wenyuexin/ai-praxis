# Academic Research Skills (ARS) 全景分析

> 本文档是对 [academic-research-skills](https://github.com/Imbad0202/academic-research-skills) 项目的整体概述。项目装机与使用说明请参见 [README.md](./README.md)，deep-research 技能深度分析请参见 [deep-research-skill.md](./deep-research-skill.md)。

**来源项目版本**: v3.8.2 | **许可**: CC BY-NC 4.0 | **维护者**: Cheng-I Wu

---

## 1. 技能全景：4 个技能，25 种模式

| 技能 | 版本 | 模式数 | 数据级别 | 核心能力 |
|------|------|--------|---------|---------|
| **deep-research** | v2.9.2 | 7 | raw | 深度研究：从问题构思到 APA 报告的全研究管线 |
| **academic-paper** | v3.1.1 | 10 | redacted | 论文写作：从大纲到最终稿（含 LaTeX/DOCX 输出） |
| **academic-paper-reviewer** | v1.9.0 | 6 | verified_only | 论文评审：5 评审人面板 + 编辑决策 + 修改指导 |
| **academic-pipeline** | v3.8.0 | 2 | verified_only | 10 阶段完整管线编排器：研究→写作→评审→修改→终稿 |

### 模式频谱分布

| 频谱 | 数量 | 含义 |
|------|------|------|
| Fidelity (保真) | 14 (56%) | 模板驱动，输出可预测 |
| Balanced (平衡) | 7 (28%) | 默认模式 |
| Originality (原创) | 4 (16%) | 探索导向，模板稀疏 |

**三者的边界**：Fidelity 模式严格遵循预定义模板和检查清单，输出格式高度可预测，适合需要标准化产出的场景（如 re-review、quick review）。Balanced 模式在模板和自由探索之间均衡，是大多数技能调用的默认入口。Originality 模式模板稀疏，允许 Agent 根据对话上下文自主构建输出结构，适合开放探索（如 guided 评审）——但也因此导致输出一致性最低，不宜用于需要可比较产出的场景。Reviewer 技能中 6 种模式分布为 4 个 Fidelity、1 个 Balanced、1 个 Originality，与 deep-research 的分布（7+7+4）有显著差异，反映了"评审需要标准化"与"研究需要探索空间"的本质区别。

---

## 2. 架构本质：纯 Prompt-as-Code

### 2.1 核心发现

ARS 的整个编排层由 **SKILL.md 文件本身** 驱动，没有任何 Python/TypeScript 编排代码。LLM（Claude Code）读取 SKILL.md 的指令，自主顺序调用各 Agent prompt。没有 LangChain/LangGraph，没有 Function Calling 触发式调度——全部在一个长 context 的 LLM 会话中完成。

### 2.2 deep-research 的 6 阶段管线

```
Phase 1: SCOPING      → research_question_agent → research_architect_agent → DA CHECKPOINT 1
Phase 2: INVESTIGATION → bibliography_agent → source_verification_agent
Phase 3: ANALYSIS     → synthesis_agent → DA CHECKPOINT 2
Phase 4: COMPOSITION  → report_compiler_agent → APA 7.0 草稿
Phase 5: REVIEW       → editor_in_chief_agent + ethics_review_agent + DA CHECKPOINT 3
Phase 6: REVISION     → report_compiler_agent → 最终报告
```

### 2.3 关键设计特征

| 维度 | 实际实现方式 |
|------|------------|
| **管线编排** | 纯 prompt-as-code。SKILL.md 中的 ASCII 流程图 + 自然语言描述驱动 LLM 自主编排 |
| **状态传递** | 通过对话文本传递 + LLM 自行维护的 JSON 状态块 |
| **并行评审** | 逻辑多维度，物理上仍是单 LLM 依次扮演不同角色 |
| **Agent 交互** | 全量 context 保留，无序列化/消息队列/共享内存 |
| **错误处理** | `references/failure_paths.md` 定义 12 种故障场景，含触发/检测/恢复/降级路径 |

### 2.4 Agent 团队（deep-research 13 个 Agent）

| Agent | 角色 | 阶段 |
|-------|------|------|
| research_question_agent | FINER 框架研究问题工程 | Phase 1 |
| research_architect_agent | 方法论蓝图设计 | Phase 1 |
| bibliography_agent | 系统文献检索与筛选 | Phase 2 |
| source_verification_agent | 来源验证 + 7 级证据层级评分 | Phase 2 |
| synthesis_agent | 跨来源综合 + 矛盾分析 | Phase 3 |
| report_compiler_agent | APA 7.0 完整报告撰写 | Phase 4, 6 |
| editor_in_chief_agent | 主编级编辑评审 | Phase 5 |
| devils_advocate_agent | 反方辩手：3 检查点 + 10 种逻辑谬误检测 | Phase 1, 3, 5 |
| ethics_review_agent | 研究伦理审查 | Phase 5 |
| socratic_mentor_agent | 苏格拉底导师：5 层对话引导 | Socratic 模式 |
| risk_of_bias_agent | 偏倚风险评估（RoB 2 + ROBINS-I） | Systematic Review |
| meta_analysis_agent | 荟萃分析：效应量、异质性、GRADE | Systematic Review |
| monitoring_agent | 发表后文献监测 | 可选后管线 |

### 2.5 academic-paper-reviewer 的 7 个 Agent

| Agent | 角色 | 阶段 |
|-------|------|------|
| field_analyst_agent | 6 维度论文分析，动态生成审稿人配置卡 | Phase 0: Field Analysis |
| eic_agent | 主编视角评审（第一印象、原创性、重要性、结构、期刊匹配） | Phase 1: Parallel Review |
| methodology_reviewer_agent | 方法论专家（范式自适应 + APA 7.0 统计报告检查） | Phase 1: Parallel Review |
| domain_reviewer_agent | 领域知识专家（文献覆盖审计、理论框架、论证准确性） | Phase 1: Parallel Review |
| perspective_reviewer_agent | 跨学科视角（假设审计、跨学科连接、实际影响） | Phase 1: Parallel Review |
| devils_advocate_reviewer_agent | 反方辩手（8 大挑战维度 + CRITICAL 严格定义 + 攻击强度保持协议） | Phase 1: Parallel Review |
| editorial_synthesizer_agent | 编辑合成（4 级共识分类 + 置信度加权 + 争议仲裁 + 修改路线图） | Phase 2: Editorial Synthesis |

---

## 3. 完整管线（academic-pipeline）

### 3.1 10 阶段流程

```
1. RESEARCH   → deep-research（7 种子模式可选）
   ↓ 🧑 用户确认 RQ + 方法论
2. WRITE      → academic-paper
   ↓ 🧑 用户确认大纲
2.5 INTEGRITY → 7 模式 AI 故障检查清单
   ↓ ✓ 用户确认完整性报告（FAIL 时最多 3 次修复）
3. REVIEW     → academic-paper-reviewer（5 评审人面板）
   ↓ 🧑 用户决定 Accept/Minor/Major/Reject
3→4 Coaching  → EIC 苏格拉底修改指导（最多 8 轮）
4. REVISE     → academic-paper
   ↓ 🧑 用户确认修改
3'. RE-REVIEW → 验证性再评审
   ↓ 🧑 用户决定
4'. RE-REVISE → 终稿冻结
   ↓ 🧑 用户确认
4.5 FINAL INTEGRITY → 零容忍深度检查
   ↓ ✓ 用户确认
5. FINALIZE   → 格式选择（MD/DOCX/LaTeX/PDF）+ AI 披露声明
6. PROCESS SUMMARY → 流程总结 + 协作深度报告
```

### 3.2 两类检查点

| 类型 | 含义 | 数量 |
|------|------|------|
| 🧑 决策型 | 用户选择分支或批准材料决策 | 10 个 |
| ✓ 完整性门控 | 机器验证 + 用户确认 | 2 个 |

完整性门控不可跳过，observer 在此被显式跳过以防止稀释。

### 3.3 数据访问级别递进

```
raw (deep-research) → redacted (academic-paper) → verified_only (reviewer/pipeline)
```

每个阶段的数据权限递减，确保评审方只能看到已验证的信息源。

**各数据级别的实际含义**：

| 级别 | 所属技能 | 可访问内容 | 为何这样设计 |
|------|---------|-----------|-------------|
| **raw** | deep-research | 全部研究数据：文献语料、Semantic Scholar 原始查询、DA 检查点原始输出 | 研究阶段需要无过滤地访问所有信息源，包括未验证的、低质量的、矛盾的——后续阶段会过滤 |
| **redacted** | academic-paper | 通过 `verification_status` 过滤后的可信来源；论文草稿中引用已验证文献 | 写论文时应只引用可信来源，但论文本身还在草稿阶段，可以包含作者的初步分析和解释 |
| **verified_only** | academic-paper-reviewer / pipeline | 论文最终稿 + 已通过完整性门控的参考文献 | 评审者不需要看到研究过程中的"脏数据"（失败的假设、被放弃的方法），这会污染评审的客观性 |

**reviewer 与上游的数据交换方式**：reviewer 不直接读取 Material Passport，而是消费 academic-paper 输出的已过滤论文 + 参考文献。完整性门控（Stage 2.5）确保只有通过 7 项 AI 故障检查的论文才能进入 reviewer。这意味着 reviewer 的 "verified_only" 是一个被动接收的约束，而非 reviewer 自身的主动过滤机制。

---

## 4. Material Passport 数据契约

Material Passport 是 ARS 套件的跨技能数据契约，以 YAML 文件持久化：

- **严格 Schema**：`additionalProperties: false`，不允许扩展字段
- **CSL-JSON 作者格式**：兼容 Zotero/Obsidian
- **信任链**：`verification_status`（S2_VERIFIED/VERIFIED/PLAUSIBLE/UNVERIFIABLE/FABRICATED）+ `contamination_signals`
- **持久化方式**：适配器脚本生成，ARS 管线只读取不写入
- **重置边界（v3.6.3）**：`ARS_PASSPORT_RESET=1` 允许在 FULL checkpoint 清空 context 跨会话续跑

### 4.1 文献语料输入流（v3.6.4+）

```
用户语料源（Zotero/Obsidian/PDF 文件夹）
  → 适配器（folder_scan.py / zotero.py / obsidian.py）
  → passport.yaml（含 literature_corpus[]）
  → Phase 1 Agent（corpus-first / search-fills-gap 流程）
  → Search Strategy 报告（含 PRE-SCREENED 可复现块）
```

4 条铁律：相同筛选标准、无静默跳过、不变更语料、解析失败优雅降级。

---

## 5. 关键 Agent Prompt 设计亮点

详细分析见 [deep-research-skill.md](./deep-research-skill.md)

| Agent | 设计亮点 |
|-------|---------|
| **socratic_mentor_agent** (550 行) | CoT 角色定义 + 隐式 ReAct（SCR 协议）+ 对话健康自检 + 意图检测（6 信号）+ INSIGHT 提取 + 收敛信号 S1-S5 自我评估 + 阅读探针 |
| **devils_advocate_agent** (193 行) | 3 检查点 + 10 种逻辑谬误 + 让步阈值协议 1-5 评分 + 反阿谀设计 + 跨模型 DA |
| **source_verification_agent** (183 行) | 7 级证据层级 + 三重验证（Semantic Scholar 100% / DOI 100% / WebSearch 50%）+ DOI 不匹配检测 |
| **synthesis_agent** (260 行) | 3 层引用发射 + 声明意图清单预承诺清单 + 3 种反模式保护 |
| **research_question_agent** (186 行) | FINER 5 维度评分 + 苏格拉底模式分支 |

---

## 6. 错误处理：12 种故障场景

| 严重程度 | 场景 | 处理策略 |
|---------|------|---------|
| **Critical** | DA CRITICAL 发现（F4） | 停止推进，等待用户修正 |
| **Critical** | 伦理 BLOCKED（F5） | 停止交付，区分可修复/不可修复 |
| **High** | 文献不足（F2） | 扩展关键词、跨学科、放宽时间 |
| **High** | 方法论不匹配（F3） | 返回 Phase 1，提供替代方案 |
| **High** | 全部来源质量低于阈值（F9） | 降级为初步探索报告 |
| **High** | 结论与证据不一致（F10） | 返回 Phase 3，削弱或补充证据 |
| Medium | RQ 不收敛 / 修改轮超限 / 仅中文文献等 | 提供候选方案或保存进度 |

### 反阿谀 4 层防御

1. 让步阈值协议（DA：评分 1-5，不轻易让步）
2. 对话健康自检（Mentor：每 5 轮检查 Agreement/Conflict-Avoidance/Convergence）
3. 意图检测（探索型模式禁用自动收敛）
4. 反模式清单（7 条铁律）

---

## 7. 性能与成本

基于 ~15,000 词论文 + ~60 篇参考文献，在 Claude Opus 4.7 上运行：

| 模式 | 输入 Tokens | 输出 Tokens | 估算成本 |
|------|------------|------------|---------|
| deep-research socratic | ~30K | ~15K | ~$0.60 |
| deep-research full | ~60K | ~30K | ~$1.20 |
| deep-research systematic-review | ~100K | ~50K | ~$2.00 |
| **全管线（10 阶段）** | **~200K+** | **~100K+** | **~$4-6** |

---

## 8. 版本演进关键里程碑

| 版本 | 新增能力 |
|------|---------|
| v3.3 | Semantic Scholar API 验证、反泄漏协议、VLM 图表验证 |
| v3.4 | PRISMA-trAIce 合规检查、RAISE 评估 |
| v3.5 | 协作深度 Observer、苏格拉底阅读探针 |
| v3.6 | Sprint Contract 硬门控、Passport 重置边界、文献语料输入 |
| v3.7 | Claude Code 插件打包、10 个斜杠命令 |
| v3.7.3 | 三层引用发射（<!--ref--> + <!--anchor-->）、污染信号 |
| v3.8 | L3 声明忠实度审计（claim-ref-alignment）、高警告表单 REFUSE 规则 |

---

*最后更新: 2026-05-17 (updated)*