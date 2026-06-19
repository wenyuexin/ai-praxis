# Capability Meta

本文回答：**维护 `docs/capabilities/` 这一层时，哪些内容属于 capability layer，哪些不属于；什么时候该补能力文档，什么时候应回到 design layer 或 rules layer。**

它的作用类似 `docs/contributing/meta-rules.md` 对规则层的约束，但对象不同：这里约束的是 capability layer 的边界、增长方式与联动更新面，而不是具体维护规则。

## 1. 先守住 capability layer 的职责

`docs/capabilities/` 只承接三类内容：

- 系统提供什么公共能力
- 读者、人类维护者与 AI 协作者如何理解和使用这些能力
- 不同能力之间如何衔接，而不是只靠零散经验维持

一句话说：**capability doc 解释系统“能做什么、怎么用”，不是解释“修改仓库时必须怎么做”。**

## 2. 不要把 capability doc 写成另外三层

维护 capability layer 时，最常见的退化不是内容不够，而是内容写到了别层去。

### 2.1 不要写成第二份规则文档

下面这类内容通常不该写进 `docs/capabilities/`：

- stop-line
- 必须/禁止类硬约束
- 具体文件写法细则
- 目录迁移或重构的执行步骤
- Evidence / Trace 字段级操作说明

这些内容应回到 `docs/contributing/`。

### 2.2 不要写成第二份 design 文档

下面这类内容通常也不该由 capability doc 长篇承接：

- 为什么整套系统这样分层
- 当前能力矩阵由哪些文档承接
- 哪些能力仍在补强、优先级如何排序
- capability layer 在系统总设计中的位置

这些内容应主要留在 `docs/design/system-design.md` 与 `docs/design/capability-design.md`。

### 2.3 不要写成第二份案例库

如果内容的重点已经变成：

- 某个真实目录当时怎么演化
- 某次误判的完整复盘
- 某条规则在复杂对象里怎么落地

那它通常更适合进入 `docs/contributing/cases/`，而不是继续扩进 capability doc。若案例已经经常跨 capability、contributing 与 design 组合阅读，不要在 capability layer 内另造案例库；先参考 [`../design/cases-layer-design.md`](../design/cases-layer-design.md) 判断是否已经出现未来独立 `docs/cases/` 的升级信号。

## 3. 什么时候该补新 capability doc

不是每个反复出现的话题都值得独立成篇。更稳的判断是同时满足下面三个条件：

1. 它描述的是**稳定公共能力**，而不是某次局部操作。
2. 它面向的不只是规则维护者，也面向普通读者、维护者与 AI 协作者的共同理解。
3. 它的主要价值是帮助识别问题类型、理解能力边界或决定下一步该去哪一层，而不是直接给出执行细则。

如果缺少其中任一条件，优先考虑补现有文档，而不是新增 capability doc。

## 4. 什么时候不该新增 capability doc

以下几种情况，通常不该上升成新的能力文档：

- 只是某条主规则还不够直接
- 只是某个 intent 解释还不够清楚
- 只是某个案例已经失真，需要更新或退役
- 只是某个能力文档还缺误判样例、边界说明或使用场景
- 只是为了让目录看起来对称完整

一句话说：**capability doc 按稳定能力位增长，不按目录对称或主题名补齐。**

## 5. capability 命名必须保持稳定

能力名一旦公开成层，就不应在不同层里随意漂移。

当前应保持统一的核心能力域是：

- `Navigate`
- `Place`
- `Ingest`
- `Structure`
- `Answer`
- `Maintain`

如果设计层、入口层、能力文档之间出现命名不一致，例如一处叫 `Navigate`、另一处叫 `Retrieve`，应优先先修正命名漂移，再讨论是否真的需要能力拆分。

## 6. capability layer 变动后的最小联动面

当你新增、改名、删除或明显重写某篇 capability doc 时，至少复核下面这些文件是否也需要同步：

- `docs/capabilities/README.md`
- `docs/capabilities/index.md`
- `docs/design/capability-design.md`
- `docs/design/system-design.md`
- `docs/README.md`

如果变动影响仓库级入口理解，再继续复核：

- `AGENTS.md`
- `CONTRIBUTING.md`

这里的重点不是机械全改，而是防止 capability layer 已经长出来了，但入口、总表和设计说明还停在旧状态。

## 7. capability layer 的最小维护顺序

拿不准时，可先按下面顺序判断：

```text
发现 capability layer 相关问题
  → 这是能力边界问题，还是规则/设计/案例问题？
  → 如果留在 capability layer，它回答的是“系统能做什么、怎么用、如何分流”吗？
  → 这是补现有能力文档就够，还是已经形成新的稳定能力位？
  → 改完后，README / index / design 总表是否需要一起同步？
```

最后一步很关键。能力层最容易出现的问题，不是单篇文档写差，而是整层已经变化，但入口与设计总表没有一起更新。

## 8. 三种最常见的 capability-layer 退化

### 8.1 退化一：把 capability 写成第二份 rule

表现通常是：正文里开始出现 stop-line、字段要求、硬约束顺序或文件级禁止事项。

### 8.2 退化二：把 capability 写成 design 摘要

表现通常是：正文大段重复系统设计、能力矩阵、历史演化和补强优先级，结果真正的“这项能力怎么理解和使用”反而被挤薄。

### 8.3 退化三：入口层、能力层、设计层不同步

表现通常是：

- README 还在按旧能力集介绍
- index 没把新增能力纳入分流
- design 总表仍在沿用旧命名或旧文件树

这类问题看起来像文案小错，实际上会直接破坏这一层的可读性和可维护性。

## 9. 本文与其他文件的分工

- `README.md`：回答 capability layer 是什么、为什么存在、与相邻层如何分工
- `index.md`：回答进入 capability layer 后，按能力问题该读哪篇
- `meta.md`：回答 capability layer 自己该怎么长、怎么守边界、变动后要同步哪些面
- `../design/capability-design.md`：回答 capability layer 为什么这样设计、当前能力由谁承接、哪些仍需补强

## 10. 本文的边界

本文不负责：

- 代替单篇 capability doc 解释某项能力本身
- 代替 `docs/contributing/` 给出执行规则
- 代替 design layer 给出系统总设计论证
- 对具体一次修改直接给出最终分层结论

它只负责把 capability layer 这一层本身的维护边界说清楚，避免它在后续增长时重新混回规则层或设计层。
