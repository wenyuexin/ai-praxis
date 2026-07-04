# GLM-5.x：版本演进与差异主线

## 文档信息

| 维度 | 内容 |
|------|------|
| 对象 | GLM-5.x |
| 文档类型 | Series evolution synthesis |
| 当前状态 | Cross-version synthesis based on `glm-5.md`, `glm-5.1.md`, and `glm-5.2.md` |
| 主要材料 | `GLM-5`、`GLM-5.1`、`GLM-5.2` 三篇版本对象文及其已整理的一手来源 |
| 最后整理 | 2026-07-04 |

---

## 一、这篇文档承接什么

这篇文档不替代：

- `overview.md` 的目录级入口职责
- `glm-5.md`、`glm-5.1.md`、`glm-5.2.md` 各自的版本对象研究职责

它只承接一件事：把 `GLM-5` 到 `GLM-5.2` 之间当前已经较稳的版本演进主线单独抽出来，方便比较“哪一版改变了什么、哪些是连续继承、哪些是增量增强”。

因此，它不是新的证据池，也不是某一版对象正文，而是系列内部的差异整理文。

## 二、当前最清晰的演进主线

如果只看当前已经整理过的公开材料，`GLM-5.x` 的演进方向可以先概括成三步：

1. `GLM-5`：建立从 `vibe coding` 转向 `agentic engineering` 的系列技术锚点
2. `GLM-5.1`：把“长程任务执行”更明确地变成版本对象身份，并补齐面向 agent workflow 的接口能力叙事
3. `GLM-5.2`：把长程任务叙事进一步推进到 `1M` 上下文、项目级工程执行与开放权重发布的更强版本形态

换句话说，这个系列目前不是在做简单的“聊天模型迭代”，而是在持续强化：

- 长链路任务执行
- coding / agentic engineering
- 工具调用与结构化输出
- 长上下文下的工程约束保持
- 更完整的开放发布与部署承接

## 三、分版本看，主变化在哪里

### 3.1 GLM-5：系列技术锚点

`GLM-5` 当前最特殊的地方，不只是“更早”，而是它直接承接了公开技术报告。

因此它在系列中的角色更像：

- 技术路线的基础版本
- `agentic engineering` 叙事的最直接报告锚点
- 后续版本做能力增强时的对照起点

在现有结构里，`GLM-5` 更负责回答：

- 这一代系列最初想解决什么问题
- 为什么路线会从单次代码生成转向更长程的工程执行
- MoE、sparse attention、长上下文效率优化等背景最初如何进入这条路线

### 3.2 GLM-5.1：把“长程任务版本对象”单独立起来

相对 `GLM-5`，`GLM-5.1` 的变化不是只有“参数更新”或“代码更强”。

当前更稳的变化主线包括：

- 明确把 `long-horizon task` / `agentic engineering` 写成版本对象的核心身份
- 把“连续执行、自主迭代、长链路保持”推到更前台
- 官方稳定给出一组 agent workflow 相关能力：`200K` 上下文、`128K` 最大输出、MCP、Function Call、Structured Output、Context Caching
- 公开发布形态开始更清楚地体现为“系列仓库 + 权重页 + 产品文档 + 官方博客”的组合对象

因此，`GLM-5.1` 像是在 `GLM-5` 的技术路线之上，把“长程任务模型”从系列口号推进成一个更可识别的版本对象。

### 3.3 GLM-5.2：把长程任务能力推向更强发布形态

相对 `GLM-5.1`，`GLM-5.2` 当前最清晰的升级集中在三件事：

- 上下文窗口从 `200K` 推进到 `1M`
- 官方叙事更强地转向项目级工程执行、长程重构、多步骤持续约束保持
- 开放权重、`MIT` 许可、Hugging Face / ModelScope / 系列仓库 / 产品文档的发布承接方式更完整可见

与此同时，`GLM-5.2` 还开始暴露更多公开技术线索，例如：

- `IndexShare`
- speculative decoding 相关优化
- 在超长上下文下控制 FLOPs 与提高 acceptance length 的方向

这些线索说明 `GLM-5.2` 不只是能力宣传更强，也开始在公开材料里暴露更多“为什么这一版能把长程任务推得更远”的技术端口。

