# Agentic AI 论文集

**研究主题**: 基于大语言模型的自主智能体（Agentic AI）架构、能力与协作

**创建日期**: 2026-04-22

---

## 目录结构

```text
papers/
└── images/
```

## 论文列表

### 2023-2024：基础架构与方法

| 论文 | 文档 | arXiv | 会议 | 核心贡献 | 重要性 |
|------|------|-------|------|----------|--------|
| ReAct | [ReAct_2210.03629.md](./ReAct_2210.03629.md) | 2210.03629 | NeurIPS 2023 | 推理与行动交错，LLM Agent基础范式 | ⭐⭐⭐ 必读基础 |
| Toolformer | [Toolformer_2302.04761.md](./Toolformer_2302.04761.md) | 2302.04761 | arXiv 2023 | 教LLM自主使用工具的开创性工作 | ⭐⭐⭐ 必读基础 |
| AutoGPT | [AutoGPT_2023.md](./AutoGPT_2023.md) | - | 开源项目 | 完全自主Agent范式，自动设定子目标 | ⭐⭐⭐ 必读基础 |
| Reflexion | [Reflexion_2303.11366.md](./Reflexion_2303.11366.md) | 2303.11366 | NeurIPS 2023 | 语言自我反思改进Agent行为 | ⭐⭐⭐ 必读基础 |

### 2024-2025：多智能体协作与规划

| 论文 | 文档 | arXiv | 会议 | 核心贡献 | 重要性 |
|------|------|-------|------|----------|--------|
| Agentic LLM Survey | [AgenticLLM_Survey_2503.23037.md](./AgenticLLM_Survey_2503.23037.md) | 2503.23037 | arXiv 2025 | Agentic LLM全面综述：推理、行动、交互 | ⭐⭐⭐ 必读基础 |
| Agentic AI Architectures | [AgenticAI_Architectures_2601.12560.md](./AgenticAI_Architectures_2601.12560.md) | 2601.12560 | arXiv 2026 | 系统架构综述：感知、记忆、大脑、行动 | ⭐⭐⭐ 必读基础 |
| AgentsNet | [AgentsNet_2507.08616.md](./AgentsNet_2507.08616.md) | 2507.08616 | arXiv 2025 | 多智能体协调与协作推理 | ⭐⭐ 值得关注 |
| Player-Coach Teamwork | [PlayerCoach_NeurIPS2025.md](./PlayerCoach_NeurIPS2025.md) | - | NeurIPS 2025 | 多智能体协作提升LLM推理能力 | ⭐⭐ 值得关注 |

### 2025-2026：适应性与技能学习

| 论文 | 文档 | arXiv | 会议 | 核心贡献 | 重要性 |
|------|------|-------|------|----------|--------|
| Adaptation of Agentic AI | [Adaptation_Survey_2512.16301.md](./Adaptation_Survey_2512.16301.md) | 2512.16301 | arXiv 2025 | Agent适应性综述：后训练、记忆、技能 | ⭐⭐⭐ 必读基础 |
| SoK: Agentic Skills | [AgenticSkills_SoK_2602.20867.md](./AgenticSkills_SoK_2602.20867.md) | 2602.20867 | arXiv 2026 | 超越工具使用：Agent技能系统综述 | ⭐⭐⭐ 必读基础 |
| Agentic RL Survey | [AgenticRL_Survey_2509.02547.md](./AgenticRL_Survey_2509.02547.md) | 2509.02547 | arXiv 2025 | Agentic强化学习全景综述 | ⭐⭐⭐ 必读基础 |

### 2025-2026：安全与优化

| 论文 | 文档 | arXiv | 会议 | 核心贡献 | 重要性 |
|------|------|-------|------|----------|--------|
| Attack and Defense Landscape | [AttackDefense_2603.11088.md](./AttackDefense_2603.11088.md) | 2603.11088 | arXiv 2026 | Agentic AI攻击与防御全景分析 | ⭐⭐⭐ 必读基础 |
| Understanding and Optimizing | [UnderstandingOptimizing_2511.00739.md](./UnderstandingOptimizing_2511.00739.md) | 2511.00739 | arXiv 2025 | Agentic AI系统优化：CPU-GPU协同 | ⭐⭐ 值得关注 |
| Evolution of Software Architecture | [Evolution_Architecture_2602.10479.md](./Evolution_Architecture_2602.10479.md) | 2602.10479 | arXiv 2026 | Agentic AI软件架构演进 | ⭐⭐ 值得关注 |

### 2026：工具使用与开放世界

| 论文 | 文档 | arXiv | 会议 | 核心贡献 | 重要性 |
|------|------|-------|------|----------|--------|
| ToolOmni | [ToolOmni_2604.13787.md](./ToolOmni_2604.13787.md) | 2604.13787 | arXiv 2026 | 开放世界工具使用：主动检索+接地执行 | ⭐⭐ 值得关注 |
| AI Agent Index 2025 | [AIAgent_Index_2602.17753.md](./AIAgent_Index_2602.17753.md) | 2602.17753 | arXiv 2026 | 2025 AI Agent技术与经济影响指数 | ⭐⭐ 值得关注 |

