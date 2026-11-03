# Mem0 派生产物：模式记忆与用户画像

> 最后更新：2026-11-02
> 版本锚点：`mem0ai/mem0@94c3fe9f238f3dbf29c9ce98643bd71eb13077cd` 的 Platform 文档、OpenAPI 与 Python／TypeScript 客户端。
> 阅读基础：了解消息、记忆记录与 scope。本文解释公开契约及其边界，没有运行托管服务；例子用于说明，不是模型实测输出。

Mem0 不只保存个别事实。Platform 还可以从多条事实提炼新的模式记忆，或按应用定义的字段维护一份用户画像。这两种产物都减少了应用逐条拼接记忆的工作，但生成时机、读取方式和证据可追溯性不同，不能把它们统称为一个始终最新的“用户总结”。

## 三种产物分别回答什么问题

假设某位用户多次提到晨跑、力量训练和马拉松目标：

| 产物 | 示例与表达方式 | 应用怎样消费 |
|---|---|---|
| 个体记忆 | “用户每周跑约40公里”“用户今年想完成马拉松”这类具体条目 | 按问题检索相关记录，或按 scope 分页读取 |
| Synthesis 模式记忆 | 从多条事实概括“用户保持有计划的耐力与力量训练”；仍是一条自由文本记忆 | 进入普通记忆集合，随相关记忆读取；可追查来源 |
| Profiles 用户画像 | 按应用 schema 得到如 `{"training_style":"耐力与力量结合","current_goals":["完成马拉松"]}` 的对象 | 按用户 ID 直接取画像，无需先写语义查询，再由应用放进 prompt |

模式记忆表达跨条目的规律，画像把信息组织成应用预先规定的字段。模式没有替换来源；画像也没有被公开定义为“把全部个体记忆删掉后留下的压缩档案”。它们是否覆盖所有事实、是否概括正确，不能由产物名称推定。

这里的 Synthesis 与 Profiles 是托管能力。固定 OSS 的普通 `add()` 维护消息缓存、history 和实体索引，详见[普通写入机制](./write-pipeline.md)；那些数据不是这两种派生产物，也不能从 OSS 调用链还原托管生成实现。

## Synthesis：新增带来源的模式记忆

Synthesis 是 Dream 中按项目开启的一项功能，与随新增记忆评估的 Supersede／Merge 分开。Supersede 标记旧事实过时，Merge 处理重复或更丰富的事实；Synthesis 则从同一用户的多条记忆中产生新模式，源记录继续保留。官方承诺它对相同 underlying evidence 不重复生成相同模式，但没有公开 evidence 集合的判等规则、提示词或质量阈值。

能否进入合成，要先看资格和时间：

| 条件 | 固定公开契约 | 对判断的影响 |
|---|---|---|
| 套餐与开关 | Pro／Enterprise，按项目 opt-in | 开启 Synthesis 不等于开启整个 Dream；Supersede／Merge 全套餐自动运行 |
| 创建边界 | 只考虑开启后创建的记忆；OpenAPI 表述为不早于 `reflection_enabled_at` | 大量旧记忆不会因开关打开而立即全部回填 |
| scope | 记忆仅带 `user_id`；同时带 agent／run／app 的记录排除 | 同一用户的记录也可能不合格 |
| 数量 | 用户至少20条 memories 才会被考虑 | 不是20条消息；正式材料没有明确该计数如何处理状态、过期和创建边界 |
| 周期 | 每用户每周期最多一次；Pro为7天，Enterprise每日且可配置 | 达到阈值不立即执行，也不保证每次执行必有新模式 |
| 处理延迟 | 预计在计划运行后约24小时内出现新模式 | 不是开启后24小时必有结果，也不是硬 SLA |

例如用户已有100条旧记忆，今天才开启 Synthesis：不能仅由总数认定它会立刻对100条旧事实合成。又如应用一直给记忆附上 `run_id`，即使数量很多，这些项也不在纯 user scope 的合成范围。文档用四条训练事实演示怎样概括，只是内容例子，不覆盖正式的20条门槛。

模式与普通记忆同处读取集合，但“同处”不表示一次查询返回全部记录。相关性、scope、分页及生命周期仍影响结果。`synthesized` 表示产生方式，`active/superseded/merged` 表示生命周期状态；它不是第四种状态。文档默认读 active + superseded、隐藏 merged，`latest_only=true` 只保留 active，`include_merged=true` 扩大到 merged；两个开关同时使用的优先级未定义。模式以后能否再次作为合成来源或被标记替代，公开材料也没有解释。

实际观察来源时，Dream 的几个接口职责不同。以下为固定 OpenAPI 的路径后缀，统一前缀是 `/api/v1/orgs/organizations/{org_id}/projects/{project_id}/dream/`：

