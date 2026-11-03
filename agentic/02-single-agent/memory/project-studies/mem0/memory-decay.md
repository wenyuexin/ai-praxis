# Mem0 访问强化：一次搜索怎样影响后续排序

> 最后更新：2026-11-02
> 版本锚点：`mem0ai/mem0@94c3fe9f238f3dbf29c9ce98643bd71eb13077cd` 的 Platform 文档、OpenAPI、客户端及对照用 Python OSS 源码。
> 阅读基础：了解候选、top-k、检索分数和scope。以下是公开契约与静态推演，没有运行托管服务；数字例子不是效果测量。

Memory Decay 在搜索时根据访问历史调整排名，并在搜索后记录返回项的访问。它让“此前哪些记忆经常被找到”参与后续排序，因而连续两次查询不必是对同一静态状态的重复读取。这里的“用过”首先指搜索返回，不等于用户认可、回答采用或事实正确。

## 五种时间或状态不能混成“新旧程度”

| 维度 | 回答的问题 | 对读写的作用 |
|---|---|---|
| created_at／导入timestamp | 记录何时创建，导入对话原来是什么时间？ | Platform timestamp可回填created_at；不等于文本描述的事件日期 |
| event_date／Temporal Reasoning | 事情何时发生，是否匹配问题中的“上周”等时间？ | Platform v3自动给事件时间匹配的记忆排序加成；不是总选最新写入 |
| 访问历史／Memory Decay | 最近何时被搜索返回、返回多频繁？ | 项目开启后，以访问历史乘数调整排名 |
| expiration_date | 应用指定哪一天后不再默认展示？ | 按UTC日期硬隐藏过期项；当天仍可见，次日起默认search/get_all排除 |
| Dream生命周期 | 哪条事实被替代或合并？ | 默认含superseded、隐藏merged；latest_only只选active，与时间排序不同 |

例如“去年在巴黎开会”刚刚导入，记录创建时间可以保留原对话时间，事件时间仍是去年，访问时间则可能还没有。问去年行程时，这条旧事件可以很相关；问现在住哪里时，常被搜索的旧住址也不能仅凭访问热度成为当前事实。

`reference_date`指定本次时间查询的“现在”，用于解释相对时间。公开材料没有保证它同步回拨过期判定、Decay时钟、访问记账或Dream状态；把查询参考日期改回过去，不等于恢复那一天的整个系统快照。Platform时间元数据也可能异步补齐，文档称新记忆日期通常数秒后影响检索，不是写入完成即保证全部时间信号已可用。

## 本次排序与本次访问记账分两步

Decay是项目级`decay`布尔开关，默认关闭，只作用于Platform v3 search。固定Python客户端通过`client.project.update(decay=True)`开启，通过`client.project.get(fields=["decay"])`读取配置；TS对应updateProject／getProject。这些是接口形状说明，本轮没有执行。固定版本未开放调节衰减强度的配置，已查材料也未明确完整套餐资格。

文档公开了以下局部顺序：

1. 扩大候选池到`max(top_k × 3, 50)`，为重排留空间。它仍是有限候选池，不是扫描全部记忆。
2. 对进入Decay前的排名分，乘以访问历史产生的`0.3–1.5`倍数；若启用reranker，文档明确先rerank，再Decay。
3. 按未clamp的乘积排序，公开score限制在`[0,1]`，取最终top-k。
4. 对最终返回的每条记忆fire-and-forget追加一次访问，最多保留最近20次touch。

另一个明确关系是**请求threshold过滤先于Decay**。OpenAPI把threshold描述为最低semantic relevance；公开材料没有完整锁定它相对所有融合、Temporal和rerank步骤的位置。因此不能把上述局部顺序扩写成已经确认的完整托管执行图。

可以用以下记号解释分数，而不假装知道强化函数：

$$
r_i=s_i\,d_i,\quad 0.3\le d_i\le1.5,\qquad
\mathrm{score}_i=\min(1,\max(0,r_i))
$$

$s_i$是进入Decay时的排名分，$d_i$是未公开具体计算式的访问乘数，$r_i$用于内部排序。公开score是多信号相关性分数，不是纯向量余弦、事实置信度或满意度。0.3是**乘数**下限，不是score下限。

假设A与B已通过请求阈值，进入Decay的分数分别为0.8、0.9，假定乘数分别为1.5、0.3，则内部排序值是1.2、0.27，公开score为1.0、0.27，A排前。若请求threshold为0.5，B仍可能以0.27返回：它通过的是Decay前的阈值。这个算例不证明访问一次就获得1.5，也不代表真实样本分布。

两个不同内部乘积若都超过1，公开分都为1，不能据此认定内部并列或反推乘数。客户端如要求最终score不低于阈值，可以再筛选，但这不会撤销服务端已经按“返回过”发起的记账，也可能使应用实际使用少于top-k条。

“软调整、不硬过滤”不保证原来的每个结果仍在top-k。乘数不把候选分清零，但重排和截断会改变返回集合；候选不足、阈值、过期或scope也可能导致不足top-k条。冷记忆仍被保存，不表示它必然会再次被召回或得到探索机会。

## 连续查询中，哪些状态发生了变化

假设A是“3月12日在东京见Maya”，B是“3月20日在巴黎见Maya”。二者符合scope、未过期且具有读取资格。以下逐步改变条件，不是实际测试：

