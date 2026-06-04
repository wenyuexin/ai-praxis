# AGENTS.md

本文件是 AI 协作者在本仓库工作的入口说明。它不替代 `CONTRIBUTING.md`，只提炼 AI 执行时必须优先遵守的规则。

## 1. 先读哪些规则

开始修改前，先读本文件和 [`CONTRIBUTING.md`](./CONTRIBUTING.md) 了解仓库级强约束，再按任务类型阅读最相关的规则文件；不要为了小任务一次性通读所有规则。

按任务类型优先阅读：

- 系统研究论文、开源项目、产品、benchmark 或具体案例：[`docs/contributing/research-artifacts.md`](./docs/contributing/research-artifacts.md)
- 仓库建设与文档治理规则索引：[`docs/contributing/`](./docs/contributing/)
- 新材料、外部调研、`temp/` 回流：[`docs/contributing/documentation-workflow.md`](./docs/contributing/documentation-workflow.md)
- Evidence 与 Traceability：先读组合流程 [`docs/contributing/evidence-and-traceability.md`](./docs/contributing/evidence-and-traceability.md)，需要细则时再读 [`docs/contributing/evidence-rules.md`](./docs/contributing/evidence-rules.md) 和 [`docs/contributing/traceability-rules.md`](./docs/contributing/traceability-rules.md)
- README 和元信息文件：[`docs/contributing/readme-rules.md`](./docs/contributing/readme-rules.md)、[`docs/contributing/metadata-files.md`](./docs/contributing/metadata-files.md)

## 2. 仓库内容边界

- 具体机器学习、Agentic AI、LLM、RAG、CV 等知识正文放在对应主题目录中，例如 `agentic/`、`llm/`、`rag/`、`training-infra/`。
- 仓库建设规则、贡献流程、证据分级、可追溯性和 README 规则放在 `docs/contributing/`。
- 文档模板放在 `docs/templates/`。
- Markdown、Mermaid 等渲染测试材料放在 `docs/test/`。
- 不要把仓库建设约束写进知识正文目录，也不要把具体领域知识写进 `docs/contributing/`。

## 3. 默认工作流程

处理新材料或修改文档时，默认按以下顺序执行：

1. 判断材料类型：整体认知、主题缺口、待研究对象、冲突问题，还是正文专题材料。
2. 判断 Evidence 状态：`Verified`、`Observed`、`Inferred`、`Unverified`、`Conflicting`、`Deprecated`。
3. 判断目标落位：`overview.md`、`backlog.md`、`candidates.md`、`conflict.md`，还是正文专题。
4. 必要时记录轻量 Trace：`Source / Decision / Placement / Gap`。
5. 最后再写入正文或元信息文件。

默认策略：**拿不准时，先写元信息文件或保留在 `temp/`，不要抢写正文。**

## 4. Evidence 与 Traceability

- Evidence 回答：这条知识凭什么成立？
- Traceability 回答：这条内容从哪里来、为什么落在这里？
- 外部 AI 调研、网页搜索、博客摘录、issue/PR 讨论默认只是上游输入或候选证据，不直接写成主线结论。
- 从 `temp/` 或外部材料回流主线时，必须能说明 Evidence 状态和 Trace 字段。
- 模型综合判断默认标为 `Inferred` 或 `Unverified`，不得写成论文、官方文档或已验证事实。

## 5. README 与元信息文件

- README 负责定向和目录导航，不主动列出 `overview.md`、`backlog.md`、`candidates.md`、`roadmap.md`、`conflict.md` 等元信息文件。
- README 目录树原则上只展示目录，不展示内容文件。
- 根 README、一级目录 README、二级及更深层 README 的展开深度不同，按 `readme-rules.md` 执行。
- `index.md`、`candidates.md`、`roadmap.md` 是按需启用文件，不要为每个目录机械创建。
- `conflict.md` 仅在发现术语、事实、版本、适用边界或结论力度冲突时创建。

## 6. 修改范围控制

- 只改与当前任务直接相关的文件，不顺手重构无关内容。
- 文档重组时优先移动原文件，再做小范围后续编辑，避免迁移和重写混在一起。
- 删除文件前先确认内容已经迁移、替代或确实不再需要。
- 不要无任务依据地保留临时反馈文件、外部审查草稿、一次性中间产物；若用户明确要求生成审查反馈或委托材料，可以写入 `docs/temp/`，并在交付时说明其临时性质。
- 不要主动创建新的规则文档，除非现有文档已经明显过长或用户明确要求拆分。
- 如果用户要求与仓库规则冲突，先说明冲突点并给出替代方案；如果用户确认坚持原要求，可以按用户要求执行，并在交付时说明偏离了哪条规则。

## 7. 提交前自检

交付前至少检查：

- 链接是否仍然有效。
- 是否残留旧路径，例如根级 `assets/` 的旧引用。
- 是否残留旧术语，例如已废弃的“证据强度”表述、`Unverified / Unreliable`、`主干正文`、`回流主干`。
- 新增 Markdown 文件是否使用 LF 行尾。
- README 是否违反“不主动列出元信息文件”的规则。
- 是否把未验证材料写成了主线定论。