---

## 方法分类

### 按技术路线分类

```
1. 基础架构
   ├── ReAct: 推理-行动交错循环
   ├── Toolformer: 工具学习范式
   └── AutoGPT: 完全自主Agent

2. 自我改进
   └── Reflexion: 语言自我反思机制

3. 多智能体协作
   ├── AgentsNet: 协调与协作推理
   ├── Player-Coach: 角色分离的实时纠正
   └── CASCADE: 级联范围通信重规划

4. 适应性与技能
   ├── Adaptation Survey: 后训练、记忆、技能框架
   ├── Agentic Skills: 技能生命周期管理
   └── Agentic RL: 工具集成推理强化学习

5. 安全与防御
   ├── Attack-Defense: 攻击面与防御机制
   └── Security of Agentic Commerce: 商业Agent安全

6. 系统优化
   ├── Understanding-Optimizing: CPU-GPU协同调度
   └── Evolution Architecture: 软件架构演进

7. 开放世界工具
   └── ToolOmni: 主动检索+接地执行
```

### 主流架构对比

```
方案1: 单智能体循环 (ReAct)
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   观察      │ ──► │   推理      │ ──► │   行动      │
│  (Obs)      │     │  (Thought)  │     │  (Action)   │
└─────────────┘     └─────────────┘     └──────┬──────┘
       ▲                                       │
       └───────────────────────────────────────┘

方案2: 自我反思 (Reflexion)
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   尝试      │ ──► │   失败      │ ──► │   反思      │
│  (Attempt)  │     │  (Failure)  │     │  (Reflect)  │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
       ┌───────────────────────────────────────┘
       ▼
┌─────────────┐
│   重试      │
│  (Retry)    │
└─────────────┘

方案3: 多智能体协作 (Player-Coach)
┌─────────────┐         ┌─────────────┐
│   Player    │ ◄─────► │    Coach    │
│  (执行者)    │  不确定时 │  (指导者)    │
│ 推理-行动循环 │ ◄─────► │ 元认知反馈   │
└──────┬──────┘  触发   └─────────────┘
       │
       ▼
    环境交互

方案4: 工具集成 (Toolformer/ToolOmni)
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   查询      │ ──► │  工具检索    │ ──► │  工具执行    │
│  (Query)    │     │ (Retrieval) │     │ (Execution) │
└─────────────┘     └─────────────┘     └──────┬──────┘
       ▲                                       │
       └───────────────────────────────────────┘
                    结果反馈
```

---

## 核心挑战

1. **规划与推理**: 多步骤任务的分解与执行
2. **工具使用**: 何时、如何选择和调用工具
3. **记忆管理**: 长期记忆存储与检索
4. **多智能体协调**: 协作机制与通信协议
5. **安全与对齐**: 防止有害行为和攻击
6. **适应性**: 从经验中学习和改进

### 关键技术

| 技术 | 描述 | 代表工作 |
|------|------|----------|
| ReAct | 推理与行动交错 | ReAct (NeurIPS 2023) |
| 工具学习 | 教LLM使用外部工具 | Toolformer, ToolOmni |
| 自我反思 | 通过语言反思改进 | Reflexion |
| 多智能体协作 | 角色分离与协作 | Player-Coach, AgentsNet |
| 强化学习 | 结果驱动的优化 | Agentic RL Survey |
| 记忆管理 | 上下文压缩与检索 | Active Context Compression |

---

## 技术演进

```
2022-2023: 基础范式确立
    ├── ReAct: 推理-行动循环
    ├── Toolformer: 工具学习
    └── AutoGPT: 自主Agent
    ↓
2023-2024: 自我改进与反思
    ├── Reflexion: 语言自我反思
    └── 各类Agent框架涌现
    ↓
2024-2025: 多智能体协作
    ├── AgentsNet: 协调与推理
    ├── Player-Coach: 角色分离
    └── 多智能体Benchmark发展
    ↓
2025-2026: 适应性与规模化
    ├── Adaptation Survey: 统一适应框架
    ├── Agentic Skills: 技能生命周期
    ├── Agentic RL: 强化学习优化
    └── 安全与防御体系建立
```

---

## 重要性标注说明

- **⭐⭐⭐ 必读基础**: 领域奠基性工作，必读论文，理解核心技术路线的关键
- **⭐⭐ 值得关注**: 有创新点或特定场景有价值，建议浏览了解
- **⭐ 简单参考**: 数据集或特定应用，快速了解即可

---

## 阅读建议

### 入门路线（必读基础 ⭐⭐⭐）

1. **ReAct** → 理解推理-行动循环基础（NeurIPS 2023）
2. **Toolformer** → 理解工具学习范式
3. **AutoGPT** → 理解完全自主Agent设计
4. **Reflexion** → 理解自我反思改进机制

### 进阶路线（必读基础 ⭐⭐⭐）

