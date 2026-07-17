# Incident: 研究正文尾部章节的职责归并

- 当前状态：Resolved
- 是否复发：未复发
- 记录日期：2026-07-17

## 问题背景

在整理 `agentic/07-evaluation/agent-benchmarks/llm-as-a-verifier_2607.05391.md` 时，文档尾部先后出现“论文真正值得带走什么”“不应过度泛化的地方”“读者最容易卡住的点”“证据边界”“一句话总结”“主要证据入口”和 `Evidence` 等连续同级章节。

这些章节分别包含结论、适用边界、阅读辅助、数值核验范围、来源链接和处理链路。单项内容大多有保留价值，但同级平铺后，正文收束与治理元数据相互重复，读者难以判断哪些内容属于论文结论，哪些内容只负责说明证据状态和处理过程。

## 冲突点

1. `research-artifacts.md`、Evidence 记录规则和 Traceability 规则已经分别定义正文、Evidence、Sources 与 Trace 的职责，但没有直接约束这些合法组件在文档尾部如何归并。
2. 局部 `Limitations`、`Gap` 或来源说明可以补充正文，但机械增加多个同级章节会重复表达 `## Evidence` 已承接的信息。
3. 直接把当前修订结果固化为统一的“两章结构”也不合适；不同研究对象的结论体量和阅读任务不同，规则应约束职责，不应固定章节数量和标题。

## 解决方案

### 文档层面

将一句话摘要移到标题后；将主要结论、解释性判断、适用边界与数值核验边界归入 `## 核心结论与适用边界`；将正式来源、处理链路和待补缺口统一归入 `## Evidence` 的 `Status / Sources / Trace / Needs` 字段。删除不再承担独立阅读任务的重复尾部章节。

### 规则层面

在 `research-artifacts.md` §4 增加正文尾部职责归并的 stop-line：不要把结论、适用边界、阅读提示、来源入口和 Evidence 机械拆成连续同级章节；先让正文承接结论、解释和边界，再由 `## Evidence` 统一承接 `Status / Sources / Trace / Needs`。

### Intent 层面

在 `intent/evidence-governance.md` 增加“Evidence 不是正文尾部的通用容器”的解释，同时防止两种误读：既不能让 Evidence 吞掉正文论证，也不能用多个证据类尾部章节架空 Evidence。当前论文的归并结果不构成全仓固定模板。

## 验证

- 修订后的论文正文以“核心结论与适用边界”完成读者论证，以单一 `## Evidence` 汇总治理信息。
- 正式 arXiv 来源保留在 `Sources`，本地处理过程保留在 `Trace`，待补元信息与附录核验保留在 `Needs`。
- 新增规则没有要求普通任务读取额外文件，也没有规定统一章节数量、标题或固定尾部模板。

## 复发信号

出现以下任一情况时，重新检查本 incident：

- 其他论文、代码库或 benchmark 文档再次出现多个职责重叠的同级尾部章节。
- `## Evidence` 再次吸收正文结论和适用边界，导致正文论证不完整。
- 正文已有完整结论和限制说明后，又平铺“证据边界”“来源入口”等章节重复 Evidence 字段。
- AI 将本次“核心结论与适用边界 + Evidence”误读为所有研究文档必须复制的固定模板。

## 关联文件

- `docs/contributing/rules/research-artifacts.md` §4
- `docs/contributing/intent/evidence-governance.md` §5
- `docs/contributing/rules/evidence-recording-rules.md` §3、§4
- `agentic/07-evaluation/agent-benchmarks/llm-as-a-verifier_2607.05391.md`

## 边界

本 incident 记录的是单次完整治理问题轨迹，不证明该模式已经在多个独立对象中稳定复发，因此暂不创建 `cases/` 条目。后续只有在其他研究文档出现同类职责混写，且现有 stop-line 仍不足以阻止时，才考虑升级为复发案例或继续调整规则。
