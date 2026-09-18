# DeepSeek Harness 插件系统

> 版本锚点：`0.1.0-rc.7`（commit `99f6f02fecdb`，2026-08-17）。本文基于本地 checkout 源码核验（`Status: Observed`，锚点见 `notes/evidence.md` C1-C5/C8/C9）；官方口径建档见 `notes/source.md` 一/二。

DeepSeek Harness（`dsh`）的核心设计主张是 **everything is a plugin**：模型适配器、工具注册表、会话日志、agent loop 本身都是插件，因此每一部分都可从配置替换，不存在需要打补丁的特权内核。扩展 dsh 的方式就是挂载插件，注册行为是可逆 effect，插件卸载时自动回卷。

## 1. 插件是什么

插件有三种形态，经 `ctx.plugin()` 归一化为可执行 callback（callback 是注册表的身份键）：

- **函数插件**：`(ctx, config) => any`，直接作为 callback。
- **类插件**：构造时 `new callback(ctx, config)`，随后执行 `[Service.init]` 钩子。
- **对象插件**：`{ apply(ctx, config) }`，取其 `apply` 方法。

插件元数据（`Plugin.Base`）包括：`name`（诊断用）、`Config`（standard-schema 校验器，config 在插件启动前经它校验，失败抛 `ValidationError` 聚合 issue）、`inject`（服务依赖声明）、`provide`（插件提供的服务名）。

## 2. 服务与 context

**context 是服务的仓库**：服务声明稳定的 `ctx.<key>`（如 `ctx.tools`、`ctx.llm`、`ctx.sessions`），其他插件按 key 找服务，不 import 具体实现。

- **服务注册即 effect**：`Service` 子类构造时调用 `ctx.reflect.provide(name, self, check)`，注册进当前 fiber 的 effect 列表——fiber 卸载时自动移除。同一 isolate 作用域内重复注册同名服务抛错（fail loud）。
- **`ctx.<key>` 解析走 Proxy**：属性读取经 `internal/get` waterfall，沿 fiber 父链向上找服务实现，**遇 isolate 标签变化即停止**。`ctx.isolate(name, label)` 创建独立服务作用域——同一个服务名在不同 isolate 标签下可有不同实现，互不污染（architecture.md 的 `isolate` realm 即此原语）。
- 可选服务用 `ctx.get(name)` 读取全局服务存储，不经过注入门控。

## 3. 生命周期：Fiber 状态机

每个插件实例是一个 **Fiber**，状态机：`PENDING`（等待依赖）→ `LOADING`（callback 运行）→ `ACTIVE`（加载并提供中）→ `UNLOADING`/`DISPOSED`；启动或配置失败进入 `FAILED`。

**inject 门控经 epoch 机制实现**：

1. 插件声明 `inject`（数组 = 只需服务存在；对象 = 服务名 → 该服务的 intercept config）。
2. 服务出现/消失时，`reflect.notify()` 对每个注入该服务的 fiber 重查 `_checkImpl`（含 `impl.check` 可用性谓词）与 `_refresh`（按注入服务的 fiber uid 链计算 epoch）。
3. epoch 变化 → 卸载 + 重载：**插件 callback 只在依赖齐备后运行，依赖变化时自动重启**（`ctx.inject()` 的文档语义"callback 在所需服务变化时卸载并重跑"即此）。

config 的解析发生在注入激活之后（经 `internal/config` waterfall），因此 config 表达式可以引用已注入的服务。

## 4. 事件系统

事件经声明合并定义，有五种分发模式：`emit`（同步不等待）、`parallel`（并行等待）、`serial`（按序等待至 bail）、`bail`（同步首值短路）、`waterfall`（环绕组合）。

**waterfall 是 around-middleware**：listener 收到 `(...args, next)`，调 `next()` 委托给链上下一环，不调 `next()` 直接返回即 veto 整链（含内置行为）；值经 `next()` 的返回值传播。单决策事件中 short-circuit 是设计意图——policy listener 拥有决策时可不委托，只观察的 listener 必须委托。

**事件注册即 effect**（`ctx.on` → `fiber.effect`）：listener 随注册它的 fiber 卸载自动移除；同一事件按注册序分发，`prepend: true` 可提前。

## 5. 配置驱动的组合（loader）

`cordis.yml` 的每一行（id/name/config/disabled/group/inject）成为 EntryTree 中的一个 **Entry**，每个 Entry 拥有独立子 context。组合层序由 context 原型链表达（`setPrototypeOf(entry.ctx, parent.ctx)`）。

- **启动桥接**：`Entry._start` → `ctx.registry.plugin(importedPlugin, config)`；entry 级 `inject` 字段经 `internal/plugin` listener 合并进 fiber 的依赖声明。
- **patch 层序**：profile 列出的各 bundle（按序）→ profile 的 `cordis.patch.yml` → home 级 patch → `--patch` overlay；patch 按 id 整体替换 config 或插入新行。
- **`disabled` 懒求值**：`!!js` 表达式在每次挂载决策时对 loader context 求值，且祖先 entry 的 disabled 链生效；`disabled` 是唯一可插值元数据字段，其余保持字面量。
- **config 插值边界**：`!!js` config 表达式仅在 entry root fiber 上、注入激活后解析；tree carrier（Group/Include）保持字面量。
- **事务性更新**：entry 更新走 import → dispose → start，失败回滚到旧插件；插件自毁（self-dispose）经 loader 判别后持久化为 `disabled: true`。

**重要前提**：`vendor/` 是 vendored 拷贝且带 18 条本地修改——loader/include 的组合语义（patch 不跨 include 边界、懒解析、disabled 插值）是"vendored + 本地修改后"的语义，不是上游 Cordis 文档的原义；研究 vendor/ 必须以 checkout 为准。

## 6. 与子领域文档的关系

- 插件生命周期与事件机制可迁移理解：`agentic/02-single-agent/` 的 agent loop、`04-skill-and-tool-systems/` 的插件/能力注册系统可引用本文件的三/四节。
- capability seam 三角色代码范式见 `capability-seam.md`（机制专题）；turn/step 流程见 `agent-loop.md`。

## Evidence

- **Status**: 本文件全部结论为 `Observed`（本地 checkout `0.1.0-rc.7` 源码直接核验，锚点见 `notes/evidence.md` C1-C5/C8/C9）；vendored 本地修改为 `Observed`（C8）。
- **Sources**: `vendor/cordis/src/{registry,fiber,context,reflect,events,service}.ts`；`vendor/loader/src/{index,config/entry}.ts`；`vendor/README.md` Local modifications。
- **Trace**: 2026-08-21 由 ai-praxis 侧深研脚手架回流提炼（机制主线插件层/组合层压实后）；版本锚点 `0.1.0-rc.7`。
- **Needs**: 若切换版本锚点，按 `notes/source.md` 5.4 联动纪律复查受影响项（当前高风险：loader 组合/PTC Mode 相关）。
