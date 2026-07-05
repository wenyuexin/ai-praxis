# Incident: arXiv Source Package 的 Sources 与 Trace 写法

- 当前状态：Resolved
- 是否复发：未复发
- 记录日期：2026-07-05

## 问题背景

在整理 `llm/02-models/open-source-models/glm/glm-5-report.md`（GLM-5 技术报告研究文档）时，研究者基于 arXiv HTML 全文和 LaTeX source package 补全了公式、表格和训练细节。LaTeX source package 被下载到本地 `temp/` 目录用于提取。

## 冲突点

最终文档的 Evidence 节出现了两种不理想写法：

1. **把本地 `temp/` 路径写进正式 Sources**：`temp/arXiv-2602.15763v2.tar.gz` 作为来源条目出现在 Sources 中。`temp/` 是暂存区，不是稳定来源端点——它会被清理，且对其他协作者不可共享。
2. **把过程性说明混写进 Sources**：`（LaTeX source，用于补全 HTML 版本中不便直接提取的公式、表格与训练细节）` 作为括号注释写在 Sources 条目中。"用什么工具补全了什么"是过程信息，属于 Trace 的职责，不属于 Sources。

当前 `traceability-rules.md` §3.1 的 stop-line 只覆盖了"不要写仓库外本机绝对路径"，没有显式覆盖"不要写仓库内 `temp/` 路径"和"过程性说明写进 Trace 不写进 Sources"。

## 解决方案

### 规则层面

在 `traceability-rules.md` §3.1 Source 中：

1. 将两个 stop-line 合并为一条通用 stop-line：**"正式 Trace、Evidence、Version Basis 和类似来源字段中，不要写本机绝对路径或 `temp/` 暂存路径。"**
2. 在解释段落中补一句区分：稳定仓库内路径（如 `docs/contributing/xxx.md`）可以用于正式 Trace；`temp/` 是暂存区，不是稳定来源端点
3. 补一句通用规则：当上游提供 source package（如 arXiv LaTeX source、GitHub release assets）时，优先引用上游链接；过程性说明写进 Trace，不写进 Sources
4. 删掉了最初添加的 arXiv 专属推荐示例和错误示例——arXiv 是具体来源，不是通用规则；具体来源的写法属于 case 级内容，不应放进主规则

在 `intent/traceability-rules.md` §8 中：

5. 补了一句解释：`temp/` 路径同理——它是输入暂存与提取过程中的中间站，不是稳定来源端点

### 文档层面

`glm-5-report.md` 的 Evidence 节修正为：

```md
- Sources:
  - https://arxiv.org/abs/2602.15763
  - https://arxiv.org/html/2602.15763v2
  - https://arxiv.org/e-print/2602.15763
- Trace: Technical details extracted from arXiv HTML and LaTeX source package. The LaTeX source was used to recover formulas, architecture spec tables, and training hyperparameters not directly extractable from the HTML rendering.
```

## 通用可迁移判断

- 任何上游提供 source package 的场景（arXiv、GitHub release、conference proceedings 等）都适用同一规则：Sources 用上游链接，Trace 写过程说明
- `temp/` 路径出现在正式 Sources 中是一个通用误判模式，不限于 arXiv 场景
- 过程性说明混入 Sources 也是一个通用误判模式——Sources 回答"来自哪里"，Trace 回答"怎么处理的"

## 关联文件

- `docs/contributing/rules/traceability-rules.md` §3.1 Source（规则修改）
- `docs/contributing/intent/traceability-rules.md` §8（intent 补充）
- `llm/02-models/open-source-models/glm/glm-5-report.md` Evidence 节（文档修正）
