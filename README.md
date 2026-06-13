<div align="center">

  <h1>🧠 Machine Learning Study Notes</h1>
  <p>
    <strong>从经典算法到大语言模型：一场结构化的现代机器学习之旅</strong><br>
    <em>ML / DL / NLP / CV / LLM / RAG / KG</em>
  </p>

  <!-- 核心状态栏 -->
  <p>
    <!-- 动态获取最后提交时间 -->
    <img src="https://img.shields.io/github/last-commit/wenyuexin/machine-learning?style=flat-square&label=Last%20Update&color=00bcd4" alt="Last Commit">
    <!-- 静态徽章 -->
    <img src="https://img.shields.io/badge/License-MIT-blue?style=flat-square" alt="License">
    <img src="https://img.shields.io/badge/Status-Active%20Learning-yellow?style=flat-square" alt="Status">
    <img src="https://img.shields.io/badge/Format-Markdown-lightgrey?style=flat-square" alt="Format">
  </p>
</div>

---

## 层级关系

```
基础方法层：
├── traditional-ml/              # 传统机器学习
├── deep-learning/               # 深度学习基础
└── reinforce-learning/          # 强化学习

核心领域层：
├── llm/                         # 大语言模型
├── cv/                          # 计算机视觉
└── knowledge-graph/             # 知识图谱

应用与集成层：
├── rag/                         # 检索增强生成（LLM + 外部知识）
├── agentic/                     # AI智能体（LLM驱动的自主系统）
├── world-models/                # 世界模型（环境建模与预测）
└── embodied-intelligence/       # 具身智能（物理世界落地）

基础设施层：
└── training-infra/              # 训练基础设施

跨领域视角层：
└── interdisciplinarity/           # 跨学科交叉（认知科学·社会学·博弈论·哲学·语言学·控制论·法学）

支撑资源：
├── learning-materials/          # 学习资料与书籍推荐
└── docs/                        # 仓库建设、贡献规则与文档模板
```

## 知识体系

### 纵向技术栈依赖

```
┌────────────────────────────────────────────────────────┐
│ 应用层:  RAG          智能体    世界模型    具身智能     │
│ 核心层:  知识图谱      LLM      CV                      │
│ 基础层:  传统机器学习  深度学习  RL                      │
│ 设施层:  训练基础设施  学习资料                          │
└────────────────────────────────────────────────────────┘
依赖: 
  深度学习,传统机器学习 → CV, LLM, 知识图谱
  RL → 智能体, 世界模型
  LLM → RAG, 智能体
  知识图谱 → RAG
  CV, 世界模型 → 具身智能
```

### 跨学科映射

8个学科视角对技术领域的主要启发关系（详细映射见 [interdisciplinarity/](./interdisciplinarity/)）。

```
┌──────────────────────────────────────────┐
│ 认知科学  语言学  心灵哲学  控制论         │
│ 复杂系统  社会学  博弈论    法学           │
├──────────────────────────────────────────┤
│ 深度学习  大语言模型  AI智能体  强化学习    │
└──────────────────────────────────────────┘
映射: 
  认知科学 → DL, AG    语言学 → LLM    控制论 → RL
  社会学 → AG          博弈论 → AG     法学 → LLM
```

## 目录结构

> 编号目录展开至子目录名；核心领域展开较深，扩展领域仅列三级目录。各领域详细结构见对应 README。