## 四、横向对比：哪些能力是连续继承，哪些是明显升级

### 4.1 连续继承的主线

从 `GLM-5` 到 `GLM-5.2`，至少有几条线是持续不变或持续强化的：

- 系列一直围绕 `agentic engineering` 理解自己
- coding 能力强调的不只是一次生成，而是多步骤执行
- tool-use / workflow integration 一直是重要方向
- 长上下文不是孤立卖点，而是为长链路任务服务
- 大规模 MoE 与稀疏注意力相关路线始终是重要背景

### 4.2 明显升级的主线

当前最清楚的代际增强主要发生在：

- 长上下文规模：`GLM-5.1` 为 `200K`，`GLM-5.2` 为 `1M`
- 长程任务叙事强度：从系列技术路线，推进到版本对象级核心卖点
- 工程任务粒度：从一般 coding / tool-use，推进到项目级工程接管、迁移、重构与规范保持
- 公开发布形态：从系列技术报告锚点，逐步发展为更完整的开放权重发布与部署入口组合
- 技术可见度：`GLM-5.2` 比 `GLM-5.1` 暴露出更多可继续深挖的架构优化线索

## 五、当前最值得保留的版本边界

### 5.1 不要把 GLM-5 报告当成每一版的独立报告

当前一个最重要的边界是：

- `GLM-5` 有直接技术报告承接
- `GLM-5.1`、`GLM-5.2` 当前没有发现各自独立的 technical report
- 后续版本经常回引 `GLM-5` 报告作为系列背景

所以更稳的理解是：

- `GLM-5` 负责承接系列基础技术路线
- `GLM-5.1`、`GLM-5.2` 负责承接版本增量变化与公开发布信息

### 5.2 不要把版本规格写成跨版本静态事实

当前仍需要保守处理的边界包括：

- `GLM-5` 的训练细节、参数规模、benchmark 不能自动外推到 `GLM-5.1` 或 `GLM-5.2`
- `GLM-5.1` 与 `GLM-5.2` 各自公开页里的参数口径仍有待更直接材料统一
- “8 小时持续工作”“项目级工程接管”等表述，更适合写成官方能力叙事，而不是脱离任务条件的固定上限

### 5.3 不要把系列演进文误写成总览或单版正文

这篇文档的边界也要明确：

- 它不是 `overview.md`，不承担目录入口职责
- 它不是 `glm-5.md`、`glm-5.1.md`、`glm-5.2.md` 的替代品
- 它只负责抽取差异主线，不负责重新展开每一版的全部证据

## 六、一个更稳的阅读顺序

当前更适合的阅读路径是：

1. 先读 `overview.md`，确认这个目录如何分工
2. 再读 `glm-5.x-evolution.md`，先抓住系列演进主线
3. 再按需要进入 `glm-5.md`、`glm-5.1.md`、`glm-5.2.md` 看每一版的对象正文与证据边界

这样读可以避免两种常见误读：

- 误把某一版对象文当成整个系列说明书
- 误把系列技术路线背景写成每个版本都已独立验证的事实

## 七、后续最值得继续补强的比较问题

如果后续继续扩写，这篇文档最值得补强的方向包括：

1. `GLM-5.1` 到 `GLM-5.2` 的能力变化，哪些已有更直接官方对照材料
2. `200K` 到 `1M` 的上下文提升，是否伴随更明确的任务类型边界变化
3. `GLM-5.2` 暴露的架构优化线索，是否能在源码、模型卡或后续报告中找到更直接解释
4. 开放权重发布与本地部署支持，在 `GLM-5.1` 和 `GLM-5.2` 之间是否存在更具体的承接差异

## Evidence

- Status: Verified
- Sources:
  - `llm/02-models/open-source-models/glm/glm-5.md`
  - `llm/02-models/open-source-models/glm/glm-5.1.md`
  - `llm/02-models/open-source-models/glm/glm-5.2.md`
- Trace: This synthesis is derived from the already-written versioned object studies in the same directory. It does not introduce new external evidence; it extracts the cross-version evolution line that remains stable across the existing object studies and their cited first-party sources.
- Needs:
  - More direct official comparison materials between `GLM-5.1` and `GLM-5.2`
  - Stronger version-specific model cards or technical reports for later revisions of the comparison
