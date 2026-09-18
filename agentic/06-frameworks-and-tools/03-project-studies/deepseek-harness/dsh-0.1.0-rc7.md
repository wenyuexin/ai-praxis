# DeepSeek Harness 0.1.0-rc.7

> 版本化对象正文：承接 `0.1.0-rc.7` 版本锚点下的**整合视图**。机制级细节见各机制专题（`plugin-system.md` / `agent-loop.md` / `capability-seam.md` / `guard.md` / `approval.md`）；证据锚点见 `notes/evidence.md`（C1-C18）；版本谱系与差异见 `notes/source.md` 五。

## 版本锚点

- 版本：`0.1.0-rc.7`（release commit `bb4ca698d6`，tag `dsh-v0.1.0-rc.7`，2026-08-17）
- 本地 checkout：`/Users/wenyuexin/github/deepseek-harness`
- 状态：developer preview，上游声明存在兼容性破坏变更；**本文全部机制结论锚定此版本**，版本切换后按 `notes/source.md` 5.4 联动纪律复查

## 定位（源码核验版）

DeepSeek Harness 是 DeepSeek AI 开源的 agent harness（MIT），核心主张 **everything is a plugin**：模型适配器、工具注册表、会话日志、agent loop 本身都是插件，可从配置替换，无特权内核。运行时基于 vendored Cordis（`@deepseek-ai/cordis` 4.0.0-rc.7，带 18 条本地修改——研究必须以 checkout 为准，上游文档不是权威）。

名字中的 "harness" 指 agent 运行时/外壳，不是 evaluation harness。

## 架构整合视图（0.1.0-rc.7）

```text
配置组合（boot 级）
  profile → bundle 层序 → patch 覆盖（loader/include，C9）
  └─ agent 创建时挂载 per-session preset（PresetTree include + 两道 guard，C18）
        │
        ▼
插件运行时（Cordis，C1-C5）
  Fiber 生命周期状态机（PENDING→LOADING→ACTIVE）
  inject epoch 门控（依赖齐备才运行、变化自动重启）
  ctx.<key> Proxy 解析 + isolate 边界（isolate = 独立服务作用域）
  事件系统：emit/parallel/serial/bail/waterfall（waterfall 不调 next() 即 veto）
  config standard-schema 校验 + 注入激活后解析
        │
        ▼
agent-loop（C11/C15/C17）
  turn/step 流程（ReactLoopAgent，唯一 loop 插件）
  session log：append-only 事件流 + surface 派生历史（model-visible ⟺ logged）
  system-prompt：四类注册面 + order + complete 节
  agent/request-error waterfall、request/header+request/context 持久事件（文档未覆盖）
        │
        ▼
capability seam（C6/C7/C12）
  Service Definition（抽象类 extends Service + declare module 合并）
  Service Provider（子类化 + resolve(request): Spec 显式默认化）
  Consumer（ctx.inject/ctx.get 接入）
  图谱（60+ 服务）为包级权威映射
        │
        ▼
安全治理（C13/C14）
  guard：timeout-policy + repeat-tool-reminder（纯 listener 插件）
  approval：scope-filtered waterfall + 三层 fail-closed + 审计对 + 持久策略事件
        │
        ▼
对外投影（C16）
  ACP server（agents 依赖 + 只输出已提交消息 + approval answerer）
  hooks 桥接（CC/Codex 事件 → dsh 扩展点 6 映射）
  SDK（JSON-RPC，协议面）
```

## 机制专题索引（机制级细节入口）

| 专题 | 内容 | 证据 |
|---|---|---|
| `plugin-system.md` | 插件三形态、Fiber 生命周期、ctx 解析、事件、loader 组合 | C1-C5/C8/C9 |
| `agent-loop.md` | turn/step 流程、请求组装、扩展点清单 | C11 |
| `capability-seam.md` | 三角色代码范式、图谱、新增能力检查清单 | C6/C7/C12 |
| `guard.md` | tool-timeout + 重复调用检测 | C13 |
| `approval.md` | 决策链、fail-closed、审计 | C14 |

## 本版本特有信息

- **命名状态**：`self-modification/` 组已于 2026-08-11 改名 `extensions/`（根 AGENTS.md 未同步，`Deprecated`；`tool-cordis` 即其工具面，C10）。
- **rc.6→rc.7 机制相关变更**（采样）：PTC Mode 改名（preset 概念）、ACP 图片 prompt 桥接、agent-presets 后台 subagent 任务（`notes/source.md` 5.2）。
- **已知限制**（源码 TODO）：hooks `updatedInput` 不 honored、Stop 续跑无 loop guard、SessionStart 无 startup gate；timeout-policy 计划改名 `dsh-timeout-guard`。

## Evidence

- **Status**: 本文为版本整合视图；机制结论均 `Observed`（源码核验，锚点见 `notes/evidence.md` C1-C18），版本谱系为 `Observed`（`notes/source.md` 五）。
- **Sources**: 本地 checkout（`vendor/` + `packages/` + `.agents/notes/`）；上游 `https://github.com/deepseek-ai/deepseek-harness`。
- **Trace**: 2026-08-21 由 ai-praxis 侧研究回流整合；版本锚点 `0.1.0-rc.7`（`bb4ca698d6`）。
- **Needs**: 版本切换后按 `notes/source.md` 5.4 复查 C1-C18 受影响项（当前高风险：C9/C10/C11）；版本级待核验点见 `notes/deep-research-question-tree.md` §7。
