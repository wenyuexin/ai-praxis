# GitHub Naming Counts

这份文档用于记录当前命名讨论中，已经实际查过的 GitHub 搜索命中量。

它不是命名结论，也不直接判断某个名字是否可用；它的作用是把重复度证据单独收拢，避免后续反复重查同一批词。

## 使用方式

- 这里只记录已经查过的词和数量。
- 命中量只作为重复度参考，不单独决定名字好坏。
- 即使命中量不高，如果某个词属于编程语言默认用词、基础技术占位词，仍然应优先降权或排除。
- 组合词需要同时关注组成词的拥挤度，不要只看组合后的主观感觉。

---

## 词根 / 单词

- `ai`：约 `5.2M`
- `signal`：约 `203k`
- `kernel`：约 `148k`
- `intelligence`：约 `269k`
- `forge`：约 `110k`
- `atlas`：约 `84k`
- `nexus`：约 `97.8k`
- `framing`：约 `77.8k`
- `prism`：约 `67.9k`
- `lens`：约 `58.7k`
- `horizon`：约 `44.2k`
- `parallax`：约 `43.2k`
- `foundry`：约 `41k`
- `synthesis`：约 `33k`
- `compass`：约 `34k`
- `corpus`：约 `26.2k`
- `lighthouse`：约 `24k`
- `cortex`：约 `20.2k`
- `frontier`：约 `13k`
- `contour`：约 `8.4k`
- `loom`：约 `10k`
- `confluence`：约 `8k`
- `locus`：约 `7.4k`
- `cadence`：约 `6.3k`
- `waypoint`：约 `6.2k`
- `vantage`：约 `5.9k`
- `keystone`：约 `5.1k`
- `observatory`：约 `5k`
- `compendium`：约 `5k`
- `praxis`：约 `4.9k`
- `aperture`：约 `4.2k`
- `physis`：约 `358`
- `throughline`：约 `107`

## 组合词

以下同时记录组合词本身的 GitHub 搜索命中量，以及组成词的拥挤度，便于区分“组合本身不算高频”与“模板语义仍然过于常见”这两类问题：

- `ai-compass`：组合约 `1.1k`；`ai` 约 `5.2M`，`compass` 约 `34k`
- `ai-observatory`：组合约 `161`；`ai` 约 `5.2M`，`observatory` 约 `5k`
- `intelligence-compass`：组合约 `33k`；`intelligence` 约 `269k`，`compass` 约 `34k`
- `frontier-intelligence`：组合约 `179`；`frontier` 约 `13k`，`intelligence` 约 `269k`
- `ai-lighthouse`：组合约 `215`；`ai` 约 `5.2M`，`lighthouse` 约 `24k`

---

## 当前的弱结论

- `ai-*` 组合即使自身命中量不高，也仍然处在高拥挤模板中，不能只看组合数量就放松判断。
- `signal`、`kernel`、`forge`、`nexus`、`atlas`、`framing` 已进入明显高拥挤词池。
- `lens`、`prism`、`horizon`、`parallax`、`foundry` 也都处在偏高区间，不能只凭语义顺眼就继续推进。
- `synthesis`、`compass`、`corpus`、`lighthouse`、`cortex` 处于中等拥挤区间，不适合单独作为主名，需结合模板拥挤度与语义占位继续判断。
- `intelligence-compass` 的组合命中量也已经偏高，说明它不只是组成词常见，组合本身也不够稀缺。
- `throughline`（107）极低，但这类词仍需额外检查是否过长、是否更像表达口号而不是仓库主名。
- `praxis`、`aperture`、`observatory`、`vantage`、`waypoint`、`cadence`、`keystone`、`compendium`、`locus`、`confluence`、`contour` 这类词的命中量相对更可控，但仍需结合语义占位、主观吸引力和程序员默认语义风险继续判断。

---

更新时间 2026.06.14