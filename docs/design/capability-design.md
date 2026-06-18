# Capability Layer Design

本文回答：**这个仓库作为 knowledge agent system，为什么需要独立的 capability layer，当前依赖哪些核心能力，这些能力分别由哪些文档承接，以及哪些已经有公共能力文档。**

它属于 design layer 下的专项设计文档，不替代 `docs/contributing/` 下的专题规则，也不直接给出某条规则的完整细则。

## 1. 能力层设计的作用

把 capability layer 的专项设计单独写出来，不是为了再造一层抽象术语，而是为了避免两类长期混淆：

- 把“系统提供什么能力”和“维护者该遵守什么规则”混成同一层
- 只看到当前已有哪几篇文档，而看不到仓库长期会持续依赖哪些底层能力

因此，这份设计文档关注的是 capability layer 本身，而不是当前文件数。

## 2. 能力层不只包含能力正文

当 capability layer 逐步稳定后，它不只需要单篇能力文档，还需要至少三类彼此分开的承接位：

- `README.md`：回答 capability layer 是什么、为什么存在、与相邻层如何分工
- `index.md`：回答进入 capability layer 后，按能力问题该读哪篇
- `meta.md`：回答 capability layer 自己该怎么守边界、怎么增长、变动后要同步哪些面

这三者不直接承接某一项具体能力，但它们共同决定 capability layer 能否长期保持：

- 能力说明不混回规则层
- 能力分流不退化成另一份 README
- 能力层自己的边界知识不再散落在入口页和设计页里

因此，下面的六类核心能力，是建立在这组能力层基础承接位之上的，而不是孤立存在的几篇说明文档。

## 3. 六类核心能力

### 3.1 Navigate

回答：从问题出发，如何沿 `README.md` / `index.md` 缩小范围，找到最可能的入口或目标位置。

典型问题：

- 我只知道问题，不知道目录名，该先去哪里
- 如何先判断某个目录是否属于我的问题空间
- 进入目录后，应如何继续往下钻

当前承接：

- 运行时接口：各目录 `README.md` / `index.md`
- 公共能力文档：[`../capabilities/navigate.md`](../capabilities/navigate.md)

### 3.2 Place

回答：在已经找到稳定区域之后，这个内容应停在哪一层、应写成正文还是哪类元信息文件。

典型问题：

- 应停在当前层，还是继续下沉到更细子目录
- 应写正文、`backlog.md`、`candidates.md`、`landscape.md` 还是 `conflict.md`
- 是先保留在元信息文件里，还是已经适合进入正文

当前承接：

- [`../contributing/documentation-workflow.md`](../contributing/documentation-workflow.md)
- [`../contributing/metadata-files.md`](../contributing/metadata-files.md)
- [`../capabilities/place.md`](../capabilities/place.md)

当前状态：已有规则承接，并已补出首份独立能力文档；后续仍可继续补强示例、边界案例与用户侧说明。

### 3.3 Ingest

回答：新材料进入系统前，先如何判断类型、Evidence 状态与临时落点，不直接抢写正文。

典型问题：

- 新材料是什么类型
- Evidence 状态如何
- 今天不写正文时先放哪里
- 什么时候该先留在 `temp/`、缺口、对象队列或冲突记录里

当前承接：

- [`../contributing/documentation-workflow.md`](../contributing/documentation-workflow.md)
- [`../contributing/evidence-and-traceability.md`](../contributing/evidence-and-traceability.md)
- [`../contributing/evidence-rules.md`](../contributing/evidence-rules.md)
- [`../contributing/traceability-rules.md`](../contributing/traceability-rules.md)
- [`../capabilities/ingest.md`](../capabilities/ingest.md)

当前状态：已有较完整规则承接，并已补出首份独立能力文档；后续仍可继续补强输入类型样式、降级策略与回流边界说明。

### 3.4 Structure

回答：如何让目录树、README、index 与元信息文件边界长期保持可导航、可维护、可扩展。

典型问题：

- README 与 index 如何分工
- overview 与 landscape 如何拆分
- backlog / candidates / roadmap / conflict 何时出现
- 父层与子层导航如何分工

当前承接：

- [`../contributing/readme-rules.md`](../contributing/readme-rules.md)
- [`../contributing/metadata-files.md`](../contributing/metadata-files.md)
- [`../contributing/organization-principles.md`](../contributing/organization-principles.md)
- [`../contributing/structure-refactoring-rules.md`](../contributing/structure-refactoring-rules.md)
- [`../capabilities/structure.md`](../capabilities/structure.md)

