# `agentic/` 当前组织体系问题说明

本文件不是通用规则文档，也不是长期保留的仓库级制度文件。

它只记录**当前 `agentic/` 目录体系中已经暴露出来的具体问题**：问题出现在哪个文件或目录、现象是什么、为什么会造成维护负担，以及当前推荐的处理方式。只有当某条判断在更多场景中被稳定复用时，才考虑回流到 `docs/contributing/` 的正式规则体系。

相关规则背景：[`docs/contributing/metadata-files.md`](../contributing/metadata-files.md) / [`docs/contributing/readme-rules.md`](../contributing/readme-rules.md) / [`docs/contributing/organization-principles.md`](../contributing/organization-principles.md) / [`docs/contributing/structure-refactoring-rules.md`](../contributing/structure-refactoring-rules.md)

> 状态说明：`已解决` = 问题本身已处理；`已完成当前处理` = 当前范围内已收敛，但不表示已上升为通用规则；`已执行` = 建议方案已经按当前范围落实；`部分完成，保留后续条件` = 已完成当前回流，但是否进一步上升为正式规则仍取决于后续复用情况。

## 1. 核心问题【已解决】：多个二级 `overview.md` 越界承担了目录体系分工说明

### 具体来源

- `agentic/README.md:17-30`
- `agentic/README.md:31-84`
- `agentic/01-foundations/README.md:49-55`
- `agentic/01-foundations/overview.md:145-150`（已收缩）
- `agentic/02-single-agent/overview.md:147-151`（已收缩）
- `agentic/03-multi-agent/overview.md:167-171`（已收缩）
- `agentic/04-human-agent-interaction/overview.md:143-147`（已收缩）
- `agentic/05-environments/overview.md:231-235`（已收缩）
- `agentic/07-evaluation/overview.md:172-176`（已收缩）
- 对照规则：`docs/contributing/metadata-files.md:90-112`

### 现象与证据

当前已经确认，这不是 `01-foundations/` 的单点问题，而是 `agentic/` 多个二级目录 `overview.md` 的共性写法问题。

这些 `overview.md` 中都曾出现类似段落：

- 标题通常写作“与其他目录的关系”
- 内容按 `02` / `03` / `05` / `06` / `07` 等目录逐条展开
- 既讲概念依赖，也在逐目录解释每个下游目录“负责什么”

这意味着它们在同时做两件事：

1. 说明领域间概念关系
2. 说明 `agentic/` 目录体系分工

其中第 2 件事并不属于 `overview.md` 的稳定职责。

### 为什么这是问题

按 `metadata-files.md`：

- `README.md` 负责“这是什么地方、怎么进入”
- `overview.md` 负责“这个领域的全貌和现状是什么”

因此，`overview.md` 可以讲领域间概念依赖，但不应系统性承担**目录体系分工说明**。一旦多个 `overview.md` 都逐目录解释职责分工，就会带来：

- 同一组关系需要在 `agentic/README.md`、二级目录 `README.md`、二级目录 `overview.md` 中重复维护
- 目录边界调整或重构后，容易漏改部分文件
- `overview.md` 从领域综述滑向目录体系说明，文件边界变得不清晰

### 当前处理状态

当前已经完成以下收缩：

- `agentic/01-foundations/overview.md`
- `agentic/02-single-agent/overview.md`
- `agentic/03-multi-agent/overview.md`
- `agentic/04-human-agent-interaction/overview.md`
- `agentic/05-environments/overview.md`
- `agentic/07-evaluation/overview.md`

处理方式统一为：

- 把“与其他目录的关系”改写为“与其他领域的概念关系”
- 只保留概念层面的上游/下游依赖总说明
- 不再逐目录展开职责分工
- 明确目录分工与导航入口由 `agentic/README.md` 和各目录 `README.md` 承担

### 当前推荐处理方式

- 保留 `agentic/README.md` 作为 `agentic/` 全局分工主入口。
- 保留各二级目录 `README.md` 中最小必要的导航级、归属级边界说明。
- `overview.md` 只保留概念层面的领域关系，不再逐目录解释 `agentic/` 内部职责分工。

