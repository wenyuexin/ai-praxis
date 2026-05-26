# Codex 文档冲突记录

> **校验文档**（L6）。本文是 codex 目录下**冲突清单的统一收口点**。涉及术语歧义、分数可比性、沙箱定义差异等问题，各主线文档将以本文为准。本文记录当前已识别的知识冲突和待核验项，不是最终结论。解决冲突后应更新相关文档并同步调整本文件。



> 根据 CONTRIBUTING.md L6 规范，记录 codex 目录范围内已识别的内容冲突。
> 本文件是**研究与修订输入**，冲突解决后应在相关文档修正并同步更新或移除对应记录。

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

## 冲突 2："沙箱"术语过载

| 字段 | 内容 |
|------|------|
| **冲突描述** | 两个文档使用同一术语"沙箱"指代两种不同的隔离模型：`codex-agent-mechanisms.md` 说沙箱是 OS 内核级（Landlock+Seccomp/Seatbelt），`codex-cloud-sandbox.md` 说 Cloud 沙箱是完全远程隔离。前者是本地进程隔离，后者是云端容器隔离，安全保证和适用场景不同。读者可能误以为两者的安全等级一致 |
| **涉及文档** | `codex-agent-mechanisms.md` 第 28-32 行（沙箱隔离节）、`codex-cloud-sandbox.md` 第 48-56 行（Cloud vs 本地沙箱对比表） |
| **冲突类型** | **术语** — 同名概念指代不同实体 |
| **当前证据** | 本地沙箱：OS 内核级（Landlock+Seccomp+bubblewrap / Seatbelt / ACL+WFP），按任务粒度。Cloud 沙箱：完全远程隔离，Setup 阶段有网络，Agent 阶段默认关闭，任务结束后销毁 |
| **待核验问题** | Cloud 沙箱底层的具体技术是什么？也是 Linux 内核隔离，还是容器化方案（如 Firecracker）？ |
| **建议处理方向** | 在 `codex-agent-mechanisms.md` 的沙箱节末尾增加说明："Cloud 沙箱与本地的隔离模型不同，详见 `codex-cloud-sandbox.md`"。两个文档都写上"注意：`沙箱` 一词涵盖两种隔离模式——详见 `conflict.md`" |

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

## 变更记录

- 2026-05-26：初始创建，记录 3 个已识别的冲突点。

*最后更新: 2026-05-26*