| 路径后缀 | 能看见什么 | 不能据此推定什么 |
|---|---|---|
| `runs/`，`runs/{run_id}/memories/` | 合成运行、产出模式与来源；运行列表用 next_cursor，单次运行内用 has_more/mem_cursor 翻页 | 运行列表不是完整失败／重试状态机；某页不含全量来源，来源 text 是截断摘录 |
| `memory/{memory_id}/sources/` | `memory_id`、`synthesized`，以及来源的 ID、文本和 lifecycle_state | 有来源链接不代表来源变化自动传播 |
| `activity/` | Supersede／Merge 活动 | 查不到 Synthesis 不是没有合成；该 feed 不包含合成产物 |
| `stats/` | 项目级 `last_run_at`、`processed_through`、`running`、`next_run_at` | 项目水位不能证明每个用户、每条模式都已刷新 |
| `preview/` | 有上限的样本／数量估算，不调用 LLM、不写入 | 不能检验实际模式质量 |

固定材料之间还存在表示缺口：v3 get-all 响应 schema 声明 `synthesized`，search schema 未声明这个字段；SDK 类型与 Dream 文档的生命周期选项也未完全对齐。Python 可以通过 kwargs 转发额外参数，不等于服务端接受和行为已验证。因此正文采用“普通读取可返回模式”的产品契约，不保证每种 search 响应都带识别模式与来源所需的全部字段；审计应回到明确的来源接口。`replaced_by` 是替代关系，也不能拿来当合成来源列表。

## Profiles：按 schema 保存和读取用户画像

Profiles 当前是按组织申请的 beta，项目还需开启并配置 schema；公开接口当前只支持 user，且不跨 API key 所属项目。主文档要求记忆带 `user_id`，没有声明与 Synthesis 相同的“不得附带其他 scope”规则，不能把那条限制移植过来。示例 notebook 另称需 Pro+，主文档与 OpenAPI 未提供同样完整的资格表，具体套餐边界仍需核实。

应用先定义想知道哪些字段及各自含义，例如：

```json
{
  "type": "object",
  "properties": {
    "current_goals": {
      "type": "array",
      "items": {"type": "string"},
      "description": "用户目前持续追求的目标，排除助手建议和已结束的目标"
    }
  }
}
```

字段描述是生成指令，不保证模型正确区分目标、建议与历史。文档给 schema 约10000 tokens 的序列化预算；完整 JSON Schema 关键字支持和输出强校验实现没有公开。REST 设置把 schema 放在 `entities.user.schema`，SDK 的 `update_profile_settings`／`updateProfileSettings` 才负责将平铺参数嵌套；只写传入字段，schema 的 null 表示清空配置，不等于已证明清空既有画像。

生成与读取是两个动作。自动刷新约每10条 **messages** 触发，不能换算成10条提取后的 memories；计数具体包含哪些消息未定义。也可调用 Python `generate_profile("alice")` 或对应 TS 方法，提交按需生成。作业受理不等于画像已经可用：`POST /v2/profiles/jobs/` 返回202后，可按 `status_url` 查询作业，再通过 `get_profile("alice")`／`GET /v2/entities/user/alice/profile/` 读取保存的结果。

这里有两套状态，应用不能混用：

- 作业状态为大写 `QUEUED/RUNNING/SUCCEEDED/PARTIALLY_SUCCEEDED/FAILED/CANCELLED`，表示一次执行的进度与终态。
- 画像读取状态为小写 `succeeded/pending/insufficient_data/not_enabled/failed`。主文档要求按 status 分支；`succeeded` 时才消费 profile，404表示该项目中没有这个用户实体。

读取 envelope 还给 `updated_at` 和 `generation_count`；profile 内部使用应用原来的字段名。TS 会转换系统 envelope 键的命名，但不改用户 schema 字段。应用可以把画像字段用于初始回答的个性化，而普通搜索仍用于“这次问题需要哪些具体事实”。这是两种输入组织方式，不是已证明一种普遍优于另一种。

主文档说刷新异步、增量进行：新 schema 下次生成才生效，旧画像在此之前保留原值；schema 移除字段，也是在下次生成时裁去字段。项目全量 rebuild/backfill 尚不可用。`sample_profiles` 最多覆盖10个真实实体，**会真实生成、保存并计费**，与 Dream 不写入的 preview 不同。

按需作业支持 `Idempotency-Key`，相同键重放同一请求可取回同一 job；同键用于不同请求则409，已有运行中作业也可能409，cooldown会返回429。SDK 省略该键时会新生成一个键。因此应用重试同一业务意图需复用原键，不能把再次调用且省略键当成同一作业，新键也不保证任务被受理。此处是请求去重契约，不代表服务端多实体写入具有全局事务；`PARTIALLY_SUCCEEDED` 本身也要求应用检查执行结果。

## 新鲜度、修改与删除必须分别判断

“生成过”与“已经反映最新事实”不同。Synthesis 有计划周期与项目级处理水位；Profiles 有更新时间和生成次数，但没有公开逐用户的输入记忆水位。主文档明确紧接 add 的读取可能仍显示旧画像或 pending，故 `succeeded` 不能读成“覆盖到这次读取前所有新增”。画像生成次数增长也不证明每条源记忆均被纳入。

