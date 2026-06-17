# Repository Documentation

本目录是 `docs/` 的说明入口，只回答：**`docs/` 是什么、下面有哪些稳定层、它们分别负责什么。**

它不展开具体贡献规则细节，也不替代 `docs/index.md` 的查找导航职责。

## 稳定承接位

- `capabilities/`：公共能力文档目录，用来承接这套系统提供什么能力、如何使用这些能力，以及能力之间如何衔接；进入后先读 `capabilities/README.md`，需要按能力问题查找具体文档时再读 `capabilities/index.md`。
- `contributing/`：仓库建设、贡献协作、文档治理、Evidence、Traceability 等规则目录；进入后先读 `contributing/README.md`，需要按任务查找具体规则时再读 `contributing/index.md`。
- `index.md`：`docs/` 级查找导航页；当你已经知道自己要找的是规则、能力、设计还是支撑材料时，从这里继续分流。
- `design/`：专项设计文档目录；承接系统设计说明、能力矩阵等设计侧文档，进入后先读 `design/README.md`。
- `templates/`：仓库文档模板，例如综述写作模板。
- `test/`：Markdown、Mermaid 等文档渲染兼容性测试材料。

## 阅读建议

- 想了解 `docs/` 这个目录整体做什么：读本文件。
- 想按任务查找 `docs/` 里的具体入口：读 [`index.md`](./index.md)。
- 想知道贡献、文档治理、README、元信息、Evidence 等规则分别在哪：进入 `contributing/README.md`。
- 想理解这个仓库为什么能被看成一个知识型 agent system：读 `design/system-design.md`。
- 想从问题出发快速定位知识入口：进入 `capabilities/README.md`，再读 `capabilities/navigate.md`。

## 边界说明

- 机器学习与 Agentic 知识内容放在对应主题目录中，例如 `agentic/`、`llm/`、`training-infra/`。
- 仓库级强约束入口仍是根目录 `CONTRIBUTING.md`。
- `docs/README.md` 只负责 `docs/` 级别的说明入口；`docs/index.md` 负责 `docs/` 级别的查找导航；`docs/design/README.md` 承接系统设计入口；`docs/contributing/README.md` 才负责规则索引与文件分工。
- 若未来需要保存图片、图表、附件等静态资源，再按实际使用场景创建专门资源目录。
