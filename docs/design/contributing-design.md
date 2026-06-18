# Contributing Layer Design

本文回答：**为什么仓库需要 `docs/contributing/` 这一层，它在整套 knowledge agent system 中承接什么问题，又为什么要继续拆成主规则、`intent/`、`cases/` 等不同子层。**

它属于 design layer 下的专项设计文档，不替代 `docs/contributing/README.md` 的规则入口职责，也不代替任何单条规则本身。

## 1. 为什么需要 `docs/contributing/`

如果这个仓库只有知识目录，而没有独立的规则层，长期会反复出现三类问题：

- 写作者不知道新材料该怎么分流、该写正文还是元信息文件
- 结构调整只能靠口头记忆，无法稳定复用
- AI 与人类协作者会在 README、Evidence、目录边界、案例退役等问题上反复犯同类错误

因此，仓库需要一个专门承接**规则、方法、约束与维护流程**的层，而不是把这些说明散落在主题目录里，或者全压在根 `CONTRIBUTING.md` 一处。

`docs/contributing/` 的存在，不是为了堆规则文件，而是为了让这套系统的运行方式本身可维护、可复用、可迭代。

## 2. `docs/contributing/` 在系统中的角色

从整套仓库结构看，`docs/contributing/` 更像 rules / methods layer。

它主要回答：

- 维护仓库时应遵守什么规则
- 处理新材料时应按什么顺序判断
- README、元信息文件、Evidence、结构调整分别有什么边界
- 规则本身何时该调整、如何最小改动

它不回答的则包括：

- 某个主题目录中的领域知识结论是什么
- 某项公共能力本身如何被理解和使用
- 某个专项设计为什么在系统层面要这样分层

所以它与相邻层的最小分工是：

- 主题目录：承接知识正文与对象内容
- `docs/capabilities/`：承接公共能力说明
- `docs/design/`：承接系统级与专项设计说明
- `docs/contributing/`：承接规则、方法与维护执行约束

## 3. 为什么不能只靠根 `CONTRIBUTING.md`

根 `CONTRIBUTING.md` 的地位是特殊的，但它不适合无限膨胀成整套规则系统。

它更适合承接的是：

- 仓库级最短硬约束
- 首次进入时必须知道的原则
- 指向下层专题规则的总入口

而不适合长期承接的是：

- README 细则
- 元信息文件完整模型
- Evidence / Traceability 细分规则
- 结构重构操作细则
- 复杂案例与误判复盘

因此，`docs/contributing/` 的存在，本质上是在保护根 `CONTRIBUTING.md` 继续保持“仓库级入口”身份，而不是退化成一个不断变长的规则杂糅页。

## 4. 为什么 `docs/contributing/` 内部还要继续分层

`docs/contributing/` 不是一个单平面目录，因为规则系统内部至少有三类不同问题：

1. **直接执行规则**：现在动手时应该怎么做
2. **设计意图与误读防护**：为什么规则这样定、最容易被误读成什么
3. **复杂真实样本**：在边界场景里，这类规则具体怎样体现

如果把这三类内容全写进主规则，会出现两个问题：

- 主规则越来越重，失去快速执行入口的作用
- 解释、案例和规则条文本身互相缠绕，后续更难维护

所以内部继续分层是必要的。

## 5. 主规则、`intent/`、`cases/` 的分工

### 5.1 主规则

主规则回答：**现在该怎么做。**

它应优先保留：

- stop-line
- 最小顺序
- 文件职责边界
- 直接可执行的判断与约束

它不应承担过多长解释或对象级复盘，否则会失去“快速执行规则”的功能。

### 5.2 `intent/`

`intent/` 回答：**为什么这样定，最容易被误解成什么。**

它适合承接：

- 规则背后的设计原理
- 一些短规则无法完整表达的概念边界
- “不要机械补齐”这一类容易被字面化执行的原则说明

它不是 capability doc，也不是主规则的重复摘要，而是规则层内部的长解释位。

### 5.3 `cases/`

`cases/` 回答：**复杂真实场景里，这些规则具体怎么体现。**

它适合承接：

- 高价值边界样本
- 误判复盘
- 某种目录模式为什么会先长出某种文件组合
- 已有规则在对象级场景里怎样被验证或纠偏

它不是新的抽象规则层，而是帮助维护者在真实样本中建立边界感。

## 6. `docs/contributing/` 与 `docs/capabilities/` 为什么要分开

这两层最容易混淆，因为它们都在回答“系统怎么工作”。

但它们的核心区别是：

- `docs/capabilities/`：回答“系统提供什么能力、如何理解和使用这些能力”
- `docs/capabilities/meta.md`：回答 capability layer 自己该怎么守边界、怎么增长、变动后要同步哪些面
- `docs/contributing/`：回答“真正维护仓库时，应该按什么规则和流程执行”

如果混在一起，最常见的退化是：

- capability doc 写成第二份规则文档
- 规则文档反过来承担能力说明
- capability layer 的边界知识仍然散落在入口页和设计页里，没有独立自我约束位
- 维护者不知道自己现在是在学系统能力，还是在执行具体约束

因此，两层并列存在是必要的。

## 7. `docs/contributing/` 与 `docs/design/` 为什么也要分开

`docs/design/` 关注的是：

- 为什么系统要这样分层
- 哪些专项设计值得独立成文
- 设计层本身如何组织

`docs/contributing/` 关注的是：

- 这套已确定的分层在维护实践里如何执行
- 什么时候改规则，怎么改规则
- 具体约束与流程细则是什么

两者的区别可以压成一句话：

- `docs/design/` 偏系统设计与专项设计说明
- `docs/contributing/` 偏规则执行与维护方法

## 8. 当前最值得继续守住的边界

按当前仓库状态，`docs/contributing/` 最值得继续守住的边界包括：

- 不把 capability 说明重新吞回规则层
- 不把长解释全部塞回主规则，而是继续利用 `intent/`
- 不把复杂样本泛化成新规则，而是继续利用 `cases/`
- 不让根 `CONTRIBUTING.md` 重新膨胀成所有规则的默认正文

这些边界守住了，`docs/contributing/` 才能继续像一层稳定的 rules / methods layer，而不是重新变成“所有治理内容都往这里堆”的杂物层。

## 9. 阅读建议

- 想知道 `docs/contributing/` 在整套系统里的设计角色：读本文
- 想实际进入规则层按任务找规则：读 [`../contributing/README.md`](../contributing/README.md)
- 想理解整个仓库为什么这样分层：读 [`system-design.md`](./system-design.md)
- 想看 capability layer 的专项设计文档与横向总表：读 [`capability-design.md`](./capability-design.md)
- 想看 capability layer 自己如何守边界、如何增长：读 [`../capabilities/meta.md`](../capabilities/meta.md)
