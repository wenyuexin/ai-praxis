# Multi-Agent Overview

> 适用范围：多智能体协作层的整体认知框架
> 阶段状态：研究综述，持续补充
> 使用说明：本文件回答“多智能体系统为何存在、主要问题是什么、当前有哪些核心矛盾与失败模式”，不直接提供某个编排框架的实现方案

## 一、定位

`03-multi-agent/` 关注的不是“把一个 agent 复制成多个”，而是当多个决策单元共同参与任务时，系统会出现哪些新的收益、成本与失败模式。

多智能体系统的价值通常来自：

- 任务拆分与并行化
- 角色专业化
- 视角互补
- 局部错误隔离

但它的代价也同样明确：

- 协调开销
- 状态同步成本
- 错误放大
- 角色漂移与重复劳动
- 隐性 token / latency / orchestration 成本

因此，本目录的核心问题不是“multi-agent 是否更先进”，而是：

- 何时 multi-agent 真正有收益？
- 收益来自哪一种 coordination topology？
- 成本来自什么结构性来源？
- 哪些失败是 multi-agent 特有的，而不是 single-agent 的简单放大版？

---

## 二、核心问题轴

### 2.1 Collaboration

协作关注任务如何被拆成多个角色或子问题，以及交接与互补如何发生。

关键点不是“是否分工”，而是：

- 分工粒度是否合理
- 交接成本是否过高
- 专业化是否带来真实收益
- 是否只是把 single-agent 的问题复制了多份

### 2.2 Coordination

协调负责管理执行顺序、依赖关系、冲突消解和全局一致性。

这是 multi-agent 与 single-agent 最本质的差别之一，因为很多代价正是在这里产生。

### 2.3 Communication Protocols

通信协议定义 agents 如何交换消息、同步状态、表达请求和报告结果。

当前尤其需要避免把不同层次的协议混写：

- agent↔tool 的协议
- agent↔agent 的协议
- 结构化协议
- 更松散、分布式的协调 substrate

### 2.4 Shared Memory

共享记忆和共享状态负责让多个 agent 在同一任务上下文中获得必要的一致性，但也会带来污染、冲突、冗余和过时信息传播。

### 2.5 Organizational

组织结构关注角色制度、层级关系、中央化与去中心化的平衡，以及系统整体治理方式。

### 2.6 Competition

竞争与博弈提醒我们：多智能体不一定总是合作。资源争用、策略对抗、偏差扩散和失控风险同样属于 multi-agent 研究的一部分。

---

## 三、当前最重要的结构性矛盾

### 3.1 协调开销 vs 并行收益

这是 multi-agent 最根本的 trade-off。

一方面，多个 agent 可以并行处理子任务、分担复杂工作；另一方面，消息同步、角色协调、状态一致性维护会快速吞噬这些收益。

因此，multi-agent 并不天然优于 single-agent，而是高度依赖任务类型、分工结构和协调模式。

### 3.2 Role Specialization vs Communication Redundancy

角色专业化可以减少单个 agent 的上下文负担和工具复杂度，但也会引入更高的消息传播与上下文重复成本。

### 3.3 Structure vs Emergence

一端是结构化、可控、可审计的协调协议；另一端是更灵活、更分布式、可扩展但一致性更弱的 emergent coordination。

这是一个仍在演化中的研究前沿，不宜过早定为单一路线。

### 3.4 Agent 数量 vs 边际收益

增加更多 agent 不一定带来持续改善。随着基线能力增强和任务耦合度上升，边际收益可能迅速下降，甚至转为负收益。

---

## 四、最值得强调的认知提醒

### 4.1 Multi-agent 不是 single-agent 的线性扩展

多出几个 agent，系统问题并不会只是“多几次调用”，而会进入一个新的设计空间：拓扑、组织、共享状态、故障传播、成本建模都发生变化。

### 4.2 Coordination Topology 是主分类轴

比起简单区分“single vs multi”，更值得在知识组织上明确：

- centralized
- decentralized
- hybrid
- independent / loose aggregation

这些拓扑决定了 multi-agent 的能力边界和失败模式。

### 4.3 Failure Modes 应成为主线，不是附录

如果只记录成功案例，`03-multi-agent/` 很容易失真。更合理的组织方式是把 failure modes 视为理解 multi-agent 的核心入口之一。

---

## 五、当前最容易混淆的问题

以下内容在当前阶段不应轻易写成定论：

- “multi-agent 总比 single-agent 强”
- MCP 就是多智能体通信协议
- decentralized 只有一种定义
- multi-agent 成本近似线性叠加
- independent agents 是最简单也最稳妥的起点

这些问题都更适合写入 `conflict.md` 或后续专题。

---

## 六、建议的目录理解方式

可以把 `03-multi-agent/` 理解为围绕“多个决策单元如何共同完成任务”展开：

```text
任务拆分
  → collaboration
  → coordination
  → communication protocols
  → shared memory
  → organizational structure
  → failure / competition / robustness
```

其中：

- `collaboration/` 负责分工与互补
- `coordination/` 负责依赖与全局一致性
- `communication-protocols/` 负责交互结构
- `shared-state-and-context/` 负责共享上下文与状态传播
- `organizational/` 负责层级与治理
- `competition-and-conflict/` 负责对抗、博弈与稳健性视角

---

## 七、与其他目录的关系

- 与 `../02-single-agent/`：单体能力决定多智能体分工的上限
- 与 `../05-environments/`：执行环境和通信基础设施决定协调成本与隔离边界
- 与 `../06-frameworks-and-tools/`：具体框架只是多智能体机制的实现载体，不应替代机制分类
- 与 `../07-evaluation/`：多智能体必须在质量、成本、延迟、稳定性四个维度同时评估

---

## 八、当前最值得补齐的专题

- `coordination/topology.md`：协调拓扑比较
- failure modes catalog
- specialization 粒度与 hidden cost
- structured protocols vs emergent coordination
- multi-agent 何时优于 single-agent 的任务判断框架

这些内容应优先进入专题文档、`backlog.md` 或 `conflict.md`，再逐步反哺主干。