### 例外说明：`06-frameworks-and-tools/overview.md`

`agentic/06-frameworks-and-tools/overview.md:112-128` 当前仍保留为例外，因为这一段写的是：

- `06` 为什么必须保持 object-centric
- 它与 `02/03/05/07` 的知识边界如何划分

这属于 `06` 作为对象视角目录的核心边界定义，而不是普通的目录导航替代。因此当前保留，后续如需再调整，也应按“领域核心边界是否不可缺”而不是按标题是否包含“关系”机械处理。

### 如果后续需要回流为规则，最轻量的回流位置

这类问题本质上已经被 `metadata-files.md` 的 L3 `overview.md` 职责定义覆盖：`overview.md` 讲领域全貌和现状，而不是目录结构说明。

当前已完成的最轻量回流是：

- 在 `docs/contributing/metadata-files.md` 的 L3 `overview.md` 说明中补了显式排除：
  - `overview.md` 不负责目录体系分工说明或跨目录导航
  - 可以讲概念依赖，但不逐目录解释下游目录职责分工

因此，后续不需要再为这类问题新建额外规则，除非出现新的变体超出了现有 L3 定义的覆盖范围。

## 2. 元问题【当前处理完成，未上升为通用规则】：这类问题已经从单点争议演变为 `agentic/` 层面的共性模式，正式规则只需最小必要补强

### 具体来源

- 当前已实际处理并完成收缩的文件集中在：
  - `agentic/01-foundations/overview.md`
  - `agentic/02-single-agent/overview.md`
  - `agentic/03-multi-agent/overview.md`
  - `agentic/04-human-agent-interaction/overview.md`
  - `agentic/05-environments/overview.md`
  - `agentic/07-evaluation/overview.md`
- 对照规则：`docs/contributing/organization-principles.md` 中“规则沉淀也要分层”相关段落

### 现象

现在可以确认：这已经不是 `agentic/01-foundations/` 的偶发写法，而是 `agentic/` 多个二级目录在写 `overview.md` 时容易出现的共性偏移。

但它仍然主要集中在 `agentic/` 体系内，尚未证明这是其他顶层知识域都会稳定出现的问题。

### 为什么这是问题

如果现在就把这次 `agentic/` 内部收敛直接写成宽泛的仓库级制度，会出现两个风险：

- 把 `agentic/` 当前体系的局部模式误写成全仓库共识
- 形成泛化过度的规则，反而不利于后续在其他领域按实际情况判断

### 当前推荐处理方式

- 先把问题明确记录在本文件中，限定为 `agentic/` 当前已验证出的共性问题。
- 正式规则层面只做最小必要补强（当前已补到 `metadata-files.md` 的 L3 定义），不再额外新增平行规则文件。
- 等后续在其他顶层知识域也出现同类问题，且处理方式稳定复用后，再考虑是否需要把这条边界写成更显式的仓库级说明。

## 3. 当前建议的最小改动方案【已执行】

基于当前已验证的问题模式，最小改动方案是：

1. 保留 `agentic/README.md` 作为 `agentic/` 全局分工主入口。
2. 保留各二级目录 `README.md` 中最小必要的“与其他目录的关系”或边界说明，因为它们仍属于导航级与归属级信息。
3. 保持各二级目录 `overview.md` 只写领域概念关系，不再逐目录承担 `agentic/` 内部职责分工说明。

## 4. 当前回流状态与后续条件【部分完成，保留后续条件】

当前已经完成的正式规则回流是：

- 在 `docs/contributing/metadata-files.md` 的 L3 `overview.md` 说明中补了显式排除：
  - `overview.md` 不负责目录体系分工说明或跨目录导航
  - 可以讲概念依赖，但不逐目录解释下游目录职责分工

后续只有在满足以下条件时，才建议继续把本文件中的判断进一步上升为更显式的仓库级规则说明：

- 同类问题在 `agentic/` 以外的目录中重复出现
- 处理方式在多个场景下都稳定有效
- 已经能抽象成不依赖当前 `agentic/` 背景的通用规则

否则，本文件应继续保持为**当前问题澄清文档**，而不是继续扩写正式规则文档。