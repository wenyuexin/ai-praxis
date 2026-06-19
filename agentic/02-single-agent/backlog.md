# Single-Agent Backlog

> 适用范围：`02-single-agent/` 的内容缺口、前沿问题与待补专题
> 使用原则：本文件记录已识别但尚未充分覆盖的知识空白，不是任务管理列表，也不代表相关结论已进入主干定论

## 一、收录规则

适合进入本文件的内容通常满足以下条件之一：

- 已被论文、官方文档、源码或高质量调研明确指出有研究价值
- 能补齐 `planning / memory / tool-use / reflection / architectural-patterns` 之间的关键桥梁问题
- 代表 single-agent 研究中的核心矛盾、常见误区或新兴分岔
- 尚不适合直接写入主干正文，但值得继续跟踪

不适合进入本文件的内容：

- 具体框架实现细节（应放 `../06-frameworks-and-tools/`）
- 已形成稳定主干知识且没有明显争议的内容
- 只靠单篇博客、单个 demo 或热度数据支撑的判断

---

## 二、P0：当前高优先级缺口

### 2.1 Tool Invocation Reliability

- **关联目录**：`tool-use/`
- **为什么重要**：工具能力本身并不等于调用可靠性，参数构造、失败恢复、结果校验与约束遵守往往才是单智能体系统真正的不稳定来源。
- **现状**：`tool-invocation-reliability.md` 已形成独立主干专题，稳定写下 schema、前置条件、结果校验与失败恢复是可靠性的核心环节；后续更多是补不同 agent 类型下的 failure mode 分布，而不是再证明这条主轴是否存在。
- **建议产物**：后续更适合补 invocation reliability checklist、failure taxonomy 对照与跨场景案例。

### 2.2 Reflection Trigger Design

- **关联目录**：`self-reflection/`
- **为什么重要**：即便已经明确 reflection 存在收益/成本 trade-off，系统仍需要决定触发信号来自任务复杂度、风险等级，还是外部失败反馈。
- **现状**：`reflection-trigger-design.md` 已形成独立主干专题，能够稳定表达 trigger、stopping rule 与 recovery routing 的关系；后续仍需补证不同任务类型下哪些触发信号最常带来净收益。
- **建议产物**：后续更适合补 trigger checklist、risk-aware policy 与 task-specific trigger pattern。

### 2.3 Bilevel Planning for Tool Navigation

- **关联目录**：`planning/`、`tool-use/`
- **为什么重要**：为大规模工具生态中的 planning/execution 解耦提供新设计点，也能把 planner 与 orchestration 的中间层表达得更清楚。
- **现状**：`bilevel-planning-tool-navigation.md` 已形成独立主干专题，能够稳定表达高层任务结构与低层工具导航的分层价值；后续重点转向哪些失败应在低层消化、哪些应升级到高层 replanning。
- **建议产物**：后续更适合补分层接口、失败升级边界与跨 agent 类型案例，而不是重复补定义。

---

## 三、P1：值得持续跟踪的专题

### 3.1 Input Reformulation for Tool Use

- **关联目录**：`tool-use/`
- **为什么值得关注**：提示单智能体 tool usage 的瓶颈未必在 model 内部，也可能在输入组织和上下文准备。
- **当前状态**：已有前沿线索，但证据仍偏新，需要继续核验其是否能形成稳定主题。
- **不宜直接定论的原因**：目前主要来自较新的单篇研究，尚需更多系统级复现或横向比较。

### 3.2 Memory Writeback and Contamination Control

- **关联目录**：`memory/`
- **为什么值得关注**：即便已经明确 long context 与 external memory 的边界，系统仍需要决定什么信息该写回、如何修订、如何防止污染与过时状态扩散。
- **当前状态**：`memory/README.md` 中已有提醒，`long-context-vs-external-memory.md` 也已触及，但还没有围绕 writeback governance 的独立整理。

### 3.3 State Persistence vs Workspace State

- **关联目录**：`memory/`、`../05-environments/`
- **为什么值得关注**：single-agent 的 memory、artifact、workspace state 与 environment traceability 边界容易混写，尤其在 coding / browser agent 场景中更明显。
- **当前状态**：`long-context-vs-external-memory.md` 与 `05-environments/` 已分别提供局部视角，但跨目录边界仍值得持续梳理。

### 3.4 Agent Definition Boundary Drift

- **关联目录**：`overview.md`、`conflict.md`
- **为什么值得关注**：即使已经补出 `agent-vs-tool-workflow-boundary.md`，市场与研究对 agent 的定义口径仍在变化，后续仍需持续记录新的分层方式与边界漂移。
- **当前状态**：已有基础主干，但不适合过早写成永久稳定结论。

---

## 四、P2：保留观察线索

### 4.1 新型 single-agent workflow pattern

- **关联目录**：`architectural-patterns/`
- **说明**：如 `RA.Aid`、AutoGPT 衍生模式等，适合持续观察其是否只是产品包装差异，还是确实代表新的能力组合方式。

### 4.2 新型 long-context / memory interaction pattern

- **关联目录**：`memory/`、`tool-use/`
- **说明**：随着更长上下文模型与新型 memory runtime 出现，值得持续观察它们到底是在替代 external memory，还是只是在重组 retrieval 与 state persistence 方式。

---

## 五、与现有材料的关系

- `overview.md` 已提供单智能体的高层组织框架，其中早期最关键的几条主轴已拆成独立专题。
- `conflict.md` 继续承担术语、结论与边界冲突的记录职责；本 backlog 现在更多保留“已识别但尚未充分展开”的下一批主题。
- 早期外部调研仍为部分前沿线索提供了输入，但这些线索不应替代后续专题化整理；若继续回流，应优先改写为对应论文、官方文档或案例对象来源。
- 后续若某条线索经过充分核验，应拆分进入对应子目录专题，而不是长期停留在 backlog。
