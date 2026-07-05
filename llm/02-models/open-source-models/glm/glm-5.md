# GLM-5：从 vibe coding 到 agentic engineering 的基础版本对象

## 文档信息

| 维度 | 内容 |
|------|------|
| 对象 | GLM-5 |
| 文档类型 | Versioned object study |
| 当前状态 | Versioned object study based on series technical report, official release pages, and downstream version references |
| 主要材料 | GLM-5 技术报告、系列发布页、后续版本对象文中可确认的系列背景 |
| 最后整理 | 2026-07-01 |

---

## 一、当前可确认的定位

`GLM-5` 是 `GLM-5.x` 系列中的基础版本对象，也是当前公开技术报告最直接承接的那个版本节点。

从现有公开材料看，`GLM-5` 的关键定位在于：

- 它不是一个单纯的“代码模型版本”
- 它被明确放在 `agentic engineering` 的叙事里理解
- 它代表了从 `vibe coding` 向更长程、更结构化、更可持续执行的智能体工程范式转向

因此，在 `GLM` 系列中，`GLM-5` 不只是前代背景，也应被当作一个独立版本对象来研究。

## 二、GLM-5 版本对象的技术背景来源

当前最关键的事实是：

- 公开的系列技术报告直接对应的是 `GLM-5`
- 报告标题为 `GLM-5: from Vibe Coding to Agentic Engineering`
- arXiv: `2602.15763`

这意味着 `GLM-5` 相比 `GLM-5.1`、`GLM-5.2` 更适合直接承接“系列技术路线”的正文描述。

基于当前可见材料，较稳的技术背景包括：

- `GLM-5` 明确将技术路线放在 `agentic engineering` 方向
- 长程任务、coding、tool-use 与持续执行是这一代路线的核心叙事
- 系列中采用大规模 MoE 与稀疏注意力相关路线来处理长上下文与成本问题
- DSA / sparse attention、长上下文效率优化等，是这一代路线的重要背景线索

这些信息可以较稳地作为 `GLM-5` 本体事实或直接技术背景来写；而对 `GLM-5.1`、`GLM-5.2`，则应保守地视为前代背景，不直接外推。

## 三、GLM-5 在系列中的角色

相对于后续版本，`GLM-5` 的角色更像：

- 系列技术路线的基线版本
- 报告、权重页与公开叙事的基础锚点
- 后续 `GLM-5.1`、`GLM-5.2` 做版本增强时的对照起点

因此它和 `GLM-5.1`、`GLM-5.2` 的关系不是“被后者覆盖掉”，而是：

- `GLM-5` 负责承接更基础的系列背景
- `GLM-5.1`、`GLM-5.2` 负责承接版本对象的公开变化、能力增强与发布形态差异

## 四、当前不应写得过满的地方

虽然 `GLM-5` 有更直接的技术报告承接，但现阶段仍应保持以下边界：

- 不自动把 `GLM-5` 报告里的所有数字和训练细节，外推到 `GLM-5.1` 或 `GLM-5.2`
- 不因为后续版本引用了 `GLM-5` 报告，就把它们命名成 `*-report.md`
- 不把 `GLM-5` 的技术路线背景，直接当作整个 `GLM-5.x` 系列每一版都已独立验证的事实

## 五、与后续版本的阅读关系

阅读顺序上，更合理的路径通常是：

1. 先读 `overview.md` 了解系列结构
2. 再读 `glm-5.md` 了解系列技术锚点
3. 再根据需要进入：
   - `glm-5.1.md`
   - `glm-5.2.md`

这样可以避免把后续版本文误当成整个系列的基础说明书，也避免把 `GLM-5` 技术报告误当成后续每一版的直接事实文档。

## Evidence

- Status: Verified
- Sources:
  - https://arxiv.org/abs/2602.15763
  - https://huggingface.co/zai-org/GLM-5.1
  - https://huggingface.co/zai-org/GLM-5.2
  - https://github.com/zai-org/GLM-5
- Trace: This versioned object study treats GLM-5 as the direct report-bearing anchor version in the GLM-5.x series. Later version pages are used only to confirm that subsequent objects cite back to the GLM-5 report as series-level background.
- Needs:
  - More direct release-page evidence specific to GLM-5 open-weight publication form
  - Clearer separation between GLM-5 direct specs and series-level background where later pages only cite the report
