# docs/DEVELOPING.md — 开发态入口（改控制面）

本文件是**开发态入口**：当你这次的任务是**改控制面本身**——规则、结构、设计、方法论——从这里进。它是第二跳入口，由根 [`AGENTS.md`](../AGENTS.md) 的模式分流指过来；为什么把开发态单独设入口，见 [`design/runtime-development-design.md`](./design/runtime-development-design.md)。

注意与 [`../CONTRIBUTING.md`](../CONTRIBUTING.md) 的区别：CONTRIBUTING 是**控制面规则书**，你**贡献知识（运行态）时读它、遵守它**；本文件是你**要改那本规则书 / 控制面（开发态）时**走的入口。本文件只做**路由 + 下潜触发器**，不复制任何 owner 的条文。

## 1. 先确认：你真的在开发态吗

- **在开发态**（继续读本文件）：改规则 / 改能力边界 / 改目录结构 / 改设计 / 改方法论——即动到控制面本身。
- **不在开发态**（别从这里走）：**贡献知识**（写笔记、`ingest` 灌材料、整理综述）是运行态 × 数据面——读 [`../CONTRIBUTING.md`](../CONTRIBUTING.md) 规则书 + 走 [`capabilities/`](./capabilities/README.md)；**只是取用 / 导航知识**回根 `AGENTS.md`。

判据：**这次会不会改到控制面文件本身**（`contributing/` 规则、`design/`、能力边界）？改，才是开发态；只是遵守它们去写知识，是运行态。

## 2. 下行路由：按“要改什么”进对应 owner

- **规则该不该改、怎么最小改**：读 [`contributing/rules/meta-rules.md`](./contributing/rules/meta-rules.md) + [`contributing/intent/meta-rules.md`](./contributing/intent/meta-rules.md)。
- **capability layer 自己如何增长、守边界、联动更新面**：读 [`capabilities/meta.md`](./capabilities/meta.md)。
- **改设计 / 理解为什么这样分层**：从 [`design/README.md`](./design/README.md) 进，系统全貌见 [`design/system-design.md`](./design/system-design.md)。
- **改方法论 / 运行态与开发态如何交替**：读 [`design/evolution-design.md`](./design/evolution-design.md)。
- **运行态/开发态、控制面/数据面的边界，或开发态入口本身**：读 [`design/runtime-development-design.md`](./design/runtime-development-design.md)。
- **导航 / 索引体系怎么搭、怎么校验**：读 [`design/navigation-design.md`](./design/navigation-design.md)。
- **结构是否长期合理、分类轴要不要调**：读 [`contributing/rules/organization-principles.md`](./contributing/rules/organization-principles.md)。
- **真正执行目录迁移 / 重构**：读 [`contributing/rules/structure-refactoring-rules.md`](./contributing/rules/structure-refactoring-rules.md)。
- **拿不准读哪篇规则**：回 [`contributing/index.md`](./contributing/index.md) 按任务找。

## 3. 下潜触发器：动笔前必须先加载（否则大概率误判）

- **改任何规则前**：先确认命中 `meta-rules.md` §1 的触发信号（复发 / 直接观察 / 高危维度）；命中高危维度要写具体场景并记入 `contributing/incidents/`，大改需 §3 维护者签核。**没命中触发信号就先别改。**
- **要写 / 读 `overview.md` 或 `landscape.md`**：先读 [`contributing/intent/overview-landscape.md`](./contributing/intent/overview-landscape.md)——高频误读点（landscape 是文档体系视图，不是主题内容板块图）。
- **判断某个 Claim 的证据状态**：若它属于“指称对象本身模糊、定不死”的一类，读 [`contributing/rules/evidence-assessment-rules.md`](./contributing/rules/evidence-assessment-rules.md) §4.2，别硬塞进“等待补证”。

## 4. 止步线

- 本文件是**入口**，不是规则本身：条文与 stop-line 一律回到上面链接的 owner。
- 它**不替代**根 `AGENTS.md` 的通用硬约束（证据纪律、正文 vs `contributing/` 落位、只改相关文件、提交前自检）——那些两态都成立。
- 本文件只管**改控制面（开发态）**；**贡献知识（运行态）**读 [`../CONTRIBUTING.md`](../CONTRIBUTING.md) 规则书，不从这里走。
