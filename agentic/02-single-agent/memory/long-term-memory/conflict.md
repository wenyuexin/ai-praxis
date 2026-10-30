# 长期记忆研究中的冲突记录

> 更新：2026-10-30
> 范围：当前记录 A-MEM 固定论文版本内的口径差异；不代表长期记忆领域存在对应共识冲突。

## A-MEM v11：实验叙述与结果表不一致

- Status: Conflicting；差异已核对，上游解释未获得。
- 涉及正文：[A-MEM 论文研究](./papers/a-mem_2502.12110.md)。
- Sources: [论文 §4.3](https://arxiv.org/html/2502.12110v11#S4.SS3)、[表 1](https://arxiv.org/html/2502.12110v11#S4.T1)、[附录 A.3](https://arxiv.org/html/2502.12110v11#A1.SS3)、[表 5](https://arxiv.org/html/2502.12110v11#A1.T5)、[表 6](https://arxiv.org/html/2502.12110v11#A1.T6)。
- 当前证据：§4.3 称 GPT 设置的 Multi-Hop 至少翻倍；表 1 中 mini 是 27.02 对 MemGPT 26.65，4o 是 32.86 对 30.36。A.3 用于说明 mini Multi-Hop 的 ROUGE-L 44.27、METEOR 23.43、SBERT 70.49，实际出现在对应表的 Temporal 列。
- 其他反例：A.3 称非 GPT 模型所有指标均领先，但表 5 的 Qwen2.5-3B Open-Domain 中，A-MEM 的 ROUGE-2／ROUGE-L 为 1.51／7.51，低于 MemGPT 的 1.90／7.90。不能把表 1 的 F1／BLEU-1 优势扩展为全部指标。
- 处理：正文分别转述作者主张和表格所支持的比较，不将“多跳翻倍”作为有效结论；不擅自改写原论文或猜测实验重跑、版本混用等原因。
- Trace: 固定 `arXiv:2502.12110v11` 的 HTML／PDF 交叉核对后登记，供正文评价与后续维护使用。
- Needs: 作者勘误、可对应该版本的实验记录，或足以解释差异的补充材料。

## A-MEM v11：Single-Hop 的图表数字不一致

- Status: Conflicting；数值差异已核对，实验对应关系未获解释。
- Sources: [图 3(d)](https://arxiv.org/html/2502.12110v11#S4.F3)、[表 1](https://arxiv.org/html/2502.12110v11#S4.T1)、[表 3](https://arxiv.org/html/2502.12110v11#S4.T3)、[表 8](https://arxiv.org/html/2502.12110v11#A1.T8)、[同版本 PDF](https://arxiv.org/pdf/2502.12110v11)。
- 当前证据：表 8 指定 GPT-4o-mini Single-Hop 使用 k=50。图 3 在该配置标注 F1／BLEU-1 为 44.55／37.02；表 1 与表 3 为 44.65／37.06。
- 处理：正文摘表时保持表值，讨论超参数时并列说明图值；不自行归因于笔误、重跑波动或四舍五入，不用其中一个值覆盖另一个。
- Trace: 固定版本 PDF 的图内标注与结果表交叉核对后登记。
- Needs: 作者对图表实验配置、数据或数字修订的说明。

## A-MEM v11：提示词动作名与文字定义不一致

- Status: Conflicting；模板差异已核对，执行语义未确定。
- Sources: [论文 §3.3](https://arxiv.org/html/2502.12110v11#S3.SS3)、[附录 B.2–B.3](https://arxiv.org/html/2502.12110v11#A2.SS2)。
- 当前证据：B.3 的动作说明是 `strengthen`、`update_neighbor`，JSON 示例却列 `strengthen`、`merge`、`prune`；§3.3 说可更新 context、keywords、tags，模板没有关键词更新字段。B.2 标为建链提示词，但只问是否演化，没有完整链接输出契约。
- 处理：正文保留方法级定义与模板差异，不据动作名字推断已经实现合并、删除或遗忘；不把当前代码自动等同该论文实验实现。
- Trace: 从同版本公式、文字和提示词对照中形成；此处管理差异，正文解释其对可复现性的影响。
- Needs: 与论文实验版本对应的输出 schema、动作实现及提示词映射说明。
