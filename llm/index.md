# LLM Index

> 如果你是第一次进入本目录，先读 [`README.md`](./README.md) 建立方向感；如果你已经知道想找什么，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `overview.md` 的整体理解主线。

## 目录结构

```
llm/
├── 01-foundations/                  # 基础
│   ├── transformer-architecture/    # Transformer架构
│   │   ├── attention-mechanisms/    # 注意力机制
│   │   │   └── position-bias/       # 位置偏差
│   │   ├── positional-encoding/     # 位置编码
│   │   ├── normalization-and-residual/ # 归一化与残差
│   │   └── long-context/            # 长上下文
│   ├── tokenization/                # 分词
│   └── scaling-laws/                # 缩放定律
│
├── 02-models/                       # 模型
│   ├── open-source-models/          # 开源模型
│   ├── architectural-variants/      # 架构变体
│   └── emergent-abilities/          # 涌现能力
│
├── 03-training/                     # 训练
│   ├── pre-training/                # 预训练
│   ├── fine-tuning/                 # 微调
│   └── alignment/                   # 对齐
│       ├── rlhf/
│       └── dpo/
│
├── 04-serving/                      # 推理与部署
│   ├── optimization-techniques/     # 优化技术
│   ├── serving-frameworks/          # Serving框架
│   └── prompt-engineering/          # 提示工程
│
├── 05-evaluation/                   # 评估
│   ├── benchmarks/                  # 基准测试
│   ├── evaluation-methods/          # 评估方法
│   └── evaluation-frameworks/       # 评估框架
│
├── 06-explainability/               # 可解释性
│   ├── mechanistic/                 # 机制可解释性
│   ├── attribution/                 # 归因方法
│   ├── probing/                     # 探测技术
│   └── counterfactual/              # 反事实解释
│
├── 07-multimodal/                   # 多模态
│   ├── vlm/                         # 视觉语言模型
│   ├── audio/                       # 音频语言模型
│   ├── video/                       # 视频语言模型
│   └── any2any/                     # 全模态模型
│
└── 08-safety-and-society/           # 安全与社会影响
    ├── safety-and-alignment/        # 安全与对齐
    └── social-impact/               # 社会影响
```

## 按问题找入口

- 想看 Transformer 架构原理、分词方法或 Scaling Law：先看 `01-foundations/`
- 想看开源模型系列、架构变体或涌现能力：先看 `02-models/`
- 想看预训练、微调或对齐技术：先看 `03-training/`
- 想看推理优化、服务框架或提示工程：先看 `04-serving/`
- 想看评估基准、评估方法或评估框架：先看 `05-evaluation/`
- 想看机制可解释性、归因、探测或反事实：先看 `06-explainability/`
- 想看 LLM 扩展至视觉、音频、视频或全模态：先看 `07-multimodal/`
- 想看安全对齐或社会影响：先看 `08-safety-and-society/`

## 按阅读目标找入口

- 想查具体的开源模型技术报告：`02-models/open-source-models/`
- 想对比各模型的 benchmark 表现：`05-evaluation/benchmarks/`
- 想了解微调方法选型：`03-training/fine-tuning/` 或 `03-training/alignment/`
- 想了解推理优化工具选型：`04-serving/optimization-techniques/`
- 想查 LLM 多模态模型：`07-multimodal/vlm/`

## 按子目录定位

- `01-foundations/`：Transformer 架构原理、分词、Scaling Law
- `02-models/`：各开源模型系列及其架构变体
- `03-training/`：预训练、微调、对齐的完整训练管线
- `04-serving/`：推理优化、服务框架、提示工程
- `05-evaluation/`：基准测试、评估方法与框架
- `06-explainability/`：机制解释、归因、探测、反事实
- `07-multimodal/`：LLM 扩展至视觉、语音、视频
- `08-safety-and-society/`：安全对齐、社会影响

## 开源仓库与工具存放指南

| 内容类型 | 放入目录 | 示例 |
|---------|---------|------|
| 开源模型系列（Anthropic, DeepSeek, Meta 等） | `02-models/open-source-models/` | 按公司组织，每公司一个目录 |
| 预训练数据工程 | `03-training/pre-training/data-curation/` | 数据构建与质量控制工具 |
| 微调与对齐方法 | `03-training/fine-tuning/` / `03-training/alignment/` | LoRA, RLHF, DPO |
| 推理优化框架 | `04-serving/optimization-techniques/` | vLLM, TensorRT-LLM |
| 提示工程工具 | `04-serving/prompt-engineering/` | PromptFlow, LangChain Prompts |
| 评估基准 | `05-evaluation/benchmarks/` | MMLU, C-Eval, HumanEval |
| 可解释性工具 | `06-explainability/` | TransformerLens, Ecco |
| 多模态模型 | `07-multimodal/` | LLaVA, Qwen-VL |
| LLM 知识库/wiki | `../rag/03-advanced-patterns/llm-wiki/` | Karpathy's LLM Wiki, nashsu/llm_wiki |
