# Frameworks and Tools（框架与工具）

本目录用于沉淀 Agent 工程生态中的具体对象：通用框架、编码工具、完整项目案例、Skill/Tool 系统与横向对比。

## 边界说明

`06-frameworks-and-tools/` 不负责重新讲解 Agent 的底层能力体系，而负责分析现实框架、工具和项目如何实现、组合、产品化这些能力。底层能力理论（planning、memory、tool-use、multi-agent、environment、evaluation 等）回到 `agentic/` 对应层级；本目录只保留工程实现引用。

## 入口说明

- 第一次进入本目录时，先用本页判断它是否与你当前的问题匹配
- 想先恢复本目录的整体理解与主线判断时，继续转到 `overview.md`
- 想查目录结构、子目录定位与后续入口时，转到 [`index.md`](./index.md)

## 分类依据

`06-frameworks-and-tools/` 下的编号表达本目录内对象类型的稳定分组：

- `01-frameworks/`：通用框架与编排框架
- `02-coding-agents-and-tools/`：编码 Agent 工具、产品与平台
- `03-project-studies/`：完整 Agent 系统与开源项目案例
- `04-skill-and-tool-systems/`：Skill、Tool、插件与能力注册系统
- `05-comparisons/`：横向比较、选型与生态地图

这些编号同时提供一个轻量阅读顺序，但不代表其他 `agentic/` 二级领域也需要采用编号三级目录。

## 归属规则

本目录内子目录之间的归属判断（如 `02-coding-agents-and-tools/` 与 `03-project-studies/` 的分工、开源项目研究的组织方式），稳定部分由本页、`index.md`、`overview.md` 或相应专题承接；仍未稳定且无法由既有 owner 承接的结构判断，再按治理规则评估研究辅助材料。

最小归属判断如下：

- 可复用框架、SDK 与编排平台优先进入 `01-frameworks/`；
- 面向软件工程任务的产品与工具优先进入 `02-coding-agents-and-tools/`；
- 需要系统拆解完整运行时、工具链、记忆、网关或部署面的项目优先进入 `03-project-studies/`；
- Skill、Tool、插件与能力注册系统进入 `04-skill-and-tool-systems/`；
- 跨对象的选型、比较和边界收口进入 `05-comparisons/`。

同一对象同时具备产品视角和系统研究价值时，系统性架构研究以 `03-project-studies/` 为主，另一侧只保留必要入口或交叉引用；不要复制完整对象分析。

---

*最后更新: 2026-06-16*
