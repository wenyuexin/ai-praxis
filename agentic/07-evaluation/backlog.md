# Evaluation Backlog

> 适用范围：`07-evaluation/` 的内容缺口、关键问题与待补专题
> 使用原则：本文件记录 Agent 评测与调试主题中已经识别但尚未充分沉淀的高价值问题，不是任务计划，也不表示相关结论已成为主线定论

## 一、收录规则

适合进入本文件的内容通常满足以下条件之一：

- 能帮助解释 Agent 结果质量、过程可靠性与失败边界
- 当前目录骨架已建立，但缺少足够稳定的正文专题来承接
- 属于 benchmark、人工评估、可观测性与鲁棒性之间的关键矛盾
- 不宜直接写进 `overview.md` 的稳定结论，但值得长期跟踪

不适合进入本文件的内容：

- 某一个具体框架或产品的评测结果截图（应放 `../06-frameworks-and-tools/` 的对象案例）
- 只关注排行榜热度、没有机制分析价值的话题
- 已经充分沉淀、当前没有明显争议的基础概念

---

## 二、P0：高优先级缺口

### 2.1 Success Definition and Partial Completion

- **关联目录**：`task-completion-metrics/`
- **为什么重要**：很多 Agent 任务并不是简单的“成功/失败”二元结果，部分完成、带缺陷完成、可恢复完成在真实系统里都很常见。
- **当前状态**：目录已建立，但主线中尚无系统专题承接 success definition、partial completion 与 quality threshold。
- **建议产物**：`success-definition.md`

### 2.2 Benchmark Taxonomy and Transferability

- **关联目录**：`agent-benchmarks/`、`swe-benchmarks/`
- **为什么重要**：benchmark 测到的能力是否能外推到真实任务，是整个评估目录最关键的问题之一。
- **当前状态**：已有子目录骨架，但缺少一个统一专题来讨论 benchmark 类型、任务真实性与 transferability。
- **建议产物**：`benchmark-taxonomy.md`

### 2.3 Failure Attribution and Replayable Debugging

- **关联目录**：`observability-and-debugging/`
- **为什么重要**：如果失败无法稳定归因，评测结果就难以回流到 prompt、tool、memory、environment 或 orchestration 的改进。
- **当前状态**：目录已建立，但尚无专题系统展开 trace、checkpoint、artifact、replay 和 failure attribution 的关系。
- **建议产物**：`failure-attribution.md`

### 2.4 Human Evaluation Rubrics

- **关联目录**：`human-evaluation/`
- **为什么重要**：人工评估不是自动分数的附属品，而是协作体验、可信度和交付可用性的重要来源。
- **当前状态**：目录已建立，但主线中尚无专题讨论 rubric design、review workflow 与 trust-related judgment。
- **建议产物**：`human-eval-rubrics.md`

---

## 三、P1：值得持续跟踪的专题

### 3.1 Cost-Quality-Time Frontier

- **关联目录**：`task-completion-metrics/`
- **为什么值得关注**：Agent 质量不能脱离 token、工具调用、运行时间与人工介入成本单独讨论。
- **当前状态**：值得持续比较，但目前不宜写成单一评价公式。

### 3.2 Benchmark Overfitting vs Real Utility

- **关联目录**：`agent-benchmarks/`、`swe-benchmarks/`
- **为什么值得关注**：越来越多系统会针对特定 benchmark 优化，但这不一定意味着现实可用性同步提升。
- **当前状态**：适合结合具体 benchmark 案例持续观察。

### 3.3 Safety Evaluation Beyond Prompt Injection

- **关联目录**：`safety-and-robustness/`
- **为什么值得关注**：安全评估不应只停留在 prompt injection，还应覆盖工具误用、环境扰动、状态污染与错误恢复边界。
- **当前状态**：尚需更多主线证据与专题支撑。

---

## 四、P2：保留观察线索

### 4.1 Agent-as-Judge Reliability

- **关联目录**：`human-evaluation/`、`agent-benchmarks/`
- **说明**：用 Agent 评 Agent 的做法值得长期跟踪，但当前可靠性边界尚不稳定。

### 4.2 Interactive Evaluation Loops

- **关联目录**：`observability-and-debugging/`、`human-evaluation/`
- **说明**：交互式评测、在线调试和滚动回归机制值得观察，但目前仍偏工程实践议题。

---

## 五、与现有材料的关系

- `README.md` 已建立 `07-evaluation/` 的目录骨架与主题边界。
- `overview.md` 已提供结果质量、benchmark、可观测性、人工评估与鲁棒性五个核心视角。
- 当前目录中的新增 overview 与正文专题仍以结构化归纳为主，后续需要按 `docs/contributing/evidence-rules.md` 和 `docs/contributing/traceability-rules.md` 继续补充 Evidence 状态与必要的 Trace 字段。
- 后续若某个问题获得更稳定证据，应优先拆成专题文档，而不是长期停留在 backlog。
