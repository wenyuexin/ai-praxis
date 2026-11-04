# Mem0 直接更新：同一条记忆怎样被更正

> 最后更新：2026-11-03
> Version Basis：`mem0ai/mem0@94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`，Python OSS `v2.2.1`；Qdrant client 默认值另固定到 `v1.12.0`。
> Observed At：2026-10-30；2026-11-03 固定来源复核与证据登记重建
> Scope：以 Python 同步 `Memory.update()` 为主，对照 `AsyncMemory`；Qdrant 是已核对的后端实例，各后端的过滤与更新差异见[后端可移植性](./vector-store-portability.md)。Platform 限于固定公开契约和客户端转发；TypeScript 见文末对照节。
> 阅读基础：了解基本向量检索、记录 ID 和 metadata；Drift Risk：high。

固定 Python OSS 的 `update(memory_id, text)` 接受应用选定的记忆 ID，覆盖该 ID 的主记录及检索表示，再追加一条变更 history。它不会自动找出全库哪些记忆需要更正，也不会因为保存了旧文本就让普通搜索从 history 中恢复旧值。

这与普通 `add()` 的 ADD-only 自动抽取是两个入口，也不同于论文旧版由 LLM 选择 UPDATE 的阶段和 Platform Dream 的 supersede／merge。它们怎样分层见[一致性模型](./consistency-model.md)；本文解释应用已经决定“更正这一条”之后发生什么。

## 从一次事实更正理解主链

假设应用确认“Alice 的出生地是北京”是录入错误，正确值是上海，原记录 ID 为 M。应用调用：

```python
# 教学示意，未执行；M 是应用已确认应更正的现有记录 ID。
memory.update(memory_id=M, text="Alice 的出生地是上海")
```

这里要覆盖的是错误值；“过去在北京工作，现在在上海工作”则可能是两条同时成立的时间事实，不能仅凭词语不同就选择覆盖。判断事实含义、确定应处理哪些 ID，是调用前的应用责任。

