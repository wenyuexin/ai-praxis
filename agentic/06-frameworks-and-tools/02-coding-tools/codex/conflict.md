# Codex 文档冲突记录

> **校验文档**（L7）。本文是 `codex/` 目录下**冲突清单的统一收口点**。涉及术语歧义、分数可比性、沙箱定义差异等问题时，可先在此登记并收敛核验方向。本文记录当前已识别的知识冲突和待核验项，不是最终结论。冲突解决后应更新相关文档并同步调整本文件。

> 根据 `CONTRIBUTING.md` 与 `metadata-files.md` 的 `conflict.md` 规则，记录 `codex/` 目录范围内已识别的内容冲突。
> 本文件是**研究与修订输入**，不替代主线综述、专题结论或问题缺口列表。

---

## 冲突 1：SWE-bench 分数看似矛盾

| 字段 | 内容 |
|------|------|
| **冲突描述** | `codex-agent-evidence.md` 同时列出 GPT-5.2 在 SWE-bench Verified 上得 80.0%，和 GPT-5.3-Codex 在 SWE-bench Pro 上得 56.8%。两个分数来自不同难度等级的 benchmark，但行文未明确标注难度差异。读者可能误以为"新版模型性能倒退" |
| **涉及文档** | `codex-agent-evidence.md` 第 18-19 行（Benchmark 数据表） |
| **冲突类型** | **事实** — 两个分数的可比性未交代 |
| **当前证据** | SWE-bench Verified 是标准集（GPT-5.2 80.0%），SWE-bench Pro 是更难的真实场景集（GPT-5.3-Codex 56.8%）。Pro 难度更高，因此分数低不表示模型差 |
| **待核验问题** | SWE-bench Verified vs Pro 的具体难度差异有无官方说明？是否还有其他版本的数据？ |
| **建议处理方向** | 在 Benchmark 数据表下方添加注释行，明确标注"SWE-bench Pro 难度高于 Verified，分数不可直接比较" |

---

## 冲突 2：`沙箱` 术语仍有残余歧义，但主线已基本对齐

| 字段 | 内容 |
|------|------|
| **冲突描述** | 旧版本文档曾用同一术语`沙箱`混指两种隔离模型：本地 OS 级受限执行边界，与 Cloud 远程托管容器边界。当前主线文档已显式拆分这两层含义，但读者仍可能把两者误解为同一种安全模型。 |
| **涉及文档** | `codex-agent-mechanisms.md` 第 17 行后（治理层 / 沙箱与执行边界）、`codex-cloud-sandbox.md` 第 25 行后（Cloud 容器与本地沙箱边界） |
| **冲突类型** | **术语** — 同名概念曾指代不同实体，现已部分收敛为低强度未决项 |
| **当前证据** | 当前较稳的一手口径已对齐：1) 本地执行路径由 OS-level mechanisms 约束，默认无网络、默认只允许写活动 workspace；2) Cloud 运行在 `isolated OpenAI-managed containers` 中；3) Cloud 采用 `setup → agent phase` 两阶段，Setup 可联网，Agent 默认离线，Secrets 仅在 Setup 阶段可用。 |
| **临时检索摘要** | 1) `codex-agent-mechanisms.md` 已不再把 Cloud 写成本地 OS 沙箱的同类实现；2) `codex-cloud-sandbox.md` 已把 Cloud / Local / Worktree 的运行位置与边界拆开；3) 当前剩余问题主要是 Cloud 底层容器/虚拟化技术与更细数据留存规格尚未公开打实。 |
| **待核验问题** | 1) Cloud 沙箱底层具体技术是否有公开说明；2) 环境销毁、缓存寿命、数据留存是否有更明确官方规格；3) App 本地执行路径是否有比产品页更细的一手定义。 |
| **建议处理方向** | 将本冲突下调为“低强度未决项”：主线继续沿用“本地 OS 级边界 vs Cloud 远程容器边界”的双层口径；冲突区仅继续跟踪底层实现、留存规格与 App 边界。 |
| **已解决项（2026-06-04）** | 1) `codex-agent-mechanisms.md` 与 `codex-cloud-sandbox.md` 已完成主边界对齐；2) `approval policy` 与 `sandbox mode` 已在机制文档中分开写；3) Local / Worktree / Cloud 的运行位置已在 Cloud 专题中明确拆开。 |
| **未解决项（持续跟踪）** | 1) Cloud 底层隔离技术名称；2) 缓存寿命、环境销毁与数据留存的稳定公开规格；3) App 本地执行模型的更细一手定义。 |

---

## 冲突 3：SWE-bench Pro 结论表述模糊