| 时点／操作 | 可以推演的变化 | 不能承诺的结果 |
|---|---|---|
| Q1问“上周在哪见Maya”，指定3月21日为reference_date | 若时间解析覆盖3月12日，A可获Temporal加成 | A必定第一；完整时间boost和分数组合未公开 |
| Q1最终返回A，B只在候选中 | 文档对A发起异步touch；仅作为候选不是该强化条件 | HTTP响应一到，touch就已落库 |
| Q2紧接Q1再查 | 若Q1强化已可见，A的访问历史可参与本轮Decay；否则可能仍读旧历史 | Q2必然提高A排名；其他信号和竞争项也会变化 |
| 很久未再访问A | 访问乘数可向冷记忆区间降低，最低0.3 | 文本被删、到期，或A必然退出每次搜索 |
| 关闭Decay再开启 | 关闭恢复不施加Decay的排序，开启前积累的访问历史保留 | 恢复某次历史查询的同一结果，或重新开启等于清零 |

```mermaid
flowchart LR
    H["已有访问历史"] --> D["Q1：Decay与排序"]
    D --> R["最终返回项"]
    R --> A["异步追加touch"]
    A --> H2["更新后的访问历史"]
    H2 --> Q2["Q2及后续查询可使用"]
```

图中异步箭头没有延迟上界或成功保证。`await search()`等的是HTTP响应，客户端没有额外等待强化完成。公开响应没有Decay乘数、最近20次访问列表或强化完成标志；文档称细节记录在search Event用于支持排查。OpenAPI另声明semantic/BM25/entity的score_breakdown，因此准确边界是**没有公开Decay细分字段**，不能泛称所有分数分解都不存在。普通记录的updated_at也不是touch完成信号。

关闭开关不删除记忆，也不清掉开启期间积累的访问历史。关闭期间是否继续记录访问、在途touch怎样处理、是否暂停某种计时未有明确契约；不能把保留历史解释成“停表”。“恢复排序”只表示移除这一偏置，并不冻结其他信号或系统时间。

旧记忆没有访问历史时，文档把event_date当作一次历史touch；缺失event_date才回退updated_at，不是统一从created_at开始。刚导入的陈年事件不因此必获最高加成。文档一处称未来事件“fresh/full activation”，另两处描述约1.0中性区间，故不把“fresh”解释成确定1.5倍；精确初始化值尚未闭合。具体函数、半衰期、时间单位、最近20次如何合成频率效应也未公开。

## 时间匹配、过期与事实状态怎样约束访问热度

Temporal依据事件时间匹配，Decay依据访问历史。反复返回不能改变事件发生在哪一天，访问频繁也不是旧事实仍有效的证明。Dream状态和expiration另有职责。

假设A的expiration_date是3月31日，UTC进入4月1日后，默认search/get_all排除A；频繁访问不让A突破到期条件。`show_expired=True`仅允许纳入过期记录，不取消scope、候选池、阈值、top-k或Dream状态限制，也不保证A必返回。直接get(id)仍可读取过期主记录，这不是过滤失效；expiration本来就不是删除或权限控制。

若A另被Dream标记superseded，默认仍可能包含，latest_only才按状态排除。include_merged扩大merged可见性，不等同show_expired。多个开关组合的全部优先级、expiry相对每个评分步骤的实现顺序未公开，不据局部说明还原全貌。状态与派生内容新鲜度的关系另见[派生记忆](./derived-memory.md)。

## 访问反馈的意义，以及OSS对照边界

搜索返回就触发记账，意味着调试、测试、预取或反复检索，即使结果未被采用，也可能影响后续排名。因此存在“被返回→获得访问强化→更可能再次返回”的条件性反馈环；乘数有界、只留最近20次，不能证明没有热门偏置或冷记忆长期不被选中，每次touch也不保证严格提高排名。

显式feedback的POSITIVE／NEGATIVE等是另一套接口。已查契约没有说明它增加／抵消touch或直接改Decay乘数，不能把自动强化当正反馈，或承诺负反馈立即抵消曝光。重复query去重、网络重试是否重复计数、并发追加原子性、丢更新防护、强化失败重试和读取一致性同样未知。

若比较两种检索方案，查询顺序和历史访问也可能成为条件；在同一开启Decay的项目先测A再测B，不能自动视为独立静态读。需要固定输入、历史和观测条件才能评价这种偏置是否改善任务效果，本轮没有运行该实验。

固定Python OSS的已核混合评分只合成semantic、keyword和entity，不读created_at、event_date或访问历史；调用链也没有Platform访问乘数。其search扩大`max(top_k×4,60)`候选、过滤过期，再评分，可另接reranker，详见[检索机制](./retrieval.md)。这与Platform Decay的`max(top_k×3,50)`是不同路径，不能混用。

准确结论是“固定已核OSS默认路径没有显式访问recency/frequency评分”，不是所有provider、自定义reranker和外部包装永远不能用时间。OSS支持expiration；直接get与history没有该过期过滤，search/get_all默认有。OSS显式拒绝Platform非空timestamp／reference_date，不能因方法同名继承托管时间能力。

## Evidence

- **Status**：固定Platform文档／OpenAPI的配置、局部顺序和范围为 `Verified`；客户端等待方式、固定OSS函数为 `Observed`；连续查询、反馈环和score算例为条件性 `Inferred`。真实效果、强化函数、延迟与并发为 `Unverified`。
- **Sources**：固定 [Memory Decay](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/memory-decay.mdx)、[Temporal](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/temporal-reasoning.mdx)、[Expiration](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/memory-expiration.mdx)、[OpenAPI](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/openapi.json)、[OSS scoring](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/utils/scoring.py)、[Python client](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/client/main.py)。
- **Trace**：延续M4b、M23、M29；新增 [`notes/evidence.md`](./notes/evidence.md) M51–M55与 [`notes/source.md §8`](./notes/source.md#8-访问强化与时间排序补查2026-11-02)。
- **Needs**：未调用API、未测排名或并发；精确时钟、组合顺序与服务端实现仍未知。未来event_date的措辞不足以定倍率；不将“不硬过滤”升级为原结果全部保留或质量保证。
