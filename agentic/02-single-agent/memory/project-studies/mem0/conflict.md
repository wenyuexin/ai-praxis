# Mem0 冲突记录

> 最后更新：2026-10-30
> 作用域：Mem0 对象研究目录内，以及该对象直接关联的评测、协议和对比材料。
> 文件职责：记录已经出现的口径冲突，作为后续核验和正文修订输入；不把未裁决主张写成最终结论。

## 使用边界

本文件只记录存在相互对立材料的事项。单纯缺少实现细节、尚未验证的性能线索，或尚未回答的评测适用范围问题，继续保留在 `notes/` 的 Evidence 和问题树中，不因“不确定”自动升级为冲突。

当前记录中的 `Verified` 只表示材料、数字或文档关系确实存在；`Observed` 表示当事方或审计者提出了该主张；`Inferred` 表示基于现有材料的解释性推断；`Unverified` 表示尚未完成独立复核、同条件复现或完整 provenance 闭环。

本文件记录由 Mem0 自身评测主张引出的冲突。Zep、Letta 等对象若建立独立研究目录，相关跨对象条目应在对应层级镜像，而不是长期只保留在本文件。

## 1. Platform 公开分数与固定 artifacts 不一致

### 冲突描述

当前 Mem0 文档声明 Platform 在 LoCoMo 上为 `92.5`（`1425/1540`），在 LongMemEval 上为 `94.4`（`472/500`）；固定 benchmark 子模块中的 Platform artifacts 则分别记录 `1410/1540 = 91.5584…`（约 `91.6`）和 `467/500 = 93.4`。两组数字不是简单四舍五入差异。

### 涉及来源

- `notes/source.md` §3.3–§3.3.1
- 固定 benchmark 子模块 `edcd6f1` 的结果 artifacts 与 `4b61c5d31b9c668a12b4f5e78064248a02c82d2b` 的 README 更新提交
- Mem0 `README` / `docs/core-concepts/memory-evaluation.mdx` 中的 Platform 声明
- `notes/evidence.md` M19

### 冲突类型

版本、运行配置与 benchmark provenance 不一致；不是单纯的展示精度差异。

### 当前证据

- `Verified`：`edcd6f1` 中存在 91.6/93.4 Platform artifacts，且其分子分母可由静态文件核对；`4b61c5d` 的 diff 只更新 README，没有替换这些 JSON。
- `Observed`：当前官方文档和 `4b61c5d` 的提交说明存在 92.5/94.4 声明，并将其描述为 v3 + temporal reasoning 的更新结果。
- `Inferred`：新版结果来自不同于旧 JSON 的评测执行或配置，但不能据此绑定具体服务端版本。
- `Unverified`：92.5/94.4 对应的服务端 revision、数据 hash、完整模型 stack、随机性、逐题输出和完整 run manifest 尚未公开闭合。

### 尚未裁决的原因

旧 artifacts 与新版 README 数字已经可以按官方提交链区分为不同结果/配置代际，但新版没有独立 JSON、run ID 或完整运行清单。因此仍不能判断差异具体来自哪一个服务端版本、数据快照、模型 stack 或其他未公开配置。

### 下一步核验

获取能同时绑定服务端 revision、数据版本、模型与 embedding、检索预算、judge、随机性及逐题输出的 run manifest；在缺少这些材料前，不运行 benchmark，也不将两组数字合并。

### 当前不做的结论

不宣布 92.5/94.4 已被 91.6/93.4 推翻，也不把差值归因于 OSS 与 Platform 的某一个具体机制。

### 1.1 README 标题与数值版本标注不一致

`mem0ai/mem0@30469ae` 的 “New Memory Algorithm (April 2026)” 标题对应 91.6 / 93.4；后续文档同步将当前数字更新为 92.5 / 94.4，但根 README 标题仍保留 April 标注。这是文档标题滞后于当前数字的版本标注问题，不是新版结果不存在的证据。

- `Verified`：April 标题与 91.6 / 93.4 的早期提交关系，以及后续 README 数字更新的提交关系均可定位。
- `Unverified`：92.5 / 94.4 对应的具体运行时间、run ID、数据版本和服务端 revision 仍无完整 manifest。
- 当前不做的结论：不据此断言 92.5/94.4 无效，也不把文档同步时间直接当作评测运行时间。

## 2. Zep 与 Mem0 的 LoCoMo 分数及计分口径冲突

### 冲突描述

围绕 **Zep 这一同一评测对象**，至少同时存在三组数字：Zep 博客报告的 `75.14% ± 0.17`、Mem0 issue 中对 Zep 的复核主张 `58.44% ± 0.20`，以及 Mem0 论文 Table 2 中报告的 Zep baseline `65.99% ± 0.16`。这不是 Zep 与 Mem0 自身成绩的三方比较。相关材料对 user model、timestamp handling、sequential/parallel search、Category 5 和分母处理也提出了不同说法。

### 涉及来源

- Zep 原始博客及其修正结果
- `getzep/zep-papers#5`
- Mem0 论文 Table 2
- `notes/source.md` §3.4
- `notes/evidence.md` M13
- Zep 公开评测代码提交 `afc6474`、`4306ce6`、`4b7f26c`

### 冲突类型

实现配置、题类范围、时间字段处理、计分和分母口径冲突。

