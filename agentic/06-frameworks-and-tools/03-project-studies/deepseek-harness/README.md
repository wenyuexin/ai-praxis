# DeepSeek Harness

*最后更新: 2026-08-21*

DeepSeek Harness（`dsh`）是 DeepSeek AI 开源（MIT 许可）的 agent harness / agent 运行时：插件化架构（everything is a plugin），基于 vendored Cordis，提供 session、system-prompt、tools、agent-loop，以及 shell / fs / lsp / web / subagent / workflow / plan / todo 等 capability，并附带 Web UI、preset 组合与 ACP server。本目录用于集中沉淀 DeepSeek Harness 的项目级研究；其他专题目录只按需要克制回填与其机制直接相关的证据。

注意：名字中的 "harness" 指 agent 运行时/外壳，不是 evaluation harness；不要与评估框架（`llm/05-evaluation/` 或 `agentic/07-evaluation/` 下的对象）混淆。

## 研究边界

本目录优先回答：

- dsh 的项目定位、插件化架构（vendored Cordis）与 capability seam 模型。
- 版本演进：developer preview 阶段的破坏性变更如何改变能力边界与研究结论。
- 官方文档能直接支撑哪些结论，哪些问题必须进入源码阶段核验。
- dsh 对 agentic 工程系统（preset 组合、self-modification、session 持久化、ACP）的可迁移设计启发。

不在本目录泛化为所有 agent harness 的通用结论；跨项目比较应进入 `../05-comparisons/`，skill / tool / 插件能力注册系统的主线归纳应回填到 `../../04-skill-and-tool-systems/` 下对应专题。

## 当前阶段

- **骨架建立（2026-08-21）**：目录与初始文档骨架已建立，版本锚点为 `0.1.0-rc.7`（commit `99f6f02fecdb`，2026-08-17）。官方资料建档与源码核验尚未开始。

## 目录结构

```text
deepseek-harness/
├── README.md                # 本页：目录定向与导航
├── overview.md              # 目录级系列总览：定位、版本关系与研究边界
├── plugin-system.md         # 机制专题：插件系统（everything-is-a-plugin 落地机制）
├── agent-loop.md            # 机制专题：turn/step 流程（ReactLoopAgent）
├── capability-seam.md       # 机制专题：capability seam 三角色代码范式
├── guard.md                 # 机制专题：执行卫生（tool-timeout + 重复调用检测）
├── approval.md              # 机制专题：决策闸门（approval 决策链与审计）
├── dsh-0.1.0-rc7.md         # 版本化对象正文：0.1.0-rc.7 锚点
└── notes/                   # 研究辅助材料入口（不参与知识主线）
```

## 阅读入口

推荐阅读顺序：先读 `overview.md` 建立对象定位与版本意识；进入机制级理解时先读 `plugin-system.md`（一切机制的底座），再读 `agent-loop.md`（一次任务如何跑起来）与 `capability-seam.md`（能力如何组织）；安全治理看 `guard.md` 与 `approval.md`；需要追溯证据时进入 `notes/`。

- **`overview.md`**（推荐优先阅读）：面向读者的目录级总览。回答 dsh 是什么、核心设计主张是什么、为什么必须按版本研究。
- `plugin-system.md`：机制专题。插件三形态、Fiber 生命周期与 inject 门控、ctx 服务解析、事件系统、loader 配置组合——everything-is-a-plugin 的落地机制。
- `agent-loop.md`：机制专题。turn/step 完整流程、请求组装、扩展点（waterfall 事件）清单，含两处官方序图未覆盖的细化。
- `capability-seam.md`：机制专题。seam 三角色代码范式（Service Definition / Provider / Consumer），含新增能力检查清单。
- `guard.md`：机制专题。工具超时强制与重复调用检测（两个纯 listener 插件）。
- `approval.md`：机制专题。approval 决策链、三层 fail-closed、审计与策略持久化。
- `dsh-0.1.0-rc7.md`：版本化对象正文。承接当前版本锚点（`0.1.0-rc.7`）下的对象研究。
- `notes/`：研究辅助材料入口，承接源码核验、证据表与未闭合问题，服务于 traceability 与后续维护；不作为主要阅读入口。

## 与专题目录的关系

- `agentic/04-skill-and-tool-systems/`：dsh 的插件 / skill / 能力注册机制相关证据在主线归纳需要时回填。
- `agentic/02-single-agent/`、`agentic/05-environments/`：仅在后续研究确实涉及 agent loop、tool-use、sandbox 等机制时做少量证据引用。
- `agentic/06-frameworks-and-tools/05-comparisons/`：跨对象横向比较放此处，本目录不泛化结论。
