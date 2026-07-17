# Evidence 治理设计意图

本文件不重复 Evidence 执行规则，而是解释 Evidence 规则为什么采用“判断、记录、来源边界”三层分工，以及这些分工要防止什么误读。

## 1. 三类规则为什么分开

Evidence 判断回答“这条知识凭什么成立”；Evidence 记录回答“判断结果写在哪里、如何让读者复核”；来源边界回答“不同材料类型有哪些不能外推的地方”。

这三类问题经常在同一任务中连续发生，但不是同一个判断：

- 判断 Claim 是否成立，不等于决定使用表格还是 bullet。
- 判断源码观察是 `Observed`，不等于填写 `Version Basis`。
- 判断论文结果是否被原文支持，不等于决定它进入正文还是研究辅助材料。

因此主规则按任务阶段分开，但仍由 [`evidence-and-traceability.md`](../rules/evidence-and-traceability.md) 提供组合入口。

## 2. 为什么不按六种 Status 分文件

六种 Status 不是六个独立主题，而是同一个 Evidence 判断模型的互相约束：协作者需要同时比较 `Verified`、`Observed`、`Inferred`、`Unverified`、`Conflicting` 和 `Deprecated`，才能避免把不确定结论升级成稳定事实。

将 Status 单独拆出会让 Claim 定义、来源边界和表达力度被分散，普通任务反而需要读取更多文件。因此 Status 保持在 Evidence 判断规则中。

## 3. 为什么来源类型先集中成一个模块

论文、代码库、实验报告和讨论材料确实有不同的误判模式，但不是每种材料都值得立即拥有独立规则文件。

来源类型模块提供一个横向边界层，避免把论文或实验的局部 caveat 扩大成新的通用制度。只有某类材料形成独立、稳定且无法由现有规则承接的 stop-line 时，才考虑继续拆出单独文件。

## 4. 为什么 Evidence registry 不是默认文件

Evidence registry 是为证据对照已经成为稳定维护负担的主题服务的。它不是每个目录都应拥有的结构槽位，也不是为了让目录看起来完整而创建的空文件。

少量证据直接写在正文的 `## Evidence`；待补证方向进入 `backlog.md` 或 `candidates.md`；来源冲突进入 `conflict.md`。只有证据对照本身需要长期查询、版本维护或跨文档复用时，才考虑 registry。

## 5. 主规则与 intent 的边界

主规则必须保留定义、触发条件、stop-line、字段骨架和必要的短示例，使 AI 在通常任务中不查 intent 也能执行正确。

本文件只解释：

- 为什么判断和记录分开；
- 为什么来源类型是横向模块而不是新的 Status；
- 为什么 registry 按需启用；
- 为什么组合入口不能重复定义三套细则。

它不替代 `evidence-assessment-rules.md`、`evidence-recording-rules.md` 或 `evidence-source-rules.md` 的执行要求。
