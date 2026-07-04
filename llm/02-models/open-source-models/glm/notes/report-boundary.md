# GLM-5.x 报告边界笔记

## 这份笔记承接什么

这份笔记只处理一个稳定问题：`GLM-5` 技术报告与 `GLM-5.1`、`GLM-5.2` 版本对象文之间到底是什么关系。

它不是版本对象正文，也不是新的技术报告综述；它只负责保留系列内反复出现的引用边界，避免后续把 `GLM-5` 报告误写成每一版的独立报告。

## 当前已知

### 1. `GLM-5` 有直接技术报告承接

当前已确认的系列技术报告是：

- `GLM-5: from Vibe Coding to Agentic Engineering`
- arXiv: `2602.15763`

这份报告最直接承接的是 `GLM-5` 这个基础版本节点。

### 2. `GLM-5.1` 当前没有发现独立 technical report

目前已整理的一手与高信号材料表明：

- `GLM-5.1` Hugging Face 模型页的 citation 回指 `GLM-5` 报告
- 官方博客、产品文档、模型页共同承接 `GLM-5.1` 的发布说明
- 未发现名为 `GLM-5.1 Technical Report` 的独立报告

因此更稳的写法是：

- `GLM-5` 报告是 `GLM-5.1` 的系列背景
- 不是 `GLM-5.1` 的独立技术报告

### 3. `GLM-5.2` 当前也没有发现独立 technical report

`GLM-5.2` 当前公开承接方式仍主要是：

- 官方博客
- 产品文档
- Hugging Face 模型页
- release note
- 系列仓库

已整理材料也表明其 citation 仍回指 `GLM-5` 报告，而不是一个 `GLM-5.2` 专属报告。

## 当前更稳的引用边界

后续在写 `GLM-5.1`、`GLM-5.2` 或 `glm-5.x-evolution.md` 时，可优先遵守下面这个边界：

- `GLM-5`：可直接承接 `GLM-5` 报告中的系列技术路线背景
- `GLM-5.1` / `GLM-5.2`：优先承接各自发布页、产品文档、模型页、release note 中再次出现的版本事实
- `GLM-5` 报告：只作为后续版本的系列背景，不自动变成后续版本的独立事实来源

## 当前最容易误写的地方

### 1. 把“共用 citation”误写成“共用独立报告”

模型页要求引用同一份报告，并不等于：

- 每个版本都有各自报告，只是暂时共用 citation
- 或者 `GLM-5.1` / `GLM-5.2` 可以完整继承 `GLM-5` 报告中的全部数字与训练细节

### 2. 把系列技术路线背景写成版本独占事实

当前更容易安全复用的是：

- `agentic engineering`
- long-horizon task
- tool-use / coding / 长上下文路线
- MoE / sparse attention / DSA 相关系列背景

但不应直接复用成后续版本独占事实的是：

- `GLM-5` 报告里的具体参数规模
- `GLM-5` 报告里的训练 token、benchmark 和训练流程细节
- 尚未在后续版本自己页面再次明确出现的实现细节

## 待确认

1. `GLM-5.2` 后续是否会补出独立 technical report 或更完整 model card
2. `GLM-5.1` 是否存在更正式但目前未进入主链路的 system card / report 型材料
3. 系列仓库未来是否会出现更明确的“base report vs version release”分工说明

## 后续动作

1. 如果后续发现 `GLM-5.1` 或 `GLM-5.2` 独立报告，先不要直接改正文，先回到这份笔记更新边界
2. 如果系列仓库 release note 明确说明某版只是在 `GLM-5` 基础上做 updated weights，可把该材料补成更强版本边界证据
3. 如果某版模型卡补出了更完整架构说明，应判断它属于“版本直接事实”还是“系列背景补充”

## Trace

- Input staging:
  - `llm/02-models/open-source-models/glm/temp/1.md`
  - `llm/02-models/open-source-models/glm/temp/2.md`
  - `llm/02-models/open-source-models/glm/temp/glm-5.1-evidence-cleanup.md`
- This note extracts a stable cross-version question from cleaned temp materials. It should not be cited as the final upstream source when a direct report page, model page, or official document can be referenced instead.
