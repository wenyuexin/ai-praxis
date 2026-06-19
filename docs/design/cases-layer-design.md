# Cases Layer Design

本文回答：**复杂样例、误判复盘、正反例组合与跨层案例，长期应该放在哪里；`docs/contributing/cases/`、潜在的 `docs/capabilities/cases/` 与潜在的 `docs/cases/` 分别意味着什么。**

它属于 design layer：不直接迁移现有案例，也不直接规定某个案例的最终落位。本文先给出长期结构判断，避免因为当前案例数量少或现有目录已经能放，就忽略后续的语义边界与迁移成本。

## 1. 问题是什么

当前复杂案例主要放在 `docs/contributing/cases/`。这在早期是自然的，因为这些案例最初主要用于解释规则层误判，例如：

- README / index / roadmap 的边界
- `overview.md` / `landscape.md` 的误用
- `backlog.md` / `candidates.md` 的触发条件
- 混合型研究单元如何组织

但随着 `docs/capabilities/` 和 `docs/design/` 逐步成形，case 的语义开始变宽。一个案例可能同时说明：

- 某条规则为什么这样执行
- 某项 capability 在真实场景中如何发挥作用
- 某个 design 判断为什么要这样分层
- 正例和反例如何组合阅读才完整

这说明 case 不一定只是 contributing layer 的附属材料。

## 2. 为什么不能只按当前目录状态判断

按长期主义，目录结构首先服务未来持续演化后的正确落位，而不是当前文件数量或已有目录是否暂时能放。

如果继续默认所有 case 都放在 `docs/contributing/cases/`，短期最省事，但长期可能产生三个问题：

1. **把所有 case 都解释成规则案例。** 这会遮蔽 capability 正例、design 对照例和跨层组合例。
2. **让未来维护者误判入口。** 想理解某项 capability 的真实使用样式时，可能被迫进入 contributing layer 找例子。
3. **提高后续迁移成本。** 如果 case 数量增长后再拆，路径、索引、引用和阅读路径都会更难调整。

因此，case 层级问题不能只问“现有目录能不能放”，还要问“未来内容增长数倍后，这个目录名和父层身份是否仍能帮助正确落位”。

## 3. 三种候选结构

### 3.1 继续只使用 `docs/contributing/cases/`

这种方案把所有复杂样例继续视为规则层材料。

优点：

- 结构不变，迁移成本最低。
- 已有入口和规则层设计都能解释当前内容。
- 对规则误判复盘仍然很自然。

问题：

- capability 使用样例会被迫寄居在规则层。
- design 组合例会被解释成规则复盘，语义变窄。
- 当一个 case 同时说明 capability 与 contributing 问题时，读者很难判断它到底代表哪一层。

适用条件：

- case 主要还是规则误判复盘。
- capability doc 内只需要少量内嵌例子，不需要形成可复用案例对象。
- 跨层组合 case 很少。

### 3.2 新增 `docs/capabilities/cases/`

这种方案给 capability layer 自己的 case 容器。

优点：

- capability 正例可以贴近能力层。
- 读者理解某项能力时，不必进入 contributing layer 找例子。
- 对“系统能做什么、怎么用”的真实样式更友好。

问题：

- 容易和 `docs/contributing/cases/` 形成双轨 case 体系。
- 同一个案例如果同时是 capability 正例和 contributing 反例，会被迫二选一或重复维护。
- 正反例组合、能力-规则联动、design 说明可能被父目录拆散。

适用条件：

- capability case 大量增长，且大多数只服务能力理解，不涉及规则误判与设计分层。
- case 与 rules layer 交叉较少。
- 能力层需要独立的例子索引，但不需要跨层组合阅读。

### 3.3 新增独立 `docs/cases/`

这种方案把 case 视为一种独立的特殊对象层，而不是某一层的附件。

优点：

- 可以承接跨层案例、正反例组合和长期复盘对象。
- 不强迫案例归属于 contributing 或 capabilities 的单一父身份。
- 允许内部按语义重新组织，例如 `capability/`、`rules/`、`design/`、`paired/`、`research-ingestion/` 等子目录。
- 更符合“case 是一种特殊对象”的长期结构判断。

问题：

- 这是结构升级，不是局部补目录。
- 需要重新定义它和 `docs/contributing/cases/` 的关系。
- 需要新增 README / index / 迁移策略，否则容易变成公共堆场。

适用条件：

- 经常出现一个 case 同时论证 capability、contributing 与 design 问题。
- 正例与反例需要组合阅读，拆到不同父目录会损害语义完整性。
- case 自身开始成为可复用研究对象，而不只是某条规则的附属例子。

## 4. 三类 case 的侧重点

为了避免“都叫 case 所以都放一起”或“都按父目录拆开”的两个极端，需要先区分 case 的侧重点。

### 4.1 Capability case

回答：**某项能力在真实场景中如何被理解和使用。**

典型内容：

- `Ingest` 如何把外部输入分流、暂存、吸收和回流。
- `Place` 如何在多个候选目录之间做稳定落位。
- `Navigate` 如何从问题出发找到入口。
- `Maintain` 如何识别长期复发问题。

它偏正向、偏能力使用样式，不应写成规则条文复述。

### 4.2 Contributing case

回答：**真实维护场景里，规则如何体现，误判如何发生，应该如何纠偏。**

