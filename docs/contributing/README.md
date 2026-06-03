# 贡献与仓库建设规则

本目录沉淀与仓库建设、文档维护、证据分级、可追溯性相关的规则。

它不保存具体机器学习或 Agentic 知识内容；知识内容应放在对应主题目录中，例如 `agentic/`、`llm/`、`training-infra/` 等。

## 职责边界

- [`CONTRIBUTING.md`](../../CONTRIBUTING.md)：仓库级贡献入口，保留必须遵守的核心规则。
- [`AGENTS.md`](../../AGENTS.md)：AI 协作者入口，提炼执行时必须优先遵守的规则。
- `docs/contributing/`：承接更细的文档治理、证据、可追溯性和维护流程规则。
- 主题目录：保存经过整理后的知识正文、综述、backlog、research queue 和 conflict 记录。
- `docs/templates/`：保存仓库文档模板。
- `docs/test/`：保存 Markdown、Mermaid 等文档渲染测试材料。

## 目录结构

```text
docs/contributing/
├── documentation-workflow.md
├── evidence-and-traceability.md
├── evidence-rules.md
├── metadata-files.md
├── readme-rules.md
├── traceability-rules.md
└── README.md
```

## 当前规则入口

当前仓库级强约束仍以根目录 [`CONTRIBUTING.md`](../../CONTRIBUTING.md) 为准，AI 协作者入口见 [`AGENTS.md`](../../AGENTS.md)。本目录承接更细的规则设计；当 `CONTRIBUTING.md` 中某类规则继续增长时，应逐步迁移到本目录下的专题文件，并在 `CONTRIBUTING.md` 中保留摘要和链接。

已拆分规则：

- [材料处理与文档增改流程](./documentation-workflow.md)：定义新材料、外部调研、`temp/` 内容和正文修改的判断、落位与回流流程。
- [Evidence 与 Traceability 工作流](./evidence-and-traceability.md)：统一执行入口，说明正式 Evidence + 轻量 Traceability 如何一起使用。
- [Evidence 规则](./evidence-rules.md)：定义 Claim、Evidence 状态、来源类型、标注密度和正文标注方式。
- [Traceability 规则](./traceability-rules.md)：定义 Source / Decision / Placement / Gap 四个轻量链路字段。
- [README 规则](./readme-rules.md)：定义 README 出现条件、目录树展开深度、导航边界和类型化结构建议。
- [元信息文件模型](./metadata-files.md)：定义 `README.md`、`overview.md`、`backlog.md`、`research-queue.md`、`roadmap.md`、`conflict.md` 等文件的职责边界。

后续优先拆分的规则包括：

- AI 协作者执行前检查清单