固定同步路径按以下顺序执行（[公共入口](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/main.py#L1829)、[内部更新函数](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/main.py#L2052)）：

1. 对新文本生成 embedding，然后进入 `_update_memory()`。直接文本更新没有调用普通 add 的抽取 LLM。
2. 按 ID 读取旧记录；返回不存在时抛出 `ValueError`，存储读取异常则向外传播。
3. 复制旧 payload，合并允许更新的 metadata，写入新 `data`、MD5 `hash` 和关键词字段 `text_lemmatized`；保留原 `created_at`，刷新 `updated_at`。
4. 用新 embedding 和新 payload 更新同一个 ID 的主记录。
5. 向 SQLite history 写一条 `UPDATE`，保留旧文本和新文本。
6. 仅在文本确实变化时，尝试清理旧实体关联、从新文本抽取实体并重新关联。实体 helper 内捕获的异常不会使主入口失败。

这里的实体关联，是把文本中识别出的人名、地点等实体连接到相关 memory IDs；搜索时，这些连接可为相关候选提供排序加分（entity boost）。它不是事实替代链，也不保证所有实体都会被识别和成功关联。

在无并发、主链成功且后端读取已反映写入的假设下，状态可以这样理解。这是源码推演，未运行 API：

| 时点 | 主记录 M | SQLite history | 实体关联 |
|---|---|---|---|
| 更新前 | 北京、旧向量、创建时间 t0 | 假设已有 ADD | 可能关联北京等实体 |
| 主存储更新后 | 同 ID 改为上海、新向量、新 hash／关键词；创建时间仍为 t0，更新时间 t1 | 尚未追加本次事件 | 尚未修复 |
| history 提交后 | 上海 | 新增 `UPDATE: 北京 → 上海`，传入 `created_at=t0`、`updated_at=t1` | 尚未修复 |
| 实体维护尝试结束 | 上海 | 保留本次 UPDATE | 尝试移除旧关联并建立新关联，可能部分失败 |

没有新建一个替代 ID，也没有因此建立 Dream 版本链。若另一个 ID N 仍写“出生地是北京”，它不会随 M 自动改写，之后仍可能被召回；单条 update 不等于全库矛盾消解。

## 哪些字段保留，哪些可以改变

仅传新文本时，旧 payload 的 scope、其他 metadata 和过期日期随复制保留。允许更新的 metadata 是顶层合并，不是嵌套对象的递归合并。`data/hash/text_lemmatized/updated_at` 按更新逻辑重算，`created_at` 保留原值。

公共入口的 `text` 默认是 `None`，因此也可只更新 metadata，例如：

```python
# 教学示意，未执行；M 存在且原记录包含字符串正文。
memory.update(memory_id=M, metadata={"reviewed": True})
```

这时内部函数继续使用旧文本；公共入口没有缓存新文本的 embedding，内部仍对旧文本重新计算 embedding 并更新主存储，而不是直接复用已存向量。它也会刷新更新时间、写 UPDATE history，但因文本未变而不进入实体修复分支。因此“只改 metadata”不能理解成只写一个轻量字段。

`user_id/agent_id/run_id/actor_id` 会被 [`_strip_identity_keys()`](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/main.py#L143) 从传入 metadata 中过滤：相同值不改变归属，试图改值会记录警告。因此不能借这个参数把已有记录迁给另一个用户或会话。但“身份字段不可改”并不等于“已验证调用者有权修改”：该入口按 ID 取记录，应用仍须校验调用者与目标记录的关系。

过期日期还要区分两种入口：

| 调用方式 | 固定路径的处理 |
|---|---|
| 独立 `expiration_date` 参数省略，metadata 也未覆盖该字段 | 保留旧日期 |
| 独立参数传日期 | 先规范化，再写入 |
| 独立参数显式传 `None` | 清除旧过期日期 |
| `metadata["expiration_date"]` | 经过普通顶层合并，不经过独立参数的同一规范化入口 |

两种入口同时提供时，公共入口先复制 metadata，再用独立参数的规范化结果覆盖同名键，因此独立参数优先；若独立参数非法，会先抛错，不退回采用 metadata 中的值。独立参数省略而 `metadata={"expiration_date": None}` 时，顶层合并同样把日期值覆盖为 `None`。“清除”指写入空日期值，不是移除 payload 的这个键；两条路径的日期校验不能视为等价。

所以，单纯更正文案不会让已经过期的记录恢复默认搜索可见；`expiration_date` 是读取期过滤字段，不是向量存储的 TTL。日期严格早于查询时 UTC 当天才按此条件隐藏，到期当天不被该条件隐藏，详见[检索例子](./retrieval.md#一个完整的构造例子)。

Python [`AsyncMemory.update()`](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/main.py#L3507) 的正常主链对应一致：embedding、主存储和 SQLite 操作通过 `asyncio.to_thread` 逐步 await，再 await 实体维护。async 没有把这些步骤变成一个事务，也不表示 Platform 后台事件机制。

## 为什么 get、search 和 history 可以给出不同现象

| 入口 | 从哪里读取 | 回答的问题 |
|---|---|---|
| `get(M)` | 按 ID 从主存储取 payload；不执行 search 的过期判断 | 后端此次返回的 M 当前写着什么 |
| `search(query, filters=...)` | 主存储的候选，经范围选择、过期过滤、评分与 top-k | 此次查询选中了哪些记忆 |
| `history(M)` | SQLite 的变更条目 | 本地记下了 M 的哪些文本变化 |

这三者没有共同的“查历史并重建当前值”步骤。若 M 的主存储已更新，普通搜索再次返回 M 时，正文取自其新 payload。history 中仍有北京，是历史文本被保留；不是当前值自动退回北京。

搜索没有返回 M 也不足以判断更新失败。M 需要匹配 filters、进入 semantic 候选池、通过过期过滤与原始 semantic threshold，并在融合排序和 top-k 后保留；包装还会丢弃空正文。新 embedding 会改变查询相似度，刷新 `updated_at` 则不会直接提高固定评分函数的分数。关键词和实体信号不能把 semantic 池外 ID 直接纳入结果，完整计算过程由[检索机制](./retrieval.md#从查询到返回资格先于融合)解释。

因此，`get(M)` 返回上海、默认 `search()` 无 M、`history(M)` 含北京可以同时成立。例如 M 的原过期日期被保留，或它未进入本次候选／排名。若 history **只有旧事件而缺本次 UPDATE**，则另需考虑下一节的部分失败情形。

history 也不是完整可回滚快照。[固定表结构和写入](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/memory/storage.py#L102) 保存独立 history UUID、memory ID、旧／新文本、事件、时间、actor 和 role，没有完整 metadata、过期日期与向量快照。直接更新写入的 `created_at` 是原 memory 的创建时间，本次更新时间是 `updated_at`，不能都当作 history 条目自己的创建时点。

相同文本重复 update 仍更新主存储、刷新时间并写 UPDATE history，`text_changed` 仅控制实体修复。因此 UPDATE 事件不保证文本变化；metadata-only 更新也不能只从旧／新文本恢复完整字段差异。

## 主记录、history 与实体维护的失败边界

主存储更新先于 history，history 又先于实体维护。SQLite 的事务仅包住 history 插入，没有跨三者的统一回滚。以下是控制流支持的可能状态，属于推断，不是故障复现实测：

| 失败位置 | 对外与持久化状态的可能关系 |
|---|---|
| 主存储更新调用抛错 | 后续 history 与实体维护不执行；网络等异常下，报错本身不能证明远端未应用写入 |
| 主存储调用已返回，history 写入失败 | 异常向外传播；主记录可能已是上海，但缺此次 UPDATE，也没有回滚主记录的代码；实体维护尚未执行 |
| history 已提交，实体维护发生被 helper 捕获的异常 | 主记录与 history 已更新，接口仍可成功；实体关联可能缺失或残留 |

实体清理还受当前进程的 entity store 是否初始化及扫描 `top_k=10000` 的范围影响，不能理解成无条件修复任意规模索引。残留旧链接可能改变 M 的 entity boost；在主记录已更新的条件下，M 的返回正文仍是上海。旧实体信号、其他 ID 的旧文本和 history 的旧文本，需要分别判断。

Qdrant 适配器在新 vector 与 payload 都存在时，以同 ID `PointStruct` 调用 [`upsert`](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/vector_stores/qdrant.py#L500)。dense 使用新 embedding；只有具备 BM25 slot、文本非空且编码成功时才加入重算的 sparse vector。这里的有条件刷新不能外推给所有后端。

适配器省略 `wait` 不代表只确认入队：依赖下界 [`qdrant-client v1.12.0`](https://github.com/qdrant/qdrant-client/blob/v1.12.0/qdrant_client/qdrant_client.py#L1472) 的 `upsert` 默认 `wait=True`，等待变更应用。但 Mem0 只声明版本下界，且允许注入 client；实际安装版本、服务端与读一致性配置未确认。等待写入应用也不保证记录通过某次 search 的筛选。故不能由此承诺任意部署即时可搜，也不能反向断言必有延迟。

## Platform 的相似接口到哪里为止

固定 Python [`MemoryClient.update()`](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/client/main.py#L463) 构造 payload 并发送单条 memory 的 PUT；history 另发 GET。[固定 OpenAPI](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/openapi.json) 描述更新记录与历史响应；这支持公开接口形状，不能展示服务端更新与 Dream 状态链如何联动。

因此不能把 OSS 的 SQLite 顺序和部分失败状态外推到托管服务，也不能把 Platform **add** 的 `PENDING/event_id` 契约直接套给 update。线上索引刷新、supersede／merge 联动和失败语义仍需服务端材料或行为证据。

## TypeScript 对照

固定 `ts-v3.3.1` 的 OSS `Memory.update(memoryId, options)` 与 Python 同构，但契约面不同：`options` 为 `{ text?, data?, metadata?, expirationDate? }`，也接受字符串简写为 `text`；`data` 是弃用别名，二者都没有时以已存文本回退。至少需提供 `text`／`metadata`／`expirationDate` 之一，否则抛错（`mem0-ts/src/oss/src/memory/index.ts:1651–1697`）。`expirationDate: null` 表示清除该字段，其余值先按 `YYYY-MM-DD` 归一化（`index.ts:1681–1685`；`utils/expiration.ts:15–36`）。

控制流与 Python 对齐：读现有记录 → embedding → 写回全量 payload（合并旧 payload、刷新 `updatedAt`）→ 写一条 `UPDATE` history → **文本变化时**才清理并重建实体关联（入口 `index.ts:1688`；内部 `index.ts:1951–2012`）。embedding 不是“文本变化才算”：提供 `text` 时按新文本计算，metadata-only 时对回退得到的旧文本计算，文本未变也照算。因此 metadata-only 更正同样有 embedding 调用与失败点，不是轻量字段更新。但有三处可见差别：

- **history 字段**：TS 的 `addHistory` 只写 memory id、旧／新文本、事件与时间（`storage/SQLiteManager.ts:50–62`），没有 Python 那样的 `actor_id` / `role`；两边都保留旧文本，都不是回滚快照。
- **字段命名**：TS 写 `textLemmatized` / `createdAt` / `updatedAt`，Python 写 snake_case；共享同一 collection 时，这些派生字段互不认识，但 `data` 与 scope 键仍共享，属部分字段契约不兼容，见[后端可移植性 §5](./vector-store-portability.md#5-同一-collection-混用-python-与-typescript)。
- **实体维护失败**：TS 用 `try/catch` 记录 warning 后继续（`index.ts:2002–2011`）；Python 的同类错误也在 helper 内吞掉，因此两者对外都表现为“接口成功但实体关联可能缺失”。

Platform 侧两个客户端都只是转发：Python `MemoryClient.update()` 与 TS `MemoryClient.update()` 都把 `expiration_date` 放进 PUT payload（TS 另接受 `timestamp`），history 走独立 GET（`mem0-ts/src/client/mem0.ts:415–458`、`:558–566`）；服务端如何把更新与 Dream 状态链联动仍未公开。

## Evidence

- **Status**：固定 Python 同步／异步路径、字段处理、history、Qdrant 适配器与依赖默认值为 `Observed`；TypeScript OSS 的入口、字段与历史写入为 `Observed`；失败状态和教学例子为 `Inferred`；Platform 接口说明限于公开契约为 `Verified`，客户端转发为 `Observed`；具体部署的可见性、并发与故障行为为 `Unverified`。
- **Sources**：上述固定上游锚点及 [`notes/source.md §9`](./notes/source.md#9-直接更新来源重建2026-11-03)、[`§12`](./notes/source.md#12-后端可移植性与-typescript-补查2026-11-03)，按入口、存储、读取、后端和公开契约分组；Claim 对照见 [`notes/evidence.md`](./notes/evidence.md) M32–M35、M57（本轮重新核对后的重建登记）。
- **Trace**：从已核对的固定源码研究提炼为直接更新专题；试用过程记录留作历史，不作为正式来源终点。本页保留更新与读取的连接，完整检索算法仍由检索正文承接。
- **Needs**：实际部署下写后读、并发更新、故障残留与旧值再写入的行为证据；其他后端适配器的更新语义；Platform 服务端状态链与事务材料；Python 与 TS 共享 collection 的实测。固定 Python 显式删除及批量清理已另见[删除与重置](./deletion-lifecycle.md)，其范围来自独立路径核查，不能由本页的更新顺序推出。
