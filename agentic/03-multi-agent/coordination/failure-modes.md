# Failure Modes

> 适用范围：多智能体协调中的典型失败模式
> 阶段状态：问题专题，第一版
> 使用说明：本文件用于整理多智能体系统在协作、协调、状态传播与消息交换过程中出现的典型失败模式，目标是建立问题地图，而不是给出穷尽分类或最终定量结论

## 一、为什么 failure modes 值得单独成文

如果只看成功案例，多智能体系统很容易被叙述成：

- 分工更细
- 并行更强
- 视角更多
- 能力更高级

但在真实系统中，multi-agent 的主要难点往往并不在“如何多加几个 agent”，而在：

- 任务如何拆得不过细也不过粗
- 中间状态如何同步而不污染
- 消息如何传递而不扭曲
- 局部错误如何被限制，而不是被放大
- 系统如何避免死锁、空转和冗余劳动

因此，failure modes 不是 multi-agent 的边角问题，而是理解它的主入口之一。

也正因为如此，`failure-modes.md` 应与 `coordination/topology.md` 配套阅读：

- topology 解释系统怎么组织
- failure modes 解释系统会怎么失效

---

## 二、先建立一个保守的问题地图

当前先不追求穷尽分类，而采用一个保守、可扩展的高层问题地图：

1. coordination failure
2. state divergence
3. error amplification
4. redundancy and repeated work
5. deadlock and loop
6. semantic misalignment
7. observability and attribution failure

这个分类框架的目标不是替代后续更细的 failure taxonomy，而是帮助当前目录先形成稳定的问题入口。

---

## 三、七类高层失败模式

### 3.1 Coordination Failure

协调失败指的是：系统中虽然存在多个 agent，但任务分配、执行顺序、依赖管理或结果归并没有被有效组织。

常见表现：

- 任务拆解不合理
- 子任务边界相互重叠
- 依赖关系没有被清晰表达
- 结果聚合阶段缺少一致性检查

为什么它关键：

- 它是很多其他 failure 的上游来源
- 如果 coordination 层出了问题，communication、shared memory、specialization 都会被连带削弱

与 topology 的关系：

- `centralized` 中更容易表现为协调器分配错误
- `independent` 中更容易表现为缺少真正协调
- `hybrid` 中更容易表现为层级边界不清

### 3.2 State Divergence

状态漂移是指多个 agent 对当前任务上下文、世界状态、共享记忆或中间结果的理解逐渐分叉，最终不再对齐。

常见表现：

- 多个 agent 使用不同版本的中间结论
- 某个 agent 基于过时状态继续推理
- 共享上下文被局部覆盖或污染
- 局部修正没有回写到全局视图

为什么它关键：

- multi-agent 的很多错误不是单点错误，而是“多个局部都自洽，但系统整体已不一致”
- 这类问题通常比单纯执行错误更难发现

与 topology 的关系：

- `decentralized` 最容易受影响，因为状态传播依赖横向同步
- `hybrid` 如果层级回写机制不清，也很容易出现隐蔽状态漂移

### 3.3 Error Amplification

错误放大指的是：本来局部可控的小错误，在多 agent 协作中被复制、传播、叠加或强化，最终导致全局性偏差。

常见表现：

- 上游 agent 的错误假设被多个下游 agent 当作前提继续扩展
- 聚合器没有识别局部错误的一致性幻觉
- 多个 agent 独立得出相似但同样错误的结论，被系统误判为“交叉验证通过”

为什么它关键：

- 这是 multi-agent 区别于 single-agent 的重要风险来源之一
- agent 越多、消息链越长、状态层次越深，错误放大的机会越多

与 topology 的关系：

- `independent` 聚合结构特别容易出现“错误一致性被误判为共识”
- `centralized` 至少更容易在某些阶段拦截和收束错误

### 3.4 Redundancy and Repeated Work

冗余与重复劳动是指多个 agent 在没有明确必要的情况下重复处理相似问题、重复检索相似信息或生成近似相同的中间产物。

常见表现：

- 多个 agent 重复查找同一信息
- 相似任务被拆给多个角色却没有真正分化
- token、消息和上下文被反复消耗在低价值重复工作上

为什么它关键：

- 这类问题不一定会立刻导致错误结果，但会快速吞噬 multi-agent 的收益
- 从外部看系统还在“正常工作”，但成本结构已经失衡

与 topology 的关系：

- `independent` 最容易产生大规模重复劳动
- `centralized` 若协调器足够清晰，较容易减少重复

### 3.5 Deadlock and Loop

