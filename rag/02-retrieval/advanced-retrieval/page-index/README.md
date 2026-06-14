# PageIndex

长文档层级树索引与推理导航检索框架（VectifyAI）。

## 定位

这里是 **PageIndex 的对象研究目录**。PageIndex 是一个具体的工程对象，不是泛化范式名。

它放在 `advanced-retrieval` 下，是因为它对这个方向有高研究价值——它是某个重要索引思路（可概括为 page-level / structure-aware indexing）的高信号载体。但这**不等于**该思路已经在开源生态中形成成熟、丰富、可充分横向比较的对象簇。

本目录重点研究 PageIndex 本体，不替代该方向的完整主题综述。它不是与同层主题目录完全对称的通用主题节点。

## 正文与专题

- [overview.md](./overview.md) — 对象总览：它是什么、核心机制、系统边界
- [pdf-indexing.md](./pdf-indexing.md) — PDF 树索引构建机制
- [tree-and-navigation.md](./tree-and-navigation.md) — 树结构抽象、构树收敛路径与本地导航行为链
- [retrieval-protocol.md](./retrieval-protocol.md) — 检索协议、树导航模式、示例层 reasoning pattern
- [sdk-and-workspace.md](./sdk-and-workspace.md) — 本地 SDK 架构、workspace、多文档存储、Cloud API 边界

## 辅助材料

[notes/source.md](./notes/source.md) 和 [notes/evidence.md](./notes/evidence.md) 存放源码入口、Claim-Source 对照与未闭环问题，供深入核实使用。[notes/deep-research-question-tree.md](./notes/deep-research-question-tree.md) 用于沿主线递进展开机制级深度研究；[notes/reflections.md](./notes/reflections.md) 用于记录基于当前对象现状形成的阶段性理解、直觉与待压实假说。

---

*最后更新: 2026-06-14*
