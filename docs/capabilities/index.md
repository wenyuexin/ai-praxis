# Capabilities Index

本文件是 `docs/capabilities/` 的查找导航页，回答：**进入 capability layer 之后，如果我已经知道自己要理解的是哪类能力问题，接下来该读哪篇能力文档。**

它不代替 `README.md` 的目录说明职责，也不代替 design layer 下的 capability 专项设计文档。

## 目录骨架

```text
docs/capabilities/
├── README.md
├── index.md
├── meta.md
├── navigate.md
├── place.md
├── ingest.md
├── structure.md
├── answer.md
└── maintain.md
```

## 按问题找入口

- **我想从问题出发找到目录入口或目标位置，或想结合目录导航与定向搜索缩小范围**：读 [`navigate.md`](./navigate.md)；定位后，读者通常继续看 `answer.md`，贡献者继续看 `place.md` / `ingest.md`，结构维护者继续看 `structure.md` / `maintain.md`
- **我想在找到区域后判断应停在哪一层、应写什么文件**：读 [`place.md`](./place.md)
- **我想理解新材料进入系统时为什么必须先分流、降级、暂存，并在条件成熟后再回流到元信息、按作用域的研究辅助材料或正文**：读 [`ingest.md`](./ingest.md)
- **我想理解目录树、README、index 与元信息文件为什么不能重新混写**：读 [`structure.md`](./structure.md)
- **我想理解系统如何基于已有知识回答问题，同时不越过 Evidence 边界**：读 [`answer.md`](./answer.md)
- **我想理解这套系统长期如何维护、哪些对象需要持续校准**：读 [`maintain.md`](./maintain.md)

## AI 协作者默认工作流

处理目录定位、落位判断与新材料吸收问题时，按当前任务选择入口：

```text
位置不明确 → Navigate：缩小主入口或候选支路
外部输入 / 临时材料 → Ingest：判断材料类型、Evidence 与暂存、吸收路径
已具备稳定落位前提 → Place：判断层级与正文、元信息、研究辅助职责
已明确开展研究 / 组织产物 → 按 AGENTS.md 的专项规则路由
```

这些入口可按需衔接，不要求每次顺序通读。新材料先判断类型与 Evidence，再决定目标落位；具体顺序、stop-line 与文件职责由 contributing rules 承接，能力文档帮助识别任务与后续入口。

## 按角色找后续能力

- **普通读者**：通常先用 `navigate.md` 找到入口，再用 `answer.md` 判断现有知识能回答到哪里。
- **文档贡献者**：位置不明时用 `navigate.md`；新材料先用 `ingest.md` 判断处理路径，具备落位前提后用 `place.md` 判断层级与文件职责。
- **结构维护者**：通常从 `structure.md` / `maintain.md` 进入，必要时回到 `navigate.md` 检查入口链是否顺畅。
- **AI 协作者**：遵守 `AGENTS.md` 与 `CONTRIBUTING.md` 的任务路由，按上面的分支选择能力入口；已明确研究或产物组织任务时直接进入专项规则。

## 当前能力文档

- `navigate.md`：从问题出发的搜索与导航能力说明
- `place.md`：定位完成后的落位决策能力说明
- `ingest.md`：新材料进入系统时的分流、吸收与回流能力说明
- `structure.md`：让目录树与各类入口长期保持分工清楚的能力说明
- `answer.md`：基于已有知识进行回答与解释的能力说明
- `maintain.md`：系统长期维护与演化能力说明

## 最小分工

- `README.md`：回答 capability layer 是什么、为什么存在、与相邻层如何分工
- `index.md`：回答进入 capability layer 后，如果我已经知道自己要理解哪类能力问题，该读哪篇能力文档
- `meta.md`：回答 capability layer 自己该怎么长、怎么守边界、变动后要同步哪些面
- `../design/capability-design.md`：回答 capability layer 为什么这样设计、当前核心能力如何承接、后续补强状态如何

## 边界说明

- 本文件不重复 capability layer 的专项设计说明；为什么要有这一层、当前能力域与承接关系如何组织，读 [`../design/capability-design.md`](../design/capability-design.md)
- 本文件不展开单篇能力文档的完整设计理由；更细的能力边界继续留在对应能力文档
- 如果 capability layer 继续增长，本文件应优先继续补能力分流，而不是退化成另一份 README