死锁与空转指的是：系统进入无法有效推进的状态，要么互相等待，要么在低价值路径上不断循环。

常见表现：

- agent A 等 agent B，agent B 又等待 agent A
- 聚合器不断要求补充信息，但下游 agent 只返回重复结果
- 系统在任务分配、澄清、确认之间来回循环，却不产生新信息

为什么它关键：

- 这是 multi-agent hidden cost 的典型来源
- 也是最容易让系统在表面“还在运行”时持续消耗预算的问题

与 topology 的关系：

- `decentralized` 和 `hybrid` 更容易因为横向依赖复杂而出现循环
- `centralized` 若控制流写得不清，也会出现协调器主导的空转

### 3.6 Semantic Misalignment

语义失配是指多个 agent 虽然交换了消息，但并没有对齐真正的任务目标、术语含义、约束条件或成功标准。

常见表现：

- 同一术语在不同 agent 中含义不同
- 上下游 agent 对“完成任务”的定义不一致
- 局部 agent 解决了一个看似合理但其实偏题的问题

为什么它关键：

- 这类问题常被误判为 communication failure，但本质是更深层的目标或语义对齐问题
- 在专业化强、角色差异大的系统里尤其常见

与 topology 的关系：

- `hybrid` 与 `organizational` 结构复杂时更容易出现层级语义漂移
- `decentralized` 中因为缺少统一裁决点，也更容易长时间潜伏

### 3.7 Observability and Attribution Failure

可观测性与归因失败指的是：系统出错了，但我们无法清楚知道是哪个 agent、哪条消息、哪次状态传播、哪段协调逻辑导致了问题。

常见表现：

- 日志足够多，但没有结构化关联
- 系统能看到“结果错了”，却看不到“错误从哪一层开始传播”
- 多轮交互后，很难判断是 planning 问题、protocol 问题还是 shared memory 问题

为什么它关键：

- 这类失败不会直接改变任务结果，却会极大增加调试和维护成本
- 它决定 multi-agent 系统是否真的可治理

与 topology 的关系：

- `decentralized` 与 `hybrid` 更容易归因困难
- `centralized` 通常更利于建立清晰 trace，但也可能把所有问题都错误归因到协调器

---

## 四、failure modes 与 topology 的对应关系

可以先形成一个保守映射：

- `independent / loose aggregation`
  - 高风险：重复劳动、错误放大、聚合误判
- `centralized`
  - 高风险：协调器瓶颈、集中式偏差、全局分配错误
- `decentralized`
  - 高风险：状态漂移、消息扩散失控、定义不统一、归因困难
- `hybrid`
  - 高风险：边界模糊、局部与全局规则冲突、复杂性过高导致调试困难

这个映射不是严格一一对应，但足以帮助当前主干建立“结构如何影响失败”的基本判断力。

---

## 五、为什么 failure 不应只理解为 communication breakdown

把所有问题都归结为 communication breakdown 太粗糙了，因为：

- 有些问题是任务拆解失败，而不是消息传不清
- 有些问题是状态版本失配，而不是消息没送达
- 有些问题是语义目标没对齐，而不是协议格式错误
- 有些问题是系统进入预算黑洞和循环，而不是单次沟通失败

因此，communication 只是 failure map 的一部分，不应被当作总括概念。

---

## 六、当前可形成的保守结论

在不补充新一轮调研的前提下，当前至少可以保守接受以下判断：

1. failure modes 应被视为 multi-agent 的主问题之一，而不是边缘话题。
2. multi-agent 的失败通常是结构性的，而不是单个 agent 能力不足的简单累加。
3. topology 会显著改变失败的形态、传播路径和调试难度。
4. `error amplification`、`state divergence`、`redundancy`、`deadlock/loop` 是最值得优先关注的高层失败类别。
5. 关于更细颗粒度的 failure taxonomy 与定量数据，在写成主干定量结论前仍需要进一步核验原始来源。

---

## 七、后续最值得补齐的内容

- 不同 failure mode 的代表案例
- failure mode 与评估指标的对应关系
- failure mode 与 protocol 设计的对应关系
- failure mode 与 shared memory / specialization 的耦合
- 更细粒度的 taxonomy 与定量证据整理

后续如果证据更充分，这篇文档可以继续拆成：

- `coordination/failure-modes-overview.md`
- `coordination/error-amplification.md`
- `coordination/deadlock-and-loop.md`
- `coordination/state-divergence.md`

当前第一版的目标，是先把多智能体失败问题从“零散提醒”提升为“可组织、可继续扩展的问题地图”。