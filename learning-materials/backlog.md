# Backlog：技术项沉淀汇总

记录已识别但尚未覆盖的技术内容空白，按优先级分组。

---

## P0 — 高价值，目录框架已就绪

| 技术项 | 目标目录 | 说明 |
|--------|---------|------|
| Qwen-VL 多模态模型知识 | `llm/07-multimodal/vlm/` | 目录已建但为空，需补充 Qwen-VL 的选型评估、SFT 微调、LoRA、量化部署等实践知识 |
| LoRA 微调原理与实践 | `llm/03-training/fine-tuning/lora-and-qlora/` | 需补充 LoRA 原理、rank 选择实验、LoRA 融合推理等实操经验作为已有内容的案例补充 |
| Prompt Engineering 方法论 | `llm/04-serving/prompt-engineering/` | 目录为空，需补充迭代优化 Prompt 的系统性方法（格式约束→字段枚举→CoT→坏例分析） |
| 推理优化实践 | `llm/04-serving/optimization-techniques/` | 需补充模型压缩、量化 (int8/int4)、缓存策略、Early Exit 等轻量化部署手段 |
| 命名实体识别 (NER) | `knowledge-graph/02-construction/named-entity-recognition/` | 目录为空，需补充 BERT-CRF、BIES、动态标签生成、切分一致性验证等完整 NER 技术笔记 |

## P1 — 有价值，需确认目标目录

| 技术项 | 目标目录 | 说明 |
|--------|---------|------|
| Agent 框架对比（OpenClaw vs LangChain） | `agentic/06-frameworks-and-tools/langchain-agents/` | 需补充 OpenClaw 框架的选型理由、Skill 设计、任务规划流程，丰富 agent 框架对比视角 |
| AI Agent 可靠性设计 | `agentic/07-evaluation/` 或 `agentic/06-frameworks-and-tools/` | 需补充 Agent 循环检测、限流、分层容错等工程策略 |
| LLM 数据标注流水线 | `training-infra/06-ml-operations/data-pipelines/` | 目录为空，需补充多线程标注、容错、质量保障等完整实践经验 |
| SFT 全链路落地 | `llm/03-training/fine-tuning/sft/` | 需补充 SFT 中的动态标签生成、DataLoader 优化、CRF 解码等全链路经验 |
| 模型评估方法论 | `llm/05-evaluation/` | 需补充字符级 vs 实体级评估、LLM as Judge、分层验证等多层评估体系 |
| 模型选型决策框架 | `llm/02-models/` | 需补充任务复杂度/延迟/成本/数据/迭代频率五维决策框架 |

## P2 — 长期方向，内容边界待明确

| 技术项 | 目标目录 | 说明 |
|--------|---------|------|
| 多模态大模型 vs 纯视觉/纯文本选型对比 | `llm/07-multimodal/vlm/` | 选型分析可作为 vlm overview 的一部分 |
| 开源 vs 闭源模型 TCO 分析 | `llm/04-serving/` | 需补充成本对比和场景化选型判断 |
| BIES vs BIO 标注方案对比 | `knowledge-graph/02-construction/named-entity-recognition/` | 可补充为 NER 目录下的具体技术笔记 |
| 深度追问集 | 待定 | 各技术方向的进阶追问可整理为深度技术指南 |

*最后更新: 2026-05-19*