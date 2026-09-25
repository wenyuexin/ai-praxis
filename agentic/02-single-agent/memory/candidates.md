# Memory Layer — 候选研究对象

> 来源（Trace / Source）：DeepSeek 分享对话《探索未至之境》（AI 综合 + 网页检索产物，无公开上游 URL；本地快照仅作一次性核验上下文，不作正式来源）。
> Evidence 基线：本页所有条目默认 `Inferred / Unverified`（AI 综合 + 二手来源）；进入正文或专题前，须回溯一手来源（论文 / 官方文档 / 源码 / 基准原始实现）核验。
> 阶段：调研队列，非结论；此处不做完整评测，只保留"值得逐个研究的对象 + 核验点"。

## 用途

收拢"Agent 记忆层（memory layer）"方向下值得逐个研究的框架 / 产品 / 服务。研究某一对象时，按 [`research-artifacts.md`](../../../docs/contributing/rules/research-artifacts.md) 决定是否单独建对象研究，核验后的稳定结论再回流到本目录正文或专题。

## 候选对象（记忆层框架 / 产品）

- **Mem0** —— 框架无关的开源记忆层中间件（`add()` / `search()`）。品类基准参照；核验点：LOCOMO 基准之争、OSS 版过度提取噪音、遗忘仅按时间、无置信度追踪。
- **Letta（原 MemGPT）** —— 有状态智能体运行时，core / archival / recall 三层记忆（OS 虚拟内存类比）。记忆是一等公民、支持自我编辑。
- **Zep** —— 企业级时序知识图谱记忆，双时序、支持时间点查询；擅长事实随时间演变。
- **Cognee** —— 开源知识图谱记忆引擎，ECL 管道，向量 + 图，偏 GraphRAG / 多模态。
- **LangMem** —— LangChain / LangGraph 生态的记忆原语，与 LangGraph 存储深度集成。
- **Supermemory** —— 记忆即服务（MaaS），主打高性能 / 低延迟；`Unverified`：自称 LongMemEval 第一。
- **GENOME** —— 轻量本地优先、写入不调 LLM；`Unverified`：自称写入成本约为 Mem0 的 1/2000、~10ms。
- **agentmemory** —— 面向编程智能体，Hook 零干预自动捕获工具调用；`Unverified`：自称 LongMemEval-S 召回 95.2% vs Mem0 68.5%。
- **HyperRecall** —— 超边（Hyperedge）保留事件完整语境 + 仿生衰减（Ebbinghaus / 幂律）/ Hebbian 强化 / 激活扩散；`Unverified`：Apache 2.0 但未找到公开 GitHub、无 star 数、"87.5% / 零摄取成本"等为 PyPI 页面自述。
- **MemOS** —— 定位"记忆操作系统"，可插件化、可本地部署的记忆运行环境。

## 相关分类框架（对接 `memory-taxonomy-conflicts.md`，待回溯一手）

- **CoALA 分类法**：working / episodic / semantic / procedural 四类；学术框架，须回溯 CoALA 原论文，勿以本 AI 综述为准。
- **工程四层（L1–L4）**：寄存器 / SSD / 向量库 / ROM 类比；**载体形式三分**：词元级 / 参数化 / 潜在。均为二手归纳，`Inferred`。

## 从综述归纳的共性缺口（`Inferred`，可作 memory 正文 / 研究切入点）

- 置信度追踪（可靠信息 vs 模糊信息不做区分）；
- 基于重要性 / 访问频率的动态遗忘（而非仅按时间 TTL）；
- 召回率可调（提取"全面 vs 精准"的平衡）；
- 时序推理（"现在用什么 vs 去年用过什么"）与记忆污染 / 迎合偏差。

## 下一步

- 逐个对象核验一手来源后，决定进入正文还是保留为对象研究。
- **Mem0 vs Letta 的 LOCOMO 基准之争**是一处真实口径冲突，若要长期跟踪，考虑记入 `../conflict.md`。
- 分类法内容与 `memory-taxonomy-conflicts.md` 合流前，先核 CoALA 等一手来源。
