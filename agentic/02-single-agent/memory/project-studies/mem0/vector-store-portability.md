# Mem0 后端可移植性：同一 `filters` 在不同向量库上的差别

> 最后更新：2026-11-03
> Version Basis：`mem0ai/mem0@94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`，Python OSS `v2.2.1`，TypeScript OSS `ts-v3.3.1`
> Observed At：2026-11-03
> Scope：已核对 Python `qdrant`／`pgvector`／`chroma`／`weaviate`／`milvus`／`redis` 六个适配器与 `VectorStoreBase` 契约，以及 TypeScript OSS 的 payload 键与过期过滤。其余 Python 适配器、各 TypeScript 适配器未逐一核对，不能据本页外推。本文是静态源码阅读，未连接真实后端运行。
> 阅读基础：了解 `add()`／`search()` 的 scope filters、payload metadata 与向量检索；Drift Risk：high。

Memory 层不是完全不碰 filter：`search()` 检测到高级操作符时会先做归一化和白名单校验；但 `get_all()` 不做这一步，校验通过也不等于后端支持该操作符。因此“同一段 filters 在 Qdrant 上成立”并不能推出它在 Chroma 或 Milvus 上成立：有的后端报错，有的把条件降级成等值，有的直接丢掉条件。本页解释这条边界落在哪一层、六个已核适配器各差在什么位置，以及哪些差别不属于后端而属于 Memory 层。

## 1. 处理边界：搜索路径归一化，`get_all` 透传，后端决定执行

- `search()` 在 `mem0/memory/main.py:1481–1493` 检测 `_has_advanced_operators`，命中后调用 `_process_metadata_filters`（`:1538–1613`）：已知比较操作符按通用格式保留，`OR`／`NOT` 改写为 `$or`／`$not`，白名单外的操作符直接 `raise ValueError`（`:1576–1577`）。
- 触发条件是“高级操作符”：`AND`／`OR`／`NOT`、**已知**操作符字典、通配 `*`。纯简单等值原样下传；**只含未知操作符的字典也不触发**，会绕过校验直达适配器。
- `get_all()`（`main.py:1269–1339`）没有这段处理：同一份 filters 在 `get_all` 直接进入适配器，在 `search` 可能先被 Memory 拒绝。异步实现同理。
- 校验通过也不代表后端支持：`$or`／`$not` 是否被接受、具体操作符是否降级，仍由适配器决定。失败方式因此分成三类：**显式报错**、**静默降级为等值/丢弃**、**按后端自己的类型系统解释**。

## 2. 六个适配器的实际行为

| 适配器 | 简单等值 | 范围 `gt/gte/lt/lte` | `in`／`nin` | `contains`／`icontains` | `*` 通配 | 不支持的输入 |
|---|---|---|---|---|---|---|
| Qdrant | 支持 | 支持，且能识别日期区间 | 支持 | `MatchText`，是否分词取决于全文索引 | 跳过（匹配全部） | 同字段混用范围与其他操作符会 `raise`；未知操作符 `raise` |
| pgvector | 支持 | 支持，但强制 `::numeric` | 支持，须为 list | SQL `LIKE`／`ILIKE` | `payload ? key` | 未知操作符 `raise` |
| Chroma | 支持 | 支持 | 支持 | **降级为等值** | **跳过（条件消失）** | 未知操作符同样降级为等值 |
| Milvus | 支持 | **不支持** | **不支持** | **不支持** | **跳过（条件消失）** | 操作符字典 `raise`；key 须匹配 `^[a-zA-Z_][a-zA-Z0-9_]*$` |
| Weaviate | 仅 `user_id`／`agent_id`／`run_id` | **丢弃** | **丢弃** | **丢弃** | 按等值比较 | 其他字段与非等值条件**静默丢弃**；falsy 值也跳过 |
| Redis | 转成 `Tag(key) == value` | **无范围语义** | **无集合语义** | **无子串语义** | 按字面值比较 | 操作符字典会被当作值传入，行为未定义 |

证据位置：Qdrant `mem0/vector_stores/qdrant.py:278–336`；pgvector `pgvector.py:38–115`；Chroma `chroma.py:259–350`；Milvus `milvus.py:149–183`；Weaviate `weaviate.py:181–196`、`:357–368`；Redis `redis.py:148–155`、`:324–330`。

表列的是**到达适配器之后**的行为。若一组 filters 同时含已知与未知操作符，会在 Memory 层先报错；纯未知操作符字典才会落到这里。`$or`／`$not` 的支持也按适配器不同，本页未逐一核对。

两处值得单独说明：

- **Qdrant 的日期区间识别**：范围值能识别为日期时使用 `DatetimeRange`，否则用数值 `Range`（`qdrant.py:300–311`）。这让 ISO 时间字符串的范围过滤在 Qdrant 上语义正确。
- **pgvector 的数值强转**：`gt/gte/lt/lte` 一律把字段转成 `numeric` 再比较（`pgvector.py:40–43`）。对 `created_at` 这类 ISO 字符串，这更像一个会失败的查询，而不是“按时间比较”；该后果为静态推断，未运行验证。

## 3. hybrid：初始化告警与“本次无关键词结果”是两件事

`VectorStoreBase.keyword_search` 默认返回 `None`（`mem0/vector_stores/base.py:68–89`），但两种情形不能混写：

