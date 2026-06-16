# Interdisciplinarity Index

> 如果你是第一次进入本目录，先读 [`README.md`](./README.md) 建立方向感；如果你已经知道想找什么，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `roadmap.md` 的路径指引。

## 目录结构

```
interdisciplinarity/
│
├── 01-cognitive-neuroscience/              # 认知科学与神经科学
│   ├── neural-networks-as-brain-models/     # 神经网络作为脑模型
│   ├── attention-and-consciousness/          # 注意力机制与意识
│   ├── memory-systems/                       # 记忆系统
│   └── learning-theories/                    # 学习理论
│
├── 02-linguistics-and-pragmatics/           # 语言学与语用学
│
├── 03-philosophy-of-mind/                   # 心灵哲学
│
├── 04-cybernetics-systems/                  # 控制论与系统论
│
├── 05-complex-emergence/                    # 复杂系统与涌现
│
├── 06-sociology-and-organization/           # 社会学与组织管理
│   ├── ambidexterity-and-paradox/           # 双元性与悖论
│   ├── social-network-analysis/             # 社会网络分析
│   ├── institutional-theory/                # 制度理论
│   ├── organizational-forms/                # 组织形式理论
│   ├── coordination-theory/                 # 协调理论
│   ├── social-capital/                      # 社会资本理论
│   └── organizational-ecology/              # 组织生态学
│
├── 07-economics-game-theory/                # 经济学与博弈论
│
└── 08-law-and-governance/                   # 法学与治理
```

## 按子目录定位

- `01-cognitive-neuroscience/`：人脑认知机制对深度网络架构、训练范式和智能体设计的启发
- `02-linguistics-and-pragmatics/`：语言学理论对理解大语言模型能力与局限的启发
- `03-philosophy-of-mind/`：意向性、意识、具身认知等哲学论证对 AI 能力边界的反思
- `04-cybernetics-systems/`：反馈、调节、稳态视角下的 Agent 感知-行动闭环与具身控制
- `05-complex-emergence/`：涌现、相变、自组织临界性对 LLM 涌现能力和 Agent 集体行为的启发
- `06-sociology-and-organization/`：人类组织形态对多智能体系统协作、治理与涌现的审视
- `07-economics-game-theory/`：机制设计、激励兼容、博弈论对 Agent 对齐与激励设计的框架支撑
- `08-law-and-governance/`：AI 安全对齐、责任归属与治理框架的法律与制度基础

## 按问题找入口

### 01 认知科学与神经科学

| 外部学科概念 | AI/ML 对应 | 关联目录 |
|---|---|---|
| 预测编码 | 自监督学习、下一 token 预测 | [`llm/03-training/pre-training/`](../llm/03-training/pre-training/) |
| 工作记忆 / 长期记忆 | Context Window / RAG | [`rag/`](../rag/) |
| 注意力机制 | Transformer Self-Attention | [`deep-learning/03-architectures/transformers/`](../deep-learning/03-architectures/transformers/) |
| 赫布学习 | 局部学习规则、脉冲网络 | [`deep-learning/01-neural-network-fundamentals/`](../deep-learning/01-neural-network-fundamentals/) |
| 认知架构（SOAR, ACT-R） | Agent 认知架构 | [`agentic/01-foundations/`](../agentic/01-foundations/) |

### 02 语言学与语用学

| 外部学科概念 | AI/ML 对应 | 关联目录 |
|---|---|---|
| 语用学 / 言语行为理论 | 对话 Agent 的意图理解与生成 | [`agentic/02-single-agent/`](../agentic/02-single-agent/) |
| 乔姆斯基层级 / 普遍语法 | LLM 语法能力的理论边界 | [`llm/02-models/`](../llm/02-models/) |
| 话语分析 | 长文本连贯性、多轮对话管理 | [`llm/04-serving/`](../llm/04-serving/) |
| 语义学 / 组合性 | LLM 的语义组合能力与幻觉 | [`llm/06-explainability/`](../llm/06-explainability/) |
| 社会语言学 | 多语言/多文化 LLM 对齐 | [`llm/08-safety-and-society/safety-and-alignment/`](../llm/08-safety-and-society/safety-and-alignment/) |

### 04 控制论与系统论

| 外部学科概念 | AI/ML 对应 | 关联目录 |
|---|---|---|
| 负反馈 / 正反馈 | RL 奖励调节、策略梯度 | [`reinforce-learning/`](../reinforce-learning/) |
| 自调节系统 | Agent 自反思与纠错 | [`agentic/02-single-agent/`](../agentic/02-single-agent/) |
| 稳态 | Agent 目标维持与安全约束 | [`agentic/05-environments/`](../agentic/05-environments/) |
| 黑箱系统 | 可解释性与机制可解释性 | [`llm/06-explainability/`](../llm/06-explainability/) |
| 感知-动作闭环 | 具身智能控制回路 | [`embodied-intelligence/02-perception/`](../embodied-intelligence/02-perception/) |

### 06 社会学与组织管理

| 外部学科概念 | AI/ML 对应 | 关联目录 |
|---|---|---|
| 韦伯式科层制 | 层级式 Multi-Agent | [`agentic/03-multi-agent/organizational/`](../agentic/03-multi-agent/organizational/) |
| 市场机制 / 价格信号 | 竞价式 Agent 协调 | [`agentic/03-multi-agent/coordination/`](../agentic/03-multi-agent/coordination/) |
| 社会网络 / 结构洞 | Agent 通信拓扑优化 | [`06-sociology-and-organization/`](./06-sociology-and-organization/) |
| 制度主义 | Agent 规范与约束设计 | [`06-sociology-and-organization/institutional-theory/`](./06-sociology-and-organization/institutional-theory/) |
| 组织生态学 | Agent 种群竞争与生态位 | [`06-sociology-and-organization/`](./06-sociology-and-organization/) |
| 社会资本 / 信任 | Agent 间信任评估 | [`agentic/07-evaluation/safety-and-robustness/`](../agentic/07-evaluation/safety-and-robustness/) |
| 组织学习 | Multi-Agent 经验共享 | [`agentic/03-multi-agent/`](../agentic/03-multi-agent/) |

### 08 法学与治理

| 外部学科概念 | AI/ML 对应 | 关联目录 |
|---|---|---|
| 责任归属 | Agent 行为的法律责任主体 | [`agentic/07-evaluation/`](../agentic/07-evaluation/) |
| 合规 | 数据合规、模型合规审计 | [`llm/08-safety-and-society/`](../llm/08-safety-and-society/) |
| 权利理论 | AI 权利、人格与道德地位 | [`llm/08-safety-and-society/social-impact/`](../llm/08-safety-and-society/social-impact/) |
| 规则制定 | AI 治理框架与标准制定 | [`agentic/05-environments/`](../agentic/05-environments/) |
| 程序正义 | 算法公平性与可申诉性 | [`llm/05-evaluation/`](../llm/05-evaluation/) |

## 按阅读目标找入口

- 如果你的目标是先建立整体理解框架，这个问题不由本页展开，转读 [`README.md`](./README.md)
- 如果你不确定从哪开始，这个问题不由本页代答，转读 [`roadmap.md`](./roadmap.md)
- 如果你的目标是撰写新笔记，参考 [`template.md`](./template.md)
