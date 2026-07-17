# Incident: Evidence Claim 边界与源码观察状态

- 当前状态：Resolved
- 是否复发：未复发
- 记录日期：2026-07-16

## 问题背景

在整理 `agentic/07-evaluation/agent-benchmarks/llm-as-a-verifier_2607.05391.md` 的论文机制说明时，来源直接事实、解释性推论、未确认实现细节与教学性反事实容易在同一段落中混写。后续对 Evidence 规则的执行测试和跨规则检查又发现，直接源码观察的 Evidence Status 与版本元数据缺失时的处置路径不够确定。

这两类问题共同指向同一个治理缺口：高风险知识 Claim 缺少足以让协作者稳定区分来源边界与证据状态的执行约束。

## 冲突点

1. Evidence Status 用于表达当前 Claim 与材料之间的支持关系或证据状态，但单独使用不能避免正文将来源事实、解释性推论、未确认实现与作者构造的教学内容写成同层结论。
2. 直接源码观察曾可被 `Verified` 与 `Observed` 两种状态同时解释；相邻规则文件对其状态口径也不一致。
3. 基于源码的 Claim 缺少 Version Basis 或 Observed At 时，没有区分可恢复与不可恢复观察记录的明确处置路径。

## 解决方案

### 正文组织

在 `evidence-assessment-rules.md` 增加高风险机制说明的“事实 → 解释 → 边界”组织方式：先写来源直接支持的内容，再区分解释性推论，并说明未确认实现、可选实现、教学例子或反事实的边界。该方式不新增 Evidence Status、字段或逐句标注要求。

在 `evidence-and-traceability.md` 的最小流程中加入高风险时的分层组织判断，并明确它不替代 `Status / Sources / Trace / Needs`。

### 源码观察

1. 将版本可追溯的直接源码观察统一标为 `Observed`；官方 release notes 对实现行为或版本变化的明确声明仍可标为 `Verified`。
2. 对缺少 Version Basis 或 Observed At 的源码 Claim：可恢复时补齐版本、观察时间与适用范围后重标为 `Observed`；不可恢复时不保留为稳定正文，只能作为 `Unverified` 线索留在 `temp/`，或将待核验缺口写入 `backlog.md`。
3. 同步 `documentation-workflow.md` 的状态摘要，消除与 `evidence-assessment-rules.md`、`evidence-source-rules.md` 的口径冲突。

## 验证

- 在 `llm-as-a-verifier_2607.05391.md` 中完成高风险机制说明试点，并将来源事实、解释性推论、未确认实现细节和教学性反事实分层呈现。
- 对版本限定的直接源码观察、不可恢复的源码观察、官方 release-note 声明三类场景进行执行测试，确认每类均有唯一处置路径。
- 完成跨规则检查，确认 `evidence-assessment-rules.md`、`evidence-source-rules.md` 与 `documentation-workflow.md` 对直接源码观察的状态口径一致。

## 复发信号

出现以下任一情况时，重新检查本 incident：

- 新规则或知识文档再次将直接源码观察直接归为 `Verified`。
- 高风险机制说明再次把来源事实、推论、未确认实现或教学内容混写为同层结论。
- 多个独立任务反复需要同一套分层组织方式，且现有规则不足以指导执行。

## 关联文件

- `docs/contributing/rules/evidence-assessment-rules.md` §3、§4.1
- `docs/contributing/rules/evidence-source-rules.md` §3
- `docs/contributing/rules/evidence-and-traceability.md` §3
- `docs/contributing/rules/documentation-workflow.md`
- `docs/contributing/rules/traceability-rules.md` §3.5
- `agentic/07-evaluation/agent-benchmarks/llm-as-a-verifier_2607.05391.md`

## 边界

本 incident 记录的是单次完整治理问题轨迹，不代表该模式已经形成稳定复发案例；因此暂不创建 `cases/` 条目。Evidence 设计意图现在由按设计问题组织的 `intent/evidence-governance.md` 和 `intent/evidence-source-boundaries.md` 承接；本次迁移不改变 incident 当时“不新增 Evidence intent 文件”的历史处理范围。
