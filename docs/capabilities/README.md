# Capabilities

本目录沉淀仓库作为 knowledge agent system 所提供的公共能力文档。

它与 `docs/contributing/` 的规则层并列，但职责不同：

- `docs/capabilities/`：回答“这套系统提供什么能力、如何使用这些能力、能力之间如何衔接”
- `docs/contributing/`：回答“维护仓库时应遵守什么规则、按什么流程执行”

因此，本目录不负责定义 stop-line、维护规则或材料处理细则；这些内容仍回到 `docs/contributing/`。

## 使用建议

遇到下面三类任务时，不要直接跳到规则细则，先经过对应能力：

- 想从问题找到目录入口、目标区域或候选支路：先读 [`navigate.md`](./navigate.md)
- 已经找到区域，想判断该停在哪一层、该写什么文件：先读 [`place.md`](./place.md)
- 手上是新材料，想判断能否进入正文，还是应先进入 `temp/`、元信息文件或对象 `notes/`：先读 [`ingest.md`](./ingest.md)

常见衔接方式：

- 如果问题还停留在“这份材料今天应不应该继续留在 `temp/`”，通常先经过 [`ingest.md`](./ingest.md)
- 如果 `Ingest` 已经把问题推进到“它不适合继续停在临时层了，那接下来该归到哪里”，再转到 [`place.md`](./place.md)
- 如果落位判断进一步涉及“主题 / 对象 / 混合型”这类研究产物边界，再回到 [`../contributing/rules/research-artifacts.md`](../contributing/rules/research-artifacts.md)

完成能力判断后，再回到 [`../contributing/README.md`](../contributing/README.md) 查具体规则、stop-line 与执行细则。

## 阅读建议

- 想理解仓库为什么要把能力层单独显式化：读 [`../design/system-design.md`](../design/system-design.md)
- 想理解 capability layer 为什么这样设计、当前核心能力如何承接：读 [`../design/capability-design.md`](../design/capability-design.md)
- 想理解 capability layer 自己的边界、增长方式与联动维护面：读 [`meta.md`](./meta.md)
- 想按能力问题继续查找具体文档：读 [`index.md`](./index.md)
- 想实际执行文档维护、Evidence、README、元信息文件判断：回到 [`../contributing/README.md`](../contributing/README.md)