- **适配器未覆写该方法**（Python）：初始化时就记录 warning，提示 hybrid 关闭（`main.py:543–548`）。这是**有提示**的降级。
- **已覆写但本次返回 `None`**（TS 侧为方法缺失、`null` 或抛错）：只表示本次没有关键词分数，调用时不额外提示（TS 侧同样静默）；实体增强仍独立计算（见[检索正文](./retrieval.md)的补充分数步骤），因此不能称为“纯向量召回”。

覆盖率：固定版本 **25 个已注册 provider（对应 25 个适配器模块）中有 15 个实现** `keyword_search`；其余 10 个走初始化 warning 路径。已核对的六个中 Chroma 未实现，Qdrant／pgvector／Weaviate／Milvus／Redis 实现。

## 4. 过期过滤不在适配器层

`expiration_date` 是调用方写入的 payload 字段，隐藏逻辑在 Memory 层完成，而不是让各 store 自己过滤：

- Python：`_payload_is_expired`（`main.py:442–449`）、`get_all` 的超额取回（`:1324`）、`search` 的候选过滤（`:1684`）。
- TypeScript OSS：`payloadIsExpired`（`mem0-ts/src/oss/src/utils/expiration.ts:39–43`）、`search`（`index.ts:1552`）、`getAll` 的 `fetchLimit = showExpired ? topK : max(topK * 4, 60)`（`index.ts:1881`）。

两者控制流同构。旧版总览写的是“**本文已核对的** payload 过期过滤限 Python 路径”，属于已核范围表述；本轮补核 TS 后，已核范围扩为两套 OSS 的 Memory 层各自实现，不据此认为旧文曾断言 TS 缺失该能力。Platform 的过期语义仍只有公开契约，服务端过滤时机未公开。

## 5. 同一 collection 混用 Python 与 TypeScript

固定版本的 payload 键并不统一：Python 写 `text_lemmatized`／`created_at`／`updated_at`（`main.py:1033–1037`、`:2084–2090`），TS 写 `textLemmatized`／`createdAt`／`updatedAt`（`index.ts:1024–1033`、`:1982–1993`），scope 键则两套都写 `user_id`／`agent_id`／`run_id`。所以：

- 用 Python 客户端读取 TS 写入的记录时，`text_lemmatized` 缺失，关键词/BM25 侧拿不到文本；
- 用 `created_at` 做的过滤或时间展示不会命中 TS 记录；
- 反向同样成立。

但 `data`、`user_id`／`agent_id`／`run_id` 等共享字段仍在，`get()` 仍能取到正文与 scope。因此这是**部分字段契约不兼容**，不是整体互不识别；实际能否互读还取决于后端、collection schema 与向量格式，本轮没有运行证据。

这不是“跨语言契约”，而是同一 commit 的两套实现各自选取的字段名。

## 6. 更新与删除对后端的依赖比预期小

Memory 层在调用适配器前先 `get()` 再合并完整 payload，所以适配器的 `update` 无论“覆盖”还是“合并”，最终收到的都是完整字段（Python `main.py:2075–2099`；TS `index.ts:1977–1999`）。真正依赖后端行为的是两处：

- `list()` 的分页与 `top_k` 上限决定批量删除能否走到全部记录。Python 与 TS 都用固定批量 `1000` 与“重复批次即停止”的保护（`main.py:136`、`:1938–1950`；`index.ts:131`、`:1747–1760`）。
- `reset()` 的分支：TS 对 `langchain` provider 跳过集合删除并给出警告，其他 provider 删除失败也只记录错误（`index.ts:1795–1806`）；Python 走 `hasattr(self.vector_store, "reset")` 分支（`main.py:2153–2162`）。

若某个适配器把过滤条件丢弃（见 §2），受影响的不是删除本身，而是**应用传入的 `filters`**：它会比预期匹配更多记录。

## 7. 使用判据

- 跨这六个已核后端唯一稳定的，是三个 scope 键（`user_id`／`agent_id`／`run_id`）的标量等值；`in` 不是共同能力（Milvus 不接受操作符字典，Weaviate 只处理这三个键），不要把它当作可移植下限。
- 需要范围、子串、通配或逻辑组合时，先确认目标适配器，不要假设“官方都支持”。
- 同一套 filters 在不同后端上得到的差异，不能当作机制结论；需要可移植行为时，把过滤放到应用层（召回后过滤）——但它不能补回已被 `top_k`／threshold 排除的候选。
- 本文核对的是固定 commit 的六个适配器；其他适配器只依据 `VectorStoreBase` 契约推断接口存在，不能据表列行为外推。

## Evidence

- **Status**：Memory 层在搜索路径的条件校验、六个适配器的过滤翻译、`keyword_search` 覆盖、过期过滤所在层、payload 键名为 `Observed`（固定源码）；静默降级、数值强转与部分字段不兼容造成的实际结果差异为条件性 `Inferred`；各真实后端的运行行为、其余适配器与混用部署为 `Unverified`。
- **Sources**：固定上游锚点见 [`notes/source.md §12`](./notes/source.md#12-后端可移植性与-typescript-补查2026-11-03)，按 Python 适配器、Memory 层、TS OSS 与基类分组；Claim 对照见 [`notes/evidence.md`](./notes/evidence.md) M57–M62。
- **Trace**：延续“hybrid 是后端条件能力”（M4）并对过期过滤的已核范围做扩展（M59）；检索算法本身仍由[检索正文](./retrieval.md)承接，更新与删除的控制流由[直接更新](./direct-update.md)、[删除与重置](./deletion-lifecycle.md)承接。
- **Needs**：连接真实后端验证降级/报错的具体表现；其余 19 个 provider 与各 TypeScript 适配器；Python 与 TS 共享 collection 的实测；Platform 服务端过滤时机。
