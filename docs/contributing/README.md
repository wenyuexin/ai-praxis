# 贡献与仓库建设规则

本目录沉淀与仓库建设、文档维护、证据分级、可追溯性相关的规则。

它不保存具体机器学习或 Agentic 知识内容；知识内容应放在对应主题目录中，例如 `agentic/`、`llm/`、`training-infra/` 等。

更准确地说，这里不只是一个规则目录，也是在沉淀一套面向 AI 与人类协作的结构化研究方法体系：帮助研究过程稳定分层、证据可追、判断可回流、知识可长期维护。

## 职责边界

- [`CONTRIBUTING.md`](../../CONTRIBUTING.md)：仓库级贡献入口，保留必须遵守的核心规则。
- [`AGENTS.md`](../../AGENTS.md)：AI 协作者入口，提炼执行时必须优先遵守的规则。
- `docs/contributing/`：承接更细的文档治理、证据、可追溯性和维护流程规则。
- 主题目录：保存经过整理后的知识正文、综述、backlog、candidates 和 conflict 记录。
- `docs/templates/`：保存仓库文档模板。
- `docs/test/`：保存 Markdown、Mermaid 等文档渲染测试材料。

## 当前规则入口

当前仓库级强约束仍以根目录 [`CONTRIBUTING.md`](../../CONTRIBUTING.md) 为准，AI 协作者入口见 [`AGENTS.md`](../../AGENTS.md)。本目录承接更细的规则设计；当 `CONTRIBUTING.md` 中某类规则继续增长时，应逐步迁移到本目录下的专题文件，并在 `CONTRIBUTING.md` 中保留摘要和链接。

本目录文件不使用数字编号，也不要求从上到下通读。推荐阅读方式是：先根据当前任务类型选择最相关的规则文件；如果不确定该读哪一个，进入 [`index.md`](./index.md) 按任务查找。

## 阅读建议

- 想理解规则层为什么存在、与 capability layer / design layer 如何分工：读 [`../design/contributing-design.md`](../design/contributing-design.md)
- 想按任务查找具体规则文件：读 [`index.md`](./index.md)
- 想理解规则背后的设计原理与常见误读：按需进入 `intent/`
- 想看复杂真实样本与误判复盘：按需进入 `cases/`
