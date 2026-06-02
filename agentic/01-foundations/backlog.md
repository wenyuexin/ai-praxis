# Foundations Backlog

> 适用范围：`01-foundations/` 的内容缺口、前沿问题与待补基础专题
> 使用原则：本文件记录已识别但尚未充分覆盖的概念底座问题，不是任务管理列表，也不代表相关结论已进入主干定论

## 一、收录规则

适合进入本文件的内容通常满足以下条件之一：

- 能补齐 `definition-and-taxonomy / reactive-vs-deliberative / cognitive-architectures / agent-system-models` 之间的关键桥梁问题
- 会影响 `02-single-agent/`、`03-multi-agent/`、`05-environments/` 等后续目录的上位定义与边界判断
- 已被论文、官方文档或高质量综述明确指出具有基础方法价值
- 尚不适合直接写入主干正文，但值得继续跟踪

不适合进入本文件的内容：

- 具体框架、工具、产品或项目对象（应放 `../06-frameworks-and-tools/`）
- 已形成稳定主干知识且没有明显争议的内容
- 单篇论文资料汇总但尚未转化为 foundations 层问题意识的内容

---

## 二、P0：当前高优先级缺口

### 2.1 Agentic Workflow vs Agent Boundary

- **关联目录**：`definition-and-taxonomy/`、`reactive-vs-deliberative/`
- **为什么重要**：会直接影响后续 `single-agent`、`multi-agent` 与工程案例中“什么算 agent”的判断边界。
- **现状**：`definition-and-taxonomy/` 下已有局部材料，但仍缺少 foundations 视角下对 workflow、agentic workflow、agent 的统一定义框架。

### 2.2 Agent Harness as System Model

- **关联目录**：`agent-system-models/`
- **为什么重要**：`harness` 正在从工程术语演化为理解 agent runtime substrate、observability、execution control 的上位模型。
- **现状**：已有 `agent-system-models/research-queue.md` 收集 harness 方向的待研究对象，但尚未沉淀为 foundations 层正式专题。

### 2.3 Cognitive Architecture vs Orchestration Architecture

- **关联目录**：`cognitive-architectures/`、`agent-system-models/`
- **为什么重要**：认知架构、执行编排与工程 runtime 常被混写，不澄清会影响后续目录的上位分类。
- **现状**：两条线已有零散材料，但还缺少面向整个 `01-foundations/` 的边界说明。

---

## 三、P1：值得持续跟踪的专题

### 3.1 Reactive / Deliberative / Hybrid Continuum

- **关联目录**：`reactive-vs-deliberative/`
- **为什么值得关注**：很多系统并不落在纯反应式或纯审慎式两端，而是混合连续谱；这一点会影响对 planning、reflection 与 autonomy 的理解。
- **当前状态**：已有目录占位，但 foundations 层的连续谱表达仍不充分。

### 3.2 Agent System Layers and Control Surfaces

- **关联目录**：`agent-system-models/`、`05-environments/`
- **为什么值得关注**：model、memory、tools、workspace、permissions、human approval 等控制面之间的层次关系，决定了 agent 系统建模方式。
- **当前状态**：相关线索已分散出现在 `02-single-agent/` 与 `05-environments/`，但 foundations 层仍缺统一抽象。

### 3.3 Taxonomy Stability vs Market Terminology Drift

- **关联目录**：`definition-and-taxonomy/`
- **为什么值得关注**：产业界与研究界对 agent、copilot、assistant、workflow、runtime 等术语的口径持续漂移。
- **当前状态**：已有分类讨论，但缺少持续记录术语漂移的 foundations 机制。

---

## 四、P2：保留观察线索

### 4.1 具身 / 环境耦合对 agent 定义的反向塑形

- **关联目录**：`../05-environments/`
- **说明**：随着 browser / sandbox / simulated world 成为 agent 系统核心组成，环境是否应进入 foundations 层定义值得继续观察。

### 4.2 Agent runtime substrate 的统一抽象

- **关联目录**：`agent-system-models/`
- **说明**：`runtime substrate`、`harness`、`execution fabric` 等表达可能会逐渐汇聚成新的系统模型主线。

---

## 五、与现有材料的关系

- `agent-system-models/research-queue.md` 当前保留 harness 方向的待研究对象输入，但不替代 foundations backlog 本身。
- `definition-and-taxonomy/orchestration-paradigms.md` 已提供部分流程范式材料，后续可继续反哺 foundations 层边界整理。
- 后续若某条线索证据充分，应下沉为对应子目录的正式专题，而不是长期停留在 backlog。