```
machine-learning/
│
├── traditional-ml/                             # 传统机器学习
│   ├── 01-fundamentals/                        # 数学基础
│   │   ├── linear-algebra/                     # 线性代数
│   │   ├── probability-and-statistics/         # 概率论与统计
│   │   └── optimization/                       # 优化理论
│   ├── 02-supervised-learning/                 # 监督学习
│   │   ├── linear-models/                      # 线性模型
│   │   ├── tree-based-models/                  # 树模型
│   │   ├── svm/                                # 支持向量机
│   │   └── knn/                                # K近邻
│   ├── 03-unsupervised-learning/               # 无监督学习
│   │   ├── clustering/                         # 聚类算法
│   │   └── dimensionality-reduction/           # 降维方法
│   ├── 04-practical-ml/                        # 实践方法
│   │   ├── feature-engineering/                # 特征工程
│   │   ├── model-selection-and-tuning/         # 模型选择与调优
│   │   ├── ensemble-methods/                   # 集成方法
│   │   ├── imbalanced-learning/                # 不平衡学习
│   │   └── interpretability/                   # 可解释性
│   ├── 05-time-series/                         # 时间序列
│   │   ├── classical-methods/                  # 经典方法
│   │   ├── machine-learning-approaches/        # 机器学习方法
│   │   └── deep-learning-for-ts/               # 深度学习时序
│   └── 06-probabilistic-graphical-models/      # 概率图模型
│       ├── bayesian-networks/                  # 贝叶斯网络
│       └── markov-random-fields/               # 马尔可夫随机场
│
├── deep-learning/                              # 深度学习基础
│   ├── 01-neural-network-fundamentals/         # 神经网络基础
│   ├── 02-training-and-optimization/           # 训练与优化
│   ├── 03-architectures/                       # 按领域划分的架构
│   │   ├── cnns/                               # 卷积神经网络
│   │   ├── rnns-and-sequence-models/           # 循环神经网络与序列模型
│   │   ├── transformers/                       # Transformer
│   │   └── generative-models/                  # 生成模型
│   ├── 04-advanced-topics/                     # 进阶主题
│   └── 05-infra/                               # 深度学习基础设施
│
├── reinforce-learning/                         # 强化学习
│   ├── 01-foundations/                         # 基础概念
│   ├── 02-model-free-rl/                       # 无模型强化学习
│   ├── 03-policy-methods/                      # 基于策略与Actor-Critic
│   ├── 04-model-based-rl/                      # 基于模型的强化学习
│   ├── 05-advanced-topics/                     # 进阶与应用
│   └── 06-evaluation-and-tools/                # 评估与工具
│
├── cv/                                         # 计算机视觉
│   ├── 01-image-fundamentals/                  # 图像基础
│   ├── 02-image-classification/                # 图像分类
│   ├── 03-detection-and-segmentation/          # 目标检测与分割
│   ├── 04-video-and-3d-vision/                 # 视频与3D视觉
│   ├── 05-generative-and-multimodal/           # 生成与多模态
│   ├── 06-foundation-models/                   # 自监督与基础模型
│   └── 07-applications-and-tools/              # 应用与工具
│
├── llm/                                        # 大语言模型
│   ├── 01-foundations/                         # 基础
│   │   ├── transformer-architecture/           # Transformer架构
│   │   ├── tokenization/                       # 分词
│   │   └── scaling-laws/                       # 缩放定律
│   ├── 02-models/                              # 模型
│   │   ├── open-source-models/                 # 开源模型
│   │   ├── architectural-variants/             # 架构变体
│   │   └── emergent-abilities/                 # 涌现能力
│   ├── 03-training/                            # 训练
│   │   ├── pre-training/                       # 预训练
│   │   ├── fine-tuning/                        # 微调
│   │   └── alignment/                          # 对齐
│   ├── 04-serving/                             # 推理与部署
│   │   ├── optimization-techniques/            # 优化技术
│   │   ├── serving-frameworks/                 # Serving框架
│   │   └── prompt-engineering/                 # 提示工程
│   ├── 05-evaluation/                          # 评估
│   │   ├── benchmarks/                         # 基准测试
│   │   ├── evaluation-methods/                 # 评估方法
│   │   └── evaluation-frameworks/              # 评估框架
│   ├── 06-explainability/                      # 可解释性
│   │   ├── mechanistic/                        # 机制可解释性
│   │   ├── attribution/                        # 归因方法
│   │   ├── probing/                            # 探测技术
│   │   └── counterfactual/                     # 反事实解释
│   ├── 07-multimodal/                          # 多模态
│   │   ├── vlm/                                # 视觉语言模型
│   │   ├── audio/                              # 音频语言模型
│   │   ├── video/                              # 视频语言模型
│   │   └── any2any/                            # 全模态模型
│   └── 08-safety-and-society/                  # 安全与社会影响
│       ├── safety-and-alignment/               # 安全与对齐
│       └── social-impact/                      # 社会影响
│
├── rag/                                        # 检索增强生成
│   ├── 01-foundations/                         # 基础
│   │   ├── what-is-rag/                        # RAG概念与动机
│   │   ├── naive-rag/                          # 朴素RAG
│   │   ├── rag-pipeline-overview/              # 管线总览
│   │   ├── context-integration/                # 上下文构建与注入
│   │   ├── generation-strategies/              # 生成策略
│   │   └── evaluation-of-generation/           # 生成质量评估
│   ├── 02-retrieval/                           # 检索
│   │   ├── chunking-strategies/                # 分块策略
│   │   ├── embedding-models/                   # 嵌入模型
│   │   ├── vector-databases/                   # 向量数据库
│   │   └── advanced-retrieval/                 # 高级检索
│   ├── 03-advanced-patterns/                   # 高级范式（含代表实现）
│   │   ├── graph-rag/                          # 图增强RAG
│   │   ├── llm-wiki/                           # LLM Wiki / 知识编译范式
│   │   ├── self-reflective-rag/                # 自反思RAG
│   │   ├── agentic-rag/                        # Agent驱动RAG
│   │   ├── modular-rag/                        # 模块化RAG
│   │   └── multimodal-rag/                     # 多模态RAG
│   ├── 04-evaluation/                          # 评估
│   └── 05-production/                          # 生产与生态
│
├── agentic/                                    # AI智能体
│   ├── 01-foundations/                         # 基础概念
│   │   ├── definition-and-taxonomy/            # 定义与分类
│   │   ├── reactive-vs-deliberative/           # 反应式vs审慎式
│   │   ├── cognitive-architectures/            # 认知架构总论
│   │   └── agent-system-models/                # Agent系统模型
│   ├── 02-single-agent/                        # 单智能体
│   │   ├── planning/                           # 规划
│   │   ├── memory/                             # 记忆
│   │   ├── tool-use/                           # 工具使用
│   │   ├── reasoning-and-acting/               # 推理与行动
│   │   ├── self-reflection/                    # 自我反思
│   │   └── patterns/                           # 架构模式（ReAct、AutoGPT等）
│   ├── 03-multi-agent/                         # 多智能体
│   │   ├── collaboration/                      # 协作
│   │   ├── competition/                        # 竞争
│   │   ├── organizational/                     # 组织架构
│   │   ├── shared-memory/                      # 共享记忆
│   │   ├── coordination/                       # 协调机制
│   │   └── communication-protocols/            # 通信协议
│   ├── 04-human-agent-interaction/             # 人机交互
│   │   ├── human-in-the-loop/                  # 人在回路
│   │   ├── agent-ui/                           # Agent界面
│   │   ├── delegation-and-control/             # 委托与控制
│   │   └── trust-and-alignment/                # 信任与对齐
│   ├── 05-environments/                        # 环境与仿真
│   │   ├── simulated-environments/             # 仿真环境
│   │   ├── browser-environments/               # 浏览器环境
│   │   ├── code-execution-environments/        # 代码执行环境
│   │   ├── sandboxing-and-safety/              # 沙箱与安全
│   │   └── benchmarking-frameworks/            # 基准环境
│   ├── 06-frameworks-and-tools/                # 框架与工具
│   │   ├── 01-frameworks/                      # 通用框架与SDK
│   │   ├── 02-coding-tools/                    # 编码Agent工具
│   │   ├── 03-project-studies/                 # 项目案例研究
│   │   ├── 04-skill-and-tool-systems/          # 技能与工具系统
│   │   └── 05-comparisons/                     # 对比与选型
│   └── 07-evaluation/                          # 评估与可靠性
│       ├── task-completion-metrics/            # 任务完成度
│       ├── agent-benchmarks/                   # 通用Agent基准
│       ├── swe-benchmarks/                     # 软件工程基准
│       ├── safety-and-robustness/              # 安全与鲁棒性
│       ├── human-evaluation/                   # 人工评估
│       └── observability-and-debugging/        # 可观测性与调试
│
├── knowledge-graph/                            # 知识图谱
│   ├── 01-foundations/                         # 基础
│   │   ├── symbolic-representation/            # 符号表示
│   │   ├── embedding-based-representation/     # 嵌入表示
│   │   └── ontology-and-schema/                # 本体与Schema
│   ├── 02-construction/                        # 知识构建
│   ├── 03-storage-and-query/                   # 存储与查询
│   ├── 04-reasoning/                           # 知识推理
│   ├── 05-applications/                        # 应用
│   └── 06-quality/                             # 质量与演化
│
├── embodied-intelligence/                      # 具身智能
│   ├── 01-foundations/                         # 基础
│   ├── 02-perception/                          # 感知闭环
│   ├── 03-motor-control-and-policies/          # 运动控制与策略
│   ├── 04-planning-and-navigation/             # 规划与导航
│   ├── 05-manipulation/                        # 操作与交互
│   ├── 06-foundation-models/                   # 大模型驱动具身
│   └── 07-evaluation-and-benchmarks/           # 评估与基准
│
├── world-models/                               # 世界模型
│   ├── 01-foundations/                         # 基础
│   ├── 02-methods-and-architectures/           # 方法与架构
│   ├── 03-reasoning-models/                    # 推理用世界模型
│   ├── 04-large-scale-world-models/            # 大规模世界模型
│   ├── 05-evaluation/                          # 评估
│   └── 06-applications/                        # 应用
│
├── training-infra/                             # 训练基础设施
│   ├── 01-hardware-and-networking/             # 硬件与网络
│   ├── 02-distributed-training/                # 分布式训练
│   ├── 03-frameworks-and-tools/                # 训练框架与工具
│   ├── 04-memory-and-storage/                  # 内存与存储优化
│   ├── 05-compilation-and-kernels/             # 模型编译与内核
│   ├── 06-ml-operations/                       # 实验追踪与运维
│   └── 07-observability-and-debugging/         # 可观测性与调试
│
├── interdisciplinarity/                        # 跨学科交叉
│   ├── 01-cognitive-neuroscience/              # 认知科学与神经科学
│   ├── 02-linguistics-and-pragmatics/          # 语言学与语用学
│   ├── 03-philosophy-of-mind/                  # 心灵哲学
│   ├── 04-cybernetics-systems/                 # 控制论与系统论
│   ├── 05-complex-emergence/                   # 复杂系统与涌现
│   ├── 06-sociology-and-organization/          # 社会学与组织管理
│   ├── 07-economics-game-theory/               # 经济学与博弈论
│   └── 08-law-and-governance/                  # 法学与治理
│
├── learning-materials/                         # 学习资料
│   ├── interview/                              # 面试资料
│   ├── backlog.md                              # 技术项待办
│   ├── books.md                                # 书籍推荐
│   ├── courses.md                              # 课程与讲座
│   ├── papers.md                               # 论文与综述
│   ├── blogs.md                                # 博客与技术指南
│   ├── tools.md                                # 工具与框架
│   ├── datasets.md                             # 数据集
│   └── community.md                            # 社区与活动
│
└── docs/                                       # 仓库建设、贡献规则与文档模板
    ├── contributing/                           # 贡献、文档治理、证据与可追溯性规则
    ├── templates/                              # 文档模板
    └── test/                                   # Markdown 与 Mermaid 渲染测试
```

## 声明

- 📢 如需转载，请注明出处

- 🏗️ 个人的学习仓库，目录建得很全，但是不确定什么时候填坑
