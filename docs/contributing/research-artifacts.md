# 研究产物组织规则

本文件定义子领域研究与具体案例研究的职责边界、案例研究内部的三类产物（正文、机制专题、辅助材料）的组织方式，以及研究辅助材料的命名和使用细则。

相关入口：[`CONTRIBUTING.md`](../../CONTRIBUTING.md) / [`docs/contributing/README.md`](./README.md) / [`元信息文件模型`](./metadata-files.md) / [`材料处理与文档增改流程`](./documentation-workflow.md)。

## 1. 适用范围

本文件负责研究产物组织，不负责以下内容（这些由对应文件承接）：

- Evidence 状态判断与标注细则：见 [`evidence-rules.md`](./evidence-rules.md)
- Traceability 链路记录：见 [`traceability-rules.md`](./traceability-rules.md)
- 材料分流与回流流程（拿到材料后怎么做）：见 [`documentation-workflow.md`](./documentation-workflow.md)
- 元信息文件模型（README / overview / backlog / candidates / roadmap / conflict）：见 [`metadata-files.md`](./metadata-files.md)
- README 通用规则与展开深度：见 [`readme-rules.md`](./readme-rules.md)

## 2. 子领域研究与具体案例研究

研究文档需要同时保留"共性抽象"和"个例复杂性"，不要把具体案例压扁成子领域证据，也不要让个例流水替代主题总结。

**子领域专题**：放在最贴近主题的目录中，负责抽象共性问题、结构性矛盾、设计路径和适用边界。例如 workspace lifecycle、checkpoint、sandbox layers。

**具体案例研究**：集中放在对应对象目录中，负责保留论文、代码库、产品或 benchmark 的对象内聚性，使读者单独查看该对象时也能形成完整理解。

**桥接方式**：通过 `backlog.md`、`candidates.md`、`conflict.md`、正文中的 Evidence / Trace / Gap 段落建立互相引用。子领域文档只摘要引用案例证据，不复制完整案例分析。

### 具体案例的两种合法价值

具体案例包括热门开源仓库、核心论文、重要产品、协议或 benchmark。它们有两种合法价值：

1. **作为子领域 claim 的证据**：案例中的发现可以支撑子领域文档中的某个判断。
2. **作为独立研究对象本身**：案例本身值得系统记录，使读者不依赖子领域文档也能理解该对象。

对于后者，案例目录下的文档应保持内聚，不能只写成零散摘录。

## 3. 案例研究的三类产物

案例研究内部采用**预防式职责分层**：不要等到文档已经明显混乱后才处理。AI 或人类维护者的注意力有限，研究进行中也很难及时判断哪些材料会变成长期证据，因此默认应先给研究过程材料预留稳定落点，再决定是否把它们提炼进正文。

三类产物：

### 3.1 面向读者的对象正文

说明该论文、代码库、产品或 benchmark 的整体结构、核心机制、设计取舍、可迁移启发与主要限制。正文应尽量可独立阅读，不把完整源码搜索过程、长证据表和大量未闭合问题塞入叙事主体。

典型文件：`overview.md`、`architecture.md`。

### 3.2 机制专题正文

围绕该对象中的某个具体机制展开，如 runtime、sandbox、workspace、checkpoint、evaluation。它可以包含少量关键证据，但重点仍是解释机制与边界，不应退化为流水式核验记录。

典型文件：`<mechanism-name>.md`，如 `runtime.md`、`sandbox.md`。

### 3.3 研究辅助材料

承接源码笔记、证据表、局部核验、长引用、失败搜索、未闭合问题和临时判断。它们服务于 traceability 和后续维护，不要求像正文一样完整叙事。

典型文件：`notes.md`、`source-notes.md`、`evidence-notes.md`、`notes/`。

### 3.4 Notes 与 Evidence 的关系

`notes` 与 `evidence` 都可能包含“证据”，但它们回答的问题不同：

- **Notes 回答研究过程**：研究者或 AI 看到了什么、查了什么、哪些路径失败了、哪些观察尚未整理。
- **Evidence 回答结论支撑**：某个正文 Claim 凭什么成立、Evidence 状态是什么、来源和限制是什么。
- **Notes 是原材料池**：可以包含源码片段、搜索路径、失败搜索、长引用、临时判断和待查问题，允许杂乱和未完成。
- **Evidence 是论证结构**：应围绕 Claim / Status / Sources / Notes / Needs 等字段组织，服务正文可信度和复核。

因此，`notes/evidence.md` 或 `evidence-notes.md` 不是普通杂项 notes，而是从 notes 原材料中整理出来的 claim-source 对照层。正文中的关键判断应优先引用正文 `## Evidence` 或 `notes/evidence.md`，而不是要求读者从 `notes/general.md`、`notes/source.md` 这类过程材料中自行还原论证。

## 4. 少量证据写在哪里

少量证据不需要单独拆文件时，应放在正文的固定位置，而不是散落在叙事段落中：

- **简短来源依据**：放在对应结论后的 `Evidence` 或 `Source` 段落。
- **适用范围、限制和未闭合问题**：放在 `Limitations`、`Open Questions` 或 `Gap` 段落。
- **与子领域文档的关系**：放在 `Relation to Topics`、`Trace` 或 `Placement` 段落。
- **证据超载**：如果证据开始超过正文可读性承载，迁移到研究辅助材料，并在正文中只保留摘要和链接。

