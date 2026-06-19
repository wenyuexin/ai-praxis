# Multi-Agent Conflict

> 适用范围：`03-multi-agent/` 范围内的术语冲突、事实分歧、结论矛盾与待核验问题
> 使用说明：本文件只记录会影响 `collaboration / coordination / communication-protocols / shared-state-and-context / organizational / competition-and-conflict` 知识组织的问题；它是研究输入，不是主干定论

## 一、待核验问题

### 1.1 Multi-Agent 是否天然优于 Single-Agent

- **冲突类型**：结论
- **冲突描述**：不同研究与案例对 multi-agent 的收益给出相反结论。一些任务类型显示明显提升，另一些则因 coordination overhead、error amplification 和 hidden costs 出现性能退化。
- **涉及范围**：`overview.md`、`coordination/`、`collaboration/`
- **为什么重要**：会决定 `03-multi-agent/` 的整体叙述方式；若处理不当，容易把 multi-agent 写成单向进化路线。
- **待核验问题**：multi-agent 的收益究竟依赖哪些变量：任务类型、基线模型能力、topology、通信成本还是工具环境？
- **建议处理方向**：优先对比 single-agent / multi-agent 的实证论文，记录正反结果及适用条件。

### 1.2 Decentralized Coordination 的定义不统一

- **冲突类型**：术语
- **冲突描述**：不同论文对“decentralized”含义不同：有的指完全 peer-to-peer，无中央规划；有的则允许层级控制和 lateral communication 并存。
- **涉及范围**：`coordination/`、`organizational/`
- **为什么重要**：若不澄清，后续 topology 比较会混淆 centralized、decentralized、hybrid 与 independent aggregation 等概念。
- **待核验问题**：哪些定义来自理论分类，哪些只是某个框架的内部叫法？
- **建议处理方向**：建立术语对照表，标注不同论文和框架对 decentralized 的具体含义。

### 1.3 MCP 与 A2A 的协议边界

- **冲突类型**：术语 / 定义
- **冲突描述**：MCP 常被误写成多智能体通信协议，但其主要解决 agent/application 与 tool/resource 的接入；A2A 更偏 agent↔agent 交互。不同资料对两者的范围和互补关系表述不一致。
- **涉及范围**：`communication-protocols/`、`overview.md`
- **为什么重要**：会影响多智能体协议层的知识组织，也会干扰 `02-single-agent/tool-use/mcp/` 与本目录之间的边界划分。
- **待核验问题**：MCP 是否只应放在 tool-use 范围内？A2A 是否代表真正的 agent↔agent 协议主线？
- **建议处理方向**：查看 MCP 官方规范、A2A 官方文档和对比分析材料，明确不同协议的职责层级。

### 1.4 Multi-Agent 成本是否近似线性叠加

- **冲突类型**：事实 / 结论
- **冲突描述**：一种直觉认为多几个 agent 只是多几次模型调用；另一类证据强调 hidden coordination costs、状态同步、重复消息传播和潜在 loop 会带来显著非线性成本增长。
- **涉及范围**：`coordination/`、`shared-state-and-context/`、`overview.md`
- **为什么重要**：若低估 hidden cost，multi-agent 文档会过度乐观，忽略真实工程代价。
- **待核验问题**：哪些成本可被直接度量，哪些属于间接成本（如 error amplification、debugging complexity）？
- **建议处理方向**：收集真实案例、系统论文和 benchmark 讨论，区分显性 token cost 与隐性协调成本。

### 1.5 Failure Modes 是否应成为主线而非附录

- **冲突类型**：结论 / 组织方式
- **冲突描述**：不少资料仍以成功案例为主叙事，把失败视作边角问题；另一类研究则开始系统化分类多智能体 failure modes，认为失败模式本身就是理解 multi-agent 的核心入口。
- **涉及范围**：`coordination/`、`collaboration/`、`shared-state-and-context/`
- **为什么重要**：这关系到 `03-multi-agent/` 的组织逻辑，是按“协作成功模式”组织，还是按“收益与失败并存的机制系统”组织。
- **待核验问题**：failure taxonomy 的成熟度如何？哪些类别已较稳定，哪些仍是探索性归纳？
- **建议处理方向**：后续若证据充分，可将 failure modes 提升为独立专题，而非散落在案例中。

---

## 二、维护规则

- 若问题已获得较稳定证据并形成主干结论，应同步更新相关文档并移除或改写对应条目。
- 若问题明显属于环境层或单智能体层，应在对应目录的 `conflict.md` 记录主条目，本文件只保留与多智能体直接相关的部分。
- 若问题暂时只影响临时调研判断，可先保留为临时冲突整理；一旦进入本目录的稳定讨论面，应尽快收束到当前 `conflict.md`，不要长期把临时整理稿当正式入口。
