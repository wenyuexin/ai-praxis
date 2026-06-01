# Single-Agent Backlog

> 适用范围：`02-single-agent/` 的内容缺口、前沿问题与待补专题
> 使用原则：本文件记录已识别但尚未充分覆盖的知识空白，不是任务管理列表，也不代表相关结论已进入主干定论

## 一、收录规则

适合进入本文件的内容通常满足以下条件之一：

- 已被论文、官方文档、源码或高质量调研明确指出有研究价值
- 能补齐 `planning / memory / tool-use / reflection / patterns` 之间的关键桥梁问题
- 代表 single-agent 研究中的核心矛盾、常见误区或新兴分岔
- 尚不适合直接写入主干正文，但值得继续跟踪

不适合进入本文件的内容：

- 具体框架实现细节（应放 `../06-frameworks-and-tools/`）
- 已形成稳定主干知识且没有明显争议的内容
- 只靠单篇博客、单个 demo 或热度数据支撑的判断

---

## 二、P0：高优先级缺口

### 2.1 Planning 与 Execution 的耦合/解耦

- **关联目录**：`planning/`、`tool-use/`、`reasoning-and-acting/`
- **为什么重要**：当前主干已经覆盖 planning 和 tool use，但二者之间的接口问题还没有形成独立专题。`ReAct` 式紧耦合与 bilevel / decoupled orchestration 是 single-agent 扩展复杂任务时的关键分岔。
- **现状**：在 `overview.md` 中已被识别为核心矛盾，但缺少专门文档。
- **建议产物**：`planning-vs-execution.md` 或 `tool-orchestration.md`

### 2.2 Memory Taxonomy 冲突与统一框架

- **关联目录**：`memory/`
- **为什么重要**：当前 memory 分类体系并不稳定，`short-term / long-term` 不能覆盖所有当代 agent memory 设计。若不单独梳理，后续 `memory/` 子目录容易被误读为“稳定 taxonomy”。
- **现状**：已进入 `overview.md` 和 `temp/conflict.md`，但尚无专门综述。
- **建议产物**：`memory/overview.md` 或 `memory-taxonomy-conflicts.md`

### 2.3 Tool-Centric Design vs Monolithic Agent

- **关联目录**：`tool-use/`、`patterns/`
- **为什么重要**：这是 single-agent 组织能力的根本 trade-off，影响工具调用、上下文组织、状态管理与可组合性。
- **现状**：已有高层提醒，但缺少横向比较。
- **建议产物**：`tool-centric-vs-monolithic.md`

### 2.4 Reflection 的收益边界

- **关联目录**：`self-reflection/`
- **为什么重要**：reflection 已被广泛采用，但收益/成本边界仍不稳定。当前主干需要有地方专门记录“何时值得反思、何时只是增加 token cost”。
- **现状**：冲突已识别，证据分散。
- **建议产物**：`reflection-cost-tradeoff.md`

---

## 三、P1：值得持续跟踪的专题

### 3.1 Input Reformulation for Tool Use

- **关联目录**：`tool-use/`
- **为什么值得关注**：提示单智能体 tool usage 的瓶颈未必在 model 内部，也可能在输入组织和上下文准备。
- **当前状态**：前沿线索，证据较新，需二次核验。
- **不宜直接定论的原因**：目前主要来自较新的单篇研究。

### 3.2 Bilevel Planning for Tool Navigation

- **关联目录**：`planning/`、`tool-use/`
- **为什么值得关注**：为大规模工具生态中的 planning/execution 解耦提供新设计点。
- **当前状态**：值得追踪，但尚不足以写成主流范式。

### 3.3 Long Context vs External Memory

- **关联目录**：`memory/`
- **为什么值得关注**：长上下文窗口是否削弱独立 memory system 的必要性，是当前非常重要但未收敛的问题。
- **当前状态**：适合作为长期冲突专题。

### 3.4 Tool Selection vs Tool Orchestration

- **关联目录**：`tool-use/`
- **为什么值得关注**：当前很多文档只讨论“选哪个工具”，却忽略多工具调用序列如何被组织与约束。
- **当前状态**：主题尚未被显式建模。

---

## 四、P2：保留观察线索

### 4.1 新型 single-agent workflow pattern

- **关联目录**：`patterns/`
- **说明**：如 `RA.Aid`、AutoGPT 衍生模式等，适合持续观察其是否只是产品包装差异，还是确实代表新的能力组合方式。

### 4.2 Agent 定义边界的持续变化

- **关联目录**：`overview.md`、`conflict.md`
- **说明**：市场与学术界对“什么才算 agent”的定义仍在变化，适合持续记录而不是一次性写死。

---

## 五、与现有材料的关系

- `overview.md` 已提供单智能体的高层组织框架。
- `temp/web-search/2.md` 提供了本 backlog 的重要输入线索。
- `temp/conflict.md` 用于存放 taxonomy、definition、trade-off 等尚未收敛的问题。
- 后续若某条线索经过充分核验，应拆分进入对应子目录专题，而不是长期停留在 backlog。