## 5. 研究辅助材料如何组织

研究辅助材料不是七层元信息文件，也不是普通正文。它们是具体案例研究中的辅助材料层。具体使用方式：

- **辅助材料很少**：不创建 `notes/`，放在专题文件末尾的 `## Research Notes`，或在案例根目录保留一个 `notes.md`。
- **中等规模辅助材料**：不创建 `notes/`，可在案例根目录使用 `notes.md`、`source-notes.md`、`evidence-notes.md` 或 `<mechanism>-notes.md`，但数量应保持很少。
- **辅助材料很多**：创建 `notes/` 子目录，所有辅助材料统一放入 `notes/`；案例根目录不再同时保留 `notes.md`、`source-notes.md`、`evidence-notes.md` 等平铺辅助文件。
- **`notes/` 内部命名**：可使用 `general.md` 承接原 `notes.md`，记录杂项研究过程和临时观察；`source.md` 承接源码核验、长引用和失败搜索；`evidence.md` 承接已整理的证据表、Claim-Source 对照与 Evidence 状态；也可按机制或来源命名为 `<mechanism>.md`、`<source-name>.md`。
- **导航边界**：不要把 `notes/` 当成正文目录，也不要在 README 目录树中把它提升为主要阅读入口；只说明它是研究辅助材料入口。

核心原则：拆分不是"每个研究点固定两篇"，但也不是"混乱后再救火"。**重要案例一开始就允许存在正文与辅助材料两个层次**；正文保持读者可读，辅助材料保持证据可追溯。是否拆成多个文件，取决于材料体量、机制数量和后续维护频率，而不是等到不可读才触发。

## 6. 推荐目录树

```text
<domain>/
├── <subfield>/
│   ├── README.md                    # 子领域定向与目录导航
│   ├── overview.md                  # 子领域共性综述：分类、边界、trade-off
│   ├── backlog.md                   # 子领域问题缺口
│   ├── candidates.md                # 候选论文、仓库、产品、benchmark
│   ├── conflict.md                  # 术语、事实、边界或结论冲突
│   ├── <topic-a>.md                 # 共性专题正文
│   ├── <topic-b>.md                 # 共性专题正文
│   └── project-studies/
│       └── <case-name>/
│           ├── README.md            # 案例目录定向，只做导航
│           ├── overview.md          # 案例整体介绍，适合人类直接阅读
│           ├── architecture.md      # 可选：研究对象的架构与关键机制总览
│           ├── <mechanism-a>.md     # 案例机制专题正文
│           ├── <mechanism-b>.md     # 案例机制专题正文
│           └── notes/               # 可选：辅助材料很多时启用；启用后辅助材料不再平铺在根目录
│               ├── README.md        # 可选：说明 notes 内部组织
│               ├── general.md       # 原 notes.md：一般研究过程材料
│               ├── source.md        # 原 source-notes.md：源码核验、长引用、失败搜索
│               ├── evidence.md      # 原 evidence-notes.md：证据表、Evidence 对照
│               ├── <mechanism-a>.md
│               └── <source-name>.md
```

## 7. 裁剪规则

不同重要程度的案例采用不同目录结构：

- **小案例**：可以只有 `README.md` + 一篇正文；少量证据放正文 `Evidence` / `Gap` 段落。
- **中等案例**：建议有 `overview.md` 或 `architecture.md`，再按机制拆少量专题；少量研究过程材料可放案例根目录的 `notes.md`，但不同时创建 `notes/`。
- **重要案例**：建议保留"读者正文 + 机制专题 + 辅助材料层"；一旦启用 `notes/`，源码核验、证据表、失败搜索和其他 notes 都放入 `notes/`，根目录只保留读者正文与机制专题。

**子领域目录不复制完整案例分析**，只在专题正文中摘要引用案例证据，并通过 Evidence / Trace / Gap 指回案例目录。

## 8. 与其他规则文件的关系

| 文件 | 与本文件的关系 |
|---|---|
| [`documentation-workflow.md`](./documentation-workflow.md) | 本文件从中迁移了"组织子领域研究与具体案例研究"的规则；工作流中保留指向本文件的简短指引 |
| [`metadata-files.md`](./metadata-files.md) | 本文件定义 `notes.md` / `source-notes.md` / `evidence-notes.md` 等辅助材料的具体组织方式；元信息模型只说明它们不是七层元信息文件 |
| [`readme-rules.md`](./readme-rules.md) | 本文件定义案例目录内的文档结构；README 规则补充说明案例目录 README 如何导航这些产物 |
| [`evidence-rules.md`](./evidence-rules.md) | 本文件不替代 Evidence 规则，只说明少量证据如何写在正文中、何时迁移到辅助材料 |
| [`traceability-rules.md`](./traceability-rules.md) | 本文件说案例目录中的 Trace / Gap 段落位置；详细 Trace 规则由该文件定义 |

## 9. 默认策略

- **重要案例预防式保留正文层与辅助材料层**，不等文档混乱再拆。
- **子领域文档只摘要引用案例证据**，不复制完整案例分析。
- **研究辅助材料不提升为新的元信息层**；它们服务于具体案例研究的 traceability。
- **拿不准目录深度时，先观察材料体量和维护频率**，按裁剪规则调整。