5. **Agentic LLM Survey** → Agentic LLM全景综述
6. **Agentic AI Architectures** → 系统架构深度解析
7. **Adaptation Survey** → 适应性学习框架
8. **Agentic Skills** → 技能系统综述
9. **Agentic RL Survey** → 强化学习优化方法

### 扩展阅读（值得关注 ⭐⭐）

- **AgentsNet/Player-Coach**: 多智能体协作
- **ToolOmni**: 开放世界工具使用
- **Attack-Defense**: 安全攻防
- **Understanding-Optimizing**: 系统优化

---

## 开源项目与框架

| 项目 | 链接 | 特点 |
|------|------|------|
| AutoGPT | [github.com/Significant-Gravitas/AutoGPT](https://github.com/Significant-Gravitas/AutoGPT) | 完全自主Agent |
| LangChain | [github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain) | 工具链与Agent编排 |
| LangGraph | [github.com/langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) | 多智能体工作流图 |
| AutoGen | [github.com/microsoft/autogen](https://github.com/microsoft/autogen) | 多智能体对话框架 |
| MetaGPT | [github.com/geekan/MetaGPT](https://github.com/geekan/MetaGPT) | 多智能体软件开发 |
| CAMEL | [github.com/camel-ai/camel](https://github.com/camel-ai/camel) | 多智能体协作框架 |
| OpenClaw | 开源 | 区块链+Agent钱包 |

---

## 研究方向

1. **高效规划**: 长序列任务的规划与执行
2. **工具学习**: 开放世界工具发现与使用
3. **多智能体协作**: 去中心化协调机制
4. **记忆管理**: 长期记忆与上下文压缩
5. **安全对齐**: 鲁棒的Agent安全机制
6. **适应性学习**: 持续学习与技能积累
7. **评估基准**: 全面的Agent能力评测

---

## 参考文献 BibTeX

```bibtex
% 基础论文
@article{yao2023react,
  title={ReAct: Synergizing Reasoning and Acting in Language Models},
  author={Yao, Shunyu and Zhao, Jeffrey and Yu, Dian and Du, Nan and Shafran, Izhak and Narasimhan, Karthik and Cao, Yuan},
  journal={arXiv preprint arXiv:2210.03629},
  year={2023}
}

@article{schick2023toolformer,
  title={Toolformer: Language Models Can Teach Themselves to Use Tools},
  author={Schick, Timo and Dwivedi-Yu, Jane and Dess{\`i}, Roberto and Raileanu, Roberta and Lomeli, Maria and Hambro, Eric and Zettlemoyer, Luke and Cancedda, Nicola and Scialom, Thomas},
  journal={arXiv preprint arXiv:2302.04761},
  year={2023}
}

@article{shinn2023reflexion,
  title={Reflexion: Self-Reflective Agents with Verbal Reinforcement Learning},
  author={Shinn, Noah and Cassano, Federico and Gopinath, Ashwin and Narasimhan, Karthik and Yao, Shunyu},
  journal={arXiv preprint arXiv:2303.11366},
  year={2023}
}

% 综述论文
@article{plaat2025agentic,
  title={Agentic Large Language Models, a survey},
  author={Plaat, Aske and Broekens, Joost and Dalm, Suzan and Henselmans, Mark and Broere, Dimitri},
  journal={arXiv preprint arXiv:2503.23037},
  year={2025}
}

@article{jiang2025adaptation,
  title={Adaptation of Agentic AI: A Survey of Post-Training, Memory, and Skills},
  author={Jiang, Pengcheng and others},
  journal={arXiv preprint arXiv:2512.16301},
  year={2025}
}

@article{agentic2026architectures,
  title={Agentic Artificial Intelligence (AI): Architectures, Taxonomies, and Open Challenges},
  author={Abou Ali, Mustafa and Dornaika, Fadi and Charafeddine, Jana},
  journal={arXiv preprint arXiv:2601.12560},
  year={2026}
}

@article{wang2026attack,
  title={The Attack and Defense Landscape of Agentic AI},
  author={Wang, Xinyuan and others},
  journal={arXiv preprint arXiv:2603.11088},
  year={2026}
}

@article{garg2026skills,
  title={SoK: Agentic Skills --- Beyond Tool Use in LLM Agents},
  author={Garg, Aryaman and others},
  journal={arXiv preprint arXiv:2602.20867},
  year={2026}
}

% 多智能体
@article{park2025playercoach,
  title={Player-Coach Teamwork: Multi-agent Collaboration for Improving LLM Reasoning},
  author={Park, Heewon and Kwon, Minhae},
  booktitle={NeurIPS 2025 Workshop on Scaling Environments for Agents},
  year={2025}
}

@article{agentsnet2025,
  title={AgentsNet: Coordination and Collaborative Reasoning in Multi-Agent LLMs},
  author={others},
  journal={arXiv preprint arXiv:2507.08616},
  year={2025}
}
```

---

*最后更新: 2026-04-22*