### 当前证据

- `Verified`：三组公开数字及相关一手材料确实存在于不同来源，且三者都指向 Zep 结果。
- `Observed`：Zep 与 Mem0 双方分别提出了对实现和计分的质疑或复核主张。
- `Unverified`：各方实现是否忠实、题类和分母是否一致、`75.14` 是否严格采用 C5 过滤、`±` 的重复运行/聚合是否与对应数字绑定、时间字段如何进入系统，以及这些差异对分数的具体影响，尚未逐项核验。
- `Conflicting`：不能把三组数字视为同一协议下的可直接比较结果。

### 尚未裁决的原因

现有材料同时混合了不同实现、运行配置和修正版本；没有一份统一的数据、代码、模型、judge 和计分链路可以逐题对齐三组结果。

### 下一步核验

逐项对齐数据集版本、Category 5 是否纳入及其分母、答案 fallback、计分脚本 commit、`±` 的运行次数与聚合、user model、timestamp 语义、检索并行性、answerer/judge 模型和逐题输出，再判断哪些差异是协议差异、实现差异或计算错误。

### 当前不做的结论

不宣布 Zep 或 Mem0 任一方的分数天然正确，也不把三组 Zep 结果误写成 Zep 与 Mem0 的直接对比，更不把争议简化为单一的“评测造假”或“某方未公平比较”结论。

### C5 与分母的静态代码缺口

公开 Zep 代码曾将 `gold_answer = qa.get("answer") or qa.get("adversarial_answer")` 改为只取 `answer`，并在修正路径中使用 `category != 5` 与固定 `/1540` 分母；但现有代码快照没有把 `75.14% ± 0.17` 绑定到逐题 grades、十次运行、误差计算或单一 commit。因此可以确认代码演进和双方关于 C5 的主张存在，不能由此独立复算 `75.14` 或确认其严格采用了对应过滤。

## 3. LoCoMo 第三方审计提出的协议冲突线索

### 冲突描述

固定的 `dial481/locomo-audit@9493fb4b4af4256ed17a18e8fd0b3cfdeec29539` 对 LoCoMo 数据和评测协议提出系统性审计：在 1,540 个非 adversarial 问题中，审计者将 156 项标为问题，其中 99 项归为 `score-corrupting`、57 项归为 `citation-only`，并进一步讨论 judge、Category 5 和分母问题。这些材料是待核验的协议冲突线索，不表示本研究已确认审计结论或认定既有 benchmark 协议失效。

### 涉及来源

- `dial481/locomo-audit` 固定 commit `9493fb4…`
- 其 `AUDIT_REPORT.md`、逐题标注、统计报告和结果重算脚本
- LoCoMo 原论文与 Mem0、Zep 等公开评测材料
- `notes/source.md` §3.5
- `notes/evidence.md` M21

### 冲突类型

数据集有效性、judge 可靠性、题类/分母处理和 benchmark protocol 的潜在冲突；当前只登记审计线索，不作为已裁定的相反事实。

### 当前证据

- `Verified`：审计仓库、固定 commit、数据 hash 声明、标注文件、脚本和统计材料存在；`156/99/57` 的算术关系可由固定材料读取。
- `Observed`：审计者将这些项目解释为数据或计分问题，并据此质疑公开高分的有效性。
- `Unverified`：156 项是否确实构成问题、其中 99 项是否都属于 score-corrupting、judge 宽松比例以及这些问题对具体系统分数的影响，本轮未全量独立复核或重算。已抽查 6 条 score-corrupting 标注：4 条与所引对话存在审计者指出的静态不一致，2 条无法仅凭静态文本判定为计分级错误。
- `Inferred`：不同实现若使用不同模型、prompt、judge、Category 5 处理和分母，公开分数存在显著可比性风险。

### 尚未裁决的原因

审计仓库的重算路径依赖外部 published eval results、LLM judge 和 API key；本轮没有运行脚本、下载全部被审计系统输出或调用 judge。已完成的只是数据 hash、`156/99/57` 字段来源和 6 条代表性标注的静态抽查。

### 下一步核验

数据 hash 与代表性 score-corrupting 标注的静态抽查已完成；下一步只补 judge 输入输出和统计重算。在此之前，只把它作为第三方协议审计入口。

### 当前不做的结论

不把 `156/99/57` 写成已由本研究确认的数据集错误数，不据此宣布 Mem0、Zep 或其他系统的公开分数已经被推翻。

## 4. 2026-10-30 复查记录

- **Platform 92.5 / 94.4**：`mem0ai/memory-benchmarks` 远端 `main` 仍为 `4b61c5d…`，没有新增 run manifest 或 artifact；本条继续 park，关闭条件不变。
- **Zep 三组数字**：`getzep/zep-papers#5` 未见新的裁决或同协议复现材料；三组数字仍停在主张层。
- **第三方审计**：`dial481/locomo-audit` 远端 `main` 仍为 `9493fb4…`，没有新增审计或重算结果；6 条静态抽查仍是当前上限。
- **版本漂移**：`mem0ai/mem0` 远端 `main` 已到 `26cdefb4`（2026-10-26），领先固定 `94c3fe9f` 9 个提交；核心写入 / 检索文件未变，本条不构成对上述冲突结论的修订。
