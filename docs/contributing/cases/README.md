# 规则案例与误判复盘

本目录存放 `docs/contributing/` 主规则的**补充案例、误判复盘与边界样例**。

它的作用不是提供新的默认执行规则，而是：当主规则已经给出 stop-line，但仍需要更长的背景、反例、复发错误模式或对象级案例时，提供一个**非默认入口**的补充层。

## 放什么

- 主规则无法承载、但值得长期保留的误判复盘
- 规则边界仍容易被误读时的对象级案例
- 用于解释“为什么这次不应套用某条看似相关规则”的具体实例

## 不放什么

- 主规则本体
- 日常任务记录
- 临时决定、一次性批次说明
- 还没有形成稳定问题模式的零散感想

## 什么时候才读

只有在以下场景才进入本目录：

- 主规则已经读过，但对象边界仍存在争议
- 需要复盘某类 AI 反复出现的误判
- 需要用具体案例说明一条短规则为什么成立

如果只是日常执行，请先回到 `docs/contributing/README.md` 选择主规则文件，而不是默认进入这里。

如果你已经进入本目录，但不知道先看哪篇，可先按下面顺序看：

- 先看 [`pageindex-hybrid-study-unit.md`](./pageindex-hybrid-study-unit.md)：这是“研究单元边界”样本，回答为什么有些对象不能只按“纯主题 / 纯对象”二分。
- 再看 [`interdisciplinarity-readme-index-roadmap.md`](./interdisciplinarity-readme-index-roadmap.md)：这是“复杂目录拆分”样本，回答为什么 `README.md`、`index.md`、`roadmap.md` 必须分开承接不同认知任务。

## 当前规划

- [`pageindex-hybrid-study-unit.md`](./pageindex-hybrid-study-unit.md)：`PageIndex` 作为“主题+对象混合型研究单元”的首个完整边界案例，说明为什么它既不能被压成纯主题，也不能只被当成普通对象研究。
- [`interdisciplinarity-readme-index-roadmap.md`](./interdisciplinarity-readme-index-roadmap.md)：`interdisciplinarity/` 作为复杂横向目录的拆分案例，说明为什么 `README.md`、`index.md`、`roadmap.md` 必须分别承接目录定位、结构导航与路径分流，以及何时只出现 `overview` 信号而不机械创建 `overview.md`。
- 后续如再出现同类边界问题，可继续在本目录沉淀案例；没有稳定问题模式时，不主动扩充。
- 如果后续再次出现“规则本身清楚，但 AI 在长任务中仍反复把 `README.md`、`index.md`、`overview.md` 等文件职责重新混写”的情况，可考虑补一个关于"结构语义优先于规则记忆"的误判复盘案例；在问题尚未形成稳定模式前，不单独建新案例文件。

## 与主规则、`intent/` 的关系

- 主规则负责：路由、职责边界、stop-line、默认执行顺序
- `intent/` 负责：解释规则背后的设计原理、原始意图与概念边界，防止对条文本身产生抽象误读
- 本目录负责：边界案例、误判复盘、对象级样本、反例整理，防止执行时在复杂场景里反复漂移

因此，主规则应保持短；`intent/` 解释“为什么这样定”；本目录说明“复杂场景里具体怎么用”。