这里还要先检查摄取是否完成。固定 notebook 警示：若 Platform add 尚未完成就触发 generate，作业即使成功也可能只使用旧 memories。它先检查 add event 成功，才生成画像。应用可据此按“确认摄取完成 → 提交生成并查询作业 → 单独读取画像、检查小写 status 与时间／计数”组织流程；这仍不构成逐条输入覆盖保证，示例中的固定等待秒数也不是同步 SLA。

固定 Profiles 材料需要分别看待：关闭后，主文档称读取 not_enabled，notebook却称已有画像仍可返回 succeeded，这是直接冲突。主文档描述约10条消息触发时无需等待schedule，notebook补充低于该阈值时有小时级flush，二者可以兼容，属于主文档未展开的旁路。增量刷新与整体JSON重写也可能描述不同层次，不能据文字裁决内部算法；insufficient_data是否涵盖queued仍有口径差异。因此不据这些材料定义精确刷新时钟、完整状态机或关闭后的可靠读取行为。原句和来源见[冲突记录](./conflict.md#6-profiles-主文档与示例-notebook-的行为表述不一致)。

修改与清理的边界如下：

| 操作 | 公开材料支持的判断 | 尚未闭合的传播问题 |
|---|---|---|
| 修改／删除来源 memory | 通用 memory CRUD 存在 | 两种派生产物是否撤回、重算、标失效，何时传播，均未见明确契约 |
| 来源被 supersede／merge 或过期 | 来源可有独立生命周期；模式来源接口可显示非 active 状态 | 不能推出模式或画像已经根据变化刷新 |
| 使用 `latest_only=true` | 按 Dream 生命周期只选 active | 不保证被选出的模式已基于最新来源重算 |
| 关闭 Synthesis | 明确保留已有记忆和模式 | 在途任务如何处理、降级后的行为未定义 |
| 关闭 Profiles | notebook 声称不删 schema 与已有画像 | 读取行为有冲突，不能当作擦除或隐藏的保证 |
| schema 删字段 | 下次画像生成时裁去该字段 | 不立即清除现有值，也不代表来源记忆被删 |
| `delete_all(user_id=...)` | 删除指定范围的 memories | 不等同于删除 user entity；不能据此保证 profile 被清 |
| 删除 user entity | notebook 声称 profile row 随实体级联；实体删除 API 只返回已排队 | 完成时点、在途生成、原子性、备份保留均未知 |

`delete_linked=True` 也不能填补上述缺口。Python 客户端说明面向被目标记忆替代的旧记忆链，OpenAPI 泛称 linked memories；两者都没有明确承诺“Synthesis 来源→模式”的级联删除，更没有声明它删除 Profiles。

例如把“正在备战马拉松”的来源删除后，当前证据只足以说主记录的删除流程存在；不能保证训练模式和画像目标字段同步消失，也不能断言服务端一定不处理。Synthesis 有可查来源这一优势，Profiles 公开读取却没有逐字段 source memory IDs、证据文本或历史 diff。应用能知道某次生成结束，并不等于能审计每个字段当前由哪些未删除记忆支持。

这些差别决定应用怎样使用派生结果：模式适合以来源为入口追查概括，画像适合按固定字段组装上下文；需要判断“为什么仍相信这个事实”或“删除是否已覆盖所有派生内容”时，当前公开契约不足以给完整保证。托管 worker 的输入快照、合并／撤回策略、重试与跨存储事务仍未公开，不能用固定 OSS 的内部实现补成答案。

## Evidence

- **Status**：固定官方文档与 OpenAPI 的产物、资格、接口和状态为 `Verified`（公开契约）；SDK 请求组装与类型覆盖为 `Observed`；使用含义及条件推演为 `Inferred`；Profiles 表述差异为 `Conflicting`。模型效果、服务端机制与删除传播为 `Unverified`，查不到契约不等于不存在实现。
- **Sources**：固定 [Dream](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/dream.mdx)、[Profiles](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/platform/features/user-profiles.mdx)、[OpenAPI](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/docs/openapi.json#L7211)、[Profiles notebook](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/examples/notebooks/user-profiles.ipynb)、[Python client](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0/client/main.py)、[TS client](https://github.com/mem0ai/mem0/blob/94c3fe9f238f3dbf29c9ce98643bd71eb13077cd/mem0-ts/src/client/mem0.ts)。
- **Trace**：延续 M30–M31；新增 [`notes/evidence.md`](./notes/evidence.md) M46–M50，精确来源与搜索边界见 [`notes/source.md §7`](./notes/source.md#7-派生记忆与-profiles-补查2026-11-02)。
- **Needs**：未运行 API、notebook 或模型；Profiles 文档差异需上游澄清，修改／删除传播需明确契约或受控行为证据。此处完成公开机制与边界解释，不代表 Mem0 对象整体深入研究结束。