典型内容：

- 为什么某个 README 不该吞并 index 职责。
- 为什么某个材料不该直接进入正文。
- 为什么某个对象应进 `candidates.md` 而不是 `backlog.md`。
- 为什么某次结构调整需要先移动再小改。

它偏反例、边界样本、误判复盘和规则落地。

### 4.3 Cross-layer case

回答：**一个真实样本如何同时说明 capability、rule 与 design 问题。**

典型内容：

- 同一个外部调研回流案例，既说明 `Ingest` 能力，也说明 `documentation-workflow.md` 的 stop-line，还说明 `research-artifacts.md` 中 `notes/` 的结构作用。
- 一个 README/index/roadmap 案例，既说明 `Navigate` / `Structure` 能力，也说明规则层的边界约束。
- 一个正例和一个反例必须成对阅读，才能理解稳定模式。

这类 case 最难放进单一父层，也最能说明是否需要独立 `docs/cases/`。

## 5. 当前阶段的设计判断

当前不应急着新增 `docs/capabilities/cases/`。

原因是：如果未来确实出现大量 capability case，且它们与 contributing case 交叉很少，那么 `docs/capabilities/cases/` 才自然成立。但现在真正暴露的问题不是“能力层没有自己的案例目录”，而是**case 可能经常跨层组合阅读**。

当前也不应立刻迁移出 `docs/contributing/cases/`。

原因是：已有 case 主要仍是规则误判复盘，当前放在 contributing layer 有历史合理性；直接迁移会造成不必要路径 churn。

当前最合理的判断是：

1. 承认 `docs/contributing/cases/` 可能只是早期承接位，不一定是长期唯一 case 层。
2. 暂不新增 `docs/capabilities/cases/`，避免先制造双轨 case 体系。
3. 开始观察是否出现足够多 cross-layer case。
4. 如果 cross-layer case 成为常态，优先考虑 `docs/cases/` 独立层，而不是在 contributing / capabilities 两边各建一套 case。

## 6. 什么时候应升级为 `docs/cases/`

如果出现以下信号，`docs/cases/` 就比 `docs/contributing/cases/` 或 `docs/capabilities/cases/` 更合理：

- 一个 case 经常同时被 capability doc、contributing rule 和 design doc 引用。
- 正例 / 反例需要合并阅读，拆开会破坏理解。
- 案例开始按对象或模式增长，而不是只按某条规则增长。
- 新 case 的主要问题不是“属于哪个规则”，而是“它展示了哪种跨层系统行为”。
- 未来维护者需要先读案例对象，再回到 capability / rule / design 文档，而不是从某条规则反查案例。

到那时，`docs/cases/` 应被设计成独立层，而不是平铺堆场。

可能结构可以是：

```text
docs/cases/
├── README.md
├── index.md
├── research-ingestion/
├── navigation-and-structure/
├── metadata-boundaries/
└── cross-layer-patterns/
```

这只是方向示意，不是当前迁移计划。

## 7. 如果未来建立 `docs/cases/`，它应如何与其他层分工

- `docs/design/`：解释为什么 case layer 存在、整体结构如何演化。
- `docs/capabilities/`：在能力正文中保留少量短例子，用于理解能力；复杂案例链接到 `docs/cases/`。
- `docs/contributing/`：在规则正文中保留最小误判提示；复杂复盘链接到 `docs/cases/`。
- `docs/cases/`：承接可复用、可组合、可长期维护的案例对象。

也就是说，`docs/cases/` 如果成立，应是独立对象层，而不是 `contributing/cases/` 的简单搬家。

## 8. 当前执行建议

当前先不迁移目录。

更稳的做法是：

1. 暂时保留 `docs/contributing/cases/`。
2. 新增 case 前先判断它是 contributing case、capability case，还是 cross-layer case。
3. 如果只是规则误判复盘，仍可先放 `docs/contributing/cases/`。
4. 如果只是能力正文中的短例子，优先内嵌在对应 capability doc。
5. 如果是 cross-layer case，当前先不要迁移目录；仍可暂放在最接近的现有承接位（通常是 `docs/contributing/cases/`），但应在该 case 文件中用一小段说明明确：它为什么已经超出单一规则案例、涉及哪些 capability / design 维度，以及为什么这目前只被视为独立 `docs/cases/` 的候选信号。
6. `docs/contributing/cases/README.md` 可只保留很轻的一句提醒：哪些案例已带有 cross-layer signal，供后续集中复核；不要因此把 README 扩成第二份迁移台账。
7. 等 cross-layer case 累积到影响阅读和落位判断时，再迁移到 `docs/cases/`。

## 9. 本文的边界

本文不负责：

- 立刻创建 `docs/cases/`。
- 迁移现有 `docs/contributing/cases/`。
- 规定所有未来 case 的具体模板。
- 代替 `docs/contributing/cases/README.md` 解释现有案例。
- 代替 capability doc 给出能力本身的使用说明。

本文只负责先把长期结构判断说清楚：

> 如果 case 长期只是规则误判复盘，继续留在 `docs/contributing/cases/` 是合理的；如果 case 经常跨 capability、contributing 与 design 组合阅读，那么更长期的稳定结构不是 `docs/capabilities/cases/`，而是独立的 `docs/cases/` case layer。