| 字段 | 内容 |
|------|------|
| **冲突描述** | `codex-agent-evidence.md` 关键事实结论中写了"SWE-bench Pro 真实场景表现优于 Verified 标称值"，其中"表现优于"易被误解为"Codex 在 Pro 上得分更高"。实际含义是"SWE-bench Pro 作为评估场景，比 Verified 更接近真实开发"，不是分数更高 |
| **涉及文档** | `codex-agent-evidence.md` 第 32 行（关键事实结论表） |
| **冲突类型** | **结论** — 表述存在歧义 |
| **当前证据** | Pro 得分（56.8%）低于 Verified（80.0%），所以不是"分数更高"；"表现优于"指的是 benchmark 设计本身的场景真实度 |
| **待核验问题** | 是否可以从 SWE-bench 官方文档确认 Pro 的具体设计差异？ |
| **建议处理方向** | 将结论行改为"SWE-bench Pro 比 Verified 更接近真实开发场景（评测设计差异，非分数对比）"，消除歧义。两个分数中间加一列标注难度等级 |

---

## 冲突 4："本地执行 Python 工具 / 本地 MCP"能力口径冲突

| 字段 | 内容 |
|------|------|
| **冲突描述** | 基于 `https://github.com/openai/codex` 仓库核验，旧冲突已部分收敛：Codex CLI 存在明确本地执行路径，MCP 在代码配置层明确为 `stdio` + `streamable_http`。当前主要剩余的是术语精度问题（是否写成 HTTP/SSE）与 App 边界说明不足。 |
| **涉及文档** | `https://github.com/openai/codex`（重点核验 `README.md`、`codex-rs/config/src/mcp_types.rs`、`codex-rs/cli/src/main.rs`、`codex-rs/core/src/tools/handlers/shell.rs`）；`agentic/06-frameworks-and-tools/02-coding-tools/codex/overview.md`；`agentic/06-frameworks-and-tools/02-coding-tools/codex/codex-agent-mechanisms.md` |
| **冲突类型** | **术语 + 边界** — 事实冲突已显著下降，重点转为“传输术语精确化”和“产品形态边界补证” |
| **当前证据** | `https://github.com/openai/codex` 仓库给出一手证据：1) CLI README 明确本地运行；2) MCP 类型为 `Stdio` 与 `StreamableHttp`；3) CLI 子命令含 `exec`、`sandbox`、`app`、`cloud`；4) shell handler 与多平台 shell 测试证明本地命令执行是设计能力。 |
| **临时检索摘要** | 1) 可将“CLI 本地执行”从候选结论升级为主线事实；2) 可将“MCP 双形态”升级为主线事实，但表述建议用 `stdio + streamable_http`；3) Code Interpreter 不可外推到 CLI 的边界结论继续成立；4) App 执行细节仍弱证据，避免写死。 |
| **待核验问题** | 1) Codex App 的执行模型是否有独立官方文档可直接引用；2) MCP 是否在官方对外文案中将 `streamable_http` 进一步表述为 HTTP/SSE；3) local socket（非 stdio）是否被官方支持。 |
| **建议处理方向** | 将冲突 4 下调为“低强度未决项”：主线吸收已核实事实（CLI 本地执行、MCP `stdio + streamable_http`、CI 不可外推），仅保留 App 边界与术语映射问题在冲突区继续追踪。 |
| **已解决项（2026-05-28）** | 1) CLI 是否存在本地执行路径：已解决；2) MCP 是否仅远程：已解决（仓库证据为 `stdio + streamable_http`）；3) Code Interpreter 结论能否外推 CLI：已解决（不可直接外推）。 |
| **未解决项（持续跟踪）** | 1) Codex App 执行模型的一手官方定义仍不足；2) 外部文案 `HTTP/SSE` 与仓库术语 `streamable_http` 的映射关系需持续核验；3) local socket（非 stdio）是否被官方支持仍待证据。 |
| **App 边界证据清单** | **已证实**：CLI 本地执行、Web 云端 agent、`codex app` 为桌面入口。**未证实**：App 执行模型是否与 CLI 完全一致、App 是否开放独立 MCP 连接模式。 |

---

## 变更记录

- 2026-05-26：初始创建，记录 3 个已识别的冲突点。
- 2026-05-27：新增冲突 4（本地执行 Python 工具 / 本地 MCP 能力口径冲突），来源外部联网检索结果。
- 2026-05-28：根据外部检索新版报告更新冲突 4，加入“结论反转与证据稳定性”核验要求。
- 2026-05-28：冲突 4 建议处理方向补充“部分落地状态”（主线已并入双层口径，结论仍待逐链接复核）。
- 2026-05-28：根据外部检索覆盖版报告更新冲突 4 状态：主线可吸收稳定结论，冲突收敛为“边界细节待完善”。
- 2026-05-28：引入 `https://github.com/openai/codex` 一手证据，冲突 4 进一步收敛为“低强度未决项”（重点跟踪 App 边界与术语映射）。
- 2026-05-28：为冲突 4 增加“已解决项/未解决项”清单，便于后续维护与增量核验。
- 2026-06-04：根据 `codex-agent-mechanisms.md` 与 `codex-cloud-sandbox.md` 的新一轮对齐结果，收紧冲突 2；将其从主线冲突下调为低强度未决项，继续只跟踪 Cloud 底层技术与留存规格问题。

*最后更新: 2026-06-04*