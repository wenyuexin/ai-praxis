# LLM (大语言模型)

大语言模型的理论、模型、训练、推理、评估、可解释性与多模态扩展。

> **入口说明**：第一次进入时，先读本页的分类依据与边界说明建立整体方向；如需查找目录结构、子目录定位或学习路径，转向 [`index.md`](./index.md)。

## 分类依据

LLM 目录按"基础 → 模型 → 训练 → 推理 → 评估 → 可解释性 → 多模态 → 安全与社会"的能力生命周期组织。

## 边界说明

| 内容 | 适合放 LLM | 不适合放 LLM |
|------|-----------|-------------|
| Transformer 架构原理 | 01-foundations | 通用 Transformer 原理放 `deep-learning/03-architectures/transformers/` |
| 开源模型技术报告 | 02-models | 模型评估结果放 `05-evaluation/benchmarks/` |
| 预训练、微调、对齐技术 | 03-training | 分布式训练基础设施放 `training-infra/` |
| 推理优化、服务框架 | 04-serving | 训练框架对比放 `training-infra/03-frameworks-and-tools/` |
| LLM 评估基准与方法 | 05-evaluation | RAG 评估放 `rag/04-evaluation/` |
| 机制可解释性（含 Anthropic MI） | 06-explainability/mechanistic | — |
| 安全与对齐 | 08-safety-and-society/safety-and-alignment | — |
| Agent 体系知识 | — | 放 `agentic/`，LLM 目录不重复建 agents/ |
| RAG 体系知识 | — | 放 `rag/`，LLM 目录不重复建 rag/ |
| 视觉为中心的多模态模型（BLIP、CLIP） | — | 放 `cv/05-generative-and-multimodal/`，核心创新在视觉编码/对齐 |
| LLM 为中心的多模态模型（LLaVA、Qwen-VL） | 07-multimodal/vlm | 核心创新在 LLM 接入视觉的方式 |
| 跨谱系模型（BLIP-2） | 07-multimodal/vlm | Q-Former 属于"LLM 接入视觉"范式；CV 侧通过链接引用 |
| 视觉理解能力评估（检测、OCR） | — | 放 `cv/05-generative-and-multimodal/` |
| 多模态指令跟随/推理评估 | 07-multimodal/vlm | — |
| 图像/视频生成（SD、DALL-E） | — | 放 `cv/05-generative-and-multimodal/` |
| 音频/语音（WhisperX） | 07-multimodal/audio | — |

## 与其他目录的关系

- [RAG (检索增强生成)](../rag/) — LLM 与外部知识结合
- [AI 智能体](../agentic/) — LLM 驱动的自主系统
- [知识图谱](../knowledge-graph/) — 结构化知识源
- [强化学习](../reinforce-learning/) — RLHF 基础理论

---

*最后更新: 2026-05-17*