当前状态：已有较完整规则承接，并已补出首份独立能力文档；后续仍可继续补强结构退化样式、分工误判样例与层间协作模式。

### 3.5 Answer

回答：如何基于已有知识回答问题，同时守住 Evidence 边界，区分稳定结论与未稳定材料。

典型问题：

- 什么已经是稳定结论
- 什么只是观察、推断或未验证判断
- 回答问题时何时需要回到仓库重新查找

当前承接：

- 主要体现在 AI 协作者行为
- 辅助约束来自 Evidence / Traceability 相关规则
- [`../capabilities/answer.md`](../capabilities/answer.md)

当前状态：已有行为约束，并已补出首份独立能力文档；后续仍可继续补强回答样式、边界表述与问题类型分层。

### 3.6 Maintain

回答：如何处理样本漂移、案例退役、结构老化，以及规则与文档系统的长期演化。

典型问题：

- 某个案例是否仍代表原先要说明的问题模式
- 某条规则是否需要收紧、压缩、迁移或重写
- 结构老化后应如何低风险演化

当前承接：

- [`../contributing/structure-refactoring-rules.md`](../contributing/structure-refactoring-rules.md)
- [`../contributing/meta-rules.md`](../contributing/meta-rules.md)
- [`../contributing/cases/README.md`](../contributing/cases/README.md)
- [`../capabilities/maintain.md`](../capabilities/maintain.md)

当前状态：已有规则与案例承接，并已补出首份独立能力文档；后续仍可继续补强对象级示例与生命周期判断样式。

## 4. 当前映射总表

### 4.1 能力层基础承接位

| 基础位 | 当前主要承接 | 当前状态 |
|---|---|---|
| 目录说明入口 | `docs/capabilities/README.md` | 已建立 |
| 能力分流入口 | `docs/capabilities/index.md` | 已建立 |
| 能力层自我约束位 | `docs/capabilities/meta.md` | 已建立 |

### 4.2 六类核心能力

| 能力域 | 当前主要承接 | 当前状态 |
|---|---|---|
| Navigate | `README.md` / `index.md` + `docs/capabilities/navigate.md` | 已有首份公共能力文档 |
| Place | `documentation-workflow.md` + `metadata-files.md` + `docs/capabilities/place.md` | 已有规则承接，并已补出首份能力文档 |
| Ingest | `documentation-workflow.md` + Evidence / Traceability 规则 + `docs/capabilities/ingest.md` | 已有较完整规则承接，并已补出首份能力文档 |
| Structure | README / 元信息 / 组织原则 / 结构重构规则 + `docs/capabilities/structure.md` | 已有较完整规则承接，并已补出首份能力文档 |
| Answer | AI 协作者行为 + Evidence 约束 + `docs/capabilities/answer.md` | 已有行为约束，并已补出首份独立能力文档 |
| Maintain | meta-rules + structure-refactoring + cases + `docs/capabilities/maintain.md` | 已有首份能力文档，后续仍可继续补强 |

## 5. 当前最值得继续补强的能力文档


按当前成熟度看，下一批最值得继续补强的能力文档，通常是：

1. `maintain.md`
2. `structure.md`
3. `answer.md`

原因是：

- `navigate.md`、`place.md`、`ingest.md`、`answer.md` 与 `structure.md` 已经补出首份公共文档
- `maintain` 虽已独立成文，但仍最需要继续补强对象级示例与生命周期判断
- `structure` 与 `answer` 虽已独立成文，但都仍值得继续补强误判样例、边界表述与分层样式

## 6. 使用边界

如果你当前要做的是：

- 实际修改仓库结构、README、元信息文件、Evidence 标注：优先回到 `docs/contributing/`
- 理解能力层和规则层为什么要分开：读 [`system-design.md`](./system-design.md) 与 [`../capabilities/README.md`](../capabilities/README.md)
- 理解 capability layer 自己的边界、增长方式与联动更新面：读 [`../capabilities/meta.md`](../capabilities/meta.md)
- 判断下一篇 capability doc 最值得写什么：从本文开始

它不是 capability layer 的默认入口，而是 design layer 下的一份 capability layer 专项设计文档：更适合在需要横向看能力覆盖状态、承接关系、基础承接位是否齐全，以及后续补强方向时使用。
