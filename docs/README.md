# Repository Documentation

本目录是 `docs/` 的总入口，只回答：**`docs/` 下面有哪些子目录，它们分别负责什么。**

它不展开具体贡献规则细节，也不替代 `docs/contributing/README.md` 的规则索引职责。

## 目录结构

```text
docs/
├── contributing/
├── templates/
└── test/
```

## 子目录职责

- `contributing/`：仓库建设、贡献协作、文档治理、Evidence、Traceability 等规则目录；进入后先读 `contributing/README.md` 选择具体规则文件。
- `templates/`：仓库文档模板，例如综述写作模板。
- `test/`：Markdown、Mermaid 等文档渲染兼容性测试材料。

## 阅读建议

- 想了解 `docs/` 这个目录整体做什么：读本文件。
- 想知道贡献、文档治理、README、元信息、Evidence 等规则分别在哪：进入 `contributing/README.md`。
- 想找某个具体规则文件：不要停留在本文件，直接去 `contributing/README.md` 的“先读哪篇”部分。

## 边界说明

- 机器学习与 Agentic 知识内容放在对应主题目录中，例如 `agentic/`、`llm/`、`training-infra/`。
- 仓库级强约束入口仍是根目录 `CONTRIBUTING.md`。
- `docs/README.md` 只负责 `docs/` 级别的总导航；`docs/contributing/README.md` 才负责规则索引与文件分工。
- 若未来需要保存图片、图表、附件等静态资源，再按实际使用场景创建专门资源目录。
