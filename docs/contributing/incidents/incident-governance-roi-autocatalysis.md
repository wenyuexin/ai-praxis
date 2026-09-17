# Incident: 治理层自催化 / ROI 衰减信号

- 当前状态：Active
- 是否复发：首次记录
- 记录日期：2026-09-17
- 来源：两轮外部 AI 评审 + 一次互审（ReportID 见文末）

## 问题背景

近期维护几乎全部落在控制面——规则重构、incident 记录、design 文档、入口改名与本轮多轮评审——而知识层（数据面）近乎零新增。按 [`../../design/evolution-design.md`](../../design/evolution-design.md) §3 自己的尺子（轮次 / 时间占比）衡量，近期维护的开发态占比 ≈100%。规模现状：知识正文约 522–526 篇，元层约 94 篇；导航 / 路由面 11 个文件约 869 行（其中 `docs/repo-map.md` 单篇 269 行）；design 层此前把一次性设计决策（如 `repo-naming` 选名）与系统架构文档平级混放；本轮已新建 `design/decisions/`（ADR 式设计决策家）并将 `repo-naming` 迁入，design 顶层此后只承载系统架构。

## 冲突点

治理体系有完整机制管“加规则”（[`../rules/meta-rules.md`](../rules/meta-rules.md) §1 触发 + §3 签核），但缺三样对冲累积的东西：

- **无删除 / GC 触发**：“默认少改”只压新增速率，不促成合并 / 退役，天然偏向单调累积。
- **无机械校验**：链接、孤儿、术语、LF、节号引用一致性全靠人工（多轮评审中每轮都发现真实违规）。
- **数据面零度量**：所有健康判据都盯控制面，没有一个量 522+ 篇知识本身（证据状态分布、backlog 消化、candidates 晋升）。

叠加“路由是 11 个手工维护的散文面、每次变更扇出 5–7 文件”，净效应是治理预算开始挤占知识预算——而积累知识才是本仓库存在的目的。

## 危害场景

- 元层无 GC + “默认少改” → 文档只增不减 → 单任务最小必读集持续变大。
- 散文路由手工维护 → 知识层翻倍后漂移速度超过人工修复能力。
- 健康机制全在控制面 → ROI 无法度量，尾巴摇狗。

## 待处理项（结构性建议，均 `Inferred`，待量化后于开发态窗口批处理）

1. 路由面收敛 11→3–4；`repo-map`（269 行）并入 `index`、README 留薄——勿复刻 [`../intent/navigation-files.md`](../intent/navigation-files.md) §3 记录的“README 说明+导航混写”反模式。
2. design 12→5–6：合并 contributing / capability / cases-layer 三篇分层论证；理顺 intent（为什么这条规则）与 design（为什么这套系统）的 why 双重归属。
3. capabilities 是否折叠**未定**——先量化六篇能力正文的引用率 / 路由命中，再决定，不预断。
4. 加机械校验（CI）：链接 / 孤儿 / 术语 grep / LF / 节号引用，承接位用 `docs/test/`。
5. 加元层 GC 触发：定期审计，每篇规则 / intent / design 须能指出最近触发的 incident / case 或被引用，否则进合并 / 退役。
6. 加数据面度量：证据状态分布、backlog 消化率、candidates→正文晋升率。
7. 冷启动验收固化为 `docs/test/` 测试：定义 3–5 个标准任务，定期用无记忆会话跑，记录最小必读集与路由正确性。
8. 陷阱覆盖不全：overview/landscape 的“先读 intent”下潜触发器目前只在 `metadata-files §4.4` + `intent §8`，未覆盖 `place.md` / `documentation-workflow.md` 路径；按 `navigation-design §6`「陷阱覆盖」应补齐或提炼为通用机制。
9. 根 `README.md` 目录树疑与 `agentic/` 实际目录名多处不一致（flash 评审报 8 处，未独立核实）；且根 README 不在入口协议内却是读者最自然的第一站。待核实后决定是否更新 / 纳入协议。

（评审给出的“元层 94→~55、路由 800→~300”是目标锚点，`Inferred`，到达前先量化。）

## 复发信号

- 完成典型任务的必读文档数持续上升。
- 连续多个维护窗口 0 知识产出、纯元层。
- 新增元层文档靠目录对称 / 主题补齐，而非稳定能力位 / 职责位。
- 本记录本身即一个信号：发现问题的评审循环，本身也是开发态工作。

## 关联文件

- [`../../design/evolution-design.md`](../../design/evolution-design.md)（§2/§3 日常态 / 开发态节奏与轮次尺）
- [`../rules/meta-rules.md`](../rules/meta-rules.md)（§1 加规则触发；缺删除触发）
- [`../rules/organization-principles.md`](../rules/organization-principles.md)（§10 最小必读集判据——有诊断无度量）
- [`incident-governance-layering-health-criterion.md`](./incident-governance-layering-health-criterion.md)（同源：分层健康度）
- 外部评审 ReportID：4aff5d6f-0220-4766-9a7b-464d67159b93、05de85ba-1d9e-428b-af91-68774eea680c
