# GLM-5.x 架构线索笔记

## 这份笔记承接什么

这份笔记只承接 `GLM-5.x` 系列里已经稳定出现、但暂时还不适合写成完整架构定论的技术线索。

它要解决的不是“这一代模型到底完整怎么实现”，而是：

- 当前哪些架构线索已经反复出现
- 哪些线索更像系列背景
- 哪些说法还不能直接压平成 `GLM-5.1` / `GLM-5.2` 的独立事实

## 当前已知

### 1. `GLM-5` 报告提供了系列级架构背景

当前较稳可以确认的系列级背景包括：

- `GLM-5` 技术路线明确面向 `agentic engineering`
- 系列采用大规模 MoE 与稀疏注意力相关路线处理长上下文与成本问题
- `DSA` / sparse attention、长上下文效率优化等，是这一代的重要背景线索

这些内容最稳的承接位仍然是：

- `GLM-5` 本体事实
- 或后续版本的系列背景说明

而不是直接当作 `GLM-5.1` 或 `GLM-5.2` 已独立验证过的完整架构说明。

### 2. `GLM-5.1` 与 `GLM-5` 的架构继承关系已比之前更清楚

当前较高信号材料已经能支持比“只是线索”更强一层的判断：

- `GLM-5.1` 与 `GLM-5` 共享 `glm_moe_dsa` / `GlmMoeDsaForCausalLM`
- NVIDIA NeMo Automodel 文档已直接把 `GLM-5.1` 标成 `updated weights`
- 同一组覆盖页里，`GLM-5`、`GLM-5.1`、`GLM-5.2` 被放在同一架构谱系下区分

因此当前更稳的写法已经可以推进到：

- `GLM-5.1` 大概率不是全新架构改代，而是沿 `GLM-5` 主架构继续演进
- `updated weights` 目前已有较强官方二级来源支撑

但仍要保留边界：

- 这不等于我们已经拿到了 `GLM-5.1` 的完整官方 config 差异说明
- “共享架构”也不自动等于所有实现细节完全不变

### 3. `GLM-5.2` 的架构增量已出现更直接的一手承接

`GLM-5.2` 当前公开材料中已经不只是出现零散线索，而是出现了更直接的官方表述：

- `IndexShare`
- 对 `MTP` 层的改进，用于 speculative decoding
- 在 `1M` 上下文长度下，降低每 token FLOPs
- 提升 speculative decoding 的 acceptance length
- 架构名出现为 `GlmMoeDsaForCausalLM`
- NVIDIA 覆盖页把它与 `GLM-5.1` 的差异概括为 `IndexShare DSA, TileLang sparse-kernel`

这说明 `GLM-5.2` 已经可以较稳地视为：

- 仍处于 `glm_moe_dsa` 这一系列架构谱系中
- 但相对 `GLM-5.1` 已出现明确的架构级增量，而不只是普通发布叙事

但现阶段仍要保守：

- 这些足以支撑“有明确架构增量”这一层判断
- 仍不足以直接写成完整架构复原
- `IndexShare` 与 `MTP` 的更细实现关系，仍应留在线索层继续追

## 当前更稳的分层写法

### 1. 系列背景层

当前适合写成系列背景的内容：

- MoE
- sparse attention / `DSA`
- 长上下文效率优化
- `agentic engineering` 导向下的长链路任务支持

### 2. 版本对象直接事实层

当前适合写成版本对象直接事实的内容，需要满足：

- 在该版本自己的官方博客、模型页、产品文档、release note 中再次出现
- 或有更直接的官方 config / model card / repo 说明支撑

就目前材料看，`GLM-5.1` 在这一层还偏薄，`GLM-5.2` 也仍以技术线索居多。

### 3. 研究线索层

当前最适合保留在这一层的内容包括：

- `GlmMoeDsaForCausalLM` 是否可被视为 `GLM-5.1` / `GLM-5.2` 的稳定架构名
- `IndexShare` 在 GLM-5.2 中到底是新机制、公开命名的新暴露，还是旧路线的新表述
- `updated weights` 到底意味着相同架构下的权重升级，还是还伴随部分结构与推理策略调整
- speculative decoding 与 `MTP` 改进在后续是否会有更正式展开

## 当前最容易误写的地方

### 1. 把系列背景直接写成后续版本的完整架构事实

`GLM-5` 报告中的 MoE / DSA / sparse attention 背景，可以作为后续版本的技术路线来源，但不能直接当作 `GLM-5.1` / `GLM-5.2` 已独立核实的完整实现细节。

### 2. 把第三方托管页的架构字段直接写死

像 `GlmMoeDsaForCausalLM`、层数、专家数、`kv_lora_rank` 这类信息，如果当前主要来自 NVIDIA 托管页或第三方平台，更适合作为强线索，而不是立即写成最终定论。

### 3. 把公开技术线索误当成完整技术报告

`IndexShare`、`MTP` 改进、FLOPs 降低、acceptance length 提升这些线索很有价值，但目前更像：

- 官方开始暴露的局部优化方向
- 后续可继续追源码、模型卡、repo 说明的入口

还不够支持一篇完整的“GLM-5.2 架构解析”。

## 待确认

1. `zai-org/GLM-5` 系列仓库中是否有更直接的 config / 架构说明
2. `GLM-5.2` 后续是否会补出独立 technical report 或更完整 model card
3. `GLM-5.1` 的 `updated weights` 说法是否能在更直接官方链路中被确认
4. `IndexShare` 与 `GlmMoeDsaForCausalLM` 之间的关系能否从源码或权重配置中得到更明确支撑

## 后续动作

1. 如果后续获取官方 config 或 repo 结构说明，优先用来判断哪些线索可以从这里回流到版本对象文
2. 如果后续出现更正式的 GLM-5.2 技术材料，可把这份笔记中的“研究线索层”拆成更稳定的机制专题
3. 在当前阶段，涉及架构判断时，优先引用这份笔记守边界，避免在对象文里一步写满

## Trace

- Input staging:
  - `llm/02-models/open-source-models/glm/temp/1.md`
  - `llm/02-models/open-source-models/glm/temp/glm-5.1-evidence-cleanup.md`
  - `llm/02-models/open-source-models/glm/glm-5.md`
  - `llm/02-models/open-source-models/glm/glm-5.2.md`
- This note extracts recurring architecture-related clue boundaries from the existing version-object studies and cleaned temp materials. It should later be superseded by direct official configs, reports, or repo-level architecture evidence when available.
