# GLM 系列研究对象目录

本目录承接 `GLM` 系列开源 / 公开模型的对象研究，尤其是 `GLM-5.x` 这类具有明显版本演进特征的对象。

## 边界说明

- 本目录不负责重新讲解 LLM 基础理论（预训练、微调、对齐等），这些回到 `llm/` 对应层级
- 本目录只承接 GLM 系列作为研究对象的版本研究、演进差异与证据边界

## 入口说明

- 第一次进入本目录时，先用本页判断它是否与你当前的问题匹配
- 想理解 GLM 系列的整体定位与版本关系时，转到 [`overview.md`](./overview.md)
- 想查版本演进差异主线时，转到 [`glm-5.x-evolution.md`](./glm-5.x-evolution.md)
- 想看某一版对象正文时，直接进入 `glm-5.md` / `glm-5.1.md` / `glm-5.2.md`
- 想直接研究 `GLM-5` 原报告本身的技术贡献、训练组织与论证结构时，转到 [`glm-5-report.md`](./glm-5-report.md)

## 当前文档分工

- `overview.md`：GLM / GLM-5.x 系列总览，承接系列级主线判断
- `glm-5.x-evolution.md`：GLM-5 到 GLM-5.2 的版本演进与差异主线
- `glm-5.md`：GLM-5 基础版本对象文，也是当前系列技术报告最直接承接的版本节点
- `glm-5-report.md`：GLM-5 原报告对象文，承接报告本身的问题、技术贡献与论证结构
- `glm-5.1.md`：GLM-5.1 版本对象正文
- `glm-5.2.md`：GLM-5.2 版本对象正文
- `notes/`：承接系列内已稳定出现、但暂不足以直接写成版本正文定论的问题层

后续如继续研究其他版本，应继续使用带版本标识的对象文件名，而不是把它们混写进目录级总览。

## 命名规则

- 目录级总览统一使用 `overview.md`
- 具体版本对象正文优先使用带版本标识的对象文件名，如 `glm-5.md`、`glm-5.1.md`、`glm-5.2.md`
- 只有当文档主对象是明确存在的上游 report / technical report / system card / whitepaper 时，才使用 `*-report.md`
- 不因为版本对象文承担快速理解功能，就命名成 `*-overview.md`
- 不因为对象研究引用了某份 report，就默认命名成 `*-report.md`

## `notes/` 的角色

当前目录已经启用 `notes/`，但它不是新的正文层，也不是 `temp/` 的镜像备份。

`notes/` 主要承接：

- 系列内反复出现的问题边界
- 已确定属于 `GLM` / `GLM-5.x`，但暂不足以直接写成对象正文定论的材料
- 对后续版本对象文和系列差异文都有影响的稳定问题层

当前已有五类问题笔记：

- `notes/report-boundary.md`：GLM-5 报告与后续版本的引用边界
- `notes/parameter-wording.md`：参数口径冲突与保守写法
- `notes/release-stages.md`：发布阶段 vs 单一发布日期
- `notes/architecture-clues.md`：架构线索的分层处理
- `notes/variant-boundary.md`：主版本 vs 服务/部署变体边界

## 后续可扩方向

- 后续其他带版本标识的对象文
- `notes/` 下更多按问题组织的辅助材料

---

*最后更新: 2026-07-04*
