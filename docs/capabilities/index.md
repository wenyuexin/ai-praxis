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

- **我想从问题出发找到目录入口或目标位置**：读 [`navigate.md`](./navigate.md)
- **我想在找到区域后判断应停在哪一层、应写什么文件**：读 [`place.md`](./place.md)
- **我想理解新材料进入系统时为什么必须先分流、降级、暂存，并在条件成熟后再回流到元信息文件、对象 `notes/` 或正文**：读 [`ingest.md`](./ingest.md)
- **我想理解目录树、README、index 与元信息文件为什么不能重新混写**：读 [`structure.md`](./structure.md)
- **我想理解系统如何基于已有知识回答问题，同时不越过 Evidence 边界**：读 [`answer.md`](./answer.md)
- **我想理解这套系统长期如何维护、哪些对象需要持续校准**：读 [`maintain.md`](./maintain.md)

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
