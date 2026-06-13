# README 规则

本文件定义仓库中 README 的出现条件、目录树展开深度、导航边界和类型化结构建议。

相关入口：[`CONTRIBUTING.md`](../../CONTRIBUTING.md) / [`docs/contributing/README.md`](./README.md) / [`元信息文件模型`](./metadata-files.md) / [`Evidence 与 Traceability 工作流`](./evidence-and-traceability.md)。

## 1. 展开深度

根 README 的目录树展开深度按内容活跃度区分：

- **核心领域**（traditional-ml、deep-learning、reinforce-learning、llm、cv、rag、agentic、knowledge-graph）展开到 4 级——内容充实、更新频繁，需要更细粒度的导航
- **次要领域**（embodied-intelligence、world-models、training-infra、interdisciplinarity、learning-materials）展开到 3 级——内容量较少或更新节奏较慢
- 各一级目录（顶层领域目录）有自己的 README，承载该领域的主要知识骨架（不受根 README 深度限制）

子目录 README 的目录树按所在层级控制展开深度：

- 根 README：按上面的活跃度规则展开
- 一级目录（顶层领域目录）README：允许展开到三级子目录，用于展示该领域主要知识骨架
- 二级及更深层目录 README：默认只记录自己的直接子目录

这样可以避免多个上级 README 重复维护同一批深层目录。

## 2. 出现条件

每个一级目录（顶层领域目录）下必须有 `README.md`。

二级及更深层目录存在或未来可能容纳子目录时，优先考虑补 README 以解决入口定向问题；如果当前只是过渡态容器或纯论文笔记集，可暂不写。

纯文件目录（仅含笔记文件，不会有子目录）不需要 README。

## 3. 通用规则

所有 README 必须遵守：

- **标题**：使用清晰自然、语义自解释的标题即可，不要求中英对照
- **目录结构**：必填，但按 README 所在层级控制展开深度；原则上目录树只展示目录，不展示文件
- **元信息位置**：如需标注 `最后更新`、来源、适用范围、状态等元信息，统一放在标题下方、正文开始之前
- **跨目录引用**：基础内容放在所属领域目录下，其他目录通过相对路径引用，避免重复维护

例外：用于说明组织结构的元信息文件，如 `README.md`、`template.md`，可在必要时出现在目录树中。

## 4. 根 README

根 README 面向首次访问者，结构建议：

- 仓库定位与一句话说明
- 层级关系表或 Mermaid 图（纵向技术栈 + 横向跨学科层）
- 各一级目录导航链接（带一句话定位）
- 目录结构（按活跃度区分展开深度）
- 如何阅读本仓库（可选，推荐路径或适合人群）

## 5. 一级目录 README

技术类一级目录建议包含（这些建议项按实际入口定向需求裁剪，README 仍以入口定向为核心，不应退化为领域综述）：

- 一句话定位
- 分类依据（解释二级目录的组织逻辑）
- 边界说明（什么放本目录、什么应放其他目录）
- 目录结构
- 同目录导航（优先提供正文主题、子目录入口或阅读路径，不主动列出元信息文件）
- 开源仓库与工具存放指南
- 学习路径或推荐阅读顺序
- 与其他目录的关系

跨学科类一级目录建议包含：

- 一句话定位引言
- 排序逻辑说明（放在目录结构之前，先建立框架再列细节）
- 目录结构（子目录按 **个体→系统→群体→制度** 递进排列；这类有稳定递进顺序的目录组可使用数字前缀）
- 各领域概览（用映射表关联纵向技术目录）
- 与纵向技术栈的关系

跨学科目录排序示例：

```text
个体层面   01 认知科学与神经科学
          02 语言学与语用学
          03 心灵哲学
系统层面   04 控制论与系统论
          05 复杂系统与涌现
群体层面   06 社会学与组织管理
          07 经济学与博弈论
制度层面   08 法学与治理
```

## 6. 子目录 README

论文集类子目录建议包含：

- 研究主题
- 创建日期
- 论文列表（推荐表格：论文、文档链接、arXiv ID、来源、核心贡献、重要性）
- 主题分类
- 阅读建议
- 工具与资源
- 相关资源
- 参考文献 BibTeX

重要性三级标注：⭐⭐⭐ 必读 / ⭐⭐ 值得关注 / ⭐ 简单参考。

技术主题类子目录建议包含：

- 目录结构（推荐表格：目录、说明，可加“状态”列标注填充进度）
- 领域特色内容
- 相关资源

## 7. README 导航边界

README 的核心职责是定向，不是替代正文或元信息文件。

README 中如需提供导航，优先列出正文主题、子目录入口或阅读路径。

README 是入口文件，但不是默认万能文件。只有当入口定向问题真实存在或已经出现时，才需要补强 README；即使 README 很强，也不替代 `overview.md` 或 `landscape.md`。

三者分工应保持清楚：

- `README.md`：回答“这是什么地方？跟我有什么关系？”
- `overview.md`：回答“如果只读一篇，这个主题最值得先理解什么？”
- `landscape.md`：回答“这个主题内部如何展开、如何切分、如何继续下钻？”

因此，README 不应因为命名自由就持续吞并其他职责；它不应演化为长篇综述，也不应承担系统性的结构化研究说明。

不要在 README 中主动列出以下元信息文件：

- `overview.md`
- `landscape.md`
- `backlog.md`
- `candidates.md`
- `roadmap.md`
- `conflict.md`

这些文件可以在正文、维护规则或特定上下文中被引用，但不应成为 README 导航的常规项目。尤其是 `landscape.md`：它虽然服务结构化研究和下钻协作，但仍不是 README 的替代物，也不应因为 README 天然显眼就被折叠回 README。

案例目录 README 存在例外：其中的 `overview.md` / `architecture.md` 可能是对象正文入口，处理方式见第 8 节。

## 8. 案例目录 README

案例目录（如 `project-studies/<case-name>/`）的 README 仍遵循"定向和导航"的核心职责，不替代正文。

案例目录 README 可以：

- 对重要案例，导航以下文件：
  - `overview.md` / `architecture.md` 等面向读者的对象正文入口
  - 机制专题文档（如 `runtime.md`、`sandbox.md`）
  - `notes.md` 等少量平铺辅助材料，或 `notes/` 统一辅助材料目录
- 说明哪些文件适合人类直接阅读，哪些只是研究辅助材料（服务于 traceability 和后续维护）
- 避免导航案例目录中的元信息文件（如 `backlog.md`、`candidates.md`、`conflict.md`）；这些文件仍遵循通用 README 规则

但 README **不应**：

- 承接长篇案例结论、源码流水或证据表
- 替代正文的 `Evidence` / `Gap` 段落

**与"README 不主动列出元信息文件"规则的关系**：

- `overview.md` / `architecture.md` 在案例目录中是**对象正文入口**，而不是七层元信息文件中的"理解"文件（`overview.md` 在子领域目录中属于 L3 元信息文件，但在案例目录中是读者面向的正文）。
- 因此，案例目录 README 导航 `overview.md` / `architecture.md` **不违**"不主动列出元信息文件"的规则。
- `notes.md`、`source-notes.md`、`evidence-notes.md` 或 `notes/` 属于研究辅助材料，不是七层元信息文件，也可在 README 中导航；但同一案例应避免同时维护根目录平铺辅助文件和 `notes/` 两套入口。
- 但案例目录中的元信息文件（`backlog.md`、`candidates.md`、`conflict.md`）仍遵循通用规则，README 不主动列出。

案例研究内部的三类产物组织方式，详见 [`research-artifacts.md`](./research-artifacts.md)。
